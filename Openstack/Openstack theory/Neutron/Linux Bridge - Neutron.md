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

**East-west* khác network (subnet)

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
