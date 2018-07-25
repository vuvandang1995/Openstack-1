# Vấn đề về volume

Khi tạo 1 volume trống để sử dụng attach cho máy ảo,

- Tạo partition cần phân vùng
- Tạo LVM
- format định dạng
- mount vào 
 -có thể detach volume từ máy ảo này và attach sang máy ảo khác và sử dụng được luôn , dữ liệu cũ bên trong volume vẫn còn nguyên


#### Extend volume

- Extend volume ( máy ảo boot từ volume ): Ta cần phải vào tab admin chọn volume -> chọn update volume status -> chuyển từ trạng thái in-use sang trạng thái available (không cần shutdown máy ảo) , sau đó quay lại mục Project -> volume và có thể extend được , sau đó start volume sẽ chuyển lại trạng thái in-use

- Extend volume (các volume attach riêng):  làm tương tự vào mục admin -> update volume status -> chuyển từ trạng thái in-use sang available -> quay trở lại mục project -> volume -> extend -> restart instance

---------------
# Vấn đề snapshot

- Khi snapshot máy ảo có volume attach nó chỉ snapshot image

--------------

# Vấn đề network

- Ta có thể tạo fixed ip bằng cách click vào network mà ta đã tạo -> port -> tạo port tương ứng với ip fixed, sau đó detach interface và attach interface vào máy ảo là máy ảo có ip fixed


- Khi xóa máy ảo và ip tạo bằng dhcp thì port sẽ bị mất nhưng nếu tạo ip bằng fixed sẽ không mất


#### Cách xóa network

Cần xóa các resource như router, port, subnet... liên quan đến network mới có thể xóa được

- Hiển thị máy ảo  `nova list`

- Xóa router : openstack router delete <router ID>

- Detach interface khi đang có port được sử dụng nếu port không được sử dụng ta có thể xóa port luôn  `openstack port delete <port ID>`

- Hiển thị thông tin port tương ứng với máy ảo `nova interface-list <tên máy ảo>`

<img src="https://i.imgur.com/HhNfoio.png">


- Detach interface khỏi máy ảo `nova interface-detach ducnm37-22222 a733b22b-9a4e-47ee-aec8-8cd69f5e8e0e` 

- Dùng lệnh `openstack port list` xem danh sách các interface 

- Xóa  port, subnet : Dùng lệnh `openstack port delete <port ID>` để xóa interface (trong trường hợp port đang sử dụng thì mới phải detach interface rồi mới xóa) hoặc xóa subnet `openstack subnet delete <subnet ID>`

Sau khi xóa các resource như bên trên thì ta mới có thể xóa network được : `openstack network list` để xem danh sách các network và `openstack netowrk delete <network ID>` để xóa network đó đi

