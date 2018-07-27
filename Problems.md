# Tổng hợp các vấn đề liên quan trong quản trị openstack

## Vấn đề về volume

Khi tạo 1 volume trống  sử dụng attach cho máy ảo ta attach volume vào máy ảo nhưng chưa sử dụng được cần truy cập vào máy ảo và làm các bước sau:

- Tạo partition cần phân vùng
- Tạo LVM
- format định dạng
- mount vào 
 -có thể detach volume từ máy ảo này và attach sang máy ảo khác và sử dụng được luôn , dữ liệu cũ bên trong volume vẫn còn nguyên


### Extend volume

- Extend volume ( máy ảo boot từ volume ): Ta cần phải vào tab admin chọn volume -> chọn update volume status -> chuyển từ trạng thái in-use sang trạng thái available (không cần shutdown máy ảo) , sau đó quay lại mục Project -> volume và có thể extend được , sau đó start volume sẽ chuyển lại trạng thái in-use

- Extend volume (các volume attach riêng):  làm tương tự vào mục admin -> update volume status -> chuyển từ trạng thái in-use sang available -> quay trở lại mục project -> volume -> extend -> restart instance

- Dùng lệnh

xem danh sách và trạng thái volume: `openstack volume list`

Thay đổi trạng thái volume sang available `openstack volume set <volume ID> --status available`

EXtend volume `openstack volume set <volume ID> --size <dung lượng muốn mở rộng tính bằng GB>`

---------------
## Vấn đề snapshot

- Khi snapshot máy ảo có volume attach nó chỉ snapshot image

--------------

## Vấn đề network

Ta có thể tạo fixed ip bằng cách click vào network mà ta đã tạo -> port -> tạo port tương ứng với ip fixed, sau đó detach interface và attach interface vào máy ảo là máy ảo có ip fixed

Ví dụ tạo fixed ip 10.10.10.10:

```
openstack network create localll

openstack subnet create subnettt --network local --subnet-range 10.10.10.0/24

openstack port create --network localll --fixed-ip subnet=subnettt,ip-address=10.10.10.10 port1
```


Khi xóa máy ảo và ip tạo bằng dhcp thì port sẽ bị mất nhưng nếu tạo ip bằng fixed sẽ không mất

### Attach port ( thêm ip vào máy ảo)

`nova interface-attach --port-id <port ID> <ID máy ảo>` (với lệnh này thì trạng thái của port phải là DOWN mới dùng được, những port được tạo ra bởi DHCP kể cả khi xóa máy ảo cũng sẽ ở trạng thái ACTIVE nên ta cần xóa port đó đi, còn các port được tạo bởi ip fixed khi xóa máy ảo sẽ chuyển từ trạng thái ACTIVE về trạng thái DOWN)
 
- Sau khi add ta cần vào máy ảo cấu hình thêm card mạng để máy ảo nhận thêm IP mới
 
 sudo chmod 777 /etc/network/interfaces
 vi /etc/network/interfaces
  
     auto eth1
     iface eth1 inet dhcp
 
 
### Cách xóa network

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

-----------------
## Vấn đề về Resize máy ảo

Resize máy ảo boot từ local ta  chỉ resize được flavor có root disk lớn hơn không resize được root disk nhỏ hơn


-----------------

## Vấn đề về snapshot 

- Snapshot máy ảo boot từ volume sẽ chụp lại 1 file image lưu tại /var/lib/glance/images với dung lượng 0 Kb là file đường dẫn tới file cấu hình của máy ảo ban đầu (volume,image) volume lưu ở /var/lib/cinder

- Snapshot máy ảo boot từ volume và có volume attach tương tự như snapshot máy ảo sẽ chụp lại file image với dung lượng 0Kb và tạo 2 file snapshot volume và volume attach cũng được tạo ra lưu ở /var/lib/cinder

- Snapshot máy ảo boot từ local sẽ tạo ra 1 file image giống hệt và lưu tại /var/lib/glance/images

---------------
## Vấn đề về tạo máy ảo
- Tạo volume boot từ image
```
    openstack volume create \
    --image f03724d0-62de-4e5b-9350-2dcefdb09732 \
    --size 20 \
    --type iscsi \
    --availability-zone nova \
    newvolume
```
- Tạo máy ảo boot từ volume (*lưu ý: khi có thêm option --security group có thể báo lỗi attach volume cold not found*)
```
    openstack server create \
    --volume 82a1c5e8-605d-4c57-b088-7410fd284bdc \
    --flavor 2 \
    --nic net-id=8a9683d2-f755-48b3-b216-33333adc56fa \
    newvm
```
- Tạo volume trống
```
    openstack volume create \
    --size 20 \
    --type iscsi \
    --availability-zone nova \
    newvolume2
```
- Cài máy ảo từ image đồng thời sử dụng volume trống đã tạo
```
    openstack server create \
    --flavor 2 \
    --image f03724d0-62de-4e5b-9350-2dcefdb09732 \
    --nic net-id=8a9683d2-f755-48b3-b216-33333adc56fa \
    --block-device-mapping id=2d079a2f-7889-44be-b52d-021998f27714 \ (id ở đây là id của volume trống đã tạo)
    newvm2
````

- Boot máy ảo từ image đồng thời tạo volume
```
    nova boot --flavor m1.tiny \
    --block-device source=image,id=f03724d0-62de-4e5b-9350-2dcefdb09732,dest=volume,size=1,shutdown=remove,bootindex=0 \ (id ở đây là id của image)
    --nic net-id=83714bed-d295-4539-b519-045732de3502 \
    --availability-zone nova \
    newvmm
````
