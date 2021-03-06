Ghi chép về các kỹ thuật/giải pháp HA cho mysql/mariadb
Mục lục:
1. Giới thiệu về HA cho DB
2. Các giải giải pháp
2.1 Giải pháp có sẵn
2.1.1 Master - Slave
2.1.2 Master - Master
2.2 Giải pháp bên thứ 3 (3rd party)
2.2.1 Galera
2.2.2 DRBD
2.2.3 Radundant Hardware
2.2.4 Shared Storage
2.2.5 MySQL clustering
2.2.6 Percona cluster
3. Kết luận

1. Giới thiệu về HA
Ngày nay, công nghệ thông tin đã ăn sâu vào nhiều lĩnh vực trong đời sống phục vụ cho sản xuất, giải trí và đặc biệt nhu cầu thông tin. Các hệ thống này luôn được đầu tư với quy mô càng ngày càng mở rộng, là hướng phát triển trọng tâm của doanh nghiệp cung cấp nội dung. Để đảm bảo các dịch vụ chạy thông suốt, phục vụ tối đa đến nhu cầu của người sử dụng và nâng cao tính bảo mật, an toàn dữ liệu; giải pháp High Availability được nghiên cứu và phát triển bởi nhiều hãng công nghệ lớn. Với Database, tính an toàn và khả dụng được đặt lên hàng đầu. Vì vậy, ở bài viết này, chúng tôi xin phép điểm qua một vài Giải pháp HA cho hệ cơ sở dữ liệu sử dụng MySQL hoặc MariaDB đang được cộng đồng tin dùng.

HA giải quyết được gì?
Tăng tính hoạt động sẵn sàng dữ liệu mọi lúc mọi nơi
Nâng cao hiệu suất làm việc của hệ thống
Nâng cao được tính an toàn dữ liệu
Đảm bảo hệ thống làm việc không bị gián đoạn
2. Các giải pháp
Có 2 giải pháp chính cho việc HA:

Giải pháp Native: Giải pháp này được mysql/mariadb hỗ trợ.

Master - Slave
Master - Master
Giải pháp 3rd party: Cùng với mục đích là để nhất quán dữ liệu với các server với nhau nhưng cơ chế hoạt động và mô hình khác với giải pháp Native. Một số kỹ thuật mà chúng tôi đã tìm hiểu là:

Galera
DRBD

2.1 Giải pháp Native
Cơ chế làm việc như sau: Trên mỗi server sẽ có một user làm nhiệm vụ replication dữ liệu mục đích của việc này là giúp các server đảm bảo tính nhất quán về dữ liệu với nhau.

Replication là tính năng cho phép dữ liệu của (các) máy chủ Master được sao chép/nhân bản trên một hoặc nhiều máy chủ khác (Slave). Mục đích của việc này là để sao lưu dữ liệu ra các máy chủ khác đề phòng máy chủ chính gặp sự cố.

Có thể sao chép/nhân bản được những gì?

Tùy vào mục đích sử dụng, tính năng này cho phép chúng ta sao chép/nhân bản từ Tất cả các DB trên Master, một hoặc nhiều DB, cho đến các bảng trong mỗi DB sang Slave một cách tự động.

Cơ chế hoạt động

Máy chủ Master sẽ gửi các binary-log đến máy chủ Slave. Máy chủ Slave sẽ đọc các binary-log từ Master để yêu cầu truy cập dữ liệu vào quá trình replication. Một relay-log được tạo ra trên slave, nó sử dụng định dạng giống với binary-log. Các relay-log sẽ được sử dụng để replication và được xóa bỏ khi hoàn tất quá trình replication.

Các master và slave không nhất thiết phải luôn kết nối với nhau. Nó có thể được đưa về trạng thái offline và khi được kết nối lại, quá trình replication sẽ được tiếp tục ở thời điểm nó offline.

Binary-log là gì?

Binary-log chứa những bản ghi ghi lại những thay đổi của các database. Nó chứa dữ liệu và cấu trúc của DB (có bao nhiêu bảng, bảng có bao nhiêu trường,...), các câu lệnh được thực hiện trong bao lâu,... Nó bao gồm các file nhị phân và các index.

Binary-log được lưu trữ ở dạng nhị phân không phải là dạng văn bản plain-text.


2.1.1 Master - Slave
Master - Slave: là một kiểu trong giải pháp HA cho DB, mục đích để đồng bộ dữ liệu của DB chính (Master) sang một máy chủ DB khác gọi là Slave một cách tự động.



Tham khảo cách cấu hình.


2.1.2 Master - Master
Master - Master: Khi cấu hình kiểu này, 2 DB sẽ tự động đồng bộ dữ liệu cho nhau.



2.2 Giải pháp 3rd party

2.2.1 Galera
Galera Cluster là giải pháp tăng tính sẵn sàng cho cách Database bằng các phân phối các thay đổi (đọc - ghi dữ liệu) tới các máy chủ trong Cluster. Trong trường hợp một máy chủ bị lỗi thì các máy chủ khác vẫn sẵn sàng hoạt động phục vụ các yêu cầu từ phía người dùng.



Cluster có 2 mode hoạt động là Active - Passive và Active - Active:

Active - Passive: Tất cả các thao tác ghi sẽ được thực hiện ở máy chủ Active, sau đó sẽ được sao chép sang các máy chủ Passive. Các máy chủ Passive này sẽ sẵn sàng đảm nhiệm vai trò của máy chủ Active khi xảy ra sự cố. Trong một vài trường hợp, Active - Passive cho phép SELECT ở các máy chủ Passive.
Active - Active: Thao tác đọc - ghi dữ liệu sẽ diễn ra ở mỗi node. Khi có thay đổi, dữ liệu sẽ được đồng bộ tới tất cả các node
Hướng dẫn cài đặt trên:

Ubuntu
CentOS

2.2.2 DRBD (Distributed Replicated Block Device)
Khái niệm/Định nghĩa
Phục vụ cho việc sao chép dữ liệu từ một thiết bị này sang thiết bị khác, đảm bảo dữ liệu luôn được đồng nhất giữa 2 thiết bị
Việc sao chép là liên tục do ánh xạ với nhau ở mức thời gian thực
Được ví như RAID 1 qua mạng
Bonus: Bài viết chi tiết
Nhiệm vụ của DRDB trong HA mysql/mariadb
Khi được cài đặt trên các cụm cluter, DRBD đảm nhiệm việc đồng bộ dữ liệu của các server trong cụm cluters với nhau
Kết hợp với heartbeat
Là một tiện ích chạy ngầm trên máy master để kiểm tra trạng thái hoạt động. Khi máy master xảy ra sự cố, heartbeat sẽ khởi động các dịch vụ ở máy phụ để phục vụ thay máy master.



2.2.3 Radundant Hardware - Sử dụng tài nguyên phần cứng
Thuật ngữ 'Two of Everything", nghĩa là sử dụng 2 tài nguyên phần cứng cho một máy chủ. Có nghĩa rằng một máy chủ sẽ có 2 nguồn cấp điện, 2 ổ cứng, 2 card mạng,...


2.2.4 Shared Storage
Để khắc phục lại những sự cố mà server có thể gặp phải, một máy chủ backup được cấu hình nhằm mục đích sao lưu và duy trì các hoạt động khi server chính bị lỗi. Sử dụng NAS hoặc SAN bên trong các server để đồng bộ dữ liệu giữa các máy chủ với nhau.




2.2.5 MySQL clustering
Với các Database lớn, clustering làm nhiệm vụ chia nhỏ dữ liệu và phân phối vào các server nằm ở bên trong cụm máy chủ cluster. Trong trường hợp một máy chủ bị lỗi, dữ liệu vẫn được lấy từ các node khác đảm bảo hoạt động của người dùng không bị gián đoạn.


2.2.6 Percona cluster
Giống với Galera, Percona có ít nhất 3 node luôn đồng bộ dữ liệu với nhau. Dữ liệu có thể được đọc/ghi lên bất kỳ node nào trong mô hình. Một máy chủ đứng ở bên trên tiếp nhận các truy vấn và phân phối lại một cách đồng đều cho các server bên dưới.




3. Kết luận
Nâng cao khả năng hoạt động cho cơ sở dữ liệu là điều vô cùng quan trọng, nó giúp các ứng dụng sử dụng DB của bạn hoạt động nhịp nhàng, trơn tru hơn. Trên đây là một vài giải pháp nâng cao hiệu năng hoạt động của DB. Dựa vào điều kiện thực tế mà có thể lựa chọn giải pháp phù hợp với mô hình của mình.
