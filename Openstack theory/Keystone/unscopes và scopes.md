**unscoped token** không chứa project, uername, service... mà nó chỉ sử dụng để xác minh danh tính cho việc sau này tạo `scoped token` mà không cần lặp lại việc khai báo các thông tin ban đầu

Để có thế có được unscopes token cần:

- Bạn không được chỉ định phạm vi ủy quyền trong yêu cầu xác thực của mình (ví dụ: trên dòng lệnh phải có các đối số như --os-project-name hoặc --os-domain-id)

- Nhận dạng của bạn không được có "default project" liên quan đến nó

**Scoped token** chứa service, catalog, roles.... đã được ủy quyền. Scopes token chỉ sử dụng trong 1 miền duy nhất cho phép vận hành với quyền cao hơn mức người dùng trong 1 miền cụ thể
