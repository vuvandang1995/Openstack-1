## 1. Tổng quan về OMD.
<a name="1"></a>

<img src="https://i.imgur.com/jhWCjir.png">

OMD - Open Monitoring Distribution là một project được phát triển từ năm 2010 bới Mathias Kettner. OMD sử dụng nhân là Nagios Core, kết hợp với các phần mềm mã nguồn mở khác để đóng gói thành một sản phẩm phục vụ cho nhu cầu **giám sát**, **cảnh báo** và **hiển thị**

## 1.2. Lịch sử hình thành:

Dự án Check_MK được phát triển từ năm 2008 như là một plugin của Nagios Core. 

Năm 2010 dự án OMD (Open Monitoring Distribution) được khởi động bởi Mathias Kettner, là sự kết hợp của Nagios, Check_MK, NagVis, PNP4Nagios, DocuWiki, ...tạo nên sự linh hoạt trong giám sát. Các distro của OMD đang là OMD-LABS và CHECK_MK RAW.

Năm 2015, phiên bản đơn giản của OMD ra đời gọi là Check_MK, Check_MK có 2 phiên bản là Check_MK Raw Edition (CRE) và Check_MK Enterprise Edition (CEE).

## 1.3. Phân biệt OMD-LABS và CHECK_MK RAW

OMD-LABS chứa tất cả các thành phần nguyên bản của OMD và một số addons thêm vào như Grafana, InfluxDB, Naemon, Icinga 2. Check_MK RAW edition tập trung vào Check_MK, đây là một gói nhỏ hơn chứa các thành phần để chạy Check_MK.

Trong bài tóm tắt này tôi sẽ tóm tắt về OMD. Gồm 2 phiên bản là OMD-LABS và OMD thường hay còn gọi là Check_MK RAW

<a name="21"></a>
### 1.3.1. OMD-LABS

OMD Labs-Edition là một nền tảng giám sát và một khái niệm mới về cài đặt, duy trì và cập nhật một hệ thống giám sát được xây dựng trên Nagios. Không được tích hợp sẵn trong các bản phân phối Linux mà tích hợp vào hệ thống dưới dạng các Package rpm và deb. 

Phiên bản stable là 2.40, thường thì 6 tháng sẽ có một phiên bản stable được phát hành. Tính từ thời điểm hiện tại thì đến tháng 11 năm 2017, một phiên bản stable mới sẽ được phát hành.

OMD-Labs chứa nhiều thành phần phần mềm mới so với gói OMD thường (Check_MK). Một số cài đặt mặc định đã thay đổi

### Một số điểm khác của OMD-LABS với OMD thường (Check_MK):

Các thành phần phần mềm bổ sung: OMD-Labs chỉ bổ sung các thành phần phần mềm mới và không loại bỏ bất kì phần mềm nào, làm cho OMD-Labs trở nên hoàn hảo hơn, và có thể chuyển sang các phiên bản khác bằng `omd update`

- Lõi giám sát mới: Bên cạnh Nagios 3, OMD-Labs còn chứa hai Cores mới là Naemon và Icinga2. Trong khi Naemon hoàn toàn tương thích với định dạng Nagios 3 config thì Icinga2 sử dụng định dạng cấu hình mới

- Biểu đồ Grafana/Influxdb: Bên cạnh PNP4Nagios OMD có đồ thị Grafana dựa trên Influxdb. Để tạo mẫu đồ thị dựa trên mẫu, đã có histou (Histou được thiết kế để thêm các mẫu vào Grafana từ dữ liệu của Nagios). Giao diện giữa core giám sát và Influxdb được thực hiện trong thành phần Nagflux.

- Hệ thống con Prometheus: Bên cạnh việc giám sát truyền thống, OMD-Labs đi kèm với prometheus bao gồm quản lý cảnh báo, pushgateway và blackbox exporter

- Livestatus Multitool Daemon

Thay đổi Cài đặt mặc định

- Naemon Core: Naemon thay thế Nagios làm lõi mặc định trong OMD-Labs. Để trở lại cài đặt ban đầu:

```sh
omd config set CORE nagios
```

- Chế độ SSL / TLS Apache: OMD-Labs đi kèm với kích hoạt SSL / TLS Apache theo mặc định. Để trở lại mặc định:

```sh
omd config set APACHE_MODE own
```

- Giao diện Web: Thruk là trang web mặc định trong OMD-Labs

<a name="22"></a> 
### 1.3.2. CHECK_MK RAW

Check_MK là một phần của OMD, hiện tại đang có 2 phiên bản Check_MK là Check_MK Raw Edition (CRE) và Check_MK Enterprise Edition (CEE)

- <b>Check_MK Raw Edition (CRE):</b>

<ul>
<li>Check_MK Raw Edition là một giải pháp giám sát toàn diện. Có thể sử dụng CRE miễn phí, chỉ cần tuân thủ theo một số giấy phép của GNU GPL và giấy phép của nguồn mở</li>
<li>Phiên bản stable đang là 1.2.8p22, phiên bản beta là 1.4.0b8</li>
</ul>


- <b>Check_MK Enterprise Edition (CEE)</b>

<ul>
<li>Phiên bản Check_MK Enterprise Edition được phát triển dựa trên bản Raw nhưng có nhiều tính năng bổ sung cho doanh nghiệp và cũng cho phép bạn nhận được hỗ trợ từ nhà phát hành một cách chuyên nghiệp</li>
<li>Phiên bản stable đang là 1.2.8p22, phiên bản beta là 1.4.0b8</li>
</ul>

### So sánh tính năng của 2 phiên bản Check_MK

| Tính năng | Check_MK Raw Edition (CRE) | Check_MK Enterprise Edition (CEE) |
|-----------|----------------------------|-----------------------------------|
| Hệ thống giám sát toàn diện về CNTT với hơn 1000 plugin kiểm tra | Có | Có |
| Kiểm kê phần cứng / phần mềm | Có | Có |
| Bảng điề khiển sự kiện xử lý thông tin từ syslog | Có | Có |
| Hiệu suất lưu trữ dữ liệu và tính khả dụng cao | Có | Có |
| Hiệu suất cao và độ trễ thấp | Không | Có |
| Hiệu suất nâng cao cho SNMP dựa trên check bằng 100% | Không | Có |
| Livestatus-Proxy: thời gian phản hồi tối ưu và sự ổn định trong thiết lập phân phối | Không | Có |
| Agent Bakery: Tự động đóng gói của các agent giám sát cá nhân | Không | Có |
| Report: Tạo các báo cáo riêng lẻ ở định dạng PDF | Không | Có |
| Hệ thống tương tác mới để hiển thị dữ liệu hiệu suất | Không | Có |
| Tích hợp kết nối với công cụ đồ họa Graphite | Không | Có |
| Giao diện người dùng tùy chọn bằng tiếng Đức | Không | Có |
| Quản lý hệ điều hành thông qua Web-GUI | Không | Không |
| Duy trì các phiên bản và các thể hiện thông qua Web-GUI | Không | Không |
| Tích hợp HA | Không | Không |
| Giá thành | Miễn phí | Có giá từ 600 euro |

<a name="3"></a>
 
 
## 2. Ưu điểm trong thiết kế kiến trúc của OMD
<a name="2"></a>

OMD được xây dựng từ những đóng góp của cộng đồng về những khó khăn hay khuyết điểm mà Nagios gặp phải, từ đó đưa ra quyết định cần tích hợp thêm những sản phẩm gì để cải thiện.

 Việc cài đặt trở nên vô cùng đơng giản. OMD được đóng gói hoàn chỉnh trong một package, việc cài đặt và cấu hình chỉ mất khoảng 10 phút với chỉ một câu lệnh
 
 <img src="https://i.imgur.com/jWNsHpB.png">

<a name="21"></a>
### 2.1 Check_MK

Check_MK ra đời để giải quyết bài toán về hiệu năng mà Nagios gặp phải trong quá khứ.Cơ chế mới của Check_MK cho phép việc mở rộng hệ thống trở nên dễ dàng hơn, có thể giám sát nhiều hệ thống chỉ từ một máy chủ Nagios server.

<img src="https://i.imgur.com/VwoR5KA.gif">

Có 2 mô đun mà Check_MK sử dụng để cải thiện đáng kể hiệu năng là Livestatus và Livecheck

<a name="22"></a>
### 2.2 Livestatus

#### Trước khi có Livestatus

- Kết quả giám sát được sẽ lưu trong file `status.dat` gây nên hiện tượng nút thắt cổ chai cho CPU và Disk I/O
- Trạng thái của file status không phải là real-time mà update ít nhất là mỗi 10s
- NDOUtils sử dụng database để theo dõi kết quả (MySQL hoặc PostgreSQL), nhưng vẫn còn một số thiếu sót quan trọng.
- Việc cài đặt NDOUtils khá phức tạp
- NDOUtils cần một database cho việc lưu trữ dữ liệu. Hơn nữa, việc dữ liệu lưu trong database này tăng lên một cách nhanh chóng khiến cho bạn phải tiêu tốn nhiều CPU chỉ để cập nhập database.
- Một số dự án tương tự vẫn sử dụng NDOUtils: Centreon và Opsview
- Việc dọn dẹp database có thể khiến Nagios bị treo trong một khoảng thời gian nhất định

#### Sau khi có Livestatus

- Livestatus cũng sử dụng Nagios Event Broker API như NDO, nhưng nó sẽ không chủ động ghi dữ liệu ra. Thay vào đó, nó sẽ mở ra một socket để dữ liệu có thể được lấy ra theo yêu cầu
- Livestatus tiêu tốn ít CPU
- Livestatus không làm cho Disk I/O thay đổi khi truy vấn trạng thái dữ liệu
- Không cần cấu hình. Không cần cơ sở dữ liệu. Không cần quản lý
- Livestatus có quy mô lớn với hơn 50.000 dịch vụ

<a name="23"></a>
### 2.3 Livecheck

Trước Nagios 4.0, ngay cả một hệ thống hoàn hảo hiếm khi quản lý để thực hiện hơn một vài nghìn lần kiểm tra mỗi phút.

Trong khi hệ thống càng lớn, tỷ lệ check tối đa sẽ trở nên rất tệ. Càng nhiều máy chủ và dịch vụ thì đồng nghĩa với việc khoảng thời gian check cần phải tăng lên. Tại sao lại như vậy?

#### Các vấn đề tồn tại trong Nagios (trước Nagios 4.0)

- Mỗi lần check tạo ra một bản fork. Quá trình fork rất tốn kém ngay cả khi kernel được tối ưu hóa
- Quá trình fork trong Nagios Core (trước phiên bản Nagios 4.0) không phân tán ra nhiều CPU mà thực hiện trên chỉ một CPU đơn. Điều này dẫn tới việc giới hạn số lần check mỗi giây, trong khi phần lớn các CPU khác rảnh rỗi.

#### Làm thế nào để Livecheck giải quyết được vấn đề nút thắt cổ chai

- Livecheck sử dụng các helper process, các core giao tiếp với helper thông qua Unix socket (điều này không xảy ra trên file system)
- Chỉ có một một helper program được fork thay vì toàn bộ Nagios Core.
- Các tiến trình fork được phân tán trên tất cả các CPU thay vì chỉ một như trước
- Process VM size tổng chỉ khoảng 100KB
- Việc thực hiện check_icmp sẽ cho một con số cải tiến cụ thể. Giả sử nếu sử dụng CPU dual core 2800 MHz CPU:
	- Trước đây sẽ là 300 ICMP check/second
	- Sau khi cải tiến là 2600 ICMP check/second

<a name="24"></a>
### 2.4 Multisite - Một giao diện Web tiên tiến hơn Nagios

Multisite là một phần của dự án Check_MK như là một giao diện web cho người dùng tốt hơn để thay thế cho Nagios

Một GUI mới và sáng tạo để xem thông tin trạng thái Nagios và kiểm soát hệ thống giám sát. Nó dựa trên MK Livestatus và nhằm mục đích thay thế cho GUI web Nagios. Multisite hỗ trợ giám sát phân tán bằng một cách hiệu quả nhất

 - WATO : no more config file :) 

Với Nagios, mỗi khi cấu hình một host mới, hay cấu hình một cảnh báo...bạn phải mất ít nhất 15p cho việc tạo, sửa đổi file cấu hình, dùng câu lệnh để restart service...chưa kể thời gian tìm lỗi nếu cấu hình sai. Với WATO, tất cả các thao tác đó có thể thực hiện trong vòng 3p trên giao diện WEB.

 - Agent giám sát có sẵn cho  Linux và Windows

 - Đáp ứng được giao diện người dùng trên điện thoại

 - Khả năng tìm kiếm mạnh mẽ

 - Thông số đo lường trực quan với Perf-O-Meter

<img src="https://i.imgur.com/t7EdAcd.png">

<a name="3"></a>
## 3. NOC với Dashboard (Nhờ vào PNP4Nagios & NagVis)

<a name="31"></a>
### 3.1 PNP4Nagios & NagVis

NagVis là một addon trực quan cho hệ thống giám sát nổi tiếng Nagios. NagVis có thể được sử dụng để có cái nhìn trực quan dữ liệu của  Nagios, ví dụ: Để hiển thị các quy trình CNTT như một hệ thống mail hoặc một cơ sở hạ tầng mạng.

<a name="32"></a>
### 3.2 Check_MK (OMD) API cho tự động Provisioning

<a name="4"></a>
## 4. Tài liệu tham khảo
