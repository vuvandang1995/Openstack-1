# Hướng dẫn sử dụng check mk để monitor switch

## Yêu cầu

Switch phải hỗ trợ SNMP

## Hướng dẫn cấu hình

Phía switch cấu hình như sau

<img src="https://i.imgur.com/eX4GiHN.png">

Trong đó `public` là community string mà ta sẽ sử dụng để cấu hình trên phía checkmk, và ip là ip quản lý switch
Lưu ý: Cần đảm bảo checkmk server có quyền truy cập tới SW về mặt network

Ở trên phía checkmk ta khai báo như sau:

<img src="https://i.imgur.com/gdOgQ9D.png">

<img src="https://i.imgur.com/3VuB9dM.png">

Tiến hành `Discovery`

<img src="https://i.imgur.com/1O6R0bC.png">

Kết quả

<img src="https://i.imgur.com/XWF3AC6.png">
