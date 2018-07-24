
`/var/lib/glance/images` lưu image khi boot từ local lên controller để tạo image trên dashboard
`/var/lib/nova/instances/_base`  đây là nơi lưu hệ điều hành máy ảo máy ảo trên compute

Nếu boot máy ảo từ local sẽ tạo ra máy ảo chứa các file disk, hệ điều hành,  console.log `/var/lib/nova/instances` và hệ điều hành sẽ lưu ở `/var/lib/nova/instances/_base`

Nếu boot từ volume thì sẽ lưu ở controller

Kiểm tra xem máy ảo có hệ điều hành trong `/var/lib/nova/instances/_base` tên là gì:

Ví dụ: máy ảo có id là ca1dc120-46e9-4763-a700-cef9cd174c0f

- Ta truy cập vào `/var/lib/nova/instances/ca1dc120-46e9-4763-a700-cef9cd174c0f` có các file `console.log` , `disk` , `disk.info`

<img src="https://i.imgur.com/Rk9XAXU.png">

- Tiếp tục trong thư mục  `/var/lib/nova/instances/ca1dc120-46e9-4763-a700-cef9cd174c0f` ta gõ lệnh `qemu-img info disk` sẽ hiện ra đường dẫn đến hệ điều hành tương ứng (disk ở đây là disk của máy ảo với id ca1dc120-46e9-4763-a700-cef9cd174c0f)

<img src="https://i.imgur.com/WUF5AsV.png">
