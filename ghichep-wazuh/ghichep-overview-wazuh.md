## 1. Giới thiệu về Wazuh

Wazuh là 1 project mã nguồn dùng cho việc bảo vệ an ninh. Được xây dựng từ các thành phần : OSSEC HIDS, OpenSCAP và Elastic Stack.

![wazuh](/images/wazuh-00.png)

 - OSSEC HIDS : host-based Intrusion Detection System (HIDS) được dùng cho việc phát hiện xâm nhập, hiển thị và giám stas. Nó dựa vào 1 multi-platform agent cho việc đẩy dữ liệu hệ thống (log message, file hash và phát hiện bất thường) tới 1 máy quản lý trung tâm, nơi sẽ phân tích và xử lý, dựa trên các cảnh báo an ninh. Các agent truyền event data event data tới máy quản lý trung tâm thông qua kênh được bảo mật và xác thực.
 OSSEC HIDS cung cấp syslog server trung tâm và hệ thống giám sát không cần agent, cung cấp việc giám sát tới các event và thay đổi trên các thiết bị không cài được agent như firewall, switch, router, access point, thiết bị mạng....
 
 - OpenSCAP 
 OpenSCAP là 1 OVAL (Open Vulnerability Assesment Language) và XCCDF (Extensible Configuration Checklist Description Format) được dùng để kiểm tra cấu hình hệ thống và phát hiện các ứng dụng dễ bị tấn công.
 Nó được biết đến như là một công cụ được thiết kế để kiểm tra việc tuân thủ an ninh của hệ thống sử dụng các tiêu chuẩn an ninh dùng cho môi trường doanh nghiệp
 
 - ELK Stack 
 Sử dụng cho việc thu thập, phân tihcs, index, store, search và hiển thị dữ liệu log. 
 
## 2. Các thành phần

### 2.1. Wazuh agent

Chạy trên : Windows, Linux, Solaris, BSD hoặc MAC OS. Dùng thu thập các dạng khác nhau của dữ liệu hệ thống và ứng dụng. Dữ liệu được chuyển tới Wazuh server thông qua 1 kênh được mã hóa và xác thực. Để thiết lập kênh này, 1 quá trình đăng ký bao gồm pre-shared key duy nhất được thiết lập.

Các agent có thể dùng để giám sát server vật lý, máy ảo, cloud instance (AWS, Azure hoặc Google cloud). Các các cài đặt pre-compile agent có sẵn cho các OS : Linux, AIX, Solaris, Windows và Darwin (Mac OS X).

Trên các OS Unix-based, agent chạy trên multiple process, các process này liên lạc với nhau thông qua local Unix domain socket. 1 trong các process này phụ trách việc liên lạc và gửi dữ liệu tới Wazuh server. Trên Windows, chỉ có 1 agent process chạy trên multiple task sử dụng mutexes.

Các agent task hoặc process khác nhau được dùng để giám sát hệ thống theo các cách khác nhau (giám sát sự thay đổi về file, đọc log, quét các thay đổi hệ thống).

Sơ đồ sau thể hiện các internal task và process diễn ra trên các agent level.

![wazuh](/images/wazuh-01.png)

Tất cả các process agent có mục tiêu và thiết lập khác nhau. 

 - **Rootcheck** : Thực hiện các task liên quan đến phát hiện về `Rootkits`, `malware` và các bất thường của hệ thống. Nó chạy 1 số công cụ kiểm tra an ninh cơ bản dựa vào các file cấu hình hệ thống.
 - **Log Collector**: Dùng để đọc và thu thập các log message, bao gồm các các file flat log như Windows event log và thậm chí là Windows Event Channel. Nó cũng được cấu hình để chạy định kỳ và bắt 1 số output của các câu lệnh cụ thể.
 - **Syscheck** : Process này thực hiện file integrity monitoring (FIM) (Giám sát tính toàn vẹn của file). Nó cũng có thể giám sát registry key trên Windows. Nó sẽ bắt các thay đổi về nội dung file, quyền và các thuộc tính khác, cũng như phát hiện việc tạo và xóa file.
 - **OpenSCAP** : Được dùng để publish OVAL và XCCDF dựa vào các hồ sơ bảo mật cơ bản, định kỳ quét hệ thống, nó sẽ phát hiện được các ứng dụng và cấu hình sẽ bị tấn công, không tuân theo các chuẩn được xác định theo CIS (Center of Internet Security)
 - **Agent Daemon** : Process nhận dữ liệu được tạo hoặc được thu thập bởi tất cả các thành phần agent khác. Nó nén, mã hóa và phân phối dữ liệu tới server thông qua kênh được xác thực. Process này chạy trên "chroot" enviroment được cô lập, có nghĩa rằng nó sẽ hạn chế truy cập tới các hệ thống được giám sát. Điều này cải thiện được an toàn cho agent vì process đó là process duy nhất kết nối tới mạng.
 
Chú giải : 

 - Rootkits : Phần mềm hoặc công cụ phần mềm che giấu sự tồn tại của 1 phần mềm khác, thường là virus xâm nhập vào hệ thống.
 - Malware : Mọi loại mã gây hại trên máy tính người dùng : spyware, trojan, virus...
 
### 2.2. Wazuh server

Thành phần server phụ trách việc phân tích dữ liệu nhận từ agent, tạo các ngưỡng cảnh báo khi 1 event ánh xạ với rule (phát hiện xâm nhập, thay đổi file, cấu hình không tương thích với policy, rootfit...)

![wazuh](/images/wazuh-02.png)

Server thông thường chạy các thành phần agent với mục tiêu giám sát chính nó. Một số thành phần server chính là : 

 - **Registration service** : Được dùng để register agent mới được việc cung cấp và phân phối các key xác thực pre-shared, các key này là độc nhất với mỗi agent. Process này chạy như 1 network service và hỗ trợ việc xác thực qua TLS/SSL với 1 fixed password.
 - **Remote daemon service** : Service này nhận dữ liệu từ agent. Nó sử dụng pre-shared key để xác thực định danh của mỗi agent và mã hóa giao tiếp với chúng.
 - **Analysis daemon** : Process này thực hiện việc phân tích dữ liệu. Nó sử dụng các bộ giải mã để nhận dạng thông tin được xử lý (các Windows event, SSHD logs...) và sau đó giải nén các yếu tố dữ liệu thích hợp từ log message (source ip, event id, user...) Sau đó, bằng cách sử dụng các rule được định nghĩa bằng cách pattern đặc biệt trên bộ giải mã, nó sẽ tạo các ngưỡng cảnh báo thậm chí ra lệnh để thực hiện các biện pháp đối phó như chặn IP trên firewall.
 - **RESTful API** : Cung cấp interface để quản lý và giám sát cấ hình và trạng thái triển khai của các agent. Nó cũng được dùng bởi Wazuh web interface (Kibana)
 
Wazuh tích hợp với Elastic stack để cung cấp các log message đã được giải mã và đánh index bởi Elasticsearch, cũng như là 1 web console real-time cho việc cảnh báo và phân tích log. Wazuh web interface (chạy trên Kibana) có thể dùng để quản lý và giám sát hạ tầng Wazuh

Một Elasticsearch index là một tập hợp các document có một chút các đặc trưng tương tự nhau (như các trường chung hoặc các yêu cầu về data retention được chia sẻ). Wazuh sử dụng 3 index khác nhau, được tạo hàng ngày và lưu trữ các dạng event khác nhau : 

 - **Wazuh-alert** : Index cho các cảnh báo được sinh ra bởi Wazun server mỗi khi một event ứng với rule tạo ra.
 - **Wazuh-events** : Index cho tất cả các event (archive data) được nhận từ các agent, bất kể có ứng với rule hay không.
 - **Wazuh-monitoring**: Index cho dữ liệu liên quan đến trạng thái agent. Nó được dùng bởi web interface cho việc hiển thị agent đã hoặc đang "Active", "Disconnect" hoặc "Never connected"

Với các index trên, document là các cảnh báo, archived event hoặc status event riêng lẻ.

Một Elasticsearcg index được chia tới 1 hoặc nhiều shard, và mỗi shard có thể có 1 hoặc nhiều replica. Mỗi primary và replica shard là 1 Lucene index đơn lẻ. Vì vậy 1 Elasticsearch index được tạo bởi nhiều Lucene index. Khi 1 tìm kiếm chạy trên 1 Elasticsearch index, search đó được xử lý trên các shard song song, và kết quả được merge lại. Việc chia nhỏ các Elasticsearch tới nhiều shard và replica cũng được dùng với Elasticsearch cluster với mục tiêu là mở rộng việc tìm kiếm và HA. Một Elasticsearch cluster single-node thường chỉ có 1 shard mỗi index và không có replica.


