# Lý thuyết C++ STL: `vector` và `set`

> Tài liệu về hai container thường dùng trong Standard Template Library (STL), kèm ví dụ C++ và cách lựa chọn trong ứng dụng thực tế.

## 1. Container và STL là gì?

**Container** là kiểu dữ liệu quản lý một tập hợp phần tử. Trong thư viện chuẩn C++, `std::vector` thuộc nhóm *sequence containers* (container tuần tự), còn `std::set` thuộc nhóm *ordered associative containers* (container kết hợp có thứ tự).

| Container | Header | Đặc điểm cốt lõi |
| --- | --- | --- |
| `std::vector<T>` | `<vector>` | Mảng động, phần tử liên tiếp trong bộ nhớ |
| `std::set<T>` | `<set>` | Giá trị duy nhất theo bộ so sánh, luôn có thứ tự |

Khai báo mẫu:

```cpp
#include <vector>
#include <set>
#include <string>

std::vector<int> numbers;
std::set<std::string> acmDevicesList;
```

Trong cả hai trường hợp, `<T>` xác định kiểu dữ liệu của phần tử.

---

## 2. `std::vector` — Mảng động

### 2.1. Khái niệm

`std::vector` lưu các phần tử **liên tiếp trong bộ nhớ**, tương tự mảng thông thường, nhưng có thể tự thay đổi dung lượng khi thêm phần tử.

**Đặc điểm:**

- Cho phép phần tử trùng lặp.
- Giữ thứ tự do chương trình tạo ra; không tự động sắp xếp.
- Truy cập phần tử bằng chỉ số: `v[0]`, `v[1]`...
- Thêm phần tử ở cuối thường nhanh.
- Khi cần tăng dung lượng, vector có thể cấp phát vùng nhớ khác và di chuyển/sao chép phần tử.

### 2.2. Khai báo và khởi tạo

```cpp
std::vector<int> a;               // Vector rỗng
std::vector<int> b = {10, 20, 30};
std::vector<int> c(5);            // 5 phần tử int, giá trị 0
std::vector<int> d(5, 100);       // 5 phần tử, mỗi phần tử = 100
std::vector<std::string> names = {"An", "Binh"};
```

Với `std::vector<int> b = {10, 20, 30};`:

```text
Chỉ số:       0     1     2
Giá trị:    [10]  [20]  [30]
Bộ nhớ:      các phần tử được lưu liên tiếp
```

### 2.3. `size` và `capacity`

- `size()`: số phần tử đang tồn tại.
- `capacity()`: dung lượng hiện được cấp phát, tính theo số phần tử.
- `reserve(n)`: yêu cầu dung lượng tối thiểu n phần tử mà không thêm phần tử.
- `resize(n)`: thay đổi số phần tử đang tồn tại, thêm hoặc xóa phần tử nếu cần.

```cpp
std::vector<int> v = {10, 20, 30};

std::cout << v.size() << '\n';     // 3
std::cout << v.capacity() << '\n'; // Ít nhất là 3 (không cố định)
v.reserve(100);                    // size vẫn là 3
v.resize(5);                       // size = 5, hai số mới là 0
```

Không nên cho rằng mỗi lần `push_back()` thì `capacity()` đều tăng, hoặc dung lượng luôn nhân đôi: chiến lược tăng dung lượng phụ thuộc thư viện triển khai.

### 2.4. Các thao tác thường dùng

| Thao tác | Ví dụ | Ý nghĩa |
| --- | --- | --- |
| Thêm cuối | `v.push_back(40)` | Thêm một phần tử cuối |
| Thêm tại vị trí | `v.insert(v.begin(), 5)` | Chèn vào đầu |
| Xóa cuối | `v.pop_back()` | Xóa phần tử cuối (vector không được rỗng) |
| Xóa tại vị trí | `v.erase(v.begin())` | Xóa phần tử đầu |
| Truy cập | `v[1]` | Không kiểm tra vượt phạm vi |
| Truy cập an toàn hơn | `v.at(1)` | Ném `std::out_of_range` khi vượt phạm vi |
| Đầu/cuối | `v.front()` / `v.back()` | Lấy phần tử đầu/cuối (vector không rỗng) |
| Xóa hết | `v.clear()` | Xóa tất cả phần tử, không bắt buộc giải phóng capacity |
| Kiểm tra rỗng | `v.empty()` | Trả về `true` nếu rỗng |

Ví dụ:

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v = {10, 20, 30};
    v.push_back(40);                // 10 20 30 40
    v.insert(v.begin() + 1, 15);    // 10 15 20 30 40
    v.erase(v.begin() + 2);        // 10 15 30 40
    v[0] = 100;                     // 100 15 30 40

    for (int x : v) {
        std::cout << x << " ";
    }
    std::cout << '\n';
}
```

**Kết quả:** `100 15 30 40`

### 2.5. Truy cập, tìm kiếm và sắp xếp

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v = {30, 10, 20, 10};

int first = v[0];  // O(1), nhận giá trị 30
auto it = std::find(v.begin(), v.end(), 20); // O(n)
std::sort(v.begin(), v.end());               // O(n log n)
// Sau sort: 10 10 20 30
```

`std::find` tìm tuyến tính trong một vector chưa được tổ chức để tra cứu. Nếu vector đã được sắp xếp, có thể dùng `std::binary_search` hoặc `std::lower_bound` để tìm nhanh hơn theo giá trị (O(log n) phép so sánh).

### 2.6. Độ phức tạp và iterator

| Thao tác | Độ phức tạp |
| --- | --- |
| Truy cập `v[i]` | O(1) |
| `push_back` | O(1) trung bình phân bổ (*amortized*) |
| Chèn hoặc xóa ở đầu/giữa | O(n) |
| Tìm bằng `std::find` | O(n) |
| Duyệt hết | O(n) |

**Iterator của vector** là *random-access iterator*: hỗ trợ `it + n`, `it += n` và phép trừ hai iterator cùng container.

**Lưu ý về mất hiệu lực iterator:** Khi vector tái cấp phát bộ nhớ, mọi iterator, con trỏ và tham chiếu tới phần tử cũ bị mất hiệu lực. Chèn/xóa cũng có thể làm mất hiệu lực iterator tại và sau vị trí bị tác động, ngay cả khi không tái cấp phát.

---

## 3. `std::set` — Tập hợp có thứ tự và không trùng lặp

### 3.1. Khái niệm

`std::set` lưu mỗi giá trị **một lần theo quan hệ tương đương của bộ so sánh** và duy trì thứ tự đã định. Mặc định sử dụng `std::less<T>` (thường tương ứng với thứ tự tăng dần).

**Đặc điểm:**

- Không chứa hai giá trị tương đương theo bộ so sánh.
- Tự duy trì thứ tự sau khi thêm/xóa.
- Không có `operator[]` và không truy cập bằng chỉ số.
- Tìm, thêm, xóa theo khóa trong O(log n).
- Phần tử không được sửa trực tiếp trong set vì có thể phá vỡ thứ tự.

`std::set` thường được triển khai bằng **cây đỏ-đen (red-black tree)**, một loại cây tìm kiếm nhị phân tự cân bằng. Chuẩn C++ không bắt buộc mọi thư viện phải dùng đúng cấu trúc cây này.

### 3.2. Khai báo và khởi tạo

```cpp
#include <set>
#include <string>
#include <functional>

std::set<int> a;
std::set<int> b = {30, 10, 20, 10};
std::set<std::string> names = {"Nam", "An", "Binh"};
std::set<int, std::greater<int>> descending = {10, 30, 20};
```

Duyệt `b` thu được: `10 20 30`. Phần tử `10` được đưa vào hai lần nhưng chỉ tồn tại một lần.

Duyệt `descending` thu được: `30 20 10`.

### 3.3. Các thao tác thường dùng

| Thao tác | Ví dụ | Ý nghĩa |
| --- | --- | --- |
| Thêm | `s.insert(40)` | Thêm nếu chưa tồn tại |
| Xóa theo giá trị | `s.erase(10)` | Xóa khóa tương đương với 10 |
| Tìm kiếm | `s.find(20)` | Iterator hoặc `s.end()` |
| Kiểm tra tồn tại | `s.contains(20)` | Trả về bool, từ C++20 |
| Đếm | `s.count(20)` | Với set là 0 hoặc 1 |
| Cận dưới | `s.lower_bound(25)` | Phần tử đầu tiên không nhỏ hơn 25 |
| Cận trên | `s.upper_bound(20)` | Phần tử đầu tiên lớn hơn 20 |
| Số phần tử | `s.size()` | Số phần tử đang lưu |
| Xóa hết | `s.clear()` | Xóa toàn bộ phần tử |

Ví dụ:

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s = {30, 10, 20};

    auto [it, inserted] = s.insert(20); // inserted = false
    s.insert(40);

    if (s.find(30) != s.end()) {
        std::cout << "Tim thay 30\n";
    }

    s.erase(10);

    for (const auto& x : s) {
        std::cout << x << " ";
    }
    std::cout << '\n'; // 20 30 40
}
```

### 3.4. Iterator và giới hạn

`set` có *bidirectional iterator*: có thể `++it` và `--it` (nếu thao tác hợp lệ), nhưng **không** có `it + 2` hay `s[0]`.

```cpp
std::set<int> s = {10, 20, 30};
auto it = s.begin();
++it;                   // Đúng: *it == 20
// it += 2;             // Sai: không hỗ trợ random access
// s[0];                // Sai: set không có operator[]

#include <iterator>     // Nếu dùng std::advance
std::advance(it, 1);    // Dịch iterator thêm một bước, O(số bước)
```

Khi chèn thêm phần tử vào set, các iterator và tham chiếu tới phần tử đang tồn tại vẫn hợp lệ. Khi xóa, iterator/tham chiếu trỏ tới phần tử bị xóa sẽ mất hiệu lực.

Không thể sửa khóa trong set bằng `*it = value`. Cách thông thường là xóa giá trị cũ rồi chèn giá trị mới (hoặc dùng `extract` từ C++17 trong trường hợp phù hợp).

### 3.5. Độ phức tạp

| Thao tác | Độ phức tạp |
| --- | --- |
| `insert(value)` | O(log n) |
| `find(value)` | O(log n) |
| `erase(value)` | O(log n) |
| `lower_bound(value)` | O(log n) |
| Duyệt hết | O(n) |

---

## 4. So sánh `vector` và `set`

| Tiêu chí | `std::vector` | `std::set` |
| --- | --- | --- |
| Kiểu container | Tuần tự | Kết hợp có thứ tự |
| Tổ chức dữ liệu | Mảng động liên tiếp | Thường là cây tự cân bằng |
| Cho phép trùng lặp | Có | Không (theo bộ so sánh) |
| Thứ tự phần tử | Do thao tác chương trình quyết định | Tự sắp theo bộ so sánh |
| Truy cập bằng chỉ số | Có, O(1) | Không |
| Tìm theo giá trị | O(n) với `find` tuyến tính | O(log n) |
| Thêm phần tử | Cuối: O(1) trung bình phân bổ | Theo khóa: O(log n) |
| Xóa ở giữa | Thường O(n) | Theo khóa: O(log n) |
| Bộ nhớ | Liên tiếp, thường ít overhead trên mỗi phần tử | Các nút thường có thêm con trỏ và metadata |
| Iterator | Random-access | Bidirectional |

> Độ phức tạp không phải yếu tố duy nhất. Với dữ liệu nhỏ hoặc duyệt tuần tự nhiều, vector có thể nhanh hơn nhờ tính liên tiếp trong bộ nhớ và tận dụng cache CPU.

---

## 5. Ví dụ: Danh sách cổng Serial trên Linux

Giả sử chương trình đọc được các tên cổng:

```text
/dev/ttyACM1
/dev/ttyACM0
/dev/ttyACM1
```

### Dùng vector: giữ nguyên thứ tự và phần tử trùng

```cpp
#include <vector>
#include <string>

std::vector<std::string> devices = {
    "/dev/ttyACM1", "/dev/ttyACM0", "/dev/ttyACM1"
};
```

Duyệt vector thu được đúng ba phần tử theo thứ tự được thêm vào.

### Dùng set: khử trùng và sắp xếp

```cpp
#include <iostream>
#include <set>
#include <string>

int main() {
    std::set<std::string> acmDevicesList;

    acmDevicesList.insert("/dev/ttyACM1");
    acmDevicesList.insert("/dev/ttyACM0");
    acmDevicesList.insert("/dev/ttyACM1"); // Trùng: không thêm

    for (const auto& device : acmDevicesList) {
        std::cout << device << '\n';
    }
}
```

Kết quả:

```text
/dev/ttyACM0
/dev/ttyACM1
```

**Giải thích khai báo** `std::set<std::string> acmDevicesList;`:

- `std::set`: container chứa tập hợp có thứ tự.
- `std::string`: mỗi phần tử là chuỗi ký tự.
- `acmDevicesList`: tên biến do lập trình viên đặt.
- `ACM` thường viết tắt của *Abstract Control Model*, thuộc USB CDC-ACM; Linux thường biểu diễn thiết bị bằng `/dev/ttyACM0`, `/dev/ttyACM1`...
- Một số bộ chuyển đổi USB-to-UART như FT232, CP210x, CH340 thường xuất hiện dưới dạng `/dev/ttyUSB0`, tùy driver.

---

## 6. Khi nào chọn container nào?

**Nên dùng vector nếu:**

1. Cần thứ tự chèn, thứ tự duyệt hoặc truy cập theo chỉ số.
2. Cần duyệt dữ liệu liên tiếp, hiệu quả bộ nhớ và cache.
3. Chấp nhận dữ liệu trùng lặp.
4. Chủ yếu thêm ở cuối và ít chèn/xóa giữa dãy.

**Nên dùng set nếu:**

1. Muốn bảo đảm không có giá trị tương đương bị lặp.
2. Cần dữ liệu luôn có thứ tự.
3. Cần chèn/xóa/tìm theo khóa thường xuyên.
4. Không cần truy cập theo chỉ số.

**Các lựa chọn liên quan:**

- `std::unordered_set`: tập không trùng, dùng bảng băm, tìm/chèn/xóa trung bình O(1), nhưng không giữ thứ tự.
- `std::multiset`: tập có thứ tự nhưng cho phép khóa trùng.
- `std::sort` + `std::unique` + `erase`: cách loại trùng từ vector khi phù hợp với bài toán và muốn tiếp tục dùng mảng liên tiếp.

---

## 7. Bài tập thực hành

1. Tạo vector `{5, 2, 5, 1}`; in từng phần tử và chỉ số.
2. Tạo set từ cùng các giá trị; so sánh kết quả khi duyệt.
3. Thêm 100 phần tử vào vector và in `size()`, `capacity()` sau mỗi lần thêm.
4. Sử dụng `find()` của set để kiểm tra một tên thiết bị có tồn tại hay không.
5. Tự viết chương trình chuyển vector thành set và ngược lại.

**Ghi nhớ:** `vector` là **mảng động, giữ thứ tự, cho phép trùng**; `set` là **tập hợp có thứ tự, không trùng theo bộ so sánh**.
