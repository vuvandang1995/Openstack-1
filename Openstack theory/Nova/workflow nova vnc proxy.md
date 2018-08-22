# Phần này giới thiệu về quá trình kết nối tới máy ảo bằng vnc

<img src="https://i.imgur.com/pAYt8vp.png">

### Tính năng VNC Proxy

- Tạo ra 1 vùng mạng private và cô lập public network.
- VNC Client chạy trên mạng public, VNC Server chạy trên mạng private, VNC Proxy giống như 1 cầu nối kết nối giữa 2 đối tượng với nhau.
- VNC Proxy xác thực VNC Client bằng token.
- VNC Proxy không chỉ cho phép truy cập bảo mật hơn bằng cách tạo ra mạng private mà còn cho phép tách riêng việc triển khai VNC Server, nó có thể hỗ trợ Hypervisor VNC Server khác nhau nhưng không ảnh hưởng đến trải nghiệm của người dùng.

### Triển khai VNC Proxy
