# Phần này sẽ giới thiệu về việc kết nối tới mysql database từ xa

B1. Tạo 1 user có quyền remote

`create user pac6@'%';` (ở đây user là pac6 và % là quyền truy cập từ xa vào mysql

`SET PASSWORD FOR pac6@'%'=PASSWORD('123456');` (set password cho user pac6 là 123456)

`GRANT all ON * . * TO pac6@'%';` (phân quyền all cho user pac6)

`FLUSH PRIVILEGES;`  (update rule)

`select User,Host from mysql.user;` (kiểm tra user vừa tạo)

<img src="https://i.imgur.com/oB60OBy.png">

B2. mở port 

Mặc định để kết nối tới mysql database trên port 3306

Mặc định centos sẽ sử dụng firewalld

`firewall-cmd --get-active-zones` (kiểm tra card mạng đang hoạt động)

`firewall-cmd --zone=public --add-port=3306/tcp --permanent`  (mở port 3306 trên firewalld)

`firewall-cmd --reload` (update rule)

B3. Cài đặt MySQL GUI Tools hoặc mysql workbench

https://downloads.mysql.com/archives/gui/

Truy cập với giao diện trên window 

<img src="https://i.imgur.com/VMThFNU.png">

<img src="https://i.imgur.com/7YJGooC.png">
