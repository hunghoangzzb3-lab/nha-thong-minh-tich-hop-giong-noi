# 🏠 Nhà thông minh ESP32 + Xiaozhi MCP

Hệ thống nhà thông minh dùng **ESP32**, điều khiển bằng **giọng nói** qua trợ lý AI [Xiaozhi](https://xiaozhi.me) (giao thức MCP qua WebSocket), kết hợp **nút bấm vật lý**, **cảm biến nhiệt độ/độ ẩm**, **cảm biến lửa** và **đèn hiên tự động**.

## ✨ Tính năng

- 🎙️ **Điều khiển bằng giọng nói**: bật/tắt quạt, đèn phòng khách, đèn phòng ngủ, đèn phòng bếp, hoặc tất cả thiết bị cùng lúc thông qua Xiaozhi MCP.
- 🔘 **Nút bấm vật lý**: mỗi thiết bị có một nút riêng, đồng bộ trạng thái với lệnh giọng nói.
- 🌡️ **Đo nhiệt độ & độ ẩm** (DHT11): Xiaozhi có thể trả lời khi hỏi "nhiệt độ phòng bao nhiêu?".
- 🔥 **Phát hiện lửa**: còi báo động kêu liên tục ngay khi phát hiện.
- 🚨 **Cảnh báo nhiệt độ nhiều cấp**:
  - `> 40°C`: còi nháy 0,5 giây.
  - `> 45°C`: còi kêu liên tục và **tự động tắt tất cả thiết bị**.
- 🚪 **Đèn hiên thông minh**: tự bật khi cảm biến hồng ngoại (IR) phát hiện người, tự tắt sau 5 giây; có thể bật/tắt thủ công bằng nút bấm. Hoạt động độc lập, không cần mạng.

## 🧰 Phần cứng

| Linh kiện | Số lượng | Ghi chú |
|---|---|---|
| ESP32 DevKit | 1 | |
| Module relay / LED | 5 | Quạt + 3 đèn trong nhà + đèn hiên |
| Nút bấm | 5 | Quạt, 3 đèn, đèn hiên |
| Cảm biến DHT11 | 1 | Nhiệt độ & độ ẩm |
| Cảm biến lửa | 1 | Ngõ ra digital (DO) |
| Cảm biến hồng ngoại (IR) | 1 | Phát hiện chuyển động/người |
| Còi buzzer | 1 | |

## 📌 Sơ đồ chân (Pinout)

### Thiết bị đầu ra

| Thiết bị | GPIO |
|---|---|
| Quạt | 2 |
| Đèn phòng khách | 4 |
| Đèn phòng ngủ | 18 |
| Đèn phòng bếp | 19 |
| Đèn hiên | 21 |
| Còi buzzer | 33 |

### Nút bấm (`INPUT_PULLUP`, nối GND khi nhấn)

| Nút | GPIO |
|---|---|
| Quạt | 14 |
| Đèn phòng khách | 27 |
| Đèn phòng ngủ | 26 |
| Đèn phòng bếp | 25 |
| Đèn hiên | 16 |

### Cảm biến

| Cảm biến | GPIO | Ghi chú |
|---|---|---|
| DHT11 | 32 | |
| Lửa | 17 | Mặc định tích cực mức `LOW` |
| IR | 35 | Mặc định tích cực mức `LOW` |

> 💡 Nếu cảm biến lửa/IR hoạt động ngược logic, đổi `FIRE_DETECTED` / `IR_DETECTED` thành `HIGH` trong code.

## 📦 Thư viện cần cài

Cài qua Arduino IDE (Library Manager) hoặc PlatformIO:

- `WebSocketMCP` (client MCP cho ESP32)
- `DHT sensor library` (Adafruit) và `Adafruit Unified Sensor`
- `ArduinoJson` (v6)
- `WiFi` (có sẵn trong ESP32 core)

Board: **ESP32 Dev Module** (cài ESP32 board package trong Arduino IDE).

## 🚀 Hướng dẫn cài đặt

1. **Clone repo**
   ```bash
   git clone https://github.com/<username>/<ten-repo>.git
   cd <ten-repo>
   ```

2. **Lấy MCP endpoint** từ [xiaozhi.me](https://xiaozhi.me): vào trang quản lý agent → mục MCP → sao chép địa chỉ endpoint (dạng `wss://api.xiaozhi.me/mcp/?token=...`).

3. **Cấu hình thông tin** trong file `.ino`:
   ```cpp
   const char* ssid        = "TEN_WIFI";
   const char* password    = "MAT_KHAU_WIFI";
   const char* mcpEndpoint = "wss://api.xiaozhi.me/mcp/?token=YOUR_TOKEN";
   ```

4. **Nạp code** vào ESP32 và mở Serial Monitor ở tốc độ `115200` để xem log.

5. Khi thấy `[MCP] Đã kết nối tới máy chủ` và `[MCP] Đã đăng ký tất cả tools` là hệ thống sẵn sàng.

## 🗣️ Các MCP tool

| Tool | Chức năng | Tham số |
|---|---|---|
| `fan_control` | Bật/tắt quạt | `state`: `on` / `off` |
| `living_room_lights_control` | Bật/tắt đèn phòng khách | `state` |
| `bedroom_lights_control` | Bật/tắt đèn phòng ngủ | `state` |
| `kitchen_lights_control` | Bật/tắt đèn phòng bếp | `state` |
| `all_devices_control` | Bật/tắt tất cả thiết bị | `state` |
| `temperature_and_humidity_values` | Lấy nhiệt độ & độ ẩm hiện tại | không |

**Ví dụ câu lệnh với Xiaozhi:**

- "Bật đèn phòng khách"
- "Tắt quạt đi"
- "Tắt hết các thiết bị"
- "Nhiệt độ trong phòng bao nhiêu?"

## ⚙️ Logic hoạt động

### Ưu tiên còi báo động

```
Phát hiện lửa  >  Nhiệt độ > 45°C  >  Nhiệt độ > 40°C  >  Bình thường
 (kêu liên tục)   (kêu liên tục +      (nháy 0,5s)         (tắt còi)
                  tắt mọi thiết bị)
```

### Đèn hiên

```
IR phát hiện người ──► Bật đèn (chế độ tự động)
                          │
                          └─ Không còn phát hiện sau 5 giây ──► Tự tắt

Nhấn nút hiên ──► Bật/tắt thủ công (không tự tắt khi đang ở chế độ thủ công)
```

### Vòng lặp chính

```cpp
void loop() {
    handleFire();     // đọc cảm biến lửa
    handleButtons();  // nút bấm thiết bị chính
    handlePorch();    // đèn hiên + IR
    handleDHT();      // đọc nhiệt độ/độ ẩm mỗi 2 giây
    updateBuzzer();   // tổng hợp trạng thái còi
    mcpClient.loop(); // duy trì kết nối MCP
}
```

## 🛠️ Xử lý sự cố

| Vấn đề | Cách khắc phục |
|---|---|
| ESP32 tự khởi động lại | Không kết nối được WiFi sau 20 lần thử. Kiểm tra SSID/mật khẩu và đảm bảo dùng WiFi 2.4 GHz. |
| Không kết nối được MCP | Kiểm tra token còn hạn, ESP32 có Internet. |
| Còi kêu ngược (kêu khi không có lửa) | Đổi `FIRE_DETECTED` từ `LOW` sang `HIGH`. |
| Đèn hiên bật/tắt ngược | Đổi `IR_DETECTED` sang `HIGH`. |
| `[DHT] Lỗi đọc cảm biến!` | Kiểm tra dây nối DHT11, nguồn 3.3V/5V và điện trở kéo lên (nếu dùng module trần). |
| Quạt (GPIO2) nháy khi khởi động | GPIO2 là chân strapping và nối với LED onboard; có thể đổi sang chân khác nếu gặp lỗi khi nạp code. |

## 🔒 Lưu ý bảo mật

- **Không commit** token MCP và mật khẩu WiFi lên GitHub. Hãy dùng file `secrets.h` (thêm vào `.gitignore`):
  ```cpp
  // secrets.h
  #define WIFI_SSID     "TEN_WIFI"
  #define WIFI_PASSWORD "MAT_KHAU_WIFI"
  #define MCP_ENDPOINT  "wss://api.xiaozhi.me/mcp/?token=YOUR_TOKEN"
  ```
- Nếu lỡ đẩy token lên GitHub, hãy **tạo lại token mới** trên xiaozhi.me.

## 📁 Cấu trúc dự án

```
.
├── smart_home.ino    # Mã nguồn chính
├── secrets.h         # Thông tin nhạy cảm (không commit)
├── .gitignore
└── README.md
```

## 📄 Giấy phép

Phát hành theo giấy phép [MIT](LICENSE).

## 🙌 Đóng góp

Mọi đóng góp (issue, pull request) đều được chào đón!
