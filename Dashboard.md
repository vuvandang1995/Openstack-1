# Phần này sẽ giới thiệu cách sử dụng dashboard cho việc tạo máy ảo


**Đầu tiên muốn tạo máy ảo cần tạo 1 image trước**

<img src="https://i.imgur.com/B47PNeI.png">

- Click vào mục Project -> Compute -> Images

<img src="https://i.imgur.com/uQ9rJHv.png">

- Tạo image

<img src="https://i.imgur.com/ZGjyFQg.png">

**Tiếp theo tạo card mạng để sử dụng khi tạo máy ảo: click Network -> Networks -> Create Network**

<img src="https://i.imgur.com/toZ80vn.png">

<img src="https://i.imgur.com/z6yzrYr.png">

<img src="https://i.imgur.com/s1jPfUG.png">

- Nếu muốn tạo card external ta phải tạo ở mục admin đây là mục dành riêng cho admin quản lý

<img src="https://i.imgur.com/qdkLss9.png">

kiểm tra bằng lệnh `less /etc/neutron/plugins/ml2/openvswitch_agent.ini | egrep -v '^#|^$'`, ở đây kêt quả là **extnet** 

<img src="https://i.imgur.com/nCooNoZ.png">

**Tạo máy ảo**



<img src="https://i.imgur.com/t0oLLYT.png">



<img src="https://i.imgur.com/TfWgzLU.png">



<img src="https://i.imgur.com/83WCdvm.png">



<img src="https://i.imgur.com/iofH3Fq.png">



<img src="https://i.imgur.com/s7CffN1.png">

----

- Phân quyền tren project: Với mỗi project sẽ được coi như 1 vùng quản lý riêng biệt và sẽ gán quyền cho các user có thể thao tác được gì trên project đó

ví dụ ở đây là project ducnm37

<img src="https://i.imgur.com/x9flMfW.png">

ở đây gán quyền member cho user ducnm372 và quyền admin cho user admin trên project ducnm37

<img src="https://i.imgur.com/zd3WJhE.png">

