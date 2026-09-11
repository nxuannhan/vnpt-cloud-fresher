# Templates and Snapshots

### 1. Virtual Machine Templates

Một Template là một OS image đã được cài đặt và cấu hình sẵn, dùng để triển khai hàng loạt máy ảo mới một cách nhanh chóng mà không cần lặp lại quy trình cài đặt hệ điều hành và phần mềm thủ công.

**Quy trình tạo Template**

- Cài đặt & Cấu hình: Tạo máy ảo cơ sở và cài đặt các phần mềm cần thiết.

- Tẩy rửa / Niêm phong hệ thống (Sealing / Generalizing): Xóa toàn bộ các thông tin định danh đặc thù của máy ảo như SSH host keys, cấu hình mạng cố định (udev persistent network rules), địa chỉ MAC, tài khoản người dùng.

- Đổi tên máy ảo với tiền tố `template` hoặc hủy định nghĩa (undefine) máy ảo khỏi libvirt sau khi đã sao lưu lại tệp cấu hình XML.

Máy ảo đã làm thành Template tuyệt đối không được khởi động lại, nếu không sẽ làm mất trạng thái niêm phong.

**Triển khai máy ảo từ Template**

**Thin Method:** Máy ảo mới sử dụng template image làm base image (read-only) và tạo thêm một copy-on-write image (qcow2) mới lưu dữ liệu phát sinh.

- Ưu điểm: Triển khai nhanh, tiết kiệm dung lượng lưu trữ.

- Nhược điểm: VMs phụ thuộc vào template image gốc.

**Clone Method:** Tạo một bản sao độc lập hoàn toàn với template image gốc. VM mới hoạt động bình thường ngay cả khi xóa template image gốc.

- Nhược điểm: Dung lượng đĩa vật lý bằng đúng dung lượng của template image.

### 2. Virtual Machine Snapshots

Snapshot là một tệp lưu trữ trạng thái của hệ thống (bao gồm cấu hình và dữ liệu đĩa) tại một thời điểm nhất định, cho phép khôi phục (revert) máy ảo về đúng điểm đó khi có sự cố xảy ra.

**Phân loại snapshot**
| Đặc điểm | Internal Snapshot | External Snapshot |
| :------- | :---------------- | :---------------- |
| Vị trí lưu trữ | Lưu hoàn toàn bên trong duy nhất một tệp đĩa qcow2 | Tệp đĩa gốc trở thành chỉ đọc (backing_file), dữ liệu mới ghi vào tệp đĩa overlay_image mới|
| Định dạng đĩa hỗ trợ | only qcow2 | Hỗ trợ mọi định dạng image (raw, qcow2) |
| Quản lý GUI & CLI | Có sẵn giao diện đồ họa trên virt-manager và lệnh virsh dễ sử dụng | Quản lý thủ công qua CLI |
| Hạn chế | Máy ảo bị tạm dừng (paused) khi chụp; không hỗ trợ LVM pool | Quản lý revert và xóa/gộp chuỗi đĩa phức tạp hơn | 

**Các tùy chọn lệnh tạo Snapshot**

- `--atomic`: Đảm bảo thao tác chụp snapshot hoàn tất 100% hoặc thất bại mà không làm biến đổi dữ liệu, tránh nguy cơ hỏng tệp đĩa.

- `--quiesce`: Kích hoạt cơ chế đóng đóng tạm thời hệ thống tệp (fsfreeze/fsthaw) thông qua QEMU guest agent chạy bên trong máy ảo, giúp đảm bảo tính toàn vẹn khi chụp snapshot lúc máy ảo đang chạy.

**Quản lý External Snapshot**

- Khôi phục (revert) cần tắt máy ảo, dùng công cụ virt-xml hoặc chỉnh sửa trực tiếp tệp XML của máy ảo để trỏ đường dẫn đĩa boot về tệp snapshot mong muốn.


**Snapshot Best Practice**

- Snapshot không phải là phương án sao lưu (Backup), không phụ thuộc vào snapshot như một giải pháp backup độc lập.

- Không giữ Snapshot lâu: Gộp và xóa snapshot ngay khi xác nhận không còn cần khôi phục lại, vì giữ snapshot lâu sẽ làm suy giảm hiệu năng đĩa của máy ảo.

- Ưu tiên External Snapshots trong môi trường sản xuất vì ít nguy cơ hỏng dữ liệu hơn Internal Snapshot.

- Cài đặt Guest Agent bên trong máy ảo và bật cờ `--quiesce` cũng như `--atomic` khi chụp snapshot

### 3. Virt-sysprep

`virt-sysprep` là một công cụ thuộc bộ **libguestfs** trong Linux, được dùng để chuẩn bị(system preparation) một máy ảo hoặc một disk image trước khi dùng làm template hoặc clone.

  - Xóa **hostname**
  - Xóa **SSH host keys**
  - Xóa **MAC address**
  - Xóa **log files**(messgaes, secure, journal,...)
  - Reset **user account password** nếu cần.
  - Reset **machine-id**

chạy bằng lệnh:
```bash
virt-sysprep -a /var/lib/libvirt/images/centos-stream9.qcow2
```
- `-a` chỉ định file disk image
- Công cụ sẽ chỉnh sửa trực tiếp vào disk image( không cần VM đang chạy).

Sau khi chạy lệnh có thể dùng `virt-clone` hoặc `virt-install` để tạo VM mới từ image này.

## 5. Lab tạo template và cài đặt VM từ template
### 5.1 Tạo template
Cài đặt 1 VM trên host KVM. Cài đặt các gói cần thiết để dùng làm template.

Shutdown VM:

![altimage](../images/Screenshot_37.png)

Cài đặt gói `libguestfs-tools-c` trên KVM host:
```bash
sudo apt install libguestfs-tools
```
Sử dụng `virt-sysprep` để loại bỏ các thông tin cấu hình như UUID, MAC, ... đồng thời niêm phong và biến máy ảo thành template
```bash
virt-sysprep -d testvm
```

![altimage](../images/Screenshot_50.png)

Backup file xml của template bằng lệnh `dumpxml`
```bash
virsh dumpxml testvm > /root/template.xml
```
Undefine máy ảo
```bash
virsh undefine testvm
```

![altimage](../images/Screenshot_51.png)
### 5.2 Sử dụng template
Copy file image template sang host KVM02:
```bash
scp -v /root/template.xml ubuntu1@192.168.70.124:/root/
```
Tạo ra file image mới với định dạng qcow2 để file template làm file backups bằng câu lệnh
```bash
qemu-img create \
-f qcow2 \
-b /mnt/kvm-nfs/testvm.qcow2 \
-F qcow2 \
/var/lib/libvirt/images/vm1.qcow2
```
Kiểm tra xem file mới tạo ra đã được chỉ tới file backup của nó hay chưa bằng câu lệnh:
```bash
qemu-img info /var/lib/libvirt/images/vm1.qcow2
```
![altimage](../images/Screenshot_52.png)

Dùng virt-clone để tạo ra máy ảo mới từ file XML
**Lưu ý**: nếu bạn dùng phiên bản khác nhau thì cần sửa lại file xml
```bash
virt-clone \
--original-xml /root/template.xml \
-f /var/lib/libvirt/images/vm1.qcow2 \
-n vm1 \
--preserve-data
```

Khởi động máy
![altimage](../images/Screenshot_53.png)