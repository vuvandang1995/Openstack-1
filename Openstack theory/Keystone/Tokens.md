# Giới thiệu về token

Vào những ngày đầu, Keystone hỗ trợ UUID token. Đây là loại token gồm 32 kí tự được dùng để xác thực và ủy quyền. Lợi ích mà loại token này mang lại đó là nó nhỏ gọn và dễ sử dụng, nó có thể được thêm trực tiếp vào câu lệnh cURL. Tuy nhiên nó lại không thể chứa đủ thông tin để thực hiện việc ủy quyền. Các services của OpenStack sẽ luôn phải gửi lại token này lại cho Keystone để xác thực xem hành động có hợp lệ không. Điều này khiến Keystone trở thành trung tâm cho mọi hoạt động của OpenStack.

Trong nỗ lực kiếm tìm một loại token mới để khắc phục những nhược điểm của UUID token, nhóm phát triển đã tạo ra PKI token. Token này chứa đủ thông tin để xác thực và ủy quyền và đồng thời nó cũng chứa cả danh mục dịch vụ. Bên cạnh đó, token này được gán vào và các dịch vụ có thể cache lại nó để dử dụng cho đến khi nó hết hiệu lực hoặc bị hủy. Loại token này vì thế cũng khiến lượng traffic tới Keystone server ít hơn. Tuy nhiên kích cỡ của nó có thể lên tới 8k và điều này làm việc gán vào HTTP header trở nên khó khăn hơn. Rất nhiều các web server mặc định sẽ không cho phép điều này nếu chưa được config lại. Thêm vào đó, loại token này cũng rất khó được sử dụng trong câu lệnh cURL. Vì những hạn chế mà các nhà phát triển đã cố gắng sửa đổi và ra mắt một phiên bản khác đó là PKIz, tuy nhiên theo đánh giá của cộng đồng thì loại token này vẫn rất lớn về kích thước.

Điều này buộc Keystone team phải đưa ra một loại mới và đó là Fernet token. Fernet token khá nhỏ (255 kí tự) tuy nhiên nó lại chưa đủ thông tin để ủy quyền. Bên cạnh đó, việc nó chứa đủ thông tin cũng không khiến token database phải lưu dữ liệu token nữa. Các nhà vận hành thường phải dọn dẹp Keystone token database để hệ thống của họ hoạt động ổn định. Mặc dù vậy, Fernet token có nhược điểm đó là symmetric keys được dùng để tạo ra token cần được phân phối và xoay vòng. Các nhà vận hành cần phải giải quyết vấn đề này, tuy nhiên họ có vẻ thích thú với việc này hơn là sử dụng những loại token khác.

2. UUID Tokens

- Là một chuỗi UUID gồm 32 kí tự được generate random được xác thực bởi identity service Phương thức hexdigest() được sử dụng để tạo ra chuỗi kí tự hexa. Điều này khiến token URL trở nên an toàn và dễ dàng trong việc vận chuyển đến các môi trường khác
 
- UUID token bược phải được lưu lại trong một backend (thường là database). Nó cũng có thể được loại bỏ bằng các sử dụng DELETE request với token id. Tuy nhiên nó sẽ không thực sự bị loại bỏ khỏi backend mà chỉ được đánh dấu là đã được loại bỏ. Vì nó chỉ có 32 bytes nên kích thước của nó trong HTTP header cũng sẽ là 32 bytes.
Loại token này rất nhỏ và dễ sử dụng quy nhiên nếu sử dụng nó, Keystone sẽ là "cổ chai" của hệ thống bởi mọi cuộc giao tiếp đều cần tới keystone để xác thực token.

- Method dùng để sinh ra UUID token:

``
def _get_token_id(self, token_data):
 return uuid.uuid4().hex
 ``
 **Token Generation Workflow**

<img src="http://i.imgur.com/C36awEz.png">

- User request tới keystone tạo token với các thông tin: user name, password, project name
- Chứng thực user, lấy User ID từ backend LDAP (dịch vụ Identity)
- Chứng thực project, thu thập thông tin Project ID và Domain ID từ Backend SQL (dịch vụ Resources)
- Lấy ra Roles từ Backend trên Project hoặc Domain tương ứng trả về cho user, nếu user không có bất kỳ roles nào thì trả về Failure(dịch vụ Assignment)
- Thu thập các Services và các Endpoints của các service đó (dịch vụ Catalog)
- Tổng hợp các thông tin về Identity, Resources, Assignment, Catalog ở trên đưa vào Token payload, tạo ra token sử dụng hàm uuid.uuid4().hex
- Lưu thông tin của Token vào SQL/KVS backend với các thông tin: TokenID, Expiration, Valid, UserID, Extra


**Token Validation Workflow**

<img src="http://i.imgur.com/U85Kpgq.png">

- Gửi yêu cầu chứng thực token sử dụng API: GET v3/auth/tokens và token (X-Subject-Token, X-Auth-Token)
- Thu thập token payloads từ token backend KVS/SQL kiểm tra trường valid. Nếu không hợp lệ trả về thông báo Token Not Found. Nếu tìm thấy chuyển sang bước tiếp theo
- Phân tích token và thu thập metadata: User ID, Project ID, Audit ID, Token Expire
- Kiểm tra token đã expired chưa. Nếu thời điểm hiện tại < "expired time" theo UTC thì token chưa expired, chuyển sang bước tiếp theo, ngược lại trả về thông báo token not found
- Kiểm tra xem token đã bị thu hồi chưa (kiểm tra trong bảng revocation_event của database keystone)

Nếu token đã bị thu hồi (tương ứng với 1 event trong bảng revocation_event) trả về thông báo Token Not Found. Nếu chưa bị thu hồi trả về token (truy vấn HTTP thành công HTTP/1.1 200 OK )

**Token Revocation Workflow**

<img src="http://i.imgur.com/qaPVzFI.png">

- Gửi yêu cầu thu hồi token với API request DELETE v3/auth/tokens. Trước khi thực hiện sự kiện thu hồi token thì phải chứng thực token nhờ vào tiến trình Token Validation Workflow đã trình bày ở trên.
- Kiểm tra trường Audit ID. Nếu có, tạo sự kiện thu hồi với audit id. Nếu không có audit id, tạo sự kiện thu hồi với token expired
- Nếu tạo sự kiện thu hồi token với audit ID, các thông tin cần cập nhật vào revocation_event table của keystone database gồm: audit_id, revoke_at, issued_before.
- Nếu tạo sự kiện thu hồi token với token expired, các thông tin cần thiết cập nhật vào revocation_event table của keystone database gồm: user_id, project_id, revoke_at, issued_before, token_expired.
- Loại bỏ các sự kiện của các token đã expired từ bảng revocation_event của database "keystone"
- Cập nhật vào token database, thiết lập lại trường "valid" thành false (0)

**Multiple Data Centers**

<img src="http://i.imgur.com/5u1tM2m.png">

UUID Token không hỗ trợ xác thực và ủy quyền trong trường hợp multiple data centers bởi token được lưu dưới dạng persistent (cố định và không thể thay đổi). Như ví dụ mô tả ở hình trên, một hệ thống cloud triển khai trên hai datacenter ở hai nơi khác nhau. Khi xác thực với keystone trên datacenter US-West và sử dụng token trả về để request tạo một máy ảo với Nova, yêu cầu hoàn toàn hợp lệ và khởi tạo máy ảo thành công. Trong khi nếu mang token đó sang datacenter US-East yêu cầu tạo máy ảo thì sẽ không được xác nhận do token trong backend database US-West không có bản sao bên US-East.
