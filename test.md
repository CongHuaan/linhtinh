# System Architecture

## Overview
Hệ thống Microservices Đặt Hàng được thiết kế để quản lý toàn bộ quy trình từ xem sản phẩm, thêm vào giỏ hàng, đặt hàng đến gửi thông báo xác nhận đơn hàng. Kiến trúc phân tán giúp tăng tính sẵn sàng, độc lập triển khai và khả năng mở rộng cho từng thành phần chức năng.

Mỗi microservice được phát triển và triển khai độc lập, có cơ sở dữ liệu riêng, và giao tiếp với nhau thông qua REST API hoặc messaging (Kafka).

## System Components
- **Discovery Server**: Eureka Server đóng vai trò đăng ký và khám phá dịch vụ, giúp các microservice tìm thấy nhau trong hệ thống phân tán.

- **API Gateway**: Điểm vào duy nhất cho tất cả các yêu cầu từ client, chịu trách nhiệm định tuyến đến các dịch vụ phù hợp, cân bằng tải và bảo mật.

- **Product Service**: Quản lý thông tin sản phẩm, bao gồm tên, mô tả, giá, mã sản phẩm.

- **Inventory Service**: Quản lý tồn kho sản phẩm, kiểm tra và cập nhật số lượng tồn kho.

- **Order Service**: Xử lý đơn hàng, kiểm tra tồn kho và thông tin khách hàng trước khi tạo đơn hàng, phát sự kiện khi đơn hàng được tạo.

- **Customer Service**: Quản lý thông tin khách hàng, xác thực và cung cấp API xác thực cho các service khác.

- **Cart Service**: Quản lý giỏ hàng của khách hàng, thêm/xóa sản phẩm vào giỏ hàng.

- **Notification Service**: Nhận sự kiện đơn hàng từ Kafka và gửi email thông báo cho khách hàng.

## Communication
- **Đồng bộ**: Các service giao tiếp với nhau thông qua REST API sử dụng Spring Cloud OpenFeign client.
  - Order Service gọi đến Customer Service để xác thực khách hàng
  - Order Service gọi đến Inventory Service để kiểm tra tồn kho
  - Order Service gọi đến Cart Service để xóa sản phẩm khỏi giỏ hàng sau khi đặt hàng

- **Bất đồng bộ**: Sử dụng Kafka để giao tiếp không đồng bộ, giúp giảm sự phụ thuộc trực tiếp giữa các service.
  - Order Service phát sự kiện "order-events" khi đơn hàng được tạo
  - Notification Service đăng ký nhận sự kiện "order-events" để gửi email thông báo

- **Service Discovery**: Tất cả các service đăng ký với Eureka Server, giúp các service tự động tìm thấy nhau thông qua tên service mà không cần biết địa chỉ IP/Port cụ thể.

## Data Flow
1. **Đặt hàng**:
   - Client gửi request đặt hàng qua API Gateway
   - API Gateway chuyển request đến Order Service
   - Order Service kiểm tra thông tin khách hàng qua Customer Service
   - Order Service kiểm tra tồn kho qua Inventory Service
   - Nếu hợp lệ, Order Service lưu đơn hàng và phát sự kiện qua Kafka
   - Order Service yêu cầu Cart Service xóa sản phẩm khỏi giỏ hàng
   - Notification Service nhận sự kiện và gửi email xác nhận đơn hàng

2. **Quản lý giỏ hàng**:
   - Client gửi request thêm sản phẩm vào giỏ hàng qua API Gateway
   - API Gateway chuyển request đến Cart Service
   - Cart Service lưu thông tin sản phẩm vào giỏ hàng của khách hàng

3. **Quản lý sản phẩm**:
   - Admin thêm/sửa/xóa sản phẩm qua Product Service
   - Inventory Service cập nhật thông tin tồn kho tương ứng

## Sơ đồ

```mermaid
flowchart TB
    %% Clients
    subgraph "Front-end Applications"
        Customer["Web/Mobile Application\n(Customers)"]
        Admin["Admin Portal\n(Product Management)"]
    end
    
    %% API Gateway & Service Discovery
    subgraph "API Layer"
        Gateway["API Gateway\n(Port: 8080)\nSpring Cloud Gateway"]
        Eureka["Service Discovery\n(Port: 8761)\nEureka Server"]
        
        Gateway <--> Eureka
    end
    
    %% Core Services
    subgraph "Core Microservices"
        direction TB
        ProductService["Product Service\n(Port: 8081)\nManages product information"]
        InventoryService["Inventory Service\n(Port: 8082)\nManages product stock"]
        OrderService["Order Service\n(Port: 8083)\nProcesses customer orders"]
        CustomerService["Customer Service\n(Port: 8085)\nManages user accounts"]
        CartService["Cart Service\n(Port: 8087)\nManages shopping carts"]
        NotificationService["Notification Service\n(Port: 8086)\nSends email notifications"]
    end
    
    %% Databases
    subgraph "Data Layer"
        ProductDB[(Product DB\nMySQL)]
        InventoryDB[(Inventory DB\nMySQL)]
        OrderDB[(Order DB\nMySQL)]
        CustomerDB[(Customer DB\nMySQL)]
        CartDB[(Cart DB\nMySQL)]
    end
    
    %% Messaging
    subgraph "Messaging Layer"
        Kafka["Kafka\nEvent Messaging"]
    end
    
    %% Client to Gateway Connections
    Customer --> |HTTP Requests| Gateway
    Admin --> |HTTP Requests| Gateway
    
    %% Gateway to Services Connections
    Gateway --> |Routes Requests| ProductService
    Gateway --> |Routes Requests| InventoryService
    Gateway --> |Routes Requests| OrderService
    Gateway --> |Routes Requests| CustomerService
    Gateway --> |Routes Requests| CartService
    Gateway --> |Routes Requests| NotificationService
    
    %% Service Registration
    ProductService --> |Registers with| Eureka
    InventoryService --> |Registers with| Eureka
    OrderService --> |Registers with| Eureka
    CustomerService --> |Registers with| Eureka
    CartService --> |Registers with| Eureka
    NotificationService --> |Registers with| Eureka
    
    %% Service to Database Connections
    ProductService --> |JDBC| ProductDB
    InventoryService --> |JDBC| InventoryDB
    OrderService --> |JDBC| OrderDB
    CustomerService --> |JDBC| CustomerDB
    CartService --> |JDBC| CartDB
    
    %% Inter-Service Communication (Synchronous)
    OrderService --> |Feign Client| CustomerService
    OrderService --> |Feign Client| InventoryService
    OrderService --> |Feign Client| CartService
    
    %% Asynchronous Communication
    OrderService --> |Publishes Events| Kafka
    Kafka --> |Consumes Events| NotificationService
    
    %% Styling
    classDef frontEnd fill:#ff9999,stroke:#333,stroke-width:2px;
    classDef gateway fill:#66b3ff,stroke:#333,stroke-width:2px;
    classDef eureka fill:#ff9966,stroke:#333,stroke-width:2px;
    classDef service fill:#b3b3b3,stroke:#333,stroke-width:2px;
    classDef database fill:#80b3ff,stroke:#333,stroke-width:2px;
    classDef messaging fill:#ff66cc,stroke:#333,stroke-width:2px;
    
    class Customer,Admin frontEnd;
    class Gateway gateway;
    class Eureka eureka;
    class ProductService,InventoryService,OrderService,CustomerService,CartService,NotificationService service;
    class ProductDB,InventoryDB,OrderDB,CustomerDB,CartDB database;
    class Kafka messaging;
```

```mermaid
sequenceDiagram
    autonumber
    
    actor Customer as Customer
    participant Frontend as Web/Mobile App
    participant Gateway as API Gateway
    participant ProductSvc as Product Service
    participant CartSvc as Cart Service
    participant CustomerSvc as Customer Service
    participant OrderSvc as Order Service
    participant InventorySvc as Inventory Service
    participant Kafka as Kafka
    participant NotificationSvc as Notification Service
    
    %% Browse Products Sequence
    Customer->>+Frontend: Browse Products
    Frontend->>+Gateway: GET /api/products
    Gateway->>+ProductSvc: getProducts()
    ProductSvc-->>-Gateway: Return Product List
    Gateway-->>-Frontend: Product Response
    Frontend-->>-Customer: Display Products
    
    %% View Cart Sequence
    Customer->>+Frontend: View Cart
    Frontend->>+Gateway: GET /api/cart/{customerId}
    Gateway->>+CartSvc: getCart()
    CartSvc-->>-Gateway: Return Cart Details
    Gateway-->>-Frontend: Cart Response
    Frontend-->>-Customer: Display Cart with Checkout Option
    
    %% Checkout Sequence
    Customer->>+Frontend: Click Checkout
    Frontend->>+Gateway: POST /api/orders
    Gateway->>+OrderSvc: placeOrder(orderId)
    
    %% Order Service Internal Processing
    OrderSvc->>+CustomerSvc: validateCustomer(customerId)
    CustomerSvc-->>-OrderSvc: Customer Validation Response
    
    OrderSvc->>+InventorySvc: checkStock(skuCode, quantity)
    InventorySvc-->>-OrderSvc: Stock Availability Response
    
    OrderSvc->>OrderSvc: Create Order Record
    
    OrderSvc->>+InventorySvc: updateStock(skuCode, quantity)
    InventorySvc->>InventorySvc: Update Inventory Record
    InventorySvc-->>-OrderSvc: Inventory Update Response
    
    OrderSvc->>+CartSvc: removeFromCart(customerId, productId)
    CartSvc->>CartSvc: Remove Items from Cart
    CartSvc-->>-OrderSvc: Cart Update Response
    
    %% Asynchronous Event Publishing
    OrderSvc->>+Kafka: Publish OrderPlacedEvent
    OrderSvc-->>-Gateway: Order Creation Success
    Gateway-->>-Frontend: Order Confirmation Response
    Frontend-->>-Customer: Display Order Confirmation
    
    %% Asynchronous Event Processing
    Kafka-->>+NotificationSvc: Consume OrderPlacedEvent
    NotificationSvc->>NotificationSvc: Prepare Email Content
    NotificationSvc->>NotificationSvc: Send Confirmation Email
    NotificationSvc-->>-Kafka: Acknowledge Message
```

## Scalability & Fault Tolerance
- **Horizontally Scalable**: Mỗi microservice có thể được mở rộng độc lập bằng cách tăng số lượng instance dựa trên tải hệ thống. Eureka Server tự động phát hiện các instance mới.

- **Fault Isolation**: Lỗi trong một service không ảnh hưởng đến toàn bộ hệ thống. Ví dụ, nếu Notification Service gặp sự cố, các đơn hàng vẫn được xử lý bình thường.

- **Eventual Consistency**: Sử dụng mô hình nhất quán cuối cùng qua Kafka giúp hệ thống vẫn hoạt động khi có service bị lỗi tạm thời.

- **Database Per Service**: Mỗi service có cơ sở dữ liệu riêng, giảm thiểu điểm lỗi chung và tăng khả năng mở rộng.

- **API Gateway**: Cung cấp các tính năng như rate limiting, circuit breaking để bảo vệ các service khỏi quá tải.
