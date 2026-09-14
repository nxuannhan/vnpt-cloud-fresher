# KVM Migration

**Di trú máy ảo (VM Migration)** là tính năng cho phép di chuyển máy ảo đang hoạt động hoặc đã tắt từ máy chủ vật lý (host) này sang máy chủ vật lý khác với thời gian gián đoạn dịch vụ rất ngắn hoặc bằng không.

**Lợi ích của di trú máy ảo**

* **Tăng thời gian hoạt động (Uptime):** Cho phép bảo trì, nâng cấp phần cứng hoặc phần mềm máy chủ mà không làm gián đoạn ứng dụng đang chạy.

* **Tiết kiệm năng lượng:** Cho phép dồn (consolidate) các máy ảo về một số ít máy chủ vào giờ thấp điểm, sau đó tắt bớt các máy chủ không sử dụng để tiết kiệm điện.

* **Cân bằng tải linh hoạt:** Dễ dàng điều chuyển tài nguyên máy ảo giữa các máy chủ vật lý.

### 1. Cold Migration (Offline Migration/Di trú ngoại tuyến)

- Máy ảo ở trạng thái tắt (shut off) hoặc tạm dừng (suspended).

- Libvirt chỉ cần sao chép tệp cấu hình XML của máy ảo từ host nguồn sang host đích và đảm bảo hai bên truy cập cùng kho lưu trữ đĩa ảo. Sau đó, máy ảo sẽ được khởi động lại tại host đích.


```
virsh migrate --offline --verbose --persistent <tên_vm> qemu+ssh://<host_đích>/system
```

### 2. Live Migration (online Migration/Di trú trực tuyến)

- Máy ảo được di chuyển ngay trong khi đang chạy và xử lý yêu cầu của người dùng.

- Quá trình di chuyển diễn ra hoàn toàn trong suốt (invisible) với người dùng. KVM thực hiện live migration độc lập với hệ điều hành khách (Guest OS) và phần cứng (có thể di trú giữa các máy chủ sử dụng CPU AMD và Intel).

```
virsh migrate --live <tên_vm> qemu+ssh://<host_đích>/system --verbose --persistent
```

Khi thực hiện Live Migration, quá trình diễn ra ngầm qua 5 bước:

1. **Chuẩn bị host đích:** Libvirt nguồn gửi thông tin cấu hình máy ảo tới Libvirt đích. QEMU tại host đích khởi động máy ảo ở chế độ tạm dừng (pause mode) và mở cổng TCP để lắng nghe.
2. **Truyền bộ nhớ RAM:** QEMU chuyển toàn bộ dung lượng RAM của máy ảo sang host đích. Trong lúc máy ảo vẫn đang chạy ở nguồn, các trang bộ nhớ bị thay đổi (**dirty pages**) sẽ tiếp tục được truyền liên tục cho đến khi số trang bị thay đổi giảm xuống dưới ngưỡng thấp (hoặc đạt giới hạn thời gian downtime).
3. **Tạm dừng máy ảo ở nguồn:** Khi đạt ngưỡng, QEMU tạm dừng máy ảo ở host nguồn và đồng bộ đĩa ảo.
4. **Truyền trạng thái thiết bị:** Chuyển nốt trạng thái các thiết bị ảo (device state) và số trang dirty page còn lại.
5. **Khôi phục hoạt động tại đích:** Khôi phục máy ảo chạy tiếp tại host đích. Card mạng ảo gửi tín hiệu ARP thông báo (gratuitous ARP) để các switch mạng cập nhật bảng MAC và chuyển hướng lưu lượng mạng sang host mới.

Để thiết lập di trú máy ảo thành công trong môi trường production (production), hệ thống cần đáp ứng các điều kiện:

* **Kho lưu trữ chia sẻ (Shared Storage):** Đĩa ảo của máy ảo phải nằm trên kho lưu trữ chia sẻ (như NFS, iSCSI, FC, hoặc GlusterFS) có cùng tên storage pool và đường dẫn trên cả hai host.

* **Cách ly mạng:** Khuyên dùng đường mạng riêng cho lưu lượng di trú để tránh chiếm dụng băng thông mạng của máy ảo và đảm bảo an toàn thông tin.

* **Cơ chế khóa đĩa (Locking mechanism):** Sử dụng `virtlockd` (`lockd`) hoặc `sanlock` để ngăn chặn việc vô tình bật trùng máy ảo ở cả hai host gây hỏng hệ thống tệp.

* **Đồng bộ thời gian &amp; Tên miền:** Thời gian trên các host phải được đồng bộ qua NTP/PTP, và phân giải được tên miền host qua DNS.