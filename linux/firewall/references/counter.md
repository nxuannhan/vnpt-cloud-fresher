Trong `iptables`, **counter** là bộ đếm cho biết có bao nhiêu packet và bao nhiêu byte đã đi qua một rule hoặc chain.

Có 2 loại thông tin chính:

```text
PKTS   = số lượng packets
BYTES  = tổng số bytes
```

Ví dụ:

```bash
iptables -L -v
```

Có thể thấy:

```text
pkts bytes target  prot opt in  out  source        destination
1500  900K ACCEPT  tcp  --  eth0 *    192.168.1.0/24  10.0.0.5
```

Nghĩa là:

```text
1500 packets
900K bytes
```

đã **match rule** đó.

### Counter của rule vs chain

Điểm quan trọng: `iptables` thực tế lưu **counter cho từng rule**, không phải một counter độc lập kiểu "chain counter".

Ví dụ:

```text
INPUT chain

Rule 1 → ACCEPT → 100 packets
Rule 2 → DROP   → 50 packets
Rule 3 → ACCEPT → 200 packets
```

Có thể hiểu tổng traffic đã match các rule là:

```text
100 + 50 + 200 = 350 packets
```

### Counter dùng để làm gì?

Counter rất hữu ích để:

* Kiểm tra rule có thực sự được sử dụng không.
* Theo dõi traffic.
* Debug firewall.
* Phát hiện rule không bao giờ match.
* Thống kê số packet bị `DROP`, `ACCEPT`, v.v.

Ví dụ:

```bash
iptables -L -v -n
```

Trong đó:

```text
-v → hiển thị packet/byte counters
-n → không resolve IP/port thành hostname/service name
```

### Reset counter

Có thể reset counter của một rule bằng:

```bash
iptables -Z
```

Hoặc reset counter của một chain:

```bash
iptables -Z INPUT
```

Sau khi reset:

```text
PKTS  = 0
BYTES = 0
```

### Liên hệ với Match và Target

Đây là cách dễ hình dung:

```text
Packet
   ↓
Chain
   ↓
Match?
   │
   ├── NO  → Rule tiếp theo
   │
   └── YES
        ↓
   Counter +1 packet
   Counter + packet size
        ↓
      Target
        ↓
   ACCEPT / DROP / REJECT / ...
```

Ví dụ:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

Nếu có 10 packet SSH match rule này:

```text
PKTS  = 10
BYTES = tổng kích thước của 10 packet
TARGET = ACCEPT
```

**Tóm lại:**

> **Match** xác định packet có khớp rule không → nếu khớp, **counter được tăng** → sau đó **target** quyết định xử lý packet như thế nào.
