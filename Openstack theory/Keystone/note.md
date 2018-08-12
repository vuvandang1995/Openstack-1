# Ghi chú token

có 3 loại :

- uuid:  là loại token gồm 32 kí tự được dùng để xác thực và ủy quyền
<ul>
<ul>
<li>Ưu điểm: Nhỏ gọn và dễ sử dụng, nó có thể được thêm trực tiếp vào câu lệnh cURL.</li>
<li>Nhược điểm: Không thể chứa đủ thông tin để thực hiện việc ủy quyền và Các services của OpenStack sẽ luôn phải gửi lại token này lại cho Keystone để xác thực xem hành động có hợp lệ không Điều này khiến Keystone trở thành trung tâm cho mọi hoạt động của OpenStack, và UID token bược phải được lưu lại trong một backend (thường là database sẽ khiến database bị đầy)Nó cũng có thể được loại bỏ bằng các sử dụng DELETE request với token id. Tuy nhiên nó sẽ không thực sự bị loại bỏ khỏi backend mà chỉ được đánh dấu là đã được loại bỏ.
</ul>
</ul>

- 
<ul>
<ul>
<li>
</ul>
</ul>
