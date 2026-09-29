# Thực hành buổi 1: Lập trình Python với MQTT

**Sinh viên:** Phạm Anh Tuấn  
**Mã sinh viên:** B23DCCN891

**Sinh viên:** Phạm Hồng Sơn

**Mã sinh viên:** B23DCCN723

Repository này chứa các bài tập thực hành giao thức MQTT sử dụng thư viện paho-mqtt bằng Python.

## 1. Môi trường và cấu trúc thư mục

- Windows, Python 3.x.
- Thư viện Python: `paho-mqtt`.
- MQTT broker: Eclipse Mosquitto

```text
BTH1/
├── README.md
├── mosquitto-lab.conf
├── publisher_bai1.py
├── subscriber_bai1.py
├── sensor_publisher_bai2.py
├── monitor_subscriber_bai2.py
├── device_bai3.py
└── controller_bai3.py
```

## 2. Cấu hình và chạy MQTT broker

1. Tải bộ cài Mosquitto dành cho Windows tại <https://mosquitto.org/download/> và cài đặt. Nếu đã cài Mosquitto thì bỏ qua bước này.
2. Tạo file `mosquitto-lab.conf` tại thư mục gốc của repo với nội dung:

   ```conf
   listener 1884 127.0.0.1
   allow_anonymous true
   ```

   `listener 1884 127.0.0.1` cho broker lắng nghe tại `localhost:1884`, chỉ nhận kết nối từ cùng máy. `allow_anonymous true` cho phép các chương trình của bài thực hành kết nối không cần tài khoản. Sử dụng cổng `1884`.

3. Mở PowerShell tại thư mục gốc của repo và chạy:

   ```powershell
   & "C:\Program Files\mosquitto\mosquitto.exe" -c ".\mosquitto-lab.conf" -v
   ```

   Nếu cài Mosquitto ở vị trí khác, thay đường dẫn đến `mosquitto.exe`. `-c` nạp file cấu hình, `-v` hiển thị log. Khi thấy `Opening ipv4 listen socket on port 1884` và `mosquitto version ... running`, giữ cửa sổ này mở trong lúc thực hành.

Các chương trình Python trong repo kết nối tới **host `localhost`, port `1884`**. Khi kết thúc, dừng tiến trình broker trong cửa sổ đã chạy nó.

## 3. Chuẩn bị Python và làm bài

1. Mở thư mục repo bằng IDE bất kì
2. Mở Terminal tại thư mục repo, cài thư viện:

   ```powershell
   .\.venv\Scripts\python.exe -m pip install paho-mqtt
   ```

   Nếu project chưa có `.venv`, tạo bằng `py -m venv .venv` rồi chạy lại lệnh cài đặt trên.

Mỗi bài cần **hai chương trình chạy đồng thời**. Có thể mở hai tab Terminal và chạy lệnh tương ứng bên dưới. Luôn khởi động broker ở mục 2 trước. Mỗi dòng lệnh trong một cặp phải chạy ở **một Terminal riêng**, không chạy nối tiếp trong cùng Terminal đang bị chương trình đầu tiên chiếm dụng.

## 4. Bài 1: Gửi và nhận thông điệp

**Topic:** `iot/lab/message`.

- **Mô tả**: Ứng dụng Publisher cho phép người dùng nhập lời chào, sau đó gửi qua MQTT kèm theo thông tin mã sinh viên và họ tên (Phạm Anh Tuấn - B23DCCN891). Subscriber sẽ lắng nghe và in tin nhắn ra màn hình.
- **Cách chạy**:
  1. Mở Terminal trong VSCode và chạy: `python subscriber_bai1.py`
  2. Mở thêm một Terminal mới (chia đôi màn hình Terminal) và chạy: `python publisher_bai1.py`
- **Kết quả**:
  - Tại cửa sổ Publisher, khi bạn nhập tin nhắn, terminal của Subscriber sẽ hiển thị Topic, Payload (nội dung tin nhắn kèm thông tin sinh viên) và thời gian nhận.
  - Gõ `EXIT` ở Publisher để kết thúc chương trình.

---

## 5. Bài 2: Mô phỏng cảm biến nhiệt độ và độ ẩm

**Topic:** `iot/lab/sensor01/data`.

- **Mô tả**: Cảm biến (Sensor) tự động tạo dữ liệu nhiệt độ và độ ẩm ngẫu nhiên mỗi 3 giây rồi gửi lên Broker. Bộ giám sát (Monitor) sẽ đọc dữ liệu này và đưa ra cảnh báo nếu vượt ngưỡng an toàn.
- **Cách chạy**:
  1. Khởi động bộ giám sát trước: `python monitor_subscriber_bai2.py`
  2. Khởi động cảm biến: `python sensor_publisher_bai2.py`
- **Kết quả**:
  - Monitor sẽ in ra các thông số liên tục.
  - Nếu nhiệt độ > 35°C, màn hình hiện: `CANH BAO: Nhiet do cao`.
  - Nếu độ ẩm < 40%, màn hình hiện: `CANH BAO: Do am thap`.
  - Bấm `Ctrl + C` ở terminal để dừng chương trình.

---

## 6. Bài 3: Điều khiển đèn thông minh

| Topic                    | Chức năng                                         |
| ------------------------ | ------------------------------------------------- |
| `iot/lab/light01/cmd`    | Controller gửi lệnh `ON` hoặc `OFF` cho thiết bị. |
| `iot/lab/light01/status` | Thiết bị phản hồi trạng thái mới bằng JSON.       |

Chạy thiết bị trước ở Terminal thứ nhất:

```powershell
.\.venv\Scripts\python.exe device_bai3.py
```

## Bài 3: Điều khiển thiết bị từ xa qua MQTT

- **Mô tả**: Bộ điều khiển (Controller) gửi lệnh `ON` hoặc `OFF` tới thiết bị đèn (Device). Đèn sau khi nhận lệnh sẽ chuyển trạng thái và gửi thông báo phản hồi lại cho bộ điều khiển.
- **Cách chạy**:
  1. Chạy chương trình mô phỏng đèn: `python device_bai3.py`
  2. Chạy chương trình điều khiển: `python controller_bai3.py`
- **Kết quả**:
  - Nhập lệnh `ON` hoặc `OFF` trên Controller, màn hình Device sẽ thông báo "Da chuyen den sang ON/OFF".
  - Ngay sau đó, Controller sẽ nhận được tin nhắn phản hồi báo trạng thái mới nhất từ Device theo định dạng JSON.
  - Gõ `EXIT` ở Controller để thoát.

---

## 7. Kiểm tra nhanh khi không nhận được dữ liệu

1. Broker vẫn đang chạy và log cho thấy nó lắng nghe cổng `1884`.
2. Cả hai chương trình dùng `localhost` và cổng `1884`.
3. Chương trình nhận hoặc thiết bị đã chạy **trước** khi gửi tin hay lệnh.
4. Topic ở phía gửi và phía nhận phải giống hệt nhau, kể cả chữ hoa và chữ thường.
