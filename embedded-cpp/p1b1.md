# Học Embedded C++ — Bài 1

## Vi điều khiển khác máy tính như thế nào?

Trước tiên, hãy bỏ qua thanh ghi, GPIO, linker và startup code. Ta chỉ cần hiểu **vi điều khiển là gì**.

---

## 1. Máy tính thông thường

Máy tính hoặc laptop thường có:

- CPU
- RAM
- Ổ cứng
- Màn hình
- Bàn phím
- Chuột
- Hệ điều hành như Windows hoặc Linux

Khi bạn chạy chương trình C++:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello";
}
```

Quá trình diễn ra gần giống như sau:

```text
Bạn mở chương trình
        ↓
Hệ điều hành nạp chương trình vào RAM
        ↓
Hệ điều hành cấp tài nguyên cho chương trình
        ↓
Hàm main() được chạy
        ↓
std::cout nhờ hệ điều hành hiển thị chữ ra màn hình
```

Chương trình không trực tiếp điều khiển màn hình. Nó yêu cầu hệ điều hành làm việc đó.

---

## 2. Vi điều khiển là gì?

Vi điều khiển, tiếng Anh là **microcontroller**, là một máy tính nhỏ được đặt trong các thiết bị điện tử.

Ví dụ:

- Máy giặt
- Nồi cơm điện
- Điều hòa
- Đồng hồ điện tử
- Bàn phím máy tính
- Robot
- Máy đo nhiệt độ
- Hệ thống điều khiển động cơ

Một vi điều khiển thường chứa ngay bên trong:

```text
CPU
RAM
Flash
GPIO
Timer
UART
ADC
Các ngoại vi khác
```

Nó giống như một chiếc máy tính rất nhỏ được đóng gói trong một con chip.

---

## 3. Ví dụ thực tế

Giả sử ta làm một chiếc quạt tự động.

Yêu cầu:

```text
Nhiệt độ thấp  → tắt quạt
Nhiệt độ cao   → bật quạt
```

Vi điều khiển sẽ thực hiện:

```text
Đọc cảm biến nhiệt độ
        ↓
So sánh nhiệt độ
        ↓
Điều khiển quạt
        ↓
Lặp lại
```

Mô tả bằng C++:

```cpp
int main()
{
    khoi_tao_cam_bien();
    khoi_tao_quat();

    while (true)
    {
        int nhiet_do = doc_nhiet_do();

        if (nhiet_do > 30)
        {
            bat_quat();
        }
        else
        {
            tat_quat();
        }
    }
}
```

Đây vẫn là C++ bình thường.

Điểm khác là chương trình không chủ yếu làm việc với bàn phím, màn hình hoặc file. Nó làm việc với:

```text
Cảm biến
Đèn LED
Nút nhấn
Động cơ
Loa
Màn hình nhỏ
Tín hiệu điện
```

---

## 4. Khác biệt quan trọng nhất: hệ điều hành

Máy tính thông thường thường có hệ điều hành:

```text
Chương trình C++
        ↓
Windows/Linux
        ↓
Phần cứng
```

Trong nhiều vi điều khiển đơn giản, không có hệ điều hành:

```text
Chương trình C++
        ↓
Phần cứng
```

Chương trình của bạn có thể trực tiếp điều khiển phần cứng.

Ví dụ, trên máy tính:

```cpp
std::cout << "Hello";
```

Thư viện C++ và hệ điều hành giúp đưa chữ lên màn hình.

Trong vi điều khiển, để bật một đèn LED, chương trình thường phải điều khiển một chân điện:

```cpp
bat_den_led();
```

Bên trong hàm này có thể có thao tác với thanh ghi phần cứng. Ta chưa cần học thanh ghi ngay.

---

## 5. Máy tính chạy nhiều chương trình

Một máy tính có thể chạy đồng thời:

```text
Trình duyệt
Trình phát nhạc
Visual Studio Code
Discord
Trò chơi
```

Hệ điều hành chia thời gian CPU cho nhiều chương trình.

Một vi điều khiển nhỏ thường chỉ chạy một firmware chính:

```text
Firmware điều khiển máy giặt
```

Firmware là chương trình được nạp vào vi điều khiển.

Khi bật nguồn:

```text
Vi điều khiển khởi động
        ↓
Firmware bắt đầu chạy
        ↓
Firmware điều khiển thiết bị
```

---

## 6. Chương trình máy tính thường có thể kết thúc

Ví dụ:

```cpp
int main()
{
    std::cout << "Hello\n";
    return 0;
}
```

Chương trình in chữ rồi kết thúc. Sau đó quyền điều khiển quay về hệ điều hành.

Nhưng firmware thường phải chạy cho đến khi mất điện.

```cpp
int main()
{
    khoi_tao();

    while (true)
    {
        xu_ly_cong_viec();
    }
}
```

Vòng lặp:

```cpp
while (true)
```

có nghĩa là chương trình tiếp tục chạy mãi.

Ví dụ máy giặt phải liên tục:

```text
Đọc nút nhấn
Kiểm tra cảm biến
Điều khiển động cơ
Cập nhật màn hình
Kiểm tra lỗi
Lặp lại
```

Nếu chương trình kết thúc, thiết bị không còn được điều khiển.

---

## 7. Tài nguyên của vi điều khiển ít hơn

Máy tính có thể có:

```text
RAM: 8 GB, 16 GB, 32 GB
Ổ cứng: hàng trăm GB
CPU: nhiều lõi, tốc độ cao
```

Vi điều khiển có thể chỉ có:

```text
RAM: vài KB đến vài MB
Flash: vài chục KB đến vài MB
CPU: chậm hơn nhiều
```

Con số cụ thể tùy từng loại vi điều khiển.

Vì tài nguyên hạn chế, lập trình nhúng cần chú ý:

- Không lãng phí RAM.
- Không tạo quá nhiều đối tượng lớn.
- Không cấp phát bộ nhớ tùy tiện.
- Không thực hiện tính toán không cần thiết.
- Phải biết chương trình sử dụng bao nhiêu bộ nhớ.

Ví dụ, trên máy tính đoạn này thường không đáng lo:

```cpp
int data[1'000'000];
```

Mảng này chứa một triệu số nguyên. Nếu mỗi `int` chiếm 4 byte, nó cần khoảng 4 MB bộ nhớ.

Một vi điều khiển chỉ có 64 KB RAM sẽ không thể chứa mảng đó.

---

## 8. Vi điều khiển làm việc trực tiếp với điện

Máy tính thông thường làm việc với dữ liệu như:

```text
Văn bản
Hình ảnh
Video
File
Mạng Internet
```

Vi điều khiển thường làm việc với tín hiệu vật lý:

```text
Nút được nhấn hay chưa
Nhiệt độ bao nhiêu
Điện áp bao nhiêu
Động cơ quay nhanh hay chậm
Đèn đang bật hay tắt
```

Ví dụ:

```text
Nút nhấn
   ↓ tín hiệu điện
Chân vi điều khiển
   ↓
Chương trình C++
   ↓
Chân điều khiển
   ↓ tín hiệu điện
Đèn LED
```

Chương trình C++ là phần đứng giữa tín hiệu đầu vào và hành động đầu ra.

---

## 9. Vi điều khiển thường làm một nhiệm vụ cụ thể

Laptop là thiết bị đa dụng. Bạn có thể dùng nó để:

- Học tập
- Lập trình
- Xem phim
- Chơi game
- Làm việc

Vi điều khiển thường được thiết kế cho một nhóm nhiệm vụ cụ thể.

Ví dụ vi điều khiển trong nồi cơm điện:

```text
Đọc nhiệt độ
Kiểm tra nút bấm
Điều khiển bộ gia nhiệt
Hiển thị trạng thái
Phát âm báo
```

Nó không cần chạy trình duyệt hoặc chỉnh sửa video.

---

## 10. So sánh tổng quát

| Máy tính thông thường | Vi điều khiển |
|---|---|
| Thường có hệ điều hành | Thường không có hệ điều hành |
| RAM rất lớn | RAM hạn chế |
| Chạy nhiều chương trình | Thường chạy một firmware |
| Có ổ cứng | Thường lưu chương trình trong Flash |
| Làm việc qua hệ điều hành | Có thể điều khiển phần cứng trực tiếp |
| Chương trình có thể kết thúc | Firmware thường chạy liên tục |
| Thiết bị đa dụng | Thường làm nhiệm vụ cụ thể |

---

## 11. Embedded C++ là gì?

Embedded C++ không phải là một ngôn ngữ mới.

Nó vẫn có:

```cpp
int
bool
if
else
for
while
class
struct
template
```

Điểm khác nằm ở môi trường chạy.

C++ trên máy tính:

```cpp
std::cout << "Hello";
```

C++ trên hệ thống nhúng:

```cpp
if (nut_duoc_nhan())
{
    bat_den();
}
```

Bạn đã có nền tảng C++, nên ta sẽ tập trung vào những điều mới:

```text
Phần cứng hoạt động thế nào
Chương trình được nạp thế nào
Bộ nhớ được tổ chức thế nào
Cách điều khiển chân điện
Cách xử lý thời gian
Cách giao tiếp với cảm biến
Cách viết firmware ổn định
```

---

# Điều cần nhớ sau bài này

Vi điều khiển là một máy tính nhỏ nằm trong thiết bị điện tử.

Firmware là chương trình chạy trên vi điều khiển.

Trong nhiều hệ thống nhúng:

```text
Không có hệ điều hành
Tài nguyên hạn chế
Chương trình chạy liên tục
Chương trình điều khiển phần cứng
```

Cấu trúc firmware đơn giản thường là:

```cpp
int main()
{
    // Chạy một lần khi thiết bị khởi động
    khoi_tao();

    // Chạy liên tục
    while (true)
    {
        doc_dau_vao();
        xu_ly();
        dieu_khien_dau_ra();
    }
}
```

---

## Tự kiểm tra

1. Firmware là gì?
2. Tại sao chương trình nhúng thường có `while (true)`?
3. Vi điều khiển thường có hệ điều hành hay không?
4. Điểm khác biệt lớn giữa `std::cout` và `bat_den()` là gì?
5. Tại sao phải chú ý đến dung lượng RAM trong lập trình nhúng?

---
