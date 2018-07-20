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

```
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
```

Trong đó:
--name, --disk-format, --size,... là những thông số muốn thay đổi

<image> là tên/ID của image muốn sửa

- Thay đổi tên image: `openstack image set --name cirros1 cirros`

- Xóa image: openstack `image delete <tên/ID image>`

### Neutron

- Hiển thị toàn bộ danh sách các network hiện tại: `openstack network list`

- Hiển thị thông tin chi tiết một network: `openstack network show <tên network>`

- Tạo Network:

```
openstack network create
 [--share | --no-share] [--enable | --disable]
 [--enable-port-security | --disable-port-security]
 [--provider-network-type <provider-network-type>]
 [--provider-physical-network <provider-physical-network>]
 <tên network>
```

 Trong đó:
 
--share | --no-share: chọn chế độ share hoặc không share network này cho các tenant khác. Thông thường option này dùng để chọn khi tạo provider network mà người admin muốn share cho các project và user.

--enable-port-security: có ý nghĩa cho tất cả các port được tạo trên network này sẽ được kích hoạt tính năng port security. Và mặc định là bật.

--disable-port-security: tắt tính năng port security.

--provider-network-type: loại network, có thể là gre, vlan, flat,...

Ví dụ tạo network loại flat:

openstack network create --share \
 --provider-physical-network provider \
 --provider-network-type flat <tên network>

- Sửa Network: 

```
openstack network set
 [--name <name>] [--enable | --disable]
 [--share | --no-share]
 [--enable-port-security | --disable-port-security]
 <tên network>
```

Ví dụ: Sửa tên nam_network thành nữ network: `openstack network set nam_network --name nu_network`

- Xoá Network: `openstack network delete <tên network>`

Lưu ý: Khi xoá network sẽ liên quan đến một số resource khác như router, instance. Vậy nên trước khi xoá network thì cần phải xoá hết các resource liên quan đến network đó

- Hiển thị tất cả các subnet hiện tai: `openstack subnet list`

- Hiển thị thông tin chi tiết của một subne: `openstack subnet show <tên subnet>`

- Tạo subnet:

```
openstack subnet create
 --network <network>
 --prefix-length <prefix-length>
 --subnet-range <subnet-range>
 --dhcp | --no-dhcp] [--gateway <gateway>
 --ip-version {4,6}
 --ipv6-ra-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}
 --ipv6-address-mode {dhcpv6-stateful,dhcpv6-stateless,slaac}
 --allocation-pool start=<ip-address>,end=<ip-address>
 --dns-nameserver <dns-nameserver>
 --host-route destination=<subnet>,gateway=<ip-address>
```
- Sửa Subnet:
 ```
 openstack subnet set 
  --name <name> 
  --dhcp | --no-dhcp
  --gateway <gateway>
  --allocation-pool start=<ip-address>,end=<ip-address>
  --dns-nameserver <dns-nameserver>
  --host-route destination=<subnet>,gateway=<ip-address>
  <tên subnet cần sửa>
```
    
- Xoá Subnet: `openstack subnet delete <tên hoặc ID của subnet>`

- Hiển thị tất cả các port hiện tại: 
```
openstack port list 
--device-owner <device-owner>
--router <router>
```
Trong đó:

--device-owner:device-owner của port. Ví dụ: --device-owner là network:router-interface thì câu lệnh sẽ hiển thị ra tất cả những port có
device-owner là network:router-interface.

--router: tên router, khi truyền option này thì kết quả sẽ trả ra tất cả port gắn vào router đó.

- Hiển thị thông tin chi tiết một port: `openstack port show <tên hoặc ID của port>`

- Tạo sửa xóa port: 
```
openstack port create
 --network <network>
 --prefix PREFIX 
 --device <device-id>
 --device-owner <device-owner>
 --vnic-type <vnic-type>]
 --host <host-id>
 --fixed-ip subnet=<subnet>,ip-address=<ip-address>
 --binding-profile <binding-profile>
 --enable | --disable
 --mac-address <mac-address>
 --project <project>
 --project-domain <project-domain>
 <tên port>
```

Trong đó:
--network: tên hoặc ID của network chứa port.
--device: device ID của port.
--device-owner: device-owner của port.
--vnic-type: loại VNIC dành cho port, có thể là direct, direct-physical,...
--host: ID của host, port đặt lên.
--fixed-ip: thông tin của subnet, và IP muốn gán cho port.
--mac-address: gán MAC cho port.
--enable | --disable: enable hoặc disable port này.

