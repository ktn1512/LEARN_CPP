# PHẦN I — NỀN TẢNG NGÔN NGỮ C++

> **Giáo trình lý thuyết và thực hành.** Bao quát ba nhóm kiến thức: **(1) Pointer, Reference, Smart Pointer; (2) Standard Template Library — Containers, Iterator, Algorithms; (3) Template và Generic Programming.** Tài liệu sử dụng ví dụ C++ tổng quát, không phụ thuộc một dự án cụ thể.
>
> **Chuẩn khuyến nghị:** C++17 cho phần lớn ví dụ; C++20 cho `std::jthread` không xuất hiện ở đây, và đặc biệt **Concepts**. Một số nội dung có từ C++11/14.

## Mục tiêu học tập

Sau khi học xong, người đọc cần có khả năng:

1. Phân biệt **giá trị — địa chỉ — con trỏ — tham chiếu**, thao tác an toàn với bộ nhớ, xác định ai sở hữu tài nguyên.
2. Chọn đúng **container STL** theo đặc điểm dữ liệu và độ phức tạp; duyệt, tìm kiếm, lọc, biến đổi bằng **iterator/algorithm**.
3. Viết **function template**, **class template**, tham số template không phải kiểu, variadic template; hiểu `std::move`, `std::forward`, type traits và concepts.
4. Nhận diện lỗi phổ biến: dereference null, dangling pointer/reference, memory leak, iterator invalidation, truy cập vượt biên, sai vòng đời và dùng sai move/forward.

### Biên dịch ví dụ

Dùng GCC hoặc Clang trên Linux:

```bash
g++ -std=c++20 -Wall -Wextra -Wpedantic -O0 -g main.cpp -o main
./main
```

Khi học về lỗi bộ nhớ có thể bật sanitizer:

```bash
g++ -std=c++20 -Wall -Wextra -g \
    -fsanitize=address,undefined -fno-omit-frame-pointer \
    main.cpp -o main
./main
```

> **Quy ước:** Mỗi khối code có `main()` là ví dụ độc lập; không gộp nhiều `main()` vào cùng một file. Các ví dụ không có `main()` minh họa cú pháp hoặc định nghĩa để tái sử dụng.

---

# CHƯƠNG 1. POINTER, REFERENCE VÀ SMART POINTER

## 1.1. Giá trị, địa chỉ và mô hình bộ nhớ

Một biến có **kiểu dữ liệu**, **giá trị**, **vùng lưu trữ** và **vòng đời**.

```cpp
#include <iostream>

int main() {
    int x = 42;
    std::cout << "Gia tri: " << x << '\n';
    std::cout << "Dia chi: " << &x << '\n';
}
```

- `x`: giá trị của biến.
- `&x`: địa chỉ vùng nhớ chứa biến.
- `sizeof(x)`: số byte của kiểu đối tượng (phụ thuộc nền tảng).
- Địa chỉ in ra khác nhau tùy lần chạy và môi trường.

**Phân loại theo thời gian lưu trữ (storage duration):**

| Loại | Ví dụ | Đặc điểm |
|---|---|---|
| Automatic | Biến cục bộ `int x;` | Thường nằm trên stack; tự hủy khi ra khỏi scope |
| Static | Biến toàn cục hoặc `static int x;` | Tồn tại suốt chương trình |
| Thread-local | `thread_local int x;` | Mỗi thread có một bản riêng |
| Dynamic | `new int(5)` | Tồn tại đến khi giải phóng; thường nằm trên heap |

> C++ quy định **storage duration**, không bắt buộc máy phải triển khai automatic trên stack hay dynamic trên một heap cụ thể. Stack/heap là mô hình giải thích thực tế thường gặp.

## 1.2. Pointer là gì? Cú pháp khai báo

**Con trỏ** là một đối tượng lưu địa chỉ (hoặc giá trị null), cho phép truy cập gián tiếp đến đối tượng/hàm.

```cpp
#include <iostream>

int main() {
    int x = 10;
    int* p = &x;     // p lưu địa chỉ x

    std::cout << x << '\n';    // 10
    std::cout << *p << '\n';   // 10: giải tham chiếu

    *p = 25;                  // sửa x thông qua p
    std::cout << x << '\n';    // 25
}
```

**Hai toán tử rất dễ nhầm:**

| Cú pháp | Nghĩa |
|---|---|
| `&x` (biểu thức) | Lấy địa chỉ của `x` |
| `*p` (biểu thức) | Truy cập đối tượng mà `p` trỏ tới |
| `int* p` (khai báo) | Khai báo `p` là con trỏ đến `int` |
| `int& r` (khai báo) | Khai báo `r` là tham chiếu đến `int` |

Dấu `*` trong **khai báo** không phải thao tác giải tham chiếu.

**Đọc khai báo đúng:**

```cpp
int* p = nullptr;
int *a = nullptr, b = 10;  // a là pointer, b là int
```

Vì vậy nên khai báo mỗi biến trên một dòng để tránh nhầm kiểu.

## 1.3. Con trỏ null, kiểm tra trước khi sử dụng

`nullptr` (C++11) biểu diễn một giá trị con trỏ null có kiểu riêng.

```cpp
#include <iostream>

void printValue(const int* p) {
    if (p == nullptr) {
        std::cout << "Khong co doi tuong\n";
        return;
    }
    std::cout << *p << '\n';
}

int main() {
    int x = 9;
    printValue(&x);
    printValue(nullptr);
}
```

- Không được giải tham chiếu con trỏ null (`*p` hoặc `p->member`).
- Tránh con trỏ chưa khởi tạo: `int* p;` rồi sử dụng là nguy hiểm.
- Dùng `nullptr` thay vì `NULL`/`0` trong C++ hiện đại.
- **Không phải cứ khác null là hợp lệ:** con trỏ treo vẫn có thể khác `nullptr`.

## 1.4. Con trỏ đến `struct` và `class`; toán tử `->`

```cpp
#include <iostream>
#include <string>

struct Device {
    std::string name;
    int id;
};

int main() {
    Device d{"Sensor", 7};
    Device* p = &d;

    std::cout << (*p).name << '\n';
    std::cout << p->id << '\n';  // tương đương (*p).id

    p->id = 8;
    std::cout << d.id << '\n';
}
```

- `obj.member`: truy cập thành viên từ đối tượng.
- `ptr->member`: truy cập thành viên từ con trỏ.
- `(*ptr).member`: cách viết tương đương `ptr->member`.
- Cần ngoặc quanh `*ptr` trong `(*ptr).member` vì `.` có độ ưu tiên cao hơn `*`.

## 1.5. Con trỏ cấp hai (`T**`)

Con trỏ cấp hai lưu địa chỉ của một biến con trỏ.

```cpp
#include <iostream>

int main() {
    int x = 5;
    int* p = &x;
    int** pp = &p;

    **pp = 99;
    std::cout << x << '\n';  // 99
}
```

**Trường hợp sử dụng:** khi hàm cần **thay đổi chính con trỏ của bên gọi**; quản lý mảng con trỏ; một số giao diện thư viện C.

```cpp
void pointTo(int** output, int* target) {
    if (output) {
        *output = target;
    }
}

// Dùng:
// int x = 10;
// int* p = nullptr;
// pointTo(&p, &x);
```

Trong C++ thường có cách rõ ràng hơn là truyền `int*&` hoặc trả về `int*`. Không dùng `T**` chỉ vì muốn thay đổi **giá trị** mà một `T*` đang trỏ tới.

## 1.6. Con trỏ `void*`

`void*` là con trỏ không mang thông tin về kiểu của đối tượng đích tại chỗ sử dụng.

```cpp
#include <iostream>

int main() {
    int value = 123;
    void* raw = &value;

    // Phải chuyển về đúng kiểu rồi mới giải tham chiếu.
    int* typed = static_cast<int*>(raw);
    std::cout << *typed << '\n';
}
```

**Ứng dụng:** giao diện C, callback C API, giao diện truyền bộ nhớ thô.

**Giới hạn:** không thể `*raw` trực tiếp; phải bảo đảm kiểu chuyển đúng, alignment đúng và đối tượng còn sống. Không dùng `void*` thay thế thiết kế có kiểu nếu không cần.

## 1.7. `const` và Pointer: ba trường hợp

```cpp
int x = 10;
int y = 20;

const int* p1 = &x;  // không sửa *p1 qua p1; có thể đổi p1
int* const p2 = &x;  // không đổi địa chỉ p2; có thể sửa *p2
const int* const p3 = &x; // không đổi p3 hoặc sửa *p3 qua p3

p1 = &y;            // OK
// *p1 = 7;         // LỖI BIÊN DỊCH

*p2 = 7;            // OK
// p2 = &y;         // LỖI BIÊN DỊCH
```

| Cú pháp | Đổi địa chỉ đang giữ? | Sửa giá trị qua con trỏ? |
|---|---|---|
| `int* p` | Có | Có |
| `const int* p` | Có | Không |
| `int* const p` | Không | Có |
| `const int* const p` | Không | Không |

**Lưu ý:** `const int*` ngăn sửa **qua con trỏ đó**, không bảo đảm đối tượng không thể bị sửa qua tên/alias khác nếu vốn không phải đối tượng const.

## 1.8. Function Pointer (con trỏ hàm)

Con trỏ hàm giữ địa chỉ một hàm có chữ ký tương thích.

```cpp
#include <iostream>

int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

int calculate(int a, int b, int (*operation)(int, int)) {
    return operation(a, b);
}

int main() {
    int (*fn)(int, int) = &add;
    std::cout << fn(2, 3) << '\n';                  // 5
    std::cout << calculate(2, 3, &multiply) << '\n'; // 6
}
```

**Cách đọc:** `int (*fn)(int, int)` là con trỏ đến hàm nhận hai `int` và trả `int`.

Có thể viết dễ đọc hơn:

```cpp
using BinaryOperation = int (*)(int, int);
BinaryOperation op = &add;
```

**Ứng dụng:** callback, bảng dispatch theo loại yêu cầu, chọn thuật toán khi chạy. Hàm thành viên **không static** có kiểu con trỏ thành viên khác con trỏ hàm thông thường.

## 1.9. Pointer và Array; phép toán con trỏ

```cpp
#include <iostream>

int main() {
    int a[] = {10, 20, 30};
    int* p = a;  // tên mảng thường chuyển thành pointer tới phần tử đầu

    std::cout << p[0] << '\n';     // 10
    std::cout << *(p + 1) << '\n'; // 20
    std::cout << p[2] << '\n';     // 30
}
```

- `p + 1` trỏ đến **phần tử kế tiếp**, tự tính theo kích thước kiểu.
- Phép toán con trỏ chỉ hợp lệ trong phạm vi cùng mảng hoặc một-past-the-end theo quy tắc ngôn ngữ.
- Pointer tới phần tử **ngay sau cuối mảng** có thể dùng để so sánh, **không được giải tham chiếu**.
- Mảng C và `std::vector` không phải cùng kiểu; hãy dùng `std::array` hoặc `std::vector` khi phù hợp.

## 1.10. Bộ nhớ động: `new`, `delete`, `new[]`, `delete[]`

```cpp
#include <iostream>

int main() {
    int* p = new int(42);
    std::cout << *p << '\n';

    delete p;
    p = nullptr;

    int* a = new int[3]{1, 2, 3};
    std::cout << a[1] << '\n';

    delete[] a;
    a = nullptr;
}
```

| Cấp phát | Giải phóng đúng |
|---|---|
| `new T(...)` | `delete p` |
| `new T[n]` | `delete[] p` |

**Nguyên tắc:**

- Không dùng `delete[]` với vùng cấp phát bằng `new T` và ngược lại.
- Không `delete` vùng nhớ không cấp phát bằng `new`.
- Không `delete` cùng tài nguyên hai lần.
- Nếu constructor/allocation ném exception, phải bảo đảm không rò rỉ tài nguyên.
- Trong code C++ hiện đại, ưu tiên `std::vector`, `std::unique_ptr`, `std::make_unique` thay cho `new`/`delete` trực tiếp.

## 1.11. Dangling Pointer, Memory Leak, Use-after-free

### Dangling Pointer

```cpp
int* badAddress() {
    int local = 10;
    return &local;  // SAI: local hết vòng đời sau return
}
```

Đoạn này chỉ minh họa **lỗi**, không gọi hàm trong chương trình thực tế.

### Use-after-free

```cpp
int* p = new int(8);
delete p;
// std::cout << *p; // SAI: truy cập đối tượng đã bị hủy
p = nullptr;
```

### Memory Leak

```cpp
void leak() {
    int* p = new int(10);
    // Không delete: rò rỉ vùng cấp phát khi mất con trỏ cuối cùng.
}
```

**Phòng tránh:** ưu tiên RAII, tránh trả địa chỉ biến cục bộ, không lưu con trỏ qua vòng đời đối tượng, dùng sanitizer để kiểm tra.

## 1.12. Reference: `T&` và truyền tham số

**Reference** là một tên tham chiếu đến đối tượng đã có; về cách dùng giống alias.

```cpp
#include <iostream>

void increase(int& x) {
    ++x;
}

void print(const int& x) {
    std::cout << x << '\n';
}

int main() {
    int number = 10;
    int& ref = number;

    ref = 20;        // number cũng thành 20
    increase(number); // 21
    print(number);
}
```

**Lưu ý:**

- `T&` thông thường phải được khởi tạo khi khai báo.
- Gán `ref = other` là **gán giá trị** cho đối tượng mà `ref` tham chiếu, không phải chuyển reference sang đối tượng khác.
- Reference cũng có thể treo nếu đối tượng được tham chiếu đã bị hủy.
- `const T&` thích hợp để đọc đối tượng lớn mà tránh sao chép.
- Đối với kiểu nhỏ (như `int`), truyền theo giá trị thường đơn giản hơn.

### Pass by value, pointer, reference

```cpp
void byValue(int x) { x = 100; }        // không đổi biến gốc
void byPointer(int* x) { if (x) *x = 100; }
void byReference(int& x) { x = 100; }    // đổi biến gốc
```

Dùng **pointer** khi cho phép truyền `nullptr` hoặc cần biểu đạt sự tùy chọn; **reference** khi luôn yêu cầu đối tượng hợp lệ.

## 1.13. Lvalue, Rvalue và `T&&`

Hiểu giản lược:

- **Lvalue:** biểu thức định danh một đối tượng/hàm với danh tính; ví dụ tên biến `x`.
- **Prvalue:** giá trị tính toán như `5`, `x + 1`.
- **Xvalue:** giá trị có danh tính nhưng có thể được coi là sắp hết sử dụng, như `std::move(x)`.

```cpp
#include <string>
#include <utility>

int main() {
    std::string a = "hello";

    std::string& lref = a;                  // lvalue reference
    const std::string& cref = std::string("temporary"); // OK
    std::string&& rref = std::string("temporary");      // rvalue reference

    // std::string& bad = std::string("x");  // sai
    (void)lref;
    (void)cref;
    (void)rref;
}
```

**`T&&`** giúp triển khai move semantics và overload xử lý rvalue.

### Lifetime Extension: điều cần nhớ

```cpp
const std::string& a = std::string("hello");
std::string&& b = std::string("world");
```

Trong các khai báo này, temporary được kéo dài vòng đời đến hết vòng đời của reference tương ứng. **Không được tổng quát hóa:** trả về reference đến temporary, reference trong vài trường hợp khởi tạo khác, hoặc tham chiếu qua function parameter có quy tắc vòng đời riêng. Tham chiếu **không sở hữu** đối tượng.

## 1.14. So sánh Pointer và Reference

| Thuộc tính | Pointer | Reference |
|---|---|---|
| Khai báo | `T* p` | `T& r` |
| Có thể null? | Có | Tham chiếu hợp lệ phải chỉ đến đối tượng/hàm |
| Đổi sang đối tượng khác? | Có | Không (gán sẽ gán giá trị) |
| Truy cập | `*p` / `p->member` | `r` / `r.member` |
| Cho phép tham số tùy chọn? | Phù hợp | Không tự biểu thị “không có” |
| Sở hữu vùng nhớ? | Không, nếu chỉ là raw pointer | Không |

## 1.15. RAII — quản lý tài nguyên theo vòng đời

**RAII (Resource Acquisition Is Initialization):** Gắn tài nguyên (bộ nhớ, file, mutex, socket...) với đối tượng có destructor chịu trách nhiệm giải phóng.

```cpp
#include <fstream>
#include <string>

void writeLog(const std::string& path) {
    std::ofstream file(path);  // mở tài nguyên
    if (!file) return;
    file << "Hello\n";
} // file đóng tự động, kể cả khi ra scope sớm
```

**Lợi ích:** giảm rò rỉ, an toàn hơn khi có exception, rõ ownership.

## 1.16. `std::unique_ptr`

`std::unique_ptr<T>` sở hữu độc quyền một đối tượng cấp phát động.

```cpp
#include <iostream>
#include <memory>
#include <utility>

struct Worker {
    void run() const { std::cout << "Working\n"; }
};

int main() {
    auto owner1 = std::make_unique<Worker>();
    owner1->run();

    auto owner2 = std::move(owner1);  // chuyển quyền sở hữu
    if (!owner1) std::cout << "owner1 empty\n";

    owner2->run();
} // tự hủy Worker
```

Các thao tác:

- `std::make_unique<T>(args...)`: tạo an toàn và ngắn gọn.
- `p.get()`: lấy raw pointer **không chuyển quyền sở hữu**.
- `p.reset()`: giải phóng/đổi tài nguyên đang sở hữu.
- `p.release()`: bỏ quyền sở hữu và trả raw pointer; bên nhận phải quản lý.
- `std::move(p)`: chuyển quyền sở hữu khi move thành công.

**Quy tắc:** không dùng `release()` trừ khi thật sự có một cơ chế nhận ownership khác.

### Mảng động

```cpp
auto values = std::make_unique<int[]>(5);
values[0] = 10;
```

Nếu cần kích thước thay đổi, duyệt và quản lý nhiều phần tử, `std::vector<int>` thường thuận tiện hơn.

## 1.17. `std::shared_ptr` và Reference Counting

`std::shared_ptr` thể hiện **đồng sở hữu**. Đối tượng được hủy sau khi không còn shared owner nào (và điều kiện liên quan deleter/control block thỏa mãn).

```cpp
#include <iostream>
#include <memory>

int main() {
    auto first = std::make_shared<int>(42);
    auto second = first;

    std::cout << *second << '\n';         // 42
    std::cout << first.use_count() << '\n'; // 2 (ở đây)
    second.reset();
    std::cout << first.use_count() << '\n'; // 1
}
```

**Chú ý:**

- Sao chép `shared_ptr` tăng số shared owner.
- `use_count()` hữu ích để học, không nên dựa vào làm cơ chế đồng bộ trạng thái giữa thread.
- Không tạo **hai `shared_ptr` độc lập từ cùng một raw pointer** vì có nguy cơ double delete.
- Sử dụng `make_shared` cho trường hợp thông thường.
- Shared ownership có chi phí quản lý; đừng dùng `shared_ptr` khi `unique_ptr` là đủ.

## 1.18. `std::weak_ptr` và vòng tham chiếu (circular reference)

`std::weak_ptr` quan sát đối tượng đang được quản lý bởi `shared_ptr` nhưng **không làm tăng số chủ sở hữu**.

```cpp
#include <iostream>
#include <memory>

int main() {
    auto owner = std::make_shared<int>(50);
    std::weak_ptr<int> observer = owner;

    if (auto safe = observer.lock()) {
        std::cout << *safe << '\n'; // 50
    }

    owner.reset();

    if (observer.expired()) {
        std::cout << "Object destroyed\n";
    }
}
```

**Circular ownership:** nếu A chứa `shared_ptr<B>` và B chứa `shared_ptr<A>`, cả hai có thể giữ nhau sống mãi dù bên ngoài không còn dùng. Thiết kế một chiều thành `weak_ptr` nếu đó chỉ là quan hệ quan sát/quay về đối tượng cha.

## 1.19. Ownership — chọn loại con trỏ nào?

| Nhu cầu | Khuyến nghị |
|---|---|
| Biến chỉ sống cục bộ | Biến giá trị `T obj;` |
| Mảng dữ liệu co giãn | `std::vector<T>` |
| Sở hữu độc quyền đối tượng | `std::unique_ptr<T>` |
| Nhiều thành phần thực sự cùng sở hữu | `std::shared_ptr<T>` |
| Quan sát đối tượng `shared_ptr` không sở hữu | `std::weak_ptr<T>` |
| Truy cập đối tượng không sở hữu, có thể null | `T*` |
| Truy cập đối tượng không sở hữu, bắt buộc có | `T&` / `const T&` |

**Trình tự ưu tiên:** giá trị → tham chiếu → `unique_ptr` → `shared_ptr` nếu thực sự cần.

## 1.20. Tổng kết chương 1

Tự kiểm tra:

1. Vì sao `*p` có thể gây undefined behavior dù `p != nullptr`?
2. Phân biệt `const int*` và `int* const`.
3. Vì sao không trả `&localVariable`?
4. Khi nào truyền `const T&`, `T*`, `T&`?
5. Điểm khác giữa `get()`, `release()`, `reset()`?
6. Vì sao `weak_ptr` giải quyết vòng shared ownership?

---

# CHƯƠNG 2. STANDARD TEMPLATE LIBRARY (STL)

## 2.1. STL gồm những thành phần gì?

Trong cách gọi thông dụng, STL bao gồm:

1. **Containers:** nơi lưu các phần tử.
2. **Iterators:** khái niệm vị trí/đường duyệt phần tử.
3. **Algorithms:** thuật toán tổng quát thao tác trên range.
4. **Function objects / Comparators:** cung cấp hành vi tùy chỉnh.
5. **Allocators:** chiến lược cấp phát phục vụ container.

**Phân loại container:**

- **Sequence:** `vector`, `array`, `list`, `deque`.
- **Ordered associative:** `set`, `map`.
- **Unordered associative:** `unordered_set`, `unordered_map`.
- **Container adapters:** `stack`, `queue`, `priority_queue`.

## 2.2. `std::vector<T>` — mảng động

**Header:** `<vector>`. **Đặc điểm:** phần tử liên tiếp trong bộ nhớ, truy cập theo chỉ số O(1), thêm cuối trung bình khấu hao O(1), chèn/xóa giữa O(n).

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> a;                // rỗng
    std::vector<int> b{10, 20, 30};   // danh sách
    std::vector<int> c(3, 7);         // 7, 7, 7

    b.push_back(40);
    b.emplace_back(50);

    std::cout << "size=" << b.size()
              << " capacity=" << b.capacity() << '\n';

    b.reserve(100);  // ít nhất 100 chỗ; KHÔNG thêm phần tử
    b.resize(7);     // thêm phần tử (int mới = 0)
    std::cout << b.at(1) << '\n';  // 20, kiểm tra biên

    for (const int value : b) std::cout << value << ' ';
    std::cout << '\n';
    (void)a;
    (void)c;
}
```

**Phân biệt:**

| Hàm | Tác động |
|---|---|
| `size()` | Số phần tử hiện có |
| `capacity()` | Dung lượng đã cấp phát |
| `reserve(n)` | Tăng capacity tối thiểu đến n nếu cần; size không đổi |
| `resize(n)` | Đổi size, tạo/xóa phần tử |
| `operator[]` | Truy cập không kiểm tra biên |
| `at()` | Truy cập có kiểm tra biên, ném ngoại lệ nếu sai |
| `push_back(v)` | Thêm cuối |
| `insert(pos,v)` | Chèn trước vị trí |
| `erase(pos)` | Xóa tại vị trí |
| `clear()` | Hủy mọi phần tử; không bắt buộc thu hồi capacity |

**Bẫy:** `reserve(100)` **không cho phép** dùng `v[99]` nếu `size()` vẫn nhỏ hơn 100.

## 2.3. `std::array<T,N>` — mảng cố định

**Header:** `<array>`. Kích thước biết tại compile-time; quản lý giống đối tượng giá trị.

```cpp
#include <array>
#include <iostream>

int main() {
    std::array<int, 4> values{1, 2, 3, 4};
    values[0] = 10;
    std::cout << values.size() << '\n'; // 4
    for (int x : values) std::cout << x << ' ';
}
```

**Nên dùng:** khi số phần tử cố định. Không có `push_back()`.

## 2.4. `std::list<T>` — danh sách liên kết đôi

**Header:** `<list>`. Chèn/xóa tại iterator hợp lệ O(1); không có truy cập bằng `operator[]`; truy cập vị trí thứ k thường O(n).

```cpp
#include <iostream>
#include <list>

int main() {
    std::list<int> items{3, 1, 2};
    items.push_front(5);
    items.push_back(4);

    auto it = items.begin();
    ++it;
    items.insert(it, 9);
    items.sort(); // dùng list::sort(), KHÔNG dùng std::sort

    for (int x : items) std::cout << x << ' ';
}
```

**Chú ý:** O(1) chèn/xóa áp dụng **khi đã có iterator**, còn tìm vị trí có thể O(n). `list` tốn bộ nhớ node, ít locality hơn `vector`; đừng mặc định list nhanh hơn.

## 2.5. `std::deque<T>` — hàng đợi hai đầu

**Header:** `<deque>`. Hỗ trợ `push_front`, `push_back` nhanh (amortized O(1)) và truy cập chỉ số O(1); không đảm bảo phần tử liên tiếp trong bộ nhớ.

```cpp
#include <deque>
#include <iostream>

int main() {
    std::deque<int> d{2, 3};
    d.push_front(1);
    d.push_back(4);
    d.pop_front();

    std::cout << d.front() << ' ' << d.back() << '\n'; // 2 4
}
```

## 2.6. `std::set<T>` — tập hợp được sắp xếp

**Header:** `<set>`. Các giá trị là duy nhất theo bộ so sánh; duyệt theo thứ tự; tìm/chèn/xóa thường O(log n).

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s{5, 2, 5, 1};
    s.insert(3);

    if (s.find(2) != s.end()) {
        std::cout << "Found 2\n";
    }

    s.erase(5);
    for (int x : s) std::cout << x << ' '; // 1 2 3
}
```

Tùy chỉnh thứ tự:

```cpp
#include <functional>
#include <set>

std::set<int, std::greater<int>> descending{3, 1, 2};
```

**Lưu ý:** không thể sửa trực tiếp giá trị phần tử `set` thông qua iterator vì có thể phá tính đúng đắn của thứ tự.

## 2.7. `std::unordered_set<T>` — tập hợp dựa trên hash

**Header:** `<unordered_set>`. Không bảo đảm thứ tự duyệt; tìm/chèn/xóa trung bình O(1), trường hợp xấu O(n).

```cpp
#include <iostream>
#include <string>
#include <unordered_set>

int main() {
    std::unordered_set<std::string> tags{"cpp", "linux"};
    tags.insert("network");

    if (tags.contains("cpp")) { // C++20
        std::cout << "Exist\n";
    }
}
```

Dùng C++17: thay `tags.contains("cpp")` bằng `tags.find("cpp") != tags.end()`.

**Chọn `set` hay `unordered_set`?**

- Cần thứ tự, `lower_bound`, duyệt có sắp xếp: `set`.
- Chủ yếu kiểm tra tồn tại, không quan tâm thứ tự: cân nhắc `unordered_set`.
- `unordered_set` cần hash và equality phù hợp; rehash có thể làm vô hiệu iterator.

## 2.8. `std::map<Key,Value>` — key-value có thứ tự

**Header:** `<map>`. Key duy nhất và sắp xếp; tìm/chèn/xóa thường O(log n).

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> scores;
    scores["An"] = 8;          // tạo entry nếu chưa tồn tại
    scores.insert({"Binh", 9});
    scores.try_emplace("Chi", 10);

    if (auto it = scores.find("Binh"); it != scores.end()) {
        std::cout << it->first << ": " << it->second << '\n';
    }

    for (const auto& [name, score] : scores) {
        std::cout << name << '=' << score << '\n';
    }
}
```

**Bẫy `operator[]`:** `scores["Unknown"]` có thể **tạo entry mới** với giá trị được khởi tạo mặc định. Nếu chỉ muốn đọc, ưu tiên `find()` hoặc `at()`.

## 2.9. `std::unordered_map<Key,Value>` — hash map

**Header:** `<unordered_map>`. Không sắp xếp key; tra cứu trung bình O(1).

```cpp
#include <iostream>
#include <string>
#include <unordered_map>

int main() {
    std::unordered_map<std::string, int> count;
    ++count["apple"];
    ++count["banana"];
    ++count["apple"];

    std::cout << count.at("apple") << '\n'; // 2
}
```

**Nên dùng:** bảng đếm, tra cứu ID → đối tượng, bộ nhớ đệm khi không cần duyệt theo thứ tự. Lưu ý yêu cầu hash và nguy cơ iterator invalidation sau rehash.

## 2.10. `std::queue<T>` — FIFO

**Header:** `<queue>`. Vào sau xếp cuối, lấy ra từ đầu.

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;
    q.push(10);
    q.push(20);

    while (!q.empty()) {
        std::cout << q.front() << ' ';
        q.pop();
    }
}
```

Queue là **container adapter**; không có iterator trực tiếp và không hỗ trợ truy cập tùy ý.

## 2.11. `std::stack<T>` — LIFO

**Header:** `<stack>`. Phần tử vào sau được lấy ra trước.

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> s;
    s.push(1);
    s.push(2);

    while (!s.empty()) {
        std::cout << s.top() << ' ';
        s.pop();
    } // 2 1
}
```

**Ứng dụng:** duyệt cây/đồ thị, undo, parse biểu thức.

## 2.12. `std::priority_queue<T>` — lấy phần tử ưu tiên cao nhất

**Header:** `<queue>`. Mặc định phần tử lớn nhất nằm ở `top()`. Thêm/bỏ đỉnh O(log n); xem đỉnh O(1).

```cpp
#include <functional>
#include <iostream>
#include <queue>
#include <vector>

int main() {
    std::priority_queue<int> maxHeap;
    for (int x : {2, 9, 4}) maxHeap.push(x);
    std::cout << maxHeap.top() << '\n'; // 9

    std::priority_queue<int,
                        std::vector<int>,
                        std::greater<int>> minHeap;
    for (int x : {2, 9, 4}) minHeap.push(x);
    std::cout << minHeap.top() << '\n'; // 2
}
```

**Ứng dụng:** lập lịch ưu tiên, xử lý tác vụ theo trọng số, thuật toán đồ thị.

## 2.13. Bảng chọn container và độ phức tạp

| Container | Truy cập theo vị trí | Tìm key/value | Thêm/xóa đặc trưng |
|---|---|---|---|
| `vector` | O(1) theo chỉ số | O(n) nếu không sắp xếp | Thêm cuối amortized O(1); giữa O(n) |
| `array` | O(1) | O(n) | Kích thước cố định |
| `list` | O(n) | O(n) | O(1) tại iterator đã biết |
| `deque` | O(1) theo chỉ số | O(n) | Hai đầu amortized O(1) |
| `set` | Không có chỉ số | O(log n) | O(log n) |
| `unordered_set` | Không có chỉ số | Trung bình O(1) | Trung bình O(1) |
| `map` | Theo key O(log n) | O(log n) | O(log n) |
| `unordered_map` | Theo key trung bình O(1) | Trung bình O(1) | Trung bình O(1) |
| `queue` | `front()` O(1) | Không hỗ trợ tìm trực tiếp | `push/pop` thường O(1) với deque mặc định |
| `stack` | `top()` O(1) | Không hỗ trợ tìm trực tiếp | `push/pop` thường O(1) |
| `priority_queue` | `top()` O(1) | Không hỗ trợ tìm trực tiếp | `push/pop` O(log n) |

Độ phức tạp là đặc tính thuật toán chuẩn theo điều kiện đã nêu; hiệu năng thực tế còn phụ thuộc bộ nhớ đệm CPU, cấp phát và kích thước dữ liệu.

## 2.14. Iterator: định nghĩa và cách sử dụng

Iterator mô tả vị trí trong một range; có thể truy cập phần tử bằng `*it` và di chuyển bằng `++it` khi phù hợp.

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> data{10, 20, 30};

    for (auto it = data.begin(); it != data.end(); ++it) {
        std::cout << *it << ' ';
    }
    std::cout << '\n';

    for (const int& value : data) {
        std::cout << value << ' ';
    }
}
```

**Về `end()`:** iterator `end()` chỉ **vị trí sau phần tử cuối**, không phải phần tử cuối; không được `*container.end()`.

## 2.15. Các loại Iterator

| Loại | Khả năng chính | Ví dụ |
|---|---|---|
| Input iterator | Đọc tiến về trước | Input stream |
| Output iterator | Ghi khi duyệt | Output stream, inserter |
| Forward iterator | Duyệt nhiều lần theo một chiều | `forward_list`, unordered container (ít nhất) |
| Bidirectional iterator | Tăng và giảm | `list`, `set`, `map` |
| Random-access iterator | Nhảy vị trí, cộng/trừ độ dời | `vector`, `deque` |
| Contiguous iterator (C++20) | Phần tử liên tiếp trong bộ nhớ | `vector` (trừ `vector<bool>`), `array` |

**Quan trọng:** `it + 2` hợp lệ với random-access iterator, nhưng không phải với iterator của `list`/`set`.

## 2.16. `begin/end`, `cbegin/cend`, `rbegin/rend`

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3};

    for (auto it = v.cbegin(); it != v.cend(); ++it) {
        std::cout << *it << ' ';  // chỉ đọc
    }
    std::cout << '\n';

    for (auto it = v.rbegin(); it != v.rend(); ++it) {
        std::cout << *it << ' ';  // 3 2 1
    }
}
```

- `cbegin()/cend()` cấp iterator không cho sửa phần tử qua iterator.
- `rbegin()/rend()` duyệt ngược (nếu container hỗ trợ).
- `std::begin(container)` và `std::end(container)` là hàm trợ giúp tổng quát.

## 2.17. Iterator Invalidation — khi iterator không còn hợp lệ

Khi container thay đổi, iterator/reference/pointer tới phần tử **có thể** bị vô hiệu.

| Container và thao tác | Ảnh hưởng điển hình |
|---|---|
| `vector::push_back` gây reallocation | Mọi iterator/reference/pointer vào phần tử đều mất hiệu lực |
| `vector::push_back` không reallocation | `end()` cũ mất hiệu lực; iterator/reference tới phần tử trước đó còn hợp lệ |
| `vector::erase(pos)` | Iterator/reference tại hoặc sau vị trí xóa mất hiệu lực |
| `list::insert` | Iterator/reference cũ thường giữ hiệu lực |
| `list::erase(pos)` | Chỉ iterator/reference đến phần tử bị xóa mất hiệu lực |
| `map/set::insert` | Các iterator/reference đang có vẫn hợp lệ |
| `map/set::erase` | Iterator/reference đến phần tử bị xóa mất hiệu lực |
| `unordered_map/set` rehash | Iterator mất hiệu lực; reference/pointer đến phần tử còn hợp lệ |
| `deque` | Quy tắc phụ thuộc thao tác/vị trí; cần xem đúng API |

**Ví dụ lỗi cần tránh:**

```cpp
std::vector<int> v{1, 2, 3};
auto it = v.begin();
v.reserve(100);  // nếu có cấp phát lại, it có thể mất hiệu lực
// std::cout << *it; // KHÔNG dùng it cũ khi bị invalidation
```

Khi xóa trong khi duyệt `vector`:

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};

    for (auto it = v.begin(); it != v.end();) {
        if (*it % 2 == 0) {
            it = v.erase(it);  // nhận iterator mới hợp lệ
        } else {
            ++it;
        }
    }

    for (int x : v) std::cout << x << ' '; // 1 3 5
}
```

## 2.18. STL Algorithms: `find`, `find_if`, `sort`

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{5, 3, 9, 2, 9};

    auto x = std::find(v.begin(), v.end(), 3);
    if (x != v.end()) std::cout << "Found " << *x << '\n';

    auto y = std::find_if(v.begin(), v.end(),
                          [](int n) { return n > 6; });
    if (y != v.end()) std::cout << "First > 6: " << *y << '\n';

    std::sort(v.begin(), v.end()); // 2 3 5 9 9

    for (int n : v) std::cout << n << ' ';
}
```

- `find` tìm theo so sánh bằng.
- `find_if` tìm theo predicate.
- `sort` yêu cầu random-access iterator; với `list` dùng `list::sort()`.
- Muốn tìm nhanh trong dãy **đã sắp xếp**, học thêm `std::lower_bound`/`std::binary_search`.

## 2.19. `for_each`, `count`, `count_if`, `any_of`

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 4};

    std::for_each(v.begin(), v.end(), [](int x) {
        std::cout << x << ' ';
    });
    std::cout << '\n';

    std::cout << std::count(v.begin(), v.end(), 4) << '\n'; // 2
    std::cout << std::count_if(v.begin(), v.end(),
                              [](int x) { return x % 2 == 0; })
              << '\n'; // 3

    bool hasNegative = std::any_of(
        v.begin(), v.end(), [](int x) { return x < 0; });
    std::cout << std::boolalpha << hasNegative << '\n'; // false
}
```

Ngoài `any_of` còn có `all_of` và `none_of`.

## 2.20. `transform` và `copy`

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int main() {
    std::vector<int> source{1, 2, 3};
    std::vector<int> squares;
    squares.reserve(source.size());

    std::transform(source.begin(), source.end(),
                   std::back_inserter(squares),
                   [](int x) { return x * x; });

    std::vector<int> copy;
    std::copy(squares.begin(), squares.end(),
              std::back_inserter(copy));

    for (int x : copy) std::cout << x << ' '; // 1 4 9
}
```

**Bẫy thường gặp:** không được `std::copy(source.begin(), source.end(), result.begin())` khi `result` còn rỗng; cần cấp đủ phần tử trước hoặc dùng `std::back_inserter`.

## 2.21. `remove_if` và Erase–Remove Idiom

`std::remove_if` không tự làm giảm `size()` của `vector`. Nó dịch chuyển các phần tử cần giữ về đầu, trả iterator xác định phần logic cần xóa.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v{1, 2, 3, 4, 5, 6};

    auto newEnd = std::remove_if(
        v.begin(), v.end(),
        [](int x) { return x % 2 == 0; });

    v.erase(newEnd, v.end());

    for (int x : v) std::cout << x << ' '; // 1 3 5
}
```

Từ C++20 có thể viết ngắn hơn với `std::erase_if(v, predicate)` cho những container được hỗ trợ.

## 2.22. Lambda và Comparator trong STL

```cpp
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

struct Student {
    std::string name;
    int score;
};

int main() {
    std::vector<Student> students{
        {"An", 8}, {"Binh", 10}, {"Chi", 9}
    };

    std::sort(students.begin(), students.end(),
              [](const Student& a, const Student& b) {
                  return a.score > b.score;
              });

    for (const auto& s : students) {
        std::cout << s.name << ": " << s.score << '\n';
    }
}
```

**Comparator** của `sort` phải tuân theo quy tắc strict weak ordering; chẳng hạn nên so sánh `a.score < b.score` chứ không dùng `<=`.

## 2.23. Container nào cho bài toán nào?

| Bài toán | Container thường phù hợp |
|---|---|
| Dãy số có thể tăng kích thước | `vector` |
| Dữ liệu có kích thước cố định | `array` |
| Thêm/bớt thường xuyên ở cả hai đầu | `deque` |
| Cần thứ tự duy nhất | `set` |
| Kiểm tra tồn tại nhanh, không cần thứ tự | `unordered_set` |
| Ánh xạ khóa sang giá trị và cần thứ tự | `map` |
| Ánh xạ khóa sang giá trị, tra cứu trung bình nhanh | `unordered_map` |
| Công việc FIFO | `queue` |
| LIFO / hoàn tác | `stack` |
| Công việc ưu tiên theo trọng số | `priority_queue` |
| Cần giữ iterator ổn định khi chèn/xóa node | Xem xét `list`, `map`, `set` |

**Quy tắc thực dụng:** khi không có yêu cầu đặc biệt, bắt đầu với `std::vector` rồi chỉ đổi container sau khi rõ mục tiêu truy cập/độ phức tạp.

---

# CHƯƠNG 3. TEMPLATE VÀ GENERIC PROGRAMMING

## 3.1. Template giải quyết vấn đề gì?

Nếu cần cùng một thuật toán cho nhiều kiểu:

```cpp
int maxInt(int a, int b) { return a > b ? a : b; }
double maxDouble(double a, double b) { return a > b ? a : b; }
```

Template giúp mô tả thuật toán tổng quát:

```cpp
template <typename T>
T larger(T a, T b) {
    return (a > b) ? a : b;
}
```

Compiler tạo những specialization thích hợp khi template được dùng. **Không phải** mọi kiểu đều hợp lệ: `T` phải hỗ trợ các thao tác mà thân hàm yêu cầu, ở ví dụ này là `operator>` và sao chép/trả về.

## 3.2. Function Template

```cpp
#include <iostream>
#include <string>

template <typename T>
T larger(T a, T b) {
    return a > b ? a : b;
}

int main() {
    std::cout << larger(3, 7) << '\n';
    std::cout << larger(2.5, 1.8) << '\n';
    std::cout << larger<std::string>("cat", "dog") << '\n';
}
```

- `template <typename T>` khai báo tham số kiểu.
- Compiler có thể suy ra kiểu từ đối số: `larger(3, 7)`.
- Có thể chỉ định rõ: `larger<double>(2.0, 3.0)`.
- Truyền hai kiểu khác nhau như `larger(1, 2.5)` có thể thất bại khi template chỉ có một `T` do suy luận xung đột.

### Function Template với nhiều tham số kiểu

```cpp
template <typename A, typename B>
auto add(A a, B b) {
    return a + b;
}

// add(1, 2.5) -> double
```

Ở đây kiểu trả về được suy ra từ `a + b`.

## 3.3. Class Template

```cpp
#include <iostream>
#include <string>
#include <utility>

template <typename T>
class Box {
public:
    explicit Box(T value) : value_(std::move(value)) {}

    const T& get() const { return value_; }
    void set(T value) { value_ = std::move(value); }

private:
    T value_;
};

int main() {
    Box<int> a(10);
    Box<std::string> b("hello");

    a.set(20);
    std::cout << a.get() << ' ' << b.get() << '\n';
}
```

- `Box<int>` và `Box<std::string>` là hai specialization của `Box`.
- Member Functions có thể được định nghĩa trong hoặc ngoài class.
- `std::vector<T>`, `std::map<K,V>`, `std::unique_ptr<T>` đều là class template của thư viện chuẩn.

## 3.4. Non-type Template Parameters

Template không chỉ nhận **kiểu**, mà còn có thể nhận **giá trị compile-time**.

```cpp
#include <array>
#include <cstddef>
#include <iostream>

template <typename T, std::size_t N>
void printArray(const std::array<T, N>& values) {
    for (const T& v : values) std::cout << v << ' ';
    std::cout << '\n';
}

int main() {
    std::array<int, 3> a{1, 2, 3};
    printArray(a); // Compiler suy ra T=int, N=3
}
```

**Ứng dụng:** kích thước mảng, số bit, tham số thuật toán được biết lúc biên dịch.

**Giới hạn:** loại giá trị được phép làm non-type template parameter phụ thuộc chuẩn C++ và phải thỏa các quy tắc kiểu/giá trị hằng thích hợp.

## 3.5. Template Specialization

### Full specialization của class template

Có thể định nghĩa cách xử lý khác cho một kiểu cụ thể.

```cpp
#include <iostream>

template <typename T>
struct Printer {
    static void print(const T& value) {
        std::cout << value << '\n';
    }
};

template <>
struct Printer<bool> {
    static void print(bool value) {
        std::cout << (value ? "true" : "false") << '\n';
    }
};

int main() {
    Printer<int>::print(5);
    Printer<bool>::print(true);
}
```

### Partial specialization của class template

```cpp
#include <iostream>

template <typename T>
struct TypeName {
    static constexpr const char* value = "value type";
};

template <typename T>
struct TypeName<T*> {
    static constexpr const char* value = "pointer type";
};

int main() {
    std::cout << TypeName<int>::value << '\n';
    std::cout << TypeName<int*>::value << '\n';
}
```

**Quan trọng:** function template không hỗ trợ **partial specialization** như class template; thường dùng overload hoặc cấu trúc trợ giúp khác.

## 3.6. Variadic Templates và Parameter Packs

Variadic template cho phép nhận số lượng tham số template thay đổi.

```cpp
#include <iostream>

template <typename... Args>
void printAll(const Args&... args) {
    ((std::cout << args << ' '), ...);  // fold expression (C++17)
    std::cout << '\n';
}

int main() {
    printAll(1, "hello", 3.5);
    printAll(); // in dòng trống
}
```

**Cách hiểu:**

- `typename... Args`: gói các kiểu.
- `Args... args`: gói các tham số.
- `args...`: pack expansion; cách mở rộng phụ thuộc ngữ cảnh.
- Fold expression `((std::cout << args << ' '), ...)`: áp dụng thao tác cho từng phần tử.

### Fold expression tính tổng

```cpp
template <typename... Args>
auto sum(Args... values) {
    return (0 + ... + values);
}

// sum(1, 2, 3) -> 6
// sum() -> 0
```

**Lưu ý:** giá trị khởi tạo `0` ảnh hưởng kiểu kết quả và phép toán. Với thư viện tổng quát cần thiết kế quy tắc kiểu cẩn thận hơn.

## 3.7. Value Categories và Move Semantics

**Move semantics** giúp chuyển tài nguyên từ đối tượng này sang đối tượng khác (khi kiểu hỗ trợ), thay vì sao chép dữ liệu nặng.

```cpp
#include <iostream>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::string text = "long text";
    std::vector<std::string> values;

    values.push_back(text);             // sao chép từ lvalue
    values.push_back(std::move(text));  // cho phép dùng move constructor

    std::cout << values.size() << '\n';
    // text vẫn hợp lệ, nhưng nội dung sau move không nên giả định.
}
```

### `std::move` thực sự làm gì?

`std::move(x)` **không tự chuyển dữ liệu**; nó tạo biểu thức xvalue (cast) giúp chọn move constructor/move assignment nếu có và nếu phù hợp.

- Đối tượng sau khi move vẫn phải có trạng thái **hợp lệ**, nhưng thường **không được bảo đảm giá trị cụ thể**.
- Không áp dụng `std::move` lên dữ liệu còn cần dùng giá trị cũ nếu chưa thiết kế rõ.
- Không `std::move` bừa bãi với biến `const`: việc move thực sự có thể không diễn ra.
- Khi trả biến cục bộ theo giá trị, `return value;` thường cho phép NRVO/move hợp lý; không thêm `std::move` máy móc.

## 3.8. Move Constructor và Move Assignment

Ví dụ cấu trúc có thể move nhờ member là `unique_ptr`:

```cpp
#include <memory>
#include <utility>

struct Resource {
    std::unique_ptr<int> value;

    explicit Resource(int n)
        : value(std::make_unique<int>(n)) {}

    Resource(Resource&&) noexcept = default;
    Resource& operator=(Resource&&) noexcept = default;

    Resource(const Resource&) = delete;
    Resource& operator=(const Resource&) = delete;
};

int main() {
    Resource a(10);
    Resource b(std::move(a));
}
```

Đây là kiểu **move-only**: không copy, nhưng có thể chuyển ownership. Trường hợp thường nên để compiler sinh các hàm mặc định hợp lý theo **Rule of Zero**.

## 3.9. Perfect Forwarding và `std::forward`

**Forwarding reference** là trường hợp đặc biệt như `T&&` khi `T` được suy luận trong function template (hoặc `auto&&` ở một số ngữ cảnh). Nó có thể nhận cả lvalue lẫn rvalue.

```cpp
#include <iostream>
#include <utility>

void consume(const int&) {
    std::cout << "lvalue\n";
}

void consume(int&&) {
    std::cout << "rvalue\n";
}

template <typename T>
void relay(T&& value) {
    consume(std::forward<T>(value));
}

int main() {
    int x = 5;
    relay(x); // lvalue
    relay(5); // rvalue
}
```

**Vì sao cần `std::forward<T>`?**

Bên trong hàm, **tên biến `value` là một lvalue expression**, dù kiểu khai báo của nó có thể là `T&&`. Dùng `std::forward<T>(value)` để giữ đặc tính lvalue/rvalue ban đầu.

**Phân biệt:**

| Biểu thức | Mục đích |
|---|---|
| `std::move(x)` | Chuyển biểu thức thành xvalue; thể hiện sẵn sàng cho move |
| `std::forward<T>(x)` | Giữ lại value category được suy ra trong generic code |

Không dùng `std::forward` với kiểu suy luận tùy tiện ngoài ngữ cảnh thiết kế forwarding.

## 3.10. Type Traits

**Type Traits** cung cấp thông tin về kiểu tại compile-time, trong `<type_traits>`.

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
void describe(const T&) {
    if constexpr (std::is_integral_v<T>) {
        std::cout << "Integral\n";
    } else if constexpr (std::is_floating_point_v<T>) {
        std::cout << "Floating point\n";
    } else {
        std::cout << "Other\n";
    }
}

int main() {
    describe(10);
    describe(3.14);
    describe("text");
}
```

Một số traits thường dùng:

| Trait | Công dụng |
|---|---|
| `std::is_integral_v<T>` | Kiểm tra kiểu số nguyên |
| `std::is_floating_point_v<T>` | Kiểm tra kiểu dấu phẩy động |
| `std::is_pointer_v<T>` | Kiểm tra kiểu con trỏ |
| `std::is_same_v<A,B>` | Kiểm tra hai kiểu giống nhau |
| `std::remove_reference_t<T>` | Bỏ dấu tham chiếu trong kiểu |
| `std::decay_t<T>` | Chuẩn hóa kiểu tương tự biến đổi khi truyền giá trị |
| `std::is_copy_constructible_v<T>` | Có thể copy-construct? |
| `std::is_move_constructible_v<T>` | Có thể move-construct? |

**`if constexpr`** (C++17) loại bỏ nhánh không được chọn trong quá trình instantiate, giúp viết generic code theo đặc điểm kiểu.

## 3.11. Concepts (C++20)

**Concepts** mô tả ràng buộc compile-time đối với template, giúp giao diện rõ nghĩa và thông báo lỗi tốt hơn.

```cpp
#include <concepts>
#include <iostream>

template <std::integral T>
T twice(T value) {
    return value + value;
}

int main() {
    std::cout << twice(6) << '\n'; // 12
    // twice(1.5); // lỗi biên dịch: double không thỏa integral
}
```

### Concept tùy chỉnh

```cpp
#include <concepts>

template <typename T>
concept Addable = requires(T a, T b) {
    { a + b } -> std::same_as<T>;
};

template <Addable T>
T addTwo(const T& a, const T& b) {
    return a + b;
}
```

Các khái niệm:

- `requires` expression.
- `requires` clause.
- Standard concepts: `std::integral`, `std::floating_point`, `std::same_as`, `std::derived_from`.
- Generic API có ràng buộc rõ ràng.

**Lưu ý:** `Addable` ở đây chỉ chấp nhận trường hợp `a+b` trả **đúng `T`**; đây là thiết kế có chủ đích, không đại diện cho mọi phép cộng hợp lệ.

## 3.12. Template Deduction và lỗi thường gặp

### Không suy ra được kiểu

```cpp
template <typename T>
T sameType(T a, T b) {
    return a + b;
}

// sameType(1, 2.5); // lỗi suy luận T khác nhau
auto x = sameType<double>(1, 2.5); // OK
```

### Định nghĩa template đặt ở đâu?

Compiler cần thấy định nghĩa template khi instantiate specialization. Thông thường đặt **định nghĩa trong header** (`.hpp`), hoặc file được include từ header, trừ khi dùng kỹ thuật **explicit instantiation** có kiểm soát.

### Template không phải runtime polymorphism

- Template thường làm việc qua **compile-time instantiation**.
- `virtual` thực hiện **runtime dispatch**.
- Hai kỹ thuật giải quyết các nhu cầu khác nhau và có thể phối hợp.

### Code bloat

Template nhiều specialization có thể tăng mã máy. Theo dõi kích thước binary, đo hiệu năng và chọn abstraction vừa đủ.

## 3.13. Ví dụ tổng hợp: Template + STL + Iterator + Lambda

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

template <typename Container, typename Predicate>
std::size_t countMatching(const Container& items, Predicate predicate) {
    return static_cast<std::size_t>(
        std::count_if(items.begin(), items.end(), predicate));
}

int main() {
    std::vector<int> values{1, 2, 3, 4, 5, 6};

    auto even = [](int x) { return x % 2 == 0; };

    std::cout << "Even count = "
              << countMatching(values, even) << '\n'; // 3
}
```

**Phân tích:**

1. `Container` được suy ra thành `std::vector<int>`.
2. `Predicate` được suy ra từ kiểu closure của lambda.
3. `count_if` dùng iterator để duyệt container.
4. Hàm không phụ thuộc trực tiếp vào một kiểu phần tử cụ thể; điều kiện là container có `begin/end` và predicate gọi được với các phần tử.

---

# BÀI TẬP THỰC HÀNH PHẦN I

## Mức 1 — Pointer và Reference

**Bài 1.** Khai báo biến `int`, dùng pointer đọc và sửa giá trị; kiểm tra `nullptr` trước khi dereference.

**Bài 2.** Viết ba hàm nhận tham số bằng value, pointer và reference. Quan sát trường hợp nào thay đổi được biến gốc.

**Bài 3.** Viết một hàm nhận `const int*` và một hàm nhận `int* const`; thử các phép gán để thấy khác biệt compile-time.

**Bài 4.** Viết phép cộng/trừ qua con trỏ hàm và một hàm `calculate` nhận callback là function pointer.

**Bài 5.** Tạo đối tượng bằng `std::make_unique`, chuyển ownership bằng `std::move`, kiểm tra owner nguồn còn rỗng.

**Bài 6.** Dùng `std::make_shared` và `std::weak_ptr` để quan sát vòng đời đối tượng khi shared owner cuối cùng bị hủy.

## Mức 2 — STL và Iterator

**Bài 7.** Với `std::vector<int>`, thực hiện `push_back`, `insert`, `erase`, `reserve` và `resize`; in `size/capacity` sau mỗi thao tác.

**Bài 8.** Lưu danh sách tên trùng lặp bằng `std::vector<std::string>`, chuyển sang `std::set<std::string>` để lọc trùng và sắp xếp.

**Bài 9.** Đếm tần suất từng từ bằng `std::unordered_map<std::string, int>`.

**Bài 10.** Viết chương trình `queue` xử lý FIFO, `stack` hoàn tác LIFO, `priority_queue` xử lý số ưu tiên cao trước.

**Bài 11.** Lọc số chẵn khỏi `vector` theo hai cách: vòng lặp `erase` an toàn và Erase–Remove Idiom.

**Bài 12.** Dùng `find_if` tìm phần tử đầu tiên lớn hơn ngưỡng; `transform` tính bình phương; `count_if` đếm số thỏa điều kiện.

**Bài 13.** Cố ý lưu iterator `vector`, sau đó gây reallocation; **không dereference iterator cũ**. Giải thích vì sao nó không còn đáng tin.

## Mức 3 — Template và Generic Programming

**Bài 14.** Viết function template `maximum(T,T)` cho `int`, `double`, `std::string`.

**Bài 15.** Viết class template `Box<T>` có `set()`, `get()`.

**Bài 16.** Viết function template nhận `std::array<T,N>`, tính tổng và trung bình (xử lý `N==0` phù hợp).

**Bài 17.** Viết variadic template `printAll` với fold expression C++17.

**Bài 18.** Viết hai overload `consume(const T&)`/`consume(T&&)`, bọc bởi template dùng `std::forward`; thử cả lvalue/rvalue.

**Bài 19.** Dùng `if constexpr` và type traits để xử lý khác nhau cho `integral`, `floating_point` và kiểu khác.

**Bài 20.** Viết một concept C++20 giới hạn template chỉ nhận kiểu số nguyên; thử gọi với kiểu hợp lệ và không hợp lệ.

### Tiêu chí tự đánh giá

- Giải thích được **vì sao** chọn pointer/reference/smart pointer thay vì chỉ nhớ cú pháp.
- Không để lộ lỗi vòng đời và ownership.
- Chọn container dựa trên yêu cầu và độ phức tạp.
- Tránh iterator invalidation.
- Đọc hiểu khai báo `template <typename... Args>`, `T&&` và `std::forward<T>`.
- Biết ví dụ nào cần C++17 và ví dụ nào cần C++20.

---

# BẢNG TRA CỨU NHANH

## Các header phổ biến

| Header | Chủ đề |
|---|---|
| `<memory>` | Smart pointer, allocator |
| `<utility>` | `std::move`, `std::forward`, `std::pair` |
| `<type_traits>` | Type traits |
| `<concepts>` | Concepts (C++20) |
| `<vector>` | Vector |
| `<array>` | Array |
| `<list>` | List |
| `<deque>` | Deque |
| `<set>` | Set |
| `<unordered_set>` | Unordered set |
| `<map>` | Map |
| `<unordered_map>` | Unordered map |
| `<queue>` | Queue, priority queue |
| `<stack>` | Stack |
| `<algorithm>` | Các thuật toán STL |
| `<iterator>` | Iterator helpers, inserter |
| `<functional>` | `std::function`, comparator |

## Những lỗi nên tránh

| Lỗi | Cách phòng tránh |
|---|---|
| Dereference `nullptr` | Kiểm tra con trỏ khi có thể null |
| Trả reference/pointer tới biến cục bộ | Trả giá trị, hoặc đảm bảo vòng đời |
| `new` nhưng quên `delete` | Ưu tiên RAII / smart pointer |
| Dùng sai `delete`/`delete[]` | Khớp đúng cách cấp phát |
| Hai `shared_ptr` riêng quản lý cùng raw pointer | Tạo owner một lần, sao chép `shared_ptr` |
| `shared_ptr` vòng kín | Dùng `weak_ptr` cho liên kết không sở hữu |
| Dùng `v[5]` sau `reserve(10)` khi size nhỏ | Kiểm tra `size`; dùng `resize` nếu cần phần tử |
| Dùng iterator sau reallocation/erase | Tuân thủ quy tắc invalidation |
| `copy` vào container đích rỗng bằng `begin()` | `resize` trước hoặc `back_inserter` |
| Tưởng `remove_if` đã xóa phần tử | Dùng `erase` hoặc `std::erase_if` |
| Gọi `sort` với `list` | Dùng `list::sort()` |
| Dùng `std::move` rồi giả định giá trị cũ | Chỉ dựa vào trạng thái hợp lệ, không giả định giá trị |
| Định nghĩa template chỉ trong `.cpp` bất kỳ | Đặt định nghĩa nơi cần instantiate hoặc explicit instantiation |

---

## Liên kết học tiếp

- [Lộ trình tổng hợp C++ và Embedded Linux](../ROADMAP_CPP_EMBEDDED_LINUX.md)
- [Lý thuyết riêng về `std::vector` và `std::set`](README_VECTOR_SET.md)
- [Header Guards trong C++](README0.md)

**Phạm vi tài liệu:** Phần I — Nền tảng ngôn ngữ C++, gồm đầy đủ **1. Pointer & Reference**, **2. STL Containers/Iterator/Algorithms**, **3. Template & Generic Programming**. Các chủ đề OOP, multithreading và Linux system programming thuộc những phần tiếp theo của lộ trình.
