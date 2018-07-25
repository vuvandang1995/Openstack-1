# Vấn đề về volume

Khi tạo 1 volume trống để sử dụng attach cho máy ảo,

- Tạo partition cần phân vùng
- Tạo LVM
- format định dạng
- mount vào 
 -có thể detach volume từ máy ảo này và attach sang máy ảo khác và sử dụng được luôn , dữ liệu cũ bên trong volume vẫn còn nguyên


---------------
# Vấn đề snapshot

- Khi snapshot máy ảo có volume attach nó chỉ snapshot image

--------------

# Vấn đề network

- Ta có thể tạo fixed ip bằng cách click vào network mà ta đã tạo -> port -> tạo port tương ứng với ip fixed, sau đó detach interface và attach interface vào máy ảo là máy ảo có ip fixed
