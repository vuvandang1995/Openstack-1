# Phần này sẽ giới thiệu về firewalld

## 1. Giới thiệu

**Firewalld** là công cụ hoàn toàn mới trong RHEL 7. Mục đích là thay thế iptables và kết nối vào netfilter kernel code. . Nó cung cấp các dòng lệnh và giao diện đồ họa và có sẵn trong kho của hầu hết các bản phân phối Linux. Làm việc với FirewallD có hai sự khác biệt chính so với iptables kiểm soát trực tiếp

- FirewallD sử dụng các zone và dịch vụ (service) thay vì chuỗi và quy tắc.

- Nó quản lý rulesets tự động, cho phép cập nhật mà không vi phạm các phiên và kết nối hiện tại.

- Trong firewalld phân chia ra các cấu hình thành 2 loại là runtime và permanent:

<li>
<ul>
<ul>
<li>Runtime(mặc định): có tác dụng ngay lập tức, mất hiệu lực khi reboot hệ thống</li>
<li>Permanent: không áp dụng cho hệ thống đang chạy, cần reload mới có hiệu lực, tác dụng vĩnh viễn cả khi reboot hệ thống</li>
</ul>
</ul>
</li>

- Thiết lập Runtime và Permanent:
```
 firewall-cmd --zone=public --add-service=http  (Runtime)
 firewall-cmd --zone=public --add-service=http --permanent   (Permanent)
 firewall-cmd --reload
```

## Một số lệnh sử dụng trong firewalld

-Kiểm tra trạng thái firewalld

`firewall-cmd --state`

- Kiểm tra các zones trong hệ thống

`firewall-cmd --get-zones`

- Mở port theo chế độ Permanent

`firewall-cmd --zone=public --add-port=80/tcp --permanent`

-Mở 1 dải port theo chế dộ Permanent hoặc remove ra khỏi rule

`firewall-cmd --zone=public --add-port=4990-5000/tcp --permanent` (mở port 80)

`firewall-cmd --zone=public --remove-port=80/tcp --permanent` (remove port 80 ra khỏi list)

`firewall-cmd --zone=public --list-ports` (Kiểm tra lại danh sách port trong zone public)

- Kiểm tra lại các port đã mở trong zone active

`firewall-cmd --list-all`

- Kiểm tra trạng thái của tất cả các zones

`firewall-cmd --list-all-zones`

- xác định các services trên hệ thống 

`firewall-cmd --get-services`

- Cho phép service trên hệ thống

`firewall-cmd --zone=public --add-service=http --permanent`

- Block mọi kết nối ra hoặc vào server

`firewall-cmd --panic-on`

- Hủy bỏ block kết nối ra hoặc vào server

`firewall-cmd --panic-off`

- Chặn ip 

`firewall-cmd --direct --add-rule ipv4 filter INPUT 0 -s 192.168.40.70 -j REJECT` hoặc chặn 1 dải ip `firewall-cmd --direct --add-rule ipv4 filter INPUT 0 -s 10.110.4.0/22 -d 10.110.0.0/22 -j REJECT`
