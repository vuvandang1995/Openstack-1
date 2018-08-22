# Hướng dẫn cài đặt card mạng cho kvm bằng command client

- Đầu tiên ta truy cập vào thư mục /var/lib/libvirt/network/  tạo 1 file .xml (ví dụ: `/var/lib/libvirt/network/isolated.xml`) trong file isolated.xml ta thêm các dòng sau.

```
<network>
<name>isolated</name>
</network>
```

- Tiếp theo ta tiến hành define network từ file isolated.xml bằng câu lệnh `virsh net-define isolated.xml`

<img src="http://i.imgur.com/YQWwgu0.png">

- Sau khi đã define, bạn có thể sử dụng câu lệnh `virsh net-list --all` để xem network available:

<img src="http://i.imgur.com/ZQStMFe.png">

`isolated` đã xuất hiện, tuy nhiên nó vẫn chưa được active

- Sau khi define, libvirt sẽ tự động add thêm một số thành phần vào file isolated.xml bạn vừa tạo và lưu nó tại `/etc/libvirt/qemu/networks/`

<img src="https://i.imgur.com/DSYm0fk.png">

- Tiếp theo ta chỉnh sửa file isolated.xml bằng câu lệnh `virsh net-edit isolated` (bằng việc bạn có thể cat file cấu hình của card default bên cạnh và thêm các thông số ip vào) .

<img src="https://i.imgur.com/KtYWiql.png">

Bên cạnh đó, bạn cũng có thể dùng lệnh `virsh net-dumpxml isolated` để xem chi tiết cấu hình trong các file network isolated.xml của Linux Bridge.

<img src="https://i.imgur.com/I401zTH.png">

- Sau khi cấu hình xong, ta tiến hành start virtual network vừa tạo bằng câu lệnh 

`virsh net-start isolated` (lệnh `virsh net-destroy isolated` để stop card isolated) 

<img src="http://i.imgur.com/e5vs4IZ.png">

Như vậy, `isolated` đã được start và có thể sử dụng.
- Để card isolated được bật tự động ta dùng lệnh virsh `net-autostart isolated` và file isolated.xml sẽ được copy vào thư mục `/etc/libvirt/qemu/networks/autostart` và được active 

<img src="https://i.imgur.com/fkN78V3.png">


Dùng virt-manager để xem:

<img src="https://i.imgur.com/gdlTT8p.png">

- Trong trường hợp người dùng muốn thay đổi cấu hình, tiến hành chỉnh sửa trong file xml rồi dùng lệnh `virsh net-destroy` và `virt net-start` để reset lại virtual network.
