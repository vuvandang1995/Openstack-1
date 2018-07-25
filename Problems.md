# Vấn đề về volume

Khi tạo 1 volume trống để sử dụng attach cho máy ảo,

- Tạo partition cần phân vùng
- Tạo LVM
- format định dạng
- mount vào 
 -có thể detach volume từ máy ảo này và attach sang máy ảo khác và sử dụng được luôn , dữ liệu cũ bên trong volume vẫn còn nguyên


---------------
# Vấn đề snapshot

- Khi snapshot máy ảo có volume attach nó chỉ snapshot image

--------------

# Vấn đề network

- Ta có thể tạo fixed ip bằng cách click vào network mà ta đã tạo -> port -> tạo port tương ứng với ip fixed, sau đó detach interface và attach interface vào máy ảo là máy ảo có ip fixed


- Khi xóa máy ảo và ip tạo bằng dhcp thì port sẽ bị mất nhưng nếu tạo ip bằng fixed sẽ không mất


- Detach interface 
`nova interface-list <tên máy ảo>`

<img src="https://i.imgur.com/HhNfoio.png">


Lệnh detach interfaec khỏi máy ảo `nova interface-detach ducnm37-22222 a733b22b-9a4e-47ee-aec8-8cd69f5e8e0e` 

Sau đó dùng lệnh `openstack port list` xem danh sách các interface 

Dùng lệnh `openstack port delete <port ID>` để xóa interface hoặc xóa subnet `openstack subnet delete <subnet ID>`

Sau khi xóa các resource như bên trên thì ta mới có thể xóa network được : `openstack network list` để xem danh sách các network và `openstack netowrk delete <network ID>` để xóa network đó đi

