---
title: Đã có HTTP, tại sao còn cần RPC? (Tầng ứng dụng)
description: So sánh toàn diện sự khác biệt giữa HTTP và RPC, làm rõ mối quan hệ tầng thứ, định dạng tuần tự hóa (Serialization), cơ chế truyền tải và lý do kiến trúc Microservices ưu tiên sử dụng RPC.
category: Cơ sở máy tính
tag:
  - Mạng máy tính
head:
  - - meta
    - name: keywords
      content: HTTP, RPC, RESTful, gRPC, Dubbo, Thrift, Tuần tự hóa, Microservices, Giao tiếp dịch vụ
---

Trong các cuộc phỏng vấn kiến trúc Backend và hệ thống phân tán, có một câu hỏi kinh điển:

**"HTTP đã rất phổ biến và mạnh mẽ rồi, tại sao trong kiến trúc Microservices các hệ thống nội bộ lại thường sử dụng RPC?"**

Câu hỏi này thoạt nhìn như đang so sánh hai giao thức đối đầu nhau. Nhưng trên thực tế, **HTTP và RPC không hoàn toàn nằm trên cùng một mặt phẳng khái niệm**.

Bài viết này chủ yếu trả lời các câu hỏi:

1. RPC thực chất là gì, và nó khác gì so với HTTP / RESTful API?
2. Tại sao gọi dịch vụ nội bộ (Internal Service-to-Service) lại ưu tiên dùng RPC?
3. RPC tối ưu hơn HTTP ở những điểm nào (Tuần tự hóa, Giao thức truyền tải, Kết nối)?
4. Trong hệ thống thực tế, HTTP và RPC phân chia vai trò như thế nào?

## RPC là gì?

**RPC (Remote Procedure Call - Lời gọi thủ tục từ xa)** là một mô hình lập trình mạng cho phép một chương trình máy tính gọi một hàm/thủ tục trên một không gian địa chỉ khác (thường là một máy tính khác trong mạng) mà cú pháp và trải nghiệm lập trình giống hệt như đang gọi một hàm cục bộ (Local Function Call).

```java
// Lời gọi hàm cục bộ thông thường
User user = userService.getUserById(123L);

// Lời gọi RPC qua mạng - Trải nghiệm code giống hệt hàm cục bộ!
User user = remoteUserService.getUserById(123L);
```

Phía sau lời gọi tưởng chừng đơn giản đó, RPC Framework (như gRPC, Dubbo, Thrift, Spring Cloud OpenFeign) đã âm thầm thực hiện:
1. **Dynamic Proxy (Proxy động)**: Đóng gói lời gọi hàm thành đối tượng yêu cầu.
2. **Serialization (Tuần tự hóa)**: Chuyển đổi tham số hàm và đối tượng Java thành chuỗi byte nhị phân.
3. **Network Transport (Truyền tải mạng)**: Mở kết nối Socket (TCP / HTTP/2), gửi dữ liệu qua mạng tới Server.
4. **Deserialization (Giải tuần tự hóa)**: Phía Server đọc dữ liệu byte, chuyển ngược thành đối tượng và thực thi hàm thực tế.
5. **Return Value (Trả về kết quả)**: Đóng gói kết quả trả về qua mạng cho Client.

## HTTP vs RPC: So sánh bản chất

| Tiêu chí | HTTP (RESTful API) | RPC Framework |
| --- | --- | --- |
| Bản chất khái niệm | Là một Giao thức mạng tầng ứng dụng cụ thể | Là một Mô hình kiến trúc / Framework giao tiếp từ xa |
| Mô hình lập trình | Hướng tài nguyên (Resource-oriented: URI, GET, POST, PUT, DELETE) | Hướng hành động/hàm (Action-oriented: `service.method(args)`) |
| Định dạng dữ liệu | Thường dùng văn bản (JSON, XML) | Thường dùng nhị phân (Protobuf, Thrift, Hessian, Kryo) |
| Tầng giao vận | TCP (HTTP/1.1, HTTP/2) hoặc UDP (HTTP/3) | TCP tùy chỉnh (Dubbo) hoặc HTTP/2 (gRPC) |
| Khả năng tích hợp | Rất rộng rãi, phù hợp giao tiếp đa nền tảng, Frontend-Backend, Public API | Phù hợp giao tiếp nội bộ hiệu năng cao giữa các Microservices |
| Tính năng kèm theo | Cần tự tích hợp thêm | Thường tích hợp sẵn Service Discovery, Load Balancing, Circuit Breaker, Tracing |

## Tại sao trong kiến trúc Microservices nội bộ lại ưu tiên RPC?

### 1. Hiệu suất tuần tự hóa và kích thước gói tin (Serialization Efficiency)

- **HTTP / RESTful** thường sử dụng định dạng **JSON**:
  - JSON là định dạng văn bản (Text-based), chứa cả tên trường (Key) lặp đi lặp lại trong mọi gói tin.
  - Tốn nhiều băng thông mạng và tốn CPU để parse chuỗi String thành Object.
- **RPC Framework** (như gRPC dùng Protobuf):
  - Sử dụng định dạng nhị phân (Binary-based).
  - Không truyền tên trường mà dùng Tag số nguyên siêu nhỏ (1-2 byte).
  - Tốc độ Serialize/Deserialize nhanh hơn JSON từ 3 đến 10 lần, kích thước gói tin nhỏ hơn 30% - 70%.

### 2. Tối ưu kết nối và giao thức truyền tải (Connection & Transport)

- **HTTP/1.1**: Header cồng kềnh dạng text, bị vấn đề nghẽn đầu hàng (Head-of-Line Blocking), chi phí duy trì kết nối lớn.
- **RPC Framework**:
  - Nhiều RPC framework sử dụng trực tiếp kết nối TCP dài với giao thức nhị phân tối giản (Header chỉ vài byte).
  - gRPC chạy trên nền **HTTP/2**: Hỗ trợ Multiplexing (hàng nghìn request đồng thời trên 1 kết nối TCP), nén Header nhị phân (HPACK) và truyền luồng hai chiều (Streaming).

### 3. Trải nghiệm phát triển (Developer Experience & Type Safety)

- Khi dùng RESTful API, Client phải tự quản lý URL, HTTP Method, parse Response JSON thủ công, dễ sai sót kiểu dữ liệu khi Backend thay đổi API mà không thông báo.
- Với RPC (sử dụng IDL như `.proto` hoặc Java Interface chung):
  - Sinh code tự động (Code Generation) cho cả Client và Server.
  - Đảm bảo kiểm tra kiểu dữ liệu tĩnh tại thời điểm biên dịch (Compile-time Type Safety).
  - IDE hỗ trợ tự động gợi ý code (Auto-completion), refactor dễ dàng.

### 4. Hệ sinh thái quản trị dịch vụ (Service Governance)

Các RPC Framework chuyên nghiệp (như Dubbo, gRPC) được thiết kế chuyên biệt cho Microservices, tích hợp sẵn:
- Service Registration & Discovery (Nacos, Zookeeper, Consul).
- Load Balancing (Round Robin, Least Active, Random).
- Fault Tolerance (Failover, Failfast, Failsafe).
- Rate Limiting & Circuit Breaking.
- Distributed Tracing.

## Kiến trúc thực tế: Phối hợp giữa HTTP và RPC

Trong các hệ thống doanh nghiệp hiện đại, HTTP và RPC không triệt tiêu nhau mà phối hợp chặt chẽ:

```text
[ Browser / Mobile App / Third-party API ]
                  │
                  ▼  (HTTP / HTTPS / RESTful / JSON)
        [ API Gateway (Spring Cloud Gateway / APISIX / Nginx) ]
                  │
                  ▼  (RPC: gRPC / Dubbo / Thrift / Binary)
     ┌────────────┼────────────┐
     ▼            ▼            ▼
[User Service] [Order Service] [Payment Service]
```

- **Ranh giới bên ngoài (North-South Traffic)**: Dùng **HTTP / RESTful** để giao tiếp với Browser, Mobile App, đối tác bên thứ ba nhờ tính mở, tương thích đa nền tảng và dễ debug.
- **Giao tiếp nội bộ bên trong (East-West Traffic)**: Dùng **RPC** giữa hàng trăm Microservices với nhau để tối ưu độ trễ, tiết kiệm tài nguyên CPU/băng thông và tận dụng khả năng quản trị dịch vụ mạnh mẽ.

## Tổng kết

- HTTP là một giao thức mạng cụ thể; RPC là một mô hình gọi hàm từ xa.
- RPC có thể chạy trên nền TCP tùy chỉnh hoặc chạy ngay trên nền HTTP/2 (như gRPC).
- Trong kiến trúc Microservices, RPC được ưa chuộng cho giao tiếp nội bộ nhờ: Tuần tự hóa nhị phân gọn nhẹ, tối ưu kết nối, an toàn kiểu dữ liệu và tích hợp sẵn hệ sinh thái quản trị dịch vụ.

<!-- @include: @article-footer.snippet.md -->
