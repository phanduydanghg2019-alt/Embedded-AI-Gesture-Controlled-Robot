# CT408: Embedded AI & Gesture-Controlled Robot 🚗🤖

Dự án xây dựng **Hệ thống AI nhúng (Embedded AI / TinyML)** điều khiển xe robot mô hình bằng cử chỉ tay theo thời gian thực[cite: 1, 2]. Hệ thống sử dụng cảm biến **MPU6050** để thu thập dữ liệu chuyển động, truyền dữ liệu không dây qua chuẩn **ESP-NOW**, và chạy mô hình phân loại **Decision Tree** trực tiếp trên vi điều khiển **STM32** để xuất tín hiệu điều khiển động cơ qua mạch **L298N**[cite: 1, 8, 10].

---

## 📌 Giới Thiệu Đồ Án

* **Môn học**: CT408 – Chuyên đề Kỹ thuật Máy tính (Embedded AI Systems)
* **Giảng viên hướng dẫn**: ThS/TS. Trần Văn Xẻn[cite: 2]
* **Mục tiêu**: Xử lý và phân loại các cử chỉ tay (Tới, Lùi, Trái, Phải, Dừng) ngay tại thiết bị biên (Edge Device) mà không phụ thuộc vào đám mây hay kết nối Internet[cite: 1, 2, 8].

---

## 🏗 Kiến Trúc Hệ Thống (Hardware Architecture)

```text
[ MPU6050 Sensor ]
       │ (I2C)
       ▼
[ ESP32-C3 Transmitter (Tay đeo) ]
       │ (ESP-NOW Wireless)
       ▼
[ ESP32 Receiver (Gắn trên xe) ]
       │ (UART - Frame Header: 0xAA 0xBB)
       ▼
[ STM32 Nucleo-F401RE ] ───(Thực thi AI_Predict / Decision Tree)
       │ (PWM & GPIO)
       ▼
[ L298N Motor Driver ] ──► [ Động Cơ Xe Robot (2WD / 4WD) ]#
