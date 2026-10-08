# DIR và thao tác với thư mục trong C/C++ trên Linux

## 1. DIR là gì?

`DIR` là kiểu dữ liệu đại diện cho **luồng thư mục** (*directory stream*) trong API POSIX. Ta dùng `DIR*` để duyệt các mục (file, thư mục con, liên kết, v.v.) của một thư mục.

```cpp
#include <dirent.h>

DIR* dir;
```

`DIR*` không phải danh sách tên file và cũng không chứa nội dung file. Nó là đối tượng quản lý quá trình duyệt thư mục.

## 2. Các thành phần thường dùng

| Thành phần | Chức năng |
|---|---|
| `DIR*` | Con trỏ đến luồng thư mục |
| `opendir(path)` | Mở luồng thư mục |
| `readdir(dir)` | Lấy mục tiếp theo trong thư mục |
| `struct dirent` | Thông tin về một mục, đặc biệt là `d_name` |
| `closedir(dir)` | Đóng luồng thư mục |
| `rewinddir(dir)` | Đưa vị trí đọc trở về đầu |
| `telldir(dir)` | Lấy dấu vị trí đọc hiện tại |
| `seekdir(dir, pos)` | Khôi phục vị trí đọc đã lưu |
| `scandir(...)` | Lấy danh sách các mục, có thể lọc và sắp xếp |

Các chức năng này chủ yếu có trong `<dirent.h>`.

## 3. Mở thư mục bằng `opendir()`

```cpp
DIR* dir = opendir("./data");
if (dir == nullptr) {
    perror("opendir");
    return 1;
}
```

- Thành công: trả về con trỏ `DIR*` hợp lệ.
- Thất bại: trả về `nullptr` (trong C có thể viết `NULL`), ví dụ đường dẫn không tồn tại hoặc không đủ quyền.
- `opendir()` **không tạo thư mục mới**, không đọc nội dung file.

Đường dẫn có thể là tuyệt đối (`/home/user/data`) hoặc tương đối (`./data`, tính từ thư mục làm việc hiện tại).

## 4. Đọc từng mục bằng `readdir()`

```cpp
struct dirent* entry;
while ((entry = readdir(dir)) != nullptr) {
    printf("%s\n", entry->d_name);
}
```

Mỗi lần gọi `readdir()` lấy **một mục tiếp theo**. `entry->d_name` là tên của mục, có thể là tên file hoặc tên thư mục con. Khi hết mục, hàm trả về `nullptr`; lỗi cũng có thể trả về giá trị đó (dùng `errno` nếu cần phân biệt).

Ví dụ cấu trúc thư mục:

```text
data/
├── file1.txt
├── file2.json
└── folder1/
    └── file3.cpp
```

Khi mở `./data`, `readdir()` có thể trả về `file1.txt`, `file2.json`, `folder1` (và thường có `.` và `..`). **Nó không tự đi vào `folder1` để lấy `file3.cpp`.** Thứ tự trả về không được đảm bảo.

- `.`: thư mục hiện tại.
- `..`: thư mục cha.

## 5. `struct dirent` chứa gì?

Một số trường phổ biến trên Linux:

| Trường | Ý nghĩa |
|---|---|
| `d_name` | Tên mục |
| `d_type` | Loại mục (nếu hệ thống file cung cấp) |
| `d_ino` | Inode của mục |
| `d_reclen` | Độ dài bản ghi |
| `d_off` | Dấu vị trí trong luồng, không nên coi là offset byte thông thường |

Một số giá trị `d_type`:

- `DT_REG`: file thông thường.
- `DT_DIR`: thư mục.
- `DT_LNK`: liên kết tượng trưng (*symbolic link*).
- `DT_UNKNOWN`: chưa xác định loại.

**Chú ý:** Không phải hệ thống file nào cũng hỗ trợ `d_type`. Nếu cần kiểm tra loại chắc chắn, dùng `stat()` hoặc `lstat()` với đường dẫn đầy đủ của mục.

## 6. Đóng thư mục bằng `closedir()`

```cpp
closedir(dir);
```

Đóng luồng và giải phóng tài nguyên liên quan. Hàm trả về `0` nếu thành công, `-1` nếu lỗi. Không sử dụng lại con trỏ sau khi đóng.

## 7. Ví dụ đầy đủ: in tên các mục trong thư mục

```cpp
#include <cstdio>
#include <dirent.h>

int main() {
    DIR* dir = opendir("./data");
    if (dir == nullptr) {
        perror("opendir");
        return 1;
    }

    struct dirent* entry;
    while ((entry = readdir(dir)) != nullptr) {
        // Bo qua '.' va '..'
        if (entry->d_name[0] == '.' &&
            (entry->d_name[1] == '\0' ||
            (entry->d_name[1] == '.' && entry->d_name[2] == '\0'))) {
            continue;
        }
        printf("%s\n", entry->d_name);
    }

    closedir(dir);
    return 0;
}
```

Biên dịch trên Linux:

```bash
g++ -std=c++17 main.cpp -o listdir
./listdir
```

## 8. Ví dụ: chỉ in tên file thông thường

```cpp
while ((entry = readdir(dir)) != nullptr) {
    if (entry->d_type == DT_REG) {
        printf("File: %s\n", entry->d_name);
    }
}
```

Cách trên đơn giản nhưng có thể bỏ sót file khi `d_type == DT_UNKNOWN`. Muốn chính xác hơn, hãy lấy đường dẫn đầy đủ và kiểm tra bằng `stat()`/`lstat()`.

## 9. Ví dụ: tìm một tên file trong thư mục hiện tại

```cpp
#include <cstring>
// Dat trong vong lap sau khi opendir() thanh cong:
while ((entry = readdir(dir)) != nullptr) {
    if (std::strcmp(entry->d_name, "config.json") == 0) {
        printf("Tim thay ten config.json\n");
        break;
    }
}
```

Phép so sánh trên chỉ xét **tên mục**, không kiểm tra nội dung hoặc bảo đảm mục đó là file thông thường.

## 10. Đọc lại từ đầu và lưu vị trí

```cpp
rewinddir(dir);           // Quay ve dau luong thu muc
long pos = telldir(dir);  // Luu vi tri doc
// ... doc them cac muc ...
seekdir(dir, pos);        // Tro lai vi tri da luu
```

`pos` là dấu vị trí, không phải số thứ tự file. Khi các mục trong thư mục thay đổi, không nên giả định vị trí đã lưu vẫn tương ứng với cùng một mục.

## 11. Muốn duyệt cả các thư mục con thì sao?

`readdir()` **chỉ duyệt một cấp**. Để duyệt sâu nhiều cấp bằng API POSIX, phải tự mở các thư mục con rồi duyệt tiếp (thường bằng đệ quy).

Trong **C++17** có cách ngắn gọn hơn:

```cpp
#include <filesystem>
#include <iostream>

int main() {
    for (const auto& entry :
         std::filesystem::recursive_directory_iterator("./data")) {
        std::cout << entry.path() << '\n';
    }
}
```

Ví dụ này duyệt các mục trong mọi cấp thư mục con, trừ khi gặp lỗi quyền truy cập hoặc lỗi hệ thống file. Nếu chỉ muốn một cấp, dùng `std::filesystem::directory_iterator`.

## 12. Các hàm liên quan khác

| Hàm | Tác dụng | Header thường dùng |
|---|---|---|
| `mkdir()` | Tạo thư mục | `<sys/stat.h>` |
| `rmdir()` | Xóa thư mục rỗng | `<unistd.h>` |
| `chdir()` | Đổi thư mục làm việc | `<unistd.h>` |
| `getcwd()` | Lấy đường dẫn thư mục làm việc | `<unistd.h>` |
| `stat()` | Lấy thông tin file/thư mục | `<sys/stat.h>` |
| `lstat()` | Lấy thông tin, không đi theo symbolic link cuối đường dẫn | `<sys/stat.h>` |

Ví dụ:

```cpp
mkdir("new_folder", 0755);
chdir("new_folder");
// ...
```

## 13. Phân biệt `DIR*`, `dirent*` và `FILE*`

| Kiểu | Dùng để |
|---|---|
| `DIR*` | Mở và duyệt thư mục |
| `struct dirent*` | Lấy thông tin một mục trong thư mục |
| `FILE*` | Mở và đọc/ghi **nội dung** file |

Ví dụ:

```cpp
DIR* dir = opendir("./data");       // Mo thu muc
// struct dirent* e = readdir(dir); // Lay ten mot muc
// closedir(dir);                  // Dong thu muc

FILE* f = fopen("./data/a.txt", "r"); // Mo file de doc noi dung
if (f != nullptr) fclose(f);
if (dir != nullptr) closedir(dir);
```

## 14. Ghi nhớ

1. `opendir()` **mở** thư mục.
2. `readdir()` **lần lượt trả về các mục** trực tiếp bên trong thư mục đó.
3. `entry->d_name` lấy **tên** mục; không đọc nội dung file.
4. Muốn xem tên file trong các thư mục con, phải **duyệt đệ quy**.
5. `closedir()` đóng luồng thư mục khi hoàn tất.
