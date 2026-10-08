# Đọc và ghi file trong C++

Trong C++ có hai cách phổ biến để thao tác với file:

- **Kiểu C:** `FILE*` và các hàm `fopen()`, `fclose()`, `fprintf()`, `fgets()`, `fread()`, `fwrite()` từ `<cstdio>`.
- **Kiểu C++:** `std::ifstream`, `std::ofstream`, `std::fstream` từ `<fstream>`.

Cả hai cách đều xử lý được file văn bản và file nhị phân.

## 1. `FILE*` và `fopen()` (kiểu C)

```cpp
#include <cstdio>

FILE* f = fopen("data.txt", "r");
if (f == nullptr) {
    // Không mở được file
}
```

- `FILE*`: con trỏ quản lý luồng file.
- `fopen(path, mode)`: mở file theo đường dẫn và chế độ; trả về `nullptr` khi thất bại.
- `fclose(f)`: đóng file; cần gọi khi mở thành công.

### 1.1. Các chế độ mở file

| Chế độ | Chức năng | Ghi chú |
|---|---|---|
| `"r"` | Chỉ đọc | File phải tồn tại |
| `"w"` | Chỉ ghi | Tạo mới hoặc xóa nội dung cũ |
| `"a"` | Ghi nối tiếp cuối file | Tạo file nếu chưa tồn tại |
| `"r+"` | Đọc và ghi | File phải tồn tại |
| `"w+"` | Đọc và ghi | Tạo mới hoặc xóa nội dung cũ |
| `"a+"` | Đọc và ghi, mọi lần ghi đều ở cuối file | Tạo file nếu chưa tồn tại |

Thêm `b` để làm việc ở chế độ nhị phân, chẳng hạn `"rb"`, `"wb"`.

### 1.2. Ghi file bằng `fprintf()`

```cpp
#include <cstdio>

int main() {
    FILE* f = fopen("data.txt", "w");
    if (f == nullptr) return -1;

    fprintf(f, "Hello World\n");
    fprintf(f, "Number: %d\n", 123);

    fclose(f);
    return 0;
}
```

Nội dung file:

```text
Hello World
Number: 123
```

`fprintf()` ghi dữ liệu có định dạng vào file, tương tự `printf()` ghi ra màn hình.

### 1.3. Đọc file theo dòng bằng `fgets()`

```cpp
#include <cstdio>

int main() {
    FILE* f = fopen("data.txt", "r");
    if (f == nullptr) return -1;

    char buffer[256];
    while (fgets(buffer, sizeof(buffer), f) != nullptr) {
        printf("%s", buffer);
    }

    if (ferror(f)) {
        // Đã xảy ra lỗi đọc
    }
    fclose(f);
    return 0;
}
```

`fgets()` đọc tối đa `sizeof(buffer) - 1` ký tự và thêm ký tự kết thúc chuỗi `\0`. Nếu một dòng dài hơn bộ đệm thì cần nhiều lần gọi mới đọc hết dòng.

### 1.4. Đọc, ghi nhị phân bằng `fread()` và `fwrite()`

```cpp
#include <cstdio>

int main() {
    int value = 123;
    FILE* f = fopen("data.bin", "wb");
    if (f == nullptr) return -1;

    if (fwrite(&value, sizeof(value), 1, f) != 1) {
        fclose(f);
        return -1;
    }
    fclose(f);

    int result = 0;
    f = fopen("data.bin", "rb");
    if (f == nullptr) return -1;

    if (fread(&result, sizeof(result), 1, f) != 1) {
        fclose(f);
        return -1;
    }
    fclose(f);

    printf("%d\n", result);
    return 0;
}
```

Cú pháp:

```cpp
fwrite(ptr, size, count, file);
fread(ptr, size, count, file);
```

- `ptr`: địa chỉ vùng nhớ.
- `size`: số byte mỗi phần tử.
- `count`: số phần tử cần ghi/đọc.
- `file`: luồng file.
- Giá trị trả về: số phần tử đã xử lý thành công.

> Lưu ý: ghi trực tiếp biểu diễn bộ nhớ của `int` không phải định dạng trao đổi dữ liệu độc lập nền tảng; kích thước và thứ tự byte có thể khác giữa các hệ thống.

## 2. `ifstream`, `ofstream`, `fstream` (kiểu C++)

```cpp
#include <fstream>
#include <string>
```

| Lớp | Chức năng |
|---|---|
| `std::ifstream` | Đọc file |
| `std::ofstream` | Ghi file |
| `std::fstream` | Đọc và ghi file |

### 2.1. Ghi file bằng `ofstream`

```cpp
#include <fstream>

int main() {
    std::ofstream file("data.txt");
    if (!file.is_open()) return -1;

    file << "Hello World\n";
    file << "Number: " << 123 << '\n';

    if (!file) return -1; // Kiểm tra lỗi ghi
    return 0;            // File tự đóng khi đối tượng hết scope
}
```

Theo mặc định, `ofstream` tạo file nếu chưa tồn tại và xóa nội dung cũ nếu file đã tồn tại.

### 2.2. Đọc file bằng `ifstream` và `getline()`

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream file("data.txt");
    if (!file.is_open()) return -1;

    std::string line;
    while (std::getline(file, line)) {
        std::cout << line << '\n';
    }

    if (file.bad()) return -1; // Lỗi I/O nghiêm trọng
    return 0;
}
```

`std::getline()` đọc một dòng vào `std::string`, không bao gồm ký tự xuống dòng.

### 2.3. Dùng `fstream` để đọc và ghi

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::fstream file("data.txt", std::ios::in | std::ios::out);
    if (!file.is_open()) return -1; // File phải tồn tại

    std::string line;
    while (std::getline(file, line)) {
        std::cout << line << '\n';
    }
    file.close();
    return 0;
}
```

Khi chuyển giữa thao tác đọc và ghi trên cùng một `fstream`, cần xử lý trạng thái và vị trí file phù hợp (ví dụ `clear()`, `seekg()`, `seekp()`).

### 2.4. Chế độ mở file trong C++

| Chế độ | Ý nghĩa |
|---|---|
| `std::ios::in` | Đọc |
| `std::ios::out` | Ghi |
| `std::ios::app` | Luôn ghi ở cuối file |
| `std::ios::trunc` | Xóa nội dung cũ khi mở |
| `std::ios::binary` | Chế độ nhị phân |
| `std::ios::ate` | Đưa vị trí file đến cuối ngay sau khi mở |

Có thể kết hợp các chế độ bằng `|`:

```cpp
std::ofstream file("data.txt", std::ios::app); // Ghi nối tiếp
```

### 2.5. Đọc/ghi nhị phân bằng stream

```cpp
#include <fstream>

int main() {
    int value = 123;
    {
        std::ofstream out("data.bin", std::ios::binary);
        if (!out) return -1;
        out.write(reinterpret_cast<const char*>(&value), sizeof(value));
        if (!out) return -1;
    }

    int result = 0;
    std::ifstream in("data.bin", std::ios::binary);
    if (!in) return -1;
    in.read(reinterpret_cast<char*>(&result), sizeof(result));
    if (!in) return -1;
    return 0;
}
```

## 3. So sánh hai cách

| Tiêu chí | Kiểu C (`FILE*`) | Kiểu C++ (`fstream`) |
|---|---|---|
| Thư viện | `<cstdio>` | `<fstream>` |
| Mở file | `fopen()` | Constructor hoặc `open()` |
| Đọc theo dòng | `fgets()` | `std::getline()` |
| Ghi văn bản | `fprintf()` | `operator<<` |
| Đọc nhị phân | `fread()` | `read()` |
| Ghi nhị phân | `fwrite()` | `write()` |
| Đóng file | `fclose()` | `close()` hoặc tự động khi hết scope |
| Kiểm tra lỗi | Giá trị trả về, `ferror()` | Trạng thái stream (`!file`, `bad()`...) |

## 4. Liên hệ với Linux `/proc`

Trong chương trình Linux, có thể gặp:

```cpp
FILE* file = fopen("/proc/1234/cmdline", "r");
```

- `/proc` là hệ thống file ảo do kernel cung cấp.
- `/proc/1234/` chứa thông tin liên quan tới tiến trình có PID `1234`.
- `/proc/1234/cmdline` chứa các đối số dòng lệnh, thường phân cách nhau bằng byte `\0` thay vì ký tự xuống dòng.
- Vì vậy, khi đọc `cmdline`, không nên mặc định xử lý như một file văn bản có các dòng thông thường.

## 5. Kết luận

- Với chương trình **C++ mới**, nên ưu tiên `std::ifstream`, `std::ofstream`, `std::fstream` vì cú pháp tự nhiên và tài nguyên được quản lý tự động.
- Khi **đọc code C/C++ hệ thống hoặc nhúng**, cần nắm `FILE*`, `fopen()`, `fread()`, `fwrite()` vì chúng vẫn rất phổ biến.
- Với cả hai cách, luôn **kiểm tra lỗi mở file và đọc/ghi**, đặc biệt khi làm việc với file quan trọng.
