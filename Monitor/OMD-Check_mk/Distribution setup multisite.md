# cấu hình distribution omd-check_mk

### 1. Yêu cầu

`Distributed Monitoring`: Gộp các server lại để monitor các host, giải pháp thực hiện việc giám sát tập trung nhiều `site` trên một máy chủ. Thay vì phải vào từng site (Slave) để theo dõi các host/service trên đó, chúng ta chỉ cần vào `site` chính (Master) để nắm bắt được toàn bộ các site slave. Điều này vô cùng tiện lợi khi chúng ta có nhiều Data Center cần phải giám sát.

Để cấu hình được Distributed Monitoring, chúng ta cần thực hiện những yêu cầu tối thiểu sau.
- 3 site phải ping được đến nhau
- Các server đã cài đặt OMD - Check MK
- Cùng 1 timezone và thời gian trùng khớp
- Các site slave phải là site mới tạo
 




Điều kiện đầu tiên, chúng ta phải cài đặt OMD trên các server. Quay lại những [bài viết trước](#0) để biết cách cài đặt OMD, thêm site, thêm dịch vụ giám sát,... Quy ước như sau, chúng ta gọi máy chủ `main` là *master* để quản lý, thu thập thông tin từ 2 máy *slave* chủ `site-hanoi` và `site-hcm`. 

<a name="2"></a>
### 2. Cấu hình Distributed Monitoring
<a name="21"></a>
#### 2.1 Chỉnh timezone trên các server

Trên tất cả các server, chúng ta đồng nhất một Timezone là **Asia/Ho_Chi_Minh**

```
timedatectl set-timezone Asia/Ho_Chi_Minh
timedatectl
```

Sau khi chỉnh xong, chúng ta kiểm tra lại.

Trên Server `main` - Master và tương tự trên các slave

<img src="https://i.imgur.com/wf3XYXn.png">



Nhìn vào hình, chúng ta thấy thời gian giữa các Server chưa đồng nhất với nhau. Để giúp chúng đồng nhất, cài thêm gói `ntp` và đồng bộ thời gian từ máy chủ NTP.

Trên cả 3 server, chúng ta thực hiện như sau:

```
yum install ntp -y
```

Đồng bộ thời gian từ NTP server.

```
ntpdate pool.ntp.org
```


Kiểm tra lại thời gian, lần lượt các server 

```
timedatectl
```

<img src="https://i.imgur.com/hbzSL9n.png">



<a name="22"></a>
#### 2.2 Cấu hình Distributed Monitoring trên slave

Chúng ta cấu hình Distributed Monitoring cho nó như sau:

Trên site `hanoi`, chúng ta mở Terminal và thực hiện các bước sau:

- **Bước 1**: Stop hoạt động của site

```
omd stop hanoi
```


- **Bước 2**: Cấu hình site

```
omd config hanoi
```

Chọn **Distributed Monitoring**

<img src="https://i.imgur.com/4f7K1z8.png">

Chọn **LIVESTATUS_TCP** và **Change**

<img src="https://i.imgur.com/AGujL0Z.png">

Chọn **on - enable** và **Change**

<img src="https://i.imgur.com/29YafwI.png">

Dịch vụ **LIVESTATUS_TCP** đã được bật, sau đó chọn **Main menu** và **Exit** để thoát.

<img src="https://i.imgur.com/YqQJLI0.png">

<img src="https://i.imgur.com/DLcszqt.png">

- **Bước 3**: Start lại site

```
omd start hanoi
```

<a name="fw"></a>
- **Bước 4**: Cấu hình tường lửa

Nếu bạn sử dụng Firewalld, vui lòng mở port 6557 để site Master có thể "giao tiếp" với nó.

```
firewall-cmd --add-port=6557/tcp --permanent
firewall-cmd --reload
```

**Với site `hcm`, chúng ta làm tương tự như các bước làm trên site `hanoi`**


Nếu sử dụng Firewalld, vui lòng làm theo bước [trên để mở port](#fw).

<a name="23"></a>
#### 2.3 Cấu hình trên server master

Quay trở lại Web UI của site `main` trên server Master, chúng ta chọn **WATO Configuration**, **Distributed Monitoring**, chọn **New Connection**, để tạo kết nối tới các slave.

<img src="https://i.imgur.com/Unvd7Gb.png">

Điền thông tin như sau:

<img src="https://i.imgur.com/MoiivaY.png">
<img src="https://i.imgur.com/xSYbnRf.png">

**Giải thích**:

- 1 `Site ID`: Tên của site slave. Lưu ý: Phải trùng khớp với tên ở Slave
- 2 `Connection`: Chọn kiểu kết nối TCP và điền IP của Slave
- 3 `URL perfix`: URL truy cập Web UI của Slave
- 4 `Replication method`: Chọn kiểu Slave và nhận cấu hình từ Master
- 5 `Multisite-URL of remote site`: Điền URL check_mk của slave 
- 6 `WATO`: Tắt tính năng WATO trên slave, có thể bật. còn tùy thuộc nếu a muốn cho slave sử dụng WATO
	
  
Kéo xuống bên dưới và bấm vào SAVE để lưu lại thông tin.

<img src="https://i.imgur.com/8TRBngz.png">

Bấm vào `Login` để đăng nhập vào site `hanoi`

<img src="https://i.imgur.com/fxlG3wk.png">

<img src="https://i.imgur.com/7R1D0In.png">

Thông báo đã login vào site `hanoi` thành công trên site `main`.

<img src="https://i.imgur.com/5DjzalE.png">

Click vào mục `changes` và active lên

<img src="https://i.imgur.com/x9gNSzp.png">

Làm tương tự với site `hcm`

<img src="https://i.imgur.com/W3hBDFh.png">

<a name="24"></a>
#### 2.4 Thêm host giám sát các site slave

Lúc này, ở trên site `main` - Master, chúng ta có thể thêm các host cần giám sát trên bất kỳ server nào trong 3 server `main`, `hanoi` hoặc `hcm`.

**Lưu ý**: Không nên thêm host ở trên WATO của site slave, vì mỗi khi thay đổi ở trên Master, site slave sẽ mất hết dữ liệu các host.

Để thêm host giám sát, chúng ta làm như bình thường vào tab **WATO Configuration**, chọn **Hosts** và **New host**.

<img src="https://i.imgur.com/JIFfcGY.png">

Với ví dụ này, tôi sẽ thêm một host mới để check website ping YouTube trên site `hanoi`. Chúng ta cần chú ý 3 điểm tô đỏ trong hình.

<img src="https://i.imgur.com/5n0eNAi.png">

- 1 `Monitored on site`: Chọn site giám sát cho host
- 2 `Agent type`: Chọn kiểu No Agent vì chúng ta không thể cài Agent cho YouTube
- 3 `Save & Finish`: Lưu và thoát.
- 4 Click vào `changes` và `activate` host


Kiểm tra lại, chúng ta sẽ vào tab **Views**, mục **Hosts**, **All hosts**


