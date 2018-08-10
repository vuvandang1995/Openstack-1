# Các file cấu hình của keystone

`less /etc/keystone/default_catalog.templates` file catalog trong keystone, là nơi chứa Chứa URLs và endpoints của các services khác nhau , đường dẫn cho phép các tenants (project) biết nơi nào có thể gửi yêu cầu tạo máy ảo hoặc lưu trữ dữ liệu.  

`/etc/openstack-dashboard` là nơi chứa các file .json là các file chứa các (role) bao gồm các targets và rules tương ứng cho các user được cấu hình 
