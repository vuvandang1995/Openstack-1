# Tìm hiểu về NOVA

## 1. Tổng quan

Là service chịu trách nhiệm chứa và quản lí các hệ thống cloud computing. OpenStack Compute chính là phần chính quan trọng nhất trong kiến trúc hệ thống Infrastructure-as-a-Service (IaaS). Phần lớn các modules của Nova được viết bằng Python.

OpenStack Compute giao tiếp với OpenStack Identity để xác thực, OpenStack Image để lấy images và OpenStack Dashboard để lấy giao diện cho người dùng và người quản trị

Nova cho phép bạn điều khiển các máy ảo và networks, bạn cũng có thể quản lí các truy cập tới cloud từ users và projects. OpenStack Compute không chứa các phần mềm ảo hóa. Thay vào đó, nó sẽ định nghĩa các drivers để tương tác với các kĩ thuật ảo hóa khác chạy trên hệ điều hành của bạn và cung cấp các chức năn thông qua một web-based API.

## 2. Các thành phần

- nova-api : Là service tiếp nhận và phản hồi các compute API calls từ user. Service hỗ trợ OpenStack Compute API, Amazon EC2 API và Admin API đặc biệt được dùng để user thực hiện các thao tác quản trị. Nó cũng có một loạt các policies và thực hiện hầu hết các orchestration activities ví dụ như chạy máy ảo.


- nova-compute : Là service chịu trách nhiệm tạo và hủy các máy ảo qua hypervisors APIs. Ví dụ:
<ul>
  <ul>
<li>XenAPI for XenServer/XCP</li>
    
<li>libvirt for KVM or QEMU</li>

<li>VMwareAPI for VMware</li>
</ul>
</ul>

Quá trình xử lí khá phức tạp, về cơ bản, daemon tiếp nhận các hành động từ hàng đợi và thực hiện một loạt các câu lệnh hệ thống để vận hành máy ảo như chạy máy ảo kvm và update trạng thái trong database.

- nova-placement-api : Lần đầu xuất hiện tại bản Newton, placement api được dùng để theo dõi thống kê và mức độ sử dụng của mỗi một resource provider. Provider ở đây có thể là compute node, shared storage pool hoặc IP allocation pool. Ví dụ, một máy ảo có thể được khởi tạo và lấy RAM, CPU từ compute node, lấy disk từ storage bên ngoài và lấy địa chỉ IP từ pool resource bên ngoài.

- nova-scheduler : Service này sẽ lấy các yêu cầu máy ảo đặt vào queue và xác định xem chúng được chạy trên compute server host nào.Nova-scheduler sẽ scan và nhận thông kê tài nguyên từ nova-placement-api để xác định tài compute trống nhiều tài nguyên nhất để khởi tạo máy ảo trên đó.

nova-conductor : Là module chịu trách nhiệm về các tương tác giữa nova-compute và database. Nó sẽ loại bỏ tất cả các kết nối trực tiếp từ nova-compute tới database nhằm mục đích bảo mật, tránh trường hợp máy ảo bị xóa mà không có chủ ý của người dùng.

- nova-consoleauth : Xác thực token cho user mà console proxies cung cấp. Dịch vụ này buộc phải chạy cùng với console proxies. Bạn có thể chạy proxies trên 1 nova-consoleauth service hoặc ở trong 1 cluster configuration.

- nova-novncproxy :  Cung cấp một proxy để truy cập máy ảo đang chạy thông qua kết nối VNC. Hỗ trợ các novnc client chạy trên trình duyệt.

- nova-spicehtml5proxy : Cung cấp proxy để truy cập các máy ảo đang chạy thông qua kết nối SPICE. Nó hỗ trợ các trình duyệt based HTML5 client.

- nova-xvpvncproxy : Cung cấp proxy cho việc truy cập các máy ảo đang chạy thông qua VNC connection. Nó hỗ trợ OpenStack-specific Java client.

- The queue: Trung tâm giao tiếp giữa các daemons. Thường dùng RabbitMQ hoặc các AMQP message queue khác như ZeroMQ.

- SQL database : Dùng để lưu các trạng thái của hạ tầng caloud bảo gồm:
<ul>
<ul>
<li>Các loại máy ảo có thể chạy</li>
    
<li>Các máy ảo đang được dùng</li>
    
<li>Các network đã có</li>
    
<li>Các projects</li>
</ul>
</ul>

Về cơ bản, OpenStack Compute hỗ trợ bất kỳ hệ quản trị cơ sở dữ liệu
 
 ## 3. Kiến trúc của Nova

<img src="http://i.imgur.com/SWgKmDp.png">

- DB: sql database để lưu trữ dữ liệu.
- API: Thành phần để nhận HTTP request , chuyển đổi các lệnh và giao tiếp với thành các thành phần khác thông qua oslo.messaging queuses hoặc HTTP.
- Scheduler: Quyết định ,máy chủ được chọn để chạy máy ảo.
- Network: quản lý ip forwarding, bridges, và vlan.
- Compute: Quản lý giao tiếp với hypervisor và vitual machines.
- Conductor: Xử lý các yêu cầu mà cần sự phối hợp (build/resize), hoạt động như một proxy cho cơ sở dữ liệu, hoặc đối tượng chuyển đổi.


## 4. Quá trình khởi tạo máy ảo trong Nova

<img src="http://i.imgur.com/drbTrDG.png">

<img src="http://i.imgur.com/HrBos46.png">

Ở ví dụ này có 2 hosts được sử dụng: compute host (hoạt động như Hypervisor khi nova-compute chạy) và controller host (chứa tất cả các dịch vụ quản lí).

Workflow khi khởi tạo máy ảo:

1. Client (có thể là Horizon hoặc CLI) hỏi tới keystone-api để xác thực và generate ra token.

2. Nếu quá trình xác thực thành công, client sẽ gửi request khởi chạy máy ảo tới nova-api. Giống câu lệnh `nova boot`.

3. Nova service sẽ kiểm tra token và nhận lại header với roles và permissions từ keystone-api.

4. Nova kiểm tra trong database để xem có conflicts nào với tên những objects đã có sẵn không và tạo mới một entry cho máy ảo mới trong database.

5. Nova-api gửi RPC tới nova-scheduler service để lên lịch cho máy ảo.

6. Nova-scheduler lấy request từ message queue

7. Nova-scheduler service sẽ tìm compute host thích hợp trong database thông qua filters và weights. Lúc này database sẽ cập nhật lại entry của máy ảo với host ID phù hợp nhận được từ nova-scheduler. Sau đó scheduler sẽ gửi RPC call tới nova-compute để khởi tạo máy ảo.

8. nova-compute lấy request từ message queue.

9. nova-compute hỏi nova-conductor để lấy thông tin về máy ảo như host ID, flavor. (nova-compute lấy các thông tin này từ database thông qua nova-conductor vì lý do bảo mật, tránh trường hợp nova-compute mang theo yêu cầu bất hợp lệ tới instance entry trong database)

10. nova-conductor lấy request từ message queue.

11. nova-conductor lấy thông tin máy ảo từ database.

12. nova-compute lấy thông tin máy ảo từ queue. Tại thời điểm này, compute host đã biết được image nào sẽ được sử dụng để chạy máy ảo. nova-compute sẽ hỏi tới glance-api để lấy url của image.

13. Glance-api sẽ xác thực token và gửi lại metadata của image trong đó bao gồm cả url của nó.

14. Nova-compute sẽ đưa token tới neutron-api và hỏi nó về network cho máy ảo.

15. Sau khi xác thực token, neutron sẽ tiến hành cấu hình network.

16. Nova-compute tương tác với cinder-api để gán volume vào máy ảo.

17. Nova-compute sẽ generate dữ liệu cho Hypervisor và gửi thông tin thông qua libvirt.


## 5. Spawning VMs workflow

1. Dùng câu lệnh `nova-boot` hoặc tạo máy ảo trên dashboard

2. Quy trình tạo máy ảo: API -> Scheduler -> Compute (manager) -> Libvirt Driver

3. Tạo file disk: Lấy image từ glance vào thư mục  instance_dir/_ base và converrt nó sang dạng RAW (nếu đã ở dạng raw thì không cần convert)
-> Tạo instance_dir/uuid/{disk, disk.local, disk.swap}:
    -> Tạo `QCOW2` "disk" file, với backing file từ `_base image`
    -> Tạo QCOW2 “disk.local” và “disk.swap”
4. Tạo file libvirt XML và sao chép nó vào instance_dir (dù file này không được sử dụng bởi Nova)

5. Thiết lập kết nối với volume (nếu bạn chọn option boot từ volume). Các câu lệnh thực thi tiếp theo còn tùy thuộc vào volume driver:
  - Đối với iSCSI: Kết nối thông qua tgt hoặc iscsiadm
  - Đối với RBD: Tạo XML cho Libvirt, phần còn lại được thực hiện bởi QEMU.

6. Tạo network cho máy ảo

- Tùy thuộc vào các driver mà người dùng khai báo, các câu lệnh sẽ được thực hiện theo.
- Bật tất cả các bridges/VLANs cần thiết.
- Tạo security groups (iptables)

7. Difine domain với Libvirt, sử dụng file XML đã được tạo từ trước. Tương đương với câu lệnh `virsh define instance_dir/<uuid>/libvirt.xml’`

8. Giờ đây máy ảo đã được start. Tương đương với câu lệnh `virsh start`


## 6. Hard reboot workflow

1. Destroy domain (chỉ hủy đi tiến trình, không hủy dữ liệu).

2. Thiết lập lại các kết nối volume.

3. Tạo lại file Libvirt XML.

4. Kiểm tra và tải lại các backing files nếu thiếu.

5. Gán lại các network stack.

6. Tạo lại các iptables rules.

