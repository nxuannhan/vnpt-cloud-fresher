# Virtual Switch

Trong hạ tầng ảo hóa KVM/libvirt, **Virtual Switch (Switch ảo)** hay **Bridge** đóng vai trò là thành phần chuyển mạch trung tâm, kết nối các máy ảo (VM) với nhau và với mạng vật lý bên ngoài.

### 1. Nguyên lý hoạt động 

**Mô phỏng Switch vật lý:** Switch ảo hoạt động tương tự như một switch cứng vật lý nhưng cung cấp số lượng cổng ảo không giới hạn để gắn giao diện mạng của các máy ảo. Nó tự động học địa chỉ MAC từ các gói tin đi qua, lưu giữ trong bảng MAC và đưa ra quyết định chuyển tiếp frames (frames forwarding) dựa trên bảng MAC này.

**Kết nối qua thiết bị TAP:** Các card mạng ảo (vNIC) của máy ảo nối tới cổng của switch ảo thông qua một thiết bị mạng ảo hóa Layer 2 trong nhân Linux gọi là **TAP device**. Thiết bị TAP đóng vai trò như "sợi cáp mạng ảo" truyền tải các frame Ethernet giữa máy ảo và switch.

**Quản lý bởi libvirt:** libvirt tự động khởi tạo và điều phối các switch ảo trên máy chủ (như `virbr0` cho mạng NAT mặc định hoặc `virbr1` cho mạng isolated).

---

### 2. Hai giải pháp Virtual Switch trong KVM

**A. Linux Bridge (Switch ảo truyền thống)**

**Đặc điểm:** Là giải pháp chuyển mạch mặc định của Linux kernel, được quản lý thông qua bộ công cụ `bridge-utils` (`brctl`) hoặc lệnh `ip`.

**Ưu điểm:** Đơn giản, nhẹ, tích hợp sẵn trong nhân Linux và dễ cấu hình.

**Hạn chế:** Chỉ hoạt động ở Layer 2, không hỗ trợ các giao thức đường hầm (tunneling), không tương thích với OpenFlow và khả năng mở rộng hạn chế đối với các môi trường đám mây quy mô lớn.

**B. Open vSwitch (OVS - Switch ảo thông minh cho SDN)**

Open vSwitch là một switch ảo mã nguồn mở hỗ trợ chuẩn **OpenFlow**, được thiết kế chuyên biệt cho môi trường ảo hóa KVM và điện toán đám mây:

* **Khả năng kiểm tra gói tin nâng cao:** Hỗ trợ xử lý và khớp gói tin linh hoạt từ **Layer 2 đến Layer 4**.

* **Kiến trúc hai tầng:** Tách biệt rõ ràng giữa mặt phẳng dữ liệu (**OVS Kernel Module** ở Kernel Space) và mặt phẳng điều khiển (**ovs-vswitchd** và **ovsdb-server** ở User Space).

![OVS Architechture](../img/ovs-architechture.png)

* **Tương thích SDN:** Hỗ trợ giao thức OpenFlow để kết nối và chịu sự điều khiển tập trung từ bộ điều khiển SDN (như **OpenDaylight**).

* **Phân chia mạng VLAN:** Hỗ trợ mô hình VLAN chuẩn 802.1q (`access`, `trunk`, `portgroup`) giúp phân tách lưu lượng mạng an toàn.

* **Mạng đường hầm (Overlay Networks):** Cho phép kết nối các switch OVS trên nhiều máy chủ KVM vật lý khác nhau qua các giao thức **VXLAN**, **GRE**, STT, Geneve.

* **Quản lý QoS và Giám sát mạng:** Hỗ trợ giới hạn băng thông (traffic policing/shaping), sao chép cổng (**Port Mirroring / SPAN**), và giám sát lưu lượng qua NetFlow/sFlow.

---

### **3\. Tích hợp với KVM và libvirt**

Để gắn giao diện mạng của máy ảo KVM vào một switch Open vSwitch thay vì Linux Bridge truyền thống, libvirt hỗ trợ khai báo thêm thẻ `<virtualport type='openvswitch'/>` bên trong cấu hình XML của máy ảo hoặc của mạng ảo libvirt.