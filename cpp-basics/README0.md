# Header Guards trong C++
 
## Mục đích
 
Trong C++, một file header (`.h`, `.hpp`) có thể được `#include` từ nhiều nơi khác nhau.
 
Nếu cùng một header được include nhiều hơn một lần trong cùng một translation unit, compiler có thể báo lỗi:
 
```text
redefinition of class
redefinition of struct
redefinition of function
```
 
Để tránh vấn đề này, chúng ta sử dụng:
 
- `#pragma once`
- Include Guard (`#ifndef`, `#define`, `#endif`)
 
---
 
## Cách 1: Sử dụng `#pragma once`
 
```cpp
#pragma once
 
class MyClass
{
public:
void Hello();
};
```
 
### Ưu điểm
 
- Ngắn gọn.
- Dễ đọc.
- Không cần đặt tên macro.
- Được hỗ trợ bởi hầu hết compiler hiện đại:
- MSVC
- GCC
- Clang
 
### Nhược điểm
 
- Không thuộc chuẩn C++ chính thức.
- Phụ thuộc vào compiler.
 
---
 
## Cách 2: Include Guard
 
```cpp
#ifndef MYCLASS_H
#define MYCLASS_H
 
class MyClass
{
public:
void Hello();
};
 
#endif
```
 
### Cách hoạt động
 
Lần đầu include:
 
```cpp
#ifndef MYCLASS_H
```
 
Macro chưa tồn tại nên compiler xử lý nội dung file.
 
Sau đó:
 
```cpp
#define MYCLASS_H
```
 
Macro được định nghĩa.
 
Những lần include tiếp theo:
 
```cpp
#ifndef MYCLASS_H
```
 
Điều kiện sai nên toàn bộ nội dung bị bỏ qua.
 
### Ưu điểm
 
- Là giải pháp chuẩn của C/C++.
- Hoạt động trên mọi compiler.
 
### Nhược điểm
 
- Dài hơn.
- Có thể bị trùng tên macro nếu đặt tên không cẩn thận.
 
---
 
## Tại sao không chỉ dùng `#ifdef`
 
Sai:
 
```cpp
#ifdef MYCLASS_H
 
class MyClass {};
 
#endif
```
 
Nếu `MYCLASS_H` chưa được định nghĩa thì toàn bộ nội dung sẽ bị bỏ qua.
 
Đúng:
 
```cpp
#ifndef MYCLASS_H
#define MYCLASS_H
 
class MyClass {};
 
#endif
```
 
---
 
## Ví dụ lỗi khi không dùng Header Guard
 
### A.h
 
```cpp
class A {};
```
 
### main.cpp
 
```cpp
#include "A.h"
#include "A.h"
```
 
Compiler sẽ nhìn thấy:
 
```cpp
class A {};
class A {};
```
 
Kết quả:
 
```text
error: redefinition of 'class A'
```
 
---
 
## Ví dụ với `#pragma once`
 
### A.h
 
```cpp
#pragma once
 
class A {};
```
 
### main.cpp
 
```cpp
#include "A.h"
#include "A.h"
```
 
Compiler sẽ chỉ xử lý nội dung của `A.h` một lần.
 
---
 
## Nên dùng cách nào?
 
### Dự án hiện đại
 
Khuyến nghị:
 
```cpp
#pragma once
```
 
Vì đơn giản và được hỗ trợ rộng rãi.
 
### Cần tương thích tối đa
 
Sử dụng:
 
```cpp
#ifndef HEADER_NAME_H
#define HEADER_NAME_H
 
// code
 
#endif
```
 
---
 
## Kết luận
 
Mỗi file header trong C++ nên có cơ chế chống include nhiều lần.
 
Hai cách phổ biến:
 
### `#pragma once`
 
```cpp
#pragma once
```
 
### Include Guard
 
```cpp
#ifndef HEADER_NAME_H
#define HEADER_NAME_H
 
// code
 
#endif
```
 
Mục tiêu chung của cả hai là ngăn chặn lỗi định nghĩa lại (redefinition) khi một header được include nhiều lần.
