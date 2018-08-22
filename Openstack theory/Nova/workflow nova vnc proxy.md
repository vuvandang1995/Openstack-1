# Phần này giới thiệu về quá trình kết nối tới máy ảo bằng vnc

<img src="https://i.imgur.com/pAYt8vp.png">

### Tính năng VNC Proxy

- Tạo ra 1 vùng mạng private và cô lập public network.
- VNC Client chạy trên mạng public, VNC Server chạy trên mạng private, VNC Proxy giống như 1 cầu nối kết nối giữa 2 đối tượng với nhau.
- VNC Proxy xác thực VNC Client bằng token.
- VNC Proxy không chỉ cho phép truy cập bảo mật hơn bằng cách tạo ra mạng private mà còn cho phép tách riêng việc triển khai VNC Server, nó có thể hỗ trợ Hypervisor VNC Server khác nhau nhưng không ảnh hưởng đến trải nghiệm của người dùng.

### Triển khai VNC Proxy

- Triển khai nova-consoleauth trên controller cho xác minh token.
- Nova-novncproxy service được triển khai trên controller và các người dùng VNC Client sẽ connect trực tiếp tới dịch vụ (máy ảo).
- Trên controller thường có 2 card mạng để sử dụng cho việc kết nối tới 2 mạng, 1 trong số đó là cho truy cập ra mạng bên ngoài (public network hoặc API network), địa chỉ ip của card mạng này là ip external network. Một card mạng được sử dụng cho giao tiếp giữa các compute, controller và các project trong openstack dải mạng nó gọi mà management network, nó sử dụng như 1 mạng nội bộ.
- Triển khai nova-compute trên compute cần cấu hình trên file nova.conf
<ul>
  <ul>
    <li>vnc_enabled=True</li>
    <li>Vncserver_listen=0.0.0.0 //The listening address of the VNC Server</li>
    <li>Vncserver_proxyclient_address=10.10.10.2 //nova vnc proxy accesses the vnc server through the intranet IP, so nova-compute will tell vnc proxy to use this IP to connect to me</li>
    <li>Novncproxy_base_url=http://172.24.1.1:6080/vnc_auto.html (url này được đưa tới client, ip ở đây là ip của external network)</li>
  </ul>
</ul>

### Quá trình chạy VNC Proxy

- Đầu tiên 1 user thử mở 1 phiên VNC client kết nối tớ 1 máy ảo từ 1 trình duyệt.
- Trình duyệt gửi 1 yêu cầu tới nova-api và yêu cầu sẽ được trả lại url để truy cập vnc.
- Nova-api gọi tới nova-comute để lấy thông tin về kết nối tới vnc console.
- Nova-Compute gọi tới libvirt để lấy chức năng của vnc console.
- Libvirt sẽ lấy thông tin VNC Server bằng việc phân tích file /etc/libvirt/qemu/instance-0000000c.xml, file này được chạy trên máy ảo.
- Libvirt sẽ trả về host, port, và thông tin khác tới nova-compute trong file định dạng .json.
- Nova-compute sẽ tạo ra ngẫu nhiên 1 UUID token.
- Nova-compute kết hợp với thông tin trả về từ libvirt và token để cấu hình vào trong file connect_info rồi gửi tới nova-api.
- Nova-api sẽ gọi chức năng authorize_console của nova-consoleauth.
- Nova-consoleauth lưu thông tin của instance vào token, và đưa token vào trong file connect_info
- Nova-api trả về thông tin truy cập url trong file connect_info tới web http://172.24.1.1:6080/vnc_auto.html?token=7efaee3f-eada-4731-a87c-e173cbd25e98&title=helloworld(9169fdb2-5b74-46b1-9803 -60d2926bd97c), nhưng tiêu đề của console sẽ trở thành novnc thay vì tên và ID của máy ảo.
- Web sẽ thử mở đường dẫn này.
- Đường dẫn này sẽ gửi request tới nova-novncproxy.
- Nova-novncproxy gọi tới hàm check_token của nova-consoleauth.
- Nova-consoleauth sẽ xác thực token và trả về connect_info tương ứng tới instance này cho nova-novncproxy.
- Nova-novncproxy khởi động proxy bằng kết nối VNC Server trên compute với host, port và các thông tin khác trong file connect_info.
