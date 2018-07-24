
`/var/lib/glance/images` lưu image khi boot từ local lên controller để tạo imaga trên dashboard
`/var/lib/nova/instances/_base`  đây là nơi lưu hệ điều hành máy ảo máy ảo trên compute

Nếu boot máy ảo từ local sẽ tạo ra máy ảo chứa các file disk, hệ điều hành,  console.log `/var/lib/nova/instances` và hệ điều hành sẽ lưu ở `/var/lib/nova/instances/_base`

Nếu boot từ volume thì sẽ lưu ở controller
