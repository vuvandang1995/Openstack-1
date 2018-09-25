# Đặt ngưỡng cảnh báo cho dịch vụ


Trên Web UI, chúng ta tìm đến **Views**, mở tab **Services** và chọn **All services**

Trong ví dụ này, tôi sẽ đặt ngưỡng cảnh báo khi thư mục `/` được sử dụng 80% dung lượng thì cảnh báo `Warning` và 95% dung lượng là `Critical`.

<img src="https://i.imgur.com/XyQCsGq.png" />

Đặt lại ngưỡng cảnh báo

<img src="https://i.imgur.com/EDGHoxk.png" />

Thêm một rule mới

<img src="https://i.imgur.com/aB3H2OE.png" />

Tìm đến **Parameters**, chúng ta sẽ đặt ngưỡng cho `/` ở đây

<img src="https://i.imgur.com/qTTmmNc.png" />

Kéo xuống bên dưới, click vào **SAVE** để lưu lại rule

<img src="https://i.imgur.com/rGym1Mt.png" />

Lưu lại những thay đổi

<img src="https://i.imgur.com/VPHMT9D.png" />

<img src="https://i.imgur.com/3KeUHJD.png" />

<img src="https://i.imgur.com/xI2bZKK.png" />

Sau khi đặt ngưỡng cảnh báo xong, bây giờ chúng ta sẽ đi đến phần cấu hình cho OMD có thể gửi mail cảnh báo khi host/services gặp sự cố hoặc vượt ngưỡng vừa set
