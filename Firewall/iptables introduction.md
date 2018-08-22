# Tìm hiểu về iptables
-----------

## 1. Định nghĩa

**Iptables** là firewall software cơ bản được dùng nhiều nhất trong Linux, được dùng để tạo tường lửa cho máy linux của bạn, nó có các chức năng lọc gói tin, nat gói tin qua đó để giúp làm nhiệm vụ bảo mật thông tin cá nhân tránh mất mát thông tin và áp dụng nhưng chính sách đổi với người sử dụng. Iptables hoạt động bằng cách giao tiếp với packet filtering hooks trong Linux kernel's networking stack. Các hooks này là netfilter framework.

**Netfilter** là một frame bên trong nhân linux giúp cung cấp sự hoạt động mạng khác nhau để phù hợp với các cách xử lý khác nhau . Netfilter có các lựa chọn như: Packet filtering, network address translation, port translation. Các phương thức của Netfilter sẽ cung cấp các hướng dẫn cho các packets đi qua trong mạng cũng như việc tiến hành ngăn chặn các packets trong phạm vi dễ bị ảnh hưởng trong mạng máy tính. Netfilter có 3 module chính: iptables, nat , connection tracking.

- Iptables đơn giản chỉ là một danh sách các rules được tổ chức theo dạng bảng. Mỗi một rule chứa một loạt các classifiers (iptables matches) và một connected action (iptables target).

**Các tính năng**:
- Stateful packet filtering là thành phần kiểm tra trạng thái thông tin các giao thức như TCP/UDP/ICMP tại layer 4( Transport) và các tầng thấp hơn của OSI. Các gói tin được kiểm tra, nếu gói tin được kiểm tra đúng theo luật trong firewal, gói tin sẽ được cho phép đi qua và thông tin đó sẽ được đưa vào một state table (State table: la tính năng của stateful firewal, nó tự động lưu trũ thông tin về các kết nối đang hoạt động được cho phép, tạo bởi các luật) 

- Stateless packet filtering Là thành phần không theo dõi trạng thái của các kết nối mà chỉ dựa vào địa chỉ nguồn, địa chỉ đích, một số thông tin trên header để quyết định cho phép hay không cho phép gói tin đi qua

-NAT in và Nat out

**So sánh iptables và firewalld**:

<img src="http://i.imgur.com/Pu5sOu7.png">
- `iptables` lưu cấu hình tại /etc/sysconfig/iptables và /etc/sysconfig/ip6tables thì `firewalld` lại lưu nó dưới dạng một loạt các file XML trong /usr/lib/firewalld/ và /usr/lib/firewalld/

- Sử dụng `iptables` , khi restart lại hệ thống các dịch vụ chạy trên hệ thống sẽ bị ngưng một lúc do mỗi một thay đổi đồng nghĩa với việc hủy bỏ toàn bộ các rules cũ và load lại một loạt các rules mới trong file /etc/sysconfig/iptables, còn đối với `firewalld` khi restart hệ thống thì các ứng dụng sẽ không bị dừng do chỉ những thay đổi mới được applied

- Theo mặc định thì hệ thống sẽ sử dụng firewalld , nếu muốn sử dụng iptables thì phải tắt firewalld và cài iptables

--------------
**Chú ý**:

- Đối với CentOS/RHEL 7, khi bạn tắt firewalld (mặc định) hoặc tắt iptables service. Các iptables rules cũng sẽ biến mất -> Một số service hoạt động dựa trên nó như network default của KVM (LB) cũng sẽ bị ảnh hưởng.

- Đối với Ubuntu/Debian, ufw là firewall mặc định. Tuy nhiên khi disable ufw, các iptables rules không bị mất đi. Mặc dù vậy, để có thể lưu lại các iptables rules đã cấu hình, bạn cần cài thêm gói iptables-persistent

## 2. Cơ chế

- iptables định nghĩa `chain` chứa 5 "hook points" trong quá trình xử lí gói tin của kernel: PREROUTING, POSTROUTING, FORWARD, INPUT và OUTPUT. (không nhất thiết 1 chain phải có 5 hook point hoạt động đầy đủ).

<img src="http://i.imgur.com/wpfICOs.png">

| Hook | Cho phép bạn xử lí các packet ...|
|------|------------------------------|
| FORWARD | Có đích là một server khác nhưng không được tạo từ server của bạn. Chain này là cách cơ bản để cấu hình server của bạn để route các request tới một thiết bị khác |
| INPUT |  lọc các gói tin đầu vào |
| OUTPUT | Lọc các gói tin đầu ra |
| PREROUTING | Nó sẽ được thực thi trước khi quá trình routing diễn ra, khi từ ngoài vào mạng local ip sẽ được firewall chuyển đổi thành 1 địa chỉ local, thường dùng cho DNAT (destination NAT),  |
| POSTROUTING | đi ra ngoài hoặc sau khi  được định tuyến, khi ra ngoài ra khỏi firewall thì địa chỉ local sẽ đươc chuyển thành 1 địa chỉ khác, thường dùng cho SNAT (source NAT) |

### Iptables có 3 bảng chính: `filter`, `mangle`, và `nat`

**Bảng mangle: bảng này để thay đổi QoS của gói tin như thay đôi các tham số TOS,TTL,..Trong bảng mangle bao gồm tất cả các chain được xây dựng sẵn là: PREROUTING, POSTROUTING, INPUT, OUTPUT, FORWARD**

**Bảng NAT: dùng để thay đổi địa chỉ đích,nguồn, port đích, nguồn của gói tin. Bảng NAT có 3 chains PREROUTING, POSROUTING, OUTPUT**

- Chain PREROUTING : đấy là chain dùng để thay đổi địa chỉ đích của gói tin và trong chain FREROUTING target(tác vụ) được sử dụng là DNAT (destination NAT): VD: `# iptables -t NAT -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.0.30.100:80` Câu lệnh này có ý nghĩa. Đối với gói tin tcp có port đích là 80 thì sẽ được đổi địa chỉ đích thành 10.0.30.100 ví dụ trên đã định nghĩa rõ cho chúng ta:nói ra chọn bảng NAT áp dụng rule và trong chain PREROUTING sử dụng target DNAT

- Chain POSROUTING : đây là chain dùng để thay đổi địa chỉ nguồn của gói tin và targer được sử dụng là SNAT (source NAT) VD: `#iptables -t NAT -A POSTROUTING -p tcp -j SNAT --to-source 10.0.30.200` Câu lệnh này có ý nghĩa đổi địa chỉ nguồn đối với gói tin tcp thành 10.0.30.200

- Chain OUTPUT : Sử dụng cho các gói dữ liệu xuất phát từ tường lửa

**Bảng filter: áp dụng các chính sách lọc gói tin qua đó cho phép gói tin được qua hay không qua**

Trong bảng filter có các chain được xây dựng sẵn:

- chain INPUT: đây là chain dùng để lọc các gói tin đầu vào. Chain này là chain các gói tin bắt buộc phải đi qua để có thể được xử lý.

- Chain OUTPUT: đây là chain dùng để lọc các gói tin đâu ra. Chain này là chain sau khi gói tin được xử lý phải đi qua chain này để ra được bên ngoài.

- Chain FORWARD : đây là chain dùng để chuyển gói tin qua lại giữa các card mạng với nhau.

- Các target được sử dụng trong các chain này có thể là ACCEPT (đồng ý) , DROP (xóa bỏ)





**Target**: Cả hai yếu tố của rules đó là match và target đều là tùy chọn. Như vậy, cấu trúc của iptables như sau: iptables -> Tables -> Chains -> Rules

**Rules**: Mặc định thì mỗi table đều có chains trống. Mỗi chain sẽ có policy, policy này sẽ quyết định trạng thái của gói tin sẽ được chấp nhận nếu match với policy hoặc hủy bỏ khi không match với policy. Policy chỉ có 2 target là ACCEPT và DROP, mặc định là ACCEPT. Các chain được tạo bởi user sẽ có policy mặc định và không thay đổi được có target là RETURN

Mỗi tables có nhiều chains, mỗi chain có nhiều rules
<img src="http://i.imgur.com/POf5Fvx.png">

**Khi một packet match với các rules, target được thực thi. Target có thể là quyết định cuối cùng áp dụng đối với packet ví dụ như ACCEPT hoặc DROP. Nó cũng có thể chuyển packet tới chain khác để xử lí**

- Các rules này được gộp lại thành nhóm gọi là chains, khi một packet match với 1 rules, nó sẽ được thực thi với hành động tương ứng và không cần phải check với các rules còn lại.

- Mỗi chain có thể có một hoặc nhiều rule nhưng mặc định nó sẽ có 1 policy. Trong trường hợp packets không match với bất cứ rules nào, policy sẽ được thực thi, bạn có thể accept hoặc drop tùy vào người quản trị thiết lập

3. State machine
- Với iptables, có các trạng thái của các kết nối đó là: NEW, ESTABLISHED, RELATED, INVALID, UNTRACKED

**Iptables** sử dụng một framework trong kernel có tên gọi là conntrack, nó được load như là 1 phần của kernel. Tất cả các giám sát kết nối đều được thực hiện ở chain PREROUTING, những packet từ local đi ra thì được kiểm soát bởi chain OUTPUT. Ví dụ ta có 1 gói tin gửi đi, nó sẽ có trạng thái NEW ở chain OUTPUT, khi nó được phản hồi về, trạng thái của nó ở chain PREROUTING sẽ là ESTABLISHED.

- File `/proc/net/nf_conntrack` chứa toàn bộ những entries trong conntrack database.

| State | Explaination |
|-------|--------------|
| NEW	| Trạng thái này cho ta biết đó là packet đầu tiên mà conntrack module thấy (những packet có cờ SYN) |
| ESTABLISHED | Điều kiện để có trạng thái này đơn giản là 1 host gửi packet đi và nhận lại reply từ host khác |
| RELATED | Kết nối ở trạng thái này khi nó liên quan tới kết nối khác ở trạng thái ESTABLISHED. Đầu tiên ta có 1 kết nối đã ESTABLISHED, sau đó kết nối này tiếp tục tạo ra một kết nối khác ra bên ngoài kết nối chính. Kết nối mới này được coi là RELATED |
| INVALID | Có nghĩa rằng packet không thể được xác nhận hoặc nó không có bất cứ trạng thái nào, thông thường những paket như này sẽ bị drop |
| UNTRACKED | Đây là những packet được đánh dấu trong bảng raw với target là NOTRACK. Sau đó nó sẽ được đánh dấu state là UNTRACKED |

------

## Cài đặt iptables

- Cài đặt packages

`yum install -y iptables-services`

- Disable Firewalld service

`systemctl mask firewalld`

- Cho phép iptables service khởi động cùng hệ thống đồng thời cũng phải disable firewalld đi thì mới được

`systemctl enable iptables`  

`systemctl disable firewalld`

- Stop Firewalld service

`systemctl stop firewalld`

- Bật iptables service

`systemctl start iptables`

- Lưu cấu hình rules iptables

`service iptables save`
