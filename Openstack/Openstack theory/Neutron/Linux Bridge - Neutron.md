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
