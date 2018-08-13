**unscoped token** không chứa project, uername, service... mà nó chỉ sử dụng để khai báo các danh tính của user (ví dụ như neutron, nova,...) cho việc sau này tạo `scoped token` để phân quyền mà không cần lặp lại việc khai báo các thông tin ban đầu

Để có thế có được unscopes token cần:

- Bạn không được chỉ định phạm vi ủy quyền trong yêu cầu xác thực của mình (ví dụ: trên dòng lệnh phải có các đối số như --os-project-name hoặc --os-domain-id)

- Nhận dạng của bạn không được có "default project" liên quan đến nó

**Scoped token** chứa service catalog, roles, chi tiết về project mà ta có thể ủy quyền. 

- Project Scoped token sử dụng cho  việc phân quyền trong 1 project

- Domain Scoped token sử dụng cho việc phân quyền trong 1 domain

