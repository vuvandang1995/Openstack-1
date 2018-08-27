# Giới thiệu về token

Vào những ngày đầu, Keystone hỗ trợ UUID token. Đây là loại token gồm 32 kí tự được dùng để xác thực và ủy quyền. Lợi ích mà loại token này mang lại đó là nó nhỏ gọn và dễ sử dụng, nó có thể được thêm trực tiếp vào câu lệnh cURL. Tuy nhiên nó lại không thể chứa đủ thông tin để thực hiện việc ủy quyền. Các services của OpenStack sẽ luôn phải gửi lại token này lại cho Keystone để xác thực xem hành động có hợp lệ không. Điều này khiến Keystone trở thành trung tâm cho mọi hoạt động của OpenStack.

Trong nỗ lực kiếm tìm một loại token mới để khắc phục những nhược điểm của UUID token, nhóm phát triển đã tạo ra PKI token. Token này chứa đủ thông tin để xác thực và ủy quyền và đồng thời nó cũng chứa cả danh mục dịch vụ. Bên cạnh đó, token này được gán vào và các dịch vụ có thể cache lại nó để dử dụng cho đến khi nó hết hiệu lực hoặc bị hủy. Loại token này vì thế cũng khiến lượng traffic tới Keystone server ít hơn. Tuy nhiên kích cỡ của nó có thể lên tới 8k và điều này làm việc gán vào HTTP header trở nên khó khăn hơn. Rất nhiều các web server mặc định sẽ không cho phép điều này nếu chưa được config lại. Thêm vào đó, loại token này cũng rất khó được sử dụng trong câu lệnh cURL. Vì những hạn chế mà các nhà phát triển đã cố gắng sửa đổi và ra mắt một phiên bản khác đó là PKIz, tuy nhiên theo đánh giá của cộng đồng thì loại token này vẫn rất lớn về kích thước.

Điều này buộc Keystone team phải đưa ra một loại mới và đó là Fernet token. Fernet token khá nhỏ (255 kí tự) tuy nhiên nó lại chưa đủ thông tin để ủy quyền. Bên cạnh đó, việc nó chứa đủ thông tin cũng không khiến token database phải lưu dữ liệu token nữa (vấn đề này xảy ra ở UUID) Các nhà vận hành thường phải dọn dẹp Keystone token database để hệ thống của họ hoạt động ổn định. Mặc dù vậy, Fernet token có nhược điểm đó là symmetric keys được dùng để tạo ra token cần được phân phối và xoay vòng. Các nhà vận hành cần phải giải quyết vấn đề này, tuy nhiên họ có vẻ thích thú với việc này hơn là sử dụng những loại token khác.

## 1. UUID Tokens

<img src="https://i.imgur.com/rB3Ferh.png">

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

## 2. Fernet Tokens 

<img src="https://i.imgur.com/rB3Ferh.png">

Đây là loại token mới nhất, nó được tạo ra để khắc phục những hạn chế của các loại token trước đó. Thứ nhất, nó khá nhỏ với khoảng 255 kí tự, lớn hơn UUID nhưng nhỏ hơn rất nhiều so với PKI. Token này cũng chứa vừa đủ thông tin để cho phép nó không cần phải được lưu trên database.

Fernet tokens chứa một lượng nhỏ dữ liệu ví dụ như thông tin để nhận diện người dùng, project, thời gian hết hiệu lực,...Nó được sign bởi symmetric key để ngăn ngừa việc giả mạo. Cơ chế hoạt động của loại token này giống với UUID vì thế nó cũng phải được validate bởi Keystone.

Một trong những vấn đề của loại token này đó là nó dùng symmetric key để mã hóa token và các keys này buộc phải được phân phối lên tất cả các region của OpenStack. Thêm vào đó, keys cũng cần được rotated.

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

**Fernet Key rotation**

Hình dưới đây mô tả quá trình rotate fernet key:

<img src="http://i.imgur.com/kXZGhUZ.png">

Giả sử triển khai hệ thống cloud với keystone ở hai bên us-west và us-east. Cả hai repo này đều được thiết lập với 3 fernet key như sau:

``` sh
$ ls /etc/keystone/fernet-keys
0 1 2
```

Ở đây 2 sẽ trở thành Primary Key để mã hóa fernet token. Fernet tokens có thể được mã hóa sử dụng một trong 3 key theo thứ tự là 2, 1, 0. Giờ ta quay vòng fernet key bên us-west, repo bên này sẽ đươc thiết lập như sau:

``` sh
$ ls /etc/keystone/fernet-keys
0 1 2 3
```

Với cấu hình như trên, bên us-west, 3 trở thành Primary Key để mã hóa fernet token. Khi keystone bên us-west nhận token từ us-east (mã hóa bằng key 2), us-west sẽ xác thực token này, giải mã bằng 4 key theo thứ tự 3, 2, 1, 0. Keystone bên us-east nhận fernet token từ us-west (mã hóa bằng key 3), us-east xác thực token này vì key 3 bên us-west lúc này trở thành staged key (0) bên us-east, keystone us-east giải mã token với 3 key theo thứ tự 2, 1, 0.
Có thể cấu hình giá trị max_active_keys trong file /etc/keystone.conf để quy định tối đa số key tồn tại trong keystone. Nếu số key vượt giá trị này thì key cũ sẽ bị xóa.

**Kế hoạch cho vấn đề rotated keys**

Khi sử dụng fernet tokens yêu cầu chú ý về thời hạn của token và vòng đời của khóa. Vấn đề nảy sinh khi secondary keys bị remove khỏi key repos trong khi vẫn cần dùng key đó để giải mã một token chưa hết hạn (token này được mã hóa bởi key đã bị remove).
Để giải quyết vấn đề này, trước hết cần lên kế hoạch xoay khóa. Ví dụ bạn muốn token hợp lệ trong vòng 24 giờ và muốn xoay khóa cứ mỗi 6 giờ. Như vậy để giữ 1 key tồn tại trong 24h cho mục đích decrypt thì cần thiết lập max_active_keys=6 trong file keytone.conf (do tính thêm 2 key đặc biệt là primary key và staged key ). Điều này giúp cho việc giữ tất cả các key cần thiết nhằm mục đích xác thực token mà vẫn giới hạn được số lượng key trong key repos (/etc/keystone/fernet-keys/).

``` sh
token_expiration = 24
rotation_frequency = 6
max_active_keys = (token_expiration / rotation_frequency) + 2
```

**Các trường của Fernet token**

- Fernet Format Version (0x80) - 8 bits, biểu thị phiên bản của định dạng token
- Current Timestamp – số nguyên 64-bit không dấu, chỉ nhãn thời gian tính theo giây, tính từ 1/1/1970, chỉ ra thời điểm token được tạo ra.
- Initialization Vector (IV) – key 128 bits sử dụng mã hóa AES và giải mã Ciphertext
- Ciphertext: là keystone payload kích thước biến đổi tùy vào phạm vi của token. Cụ thể hơn, với token có phạm vi project, Keystone Payload bao gồm: version, user id, method, project id, expiration time, audit ids
- HMAC: 256-bit SHA256 HMAC (Keyd-Hash Messasge Authentication Code) - Mã xác thực thông báo sử dụng hàm một chiều có khóa với signing key kết nối 4 trường ở trên.

**Token Generation Workflow**

<img src="http://i.imgur.com/9Ds3E2Y.png">

Với key và message nhận được, quá trình tạo fernet token như sau:
- Ghi thời gian hiện tại vào trường timestamp
- Lựa chọn một IV duy nhất
- Xây dựng ciphertext:
  - Padd message với bội số là 16 bytes (thao tác bổ sung một số bit cho văn bản trong mã hóa khối AES)
  - Mã hóa padded message sử dụng thuật toán AES 128 trong chế độ CBC với IV đã chọn và encryption-key được cung cấp
- Tính toán trường HMAC theo mô tả trên sử dụng signing-key mà người dùng được cung cấp
- Kết nối các trường theo đúng format token ở trên
- Mã hóa base64 toàn bộ token

**Token validation workflow**

<img src="http://i.imgur.com/Fu8n1Cm.png">


- Gửi yêu cầu xác thực token với API GET v3/auth/tokens
- Khôi phục lại padding, trả lại token với padding chính xác
- Decrypt sử dụng Fernet Keys để thu lại token payload
- Xác định phiên bản của token payload. (Unscoped token: 0, Domain scoped payload: 1, Project scoped payload: 2 )
- Tách các trường của payload để chứng thực. Ví dụ với token trong tầm vực project gồm các trường sau: user id, project id, method, expiry, audit id
- Kiểm tra xem token đã hết hạn chưa. Nếu thời điểm hiện tại lớn hơn so với thời điểm hết hạn thì trả về thông báo "Token not found". Nếu token chưa hết hạn thì chuyển sang bước tiếp theo
- Kiểm tra xem token đã bị thu hồi chưa. Nếu token đã bị thu hồi (tương ứng với 1 sự kiện thu hồi trong bảng revocation_event của database keystone) thì trả về thông báo "Token not found". Nếu chưa bị thu hồi thì trả lại token (thông điệp phản hồi thành công HTTP/1.1 200 OK)

**Multiple data centers**

Vì Fernet key không cần phải được lưu vào database nên nó có thể hỗ trợ multiple data center. Tuy nhiên keys sẽ phải được phân phối tới tất cả các regions.

## 3 PKI, PKIz token

**PKI**
<img src="http://7xp2eu.com1.z0.glb.clouddn.com/pki.png">
**PKIz**
<img src="http://7xp2eu.com1.z0.glb.clouddn.com/pkiz.png">

- Token sẽ chứa toàn bộ các validation response của keystone. Do đó token sẽ chứa một lượng lớn các thông tin như nó được issue lúc nào, hết hạn lúc nào, thuộc về user nào, thông tin project, domain, role, thông tin về user, service cataloge. v.v... Các thông tin được mô tả trong một cấu trúc JSON và được ký bằng một CMS(Cryptographic Message syntax). Với PKIz thì các thông tin được nén sử dụng zlib để nén. Khi sử dụng token này thì không cần phải quay lại về keystone để verify lại nữa.
- Để có thể truyền token qua giao thức HTTP cần phải mã hóa nó dưới dạng base64. Với một yêu cầu đơn giản, một endpoint và catalog, kích cỡ xấp xỉ của nó có thể lên đến 1700bytes. Với một hệ thống lớn với nhiều endpoint, PKI token có thể lớn tới cỡ 8KB, ngay cả khi được nén lại (PKIz) nó vẫn thường không thể vừa với các HTTP header của các webserver thông thường. 
- Mặc dù PKI và PKIz token có thể cached nhưng nó cũng có những nhược điểm như khó có thể config keystone để sử dụng loại token này vì nó phải sử dụng certificate được tạo từ nhà cung cấp certificate tin cậy, và kích thước nó quá lớn sẽ gây ảnh hưởng đến các openstack service khác về hiệu năng. Keystone vẫn phải lưu trữ PKI ở backend nhằm mục đích ví dụ như tạo danh sách các revoked token  

## 4 Cách Horizon dùng token

- Tokens được sử dụng cho mỗi lần log in của user
- Horizon lấy unscoped token cho user và sau dựa vào các request để cung cấp các project scoped token.
- Token có thể được tái sử dụng bằng cách lưu lại sau mỗi session.
- Các method để lưu token:
  - Local memory cache
  - Cookie backend
  - Memcache
  - database
  - Cached Database

#### Cookie backend

- Là phương thức mặc định của devstack
- Token được lưu trên cookie của trình duyệt
- Có khả năng co giãn cao
- Khi cookie đầy, dễ dẫn tới tình trạng không xác thực được user -> back to login

#### Memcache backend

- Cho phép lưu một lượng lớn token
- Token được lưu ở phía server
- Yêu cầu cấu hình memcached
- Có thể sử dụng với backing DB

