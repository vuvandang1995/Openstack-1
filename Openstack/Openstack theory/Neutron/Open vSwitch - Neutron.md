# Phần này sẽ giới thiệu về các kiến trúc và thành phần để xây dựng network trong neutron


Mô hình kiến trúc network - provider sử dụng openvswitch

<img src="https://i.imgur.com/PbYvxnH.png">

Gói tin sẽ bắt đầu từ card eth0 trên máy ảo, nó được kết nối với tap device trên host. tap device này ở trên Linux bridge. Sở dĩ có 1 layer Linux Bridge là vì OpenStack sử dụng các rules của iptables trên tap devices để thực hiện security groups. Trước đây khi mà OpenvSwitch chưa hỗ trợ iptables thì nó buộc dùng thêm một layer là Linux Bridge nữa dù lí tưởng nhất vẫn là interface của VM gán trực tiếp vào bridge intergration (br-int) của OVS.

Ở các phiên bản gần đây, OpenvSwitch đã hỗ trợ iptables vì thế nó sẽ không cần thêm một layer của LB nữa. Bạn có thể xem thêm về cách khai báo cho OVS sử dụng iptables 

**Linux bridge**

Trên đây chứa các rules dùng cho security group. Nó có 1 đầu là tap interface có địa chỉ MAC trùng với địa chỉ MAC của card mạng trên máy ảo và một đầu là qvb... được nối với qvo... trên integration bridge.

**Integration Bridge**

Bridge này thường có tên là br-int thường được dùng để "tag" và "untag" VLAN cho traffic vào và ra VM
