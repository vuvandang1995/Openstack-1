# Test kịch bản cho iptables



<img src="https://i.imgur.com/XvfRH3j.png">

- Tất cả các ip trong mạng ping được đến nhau

- ip 15.0.0.3 ping được ra internet 

- ip 20.0.0.3 không ping được ra internet

iptables khi chưa có gì

<img src="https://i.imgur.com/wqp7GZg.png">


- Tạo kết nối từ ip 15.0.0.3 đên ip 20.0.0.3

#### Trên máy 15.0.0.3

`ip route add default via 15.0.0.2`  (tạo route để forward lưu lượng từ địa chỉ 15.0.0.3 đến địa chỉ 15.0.0.2, máy 15.0.0.3 có thể ping được thêm địa chỉ 20.0.0.2 do địa chỉ 20.0.0.2 và 15.0.0.3 cùng 1 máy)

#### Trên máy 20.0.0.3 

`ip route add default via 20.0.0.2` (tạo route để forward lưu lượng từ địa chỉ 20.0.0.3 đến địa chỉ 20.0.0.2, máy 20.0.0.3 có thể ping được thêm địa chỉ 15.0.0.2 do địa chỉ 15.0.0.2 và 20.0.0.3 cùng 1 máy)

#### Trên máy 192.168.40.68 (15.0.0.2, 20.0.0.2)

Thiết lập iptables

Tiếp theo sử dụng lệnh `sysctl net.ipv4.ip_forward=1` để forward lưu lượng.

`iptables -t nat -A POSTROUTING -s 15.0.0.3 -o eth1 -j MASQUERADE` (Cho phép nat ip 15.0.0.3 ra ngoài internet). 

---

- Ngoài ra ta có lệnh chặn ip trên card mạng:

`iptables -A FORWARD -s 20.0.0.3 -i eth1 -j DROP` (**chặn lưu lượng từ 20.0.0.3 trên eth1, lưu ý với lệnh này chỉ chặn ip 20.0.0.3 trên card eth1 khi nó sử dụng ip 20.0.0.3 để đi internet nếu ip 20.0.0.3 mà được nat để đi internet thì sẽ không chặn được**)

iptables -I FORWARD -s 20.0.0.3 -j ACCEPT (cho phép 1 ip di được)

iptables -I FORWARD -s 20.0.0.3 -j DROP (chặn 1 ip đi qua)

