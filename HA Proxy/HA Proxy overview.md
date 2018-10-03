# Tìm hiểu về HA Proxy

**HAProxy - High Availability Proxy**: Mục đích sử dụng là cân bằng tải TCP/HTTP

- Phần mềm mã nguồn mở Load Balancer và proxying TCP/HTTP chạy trên Linux, Solaris và FreeBSD

- Được dùng nhằm tăng hiệu năng và đáp ứng sự phân tán khối lượng công việc trên các server (web, ứng dụng, database...)

- Được sử dụng trong rất nhiều môi trường cấu hình cao, như : Github, Imgur, Instagram, Twitter...


## I. Một số thuật ngữ

### 1. Access Control List (ACL)

ACLs được dùng để test 1 vài điều kiện và thực hiện hành động (chọn server, block 1 request...) dựa vào kết quả kiểm tra. Sử dụng ACLs cho phép forward linh hoạt các network traffic dựa vào các nhân tố như pattern-matching và một số kết nối tới backend.

