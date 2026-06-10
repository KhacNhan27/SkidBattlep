# SokiBattlepass

**Plugin Minecraft 1.21.1** | Paper/Folia |

Hệ thống mời bạn bè, quà comeback, và sự kiện **Chung Sức** cho server SokiMC EconomySMP.

---

## 📋 Tính Năng

### 1. Hệ Thống Mời Bạn Bè (Referral)

Người chơi tạo mã mời để giới thiệu bạn bè mới vào server.

| Lệnh | Alias | Mô tả |
|------|-------|-------|
| `/invite` | `/moi` | Tạo mã mời của bạn |
| `/invite list` | | Xem các mã đang hoạt động |
| `/invite top` | | Bảng xếp hạng tháng hiện tại |
| `/invite top last-month` | | BXH tháng trước |
| `/redeem <mã>` | `/nhanma <mã>` | Nhập mã giới thiệu |

#### Cách hoạt động
1. Người chơi dùng `/invite` → nhận mã mời riêng (VD: `ZIRV3K7Q`)
2. Bạn bè mới dùng `/redeem ZIRV3K7Q` **trong vòng 24 giờ đầu**
3. Người được mời chơi đủ **60 phút** → cả hai nhận thưởng

#### Thưởng người được mời
- 100 BattlePass Points
- 250 Shards
- 1 Common Key

#### Thưởng người mời
- 200 BattlePass Points
- 100 Shards
- Lệnh extra tùy chỉnh trong config
- Tag `referrer` tạm thời (24h, có thể tắt)

#### Top Referral Hàng Tháng
BXH reset đầu mỗi tháng. Phần thưởng top:
- **Top 1:** MVP 30 ngày + 5000 BP Points
- **Top 2:** VIP+ 14 ngày + 3000 BP Points
- **Top 3:** VIP 7 ngày + 1500 BP Points

---

### 2. Quà Comeback

Người chơi vắng **≥ 15 ngày** nhận quà khi quay lại.

| Lệnh | Mô tả |
|------|-------|
| `/comeback` | Nhận quà comeback |

- Thông báo tự động khi đăng nhập nếu đủ điều kiện
- Gói quà mặc định: 500 Shards + 3 Crimson + 2 Prime + 3 Gold + 2 Money Crates + 200 BP Points
- Tùy chỉnh trong `config.yml` > `comeback.rewards.extra-commands`

---

### 3. Sự Kiện Chung Sức ⚡

Mỗi người chơi có một **thanh điểm cá nhân** với 3 mốc thưởng (300 / 600 / 900 điểm).

| Lệnh | Mô tả |
|------|-------|
| `/chungsuc` | Mở menu Chung Sức |

#### Cách tích điểm

| Hành động | Điểm | Giới hạn |
|-----------|------|----------|
| Nhập mã của người khác | +15 điểm | **Tối đa 2 lượt/ngày** |
| Người khác nhập mã của bạn | +10 điểm | Không giới hạn |

**Ví dụ:**
- Player A nhập mã của Player B → A nhận +15 điểm, B nhận +10 điểm
- Nếu A đã dùng 2 lượt hôm nay → A không nhận điểm thêm, nhưng B **vẫn nhận** +10 điểm

#### Mốc thưởng
Mỗi mốc nhận một lần. Tích lũy điểm để mở khóa:
- **300 điểm** → Thưởng nhỏ (config tùy chỉnh)
- **600 điểm** → Thưởng vừa
- **900 điểm** → Thưởng lớn

---

## ⚙️ Cài Đặt

### Yêu cầu
- Paper hoặc Folia **1.21.1**
- Java 17+

### Cài đặt
1. Tải `SokiBattlepass.jar` vào thư mục `plugins/`
2. Khởi động server
3. Chỉnh sửa `plugins/SokiBattlepass/config.yml`
4. Dùng `/sokibp reload`

### Database
Mặc định dùng **SQLite** (`data.db`). Chuyển sang MySQL trong config:
```yaml
database:
  type: mysql
  mysql:
    host: localhost
    port: 3306
    database: sokibp
    username: root
    password: yourpassword
```

---

## 🌐 Ngôn Ngữ

Hỗ trợ **Tiếng Việt** và **Tiếng Anh**.

```yaml
# config.yml
language: vi   # hoặc en
```

Đổi ngôn ngữ runtime: `/sokibp setlang vi`

File ngôn ngữ: `messages_vi.yml` / `messages_en.yml` — tùy chỉnh tất cả tin nhắn với MiniMessage format.

---

## 🛠️ Lệnh Admin

| Lệnh | Mô tả |
|------|-------|
| `/sokibp reload` | Tải lại config và messages |
| `/sokibp setlang <vi\|en>` | Đổi ngôn ngữ |
| `/sokibp give <player> <points>` | Thêm điểm Chung Sức |
| `/sokibp reset <player>` | Reset toàn bộ dữ liệu người chơi |

**Permission:** `sokibp.admin` (mặc định: OP)

---

## 📜 Tất Cả Permissions

| Permission | Mô tả | Mặc định |
|-----------|-------|---------|
| `sokibp.admin` | Lệnh admin | OP |
| `sokibp.invite` | Dùng /invite | true |
| `sokibp.redeem` | Dùng /redeem | true |
| `sokibp.comeback` | Dùng /comeback | true |
| `sokibp.chungsuc` | Dùng /chungsuc | true |

---

## 🔧 Tùy Chỉnh Lệnh Thưởng

Plugin dùng **console commands** để trao thưởng, giúp tương thích với mọi economy plugin:

```yaml
# Lệnh cộng BP Points (%player% và %amount% được thay tự động)
battlepass:
  add-points-command: "bp addpoints %player% %amount%"

# Lệnh thưởng thêm cho người mời
referral:
  inviter-rewards:
    extra-commands:
      - "eco give %player% 1000"
      - "crates give %player% rare 1"
```

---

*Plugin được viết cho SokiMC EconomySMP — v1*
