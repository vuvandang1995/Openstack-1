# Quá trình migrate

## 1. Giới thiệu về tính năng migrate trong OpenStack

<img src="http://i.imgur.com/vFgXoEK.png">

Migration là quá trình di chuyển máy ảo từ host vật lí này sang một host vật lí khác. Migration được sinh ra để làm nhiệm vụ bảo trì nâng cấp hệ thống. Ngày nay tính năng này đã được phát triển để thực hiện nhiều tác vụ hơn:

- Cân bằng tải: Di chuyển VMs tới các host khác kh phát hiện host đang chạy có dấu hiệu quá tải.
- Bảo trì, nâng cấp hệ thống: Di chuyển các VMs ra khỏi host trước khi tắt nó đi.
- Khôi phục lại máy ảo khi host gặp lỗi: Restart máy ảo trên một host khác.

Trong OpenStack, việc migrate được thực hiện giữa các node compute với nhau hoặc giữa các project trên cùng 1 node compute.

## 2. Các kiểu migrate hiện có trong OPS và workflow của chúng

OpenStack hỗ trợ 2 kiểu migration đó là:

- Cold migration : Non-live migration
- Live migration :
  - True live migration (shared storage or volume-based)
  - Block live migration

**Workflow khi thực hiện cold migrate**

- Tắt máy ảo (giống với virsh destroy) và ngắt kết nối với volume
- Di chuyển thư mục hiện tại của máy ảo (instance_dir ->
instance_dir_resize)
- Nếu sử dụng QCOW2 với backing files (chế độ mặc định) thì image sẽ được convert thành dạng flat
- Với shared storage, di chuyển thư mục chứa máy ảo. Nếu không, copy toàn bộ thông qua SCP.

**Workflow khi thực hiện live migrate**

- Kiểm tra lại xem  storage backend có phù hợp với loại migrate sử dụng không
  - Thực hiện check shared storage với chế độ migrate thông thường
  - Không check khi sử dụng block migrations
  - Việc kiểm tra thực hiện trên cả 2 node gửi và nhận, chúng được điều phối bởi RPC call từ scheduler.
- Đối với nơi nhận
  - Tạo các kết nối càn thiết với volume.
  - Nếu dùng block migration, tạo thêm thư mục chứa máy ảo, truyền vào đó những backing files còn thiếu từ Glance và tạo disk trống.
- Tại nơi gửi, bắt đầu quá trình migration (qua url)
- Khi hoàn thành, generate lại file XML và define lại nó ở nơi chứa máy ảo mới.

### 3. So sánh ưu nhược điểm giữa cold và live migrate

**Cold migrate**

- Ưu điểm:
  - Đơn giản, dễ thực hiện
  - Thực hiện được với mọi loại máy ảo
- Hạn chế:
  - Thời gian downtime lớn
  - Không thể chọn được host muốn migrate tới.
  - Quá trình migrate có thể mất một khoảng thời gian dài

**Live migrate**

- Ưu điểm:
  - Có thể chọn host muốn migrate
  - Tiết kiệm chi phí, tăng sự linh hoạt trong khâu quản lí và vận hành.
  - Giảm thời gian downtime và gia tăng thêm khả năng "cứu hộ" khi gặp sự cố
- Nhược điểm:
  - Dù chọn được host nhưng vẫn có những giới hạn nhất định
  - Quá trình migrate có thể fails nếu host bạn chọn không có đủ tài nguyên.
  - Bạn không được can thiệp vào bất cứ tiến trình nào trong quá trình live migrate.
  - Khó migrate với những máy ảo có dung lượng bộ nhớ lớn và trường hợp hai host khác CPU.

- Trong live-migrate, có 2 loại đó là True live migration và Block live migration. Hình dưới đây mô tả những loại storage mà 2 loại migration trên hỗ trợ:

<img src="https://i.imgur.com/ccoPoFC.png">

**Ngữ cảnh sử dụng:**

- Nếu bạn buộc phải chọn host và giảm tối da thời gian downtime của server thì bạn nên chọn live-migrate (tùy vào loại storage sử dụng mà chọn true hoặc block migration)
- Nếu bạn không muốn chọn host hoặc đã kích hoạt configdrive (một dạng ổ lưu trữ metadata của máy ảo, thường được dùng để cung cấp cấu hình network khi không sử dụng DHCP) thì hãy lựa chọn cold migrate.

## 4. Quá trình live migrate trên compute

<img src="https://i.imgur.com/iQenkN6.png">


- User sẽ gửi yêu cầu migrate 1 instance đến API

- API sẽ các thực instance, user, admin policies...

- Sau khi API xác thực thành công sẽ truy cập vào database để kiếm tra instance đó chắc chắn tồn tại và lấy thông tin của instance

- Sau khi lấy thông tin của instance API sẽ chuyển yếu cầu live migrate instance đến Scheduler

- Scheduler sẽ kiểm tra các compute target và tìm ra 1 compute phù hợp đáp ứng các tiêu chí để migrate (*Nếu không có host nào đáp ứng được yêu cầu thì Scheduler sẽ hủy tiến trình migrate*)

- Sau khi tìm được host phù hợp Scheduler sẽ gửi yêu cầu migrate đến compute source

- Compute source sẽ bắt đầu thực hiện migrate đến compute target đã được chọn thông qua libvirt URI

- Compute source chấm dứt kết nối tới instance volumes

- Compute source giải phóng security groups khỏi instance

- Compute source chuyển thông tin network cho compute target

- Sau khi Compute target nhận được thông tin về network của instance, Compute target sẽ gửi yêu cầu setup network cho instance tới neutron và neutron gửi thông tin bản ghi về thông tin network cho compute target (*Kết nối network tới compute target được thiết lập để compute target và compute source chuyển dữ liệu các thông số còn lại từ compute source sang compute target*)

- Compute target setup volumes cho instance

- Compute target cập nhật hoàn thành bản ghi network cho instance

- Compute target define các thông số đã cập nhật vào instance

- Compute target yêu cầu neutron setup thông số network của máy ảo như trong bản ghi đã hoàn thành

- Sau đó Compute target update các thông tin trạng thái (active) cho instance và hủy kết nối tới compute source

- Compute Source hủy bỏ source instance đã migrate

- Compute Source gửi yêu cầu tới neutron hủy bỏ tất cả các network trên instance đã migrate, neutron sẽ hủy bỏ các network liên quan đến instance đã migrate của Compute source

- Quá trình migrate hoàn thành
