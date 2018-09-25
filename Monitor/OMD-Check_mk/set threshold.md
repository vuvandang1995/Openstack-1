# Đặt ngưỡng cảnh báo cho dịch vụ


Trên Web UI, chúng ta tìm đến **Views**, mở tab **Services** và chọn **All services**

Trong ví dụ này, tôi sẽ đặt ngưỡng cảnh báo khi thư mục `/` được sử dụng 80% dung lượng thì cảnh báo `Warning` và 95% dung lượng là `Critical`.

<img src="https://i.imgur.com/XyQCsGq.png" />

Đặt lại ngưỡng cảnh báo

<img src="../images/20-set-nguong-2.png" />

Thêm một rule mới

<img src="../images/20-set-nguong-3.png" />

Tìm đến **Parameters**, chúng ta sẽ đặt ngưỡng cho `/` ở đây

<img src="../images/20-set-nguong-4.png" />

Kéo xuống bên dưới, click vào **SAVE** để lưu lại rule

<img src="../images/20-set-nguong-5.png" />

Lưu lại những thay đổi

<img src="../images/20-set-nguong-6.png" />

<img src="../images/20-set-nguong-7.png" />

<img src="../images/20-set-nguong-8.png" />

Sau khi đặt ngưỡng cảnh báo xong, bây giờ chúng ta sẽ đi đến phần cấu hình cho OMD có thể gửi mail cảnh báo khi host/services gặp sự cố hoặc vượt ngưỡng vừa set
