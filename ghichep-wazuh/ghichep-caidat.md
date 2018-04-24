## Cài đặt Wazuh

### 1. Cài đặt Wazuh server

 - Thêm repo :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=CentOS-$releasever - Wazuh
baseurl=https://packages.wazuh.com/yum/el/$releasever/$basearch
protect=1
EOF
```

 - Cài đặt Wazuh manager :
```sh
yum install wazuh-manager -y
```

 - Kiểm tra status của wazuh-manager :
	- Với Systemd :
	```sh
	systemctl status wazuh-manager
	```
	
	- Với SysV init :
	```sh
	service wazuh-manager status
	```
	
### 2. Cài đặt Wazuh API

NodeJS >= 4.6.1 là yêu cầu để chạy Wazuh API. Nếu không có NodeJS đã cài đặt, hoặc version < 4.6.1, nên cài đặt NodeJS theo repo sau : 
```sh
curl --silent --location https://rpm.nodesource.com/setup_6.x | bash -
yum install nodejs -y
```

 - Cài đặt Wazuh API. Nó sẽ update NodeJS nếu nó được yêu cầu : 
```sh
yum install wazuh-api -y
```
 - Kiểm tra status của wazuh-api :
	- Với Systemd :
	```sh
	systemctl status wazuh-api
	```
	
	- Với SysV init :
	```sh
	service wazuh-api status
	```

Python >= 2.7 được yêu cầu. Nó được cài đặt theo mặc định.

Có thể set Python path cho API trên `/var/ossec/api/configuration/config.js`, trong TH nếu version của Python quá cũ : 
```sh
config.python = [
    // Default installation
    {
        bin: "python",
        lib: ""
    },
    // Package 'python27' for CentOS 6
    {
        bin: "/opt/rh/python27/root/usr/bin/python",
        lib: "/opt/rh/python27/root/usr/lib64"
    }
];
```

Centos 6 và Red Hat 6 với Python 2.6, bạn có thể cài đặt Python 2.7 song song cùng với bản cũ :

 - Với Centos 6 :
```sh
yum install -y centos-release-scl
yum install -y python27
```

 - Với RHEL 6 :
```sh
yum install -y python27
```

### 3. Cài đặt Filebeat

Filebeat giúp Wazuh server vận chuyển các cảnh báo và archived event một cách an toàn tới Logstash service trên Elastic Stack server.

**Chú ý** : Với mô hình all-in-one, có thể bỏ qua việc cài Filebeat vì Logstash có thể đọc event/alert trực tiếp từ local filesystem.

 - Cài đặt GPG keys từ Elastic và thêm Elastic repo :
```sh
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

cat > /etc/yum.repos.d/elastic.repo << EOF
[elastic-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

 - Cài đặt Filebeat :
```sh
yum install filebeat -y
```

 - Download Filebeat config file từ Wazuh repo để cấu hình chỏ Wazuh alert tới Logstash :
```sh
curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/2.0/extensions/filebeat/filebeat.yml
```

 - Sửa file `/etc/filebeat/filebeat.yml` và thay thế `ELASTIC_SERVER_IP` với IP của máy Elastic stack.
```sh
output:
  logstash:
    hosts: ["ELASTIC_SERVER_IP:5000"]
```

 - Enable và start filebeat 
	 - Với Systemd
	```sh
	systemctl daemon-reload
	systemctl enable filebeat.service
	systemctl start filebeat.service
	```
	
	 - Với SysV init
	```sh
	chkconfig --add filebeat
	service filebeat start
	```
	
Khi đã cấu hình xong manager, API và Filebeat (nếu cần thiết), bạn có thể cài đặt Elastic Stack.

Tham khảo việc cài đặt Elastic Stack tại [đây]()

 - Load Wazuh Elasticsearch template : 
```sh
curl https://raw.githubusercontent.com/wazuh/wazuh-kibana-app/master/server/startup/integration_files/template_file.json | curl -XPUT 'http://localhost:9200/_template/wazuh' -H 'Content-Type: application/json' -d @-
```

 - Insert sample alert :
```sh
curl https://raw.githubusercontent.com/wazuh/wazuh-kibana-app/master/server/startup/integration_files/alert_sample.json | curl -XPUT "http://localhost:9200/wazuh-alerts-"`date +%Y.%m.%d`"/wazuh/sample" -H 'Content-Type: application/json' -d @-
```

 - Dowload Wazuh config và template file cho Logstash :
```sh
curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/2.0/extensions/logstash/01-wazuh.conf
curl -so /etc/logstash/wazuh-elastic5-template.json https://raw.githubusercontent.com/wazuh/wazuh/2.0/extensions/elasticsearch/wazuh-elastic5-template.json
```

 - Làm bước sau nếu mô hình cài đặt là `All-in-one` :
	 - Sửa `/etc/logstash/conf.d/01-wazuh.conf`, comment input section `Remote Wazuh Manager - Filebeat input` và uncomment `Local Wazuh Manager - JSON file input`. Lúc này Logstash sẽ được Wazuh `alerts.json` file trực tiếp từ local filesystem.
	 - Bởi vì Logstash user cần đọc alerts.json file, add OSSEC group :
```sh
$ usermod -a -G ossec logstash

 If you use CentOS-6/RHEL-6 or Amazon AMI, in this versions Logstash use Upstart like service manager, this service manager contains a `bug <https://bugs.launchpad.net/upstart/+bug/812870/>`_ in this case, follow the next steps::

  1) Edith the file /etc/logstash/startup.options and in the line 30 change the LS_GROUP=logstash to LS_GROUP=ossec.
  2) Update the service with the new parameters run the command /usr/share/logstash/bin/system-install
  3) Start Logstash again.
```

 - Install Wazuh App plugin cho Kibana :
```sh
/usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp.zip
```

## 4. Kết nối Wazuh App với API

Chúng ta sẽ register Wazuh API (đã được install trên Wazuh server) tới Wazuh App trên Kibana :

 - Mở Web browser và tới Elastic Stack server IP trên port 5601. Tới Wazuh App
 
![wazuh](/images/wazuh-10.png)

 - Click vào `Add new API`
 
![wazuh](/images/wazuh-11.png)

 - Trước khi thêm các field, tới Wazuh server và sử dụng cmd với quyền root set thông tin bảo mật cho Wazuh API :
```sh
# Replace your desired username for myUserName.
$ cd /var/ossec/api/configuration/auth
$ sudo node htpasswd -c user myUserName

# Do not forget to restart the API to apply the changes:
$ systemctl restart wazuh-api
$ service wazuh-api restart
```

 - Điền username/password với thông tin thích hợp bạn tạo ở bước trước. Nhập `http://MANAGER_IP`cho URL với `MANAGER_IP` là IP của Wazuh server. Nhập `55000` cho port.
 
![wazuh](/images/wazuh-12.png)

Nếu bạn sử dụng Wazuh Documentation cho Nginx, URL phải là `https://localhost`

 - Click và `Save`
 
![wazuh](/images/wazuh-13.png)

## 5. Cài đặt Wazuh agent

### 5.1. Cài đặt Wazuh agent trên Debian/Ubuntu

Thêm Wazuh repo 

 - Cài đặt các gói bổ trợ :
```sh
apt-get install curl apt-transport-https lsb-release -y
```

 - Install Wazuh repo GPG key :
```sh
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
```

 - Tìm tên của OS và thêm vào repo
```sh
CODENAME=$(lsb_release -cs)
echo "deb https://packages.wazuh.com/apt $CODENAME main" \
| tee /etc/apt/sources.list.d/wazuh.list
```

 - Update thông tin package
```sh
apt-get update -y
```

 - Install Wazuh agent
```sh
apt-get install wazuh-agent -y
```

### 5.2. Cài đặt Wazuh-agent trên Centos/RHEL

 - Thêm repo với Centos :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=CentOS-$releasever - Wazuh
baseurl=https://packages.wazuh.com/yum/el/$releasever/$basearch
protect=1
EOF
```

 - Thêm repo với Centos-5 :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/RPM-GPG-KEY-OSSEC-RHEL5
enabled=1
name=CentOS-$releasever - Wazuh
baseurl=https://packages.wazuh.com/yum/el/$releasever/$basearch
protect=1
EOF
```

 - Với RHEL :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=RHEL-$releasever - Wazuh
baseurl=https://packages.wazuh.com/yum/rhel/$releasever/$basearch
protect=1
EOF
```

 - Với RHEL5 :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/RPM-GPG-KEY-OSSEC-RHEL5
enabled=1
name=RHEL-$releasever - Wazuh
baseurl=https://packages.wazuh.com/yum/rhel/$releasever/$basearch
protect=1
EOF
```

 - Với Fedora :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
name=Fedora-$releasever - Wazuh
enabled=1
baseurl=https://packages.wazuh.com/yum/fc/$releasever/$basearch
protect=1
EOF
```

 - Với Amazon Linux :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
name=Amazon Linux - Wazuh
enabled=1
baseurl=https://packages.wazuh.com/yum/el/7/$basearch
protect=1
EOF
```

 - Cài đặt Wazuh agent 
```sh
yum install wazuh-agent -y
```

### 5.3. Với agent là Windows 
 
 - Dowload package từ source : https://documentation.wazuh.com/2.0/installation-guide/packages-list/index.html 
  
 - **Cách 1**: Sử dụng cmd để cài đặt :
```sh
azuh-agent-2.0.exe /S
```

 - **Cách 2** : Sử dụng GUI : Double-click vào file dowload và cài đặt với mặc định. Một khi cài đặt xong, agent sẽ có giao diện đồ họa để cấu hình, mở log file hoặc start/stop service :

![wazuh](/images/wazuh-13.png)
 
Mặc định tất cả file agent được đặt tại : `C:\Program Files(x86)\ossec-agent`



 