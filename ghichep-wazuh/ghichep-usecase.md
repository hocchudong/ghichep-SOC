## Các usecase sử dụng của Wazuh

Wazuh thông thường được dùng để đáp ứng các yêu cầu tuân thủ an ninh (như PCI DSS hoặc HIPAA) và các tiêu chuẩn cấu hình (CIS hardening guide). Nó cũng phổ biến với các IaaS user (vd Amazon, Azure hoặc Google cloud), khi triển khai 1 host-based IDS trên 1 instance đang chạy có thể được tổ hợp với phân tích của các infrastructure event (được pull trực tiếp từ cloud provider API).

Sau đây là 1 số use case phổ biến :

 - signature-based log analysis
 - file integrity monitoring
 - rootkits detection
 - security policy monitoring
 
## 2. Signature-based log analysis

Việc quản lý và phân tích log được tự đông hóa để giúp việc phát hiện các mối đe dọa nhanh hơn. Có rất nhiều case nơi các bằng chứng về việc tấn công có thể được tìm thấy trên các log của thiết bị, hệ thống và ứng dụng. Wazuh có thể dùng để tự động tổng hợp và phân tích dữ liệu log.

Wazuh agent (chạy trên host được giám sát) thông thường phụ trách việc đọc log message của OS và ứng dụng, sau đó chuyển tới Wazuh server, nơi việc phân tích diễn ra. Khi không có agent nào được triển khai, server có thể nhận dữ liệu thông qua syslog, từ các network device hoặc ứng dụng.

Wazuh dùng để giải mã để nhận dạng source app của các log message và sau đó phân tích chúng bằng các rule. Sau đây là 1 ví dụ về rule phát hiện đăng nhập SSH thật bại :
```sh
<rule id="5716" level="5">
  <if_sid>5700</if_sid>
  <match>^Failed|^error: PAM: Authentication</match>
  <description>SSHD authentication failed.</description>
  <group>authentication_failed,pci_dss_10.2.4,pci_dss_10.2.5,</group>
</rule>
```

Rule bao gồm `match` field, được dùng để chỉ định pattern mà rule sẽ tìm kiếm. Nó cũng có `level` field để chỉ rõ mức độ ưu tiên của cảnh báo.

Manager sẽ tạo 1 cảnh báo mỗi lần event được thu thập được bởi agent  (hoặc qua syslog) ánh xạ với rule (với level cao hơn 0)

Một số ví dụ trong `/var/ossec/logs/alerts/alerts.json` : 
``sh
{
  "agent": {
      "id": "1041",
      "ip": "10.0.0.125",
      "name": "vpc-agent-centos-public"
  },
  "decoder": {
      "name": "sshd",
      "parent": "sshd"
  },
  "dstuser": "root",
  "full_log": "Mar  5 18:26:34 vpc-agent-centos-public sshd[9549]: Failed password for root from 58.218.199.116 port 13982 ssh2",
  "location": "/var/log/secure",
  "manager": {
      "name": "vpc-ossec-manager"
  },
  "program_name": "sshd",
  "rule": {
      "description": "Multiple authentication failures.",
      "firedtimes": 349,
      "frequency": 10,
      "groups": [
          "syslog",
          "attacks",
          "authentication_failures"
      ],
      "id": "40111",
      "level": 10,
      "pci_dss": [
          "10.2.4",
          "10.2.5"
      ]
  },
  "srcip": "58.218.199.116",
  "srcport": "13982",
  "timestamp": "2017-03-05T10:26:59-0800"
}
```

Một khi cảnh báo được tạo bởi manager, các alert được gửi tới Elastic Stack để bổ sung thông tin như Goelocation, được lưu trữ và đánh index. Kibana sau đó được dùng để search, phân tích và hiển thị dữ liệu. Một alert được hiển thị như sau :

![wazuh](/images/wazuh-06.png)

Wazuh cung cấp ruleset mặc định, update theo định kỳ, với hơn 1600 rule cho các ứng dụng khác nhau.

## 3. File integrity monitoring

File integrity monitoring (FIM) - Giám sát sự toàn vẹn của file, phát hiện và cảnh báo khi các file của hệ thống và ứng dụng được sửa đổi. Khả năng này thường được dùng để phát hiện việc truy cập hoặc thay đổi các dữ liệu nhạy cảm. Thực tế, nếu server của bạn trong phạm vi với PCI DSS, yêu cầu 11.5 state, bạn sẽ phải cài đặt 1 FIM solution mới có thể pass qua sự kiểm tra.

VD sau là 1 cảnh báo, được tạo khi 1 file được giám sát bị thay đổi. Metadata bao gồm MD5 và SHA1 checksum, file size (trước và sau khi thay đổi), file permisssion, file owner và nội dung thay đổi :
```sh
{
  "agent": {
      "id": "003",
      "ip": "10.0.0.121",
      "name": "vpc-agent-debian"
  },
  "decoder": {
      "name": "syscheck_integrity_changed"
  },
  "full_log": "Integrity checksum changed for: '/root/hola.txt'\nSize changed from '3089' to '3213'\nOld md5sum was: '20db2c4c9bdd937975371bc5ca25af92'\nNew md5sum is : '3841e727a28f733e6d34413afd49d607'\nOld sha1sum was: 'a6c57142a6e6e7e55c58b3174ee52b4f8ec996e3'\nNew sha1sum is : '99aa4b60467a932c89f32603e410a6c194fb1ac3'\n",
  "location": "syscheck",
  "manager": {
      "name": "vpc-ossec-manager"
  },
  "rule": {
      "description": "Integrity checksum changed.",
      "firedtimes": 8,
      "groups": [
          "ossec",
          "syscheck"
      ],
      "id": "550",
      "level": 7,
      "pci_dss": [
          "11.5"
      ]
  },
  "syscheck": {
      "diff": "0a1,2\n> Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua\n> \n",
      "event": "modified",
      "gid_after": "0",
      "gname_after": "root",
      "inode_after": 398585,
      "md5_after": "3841e727a28f733e6d34413afd49d607",
      "md5_before": "20db2c4c9bdd937975371bc5ca25af92",
      "mtime_after": "2017-03-05T13:47:32",
      "mtime_before": "2017-03-05T13:44:15",
      "path": "/root/hola.txt",
      "perm_after": "100666",
      "sha1_after": "99aa4b60467a932c89f32603e410a6c194fb1ac3",
      "sha1_before": "a6c57142a6e6e7e55c58b3174ee52b4f8ec996e3",
      "size_after": "3213",
      "size_before": "3089",
      "uid_after": "1000",
      "uname_after": "admin"
  },
  "timestamp": "2017-03-05T13:44:18-0800"
}
```

Một ví dụ tổng hợp về sự thay đổi file : 

![wazuh](/images/wazuh-07.png)

## 3. Rootkits detection

Wazuh agent định kỳ quét các hệ thống được giám sát để phát hiện rootkits trên cả kernel và user level. Các dạng của malware này thông thường thay thế hoặc thay đổi các thành phần OS đã có sẵn để thay đổi hành vi của hệ thống, nó có thể giấu các process khác, file hoặc kết nối mạng như chính nó.

Wazu sử dụng các phương thức khác nhau để tìm kiếm các thay đổi bất thường của hệ thống hoặc các xâm nhập phổ biến. Việc này được thực hiện định kỳ bởi thành phần `Rootcheck` 

![wazuh](/images/wazuh-08.png)

Dưới đây là ví dụ 1 cảnh báo được tạo khi tìm thấy 1 process ẩn. Ở TH này, hệ thống bị ảnh hưởng chạy trên Linux kernel-level rootkit (name là Diamorphine) :
```sh
{
  "agent": {
      "id": "1030",
      "ip": "10.0.0.59",
      "name": "diamorphine-POC"
  },
  "decoder": {
      "name": "rootcheck"
  },
  "full_log": "Process '562' hidden from /proc. Possible kernel level rootkit.",
  "location": "rootcheck",
  "manager": {
      "name": "vpc-ossec-manager"
  },
  "rule": {
      "description": "Host-based anomaly detection event (rootcheck).",
      "firedtimes": 4,
      "groups": [
          "ossec",
          "rootcheck"
      ],
      "id": "510",
      "level": 7
  },
  "timestamp": "2017-03-05T15:13:04-0800",
  "title": "Process '562' hidden from /proc."
}
```

### 4. Security policy monitoring

SCAP là giải pháp kiểm tra các tuân thủ an ninh tiêu chuẩn cho hạ tầng với enterprise-level. Nó là 1 dòng thông số kỹ thuật với NIST với mục tiêu là duy trì các tiêu chuẩn an nình của hệ thống enterprise.

OpenSCAP là 1 tool kiểm tra dùng Extensible Configuration Checklist Description Format (XCCDF). XCCDF là một phương thức tiêu chuẩn kiểm tra nhanh chóng nội dung checklist và định rõ các security checklist. Nó là tổ hợp với các đặc điểm kỹ thuật khác như CPE, CVE, CCE và OVAL, để tạo SCAP-express checklist được thực hiện bởi SCAP validate product.

Wazuh agent sử dụng OpenSCAP bên trong để kiểm định rằng hệ thống đã tuân thủ theo chuẩn CIS hardening hay chưa. Dưới đây là ví dụ của SCAP rule được dùng để kiểm tra nếu SSH daemon được cấu hình để cho phép password trống :
```sh
<ns10:Rule id="xccdf_org.ssgproject.content_rule_sshd_disable_empty_passwords" selected="false" severity="high">
  <ns10:title xml:lang="en-US">Disable SSH Access via Empty Passwords</ns10:title>
  <ns10:description xml:lang="en-US">To explicitly disallow remote login from accounts with empty passwords, add or correct the following line in <html:code>/etc/ssh/sshd_config</html:code>: <html:pre>PermitEmptyPasswords no</html:pre> Any accounts with empty passwords should be disabled immediately, and PAM configuration should prevent users from being able to assign themselves empty passwords.
  </ns10:description>
  <ns10:reference href="http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf">AC-3</ns10:reference>
  <ns10:reference href="http://iase.disa.mil/stigs/cci/Pages/index.aspx">765</ns10:reference>
  <ns10:reference href="http://iase.disa.mil/stigs/cci/Pages/index.aspx">766</ns10:reference>
  <ns10:rationale xml:lang="en-US">Configuring this setting for the SSH daemon provides additional assurance that remote login via SSH will require a password, even in the event of misconfiguration elsewhere.</ns10:rationale>
  <ns10:fix complexity="low" disruption="low" id="sshd_disable_empty_passwords" reboot="false" strategy="enable" system="urn:xccdf:fix:script:sh">grep -q ^PermitEmptyPasswords /etc/ssh/sshd_config &amp;&amp; \ sed -i "s/PermitEmptyPasswords.*/PermitEmptyPasswords no/g" /etc/ssh/sshd_config; if ! [ $? -eq 0 ]; then; echo "PermitEmptyPasswords no" &gt;&gt; /etc/ssh/sshd_config; fi
  </ns10:fix>
  <ns10:check system="http://oval.mitre.org/XMLSchema/oval-definitions-5">
    <ns10:check-content-ref href="ssg-rhel6-oval.xml" name="oval:ssg-sshd_disable_empty_passwords:def:1" />
  </ns10:check>
  <ns10:check system="http://scap.nist.gov/schema/ocil/2">
    <ns10:check-content-ref href="ssg-rhel6-ocil.xml" name="ocil:ssg-sshd_disable_empty_passwords_ocil:questionnaire:1" />
  </ns10:check>
</ns10:Rule>
```

SCAP check chạy định kỳ (mặc định là một lần mỗi ngày), và kết quả được chỏ tới Wazuh server nơi mà chúng được xử lý thông qua OpenSCAP decoder và rule. Dưới đây là các ví dụ về một cảnh báo, được tạo khi Linux audit policy (auditd) không được cấu hình để giám sát các user action : 
```sh
{
  "agent": {
      "id": "1040",
      "ip": "10.0.0.76",
      "name": "ip-10-0-0-76"
  },
  "decoder": {
      "name": "oscap",
      "parent": "oscap"
  },
  "full_log": "oscap: msg: \"xccdf-result\", scan-id: \"10401488754797\", content: \"ssg-centos-7-ds.xml\", title: \"Ensure auditd Collects System Administrator Actions\", id: \"xccdf_org.ssgproject.content_rule_audit_rules_sysadmin_actions\", result: \"fail\", severity: \"low\", description: \"At a minimum the audit system should collect administrator actions for all users and root. If the auditd daemon is configured to use the augenrules program to read audit rules during daemon startup (the default), add the following line to a file with suffix .rules in the directory /etc/audit/rules.d: -w /etc/sudoers -p wa -k actions If the auditd daemon is configured to use the auditctl utility to read audit rules during daemon startup, add the following line to /etc/audit/audit.rules file: -w /etc/sudoers -p wa -k actions\", rationale: \"The actions taken by system administrators should be audited to keep a record of what was executed on the system, as well as, for accountability purposes.\" references: \"AC-2(7)(b) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AC-17(7) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-1(b) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(a) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(c) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(d) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-12(a) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-12(c) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), IR-5 (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), 126 (http://iase.disa.mil/stigs/cci/Pages/index.aspx), Test attestation on 20121024 by DS (https://github.com/OpenSCAP/scap-security-guide/wiki/Contributors)\", identifiers: \"CCE-RHEL7-CCE-TBD (http://cce.mitre.org)\", oval-id: \"oval:ssg:def:370\", benchmark-id: \"xccdf_org.ssgproject.content_benchmark_RHEL-7\", profile-id: \"xccdf_org.ssgproject.content_profile_common\", profile-title: \"Common Profile for General-Purpose Systems\".",
  "location": "wodle_open-scap",
  "manager": {
      "name": "vpc-ossec-manager"
  },
  "oscap": {
      "check": {
          "description": "At a minimum the audit system should collect administrator actions for all users and root. If the auditd daemon is configured to use the augenrules program to read audit rules during daemon startup (the default), add the following line to a file with suffix .rules in the directory /etc/audit/rules.d: -w /etc/sudoers -p wa -k actions If the auditd daemon is configured to use the auditctl utility to read audit rules during daemon startup, add the following line to /etc/audit/audit.rules file: -w /etc/sudoers -p wa -k actions",
          "id": "xccdf_org.ssgproject.content_rule_audit_rules_sysadmin_actions",
          "identifiers": "CCE-RHEL7-CCE-TBD (http://cce.mitre.org)",
          "oval": {
              "id": "oval:ssg:def:370"
          },
          "rationale": "The actions taken by system administrators should be audited to keep a record of what was executed on the system, as well as, for accountability purposes.",
          "references": "AC-2(7)(b) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AC-17(7) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-1(b) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(a) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(c) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-2(d) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-12(a) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), AU-12(c) (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), IR-5 (http://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r4.pdf), 126 (http://iase.disa.mil/stigs/cci/Pages/index.aspx), Test attestation on 20121024 by DS (https://github.com/OpenSCAP/scap-security-guide/wiki/Contributors)",
          "result": "fail",
          "severity": "low",
          "title": "Ensure auditd Collects System Administrator Actions"
      },
      "scan": {
          "benchmark": {
              "id": "xccdf_org.ssgproject.content_benchmark_RHEL-7"
          },
          "content": "ssg-centos-7-ds.xml",
          "id": "10401488754797",
          "profile": {
              "id": "xccdf_org.ssgproject.content_profile_common",
              "title": "Common Profile for General-Purpose Systems"
          }
      }
  },
  "rule": {
      "description": "OpenSCAP: Ensure auditd Collects System Administrator Actions (not passed)",
      "firedtimes": 3,
      "groups": [
          "oscap",
          "oscap-result"
      ],
      "id": "81529",
      "level": 5,
      "pci_dss": [
          "2.2"
      ]
  },
  "timestamp": "2017-03-05T15:00:03-0800"
}
```

Thêm vào, Wazuh WUI có thể dùng để hiển hiện và phân tích policy dựa vào kết quả quét. Ví dụ sau hiển thị dữ liệu của hệ thống Centos khi quét sử dụng `Server baseline` và `PCI DSS v3` pre-defined profiles : 

![wazuh](/images/wazuh-09.png)

