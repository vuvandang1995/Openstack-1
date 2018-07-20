# Phần này nói về việc quản lý dashboard bằng dòng lệnh

### Glance

- Chạy câu lệnh xác thực biến môi trường: `source /root/keystonerc_admin`

- Hiển thị danh sách image: `openstack image list`

- Xem thông tin chi tiết một image: `openstack image show <tên/ID image>`

- Tạo mới image từ file tải về: 

                               `openstack image create --file /tmp/cirros-0.3.4-x86_64-disk.img
                                --disk-format qcow2 \
                                --container-format bare --public cirros-0.3.4-x86_64`
- Sửa file image:

`
openstack image set
 [--name <name>]
 [--container-format <container-format>]
 [--disk-format <disk-format>]
 [--size <size>]
 [--protected | --unprotected]
 [--public | --private | --community | --shared]
 [--store <store>]
 [--location <image-url>]
 [--copy-from <image-url>]
 [--file <file>]
 [--volume <volume>]
<image>
Trong đó:
--name, --disk-format, --size,... là những thông số muốn thay đổi
<image> là tên/ID của image muốn sửa
`
- Thay đổi tên image: `openstack image set --name cirros1 cirros`

- Xóa image: openstack `image delete <tên/ID image>`

### Neutron

- Hiển thị toàn bộ danh sách các network hiện tại: `openstack network list`

- Hiển thị thông tin chi tiết một network: `openstack network show <tên network>`

- Tạo Network:

`
openstack network create
 [--share | --no-share] [--enable | --disable]
 [--enable-port-security | --disable-port-security]
 [--provider-network-type <provider-network-type>]
 [--provider-physical-network <provider-physical-network>]
 <tên network>
`

 Trong đó:
 
--share | --no-share: chọn chế độ share hoặc không share network này
cho các tenant khác. Thông thường option này dùng để chọn khi tạo
provider network mà người admin muốn share cho các project và user.

--enable-port-security: có ý nghĩa cho tất cả các port được tạo trên
network này sẽ được kích hoạt tính năng port security. Và mặc định là bật.

--disable-port-security: tắt tính năng port security.

--provider-network-type: loại network, có thể là gre, vlan, flat,...

Ví dụ tạo network loại flat:

openstack network create --share \
 --provider-physical-network provider \
 --provider-network-type flat <tên network>
`

