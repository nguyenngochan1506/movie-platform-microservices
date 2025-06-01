## Danh sách các service cần thiết

1. **Identity Service**
   - **Chức năng**: Xác thực và phân quyền (đăng ký, đăng nhập, quản lý mật khẩu, cấp phát JWT).
   - **Database**: **PostgreSQL** (dữ liệu có cấu trúc, hỗ trợ giao dịch).
   - **API mẫu**:
     - POST /api/users/register: Đăng ký người dùng.
     - POST /api/users/login: Đăng nhập, trả về JWT.
     - GET /api/users/{id}: Lấy thông tin cơ bản (username, email).
   - **Công nghệ**:
     - Spring Boot, Spring Security, JWT.
     - Spring Data JPA, PostgreSQL.
     - Kafka (xuất bản UserRegisteredEvent để đồng bộ với Profile Service).
2. **Profile Service**
   - **Chức năng**: Quản lý hồ sơ người dùng (display name, avatar, sở thích) và mối quan hệ xã hội (follow, friend).
   - **Database**: **Neo4j** (database đồ thị, tối ưu cho mối quan hệ xã hội).
   - **API mẫu**:
     - POST /api/profiles: Tạo hồ sơ người dùng.
     - GET /api/profiles/{userId}: Lấy hồ sơ.
     - POST /api/profiles/{userId}/follow: Follow người dùng khác.
   - **Công nghệ**:
     - Spring Boot, Spring Data Neo4j.
     - Neo4j (mô hình hóa node Profile và relationship FOLLOWS).
     - Kafka (lắng nghe UserRegisteredEvent để tạo profile).
3. **Movie Service**
   - **Chức năng**: Quản lý thông tin phim (danh sách phim, chi tiết phim, lịch chiếu, streaming).
   - **Database**: **PostgreSQL** (dữ liệu phim và lịch chiếu có cấu trúc).
   - **API mẫu**:
     - GET /api/movies: Lấy danh sách phim.
     - GET /api/movies/{id}: Lấy chi tiết phim.
     - GET /api/movies/{id}/showtimes: Lấy lịch chiếu.
   - **Công nghệ**:
     - Spring Boot, Spring Data JPA.
     - PostgreSQL.
     - AWS S3/Cloudflare Stream (nếu hỗ trợ streaming).
4. **Ticket Service**
   - **Chức năng**: Đặt vé xem phim tại rạp (chọn ghế, thanh toán, lịch sử giao dịch).
   - **Database**: **PostgreSQL** (hỗ trợ giao dịch phức tạp như khóa ghế).
   - **API mẫu**:
     - POST /api/tickets: Đặt vé.
     - GET /api/tickets/{userId}: Lấy lịch sử vé.
     - GET /api/tickets/showtimes/{showtimeId}/seats: Lấy trạng thái ghế.
   - **Công nghệ**:
     - Spring Boot, Spring Data JPA.
     - PostgreSQL.
     - Stripe/PayPal/VNPay (thanh toán).
5. **File Serivce**
   - **Chức năng**: Quản lý về file
   - **Database**: **MongoDB** 
   - **Dùng S3Bucked**
6. **Social Service**
   - **Chức năng**: Tính năng mạng xã hội (đăng bài, bình luận, thích bài viết về phim).
   - **Database**: **MongoDB** (dữ liệu phi cấu trúc như bài viết, bình luận).
   - **API mẫu**:
     - POST /api/posts: Đăng bài viết.
     - GET /api/posts/{movieId}: Lấy bài viết liên quan đến phim.
     - POST /api/posts/{postId}/comments: Thêm bình luận.
   - **Công nghệ**:
     - Spring Boot, Spring Data MongoDB.
     - MongoDB.
     - Kafka (xuất bản/lắng nghe sự kiện như PostCreatedEvent).
7. **API Gateway**
   - **Chức năng**: Định tuyến yêu cầu từ client, xác thực, cân bằng tải.
   - **API mẫu**:
     - Chuyển tiếp /api/users/* đến identity-service.
     - Chuyển tiếp /api/profiles/* đến profile-service.
     - Chuyển tiếp /api/movies/* đến movie-service.
   - **Công nghệ**:
     - Spring Cloud Gateway.
     - Spring Security, JWT.
     - Eureka Client (kết nối với eureka-server).
8. **Discovery Service (Eureka Server)**
   - **Chức năng**: Quản lý vị trí (IP/port) của các microservices.
   - **Công nghệ**:
     - Spring Cloud Netflix Eureka.
   - **Vai trò**: Cho phép các microservices và API Gateway tìm thấy nhau động, hỗ trợ load balancing.

---

### 3. **Công nghệ cần thiết**

Dựa trên các microservices, đây là danh sách công nghệ được đề xuất:

#### a. **Backend**

- **Framework**:
  - **Spring Boot 3.x**: Xây dựng microservices với cấu trúc DDD.
  - **Spring Cloud**:
    - **Spring Cloud Gateway**: API Gateway.
    - **Spring Cloud Netflix Eureka**: Discovery Service.
    - **Spring Cloud Sleuth + Zipkin**: Theo dõi phân tán.
- **Bảo mật**:
  - **Spring Security + JWT**: Xác thực và phân quyền trong identity-service và api-gateway.
- **Ánh xạ**:
  - **MapStruct**: Ánh xạ DTO và Entity trong user-interface.
- **Event-Driven**:
  - **Apache Kafka**: Giao tiếp bất đồng bộ (ví dụ: đồng bộ giữa identity-service và profile-service).

#### b. **Database**

- **Identity Service**: **PostgreSQL** (dữ liệu xác thực: username, password, email).
- **Profile Service**: **Neo4j** (hồ sơ và mối quan hệ xã hội: follow, friend).
- **Movie Service**: **PostgreSQL** (phim, lịch chiếu).
- **Ticket Service**: **PostgreSQL** (vé, ghế, giao dịch).
- **Social Service**: **MongoDB** (bài viết, bình luận).

#### c. **Streaming (nếu có)**

- **AWS S3** hoặc **Cloudflare Stream**: Lưu trữ video.
- **HLS**: Giao thức streaming.
- **Cloudflare/AWS CloudFront**: CDN để tăng tốc độ phân phối.

#### d. **Frontend**

- **React** (CDN: https://cdn.jsdelivr.net/npm/react@18/umd/react.development.js).
- **Tailwind CSS**: Styling.
- **Axios**: Gọi API từ frontend đến API Gateway.

#### e. **Triển khai**

- **Docker**: Container hóa từng microservice.
- **Docker Compose**: Chạy hệ thống cục bộ.
- **Kubernetes**: Triển khai production.
- **GitHub Actions**: CI/CD.

#### f. **Giám sát**

- **Prometheus + Grafana**: Giám sát hiệu suất.
- **Spring Cloud Sleuth + Zipkin**: Theo dõi yêu cầu.
- **ELK Stack**: Quản lý log.

#### g. **Thanh toán**

- **Stripe/PayPal/VNPay**: Tích hợp trong ticket-service.
