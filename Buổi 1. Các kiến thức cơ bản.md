# Buổi 1: Các kiến thức cơ bản

## A. HTTP (HyperText Transfer Protocol – Giao thức truyền tải siêu văn bản)

### I. HTTP là gì?

#### 1. Định nghĩa

**HTTP (HyperText Transfer Protocol)** là **giao thức ứng dụng (application protocol)** dùng để **truyền dữ liệu giữa client và server** qua mạng Internet.
HTTP định nghĩa **cách mà client (ví dụ trình duyệt web hoặc ứng dụng) gửi yêu cầu (request)** và **cách server phản hồi (response)**.

Nói cách khác, HTTP giống như “ngôn ngữ chung” để **trình duyệt nói chuyện với máy chủ**.

#### 2. Đặc điểm chính của HTTP

**a. Stateless (Không lưu trạng thái):**
Mỗi request độc lập, server không nhớ request trước đó. Nếu cần lưu trạng thái, ta phải dùng cookie, session, hoặc token.

**b. Client–Server Model (Mô hình máy khách – máy chủ):**
Client gửi request, server xử lý và trả response.

**c. Text-based Protocol (Dạng văn bản):**
Dễ đọc, dễ debug vì mọi thứ đều là text.

#### 3. Ví dụ cơ bản

Ví dụ request từ trình duyệt:

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Chrome
```

Response từ server:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 57

<html><body><h1>Welcome!</h1></body></html>
```

**Giải thích:**

* `GET /index.html` → client yêu cầu tài nguyên `/index.html`.
* `HTTP/1.1 200 OK` → server phản hồi thành công (status code 200).
* `Content-Type` cho biết dữ liệu dạng gì (ở đây là HTML).

### II. Các method trong HTTP

#### 1. Giới thiệu

HTTP định nghĩa **nhiều phương thức (methods)** để client nói cho server biết “muốn làm gì với tài nguyên”.

#### 2. Các method phổ biến

| Method      | Ý nghĩa                                   | Có thân (body)? | Thường dùng cho      |
| ----------- | ----------------------------------------- | --------------- | -------------------- |
| **GET**     | Lấy dữ liệu (Retrieve resource)           | ❌ Không         | Truy vấn dữ liệu     |
| **POST**    | Gửi dữ liệu mới (Create)                  | ✅ Có            | Tạo mới tài nguyên   |
| **PUT**     | Cập nhật toàn bộ (Update entire resource) | ✅ Có            | Cập nhật dữ liệu     |
| **PATCH**   | Cập nhật một phần (Partial update)        | ✅ Có            | Sửa một phần dữ liệu |
| **DELETE**  | Xóa dữ liệu (Delete resource)             | ❌/✅ Có thể có   | Xóa tài nguyên       |
| **HEAD**    | Giống GET nhưng chỉ lấy header            | ❌               | Kiểm tra metadata    |
| **OPTIONS** | Yêu cầu server cho biết các method hỗ trợ | ❌               | Dò API (CORS)        |

#### 3. Ví dụ minh họa (bằng cURL)

```bash
# GET: lấy danh sách người dùng
curl -X GET http://localhost:8080/api/users

# POST: tạo người dùng mới
curl -X POST -H "Content-Type: application/json" \
-d '{"name":"Seamus", "email":"seamus@example.com"}' \
http://localhost:8080/api/users

# PUT: cập nhật toàn bộ thông tin người dùng
curl -X PUT -H "Content-Type: application/json" \
-d '{"id":1,"name":"Seamus Updated","email":"new@example.com"}' \
http://localhost:8080/api/users/1

# DELETE: xóa người dùng
curl -X DELETE http://localhost:8080/api/users/1
```

### III. Response là gì, Request là gì?

#### 1. HTTP Request (Yêu cầu HTTP)

Request là **thông điệp mà client gửi đến server** để yêu cầu dữ liệu hoặc thực hiện hành động.

**Cấu trúc:**

```
[Request Line]
[Headers]
[Body] (tùy chọn)
```

**Ví dụ:**

```
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 45

{"email":"user@example.com", "password":"123456"}
```

**Giải thích:**

* **Request line:** `POST /api/login HTTP/1.1` → phương thức + URL + phiên bản HTTP
* **Headers:** metadata (kiểu dữ liệu, độ dài, user-agent...)
* **Body:** dữ liệu chính gửi lên server (JSON, form data, …)

#### 2. HTTP Response (Phản hồi HTTP)

Response là **thông điệp mà server trả về** để phản hồi lại request từ client.

**Cấu trúc:**

```
[Status Line]
[Headers]
[Body]
```

**Ví dụ:**

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 55

{"message":"Login successful","token":"abc123xyz"}
```

**Giải thích:**

* **Status Line:** `HTTP/1.1 200 OK`

  * 200: mã trạng thái (status code)
  * OK: mô tả kết quả
* **Headers:** thông tin meta (vd: `Content-Type: application/json`)
* **Body:** dữ liệu thực tế (JSON, HTML, v.v.)

#### 3. Phân loại mã trạng thái (Status Codes)

| Nhóm mã | Ý nghĩa         | Ví dụ                                              |
| ------- | --------------- | -------------------------------------------------- |
| 1xx     | Thông tin       | 100 Continue                                       |
| 2xx     | Thành công      | 200 OK, 201 Created                                |
| 3xx     | Chuyển hướng    | 301 Moved Permanently, 302 Found                   |
| 4xx     | Lỗi phía client | 400 Bad Request, 404 Not Found                     |
| 5xx     | Lỗi phía server | 500 Internal Server Error, 503 Service Unavailable |


#### 4. Checklist lỗi thường gặp

* **400 Bad Request:** request body sai định dạng JSON.
* **401 Unauthorized:** thiếu token xác thực.
* **404 Not Found:** gọi sai URL hoặc resource không tồn tại.
* **500 Internal Server Error:** lỗi logic hoặc exception trên server.

---

## B. API là gì, RestAPI là gì?

### I. API là gì? (Application Programming Interface – Giao diện lập trình ứng dụng)

#### 1. Định nghĩa

**API (Application Programming Interface)** là **tập hợp các quy tắc, định dạng và giao thức (rules & formats)** cho phép **các phần mềm khác nhau giao tiếp (communicate)** với nhau.

Nói đơn giản, API giống như **hợp đồng** giữa hai chương trình:

* Một bên cung cấp dữ liệu hoặc dịch vụ (Provider / Server).
* Một bên yêu cầu dữ liệu hoặc dịch vụ đó (Consumer / Client).

#### 2. Ví dụ

**a. Khi mở ứng dụng thời tiết:**
Ứng dụng (client) gọi **Weather API** để lấy thông tin thời tiết từ máy chủ (server).

**b. Khi đăng nhập Facebook bằng tài khoản Google:**
Facebook gửi yêu cầu đến **Google OAuth API** để xác thực danh tính bạn.

#### 3. API không chỉ là Web

* **Library API:** Giao diện giữa các module trong cùng chương trình.
* **OS API:** Giao tiếp với hệ điều hành (vd: Windows API).
* **Hardware API:** Điều khiển phần cứng.
* **Web API:** Giao tiếp qua Internet

### II. REST API là gì? (Representational State Transfer – Truyền tải trạng thái biểu diễn)

#### 1. REST là gì?

REST là **một phong cách kiến trúc (architectural style)** cho việc thiết kế API.
REST không phải là framework hay ngôn ngữ, mà là **tập hợp nguyên tắc (principles)** giúp API dễ sử dụng, mở rộng và tương thích với HTTP.

#### 2. REST hoạt động như thế nào?

REST dựa trên ý tưởng:

> Mỗi tài nguyên (resource) được đại diện bởi một **URL duy nhất**, và được thao tác bằng các **phương thức HTTP (HTTP methods)**.

Ví dụ REST API quản lý người dùng:

| Hành động                | HTTP Method | URL               | Mô tả                       |
| ------------------------ | ----------- | ----------------- | --------------------------- |
| Lấy danh sách người dùng | GET         | `/api/users`      | Trả về danh sách người dùng |
| Lấy 1 người dùng cụ thể  | GET         | `/api/users/{id}` | Trả về người dùng theo id   |
| Tạo người dùng mới       | POST        | `/api/users`      | Thêm người dùng mới         |
| Cập nhật người dùng      | PUT         | `/api/users/{id}` | Cập nhật toàn bộ người dùng |
| Xóa người dùng           | DELETE      | `/api/users/{id}` | Xóa người dùng theo id      |

### III. Các nguyên tắc chính của REST

#### 1. Client–Server (Phân tách máy khách và máy chủ)

Client chỉ lo giao diện, server lo xử lý logic và dữ liệu → dễ mở rộng và tái sử dụng.

#### 2. Stateless (Không lưu trạng thái)

Mỗi request độc lập, server không nhớ thông tin client giữa các lần gọi.
Nếu cần trạng thái, client gửi token hoặc session mỗi lần.

#### 3. Uniform Interface (Giao diện thống nhất)

Các URL phải rõ ràng, thống nhất, ví dụ:

* ✅ `/api/users/1` thay vì ❌ `/api/getUser?id=1`

#### 4. Resource-based (Lấy tài nguyên làm trung tâm)

Tài nguyên có thể là người dùng, sản phẩm, bài viết... được đại diện bằng **danh từ**, không phải **động từ**:

* ✅ `/api/products`
* ❌ `/api/getProducts`

#### 5. Representation (Biểu diễn dữ liệu)

Dữ liệu trả về thường ở dạng **JSON**, ví dụ:

```json
{
  "id": 1,
  "name": "Seamus",
  "email": "seamus@example.com"
}
```

### IV. Ví dụ REST API bằng Spring Boot

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final List<User> users = new ArrayList<>();

    // GET /api/users
    @GetMapping
    public List<User> getAllUsers() {
        return users;
    }

    // POST /api/users
    @PostMapping
    public String createUser(@RequestBody User user) {
        users.add(user);
        return "User created successfully!";
    }

    // GET /api/users/{id}
    @GetMapping("/{id}")
    public User getUser(@PathVariable int id) {
        return users.get(id);
    }

    // DELETE /api/users/{id}
    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable int id) {
        users.remove(id);
        return "User deleted successfully!";
    }
}
```

**Giải thích:**

* `@RestController`: đánh dấu lớp này là REST controller.
* `@RequestMapping`: định nghĩa URL gốc.
* `@GetMapping`, `@PostMapping`, `@DeleteMapping`: ánh xạ các HTTP method.
* `@RequestBody`: lấy dữ liệu JSON gửi từ client.
* `@PathVariable`: lấy biến động từ URL.

---

## C. Design Pattern: DI (Dependency Injection) & IoC (Inversion of Control)

### I. Tổng quan Design Pattern (Mẫu thiết kế)

#### 1. Định nghĩa

**Design Pattern (Mẫu thiết kế)** là **giải pháp tổng quát (general reusable solution)** cho các vấn đề lặp đi lặp lại trong lập trình hướng đối tượng (OOP).
Nó không phải là code cụ thể, mà là **ý tưởng tổ chức và tái sử dụng mã nguồn** một cách hiệu quả.

#### 2. Vai trò của Design Pattern

* Giúp code **dễ bảo trì (maintainable)** và **mở rộng (scalable)**.
* Giảm **sự phụ thuộc (dependency)** giữa các lớp.
* Làm code “nói cùng ngôn ngữ” – ai cũng dễ hiểu khi đọc.

Spring Framework được xây dựng dựa trên nhiều design pattern, nhưng hai mẫu “trụ cột” là **IoC (Inversion of Control)** và **DI (Dependency Injection)**.

### II. IoC – Inversion of Control (Đảo ngược quyền kiểm soát)

#### 1. Ý tưởng

Trong lập trình Java thông thường, lập trình viên **tự khởi tạo** đối tượng:

```java
public class StudentService {
    private StudentRepository repository = new StudentRepository();
}
```

Ở đây, lớp `StudentService` **tự tạo (new)** `StudentRepository`.
→ Điều này khiến hai lớp phụ thuộc chặt vào nhau, khó test, khó thay đổi.

**IoC (Inversion of Control)** có nghĩa là:

> “Đảo ngược quyền tạo và quản lý đối tượng từ developer sang framework.”

Framework (ở đây là **Spring Container**) sẽ chịu trách nhiệm **khởi tạo, quản lý, tiêm (inject)** đối tượng khi cần.

#### 2. IoC Container trong Spring

Spring cung cấp hai container chính để quản lý bean:

* **BeanFactory:** nhẹ, chỉ tạo bean khi cần (lazy).
* **ApplicationContext:** mở rộng BeanFactory, hỗ trợ dependency injection, AOP, events...

Khi ứng dụng Spring Boot khởi chạy, container sẽ:

1. Quét các package có annotation (`@Component`, `@Service`, `@Repository`, …).
2. Tạo và quản lý các đối tượng đó (bean).
3. Tiêm chúng vào các lớp cần dùng.

### III. DI – Dependency Injection (Tiêm phụ thuộc)

#### 1. Khái niệm

**Dependency Injection (DI)** là kỹ thuật giúp một lớp **nhận dependency (phụ thuộc)** từ bên ngoài thay vì tự tạo.
IoC là nguyên lý, còn **DI là cách hiện thực nguyên lý đó**.

#### 2. Các loại DI trong Spring

a. **Constructor Injection** (qua hàm tạo – khuyến khích dùng)

```java
@Service
public class BookService {
    private final BookRepository bookRepository;

    // Spring tự tiêm bean BookRepository vào đây
    public BookService(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

b. **Setter Injection** (qua setter)

```java
@Service
public class BookService {
    private BookRepository bookRepository;

    @Autowired
    public void setBookRepository(BookRepository bookRepository) {
        this.bookRepository = bookRepository;
    }
}
```

c. **Field Injection** (trực tiếp trong biến – không khuyến khích trong Spring Boot hiện đại)

```java
@Service
public class BookService {
    @Autowired
    private BookRepository bookRepository;
}
```

### IV. Ví dụ DI & IoC trong Spring Boot

```java
// BookRepository.java
@Repository
public class BookRepository {
    public List<String> findAll() {
        return List.of("Clean Code", "Effective Java", "Spring in Action");
    }
}

// BookService.java
@Service
public class BookService {
    private final BookRepository repository;

    // Constructor Injection
    public BookService(BookRepository repository) {
        this.repository = repository;
    }

    public List<String> getBooks() {
        return repository.findAll();
    }
}

// BookController.java
@RestController
@RequestMapping("/api/books")
public class BookController {
    private final BookService service;

    public BookController(BookService service) {
        this.service = service;
    }

    @GetMapping
    public List<String> getAllBooks() {
        return service.getBooks();
    }
}

// DemoApplication.java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

**Giải thích:**

* `@Repository`, `@Service`, `@RestController` → đều là **Spring Bean**, được IoC container quản lý.
* Spring **tự tiêm (inject)** `BookRepository` vào `BookService`, và `BookService` vào `BookController`.
* Bạn không cần `new BookService()` — framework tự lo toàn bộ quá trình khởi tạo.

### V. Hậu trường của IoC Container

Khi chạy ứng dụng, Spring Boot thực hiện các bước sau:

1. Quét toàn bộ package của lớp chính (`@SpringBootApplication`).
2. Tìm các annotation như `@Component`, `@Service`, `@Repository`, `@Controller`.
3. Tạo đối tượng tương ứng (bean) và lưu trong **ApplicationContext**.
4. Khi cần bean khác, Spring tự động tiêm vào thông qua constructor hoặc setter.

Nếu bạn gọi:

```java
ApplicationContext context = SpringApplication.run(DemoApplication.class, args);
BookService service = context.getBean(BookService.class);
```

Spring sẽ trả lại **bean sẵn có** thay vì tạo mới.
