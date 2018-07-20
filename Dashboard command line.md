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

--network: tên hoặc ID của network chứa port

--device: device ID của port

--device-owner: device-owner của port

--vnic-type: loại VNIC dành cho port, có thể là direct, direct-physical,...

--host: ID của host, port đặt lên

--fixed-ip: thông tin của subnet, và IP muốn gán cho port

--mac-address: gán MAC cho port

--enable | --disable: enable hoặc disable port này


Ví dụ: Tạo một port trên private_subnet thuộc network private_network và có IP=10.10.10.100.
`
openstack port create --network private_netwok --fixed-ip subnet=private_subnet, ip-address=10.10.10.100 port_test
`

- Sửa Port:
```
openstack port set
 --name <name>
 --device <device-id>
 --device-owner <device-owner>
 --vnic-type <vnic-type> 
 --host <host-id>
 --enable | --disable 
 --fixed-ip subnet=<subnet>,ip-address=<ip-address> | --no-fixed-ip
 --binding-profile <binding-profile> | --no-binding-profile]
 <tên hoặc ID port>
```
Trong đó:

--device: update lại device ID của port.

--device-owner: update lại device-owner của port.

--vnic-type: update lại vnic-type của port

--enable | --disable: update lại trạng thái của port.

--fixed-ip: thêm subnet và địa chỉ IP cho port

--binding-porfile: update lại binding-profile của port.

Ví dụ: Update lại thông tin của port vừa tạo với tên là “port_test" thành “port_update". `openstack port set --name port_update port_test`

-  Xoá Port: `openstack port delete <tên hoặc ID của Port>`

- Hiển thị toàn bộ router hiện có: `openstack router list --long`

Trong đó: --long: dùng để hiển thị thêm nhiều thông tin.

- Hiển thị thông tin chi tiết một router: `openstack router show <tên hoặc ID router>`

- Tạo Router:
```
openstack router create 
--enable | --disable 
--distributed
<tên router>
```
Trong đó:

--enable | --disable: trạng thái của router

--distributed: có phân tán router hay không, đây là một tính năng HA cho router
 Ví dụ: openstack router create new_router

- Sửa Router: 
```
openstack router set
 --name <name> 
 --enable | --disable
 --distributed | --centralized
 --route destination=<subnet>,gateway=<ip-address> | --no-route
 <tên router>
```
Trong đó:

--name: update tên router.

--enable | --disable: update lại trạng thái của router.

--distributed | --centralized: update lại router phân tán hay tập chung.

--router: thêm route cho router.

Ví dụ: Update lại tên của router từ “new_router” thành “update_router” `openstack router set --name update_router new_router`

- Xoá Router: `openstack router delete <tên hoặc ID router>`

### Nova

- Tạo flavor:

`openstack flavor create --id <id> --ram <size-mb> --disk <size-gb> --vcpus <num-cpu> --public|--private <flavor-name>`

Ví dụ: tạo flavor có các thông số như sau :
RAM: 512 MB
Disk: 1 GB
Vcpus: 1
Được phép sử dụng bởi các project khác
Tên flavor: m1.tiny

`openstack flavor create --ram 512 --disk 1 --vcpus 1 --public m1.tiny`

- Sửa Flavor: 
```
openstack flavor set 
--property <key=value> 
--project <project>
--project-domain <project-domain> <flavor>
```
Trong đó
<key=value> : thông số của flavor, ví dụ ram, disk,...

<project> : tùy chỉnh project được phép sử dụng flavor

<project-domain> : tùy chỉnh domain được phép sử dụng flavor

<flavor> : tên flavor muốn chỉnh sửa

- Xem danh sách các flavor đang có: `openstack flavor list`

- Xóa Flavor: `openstack flavor delete <tên hoặc ID flavor>`

- Tạo key pair: `openstack keypair create <name> <file_private>`

Trong đó:

<name> : Tên của keypair

<file_private> : Tên file để lưu private key, nếu không được chọn, private key sẽ được in ra màn hình

Lưu ý: Public key được tạo sẽ được đẩy lên OpenStack 

Ví dụ: tạo key pair key1 `openstack keypair create key1 > private_key1`

- Xem danh sách keypair: `openstack keypair list`

- Xóa key pair: `openstack keypair delete <tên key pair>`


Trước khi sử dụng private key, bạn nên chắc chắn rằng private key đó tuân thủ theo đúng GNU/Linux permissions:

```
chmod 600 private_key1
ls -l
ssh -i ~/key1 root@192.168.100.245
```

Để SSH vào máy ảo đã được chèn key pair, sử dụng câu lệnh sau: `ssh -i ~/key1 root@192.168.100.245`

Trong đó:

Tùy chọn –i giúp khai báo thông tin của private key

~/key1 là file private key

root@192.168.100.245 là tên tài khoản và ip của máy ảo muốn ssh tới

- Tạo máy ảo boot từ image:
```
openstack server create 
--image <image> 
--flavor <flavor>
--security-group <security-group> 
--key-name <key-name>
--user-data <user-data> 
--nic <net-id> <server-name>
```

Trong đó:
<image> : Tên hoặc ID của image dùng để boot máy ảo
  
<flavor> : Tên hoặc ID của flavor được dùng để boot máy ảo
  
<security-group> : Tên hoặc ID của security group để gán vào máy ảo, mặc định sẽ được gán vào default security group

<key-name> : Tên của key pair gán vào máy ảo

<user-data> : File chèn vào máy ảo trước khi boot

<net-id> : ID của network mà máy ảo sử dụng

<server-name> : Tên máy ảo muốn tạo

Các thông số bắt buộc đó là Tên, Flavor, Image, Network


Ví dụ: Tạo một máy ảo mới có tên MDT-VM02, sử dụng image ubuntu, flavor m1.small, ssh key pair là key1, file cloud-init chèn password là data.file và network sử dụng là external_network.

```
openstack server create 
--image ubuntu 
--flavor m1.small
--key-name key1 
--user-data data.file 
--nic net-id=e6aa71b0-9a50-4614-a8e2-6f371b3dc18a MDT-VM02
```
- Xem danh sách máy ảo đang có: `openstack server list`

- Lấy địa chỉ URL kết nối tới giao diện máy ảo thông qua noVNC: `openstack console url show --novnc <tên máy ảo>`

- Tắt máy ảo: `openstack server stop <tên/ID máy ảo>`

- Bật máy ảo: `openstack server start <tên/ID máy ảo>`

- Reboot máy ảo: `openstack server reboot --hard | --soft <tên/ID máy ảo>`

- Tạm ngưng máy ảo: `openstack server suspend <tên/ID máy ảo>`

- Xóa máy ảo: `openstack server delete <tên/ID máy ảo>`

- Tạo snapshot từ máy ảo: `openstack server image create --name <name> <vm_name>`

Trong đó:

<vm_name> là tên máy ảo muốn tạo snapshot

<name> là tên của snapshot được tạo ra

Ví dụ: Tạo snapshot có tên “snap1” của máy ảo vm01 `openstack server image create --name snap1 vm01`

- Tạo snapshot máy ảo (*Snapshot được tạo sẽ được lưu dưới dạng image, ta có thể dùng nó để boot mới máy
ảo*): `openstack server create --image <image> --flavor <flavor> <name>`

Trong đó:

<image> : tên hoặc ID của image snapshot được tạo từ VMs

<flavor> : tên flavor sử dụng cho máy ảo mới

<name> : tên máy ảo mới
  
- Cold migrate

Tắt máy ảo: `openstack server stop <tên máy ảo>`

Migrate vm, nova-scheduler sẽ dựa vào cấu hình balancing weitgh và filter để define ra
node compute đích: `openstack migrate <tên máy ảo>`

Theo dõi trạng thái của máy ảo: `openstack server show <tên máy ảo>`

Chờ đến khi vm thay đổi trạng thái sang VERIFY_RESIZE, confirm việc migrate: `openstack server resize --confirm`

- True live migration

`nova live-migration Compute_host Vm_name`

Trong đó:

Vm_name là tên máy ảo cần migrate

Compute_host là host chỉ định để migrate máy ảo tới

-  Block live migration

`openstack server migrate --live Compute_host --block-migration Vm_name`

Trong đó:

Vm_name là tên máy ảo cần migrate

Compute_host là host chỉ định để migrate máy ảo tới

-   resize Thay đổi kích thước máy ảo:
