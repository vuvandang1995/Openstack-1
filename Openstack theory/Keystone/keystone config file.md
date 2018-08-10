


`less /etc/keystone/default_catalog.templates` file catalog trong keystone, là nơi chứa Chứa URLs và endpoints của các services khác nhau , đường dẫn cho phép các tenants (project) biết nơi nào có thể gửi yêu cầu tạo máy ảo hoặc lưu trữ dữ liệu.  





## 1. Giới thiệu và ví dụ về file policy.json 

`/etc/openstack-dashboard` là nơi chứa các file .json là các file chứa các (role) bao gồm các targets và rules tương ứng cho các user được cấu hình

Mỗi một dịch vụ OpenStack như Identity, Compute hay Networking thì đều có riêng role-based access policies. Chúng xác định user nào có quyền truy cập tới objects nào bằng cách nào, những policies này đều được định nghĩa ở trong file `policy.json`

Mỗi khi API gọi tới một dịch vụ nào đó, policy engine của service ấy sẽ sử dụng các policy đã được define để xác định xem API call đó có hợp lệ hay không. Bất cứ sự thay đổi nào đối với file `policy.json` sẽ có tác dụng  ngay lập tức, cho phép các policy mới có thể được thực thi ngay khi service vẫn đang chạy.

File `policy.json` là một file text theo format json (Javascript Object Notation). Mỗi một policy được định nghĩa trong 1 dòng theo form : `"<target>" : "<rule>"`.

`Target` ở đây được hiểu là các actions, là các API call để tạo máy ảo, gán volume... Các actions đã được xác định sẵn. Ví dụ như API calls để lấy danh sách máy ảo, volumes và network được định nghĩa ở file `/etc/nova/policy.json` là `compute:get_all, volume:get_all và network:get_all`.

**Ví dụ**

- Một rule đơn giản:

`"compute:get_all" : ""`

Target `compute:get_all` dùng để lấy danh sách các máy ảo, rule `""` nghĩa là `always`, nó cho phép tất cả user thực hiện chức năng này.

- Bạn có thể từ chối quyền sử dụng API:

`compute:shelve": "!`

Dấu chấm than có nghĩa là `never` hoặc `nobody`, nó sẽ hủy đi Compute API “shelve an instance”.

Có một vài APIs chỉ có thể được gọi bởi admin, nó được diễn tả bằng rule: `"role:admin"`. Policy dưới đây sẽ đảm bảo chỉ có admin mới có thể tạo mới user trong database của keystone:

`"identity:create_user" : "role:admin"`

Rules cũng có thể dùng với các câu lệnh boolean như `not` hay `and`, `or`.

## 2. Cấu trúc 

File `policy.json` bao gồm các policies và alias theo form `target:rule` hoặc `alias:definition`, ngăn cách nhau bởi dấu phẩy:

``` sh
{
      "alias 1" : "definition 1",
      "alias 2" : "definition 2",
      ...
      "target 1" : "rule 1",
      "target 2" : "rule 2",
      ....
}
```

Targets là các APIs được viết dưới dạng `"service:API"` hoặc đơn giản chỉ là `"API"`. Ví dụ: `"compute:create"` hoặc `"add_image"`.

Rule sẽ xác định API call có hợp lệ hay không. Rules có thể là:

- luôn đúng, action sẽ luôn hợp lệ. Nếu nó được diễn tả bằng `""`, `[]` hoặc `"@"`.
- luôn sai, action sẽ không bao giờ được chấp nhận, được diễn tả bằng `"!"`.
- a special check
- sự so sánh giữa hai giá trị
- boolean expressions based on simpler rules

Special checks là:

- `<role>:<role name>`
- `<rule>:<rule name>`
- `http:<target URL>`

So sánh giữa hai giá trị là:

`"value1 : value2"`

Các giá trị được phép ở đây bao gồm:

- Strings, numbers, true, false
- API attributes (`project_id, user_id, domain_id`)
- target object attributes
- the flag `is_admin`

Target object attributes là các fields từ object description trong database. Ví dụ như API `"compute:start"` có object là máy ảo để khởi động.

`is_admin` thể hiện qyền admin đã được xác thực bằng token.

Alias là tên ngắn gọn của những rule phức tạp hoặc khó hiểu. Nó được define giống với policy, một khi đã được định nghĩa, người dùng có thể dùng keyword `rule` để sử dụng nó trong file policy.

# 3. Các câu lệnh để quản lí role trong keystone

Tạo role:

`openstack role create ducnm37`

Gán role ducnm37 cho user duc vào project admin

`openstack role add --user duc --user-domain default --project admin --project-domain default ducnm37`

Lúc này ta cần config file policy.json để user thaonv thực hiện 1 số quyền, ở đây mình muốn tạo được domain mới. Đầu tiên tìm tới file /etc/keystone/policy.json, tìm đến dòng "identity:create_domain": "rule:admin_required",.

Ở đây sửa thành "identity:create_domain": "rule:admin_required or role:ducnm37",.

Để test, ta tiến hành thiết lập các biến môi trường:
```
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=duc
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
```
