# Phần này sẽ giới thiệu về các kiến trúc và thành phần để xây dựng network trong neutron

## 1. Provider

Mô hình kiến trúc network - provider sử dụng openvswitch

<img src="https://i.imgur.com/PbYvxnH.png">

Gói tin sẽ bắt đầu từ card eth0 trên máy ảo, nó được kết nối với Port tap device trên host. tap device này ở trên Linux bridge. Sở dĩ có 1 layer Linux Bridge là vì OpenStack sử dụng các rules của iptables trên tap devices để thực hiện security groups. Trước đây khi mà OpenvSwitch chưa hỗ trợ iptables thì nó buộc dùng thêm một layer là Linux Bridge nữa dù lí tưởng nhất vẫn là interface của VM gán trực tiếp vào bridge intergration (br-int) của OVS.

Ở các phiên bản gần đây, OpenvSwitch đã hỗ trợ iptables vì thế nó sẽ không cần thêm một layer của LB nữa. Bạn có thể xem thêm về cách khai báo cho OVS sử dụng iptables 

**Linux bridge**

Trên đây chứa các rules dùng cho security group. Nó có 1 đầu là tap interface có địa chỉ MAC trùng với địa chỉ MAC của card mạng trên máy ảo và một đầu là qvb... được nối với qvo... trên integration bridge.

**Integration Bridge**

Bridge này thường có tên là br-int thường được dùng để "tag" và "untag" VLAN cho traffic vào và ra VM.

Port qvo... là điểm đến của qvb... và nó sẽ chịu trách nhiệm với các traffic vào và ra khỏi layer LB. Nếu bạn sử dụng VLAN, sẽ có thêm tag:1. Interface int-br-provider kết nối tới phy-br-provider trên layer bridge external.

**External bridge**

Bridge này sẽ được gán với interface external để đi ra ngoài internet (có thể dùng bond hoặc không).

## 2. Self-service

<img src="https://i.imgur.com/WX9u8Qs.png">

Tunnel Bridge

Traffic tới từ node compute sẽ được chuyển tới node controller thông qua GRE/VXLAN tunnel trên bridge tunnel (br-tun). Nếu sử dụng VLAN thì nó sẽ chuyển đổi VLAN-tagged traffic từ intergration bridge sang GRE/VXLAN tunnels. Việc chuyển đỏi qua lại giữa VLAN IDs và tunnel IDs được thực hiện bởi OpenFlow rules trên br-tun.

Đối với GRE hoặc VXLAN-based network truyền tới từ patch-tun của br-int sang patch-int của br-tun. Bridge này sẽ chứa 1 port cho tunnel có tên "vxlan-..."

Tunnel này sử dụng bảng định tuyến trên host để trao đổi gói tin vì thế ko cần bất cứ yêu cầu nào đối với hai endpoints hai bên, khác với khi sử dụng VLAN. Tất cả các interface trên br-tun được coi là internal đối với Open vSwitch. Vì thế nó sẽ không thể nhìn thấy từ bên ngoài, để giám sát traffic, bạn phải tạo ra mirror port cho patch-tun trên br-int bridge.
