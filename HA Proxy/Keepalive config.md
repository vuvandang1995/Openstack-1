# File cấu hình keepalive

### 1. Cấu trúc

<img src="https://i.imgur.com/jIhBFdR.png">


Gồm 2 thành phần chính là Health-checker và VRRP 

- Health-checker worker threads: mỗi một health-check được đăng kí với global scheduling framework. Nó sẽ đảm đương nhiệm vụ này bằng cách sử dụng Keepalived health-check framework. health-check framework chứa 3 checkers:

  - TCP_CHECK: Hoạt động ở layer 4. Nếu remote server không trả lời thì server sẽ bị loại khỏi server pool.
  
  - HTTP_GET: Hoạt động ở layer 5. Thực hiện GET HTTP với các url. Kết quả sẽ được tổng hợp lại sử dụng MD5 algorithm nếu nó ko match với giá trị mong muốn, server sẽ bị loại khỏi server pool. Module này hoạt động hiệu quả khi server bạn chạy nhiều hơn một ứng dụng.
  
  - SSL_GET: giống với HTTP_GET nhưng sử dụng ssl connection để kết nối tới server.
  
  - MISC_CHECK: cho phép người dùng tự tạo script để chạy như là Health-checker. Kết quả buộc phải là 0 hoặc 1.

- VRRP Packet Dispatcher: Phân tách I/O để quản lí các VRRP Instance

Hai thành phần này sử dụng những công cụ sau:

  - SMTP notification: cho phép keepalived gửi cảnh báo
  
  - IPVS framework: LVS kernel interface cho việc quản lí real server pool
  
  - Netlink: Kernel routing interface cho VRRP. Quản lí VRRP VIP
  
  - Multicast: Dùng để gửi VRRP adverts
  
  - NETFILTER
  
  - SYSLOG
  
### 2. Cấu hình keepalived

