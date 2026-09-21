# IPtables 

## 1. Tổng quan

`iptables` là công cụ user-space dùng để cấu hình firewall cho **Netfilter** trong kernel Linux. Với KVM/Libvirt, iptables thường được dùng để:

- Lọc traffic vào/ra host.
- Cho phép hoặc chặn traffic giữa VM và host.
- Forward traffic giữa VM và mạng ngoài.
- NAT cho VM ra Internet.
- DNAT để port forwarding từ host vào VM.
- Theo dõi packet bằng counters.

Ba thành phần chính:

| Thành phần | Ý nghĩa |
|---|---|
| `table` | Nhóm chức năng xử lý packet |
| `chain` | Điểm packet đi qua |
| `rule` | Điều kiện + hành động xử lý packet |

---

## 2. Các table chính

| Table | Chức năng | Chain thường dùng |
|---|---|---|
| `filter` | Lọc/cho phép/chặn packet | `INPUT`, `OUTPUT`, `FORWARD` |
| `nat` | NAT địa chỉ/port | `PREROUTING`, `POSTROUTING`, `OUTPUT` |
| `mangle` | Chỉnh sửa header packet | `PREROUTING`, `INPUT`, `FORWARD`, `OUTPUT`, `POSTROUTING` |
| `raw` | Xử lý trước conntrack | `PREROUTING`, `OUTPUT` |
| `security` | Chính sách MAC | `INPUT`, `OUTPUT`, `FORWARD` |

Trong thực tế KVM, hai table quan trọng nhất là `filter` và `nat`.

---

## 3. Các chain và luồng packet

### INPUT

Packet có đích là chính host.

```text
Network → PREROUTING → INPUT → Host
```

Dùng để kiểm soát SSH, HTTP, ICMP... vào host.

### OUTPUT

Packet được tạo bởi chính host.

```text
Host → OUTPUT → POSTROUTING → Network
```

### FORWARD

Packet đi qua host, ví dụ VM → Internet hoặc Internet → VM.

```text
VM → PREROUTING → FORWARD → POSTROUTING → Internet
```

Đây là chain quan trọng nhất khi host KVM đóng vai trò router/firewall.

### PREROUTING

Xử lý packet trước khi routing. Thường dùng trong `nat` để DNAT.

### POSTROUTING

Xử lý packet sau routing, trước khi packet rời host. Thường dùng để MASQUERADE/SNAT.

---

## 4. Rule cơ bản

Cú pháp tổng quát:

```bash
iptables [table] [command] [chain] [match] [target]
```

Ví dụ:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

Ý nghĩa:

- `-A INPUT`: thêm rule vào chain `INPUT`.
- `-p tcp`: áp dụng cho TCP.
- `--dport 22`: port đích 22.
- `-j ACCEPT`: cho phép packet.

### Các option thường dùng

| Option | Ý nghĩa |
|---|---|
| `-A` | Thêm rule vào cuối chain |
| `-I` | Chèn rule, thường dùng `-I CHAIN 1` để đưa lên đầu |
| `-D` | Xóa rule |
| `-R` | Thay thế rule |
| `-L` | Liệt kê rule |
| `-F` | Xóa rule trong chain/table |
| `-X` | Xóa chain tự tạo |
| `-Z` | Reset counters |
| `-P` | Đặt policy mặc định |
| `-N` | Tạo chain mới |
| `-n` | Không resolve DNS/service |
| `-v` | Hiển thị chi tiết và counters |
| `--line-numbers` | Hiển thị số thứ tự rule |

---

## 5. Xem và quản lý rule

### Xem rule

```bash
iptables -L -n -v
```

Xem riêng `FORWARD`:

```bash
iptables -L FORWARD -n -v
```

Xem rule có số thứ tự:

```bash
iptables -L --line-numbers
```

Xem NAT:

```bash
iptables -t nat -L -n -v
```

Xem riêng DNAT:

```bash
iptables -t nat -L PREROUTING -n -v
```

### Xóa rule

Xóa theo số thứ tự:

```bash
iptables -L FORWARD -n --line-numbers
iptables -D FORWARD 1
```

Xóa toàn bộ rule:

```bash
iptables -F
iptables -t nat -F
```

Reset counters:

```bash
iptables -Z
```

> **KVM/Libvirt:** Không nên tùy tiện dùng `iptables -F` và `iptables -X` trên host đang chạy Libvirt. Libvirt tạo các chain như `LIBVIRT_FWI`, `LIBVIRT_FWO`, `LIBVIRT_INP`, `LIBVIRT_OUT`. Xóa chúng có thể làm mạng VM lỗi.

Nếu chain Libvirt bị mất, có thể khởi động lại Libvirt và network:

```bash
systemctl restart libvirtd
virsh net-destroy default
virsh net-start default
```

---

## 6. Target thường dùng

### ACCEPT

Cho phép packet:

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

### DROP

Chặn packet và không gửi phản hồi:

```bash
iptables -A INPUT -s 192.168.1.100 -j DROP
```

### REJECT

Chặn packet và gửi phản hồi:

```bash
iptables -A INPUT -p tcp --dport 23 -j REJECT
```

### LOG

Ghi log packet:

```bash
iptables -A INPUT -j LOG --log-prefix "IPTables-DROP: "
```

---

## 7. Match thường dùng

### Theo IP nguồn

```bash
iptables -A INPUT -s 192.168.1.100 -j DROP
```

### Theo IP đích

```bash
iptables -A INPUT -d 192.168.1.10 -j ACCEPT
```

### Theo interface

```bash
iptables -A INPUT -i virbr0 -j ACCEPT
```

```bash
iptables -A FORWARD -o eth0 -j ACCEPT
```

- `-i`: interface packet đi vào.
- `-o`: interface packet đi ra.

### Theo protocol và port

```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

UDP:

```bash
iptables -A INPUT -p udp --dport 53 -j ACCEPT
```

ICMP:

```bash
iptables -A INPUT -p icmp -j ACCEPT
```

---

## 8. Stateful Firewall

iptables có thể sử dụng `conntrack` để theo dõi trạng thái kết nối.

Các state chính:

| State | Ý nghĩa |
|---|---|
| `NEW` | Kết nối mới |
| `ESTABLISHED` | Kết nối đã được thiết lập |
| `RELATED` | Kết nối liên quan tới kết nối khác |
| `INVALID` | Packet không xác định được trạng thái |
| `UNTRACKED` | Packet không được conntrack theo dõi |

Rule quan trọng:

```bash
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Với traffic đi qua KVM host:

```bash
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

Ví dụ firewall cơ bản:

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
```

`INPUT` và `FORWARD` chỉ cho phép những traffic đã được khai báo.

---

## 9. IP forwarding trên KVM

Khi VM cần đi qua KVM host để tới mạng khác, phải bật IP forwarding:

```bash
sysctl -w net.ipv4.ip_forward=1
```

Kiểm tra:

```bash
sysctl net.ipv4.ip_forward
```

Bật vĩnh viễn:

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```

---

## 10. KVM VM ra Internet bằng NAT

Giả sử:

```text
VM network: 192.168.122.0/24
Internet interface: ens33
```

Cho phép VM forward ra ngoài:

```bash
iptables -A FORWARD \
  -s 192.168.122.0/24 \
  -o ens33 \
  -j ACCEPT
```

Cho phép traffic phản hồi:

```bash
iptables -A FORWARD \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT
```

MASQUERADE:

```bash
iptables -t nat -A POSTROUTING \
  -s 192.168.122.0/24 \
  -o ens33 \
  -j MASQUERADE
```

Luồng:

```text
VM 192.168.122.x
        |
      virbr0
        |
   FORWARD
        |
   POSTROUTING
   MASQUERADE
        |
      ens33
        |
    Internet
```

`MASQUERADE` đổi source IP của VM thành IP của interface `ens33`.

---

## 11. DNAT - Port forwarding vào VM

DNAT dùng khi client truy cập IP của KVM host nhưng dịch vụ thực tế chạy trên VM.

Ví dụ:

```text
KVM host: 192.168.133.140
Backend VM: 192.168.122.113
Port: 80
```

DNAT port 80:

```bash
iptables -t nat -A PREROUTING \
  -p tcp \
  -d 192.168.133.140 \
  --dport 80 \
  -j DNAT \
  --to-destination 192.168.122.113:80
```

Cho phép FORWARD:

```bash
iptables -I FORWARD 1 \
  -p tcp \
  -d 192.168.122.113 \
  --dport 80 \
  -j ACCEPT
```

Port 443:

```bash
iptables -t nat -A PREROUTING \
  -p tcp \
  -d 192.168.133.140 \
  --dport 443 \
  -j DNAT \
  --to-destination 192.168.122.241:443
```

```bash
iptables -I FORWARD 1 \
  -p tcp \
  -d 192.168.122.241 \
  --dport 443 \
  -j ACCEPT
```

Luồng packet:

```text
Client
  |
  | 192.168.133.140:80
  v
PREROUTING
  |
  | DNAT
  v
192.168.122.113:80
  |
FORWARD
  |
  v
VM Backend
```

Điểm cần nhớ:

- `PREROUTING` thực hiện DNAT.
- `FORWARD` quyết định packet có được đi vào VM hay không.
- `POSTROUTING` xử lý SNAT/MASQUERADE.

---

## 12. Thứ tự rule

iptables kiểm tra rule theo thứ tự từ trên xuống.

Ví dụ:

```bash
iptables -L FORWARD -n --line-numbers
```

Nếu rule `DROP` hoặc rule Libvirt nằm trước rule `ACCEPT`, packet có thể không tới được rule phía dưới.

Đưa rule lên đầu:

```bash
iptables -I FORWARD 1 -p tcp -d 192.168.122.113 --dport 80 -j ACCEPT
```

Kiểm tra lại:

```bash
iptables -L FORWARD -n -v
```

---

## 13. Kiểm tra DNAT và FORWARD bằng counters

Xem DNAT:

```bash
iptables -t nat -L PREROUTING -n -v
```

Xem FORWARD:

```bash
iptables -L FORWARD -n -v
```

Cần quan sát cột:

```text
pkts
bytes
```

Nếu client gửi request nhưng counter của DNAT không tăng:

```text
Client
  ↓
PREROUTING
  ↓
DNAT không match
```

Nếu DNAT tăng nhưng FORWARD không tăng:

```text
Client
  ↓
PREROUTING
  ↓
DNAT OK
  ↓
FORWARD không match / bị rule phía trên xử lý
```

Đây là cách đơn giản để xác định packet đang bị dừng ở đâu.

---

## 14. tcpdump để debug

Bắt traffic trên tất cả interface:

```bash
tcpdump -i any -n
```

Theo port:

```bash
tcpdump -i any -n port 80
```

Theo IP:

```bash
tcpdump -i any -n host 192.168.122.113
```

Ví dụ kiểm tra DNAT port 80:

```bash
tcpdump -i any -n port 80
```

Có thể quan sát packet đi vào interface ngoài, sau đó xuất hiện trên `virbr0`/`vnet*` với địa chỉ VM.

---

## 15. Giới hạn traffic

Giới hạn ICMP:

```bash
iptables -A INPUT \
  -p icmp \
  --icmp-type echo-request \
  -m limit \
  --limit 1/m \
  --limit-burst 5 \
  -j ACCEPT
```

Giới hạn request TCP:

```bash
iptables -A INPUT \
  -p tcp --syn \
  -m limit \
  --limit 1/s \
  --limit-burst 4 \
  -j ACCEPT
```

Log packet:

```bash
iptables -A INPUT \
  -j LOG \
  --log-prefix "IPTables-DROP: "
```

---

## 16. Kiểm tra rule trước khi thêm

Dùng `-C` để kiểm tra rule đã tồn tại chưa:

```bash
iptables -C FORWARD \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT
```

Có thể kết hợp:

```bash
iptables -C FORWARD \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT || \
iptables -A FORWARD \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT
```

Cách này giúp tránh thêm cùng một rule nhiều lần.

---

## 17. Lưu và khôi phục rule

Ubuntu/Debian:

```bash
iptables-save > /etc/iptables/rules.v4
```

Khôi phục:

```bash
iptables-restore < /etc/iptables/rules.v4
```

Có thể cài:

```bash
apt install iptables-persistent
```

Sau đó lưu:

```bash
netfilter-persistent save
```

---

## 18. User-defined chain

Tạo chain:

```bash
iptables -N WEB
```

Đưa traffic TCP vào chain:

```bash
iptables -A INPUT -p tcp -j WEB
```

Thêm rule vào chain:

```bash
iptables -A WEB --dport 80 -j ACCEPT
iptables -A WEB --dport 443 -j ACCEPT
```

Xóa chain:

```bash
iptables -X WEB
```

Chỉ xóa được chain không còn rule/jump tham chiếu.

---

## 19. Một số cấu hình KVM thường gặp

### VM → Internet

Cần:

```bash
sysctl -w net.ipv4.ip_forward=1

iptables -A FORWARD -s 192.168.122.0/24 -o ens33 -j ACCEPT

iptables -A FORWARD \
  -m conntrack --ctstate ESTABLISHED,RELATED \
  -j ACCEPT

iptables -t nat -A POSTROUTING \
  -s 192.168.122.0/24 \
  -o ens33 \
  -j MASQUERADE
```

### Internet → VM qua port forwarding

Cần:

```bash
sysctl -w net.ipv4.ip_forward=1

iptables -t nat -A PREROUTING \
  -p tcp -d <HOST_IP> --dport 80 \
  -j DNAT --to-destination <VM_IP>:80

iptables -I FORWARD 1 \
  -p tcp -d <VM_IP> --dport 80 \
  -j ACCEPT
```

### Chặn một IP

```bash
iptables -A INPUT -s 192.168.1.100 -j DROP
```

### Chặn một IP khi forward

```bash
iptables -A FORWARD -s 192.168.1.100 -j DROP
```

---

## 20. Checklist khi debug KVM + iptables

Khi VM không ra Internet:

```bash
sysctl net.ipv4.ip_forward
iptables -L FORWARD -n -v
iptables -t nat -L POSTROUTING -n -v
```

Kiểm tra:

1. VM có IP đúng không.
2. VM có default gateway đúng không.
3. `ip_forward` có bật không.
4. `FORWARD` có cho phép traffic không.
5. `POSTROUTING` có MASQUERADE không.
6. Interface Internet có đúng không (`ens33`, `eth0`, ...).
7. Counter của rule có tăng không.

Khi DNAT không hoạt động:

```bash
iptables -t nat -L PREROUTING -n -v
iptables -L FORWARD -n -v
tcpdump -i any -n port 80
```

Kiểm tra theo thứ tự:

```text
Client
  ↓
PREROUTING / DNAT
  ↓
FORWARD
  ↓
virbr0 / vnet*
  ↓
VM
```

Nếu dùng Libvirt, kiểm tra thêm:

```bash
iptables -L -n -v
```

và chú ý các chain:

```text
LIBVIRT_FWI
LIBVIRT_FWO
LIBVIRT_FWX
LIBVIRT_INP
LIBVIRT_OUT
```

---

## 21. Tóm tắt các lệnh

```bash
# Xem rules
iptables -L -n -v
iptables -t nat -L -n -v

# Xem số dòng
iptables -L --line-numbers

# Thêm / chèn / xóa
iptables -A INPUT ...
iptables -I FORWARD 1 ...
iptables -D FORWARD 1

# Policy
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Stateful
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# IP forwarding
sysctl -w net.ipv4.ip_forward=1

# NAT VM → Internet
iptables -t nat -A POSTROUTING \
  -s 192.168.122.0/24 -o ens33 -j MASQUERADE

# DNAT
iptables -t nat -A PREROUTING \
  -p tcp -d <HOST_IP> --dport 80 \
  -j DNAT --to-destination <VM_IP>:80

# Debug
iptables -t nat -L PREROUTING -n -v
iptables -L FORWARD -n -v
tcpdump -i any -n port 80

# Lưu / khôi phục
iptables-save > /etc/iptables/rules.v4
iptables-restore < /etc/iptables/rules.v4
```

<!--Uwf -> iptables -->