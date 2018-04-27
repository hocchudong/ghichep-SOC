## Mục lục

 - [1. Định nghĩa ngưỡng cảnh báo](#1)
 - [2. Tích hợp với API bên ngoài](#2)
 - [3. Cấu hình Integrator](#3)
 - [4. Cấu hình syslog output](#4)
 - [5. Tự động tạo báo cáo](#5)
 - [6. Cấu hình mail cảnh báo](#6)

<a name="1"></a>
### 1. Định nghĩa ngưỡng cảnh báo.

Mỗi event trên Wazuh Agent được đặt severity level mặc định là 1. Tất cả các event level này tăng lên sẽ tạo ra cảnh báo trên Wazuh Manager.

 - Cấu hình

Ngưỡng cảnh báo được cấu hình trên `ossec.conf` file sử dụng `<alerts> XML tag. Chi tiết về các option có thể thiết lập xem tại [đây](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/alerts.html#reference-ossec-alerts)

```sh
<ossec_config>
  <alerts>
      <log_alert_level>6</log_alert_level>
  </alerts>
</ossec_config>
```

Cấu hình này sẽ đặt severity level nhỏ nhất để tạo cảnh báo, mà sẽ được lưu trữ tại `alerts.log` và/hoặc trên các file `alerts.json`

Khi bất kỳ giá trị nào bị thay đổi trong `ossec.conf` file, service cần phải restart để có tác dụng :

```sh
systemctl restart wazuh-manager
```
<a name="2"></a>
### 2. Tích hợp với API bên ngoài

`Integrator` là một daemon cho phép Wazuh kết nối tới các API bên ngoài và các tool cảnh báo như Slack và PagerDuty

Tích hợp này cho phép kiểm tra các file nguy hại sử dụng [VirusTotal database](https://documentation.wazuh.com/3.x/user-manual/capabilities/virustotal-scan/index.html)

<a name="3"></a>
### 3. Cấu hình Integrator

`Integrator` không mở theo mặc định. Mở theo câu lệnh sau :
```sh
/var/ossec/bin/ossec-control enable integrator
/var/ossec/bin/ossec-control restart
```

Integraotr được cấu hình trong `etc/ossec.conf`. Thêm các cấu hình sau bên trong section `<ossec_config> </ossec_config>` : 
```sh
<integration>
     <name> </name>
     <hook_url> </hook_url>
     <api_key> </api_key>

  <!-- Optional filters -->

     <rule_id> </rule_id>
     <level> </level>
     <group> </group>
     <event_location> </event_location>
</integration>
```

 - Tích hợp với Slack 
```sh
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/...</hook_url>
</integration>
```

 - Tích hợp với PagerDuty
```sh
<integration>
  <name>pagerduty</name>
  <api_key>MYKEY</api_key>
</integration>
```

 - Tích hợp với VirusTotal 
```sh
<integration>
  <name>virustotal</name>
  <api_key>VirusTotal_API_Key</api_key>
  <group>syscheck,</group>
</integration>
```

<a name="4"></a>
### 4. Cấu hình syslog output

Wazuh có thể cấu hình để gửi các cảnh báo tới syslog như sau (trong file `osssec.conf), các giá trị tùy chọn chi tiết tại [link](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/syslog-output.html#reference-ossec-syslog-output)
```sh
<ossec_config>
  <syslog_output>
    <level>9</level>
    <server>192.168.1.241</server>
  </syslog_output>

  <syslog_output>
    <server>192.168.1.240</server>
  </syslog_output>
</ossec_config>
```

Cấu hình trên sẽ gửi các cảnh báo tới `192.168.1.240` và nếu level cảnh báo >9, cũng sẽ gửi tới `192.168.1.241`.

Sau khi cấu hình trong file `ossec.conf`, client-syslog phải được bật. 
```sh
/var/ossec/bin/ossec-control enable client-syslog
systemctl restart wazuh-manager
```

<a name="5"></a>
### 5. Tự động tạo báo cáo

Bạn có thể cấu hình để tự động tạo báo cáo hàng ngày với option `report` trong `osssec.conf`. Chi tiết xem tại [Report](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/reports.html#reference-ossec-reports). Để cấu hình gửi email thông báo , tham khảo tại [cấu hình email](https://documentation.wazuh.com/3.x/user-manual/manager/manual-email-report/index.html#manual-email-report) và [cấu hình SMTP server](https://documentation.wazuh.com/3.x/user-manual/manager/manual-email-report/smtp_authentication.html#smtp-authentication)
```sh
<ossec_config>
  <reports>
      <category>syscheck</category>
      <title>Daily report: File changes</title>
      <email_to>example@test.com</email_to>
  </reports>
</ossec_config>
```

Cấu hình trên sẽ gửi báo cáo hàng ngày của tất cả các [syscheck alert](https://documentation.wazuh.com/3.x/user-manual/capabilities/file-integrity/index.html#manual-file-integrity) tới email `example@test.com`

Các rule có thể được lọc bằng level, source, username, rule id... :
```sh
<ossec_config>
  <reports>
      <level>10</level>
      <title>Daily report: Alerts with level higher than 10</title>
      <email_to>example@test.com</email_to>
  </reports>
</ossec_config>
```

Cấu hình trên sẽ gửi báo cáo với tất cả các rule với level cảnh báo lớn hơn 10.

Ví dụ về báo cáo được tạo :
```sh
From: Wazuh                      12:01 AM (10 hours ago)
to me
------------------------------------------------

Report 'Daily report: File changes' completed.
------------------------------------------------
->Processed alerts: 368
->Post-filtering alerts: 58
->First alert: 2017 Mar 08 06:31:26
->Last alert: 2017 Mar 08 13:11:42

Top entries for 'Level':
------------------------------------------------
Severity 5                                                                    |47      |
Severity 7                                                                    |11      |

Top entries for 'Group':
------------------------------------------------
ossec                                                                         |58      |
pci_dss_11.5                                                                  |58      |
syscheck                                                                      |58      |

Top entries for 'Location':
------------------------------------------------
localhost->syscheck                                                           |51      |
(ubuntu) 192.168.1.242->syscheck                                              |7       |

Top entries for 'Rule':
------------------------------------------------
554 - File added to the system.                                               |47      |
550 - Integrity checksum changed.                                             |11      |

Top entries for 'Filenames':
------------------------------------------------
/boot/grub/grub.cfg                                                           |1       |
/etc/apt/apt.conf.d/01autoremove-kernels                                      |1       |
/etc/group                                                                    |1       |
/etc/group-                                                                   |1       |
/etc/gshadow                                                                  |1       |
/etc/gshadow-                                                                 |1       |
/etc/passwd                                                                   |1       |
/etc/passwd-                                                                  |1       |
/etc/postfix/main.cf                                                          |1       |
/etc/shadow                                                                   |1       |
/etc/shadow-                                                                  |1       |
```

<a name="6"></a>
### 6. Cấu hình mail cảnh báo

Wazuh có thể gửi cảnh báo tới một hoặc nhiều email khi có rule được đặt hoặc với báo cáo event hàng ngày.

Mail ví dụ : 
```sh
From: Wazuh <you@example.com>               5:03 PM (2 minutes ago)
to: me
-----------------------------
Wazuh Notification.
2017 Mar 08 17:03:05

Received From: localhost->/var/log/secure
Rule: 5503 fired (level 5) -> "PAM: User login failed."
Src IP: 192.168.1.37
Portion of the log(s):

Mar  8 17:03:04 localhost sshd[67231]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.37
uid: 0
euid: 0
tty: ssh

 --END OF NOTIFICATION
```

### 6.1. Các option email thông thường
 
Để cấu hình Wazuh gửi email cảnh báo, các thiết lập email phải được cấu hình trong section `global` ở `osssec.conf` file : 
```sh
<ossec_config>
    <global>
        <email_notification>yes</email_notification>
        <email_to>me@test.com</email_to>
        <smtp_server>mail.test.com..</smtp_server>
        <email_from>wazuh@test.com</email_from>
    </global>
    ...
</ossec_config>
```

Để xem chi tiết các cấu hình có thể có với email, tham khảo [global section](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/global.html#reference-ossec-global) 

Khi các giá trị trên được cấu hình, `email_alert_level` cần được đặt với level cảnh báo nhỏ nhất để tạo email. Mặc định, level được được là **7** : 
```sh
<ossec_config>
  <alerts>
      <email_alert_level>10</email_alert_level>
  </alerts>
  ...
</ossec_config>
```

Ví dụ này sẽ đặt level nhỏ nhất tới `10`. Chi tiết tham khảo tại [alerts section](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/global.html#reference-ossec-global)
 
Sau khi `alert_level` được cấu hình. Wazuh cần restart : 
```sh
systemctl status wazuh-manager
```

**Chú ý**: Wazuh không kiểm soát việc authentiaction cho SMTP. Việc cấu hình SMTP người quản trị có thể tham khảo tại [đây](https://documentation.wazuh.com/3.x/user-manual/manager/manual-email-report/smtp_authentication.html#smtp-authentication)

### 6.2. Với các option của Granular email 
 
Wazuh cũng sử dụng các cấu hình chi tiết cho các cảnh báo email. Chi tiết tham khảo tại section [email_alert](https://documentation.wazuh.com/3.x/user-manual/reference/ossec-conf/global.html#reference-ossec-global)

**Chú ý** : Level nhỏ nhất được cấu hình trong section `alert` sẽ ghi đè các cấu hình này. VD, nếu bạn cấu hình hệ thống gửi khi rule 526 được trigger, nhưng rule có level nhỏ được được chỉ định, cảnh báo sẽ không được gửi.

 - Cảnh báo email dựa trên level và agent 
```sh
<email_alerts>
  <email_to>you@example.com</email_to>
  <event_location>server1</event_location>
  <do_not_delay />
</email_alerts>
```

Cấu hình này sẽ gửi email tới `you@example.com` khi rule trigger trên `server1`.

Trường `event_location` có thể được cấu hình để giám sát một log chỉ định, hostname hoặc IP.

 - Cảnh báo dựa trên rule ID
```sh
<email_alerts>
  <email_to>you@example.com</email_to>
  <rule_id>515, 516</rule_id>
  <do_not_delay />
</email_alerts>
```

Cấu hình này sẽ gửi khi rule 515 hoặc 516 được trigger với agent bất kỳ 

 - Cảnh báo dựa trên group 
```sh
<email_alerts>
  <email_to>you@example.com</email_to>
  <group>pci_dss_10.6.1</group>
</email_alerts>
```

Cấu hình này sẽ gửi cảnh báo khi bất kì rule nào là 1 phần của group `pci_dss_10.6.1` được trigger trên bất kỳ Wazuh device nào được giám sát.

 - Multi option và multi email
 
Ví dụ sau sẽ gửi tới nhiều địa chỉ email với các tiêu chuẩn riêng của nó :
```sh
<ossec_config>
  <email_alerts>
      <email_to>alice@test.com</email_to>
      <event_location>server1|server2</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>is@test.com</email_to>
      <event_location>/log/secure$</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>bob@test.com</email_to>
      <event_location>192.168.</event_location>
  </email_alerts>
  <email_alerts>
      <email_to>david@test.com</email_to>
      <level>12</level>
  </email_alerts>
 </ossec_config>
```

Cấu hình này sẽ gửi  :
	 - email tới `alice@test.com` nếu có cảnh báo trên server1 hoặc server2 được trigger
	 - email tới `is@test.com` nếu có cảnh báo từ `/log/secure`
	 - email tới `bob@test.com` nếu cảnh báo tới từ bất kỳ máy nào trên dải `192.168.0.0/24`
	 - email tới `david@test.com` nếu cảnh báo có level <= 12.
	 
 - Bắt buộc gửi cảnh bảo bới email
 
Có thể buộc gửi 1 email cảnh báo trên rule đã khai báo bên dưới với level cảnh báo nhỏ nhất. Để làm được việc này, bạn cần sử dụng một trong các option bên dưới [đây](https://documentation.wazuh.com/3.x/user-manual/ruleset/ruleset-xml-syntax/rules.html#rules-options)

	 - alert_by_email : luôn luôn cảnh báo bằng email
	 - no_email_alert: không bao giờ cảnh báo bằng email
	 - no_log : không log lại alert này
```sh
<rule id="502" level="3">
  <if_sid>500</if_sid>
  <options>alert_by_email</options>
  <match>Ossec started</match>
  <description>Ossec server started.</description>
</rule>
```

Cấu hình này sẽ gửi email mỗi khi rule 502 được trigger tương ứng với level nhỏ nhất được đặt.  

<a name="7"></a>
### 7. Cấu hình SMTP server 

Có thể sử dụng `Postfix` để thực hiện tính năng xác thực email. Các bước cài đặt và cấu hình như sau :

 - **1.** Cài đặt các gói cần thiết 
	 - Ubuntu
```sh
apt-get install postfix mailutils libsasl2-2 ca-certificates libsasl2-modules -y
```

	 - Centos 
```sh
yum update && yum install postfix mailx cyrus-sasl cyrus-sasl-plain
```

 - **2.** Cấu hình Postfix trong file `/etc/postfix/main.cf`, thêm các dòng sau vào cuối file :

	 - Ubuntu
```sh
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/thawte_Primary_Root_CA.pem
smtp_use_tls = yes
```

	 - Centos
```sh
relayhost = [smtp.gmail.com]:587
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_use_tls = yes
```

 - **3.** Cấu hình email và password
```sh
echo [smtp.gmail.com]:587 USERNAME@gmail.com:PASSWORD > /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
chmod 400 /etc/postfix/sasl_passwd
```

 - **4.** Bảo mật DB password
```sh
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

 - **5.** Restart Postfix
```sh
systemctl reload postfix
```

 - **6.** Test cấu hình 
```sh
echo "Test mail from postfix" | mail -s "Test Postfix" you@example.com
```

Bạn nên nhận được mail test tại `you@example.com`

 - **7.** Cấu hình Wazuh tại `/var/ossec/etc/ossec.conf` : 
```sh
<global>
  <email_notification>yes</email_notification>
  <smtp_server>localhost</smtp_server>
  <email_from>USERNAME@gmail.com</email_from>
  <email_to>you@example.com</email_to>
</global>
``` 

 - **8.** Restart wazuh manager
```sh
systemctl restart wazuh-manager
```

