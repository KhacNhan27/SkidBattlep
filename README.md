# VNCRAFT.ASIA
## Infrastructure Setup Guide
### TCPShield · VPS Singapore Relay · Dedicated Server HCM

---

# 📋 Mục đích

Hướng dẫn thiết lập hệ thống relay Singapore nhằm:

- Giảm ping Minecraft từ ~120–140ms xuống còn ~40–80ms
- Ẩn IP thật máy chủ vật lý HCM
- Kết hợp TCPShield chống DDoS
- Có hệ thống failover AS1 → AS2

---

# 1. TỔNG QUAN HỆ THỐNG

## Luồng hiện tại (ping cao)

```text
Player
  │
  ▼
TCPShield
  │
  ▼
Máy chủ HCM
(ping 120–140ms)
```

## Luồng sau khi setup relay

```text
Player
  │
  ▼
TCPShield
  │
  ├─▶ as1.vncraft.asia (Primary)
  │
  └─▶ as2.vncraft.asia (Backup)
            │
            ▼
      Máy chủ HCM
      ping ~40–80ms
```

---

# 2. YÊU CẦU

| Thành phần | Ghi chú |
|---|---|
| TCPShield | Gói $25+ |
| VPS Singapore #1 | Ubuntu 22.04 |
| VPS Singapore #2 | Ubuntu 22.04 |
| Máy chủ HCM | Velocity proxy |
| Domain | vncraft.asia |

---

# 3. DNS

## A Record

| Type | Name | Value |
|---|---|---|
| A | as1 | IP_AS1 |
| A | as2 | IP_AS2 |

## SRV Record

| Type | Name | Priority | Weight | Port | Target |
|---|---|---|---|---|---|
| SRV | _minecraft._tcp.vncraft.asia | 0 | 5 | 25565 | as1.vncraft.asia |
| SRV | _minecraft._tcp.vncraft.asia | 10 | 5 | 25565 | as2.vncraft.asia |

---

# 4. SETUP VPS SINGAPORE

## SSH vào VPS

```bash
ssh root@IP_AS1
```

---

## Update hệ thống

```bash
apt update && apt upgrade -y
```

---

## Bật IP forwarding

```bash
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding=1' >> /etc/sysctl.conf

sysctl -p
```

---

## iptables DNAT

> Thay `IP_HCM_REAL` bằng IP thật máy chủ HCM

```bash
iptables -t nat -F
iptables -F FORWARD

iptables -t nat -A PREROUTING -p tcp --dport 25565 -j DNAT \
  --to-destination IP_HCM_REAL:25565

iptables -t nat -A POSTROUTING -d IP_HCM_REAL -j MASQUERADE

iptables -A FORWARD -p tcp -d IP_HCM_REAL --dport 25565 -j ACCEPT
iptables -A FORWARD -p tcp -s IP_HCM_REAL --sport 25565 -j ACCEPT
```

---

## Lưu iptables

```bash
apt install iptables-persistent -y

netfilter-persistent save
```

---

## Firewall

```bash
ufw allow 22/tcp
ufw allow 25565/tcp

ufw enable
```

---

# 5. MÁY CHỦ HCM

## Whitelist relay

```bash
iptables -I INPUT -p tcp --dport 25565 -s IP_AS1 -j ACCEPT
iptables -I INPUT -p tcp --dport 25565 -s IP_AS2 -j ACCEPT

iptables -A INPUT -p tcp --dport 25565 -j DROP
```

---

## Velocity config

```toml
bind = "0.0.0.0:25565"

[advanced]
proxy-protocol = true
```

---

# 6. TCPSHIELD

## Backend

| Field | Value |
|---|---|
| Backend IP | IP_HCM_REAL |
| Port | 25565 |
| Proxy Protocol | ON |

---

## TCPShield IP Ranges

```bash
curl https://tcpshield.com/v4-ranges
```

---

# 7. TEST

## DNS

```bash
nslookup as1.vncraft.asia

nslookup -type=SRV _minecraft._tcp.vncraft.asia
```

---

## Port test

```bash
telnet IP_AS1 25565
```

---

## Minecraft

- Add server:
  `vncraft.asia`

- Ping kỳ vọng:
  `40–80ms`

---

# 8. FAILOVER

| Relay | Priority |
|---|---|
| as1 | 0 |
| as2 | 10 |

Minecraft sẽ tự dùng AS2 nếu AS1 chết.

---

# 9. SCRIPT AUTO SETUP

```bash
#!/bin/bash

IP_HCM_REAL="xxx.xxx.xxx.xxx"

apt update && apt upgrade -y

echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p

iptables -t nat -A PREROUTING -p tcp --dport 25565 -j DNAT \
  --to-destination $IP_HCM_REAL:25565

iptables -t nat -A POSTROUTING -d $IP_HCM_REAL -j MASQUERADE

iptables -A FORWARD -p tcp -d $IP_HCM_REAL --dport 25565 -j ACCEPT
iptables -A FORWARD -p tcp -s $IP_HCM_REAL --sport 25565 -j ACCEPT

apt install iptables-persistent -y
netfilter-persistent save

ufw allow 22/tcp
ufw allow 25565/tcp
ufw --force enable

echo "DONE!"
```

---

# 10. KẾT QUẢ KỲ VỌNG

| Chỉ số | Trước | Sau |
|---|---|---|
| Ping VN | 120–140ms | 40–80ms |
| Lộ IP HCM | Có thể | Không |
| DDoS Protection | Có | Có |
| Failover | Không | Có |

---

# ✅ HOÀN TẤT

Sau khi setup:

- Người chơi VN ping thấp hơn đáng kể
- IP thật HCM được ẩn hoàn toàn
- Có backup relay AS2
- TCPShield tiếp tục chống DDoS
