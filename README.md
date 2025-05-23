# 🧩 Hệ thống Đặt hàng - Microservices

## 👩‍💻 Thành viên nhóm

| Họ và Tên | MSSV | Vai trò |
|-----------|------|---------|
| Vũ Hữu Đức | B21DCCN259 | Cart Service, Discovery Server, Product Service, Customer Service, API Gateway |
| Trần Xuân Đạt | B21DCCN223 | Order Service |
| Nguyễn Công Huân | B21DCCN403 | Inventory Service, Notification Service |

---
Đây là dự án xây dựng hệ thống đặt hàng dựa trên kiến trúc microservices. Hệ thống được thiết kế để quản lý toàn bộ quy trình từ xem sản phẩm, thêm vào giỏ hàng, đặt hàng đến gửi thông báo xác nhận đơn hàng.

---
## 📁 Folder Structure

```
ecommerce-system/
├── gateway/                  # API Gateway Service
│
├── services/                 # Microservices
│   ├── product-service/     # Product Management Service
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/
│   │   │   │   │   └── com/microservices/productservice/
│   │   │   │   │       ├── config/           # Configuration classes
│   │   │   │   │       ├── controller/       # REST controllers
│   │   │   │   │       ├── service/          # Business logic
│   │   │   │   │       ├── repository/       # Data access
│   │   │   │   │       ├── model/            # Domain models
│   │   │   │   │       ├── dto/              # Data transfer objects
│   │   │   │   │       └── ProductServiceApplication.java
│   │   │   │   └── resources/
│   │   │   └── test/
│   │   ├── .mvn/
│   │   ├── target/
│   │   ├── .gitattributes
│   │   ├── .gitignore
│   │   ├── docker-compose.yml
│   │   ├── Dockerfile
│   │   ├── mvnw
│   │   ├── mvnw.cmd
│   │   └── pom.xml
│   ├── cart-service/       # Shopping Cart Service
│   ├── order-service/      # Order Management Service
│   ├── inventory-service/  # Inventory Management Service
│   ├── customer-service/   # Customer Management Service
│   ├── notification-service/ # Notification Service
│   └── discovery-server/   # Service Registry (Eureka)
│
├── docs/                    # Documentation
│   ├── api-specs/          # API Specifications
│   │   ├── gateway.yaml
│   │   ├── discovery-server.yaml
│   │   ├── product-service.yaml
│   │   ├── cart-service.yaml
│   │   ├── order-service.yaml
│   │   ├── inventory-service.yaml
│   │   ├── customer-service.yaml
│   │   └── notification-service.yaml
│   ├── analysis-and-design.md
│   └── architecture.md
│
├── frontend/                # Frontend Application
├── scripts/                 # Utility Scripts
├── docker-compose.yml       # Docker Compose Configuration
├── env.example             # Environment Variables Example
├── .gitignore
└── README.md
```

## 🚀 Hướng dẫn cài đặt

### Yêu cầu hệ thống
- Docker và Docker Compose
- JDK 17 trở lên
- Maven 3.8 trở lên

### Các bước cài đặt

1. **Clone repository**

   ```bash
   git clone https://github.com/jnp2018/mid-project-403259223.git
   ```

2. **Cấu hình biến môi trường**

   ```bash
   cp .env.example .env
   # Chỉnh sửa file .env theo môi trường của bạn
   ```

3. **Khởi động hệ thống với Docker Compose**

   ```bash
   docker-compose up --build
   ```

4. **Kiểm tra các dịch vụ**
    - Eureka Dashboard: http://localhost:8761
    - API Gateway: http://localhost:8080
    - Swagger UI: http://localhost:8080/swagger-ui.html
    - Kafka UI: http://localhost:9021 (nếu có)

---

## 📚 Mô tả Microservices

### Discovery Server (Eureka)
- **Port**: 8761
- **Chức năng**: Service discovery và đăng ký
- **Công nghệ**: Spring Cloud Netflix Eureka

### API Gateway
- **Port**: 8080
- **Chức năng**: Điều hướng request, cân bằng tải
- **Công nghệ**: Spring Cloud Gateway

### Product Service
- **Port**: 8081
- **Chức năng**: Quản lý thông tin sản phẩm (mã, tên, giá, mô tả, ảnh)
- **Công nghệ**: Spring Boot, Spring Data JPA, MySQL
- **Database**: product_service
- **Endpoints**: `/api/products/**`

### Inventory Service
- **Port**: 8082
- **Chức năng**: Quản lý tồn kho sản phẩm, kiểm tra số lượng có sẵn
- **Công nghệ**: Spring Boot, Spring Data JPA, MySQL
- **Database**: inventory_service
- **Endpoints**: `/api/inventory/**`

### Order Service
- **Port**: 8083
- **Chức năng**: Xử lý đơn hàng, kiểm tra tồn kho và khách hàng trước khi đặt hàng
- **Công nghệ**: Spring Boot, Spring Data JPA, MySQL, Kafka
- **Database**: order_service
- **Endpoints**: `/api/orders/**`
- **Events**: Phát sự kiện `order-events` qua Kafka khi đơn hàng được tạo

### Customer Service
- **Port**: 8085
- **Chức năng**: Quản lý thông tin khách hàng
- **Công nghệ**: Spring Boot, Spring Data JPA, MySQL
- **Database**: customer_service
- **Endpoints**: `/api/customers/**`, `/api/auth/**`

### Cart Service
- **Port**: 8087
- **Chức năng**: Quản lý giỏ hàng của khách hàng
- **Công nghệ**: Spring Boot, Spring Data JPA, MySQL
- **Database**: cart_service
- **Endpoints**: `/api/cart/**`

### Notification Service
- **Port**: 8086
- **Chức năng**: Gửi email thông báo đơn hàng
- **Công nghệ**: Spring Boot, Kafka, Gmail SMTP
- **Endpoints**: `/api/notification/**`
- **Events**: Lắng nghe sự kiện `order-events` từ Kafka

---

## 📌 API Endpoints

### Product Service
- `GET /api/products`: Lấy danh sách tất cả sản phẩm
- `GET /api/products/{id}`: Lấy thông tin sản phẩm theo ID
- `POST /api/products`: Tạo sản phẩm mới
  - Body: `{ "skuCode": string, "name": string, "price": number, "imageUrl": string, "description": string }`

### Inventory Service
- `GET /api/inventory`: Kiểm tra tồn kho
  - Params: `skuCode`, `quantity`
- `PUT /api/inventory/{skuCode}`: Cập nhật số lượng tồn kho
  - Params: `quantity`
- `GET /api/inventory/{skuCode}/quantity`: Lấy số lượng tồn kho hiện tại

### Order Service
- `POST /api/orders`: Tạo đơn hàng mới
  - Body: 
  ```json
  {
    "customerId": number,
    "productRequest": {
      "productId": number,
      "productName": string,
      "skudCode": string,
      "quantity": number,
      "price": number
    }
  }
  ```

### Customer Service
- `GET /api/customers/{id}`: Lấy thông tin khách hàng theo ID
- `POST /api/auth/register`: Đăng ký tài khoản mới
- `POST /api/auth/login`: Đăng nhập

### Cart Service
- `GET /api/cart/{customerId}`: Lấy giỏ hàng của khách hàng
- `POST /api/cart/add`: Thêm sản phẩm vào giỏ hàng
  - Body: `{ "customerId": number, "productId": number, "quantity": number }`
- `DELETE /api/cart/{customerId}/{productId}`: Xóa sản phẩm khỏi giỏ hàng

---

## 🧪 Demo

### Luồng Đặt Hàng Mẫu

1. Xem danh sách sản phẩm:
   ```bash
   curl -X GET http://localhost:8080/api/products
   ```

2. Thêm sản phẩm vào giỏ hàng:
   ```bash
   curl -X POST http://localhost:8080/api/cart/add \
   -H "Content-Type: application/json" \
   -d '{"customerId": 1, "productId": 1, "quantity": 2}'
   ```

3. Đặt hàng:
   ```bash
   curl -X POST http://localhost:8080/api/orders \
   -H "Content-Type: application/json" \
   -d '{"customerId": 1, "productRequest": {"productId": 1, "productName": "Smartphone", "skudCode": "PROD-123", "quantity": 2, "price": 999.99}}'
   ```

---

## 🧩 Kiến trúc hệ thống

### Luồng đặt hàng:
1. Khách hàng xem sản phẩm qua Product Service
2. Thêm sản phẩm vào giỏ hàng qua Cart Service
3. Khách hàng đặt hàng qua Order Service, hệ thống sẽ:
   - Kiểm tra xác thực khách hàng qua Customer Service
   - Kiểm tra tồn kho qua Inventory Service
   - Tạo đơn hàng và lưu vào cơ sở dữ liệu
   - Phát sự kiện đơn hàng thành công qua Kafka
   - Xóa sản phẩm khỏi giỏ hàng qua Cart Service
4. Notification Service nhận sự kiện đơn hàng và gửi email xác nhận

### Giao tiếp giữa các service:
- **Đồng bộ**: REST API thông qua Feign Client
- **Bất đồng bộ**: Kafka cho các sự kiện như đặt hàng thành công

### Khả năng mở rộng:
- Mỗi service có cơ sở dữ liệu riêng
- Service Discovery cho phép thêm nhiều instance của cùng một service
- API Gateway điều hướng yêu cầu và cân bằng tải

---

## 📚 Tài liệu tham khảo

- [Spring Boot](https://spring.io/projects/spring-boot)
- [Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)
- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Kafka](https://kafka.apache.org/)
- [Feign Client](https://spring.io/projects/spring-cloud-openfeign)
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
