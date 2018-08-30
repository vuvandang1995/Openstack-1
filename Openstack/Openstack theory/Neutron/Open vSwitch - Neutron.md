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

**Tunnel Bridge**

Traffic tới từ node compute sẽ được chuyển tới node controller thông qua GRE/VXLAN tunnel trên bridge tunnel (br-tun). Nếu sử dụng VLAN thì nó sẽ chuyển đổi VLAN-tagged traffic từ intergration bridge sang GRE/VXLAN tunnels. Việc chuyển đỏi qua lại giữa VLAN IDs và tunnel IDs được thực hiện bởi OpenFlow rules trên br-tun.

Đối với GRE hoặc VXLAN-based network truyền tới từ patch-tun của br-int sang patch-int của br-tun. Bridge này sẽ chứa 1 port cho tunnel có tên "vxlan-..."

Tunnel này sử dụng bảng định tuyến trên host để trao đổi gói tin vì thế ko cần bất cứ yêu cầu nào đối với hai endpoints hai bên, khác với khi sử dụng VLAN. Tất cả các interface trên br-tun được coi là internal đối với Open vSwitch. Vì thế nó sẽ không thể nhìn thấy từ bên ngoài, để giám sát traffic, bạn phải tạo ra mirror port cho patch-tun trên br-int bridge.

**DHCP Router on Controller**

DHCP server thường được chạy trên node controller hoặc compute. Nó là một instance của dnsmasq chạy trong một network namespace. Network namespace là một Linux kernel facility cho phép thực hiện một loạt các tiến trình tạo ra network stack (interfaces, routing tables, iptables rules).

**Router on Controller**

Router cũng là một network namespace với một loạt các routing rules và iptables rules để thực hiện việc định tuyến giữa các subnets.
Chúng ta cũng có thể xem cấu hình của router với câu lệnh ip netns exec. Mỗi router sẽ có 2 interface, một cổng sẽ kết nối tới gateway (được tạo bởi câu lệnh router-gateway-set), một cổng sẽ nối tới integration bridge.

## 3. Traffic flow

### 3.1. OpenvSwitch

#### 3.1.1 Provider
 
#### - Trường hợp cùng subnet, cùng giải mạng
 
**North-South**:

<img src="https://i.imgur.com/Y4tMrF8.png">

- Instance forward packet tới security group bridge `port tap` (2) thông qua đường veth 
- Các Security group rules trên security group bridge (3) sẽ sử dụng firewall và kiểm tra kết nối các packet 
- Security group bridge OVS port (4) forward packet tới OVS integration bridge `security group port` (5) qua đường veth
- OVS integration bridge gắn 1 vlan internal cho packet 
- OVS integration bridge `br-int` (6) forward packet trên đường internal vlan tới OVS provider bridge `br-provider`
- OVS provider bridge đổi tag internal vlan thành vlan 101 là vlan cấu hình với switch bên ngoài
- OVS provider bridge `provider network port` (8) forward packet tới physical network interface
- Physical network interface forward packet tới switch (10)
- Switch bỏ tag vlan 101 và forward tới router 
- Router định tuyến packet từ provider network (12) tới external network (13) và forward packet tới switch (14)
- Switch forward packet tới external network (15)
- External network (16) nhận packet

**East/West**

<img src="https://i.imgur.com/fs6lFRC.png">

- Instance 1 (1) forward packet tới security group `instance bridge port` (2) qua đường veth
- Security group rules (3) trên security group bridge sẽ lọc packet qua firewall và kiểm tra kết nối cho packet
- Security group bridge OVS port (4) forward packet tới OVS intergratetion bridge `security group port` (5) qua đường veth
- OVS intergration bridge sẽ tag 1 internal vlan cho packet
- OVS intergration `bridge int-br-provider` (6) sẽ forward packet tới OVS provider bridge `phy-br-provider` (7)
- OVS provider bridge đổi tag internal vlan thành vlan 101 là vlan cấu hình vơi switch bên ngoài
- OVS provider bridge provider network port (8) forward packet tới physical network interface (9)
- Physical network interface forward packet tới switch (10) trong hạ tầng physical network
- Switch forward packet từ compute 1 tới compute2 (11) qua đường vlan 101
- physical network interface (12) forward packet tới OVS provider bridge `provider network port` (13)
- OVS provider bridge phy-br-provider (14) forward packet tới OVS integration bridge `int-br-provider` (15)
- OVS integration bridge đổi tag vlan 101 thành internal vlan
- OVS integration bridge `security group port` (16) forward packet tới `security group` bridge OVS port (17)
- Security group rules (18) trên security group bridge sẽ lọc packet qua firewall và kiểm tra kết nối cho packet 
- Security group bridge instance port  (19) forward packet tới instance 2 interface qua đường veth

### - Trường hợp khác subnet, khác dải mạng

 <img src="https://i.imgur.com/aLwyurf.png">
 
- Instance 1 forward packet tới security group bridge instance port (2) trên linux bridge thông qua đường veth
- Security group rules (3) trên security group lọc packet qua firewall và kiểm tra kết nối cho packet
- Security group bridge OVS port (4) forward packet tới OVS integration bridge `security group port` (5) qua đường veth
- OVS integration bridge tag 1 internal vlan cho packet
- OVS integration bridge `int-br-provider` (6) forward packet tới OVS provider bridge `phy-br-provider` (7)
- OVS provider bridge đổi tag internal vlan thành vlan 101 là vlan cấu hình trên physical switch
- OVS provider bridge `provider network port` (8) forward packet tới physical network interface (9)
- Physical network interface forward packet tới switch (10) trên hạ tầng physical network
- Trên hạ tầng physical network, switch sẽ untag vlan 101 từ packet và forward packet tới router (11)
- Router sẽ định tuyến packet từ provider network 1 sang provider network 2 (13)
- Router forward packet tới switch (14)
- Switch sẽ tag vlan 102 tương ứng với vlan được cấu hình trên physical network 2 nơi có subnet cùng với network của instance 2
- Physical network interface (16) forward packet tới OVS provider bridge `provider network port` (17)
- OVS provider bridge `phy-br-provider` (18) forward packet tới OVS integration bridge `int-br-provider` (19)
- OVS integration bridge đổi tag vlan 102 thành internal vlan
- OVS integration bridge security group port (20) untag internal vlan và forward packet tới security group bridge OVS port (21)
- Security group rules (22) trên security group bridge sẽ lọc packet bằng firewall và kiểm tra kết nối cho packet
- Security group bridge instance port (23) forward packet tới instance 2 interface bằng đường veth

#### 3.1.2 Self-service

**North/South with a fixed IP**

<img src="http://i.imgur.com/jYh6Pph.png">

- instance interface (1) forward packet đến security group bridge instance port (2) bằng đường veth
- Security group rules (3) trên security group bridge lọc packet bằng firewall và kiểm tra kết nối cho packet
- Security group bridge OVS port (4) forward packet tới OVS integration bridge security group port (5) bằng đường veth
- OVS integration bridge tag vlan internal cho packet
- OVS integration bridge untag internal vlan, và tag 1 internal tunnel ID
- OVS integration bridge (6) forward packet tới OVS tunnel bridge (7)
- OVS tunnel bridge (8) đóng gói packet sử dụng VNI 101 (VXLAN Network Identifier) 
- overlay network đưới lớp physical network forward packet tới network node thông qua đường overlay network (10)
- Overlay network (11) forward packet tới OVS tunnel bridge (12)
- OVS tunnel bridge gỡ VNI 101 và thêm vào packet 1 internal tunnel ID
- OVS tunnel bridge patch port (13) forward packet tới OVS integration bridge patch port (14)
- OVS integration bridge port của self-service (15) untag internal vlan và forward packet tới self-service network trên router namespace
- router dùng SNAT packet thay đổi source IP packet thành IP router trên provider network và gửi nó tới gateway của provider network (17)
- router forward packet tới OVS integration bridge port bằng đường provider network (18)
- OVS integration bridge tag 1 internal vlan cho packet
- OVS integration bridge `int-br-provider` (19) forward packet tới OVS provider bridge `phy-br-provide` (20)
- OVS provider bridge untag internal vlan và tag vlan 101 là vlan được cấu hình trên hạ tâng network physical
- OVS provider bridge provider network port (21) forward packet tới physical network (22)
- physical network forward packet ra internet bằng hạ tầng physical network

**North/South with a floating IPv4 address**

<img src="http://i.imgur.com/hPYriI7.png">

**East/West: Instance on the same network**

<img src="http://i.imgur.com/EFLr4D2.png">

**East/West: Instance on different network**

<img src="http://i.imgur.com/RPUN0Ii.png">
