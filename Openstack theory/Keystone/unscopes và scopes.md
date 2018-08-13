**unscoped token** không chứa project, uername, service... mà nó chỉ sử dụng để khai báo các  user, project... (ví dụ như neutron, nova,...) cho việc sau này tạo `scoped token` để phân quyền mà không cần lặp lại việc khai báo các thông tin ban đầu

Để có thế có được unscopes token cần:

- Bạn không được chỉ định phạm vi ủy quyền trong yêu cầu xác thực của mình (ví dụ: trên dòng lệnh phải có các đối số như --os-project-name hoặc --os-domain-id)

- Nhận dạng của bạn không được có "default project" liên quan đến nó

**Scoped token** Sử dụng unscoped để lấy thông tin về service catalog,  user/pass, role, domain, project... để thực hiện việc phân quyền


- Project Scoped token sử dụng cho  việc phân quyền cho project

- Domain Scoped token sử dụng cho việc phân quyền cho domain

Các scoped token sau khi được tạo ra sẽ được đưa vào payload tạo ra file .json, từ file .json này 

- uuid sẽ sử dụng hàm uuid.uuid4().hex để mã hóa tạo ra ID token 

- fernet sẽ sử dụng các key để mã hóa tạo ra ID token (nhưng khi giải mã sẽ chỉ giải mã tạo ra file .jon chỉ đủ để xác thực chứ không tạo ra hoàn chỉnh như ban đầu.
