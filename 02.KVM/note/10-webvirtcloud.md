# WebVirtCloud

### 1. Khái niệm

WebVirtCloud là web-based KVM management tool, mã nguồn mở, viết bằng Django(Python).(WebVirtCloud=Django app).

WebVirtCloud dùng để quản lý máy ảo KVM/QEMU, thường triển khai trên Linux và kết nối tới các hypervisor thông qua libvirt.


### 2. Chức năng chính

- **Quản lý host**: kết nối đến 1 hoặc nhiều kvm host qua SSH

- **Quản lý VMs**:
  - Tạo, start, stop, restart, delete VM.
  - Snapshot, clone, migrate
  - Console VNC/SPICE ngay trên web.

- **Quản lý Storage**:
  - Upload ISO, qcow2.
  - Tạo disk mới, attack/detach disk

- **Quản lý network**:
  - Tạo và cấu hình bridge, NAT, isolated network

- **Multi-user:**
  - Có quản lý user & role (admin, user)
  - Hữu ích khi nhiều người cùng sử dụng KVM