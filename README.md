# 🚗 Đồ án Chuyên Đề: Xe Robot Điều Khiển Bằng Cử Chỉ Tay (Ứng dụng Embedded AI)

![Embedded AI Project](https://img.shields.io/badge/Project-Embedded_AI-blue)
![STM32](https://img.shields.io/badge/MCU-STM32-brightgreen)
![ESP32](https://img.shields.io/badge/Wireless-ESP32-orange)
![Machine Learning](https://img.shields.io/badge/AI-Decision_Tree-red)

Dự án này là một hệ thống xe robot thông minh được điều khiển thông qua cử chỉ tay của người dùng. Điểm nổi bật của dự án là việc ứng dụng **Trí tuệ nhân tạo nhúng (Embedded AI)** — thay vì sử dụng các thuật toán dựa trên ngưỡng (threshold-based) truyền thống, hệ thống sử dụng một mô hình Học máy (Machine Learning) được huấn luyện sẵn và nhúng trực tiếp lên vi điều khiển STM32 để phân loại hành vi theo thời gian thực.

---

## ✨ Tính năng nổi bật

* **Nhận diện cử chỉ mượt mà:** Phân loại 5 trạng thái cử chỉ tay: `Tiến`, `Lùi`, `Rẽ Trái`, `Rẽ Phải`, và `Dừng lại`.
* **Giao tiếp không dây tốc độ cao:** Sử dụng ESP32 và giao thức không dây để truyền dữ liệu cảm biến thô từ tay người dùng đến xe.
* **Embedded AI (Edge Computing):** Mô hình **Decision Tree** (Cây quyết định) được huấn luyện trên PC bằng Python, sau đó chuyển đổi thành mã C (if-else logic) chạy độc lập và trực tiếp trên STM32, không cần kết nối Internet/Cloud.
* **Xử lý tín hiệu số:** Tích hợp bộ lọc tín hiệu (Filter) và tính toán góc nghiêng (Pitch, Roll, Yaw) kết hợp từ cả Gia tốc kế (Accelerometer) và Con quay hồi chuyển (Gyroscope).

---

## 🛠️ Kiến trúc Hệ thống & Phần cứng

Hệ thống được chia làm 2 module chính:

### 1. Module Phát (Đeo trên tay người dùng)
* **Vi điều khiển:** ESP32
* **Cảm biến:** MPU6050 (6 trục 3D Accelerometer & Gyroscope)
* **Chức năng:** Đọc dữ liệu gia tốc và góc xoay từ MPU6050, đóng gói dữ liệu và gửi không dây tới mạch thu trên xe robot.

### 2. Module Thu & Chấp hành (Trạm trung tâm trên Xe)
* **Vi điều khiển chính:** STM32 (xử lý AI và xuất xung PWM)
* **Vi điều khiển phụ/Giao tiếp:** ESP32 (nhận dữ liệu không dây và truyền qua UART cho STM32)
* **Khối điều khiển động cơ:** Module L298N / TB6612 (hoặc tương đương)
* **Chức năng:** STM32 nhận chuỗi dữ liệu thô qua UART, tính toán góc `pitch` và `roll`. Sau đó đưa vào mô hình Cây quyết định (`AI_Predict`) để đưa ra quyết định điều khiển chiều quay và tốc độ PWM cho 4 bánh xe.

---

## 🧠 Quy trình phát triển mô hình AI (Workflow)

Dự án không code cứng các góc nghiêng mà sử dụng quy trình chuẩn của một dự án Embedded AI:

1. **Thu thập dữ liệu (Data Collection):** Đeo MPU6050 lên tay, thực hiện các cử chỉ lặp đi lặp lại và gửi dữ liệu thô (ax, ay, az, gx, gy, gz) về máy tính.
2. **Tạo tập dữ liệu (Dataset Creation):** Lưu dữ liệu thành tệp `dataset.csv` và gán nhãn (Labeling) cho từng trạng thái.
3. **Huấn luyện mô hình (Model Training):** Sử dụng thư viện `Scikit-Learn` trên Python để huấn luyện mô hình **Decision Tree**.
4. **Nhúng mô hình (Deployment):** Trích xuất cấu trúc cây quyết định thành các điều kiện `if-else` trong ngôn ngữ C và nhúng vào hàm `AI_Predict()` trên vi điều khiển STM32.

---

## 💻 Cấu trúc Code (Software)

* Nhúng/Firmware: Lập trình bằng C trên **STM32CubeIDE** (hoặc KeilC).
* Cấu hình ngoại vi:
  * **UART:** Giao tiếp giữa ESP32 và STM32 (Baudrate tối ưu để tránh nhiễu).
  * **Timer / PWM:** Điều khiển tốc độ (Speed Control) và bẻ lái động cơ DC.
  * **GPIO:** Điều khiển chiều quay của động cơ.

### Đoạn code minh họa (Cây quyết định nhúng)
```c
int AI_Predict(float current_pitch, float current_roll) {
    if (current_pitch <= 39.00f) {
        if (current_roll <= 23.50f) {
            if (current_pitch <= -2.00f) {
                if (current_roll <= -18.50f) return 2; // Chạy Lùi
                else return 0; // Dừng
            } else return 4; // Rẽ Phải
        } else return 1; // Chạy Tới
    } else return 3; // Rẽ Trái
}
```
