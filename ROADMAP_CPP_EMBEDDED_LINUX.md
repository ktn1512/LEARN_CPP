# TỔNG HỢP KIẾN THỨC LẬP TRÌNH C++ VÀ EMBEDDED LINUX

> Đề cương kiến thức tổng quát từ cơ bản đến nâng cao, phục vụ học C++ hiện đại, lập trình hệ thống Linux, lập trình nhúng, thiết kế kiến trúc và đọc hiểu codebase lớn. Nội dung độc lập với mọi dự án cụ thể.

## Mục lục

1. [Nền tảng ngôn ngữ C++](#phần-i-nền-tảng-ngôn-ngữ-c)
2. [Lập trình hướng đối tượng (OOP)](#phần-ii-lập-trình-hướng-đối-tượng-oop)
3. [Design Patterns](#phần-iii-design-patterns)
4. [Function, Callback và Lambda](#phần-iv-function-callback-và-lambda)
5. [Multithreading và Concurrency](#phần-v-multithreading-và-concurrency)
6. [Asynchronous Programming](#phần-vi-asynchronous-programming)
7. [Linux System Programming](#phần-vii-linux-system-programming)
8. [Inter-Process Communication](#phần-viii-inter-process-communication-ipc)
9. [Event-driven Programming](#phần-ix-event-driven-programming)
10. [Finite State Machine](#phần-x-finite-state-machine)
11. [Network Programming](#phần-xi-network-programming)
12. [Database và Serialization](#phần-xii-database-và-data-serialization)
13. [Preprocessor và Build System](#phần-xiii-c-preprocessor-và-build-system)
14. [Software Architecture](#phần-xiv-software-architecture)
15. [Debugging và Code Analysis](#phần-xv-debugging-và-phân-tích-code)
16. [Kiến thức bổ sung](#phần-xvi-các-kiến-thức-bổ-sung)
17. [Lộ trình học tổng hợp](#phần-xvii-lộ-trình-học-tổng-hợp)

---

# PHẦN I. NỀN TẢNG NGÔN NGỮ C++

## 1. Con trỏ và tham chiếu (Pointer & Reference)

### 1.1. Pointer

Con trỏ là biến lưu địa chỉ bộ nhớ của một đối tượng.

Các kiến thức cần nắm:

- Khai báo và sử dụng con trỏ.
- Toán tử lấy địa chỉ `&` và giải tham chiếu `*`.
- Con trỏ `nullptr`.
- Con trỏ tới đối tượng và `struct`.
- Con trỏ tới hàm (Function Pointer).
- Con trỏ cấp hai.
- Con trỏ `void*`.
- Con trỏ và dữ liệu `const`.
- Cấp phát bộ nhớ động bằng `new`, `delete`.
- Lỗi con trỏ treo (Dangling Pointer).
- Rò rỉ bộ nhớ (Memory Leak).

### 1.2. Reference

Tham chiếu là một tên thay thế cho đối tượng đã tồn tại.

- Lvalue Reference (`T&`).
- Rvalue Reference (`T&&`).
- Truyền tham số bằng tham chiếu.
- Tham chiếu hằng (`const T&`).
- Khác biệt giữa Pointer và Reference.
- Vòng đời của đối tượng được tham chiếu.

### 1.3. Smart Pointer

| Loại | Chức năng |
|---|---|
| `std::unique_ptr` | Sở hữu độc quyền đối tượng |
| `std::shared_ptr` | Cho phép nhiều nơi cùng sở hữu đối tượng |
| `std::weak_ptr` | Tham chiếu không sở hữu đối tượng do `shared_ptr` quản lý |
| Raw Pointer | Con trỏ thông thường, không tự quản lý bộ nhớ |

Cần hiểu quyền sở hữu (ownership), vòng đời đối tượng, reference counting, nguy cơ circular reference và RAII.

## 2. Standard Template Library (STL)

STL cung cấp những container, iterator và thuật toán có thể tái sử dụng.

### 2.1. Containers

| Container | Đặc điểm |
|---|---|
| `std::vector` | Mảng động, phần tử nằm liên tiếp trong bộ nhớ |
| `std::array` | Mảng có kích thước cố định |
| `std::list` | Danh sách liên kết đôi |
| `std::deque` | Hàng đợi hai đầu |
| `std::set` | Tập hợp phần tử duy nhất, được sắp xếp |
| `std::unordered_set` | Tập hợp phần tử duy nhất dùng bảng băm |
| `std::map` | Cặp key-value sắp xếp theo key |
| `std::unordered_map` | Key-value dùng bảng băm |
| `std::queue` | Hàng đợi FIFO |
| `std::stack` | Ngăn xếp LIFO |
| `std::priority_queue` | Hàng đợi ưu tiên |

### 2.2. Iterator

- `begin()`, `end()`; `cbegin()`, `cend()`.
- `rbegin()`, `rend()`.
- Iterator Invalidation.
- Iterator Arithmetic.
- Range-based for loop.
- So sánh Iterator và Pointer.

### 2.3. STL Algorithms

| Thuật toán | Chức năng |
|---|---|
| `std::find()` | Tìm phần tử |
| `std::find_if()` | Tìm theo điều kiện |
| `std::sort()` | Sắp xếp |
| `std::for_each()` | Áp dụng thao tác cho các phần tử |
| `std::count()` | Đếm phần tử |
| `std::count_if()` | Đếm theo điều kiện |
| `std::transform()` | Biến đổi dãy |
| `std::remove_if()` | Dịch chuyển các phần tử cần giữ, thường kết hợp `erase()` để xóa |
| `std::copy()` | Sao chép phần tử |
| `std::any_of()` | Kiểm tra có phần tử thỏa điều kiện |

Cần tìm hiểu thêm độ phức tạp thời gian, bộ nhớ và điều kiện hợp lệ của mỗi thao tác.

## 3. Template và Generic Programming

Template cho phép viết mã làm việc với nhiều kiểu dữ liệu.

- Function Template, Class Template.
- Template Type Parameters.
- Non-type Template Parameters.
- Template Specialization.
- Variadic Templates, Parameter Packs.
- Perfect Forwarding: `std::forward`.
- Move Semantics: `std::move`.
- Type Traits.
- Concepts (C++20).

Cú pháp nên nắm:

```cpp
template <typename T>

template <typename... Args>

Args&&... args

std::forward<Args>(args)...
```

---

# PHẦN II. LẬP TRÌNH HƯỚNG ĐỐI TƯỢNG (OOP)

## 4. Object-Oriented Programming

OOP tổ chức chương trình thành các đối tượng có dữ liệu và hành vi.

### 4.1. Class và Object

- Class và Object.
- Attributes và Methods.
- Constructor và Destructor.
- `public`, `private`, `protected`.
- Static Members, `this` Pointer.
- Object Lifetime.
- Copy Constructor, Copy Assignment Operator.
- Move Constructor, Move Assignment Operator.

### 4.2. Bốn tính chất OOP

| Tính chất | Ý nghĩa |
|---|---|
| Encapsulation | Đóng gói dữ liệu và hành vi |
| Abstraction | Che giấu chi tiết triển khai qua giao diện |
| Inheritance | Kế thừa và mở rộng hành vi |
| Polymorphism | Một giao diện, nhiều cách triển khai |

### 4.3. Inheritance

- Single Inheritance.
- Multiple Inheritance.
- Multilevel Inheritance.
- Hierarchical Inheritance.
- Virtual Inheritance.
- Constructor và Destructor trong quan hệ kế thừa.
- Name Hiding, Function Overriding.

### 4.4. Polymorphism

**Compile-time Polymorphism:** Function Overloading, Operator Overloading, Template.

**Runtime Polymorphism:** Virtual Function, Function Overriding, Abstract Class, Pure Virtual Function, Virtual Destructor, Dynamic Dispatch.

Cần phân biệt Overloading/Overriding; Static/Dynamic Binding; Abstract Class/Interface; Inheritance/Composition.

## 5. Các kỹ thuật quản lý đối tượng

- **RAII:** Sở hữu và giải phóng tài nguyên gắn với vòng đời của đối tượng.
- **Rule of Three, Five, Zero:** Các nguyên tắc thiết kế thao tác sao chép, di chuyển và hủy.
- **Object Ownership:** Xác định thành phần có trách nhiệm quản lý tài nguyên.
- **Type Casting:** `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`.
- **Const Correctness:** Dùng `const` để kiểm soát quyền thay đổi trạng thái.

---

# PHẦN III. DESIGN PATTERNS

## 6. Các mẫu thiết kế phần mềm

Design Pattern là các giải pháp thiết kế có thể tái sử dụng cho những vấn đề phổ biến.

### 6.1. Creational Patterns

| Pattern | Chức năng |
|---|---|
| Singleton | Quản lý một instance dùng chung và điểm truy cập đến nó |
| Factory Method | Cho phép lớp con quyết định loại đối tượng được tạo |
| Abstract Factory | Tạo các họ đối tượng có liên quan |
| Builder | Xây dựng đối tượng phức tạp theo từng bước |
| Prototype | Tạo đối tượng qua sao chép mẫu |

### 6.2. Structural Patterns

| Pattern | Chức năng |
|---|---|
| Adapter | Chuyển đổi interface |
| Bridge | Tách abstraction khỏi implementation |
| Composite | Tổ chức đối tượng theo cây |
| Decorator | Bổ sung hành vi mà không đổi lớp ban đầu |
| Facade | Cung cấp giao diện đơn giản cho hệ thống phức tạp |
| Proxy | Đại diện và kiểm soát truy cập đối tượng |
| Flyweight | Chia sẻ trạng thái để tiết kiệm bộ nhớ |

### 6.3. Behavioral Patterns

| Pattern | Chức năng |
|---|---|
| Observer | Thông báo thay đổi cho các đối tượng đăng ký |
| Strategy | Thay đổi thuật toán xử lý |
| Command | Đóng gói yêu cầu thành đối tượng |
| State | Thay đổi hành vi theo trạng thái |
| Template Method | Xác định khung thuật toán |
| Chain of Responsibility | Chuyển yêu cầu qua chuỗi handler |
| Mediator | Điều phối tương tác giữa các đối tượng |

Ưu tiên học: Singleton, Factory, Observer, Strategy, State, Command.

---

# PHẦN IV. FUNCTION, CALLBACK VÀ LAMBDA

## 7. Callback Programming

Callback là kỹ thuật truyền một hàm hoặc callable để được gọi khi cần.

- Function Pointer, Callback Function.
- Member Function Pointer.
- `std::function`, `std::bind`.
- Callback Registration, Invocation và Lifetime.
- Callback trong môi trường đa luồng.

**Synchronous Callback:** Chạy trực tiếp trong lời gọi hiện tại.

**Asynchronous Callback:** Chạy về sau khi có sự kiện hoặc tác vụ hoàn tất.

## 8. Lambda Expression

Lambda cho phép khai báo hàm ẩn danh tại vị trí sử dụng.

```cpp
[capture](parameters) -> return_type
{
    // Body
};
```

Nội dung:

- Capture by Value `[=]`.
- Capture by Reference `[&]`.
- Capture `[this]`.
- Mutable Lambda, Generic Lambda.
- Lambda trong STL, Thread, Callback.
- Vòng đời của biến capture; tránh dangling reference.

---

# PHẦN V. MULTITHREADING VÀ CONCURRENCY

## 9. Lập trình đa luồng

Multithreading cho phép một tiến trình có nhiều luồng thực thi.

- Process và Thread.
- Thread Lifecycle, Creation, Scheduling, Thread ID.
- Thread Synchronization, Thread-local Storage.
- Thread Pool, Thread Cancellation, Thread Safety.
- Phân biệt Concurrency và Parallelism.

Các API C++:

```cpp
std::thread
std::jthread
std::this_thread
std::thread::join()
std::thread::detach()
```

## 10. Đồng bộ đa luồng

### 10.1. Mutex

- `std::mutex`, `std::recursive_mutex`.
- `std::timed_mutex`, `std::shared_mutex`.
- `std::lock_guard`, `std::unique_lock`.
- `std::scoped_lock`, `std::shared_lock`.

### 10.2. Semaphore

- Binary Semaphore.
- Counting Semaphore.
- Semaphore dùng để báo hiệu hoặc giới hạn mức truy cập đồng thời.

### 10.3. Condition Variable

- `std::condition_variable`.
- `wait()`, `notify_one()`, `notify_all()`.
- Spurious Wakeup.
- Producer–Consumer Problem.

### 10.4. Atomic Operation

- `std::atomic`.
- `load()`, `store()`, `fetch_add()`.
- Compare-and-exchange.
- Memory Ordering.

### 10.5. Những vấn đề thường gặp

| Vấn đề | Ý nghĩa |
|---|---|
| Race Condition | Kết quả phụ thuộc vào thứ tự thực thi không được kiểm soát |
| Data Race | Truy cập bộ nhớ xung đột, thiếu đồng bộ theo mô hình bộ nhớ |
| Deadlock | Các luồng chờ nhau vô hạn |
| Livelock | Hoạt động liên tục nhưng không có tiến triển |
| Starvation | Luồng không nhận đủ cơ hội hoặc tài nguyên để chạy |
| Priority Inversion | Luồng ưu tiên cao bị chặn bởi luồng ưu tiên thấp |

---

# PHẦN VI. ASYNCHRONOUS PROGRAMMING

## 11. Lập trình bất đồng bộ

Lập trình bất đồng bộ cho phép biểu diễn công việc hoàn thành vào thời điểm khác, tránh chờ tuần tự không cần thiết.

- Synchronous / Asynchronous.
- Blocking / Non-blocking.
- Task, Future, Promise.
- Task Scheduling.
- Async Callback, Event Loop, Coroutine.

| Thành phần | Chức năng |
|---|---|
| `std::async` | Khởi chạy tác vụ theo chính sách thực thi |
| `std::future` | Nhận kết quả của tác vụ |
| `std::promise` | Cung cấp kết quả hoặc ngoại lệ |
| `std::packaged_task` | Gắn callable với trạng thái kết quả dạng future |
| `std::launch::async` | Yêu cầu thực thi bất đồng bộ |
| `std::launch::deferred` | Hoãn chạy đến khi kết quả được yêu cầu |
| Coroutine | Tạm dừng và tiếp tục thực thi hàm |

**Lưu ý:** Bất đồng bộ không đồng nghĩa với việc luôn phải tạo luồng mới.

---

# PHẦN VII. LINUX SYSTEM PROGRAMMING

## 12. Process Management

Process là một chương trình đang thực thi với tài nguyên và không gian địa chỉ riêng.

- PID, Parent Process, Child Process.
- Process Address Space, Lifecycle.
- Process Creation, Termination, Synchronization.
- Zombie Process, Orphan Process.
- Daemon Process.

API Linux/POSIX:

```cpp
fork()
exec()
wait()
waitpid()
getpid()
getppid()
kill()
exit()
```

## 13. File và File Descriptor

- File Descriptor.
- Standard Input, Output, Error.
- File Permissions, File Locking.
- Blocking / Non-blocking I/O.
- File Descriptor Lifecycle.

```cpp
open()
read()
write()
close()
lseek()
fcntl()
flock()
```

### File System

- Tạo, xóa, đọc, ghi file.
- Tạo, xóa và duyệt thư mục.
- Kiểm tra tồn tại.
- Đường dẫn tuyệt đối / tương đối.
- Symbolic Link.
- `std::filesystem`.

---

# PHẦN VIII. INTER-PROCESS COMMUNICATION (IPC)

## 14. Giao tiếp giữa các tiến trình

IPC cho phép các tiến trình trao đổi dữ liệu và đồng bộ hoạt động.

| Cơ chế | Đặc điểm |
|---|---|
| Pipe | Truyền dữ liệu dạng luồng |
| Named Pipe (FIFO) | Pipe có tên trong filesystem |
| Message Queue | Truyền message rời rạc |
| Shared Memory | Nhiều process truy cập vùng nhớ chung |
| Semaphore | Phối hợp truy cập tài nguyên |
| Signal | Thông báo sự kiện cho process |
| Unix Domain Socket | Giao tiếp process trên cùng máy |
| TCP/UDP Socket | Giao tiếp qua tầng mạng |

### IPC nâng cao

- Remote Procedure Call (RPC).
- Interface Definition Language (IDL).
- Serialization / Deserialization.
- Client–Server IPC.
- Synchronous / Asynchronous IPC.
- IPC Security, Service Discovery, Process Death Detection.

### Android Binder IPC

- Binder Driver, Client, Server.
- AIDL, Proxy, Stub.
- Binder Thread Pool.
- Remote Callback.
- Binder Death Notification.
- Reference Counting.

---

# PHẦN IX. EVENT-DRIVEN PROGRAMMING

## 15. Lập trình hướng sự kiện

Event-driven Programming tổ chức luồng xử lý quanh các sự kiện.

- Event, Event Source, Event Handler.
- Event Loop, Event Queue, Event Dispatcher.
- Event Callback, Event Subscription.
- Event Notification, Dispatching.
- Polling so với Event-driven.

API Linux thường dùng:

- `select()`.
- `poll()`.
- `epoll()`.
- `eventfd()`.
- `timerfd()`.
- `signalfd()`.
- `inotify()`.

---

# PHẦN X. FINITE STATE MACHINE

## 16. Máy trạng thái hữu hạn

Finite State Machine (FSM) mô tả hệ thống bằng tập trạng thái và các quy tắc chuyển đổi.

- State, Event, Transition.
- Condition / Guard, Action.
- Initial State, Final State, Error State.

### Các kiểu FSM

- **Moore Machine:** Đầu ra phụ thuộc vào trạng thái hiện tại.
- **Mealy Machine:** Đầu ra phụ thuộc vào trạng thái và đầu vào.

### Các cách triển khai

- `enum` + `switch-case`.
- Function Pointer Table.
- State Pattern.
- Transition Table.
- Hierarchical State Machine.

### Kiến thức bổ sung

- Timeout Handling.
- Retry Mechanism.
- Error Recovery.
- Transition Validation.
- State Persistence.
- Event-driven State Machine.

---

# PHẦN XI. NETWORK PROGRAMMING

## 17. Lập trình mạng

### 17.1. Kiến thức nền tảng

- OSI Model, TCP/IP Model.
- IP Address, Port, DNS, Routing.
- Network Interface.
- TCP, UDP, Socket.

### 17.2. Socket Programming

- Socket Creation.
- Bind, Listen, Accept.
- Connect, Send, Receive.
- Blocking Socket, Non-blocking Socket.
- Socket Timeout, Connection Management.

### 17.3. MQTT

MQTT là giao thức truyền thông theo mô hình Publish/Subscribe.

- MQTT Broker, MQTT Client.
- Publisher, Subscriber.
- Topic, Payload.
- QoS 0, QoS 1, QoS 2.
- Retained Message.
- Last Will and Testament.
- Keep Alive, Persistent Session.
- Reconnection, MQTT over TLS.

### 17.4. Network Security

- TLS/SSL.
- Certificate.
- Public Key, Private Key.
- Certificate Authority (CA).
- Mutual TLS.
- Authentication, Authorization.
- Certificate Verification.
- Certificate Provisioning, Renewal.

---

# PHẦN XII. DATABASE VÀ DATA SERIALIZATION

## 18. Database Programming

### Cơ sở dữ liệu quan hệ

- Database, Table, Row, Column.
- Primary Key, Foreign Key, Index.
- Transaction, ACID Properties.

SQL cần học:

```sql
CREATE TABLE
INSERT
SELECT
UPDATE
DELETE
JOIN
BEGIN TRANSACTION
COMMIT
ROLLBACK
```

### SQLite trong C/C++

- Database Connection.
- Prepared Statement.
- Parameter Binding.
- Query Execution.
- Transaction Management.
- Error Handling.
- Concurrent Database Access.

## 19. Serialization và Deserialization

**Serialization:** Chuyển dữ liệu thành định dạng để lưu hoặc truyền.

**Deserialization:** Khôi phục dữ liệu từ định dạng đã mã hóa.

Định dạng thường gặp:

- JSON, XML, YAML, CSV.
- Binary, Protocol Buffers.

Kiến thức:

- Data Encoding / Decoding.
- JSON Parsing, Data Validation.
- Schema, Version Compatibility.
- Buffer Management.
- Endianness.
- Message Framing.

---

# PHẦN XIII. C++ PREPROCESSOR VÀ BUILD SYSTEM

## 20. Macro và Preprocessor

Preprocessor xử lý mã nguồn trước khi biên dịch.

```cpp
#define
#undef
#include
#ifdef
#ifndef
#if
#elif
#else
#endif
```

Nội dung:

- Object-like Macro, Function-like Macro.
- Conditional Compilation.
- Include Guards.
- Header Files, Header-only Libraries.
- Token Pasting, Stringification, X-Macro.
- Macro Expansion.
- So sánh Macro / Function / Template / `constexpr`.

## 21. Compilation và Linking

```text
Source Code
     |
     v
Preprocessing
     |
     v
Compilation
     |
     v
Assembly
     |
     v
Object Files
     |
     v
Linking
     |
     v
Executable / Library
```

- Compiler, Linker.
- Object File, Symbol.
- Static Library, Shared Library.
- Dynamic Linking.
- Header và Source File.
- Include Path.
- Linker Error.
- Makefile, CMake.
- Cross Compilation.

---

# PHẦN XIV. SOFTWARE ARCHITECTURE

## 22. Kiến trúc phần mềm

Software Architecture xác định cách chia hệ thống thành thành phần và cách các thành phần tương tác.

| Kiến trúc | Đặc điểm |
|---|---|
| Monolithic | Các thành phần nằm trong một ứng dụng chính |
| Layered | Tổ chức thành nhiều tầng |
| Client–Server | Client gửi yêu cầu, server xử lý |
| Service-Oriented | Chức năng được tổ chức thành service |
| Microservices | Các service nhỏ có thể triển khai độc lập |
| Event-driven | Thành phần tương tác bằng sự kiện |
| Component-based | Chia hệ thống thành component có giao diện rõ ràng |
| Hexagonal | Tách logic nghiệp vụ khỏi cơ chế giao tiếp bên ngoài |

Các nguyên tắc:

- SOLID.
- Separation of Concerns.
- Low Coupling / High Cohesion.
- Dependency Injection / Dependency Inversion.
- Interface Segregation.
- Encapsulation, Modularity.
- Reusability, Testability.

---

# PHẦN XV. DEBUGGING VÀ PHÂN TÍCH CODE

## 23. Debugging

Debugging là quá trình xác định, phân tích và sửa lỗi.

- Logging, Breakpoint.
- Step Into / Step Over / Step Out.
- Watch Variable, Call Stack Inspection.
- Memory Inspection.
- Core Dump Analysis.
- Thread Debugging, Deadlock Debugging.

| Công cụ | Công dụng |
|---|---|
| GDB | Debug chương trình C/C++ |
| Valgrind | Phân tích lỗi bộ nhớ và hành vi chương trình |
| AddressSanitizer | Phát hiện lỗi truy cập bộ nhớ |
| ThreadSanitizer | Phát hiện data race |
| strace | Theo dõi system call |
| ltrace | Theo dõi lời gọi thư viện động |
| gprof | Phân tích hiệu năng |
| perf | Phân tích hiệu năng trên Linux |
| journalctl | Xem log hệ thống |
| logcat | Xem log Android |

## 24. Code Analysis

- **Call Graph:** Quan hệ gọi giữa các hàm.
- **Control Flow Analysis:** Các nhánh và thứ tự thực thi.
- **Data Flow Analysis:** Đường đi và phép biến đổi dữ liệu.
- **Dependency Analysis:** Quan hệ phụ thuộc giữa thành phần.
- **Runtime Analysis:** Hoạt động thực tế trong khi chạy.
- **Class Diagram:** Quan hệ giữa các lớp.
- **Sequence Diagram:** Tương tác theo thời gian.
- **Component Diagram:** Module và giao diện.
- **State Diagram:** Trạng thái và chuyển đổi.

---

# PHẦN XVI. CÁC KIẾN THỨC BỔ SUNG

## 25. Error Handling

- Error Codes.
- Exception Handling.
- `try`, `catch`, `throw`, `noexcept`.
- Error Propagation / Recovery.
- Retry, Timeout.
- Fallback Mechanism.
- Fail-fast Design.

## 26. Memory Management

- Stack Memory, Heap Memory, Static Storage.
- Dynamic Allocation.
- Memory Fragmentation, Memory Alignment.
- Memory Pool.
- Buffer Overflow, Buffer Underflow.
- Use-after-free, Memory Leak.
- Reference Counting.

## 27. Software Testing

- Unit Testing.
- Integration Testing.
- System Testing.
- Regression Testing.
- Mock Objects.
- Dependency Injection for Testing.
- Test Coverage.
- Stress Testing.
- Concurrency Testing.
- Fault Injection.

## 28. Communication Protocol Design

- Message Format.
- Header / Payload.
- Sequence Number.
- Checksum / CRC.
- Message Framing.
- Acknowledgement (ACK).
- Timeout, Retry.
- Duplicate Detection.
- Packet Loss Handling.
- Versioning.
- Protocol Compatibility.

## 29. Real-Time Programming

- Hard Real-Time, Soft Real-Time.
- Scheduling, Task Priority.
- Latency, Jitter, Deadline.
- Watchdog.
- Deterministic Execution.
- Priority Inversion.
- Resource Contention.

---

# PHẦN XVII. LỘ TRÌNH HỌC TỔNG HỢP

Học từ nền tảng đến lập trình hệ thống và kiến trúc.

| Giai đoạn | Chủ đề chính | Nội dung |
|---|---|---|
| 1 | C++ Fundamentals | Pointer, Reference, Const, Memory |
| 2 | STL | Vector, Set, Map, Iterator, Algorithm |
| 3 | OOP | Class, Inheritance, Polymorphism |
| 4 | Modern C++ | Smart Pointer, RAII, Template, Move Semantics |
| 5 | Design Patterns | Singleton, Factory, Observer, State |
| 6 | Functional Programming | Function Pointer, Callback, Lambda |
| 7 | Multithreading | Thread, Mutex, Semaphore, Atomic |
| 8 | Asynchronous Programming | Future, Promise, Async, Coroutine |
| 9 | Linux System Programming | Process, File Descriptor, System Call |
| 10 | IPC | Pipe, Message Queue, Shared Memory, Socket |
| 11 | Event-driven Programming | Event Loop, epoll, inotify |
| 12 | State Machine | FSM, State Transition, Timeout |
| 13 | Network Programming | TCP/IP, Socket, MQTT, TLS |
| 14 | Data Management | Database, SQL, JSON, Serialization |
| 15 | Build System | Compilation, Linking, CMake |
| 16 | Software Architecture | SOLID, Layered, Service-oriented |
| 17 | Debugging | GDB, Logging, strace, Valgrind |
| 18 | Software Testing | Unit Test, Integration Test, Mock |
| 19 | Advanced Embedded Linux | Scheduling, Resource Management, IPC |
| 20 | Large Codebase Analysis | Call Graph, Data Flow, Architecture Analysis |

## Tổng kết

Tài liệu có **17 phần, 29 nhóm chủ đề**, hướng tới năm năng lực:

1. **Lập trình C++ chuyên sâu:** Quản lý dữ liệu, bộ nhớ, đối tượng và tài nguyên.
2. **Lập trình hệ thống Linux:** Process, thread, IPC, filesystem, system call.
3. **Lập trình đồng thời và giao tiếp:** Đa luồng, bất đồng bộ, hướng sự kiện và mạng.
4. **Thiết kế kiến trúc phần mềm:** Module, interface, design pattern, khả năng mở rộng.
5. **Đọc hiểu, kiểm thử và bảo trì phần mềm:** Phân tích codebase lớn, truy vết dữ liệu, tìm lỗi và phát triển chức năng.

**Cách học gợi ý:** Học một khái niệm → viết ví dụ C++ nhỏ → thử nghiệm trên Linux → quan sát hành vi bằng debugger/log → áp dụng vào một ứng dụng lớn hơn.
