# Phần này giới thiệu về Bandwidth Test Controller 

## 1.Giới thiệu

**BWCTL** là 1 công cụ sử dụng các chương trình đo lường network như Iperf, Iperf3, Nuttcp, Ping, Traceroute, Tracepath, và OWAMP để đo băng thông TCP tối đa với một loạt các tùy chọn điều chỉnh khác nhau như sự trễ, tỉ lệ mất gói tin,... Mặc định, bwctl sẽ dùng iperf.

**Các tính năng chính:**
  <ul>
    <ul>
      
<li>Hỗ trợ Iperf, Iperf3 và Nuttcp tests</li>
  
<li>Hỗ trợ ping tests</li>
  
<li>Hỗ trợ OWAMP (One-Way Latency) tests</li>
  
<li>Hỗ trợ Traceroute và Tracepath tests</li>
  
<li>Hỗ trợ ipv6 mà không cần thêm tùy chọn nào</li>
  
<li>Dữ liệu từ cả hai phía được trả lại vì thế có thể so sánh kết quả giữa hai phía</li>
  
<li>Không yêu cầu local BWCTL server, BWCTL client sẽ kiểm tra xem có local BWCTL server hay không và sử dụng nó nếu có</li>
  
<li>Port ranges cho kết nối có thể được chỉ định</li>
  
<li>Giới hạn số lượng test có thể chạy</li>

</ul>
</ul>

- BWCTL server yêu cầu NTP được sử dụng để cả hai phía đều có cùng scheduled time window

## 2. Một số option phổ biến

|Options|Descriptions
|-------|------------
|-4, --ipv4|Chỉ dùng ipv4
|-6, --ipv6|	Chỉ dùng ipv6
|-c, --receiver|Chỉ định host chạy Iperf, Iperf3 or Nuttcp server
|-s, --sender|Chỉ định host chạy Iperf, Iperf3 or Nuttcp client
|-T, --tool|Chỉ định tool sử dụng ( iperf, iperf3, nuttcp)
|-b, --bandwidth|Giới hạn bandwidth với UDP
|-l, --buffer_length|độ dài của read/write buffers (bytes). Mặc định 8 KB TCP, 1470 bytes UDP
|-t,--test_duration|Thời gian cho bài test, mặc định là 10 giây
|-u, --udp|	Dùng UDP test, vì mặc định là dùng TCP
|-h, --help|	Show help message
|-p, --print|	In kết quả ra file
|-V, --version|Phiên bản
|-P|	Số luồng kết nối tới server
|-w|	kích thước gói tin
|--tester_port|port để test, mặc định được specific bởi công cụ sử dụng


## 3. Cài đặt

- Cài đặt EPEL RPM

`rpm -hUv https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm`

- Cài đặt Internet2-repo RPM

`rpm -hUv http://software.internet2.edu/rpms/el7/x86_64/main/RPMS/Internet2-repo-0.7-1.noarch.rpm`

- Cập nhật lại yum cache

`yum clean all`

- Cài đặt bwctl

`yum install bwctl -y`

- Cài đặt ntp

`yum install ntp -y`

- Chỉnh sửa file /etc/ntp.conf để cả hai cùng trỏ về một ntp server (chỉnh ở cả 2 máy)

`server vn.pool.ntp.org iburst`

<img src="https://i.imgur.com/r6xFaE0.png">

- Khởi động lại dịch vụ ntp

`systemctl restart ntpd`

- Kiểm tra lại

`ntpq -p`

- Sử dụng lệnh  để kiểm tra bandwith (với 192.168.40.70 là địa chỉ ip phía đầu xa)

`bwctl -c 192.168.40.70`

<img src="https://i.imgur.com/xvPu2Ek.png">

- Kết quả giống với câu lệnh phía trên nhưng nó sẽ có thêm phần kết quả của phía sender

`bwctl -x -c 192.168.40.70`

- Giống câu lệnh trên nhưng 192.168.40.70 là sender chứ không phải local 192.168.40.68

`bwctl -x -c 192.168.40.70 -s 192.168.40.68`(local 192.168.40.68 là receiver)

- 30 giây TCP iperf test với 192.168.40.70 là sender còn local 192.168.40.68 là receiver

` bwctl -t 30 -T iperf -s 192.168.40.70`

- 10 giây UDP test với sender rate là 10 Mbits/sec từ 192.168.40.70 tới local 192.168.40.68

`bwctl -I 3600 -R 10 -t 10 -u -b 10m -s 192.168.40.70`

