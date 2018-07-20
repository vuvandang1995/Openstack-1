# Giới thiệu về chức năng các giải mạng sử dụng trong openstack

### Cơ bản trong openstack có tối thiểu 3 dải mạng

- Dải external: Cho phép các máy ảo kết nối ra internet: có 2 phương pháp sử dụng cho dải này là flat hoặc vlan, các máy ảo sẽ được chia vlan nếu sử dụng phương pháp vlan hoặc các máy ảo sẽ sử dụng luôn ip thuộc giải provider

- Dải datavm: Sử dụng để cho phép các máy ảo giao tiếp trao đổi dữ liệu với nhau hoặc tải dữ liệu trên internet , sử dụng để ssh vào các **`máy ảo`** bằng việc sử dụng một router ảo trên controller, router ảo này sẽ có 1 interface thuộc dải datavm và 1 interface thuộc dải provider, khi ssh vào sẽ đi qua dải provider thông qua router ảo được floating ip vào trong dải datavm và việc floating ip này sẽ gán cho máy ảo nào muốn mạng external truy cập vào. Với việc sử dụng mô hình self service sẽ cho phép nhiều dải ip đi trong dải datavm này bằng các công nghệ vxlan hoặc gre. 

- Dải mngt: Sử dụng để quản lý các **`compute`**, controller, dải này luôn phải đi được internet để ssh vào các **`compute`**, đồng thời cũng để tải các package để cài đặt. Có thể đi chung với giải provider hoặc tạo 1 đường mạng đi internet riêng

- Dải API: Sử dụng để thao tác quản lý với các project bên trong openstack như nova, neutron, cinder...



### Trong openstack việc đặt ip sẽ được gắn mỗi ip cho 1 port cố định việc đặt trùng ip giữa các máy ảo sẽ không làm ảnh hưởng đến hoạt động đến hệ thống, và các máy ảo đặt trùng ip sẽ không thể kết nối được với các máy ảo khác hoặc bên ngoài
