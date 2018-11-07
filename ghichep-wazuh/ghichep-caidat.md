## Cài đặt Wazuh

**Cấu hình yêu cầu** :

Với nhu cầu giám sát an ninh và thu thập log tập trung cho hệ thống từ 15-20 server. 

Cấu hình yêu cầu như sau :

 ![wazuh](/images/wazuh-23.png)
	
Mô hình triển khai : 

 ![wazuh](/images/wazuh-24.png)

### 1. Cài đặt Wazuh server

 -  Install java 8
```sh
yum install java-1.8.0-openjdk -y
```

 - Thêm repo :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=Wazuh repository
baseurl=https://packages.wazuh.com/3.x/yum/
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

 - Mở port 1515 cho phép Wazuh agent kết nối : 
```sh
firewall-cmd --add-port=1515/tcp
firewall-cmd --add-port=1515/tcp --permanent
firewall-cmd --add-port=55000/tcp
firewall-cmd --add-port=55000/tcp --permanent
firewall-cmd --add-port=1514/udp
firewall-cmd --add-port=1514/udp --permanent
firewall-cmd --add-port=514/tcp
firewall-cmd --add-port=514/tcp --permanent
firewall-cmd --add-port=514/udp
firewall-cmd --add-port=514/udp --permanent
```

### 3. Cài đặt Filebeat

Filebeat giúp Wazuh server vận chuyển các cảnh báo và archived event một cách an toàn tới Logstash service trên Elastic Stack server.

**Chú ý** : Với mô hình all-in-one, có thể bỏ qua việc cài Filebeat vì Logstash có thể đọc event/alert trực tiếp từ local filesystem.

 - Cài đặt GPG keys từ Elastic và thêm Elastic repo :
```sh
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

cat > /etc/yum.repos.d/elastic.repo << EOF
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

 - Cài đặt Filebeat :
```sh
yum install filebeat-6.4.2 -y
```

 - Download Filebeat config file từ Wazuh repo để cấu hình chỏ Wazuh alert tới Logstash :
 
```sh
curl -so /etc/filebeat/filebeat.yml https://raw.githubusercontent.com/wazuh/wazuh/3.2/extensions/filebeat/filebeat.yml
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

### 4. Cài đặt Elasticsearch

 - Chú ý kiểm tra việc cài đặt java 8 ở bước đầu.

 - Thêm repo
```sh
rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch
```

 - Thêm file repo
```sh
cat <<EOF > /etc/yum.repos.d/elasticsearch.repo
[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

 - Cài đặt elastic
```sh
yum install elasticsearch-6.4.2 -y
```

 - Thêm rule firewall
```sh
firewall-cmd --add-port=9200/tcp
firewall-cmd --add-port=9200/tcp --permanent
```

 - Start và enable service
```sh
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
```

 - Kiểm tra dịch vụ Elasticsearch
```sh
curl -X GET http://localhost:9200
```

 - Load Wazuh Elasticsearch template : 
```sh
curl https://raw.githubusercontent.com/wazuh/wazuh/3.2/extensions/elasticsearch/wazuh-elastic6-template-alerts.json | curl -XPUT 'http://localhost:9200/_template/wazuh' -H 'Content-Type: application/json' -d @-
```


### 5. Cài đặt Logstash

 - Thêm file repo :
```sh
cat << EOF > /etc/yum.repos.d/logstash.repo
[logstash-6.x]
name=Elastic repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

 - Cài đặt Logstash
```sh
yum install logstash-6.4.2 -y
```

 - Dowload Wazuh config và template file cho Logstash :
 
**Mô hình All-in-one**

```sh
curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/3.2/extensions/logstash/01-wazuh-local.conf
```

 - Thêm usermode
```sh
usermod -a -G ossec logstash
```

**Mô hình Distribute**

```sh
curl -so /etc/logstash/conf.d/01-wazuh.conf https://raw.githubusercontent.com/wazuh/wazuh/3.2/extensions/logstash/01-wazuh-remote.conf
```

Follow các bước sau nếu bạn sử dụng Centos6/RHEL6 hoặc Amazon AMI (logstash sử dụng Upstart như service manager và cần sửa, xem bug [sau](https://bugs.launchpad.net/upstart/+bug/812870/) )

	 - Sửa file `/etc/logstash/startup.options` ở dòng 30 từ `LS_GROUP=logstash` thành `LS_GROUP=ossec`.
	 - Update service với tham số mới bằng câu lệnh `/usr/share/logstash/bin/system-install`
	 - Restart service

 - Start và enable service
```sh
systemctl daemon-reload
systemctl start logstash
systemctl enable logstash
```

 - Cấu hình firewall cho phép Logstash nhận log từ client (port 5044)
```sh
firewall-cmd --add-port=5044/tcp
firewall-cmd --add-port=5044/tcp --permanent
```

### 6. Cài đặt Kibana

 - Tạo repo cài đặt Kibana :
```sh
cat <<EOF > /etc/yum.repos.d/kibana.repo
[kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

 - Cài đặt Kibana :
```sh
yum install kibana-6.4.2 -y
sed -i 's/#server.host: "localhost"/server.host: "0.0.0.0"/'g /etc/kibana/kibana.yml
```


 - Install Wazuh App plugin cho Kibana :

Tăng giá trị mặc định `heap memory limit` của Node.js để bảo vệ memory khỏi lỗi khi install Wazuh APP. Set giá trị mặc định như sau :
```sh
export NODE_OPTIONS="--max-old-space-size=3072"
```

```sh
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/wazuhapp/wazuhapp-3.6.1_6.4.2.zip
```

 - Start và enable Kibana
```sh
systemctl daemon-reload
systemctl start kibana
systemctl enable kibana
```

 - Cho phép truy cập Kibana web interface (port 5601)
```sh
firewall-cmd --add-port=5601/tcp
firewall-cmd --add-port=5601/tcp --permanent
```

- Disable repo Elasticsearch để tránh việc update : 
```sh
 sed -i "s/^enabled=1/enabled=0/" /etc/yum.repos.d/elasticsearch.repo
 ```

 - Chạy Kibana : http://192.168.0.29:5601
 

## 4. Kết nối Wazuh App với API

Chúng ta sẽ register Wazuh API (đã được install trên Wazuh server) tới Wazuh App trên Kibana :

 - Mở Web browser và tới Elastic Stack server IP trên port 5601. Tới Wazuh App
 
![wazuh](/images/wazuh-10.png)

 - Click vào `Add new API`
 
![wazuh](/images/wazuh-11.png)

 - Trước khi thêm các field, tới Wazuh server và sử dụng cmd với quyền root set thông tin bảo mật cho Wazuh API :
```sh
cd /var/ossec/api/configuration/auth
sudo node htpasswd -c user myUserName
```

 - Restart service
```sh
systemctl restart wazuh-api
```

Các thông tin cần nhập như sau : 
 - user/password : Thông tin tạo bước trước.
 - URL : http://MANAGER_IP . Với `MANAGER_IP` là IP của Wazuh server
 - Port : 55000
 
![wazuh](/images/wazuh-12.png)

Nếu bạn sử dụng Wazuh Documentation cho Nginx, URL phải là `https://localhost`

 - Click và `Save`
 
![wazuh](/images/wazuh-13.png)

Sau khi cài đặt xong Wazuh server, cấu hình agent và kết nối, tham khảo link [sau](https://github.com/hocchudong/ghichep-SOC/blob/master/ghichep-wazuh/ghichep-cauhinh-agent.md)
 
