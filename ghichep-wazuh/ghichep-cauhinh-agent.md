## 1. Cài đặt Wazuh agent

### 1.1. Cài đặt Wazuh agent trên Debian/Ubuntu

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
echo "deb https://packages.wazuh.com/3.x/apt/ stable main" | tee /etc/apt/sources.list.d/wazuh.list
```

 - Update thông tin package
```sh
apt-get update -y
```

 - Install Wazuh agent
```sh
apt-get install wazuh-agent -y
```

### 1.2. Cài đặt Wazuh-agent trên Centos/RHEL

 - Thêm repo với Centos :
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

 - Thêm repo với Centos-5 :
```sh
cat > /etc/yum.repos.d/wazuh.repo <<\EOF
[wazuh_repo]
gpgcheck=1
gpgkey=http://packages.wazuh.com/key/GPG-KEY-WAZUH-5
enabled=1
name=Wazuh repository
baseurl=http://packages.wazuh.com/3.x/yum/5/
protect=1
EOF
```

 - Cài đặt Wazuh agent 
```sh
yum install wazuh-agent -y
```

### 1.3. Với agent là Windows 
 
 - Dowload package từ source : https://documentation.wazuh.com/current/installation-guide/packages-list/index.html
  
 - **Cách 1**: Sử dụng cmd để cài đặt :
```sh
wazuh-agent-3.6.1-1.msi /q
```

 - **Cách 2** : Sử dụng GUI : Double-click vào file dowload và cài đặt với mặc định. Một khi cài đặt xong, agent sẽ có giao diện đồ họa để cấu hình, mở log file hoặc start/stop service :

![wazuh](/images/wazuh-13.png)
 
Mặc định tất cả file agent được đặt tại : `C:\Program Files(x86)\ossec-agent`

## 2. Tiến trình registration tại agent

Mỗi khi Wazuh  Agent gửi dữ liệu tới Wazuh Manager thông qua một phương thức an toàn là gọi OSSEC message protocol, phương thức này sẽ mã hóa message bằng pre-shared key. Lúc đầu khi bạn cài đặt thành công Wazuh Agent sẽ không thể kết nối với Wazuh Manager vì thiếu pre-shared key.

Quá trình **registration** bao gồm việc tạo 1 mối quan hệ tin cậy giữa Manager và Agent. Điều này có thể thực hiện bởi chính Manager hoặc với registration service, service này sẽ lắng nghe trên Manager. Một Agent có thể yêu cầu pre-shared key bằng việc sử dụng 1 số thông tin xác thực và nó sẽ phản hồi pre-shared key và lưu trữ Agent mới trên local database.

Một cách tiếp cận khác là sử dụng RESTful API.

 - Agent keys

Manager sử dụng file `/var/ossec/etc/client.keys` để lưu trữ registration record cho mỗi agent, bao gồm `ID`, `name`, `IP` và `key`. VD :
```sh
001 Server1 any e20e0394dca71bacdea57d4ca25d203f836eca12eeca1ec150c2e5f4309a653a
002 ServerProd 192.246.247.247 b0c5548beda537daddb4da698424d0856c3d4e760eaced803d58c07ad1a95f4c
003 DBServer 192.168.0.1/24 8ec4843da9e61647d1ec3facab542acc26bd0e08ffc010086bb3a6fc22f6f65b
```

Agent cũng có file `/var/ossec/etc/client.keys` chứa các registration record của riêng nó. VD với `Serrver1` agent :
```sh
001 Server1 any e20e0394dca71bacdea57d4ca25d203f836eca12eeca1ec150c2e5f4309a653a
```

 - Basic data cho việc registing 1 agent
 
Để register 1 agent, nó cần cung cấp tên và IP của agent :

Có 1 số cách để đặt agent IP :

 - **Any IP** : Cho phép agent kết nối từ bất từ IP nào. VD : `Server1` có `any` IP
 - **Fixed IP** : Cho phép kết nối từ 1 IP chỉ định. VD : `ServerProd` có IP `192.246.247.247`
 - **Range IP** : Cho phép kết nối từ 1 range IP chỉ định. VD : `DBServer`
 
Một số phương thức registration tự động phát hiện IP của agent trong quá trình registration

## 3. Register cho các Agent

Trong phần này sẽ hướng dẫn cách register để giám sát các agent :

**Trên Manager server** : 

 - Chạy lệnh `manage_agents` :
```sh
# /var/ossec/bin/manage_agents

****************************************
* Wazuh v3.2.1 Agent manager.            *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist already added agents (L).
   (R)emove an agent (R).
   (Q)uit.
Choose your action: A,E,L,R or Q:
```

 - Chọn `A` để thêm 1 agent. Điền `hostname` và `IP` cho agent :
```sh
Choose your action: A,E,L,R or Q: A

- Adding a new agent (use '\q' to return to the main menu).
  Please provide the following:
   * A name for the new agent: Example
   * The IP Address of the new agent: 192.168.100.10
   * An ID for the new agent[001]:
Agent information:
   ID:001
   Name:Example
   IP Address:any

Confirm adding it?(y/n): y
Agent added with ID 001.
```

 - Chọn `E` để tạo ra 1 key mới :
```sh
Choose your action: A,E,L,R or Q: E

Available agents:
   ID: 001, Name: Example, IP: any
Provide the ID of the agent to extract the key (or '\q' to quit): 001

Agent key information for '001' is:
MDAxIDE4NWVlNjE1Y2YzYiBhbnkgMGNmMDFiYTM3NmMxY2JjNjU0NDAwYmFhZDY1ZWU1YjcyMGI2NDY3ODhkNGQzMjM5ZTdlNGVmNzQzMGFjMDA4Nw==
```

 - Exit với `Q`
 
**Trên Agent** : 

 - Chạy `manage_agents` : 
```sh
# /var/ossec/bin/manage_agents

****************************************
* Wazuh v3.2.1 Agent manager.            *
* The following options are available: *
****************************************
   (I)mport key from the server (I).
   (Q)uit.
Choose your action: I or Q:
```

 - Chọn `I` để import key, paste key đã sinh từ máy manager : 
```sh
Choose your action: I or Q: I

* Provide the Key generated by the server.
* The best approach is to cut and paste it.
*** OBS: Do not include spaces or new lines.

Paste it here (or '\q' to quit): MDAxIDE4NWVlNjE1Y2YzYiBhbnkgMGNmMDFiYTM3NmMxY2JjNjU0NDAwYmFhZDY1ZWU1YjcyMGI2NDY3ODhkNGQzMjM5ZTdlNGVmNzQzMGFjMDA4Nw=

Agent information:
   ID:013
   Name:Example
   IP Address:any

Confirm adding it?(y/n): y
Added.
```

 - Ấn `Q` để thoát
 
 - Chỉnh sửa IP của server Manager tại file `/var/ossec/etc/ossec.conf` : 
```sh
<client>
      <server-ip>MANAGE_IP</server-ip>
</client>
```

 - Restart lại agent :
```sh
# /var/ossec/bin/ossec-control restart
```

**Chú ý** : Register đè một agent 

VD : Một agent tên là `Server` với IP 10.0.0.10 đã được install và có ID là 005. Giả sử agent được reinstall, chúng ta cần phải reinstall agent mới và kết nối lại với Manager. Sử dụng câu lệnh sau trên Manager : 
```sh
/var/ossec/bin/manage_agents -n Server1 -a 10.10.10.10 -F 0
```

## 4. List các agent được cài đặt và remove agent

 - List các agent đã register trên Manager :
```sh
# /var/ossec/bin/agent_control -l
Wazuh agent_control. List of available agents:
   ID: 000, Name: vpc-ossec-manager (server), IP: 127.0.0.1, Active/Local
   ID: 1040, Name: ip-10-0-0-76, IP: 10.0.0.76, Active
   ID: 003, Name: vpc-agent-debian, IP: 10.0.0.121, Active
   ID: 005, Name: vpc-agent-ubuntu-public, IP: 10.0.0.126, Active
   ID: 006, Name: vpc-agent-windows, IP: 10.0.0.124, Active
   ID: 1024, Name: ip-10-0-0-252, IP: 10.0.0.252, Never connected
   ID: 1028, Name: vpc-debian-it, IP: any, Never connected
   ID: 1030, Name: diamorphine-POC, IP: 10.0.0.59, Active
   ID: 015, Name: vpc-agent-centos, IP: 10.0.0.123, Active
   ID: 1031, Name: WIN-UENN0U6R5SF, IP: 10.0.0.124, Never connected
   ID: 1032, Name: vpc-agent-ubuntu, IP: 10.0.0.122, Active
   ID: 1033, Name: vpc-agent-debian8, IP: 10.0.0.128, Active
   ID: 1034, Name: vpc-agent-redhat, IP: 10.0.0.127, Active
   ID: 1035, Name: vpc-agent-centos7, IP: 10.0.0.101, Never connected
   ID: 1041, Name: vpc-agent-centos-public, IP: 10.0.0.125, Active
```

 - Remove agent với ID là 001 :
```sh
# /var/ossec/bin/manage_agents -r 001


****************************************
* Wazuh v3.2.1 Agent manager.          *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist already added agents (L).
   (R)emove an agent (R).
   (Q)uit.
Choose your action: A,E,L,R or Q:
Available agents:
   ID: 001, Name: new, IP: any
Provide the ID of the agent to be removed (or '\q' to quit): 001
Confirm deleting it?(y/n): y
Agent '001' removed.

** You must restart OSSEC for your changes to take effect.

manage_agents: Exiting.
```

 - Login vào Wazuh Interface với IP : http://MANAGER_IP:5601 và kiểm tra các agent đã register :
 ![wazuh](/images/wazuh-17.png)
 
 ![wazuh](/images/wazuh-18.png)
 

