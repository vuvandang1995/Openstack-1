# Ghi chú token

có 3 loại :

**uuid**:  là loại token gồm 32 kí tự được dùng để xác thực và ủy quyền. Phương pháp dùng để sinh ra UUID token:
```
def _get_token_id(self, token_data):
 return uuid.uuid4().hex
```

- Ưu điểm: 
<ul>
<ul>
  <li> Gm 32 bytes, nhỏ gọn dễ sử dụng./li>
  <li> Nó có thể được thêm trực tiếp vào câu lệnh cURL.</li>
</ul>
</ul>

- Nhược điểm:
<ul> 
<ul>
  <li> Không thể chứa đủ thông tin để thực hiện việc ủy quyền.</li>
  <li> Các services của OpenStack sẽ luôn phải gửi lại token này lại cho Keystone để xác thực xem hành động có hợp lệ không, điều này khiến keystone phải chịu tải lớn</li>
  <li> UID token bược phải được lưu lại trong một backend (thường là database sẽ khiến database bị đầy)Nó cũng có thể được loại bỏ bằng các sử dụng DELETE request với token id. Tuy nhiên nó sẽ không thực sự bị loại bỏ khỏi backend mà chỉ được đánh dấu là đã được loại bỏ.</li>
</ul>
</ul>

**Quá trình tạo UUID token:**

<img src="http://i.imgur.com/C36awEz.png">

- User request tới keystone tạo token với các thông tin: user name, password, project name
- Chứng thực user, lấy User ID từ backend LDAP (dịch vụ Identity)
- Chứng thực project, thu thập thông tin Project ID và Domain ID từ Backend SQL (dịch vụ Resources)
- Lấy ra Roles từ Backend trên Project hoặc Domain tương ứng trả về cho user, nếu user không có bất kỳ roles nào thì trả về Failure(dịch vụ Assignment)
- Thu thập các Services và các Endpoints của các service đó (dịch vụ Catalog)
- Tổng hợp các thông tin về Identity, Resources, Assignment, Catalog ở trên đưa vào Token payload, tạo ra token sử dụng hàm uuid.uuid4().hex
- Lưu thông tin của Token vào SQL/KVS backend với các thông tin: TokenID, Expiration, Valid, UserID, Extra


**Quá trình xác thực Token**

<img src="http://i.imgur.com/U85Kpgq.png">

- Gửi yêu cầu chứng thực token sử dụng API: GET v3/auth/tokens và token (X-Subject-Token, X-Auth-Token)
- Thu thập token payloads từ token backend KVS/SQL kiểm tra trường valid. Nếu không hợp lệ trả về thông báo Token Not Found. Nếu tìm thấy chuyển sang bước tiếp theo
- Phân tích token và thu thập metadata: User ID, Project ID, Audit ID, Token Expire
- Kiểm tra token đã expired chưa. Nếu thời điểm hiện tại < "expired time" theo UTC thì token chưa expired, chuyển sang bước tiếp theo, ngược lại trả về thông báo token not found
- Kiểm tra xem token đã bị thu hồi chưa (kiểm tra trong bảng revocation_event của database keystone)

Nếu token đã bị thu hồi (tương ứng với 1 event trong bảng revocation_event) trả về thông báo Token Not Found. Nếu chưa bị thu hồi trả về token (truy vấn HTTP thành công HTTP/1.1 200 OK )

**Quá trình thu hồi Token**

<img src="http://i.imgur.com/qaPVzFI.png">

- Gửi yêu cầu thu hồi token với API request DELETE v3/auth/tokens. Trước khi thực hiện sự kiện thu hồi token thì phải chứng thực token nhờ vào tiến trình Token Validation Workflow đã trình bày ở trên.
- Kiểm tra trường Audit ID. Nếu có, tạo sự kiện thu hồi với audit id. Nếu không có audit id, tạo sự kiện thu hồi với token expired
- Nếu tạo sự kiện thu hồi token với audit ID, các thông tin cần cập nhật vào revocation_event table của keystone database gồm: audit_id, revoke_at, issued_before.
- Nếu tạo sự kiện thu hồi token với token expired, các thông tin cần thiết cập nhật vào revocation_event table của keystone database gồm: user_id, project_id, revoke_at, issued_before, token_expired.
- Loại bỏ các sự kiện của các token đã expired từ bảng revocation_event của database "keystone"
- Cập nhật vào token database, thiết lập lại trường "valid" thành false (0)
