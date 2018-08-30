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
