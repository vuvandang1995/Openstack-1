# vấn đề về volume

Khi tạo 1 volume trống để sử dụng attach cho máy ảo,

- Cần phân vùng
- Tạo LVM
- format LVM
- mount vào 
- không thể umount do đang gắn với máy ảo chỉ có thể detach

- Khi snapshot máy ảo có volume attach nó chỉ snapshot image
