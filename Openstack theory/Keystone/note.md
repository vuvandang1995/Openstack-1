# Ghi chú token

có 3 loại :

## uuid:  là loại token gồm 32 kí tự được dùng để xác thực và ủy quyền. 

**Phương pháp dùng để sinh ra UUID token:**
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

## PKI Tokens: Khắc phục được việc thiếu thông tin để thực hiện ủy quyền của UUID, Chứa một lượng khá lớn thông tin ví dụ như: thời gian nó được tạo, thời gian nó hết hiệu lực, thông tin nhận diện người dùng, project, domain, thông tin về role cho user, danh mục dịch vụ,... Tất cả các thông tin này được lưu ở trog phần payload của file định dạng JSON. 

**Cách sinh ra PKI token:**
```
def _get_token_id(self, token_data):
    try:
         token_json = jsonutils.dumps(token_data, cls=utils.PKIEncoder)
         token_id = str(cms.cms_sign_token(token_json,
                                           CONF.signing.certfile,
                                           CONF.signing.keyfile))
         return token_id
     except environment.subprocess.CalledProcessError:
         LOG.exception(_LE('Unable to sign token'))
         raise exception.UnexpectedError(_('Unable to sign token.'))
```

- Ưu điểm: 
<ul>
<ul>
 <li> Token này chứa đủ thông tin để xác thực và ủy quyền và đồng thời nó cũng chứa cả danh mục dịch vụ./li>
 <li> PKI token được gán vào và các dịch vụ có thể cache lại nó để dử dụng cho đến khi nó hết hiệu lực hoặc bị hủy. Loại token này vì thế cũng khiến lượng traffic tới Keystone server ít hơn </li>
</ul>
</ul>

- Nhược điểm:
<ul>
<ul>
 <li> Kích cỡ của token khá lớn, nếu có 1 endpoints trong danh mục dịch vụ đã rơi vào khoảng 1700 bytes, Với những hệ thống lớn, kích cỡ của nó sẽ vượt mức cho phép của HTTP header (8KB). Ngay cả khi được nén lại trong PKIz format thì vấn đề cũng không được giải quyết khi mà nó chỉ làm kích thước token nhỏ đi khoảng 10%.</li>
 <li> kích thước lớn của nó cũng ảnh hưởng đến các service khác và rất khó khi sử dụng với cURL. Ngoài ra, keystone cũng phải lưu những token này trong backend vì thế người dùng vẫn sẽ phải dọn dẹp token database thường xuyên.</li>
</ul>
</ul>

## Fernet Token: Cứa một lượng nhỏ dữ liệu ví dụ như thông tin để nhận diện người dùng, project, thời gian hết hiệu lực,...Nó được sign bởi symmetric key để ngăn ngừa việc giả mạo. Cơ chế hoạt động của loại token này giống với UUID vì thế nó cũng phải được validate bởi Keystone.



- Ưu điểm: 
<ul>
<ul>
 <li> Khá nhỏ (255 kí tự) tuy nhiên nó lại chứa đủ thông tin để ủy quyền.</li>
 <li> Không cần database phải lưu dữ liệu token.</li>
  <li>
</ul>
</ul>

- Nhược điểm:
<ul>
<ul>
 <li> Dùng symmetric key để mã hóa token và các keys này buộc phải được phân phối lên tất cả các vùng của OpenStack. Thêm vào đó, keys cũng cần được xoay vòng </li>
</ul>
</ul>

File fernet key `/etc/keystone/fernet-keys => 0 1 2 3 4`

Các loại fernet keys:

- Loại 1: Primary key
  - Dùng để mã hóa và giải mã
  - file name có số index cao nhất

- Loại 2: Secondary key
  - Chỉ được dùng để giải mã
  - file name có số index nằm giữa private key và staged key.

- Loại 3: Staged key
  - Giải mã và chuẩn bị để chuyển thành primary key
  - file name nhỏ nhất (0)


----------------

**Quá trình tạo UUID token:**

<img src="http://i.imgur.com/C36awEz.png">

- User request tới keystone tạo token với các thông tin: user name, password, project name
- Chứng thực user, lấy User ID từ backend LDAP (dịch vụ Identity)
- Chứng thực project, thu thập thông tin Project ID và Domain ID từ Backend SQL (dịch vụ Resources)
- Lấy ra Roles từ Backend trên Project hoặc Domain tương ứng trả về cho user, nếu user không có bất kỳ roles nào thì trả về Failure(dịch vụ Assignment)
- Thu thập các Services và các Endpoints của các service đó (dịch vụ Catalog)
- Tổng hợp các thông tin về Identity, Resources, Assignment, Catalog ở trên đưa vào Token payload, tạo ra token sử dụng hàm uuid.uuid4().hex
- Lưu thông tin của Token vào SQL/KVS backend với các thông tin: ID, Expiration, Valid, UserID, Extra, trust_id

ID : 

Expiration :

Valid :

UserID :

Extra :

trust_id :

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

-----------------------

**PKI token**

**Quá trình tạo Token**

<img src="http://i.imgur.com/pi0xOpi.png">

- Người dùng gửi yêu cầu tạo token với các thông tin: User Name, Password, Project Name
- Keystone sẽ chứng thực các thông tin về Identity, Resource và Asssignment (định danh, tài nguyên, assignment)
- Tạo token payload định dạng JSON
- Sign JSON payload với Signing Key và Signing Certificate , sau đó được đóng gói lại dưới định dang CMS (cryptographic message syntax - cú pháp thông điệp mật mã)
- Bước tiếp theo, nếu muốn đóng gói token định dạng PKI thì convert payload sang UTF-8, convert token sang một URL định dạng an toàn. Nếu muốn token đóng gói dưới định dang PKIz, thì phải nén token sử dụng zlib, tiến hành mã hóa base64 token tạo ra URL an toàn, convert sang UTF-8 và chèn thêm tiếp đầu ngữ "PKIZ"
- Lưu thông tin token vào Backend (SQL/KVS)

**Quá trình xác thực Token**

<img src="http://i.imgur.com/b4G7u0R.png">

Vì id được generate bằng hàm hash của token nên quá trình validate token sẽ bắt đầu bằng việc sử dụng hàm hash để "băm" PKI toekn. Các bước sau đó (validate trong backend...) hoàn toàn giống với uuid.

**Quá trình thu hồi Token**

Hoàn toàn tương tự như tiến trình thu hồi UUID token

**Multiple Data Centers**

<img src="http://i.imgur.com/ky753ou.png">

PKI và PKIz không thực sự support mutiple data centers. Các backend database ở hai datacenter phải có quá trình đồng bộ hoặc tạo bản sao các PKI/PKIz token thì mới thực hiện xác thực và ủy quyền được.
