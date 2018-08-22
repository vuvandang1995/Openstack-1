# Phần này giới thiệu về công cụ Nuttcp

## 1. Giới thiệu

**Nuttcp** là một công cụ đo lường hiệu năng mạng được sử dụng bởi các nhà quản lý mạng và hệ thống .Xác định thông lượng mạng TCP (hoặc UDP) thô bằng cách truyền memory buffers từ một hệ thống nguồn qua mạng kết nối tới hệ thống đích, truyền dữ liệu cho một khoảng thời gian nhất định hoặc chuyển một số lượng cụ thể các bytes.


**Có 2 mode cơ bản**

- Mode classic là mode transmitter/receiver, nó cũng chính là mode mà ttcp và nttcp hoạt động. Ở mode này máy nhận được khởi tạo trước bằng câu lệnh nuttcp -r sau đó máy gửi sẽ phải được start bằng câu lệnh nuttcp -t. Mode này hiện đã không được khuyến khích sử dụng nữa.

- Mode đang được khuyến khích sử dụng đó là mode client/server. Với mode này, server sẽ được start với câu lệnh nuttcp -S (hoặc "nuttcp -1") và sau đó clent có thể truyền dữ liệu (sử dụng "nuttcp -t") hoặc nhận dữ liệu (sử dụng "nuttcp -r") từ phía server. Tất cả các thông tin cung cấp bởi nuttcp sẽ được thông báo trên phía client

| Options | Descriptions |
|---------|--------------|
| -h | Các options có thể sử dụng |
| -V | Hiển thị thông tin về phiên bản |
| -t | Chỉ định máy transmitter |
| -r | Chỉ định máy receiver |
| -S | Chỉ định máy server |
| -1 | Giống với '-S' |
| -b | Định dạng output theo kiểu one-line |
| -B | Buộc receiver phải đọc toàn bộ buffer |
| -u | Sử dụng UDP (mặc định là TCP) |
| -v | Cung cấp thêm thông tin |
| -w | window size |
| -p | port sử dụng để kết nối dữ liệu, mặc định là 5001 |
| -P | với mode client-server, đây là port để kiểm soát kết nối, mặc định là 5000 |
| -n | Số lượng bufers |
| -N | Số lượng luồng dữ liệu truyền |
| -R | Tốc độ truyền |
| -l | packet length |
| -T | thời gian, mặc định là 10 giây |
| -i | thời gian gửi báo cáo (giây) |

## 2. Cài đặt Nuttcp

**Cài đặt nuttcp**

Đối với CentOS

`yum install nuttcp`

Đối với Ubuntu

`apt-get install nuttcp`

