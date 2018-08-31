# Phần này sẽ giới thiệu về các thành phần kiến trúc xây dựng network sử dụng linux bridge trong neutron

## 1. Provider

Mô hình kiến trúc network - provider

<img src="https://i.imgur.com/4muye9u.png">

**Linux bridge**

Trên đây chứa các rules dùng cho security group. Nó có 1 đầu là `tap` interface có địa chỉ MAC trùng với địa chỉ MAC của card mạng trên máy ảo và một đầu là `port interface 2` được nối với `interface 2` là physical network interface và được nối đến hạ tầng physical network.

## 2. Self-service

Mô hình kiến trúc network - self-service 

<img src="https://i.imgur.com/ydsas3H.png">

## 3. Network workflow

### 3.1 Provider

**North-south with a fixed IP address**

<img src="https://i.imgur.com/xZq92ub.png">

- instance interface (1) forward packet tới provider bridge instance interface (2) qua đường veth
- Security group rules (3) trên provider bridge lọc packet và kiểm tra kết nối cho packet
- Vlan sub-interface port (4) trên provider bridge forward packet tới physical network interface (5)
- Physical network interface (5) tag vlan 101 cho packet và forward packet tới switch trên hạ tầng physical network
- Switch untag vlan 101 và forward tới router (7)
- router định tuyến packet từ provider network (8) tới external network (9) và forward packet tới switch (10)
- switch forward packet tới external network (11)
- external network (12) tiếp nhận packet

**East-west** cùng network (subnet)

<img src="https://i.imgur.com/4z31ua8.png">

- instance 1 interface (1) forward packet tới provider bridge instance port (2) qua đường veth
- Security group rules (3) trên provider bridge lọc packet và kiểm tra kết nối cho packet
- Vlan sub-interface port (4) trên provider bridge forward packet tới physical network interface (5)
- Physical network interface (5) tag vlan 101 cho packet và forward packet tới switch (6) trên hạ tầng physical network
- Switch forward packet từ compute 1 tới compute 2 (7)
- Physical network interface (8) untag vlan 101 từ packet và forward tới Vlan sub-interface port (9) trên provider bridge
- Security group rules (10) trên provider bridge lọc packet bằng firewall và kiểm tra kết nối cho packet 
- provider bridge instance port (11) forward packet tới instance 2 (12) qua đường veth

**East-west** khác network (subnet)

<img src="https://i.imgur.com/ELdoso2.png">

- instance 1 interface (1) forward packet tới provider bridge instance port (2) qua đường veth
- Security group rules (3) trên provider bridge lọc gói tin qua firewall và kiểm tra kết nối cho packet
- Vlan sub-interface port (4) trên provider bridge forward packet tới physical network interface (5)
- Physical network interface (5) tag vlan 101 cho packet và forward tới switch (6) trên hạ tầng physical network 
- switch untag vlan 101 và forward packet tới router (7)
- router định tuyến packet từ provider network 1 (8) tới provider network 2 (9)
- router forward packet tới switch (10)
- switch tag vlan 102 tới packet và forward tới compute 1 (11)
- physical network interface (12) untag vlan 102 từ packet và forward tới Vlan sub-interface port (13) trên provider bridge
- Security group rules (14) trên provider bridge lọc packet qua firewall và kiểm tra kết nối cho packet
- provider bridge instance port (15) forward packet tới instance 2 interface (16) qua đường veth

### 3.2 Self-service

**North-south** fixed IP

<img src="https://i.imgur.com/VyQ6Oy6.png">

- instance interface (1) forward packet tới self-service bridge instance port (2) qua đường veth
- Security group rules (3) trên self-service bridge sẽ lọc gói tin bằng firewall và kiểm tra kết nối cho packet
- self-service bridge forward packet tới VXLAN interface (4) và đóng gói packet sử dụng VNI 101
- Packet được truyền bởi VXLAN interface (5) dưới lớp giao diện vật lý forward packet tới network node bằng đường overlay network (6)
- VXLAN interface (7) forward packet tới  VXLAN interface (8) và được bóc lớp VNI 101 đi chỉ còn lại packet 
- Self-service bridge router port (9) forward packet tới self-service network interface (10) trên router
- Router sử dụng SNAT thay đổi source IP của gói tin bằng ip provider network sau đó gửi tới gateway provider network (11)
- Router forward packet tới provider bridge router port (12)
- VLAN sub-interface port (13) provider bridge forward packet tới provider physical network interface (14)
- Provider physical network interface (14) tag vlan 101 cho packet và forward ra Internet thông qua hạ tầng physical network (15)

**North-south** floating IPv4

<img src="https://i.imgur.com/LofAl9a.png">

- Hạ tầng physical network forward packet tới provider physical network interface (2)
- Provider physical network interface bỏ tag vlan 101 và forward packet tới VLAN sub-interface trên provider bridge
- Provider bridge forward packet self-service router gateway port trên provider network (5)
- Router sử dụng DNAT thay đổi địa chỉ đích của packet thành IP của dải self-service network và gửi packet tới gateway của self-service network bằng self-service interface (6)
- Router forward packet tới self-service bridge router port (7)
- Self-service bridge forward packet tới VXLAN interface (8) và đóng gói packet sử dụng using VNI 101
- VXLAN interface (9) truyền packet tới network node bằng overlay network (10)
- VXLAN interface (11) forward packet tới VXLAN interface (12) và bóc lớp VNI 101 đi chỉ còn lại packet
- Security group rules (13) trên self-service bridge lọc gói tin bằng firewall và kiểm tra kết nối cho packet
- Self-service bridge instance port (14) forward packet tới instance interface (15) thông qua đường veth

**East-west** cùng network (subnet)

<img src="https://i.imgur.com/LofAl9a.png">

- Instance 1 interface (1) forward packet tới self-service bridge instance port (2) theo đường veth
- Security group rules (3) trên self-service bridge lọc packet bằng firewall và kiểm tra kết nối cho packet
- Self-service bridge forward packet tới VXLAN interface (4) và đóng gói packet sử dụng VNI 101
- VXLAN interface (5) forward packet tới compute 2 bằng đường overlay network (6)
- VXLAN interface (7) forward packet tới VXLAN interface (8) và bóc lớp VNI 101 chỉ còn lại packet
- Security group rules (9) trên self-service bridge lọc packet bằng firewall và kiểm tra kết nối cho packet
- Self-service bridge instance port (10) forward packet tới instance 1 interface (11) thông qua đường veth

**East-west** khác network (subnet)

<img src="https://i.imgur.com/0PEtMbl.png">

- Instance 1 interface (1) forward packet tới self-service bridge instance port (2) thông qua đường veth
- Security group rules (3) trên self-service bridge lọc packet và kiểm tra kết nối cho packet
- Self-service bridge forward packet tới VXLAN interface (4) và đóng gói packet sử dụng VNI 101
- VXLAN interface (5) forward packet tới network node bằng đường overlay network (6)
- VXLAN interface (7) foward packet tới VXLAN interface (8) và bóc lớp VNI 101 khỏi packet
- Self-service bridge router port (9) forward packet tới self-service network 1 interface (10) trong router namespace
- Router gửi packet tới next-hop IP là gateway của self-service network 2 thông qua đường self-service network 2 interface (11)
- Router forward packet tới self-service network 2 bridge router port (12)
- Self-service network 2 bridge forward packet tới VXLAN interface (13) và đóng gói sử dụng VNI 102 
- VXLAN interface (14) gửi packet tới compute node thông qua đường overlay network (15)
- VXLAN interface (16) gửi packet tới VXLAN interface (17) 
- Security group rules (18)
- Self-service bridge instance port (19) forward packet tới instance 2 interface (20) thông qua đường veth
