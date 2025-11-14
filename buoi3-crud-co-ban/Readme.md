# Buổi 3: CRUD Cơ Bản

## A. Spring Boot xử lý request trong Controller

## 1. Controller là gì, hoạt động ra sao?

### 1.1. Controller là gì?

Như đã nói ở các bài trước, Controller trong ứng dụng Spring Boot là nơi tiếp nhận request và trả về response cho client. Có thể hiểu controller chính là lớp trung gian giữa server của bạn và bên ngoài.

Về mặt code, Controller chỉ đơn thuần là một bean được đánh dấu với `@Controller` hoặc `@RestController`.

Trong Spring Boot, có hai dạng Controller, tương ứng hai annotation trên:

* `@Controller` có thể trả về View qua một String hoặc JSON data trong response body (nếu được chỉ định). Thích hợp cho các controller có routing, chuyển trang các kiểu.
* `@RestController` chỉ có thể trả về data trong response body. Thích hợp cho các controller để cung cấp API.

Do đó, ta có thể nói `@RestController` = `@Controller` + `@ResponseBody`.

### 1.2. Code ví dụ

Bên dưới là cấu trúc một controller nhé.

```java
@Controller
public class HomeController {
    // Bên trong controller sẽ có nhiều method, mỗi cái sẽ bắt request cụ thể
    
    // Bắt GET /home request và trả về view
    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("name", "John");
        return "index";  // Return tên của View, model sẽ tự động pass vào view
    }
    
    // Hoặc có thể trả về data trong response body (như các API)
    @GetMapping("/users")
    @ResponseBody
    public List<User> getUserList() {
        return new ArrayList<>();
    }
    
    // Hoặc cái này tương tự như trên, nhưng có thể tùy chỉnh response status code, header,...
    @GetMapping("/users/{id}")
    public ResponseEntity<User> getUserById(@PathVariable("id") String userId) {
        // Không cần @ResponseBody do có body rồi
        return ResponseEntity.status(200).body(new User());
    }
}
```

### 1.3. Các hoạt động của controller

![img](https://images.viblo.asia/8e1d82ef-0047-4ab6-a381-5dd281191268.png)

Như hình trên, khi client gửi một request tới server Spring Boot của mình, thì nó sẽ đi qua thứ gọi là Front controller trước. Đây là controller có sẵn, nó có tác dụng sau:

* Phân giải request, tìm coi request gọi tới method nào của controller nào để gọi đúng tới đó
* Các data của request sẽ được parse ra và mapping tương ứng vào các tham số controller method (có `@RequestParam`, `@PathVariable`, `@Header`,... tương ứng).
* Đặc biệt, Spring MVC có thể parse được các data phức tạp như enum, List hay object. Ví dụ enum trong request là dạng string, vẫn sẽ được parse đúng thành enum.
* Nếu data không thể parse được, front controller sẽ trả về bad request (hoặc có cơ chế khác để chúng ta ghi đè lại việc này).

Với chiều ngược lại cũng tương tự như vậy. Dữ liệu trả về từ controller sẽ được build thành response và trả cho client.

## 2. Controller mapping

### 2.1. Các loại HTTP request

Bạn nào học về web hẳn đã rõ về khái niệm HTTP request. Mình sẽ không nói sâu về phần này, nhưng tạm hiểu mỗi HTTP request sẽ gồm 2 thông tin quan trọng:

* Request tới URL nào (request tới đâu)
* HTTP method là gì (thể hiện hành động gì đấy với URL)

Trong controller, chỉ cần nắm được hai thông tin trên thì sẽ bắt được mọi request được gửi tới, sau đó mới xử lý tiếp.

Trong Rest API design, thì người ta thường dùng **danh từ** trong URL để chỉ đối tượng được tác động. Còn các HTTP method để đại diện cho **hành động** nào sẽ áp dụng lên đối tượng đó.

Ví dụ như:

* Request tới `GET /users` có đối tượng tác động là `users` (tất cả user), và hành động là `GET` (lấy thông tin)
* Request tới `PUT /users/123` có đối tượng là `users/123` (user có mã là 123) và hành động là `PUT` (cập nhật thông tin)

Thường thì theo khuyến nghị người ta sử dụng đúng HTTP method với các hành động CRUD tương ứng:

* Create: dùng POST method
* Read: dùng GET method
* Update: dùng PUT method
* Delete: dùng DELETE method

Hầu hết các ứng dụng web đều sử dụng 4 hành động CRUD cơ bản trên tới hơn 2/3 rồi. Ngoài ra có thể có các hành động khác mà không có method tương ứng, như login thì có thể thêm vào endpoint như `POST /login` (dùng POST sẽ an toàn hơn, đọc thêm về các HTTP method để hiểu rõ hơn ý nghĩa của chúng).

### 2.2. Bắt các request

Spring Boot dùng các annotation sau, đánh dấu lên từng **method** của controller, để chỉ định rằng khi HTTP method tương ứng gọi tới thì method sẽ được thực thi.

```java
@RestController
public class UserController {
    @GetMapping("/users")
    public ResponseEntity<?> getAllUsers() {}
    
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable("id") int id) {}
}
```

Ví dụ trên có 2 method, bắt tương ứng hai request là `GET /users` và `DELETE /users/{id}`. Khi có request tương ứng gửi tới, thì hai method trên sẽ thực thi và trả về kết quả cho client.

Các annotation phổ biến như `@GetMapping`, `@PostMapping`, `@PutMapping`,... có dạng là tên HTTP method cộng với từ "mapping". Ngoài ra còn có thể dùng `@RequestMapping` và chỉ định thuộc tính `method` như sau.

```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
```

Ngoài ra, `@RequestMapping` còn có thể dùng bên trên class controller, để chỉ định endpoint gốc cho toàn bộ method bên trong nó. Ví dụ như sau.

```java
@RestController
@RequestMapping("/users")
public class UserController {
    // Kết hợp với route gốc ở trên, ta có /users/info
    @GetMapping("/info")
}
```

## 3. Nhận request data trong Controller

Controller nhận dữ liệu từ request, tùy vào dữ liệu nằm ở đâu mà chúng ta có cách lấy ra khác nhau:

* Request param (query string)
* Path variable
* Request body
* Header

### 3.1. Request param (query string)

Ví dụ như request sau `GET /users?age=18&name=Dũng` thì chúng ta có 2 request param là `age = 18` và `name = Dũng`. Khi đó, muốn lấy được hai giá trị trên chúng ta dùng `@RequestParam` như sau.

```java
@RestController
public class UserController {
    ...
    @GetMapping("/users")
    public ResponseEntity<?> getAllUsers(
        @RequestParam("age") int age,
        @RequestParam("name") String name) {
        // Lúc này hai biến age và name đã có dữ liệu tương ứng
    }
}
```

Trong trường hợp `@RequestParam` có thêm các tham số khác, thì chúng ta phải viết như sau. Ví dụ cả hai trường `age` và `name` trên đều là optional, không bắt buộc, nên chúng ta dùng thuộc tính `required = false` cho `@RequestParam` (mặc định là true).

```java
@RestController
public class UserController {
    ...
    @GetMapping("/users")
    public ResponseEntity<?> getAllUsers(
        @RequestParam(value = "age", required = false) Integer age,
        @RequestParam(value = "name", required = false) String name) {
        // Lúc này hai biến age và name đã có dữ liệu tương ứng
    }
}
```

Lúc này, do biến `age` có thể không có, nên phải cho nó kiểu `Integer` có giá trị null để biết được `age` có được gửi lên hay không. Nếu là kiểu primitive thì nó sẽ luôn có giá trị mặc định.

Ngoài ra `@RequestParam` cũng có thuộc tính `defaultValue`, nếu request không chỉ định thì giá trị default đó sẽ được sử dụng.

### 3.2 Path variable

Path variable là một phần trong đường dẫn URL, ví dụ `GET /users/123/info` thì `123` là path variable. Sử dụng `@PathVariable` để làm việc này, tương tự như dùng `@RequestParam`.

```java
@GetMapping("/users/{id}/info")
public ResponseEntity<?> getUserInfo(
    @PathVariable(value = "id", defaultValue = "0") int userId) {
    // id là tên path variable, tương ứng trên url {id}
}
```

`@PathVariable` cũng có các thuộc tính tương tự `@RequestParam`.

### 3.3. Request body

Request method PUT, POST mới có request body, đây là nơi chứa data chính để gửi lên. Thường thì request body sẽ ở dạng JSON hoặc form-data, khi vào controller sẽ được tự động parse ra thành Object (ví dụ DTO chẳng hạn).

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginDto loginDto) {
    // Dữ liệu trong request body có thể là JSON, form-data,...
    // Tuy nhiên khi vào controller sẽ bị parse thành object hết
}
```

Đây là ví dụ về class `LoginDto` ở trên (có dùng lombok).

```java
@Getter
@Setter
public class LoginDto {
    private String username;
    private String password;
}
```

### 3.4. Header

```java
@PostMapping("/login")
public ResponseEntity<?> login(@Header("Authorization") String authHeader) {
    // Biến authHeader sẽ có giá trị là giá trị của Authorization header
}
```

Ví dụ như trên mình muốn thực hiện xác thực người dùng bằng Basic authentication. Thông tin username, password được encode trong header có tên là Authorization. Muốn lấy được value trong header, thì các bạn sử dụng `@Header` như trên.

---

## B. Luồng đi trong Spring Boot

## 1. Hai mô hình quen thuộc

Cấu trúc source code của Spring Boot được dựa trên hai mô hình là **mô hình MVC** và **mô hình 3 lớp**.

### 1.1. Mô hình ba lớp (three tier)

Đây là mô hình tổ chức source code rất phổ biến trong Spring Boot. Cụ thể, ứng dụng được chia làm 3 tầng (tier hoặc layer) như sau:

* **Presentation layer:** tầng này tương tác với người dùng, bằng View, Controller (trong MVC) hoặc API (nếu có).
* **Business logic layer:** Chứa toàn bộ logic của chương trình, các đa số code nằm ở đây
* **Data access layer:** Tương tác với database, trả về kết quả cho tầng business logic

Trong Spring Boot, thì có một số thành phần đại diện cho từng lớp:

* **Service:** chứa các business logic code
* **Repository:** đại diện cho tầng data access

Còn presentation layer thì chúng ta sẽ bàn tiếp qua mô hình MVC ngay bên dưới.

![img](https://images.viblo.asia/fdbe3b44-aa91-4a88-9202-814c56ef9178.png)

### 1.2. Mô hình MVC

Do Spring Boot chỉ là wrapper cho Spring, chúng ta vẫn sử dụng ngầm các module Spring khác bên dưới, ví dụ như Spring MVC. Và tất nhiên khi dùng Spring MVC thì sẽ tuân theo mô hình MVC

Cụ thể, nó chia presentation layer làm 3 phần:

* **Model:** các cấu trúc dữ liệu của toàn chương trình, có thể đại diện cho trạng thái của ứng dụng
* **View**: lớp giao diện, dùng để hiển thị dữ liệu ra cho user xem và tương tác
* **Controller:** kết nối giữa Model và View, điều khiển dòng dữ liệu

Dữ liệu từ Model qua Controller sau đó được gửi cho View hiển thị ra. Và ngược lại, khi có yêu cầu mới từ View, thì sẽ qua Controller thực hiện thay đổi dữ liệu của Model.

Tuy nhiên, MVC chỉ mô tả luồng đi của dữ liệu, nó không nói rõ như code đặt ở đâu (ở Model, View hay Controller), rồi lưu trữ Model vào database kiểu gì,... Do đó, đối với ứng dụng hoàn chỉnh như Spring Boot thì cần kết hợp cả mô hình MVC và 3-tier lại với nhau.

## 2. Áp dụng vào Spring Boot thì như nào?

### 2.1. Các thành phần quan trọng

Kết hợp hai mô hình lại, chúng ta có được ứng dụng Spring Boot hoàn chỉnh, gồm các thành phần sau:

* Controller: trả về View (có chứa data sẵn, dạng trang HTML), hoặc Model thể hiện dưới dạng API cho View (View viết riêng bằng React, Vue, hoặc Angular).
* Service: chứa các code tính toán, xử lý. Khi Controller yêu cầu, thì Service tương ứng sẽ tiếp nhận và cho ra dữ liệu trả cho Controller (trả về Model). Controller sẽ gửi về View như trên.
* Repository: Service còn có thể tương tác với service khác, hoặc dùng Repository để gọi DB. Repository là nơi trực tiếp tương tác, đọc ghi dữ liệu trong DB và trả cho service.

Ơ thế còn Model và View thì đi đâu? Mình sẽ giải thích như sau:

* Model chỉ đơn giản là các đối tượng được Service tính toán xong trả về cho Controller.
* View thì có 2 loại, một là dạng truyền thống là trả về 1 cục HTML có data rồi. Lúc này Controller sẽ pass dữ liệu vào View và return về (Spring MVC có JSP hoặc template engine như Thymeleaf làm điều đó).
* View dạng 2 là dạng View tách riêng (không thuộc về project Spring boot). Thường có trong các hệ thống dùng API. View sẽ được viết riêng bằng React, Angular,... Controller sẽ đưa dữ liệu Model thông qua API cho View, và cũng nhận lại các yêu cầu qua API.

### 2.2. Sơ đồ luồng đi

Để hiểu rõ hơn về các thành phần trong Spring Boot, chúng ta hãy xem qua sơ đồ luồng đi và sự tương tác giữa chúng

![img](https://images.viblo.asia/68d13e98-8714-4dd9-ae27-641ee729ab20.png)

Sơ đồ trên mình sẽ xét theo chiều kim đồng hồ nhé:

* Đầu tiên, user sẽ vào View để xem, tương tác
* Khi user bắt đầu load dữ liệu (ví dụ click nút Reload), thì 1 request từ View gửi cho Controller (kiểu như "ê, cho tao cái danh sách user với")
* Controller nhận được yêu cầu, bắt đầu đi hỏi ông Service (trong code là gọi method của Service)
* Service nhận được yêu cầu từ Controller, đối với các code đơn giản có thể tính toán và trả về luôn. Nhưng các thao tác cần đụng tới database thì Service phải gọi Repository để lấy dữ liệu trong DB
* Repository nhận được yêu cầu từ Service, sẽ thao tác với DB. Data lấy ra trong DB được hệ thống ORM (như JPA hoặc Hibernate) mapping thành các object (trong Java). Các object này gọi là Entity.

Và bây giờ sẽ là đi ngược lại trả về cho user:

* Service nhận các Entity được Repository trả về, biến đổi nó. Biến đổi ở đây là có thể thực hiện tính toán, thêm bớt các field,... và cuối cùng biến Entity thành Model. Model sẽ được trả lại cho Controller.
* Controller nhận được Model, nó sẽ return cho View. Có 2 cách, một là dùng template engine pass dữ liệu Model vào trang HTML, rồi trả về cục HTML (đã có data) cho client. Cách 2 là gửi qua API, View tự parse ra và hiển thị tương ứng (hiển thị thế nào tùy View).
* Khi View hiển thị xong, user sẽ thấy danh sách user hiện lên trang web.

Một số mẹo hay để tổ chức luồng đi cho tốt:

* Giữ cho Controller càng ít code càng tốt. Vì Controller chỉ là trung gian kết nối thông, nên không nên chứa nhiều code, thay vào đó nên bỏ vào Service.
* Nên tách bạch Service rõ ràng. Không nên cho 1 service thực hiện nhiều công việc, nên tách ra nhiều Service.

### 2.3. Code ví dụ

Các thành phần trên có thể tương tác, gọi lẫn nhau, đó không gì khác ngoài cơ chế dependency injection trong Spring. Cụ thể như sau:

* Trong Controller được inject các Service cần thiết vào
* Trong Service được inject các Repository cần thiết vào

Mình sẽ code một ví dụ đơn giản nhé, là hiển thị danh sách User trong DB. Code ngoài file mặc định ra thì sẽ có

```java
@RestController  // @RestController dùng cho API, còn return View HTML thì dùng @Controller
@RequestMapping("/")  // Endpoint gốc là /
public class UserController {
    // Inject Service vào để gọi được
    @Autowired  // Dùng cách này cho ngắn, chứ thường là inject qua constructor
    private final UserService userService;

    @GetMapping("/")
    // Trả về Model là một List<UserModel>
    public ResponseEntity<List<UserModel>> getUserList() {
        // Service trả về Model (là List<UserModel>) nên có thể return thẳng luôn
        return userService.getUserList();
    }

    // Có thể có các endpoint khác
}
```

```java
@Service  // Đánh dấu class này là Service
public class UserService {
    // Inject Repository vào
    @Autowired  // Dùng cách này cho ngắn, chứ thường là inject qua constructor
    private final UserRepository userRepository;

    public List<UserModel> getUserList() {
        // Lấy ra từ Repository dạng List<User> là Entity
        List<User> users = userRepository.findAll();

        // Biển đổi List<Entity> thành List<Model>
        List<UserModel> userModels = users.stream()
            .map(UserModel::new)
            .collect(Collector.toList());

        // Hoặc cách thủ công
        List<UserModel> userModels = new ArrayList<>();
        for (User user: users)
            userModels.add(new UserModel(user));

        return userModels;  // Code khá nhiều so với Controller
    }
}
```

```java
// @Repository  // Không dùng @Repository, vì annotation này chỉ dùng trên class
// Đây là ORM của MongoDB, nó sẽ mapping data trong bảng users thành dạng object User
public interface UserRepository extends MongoRepository<User, String> {
    // Không định nghĩa thêm method nào, chỉ sử dụng .findAll() có sẵn
}
```

```java
// Không có hậu tố là mặc định class là Entity nhé
public class User {
    // Để public hết cho đơn giản, bình thường sẽ là private và dùng getter, setter
    public String name;
    public Integer age;
    public String password;  // Đã được hash bcrypt
}
```

```java
// Có hậu tố Model là Model
public class UserModel {
    public String name;
    public Integer age;

    // Copy thông qua constructor
    public UserModel(User entity) {
        this.name = entity.name;
        this.age = entity.age;
        // Không gán password
    }
}
```

## B. JPA & Hibernate

## 1.JPA
### 1.1 JPA là gì?
JPA (viết tắt của Java Persistence API) là một tiêu chuẩn kỹ thuật trong lĩnh vực phát triển ứng dụng Java để lập trình đối tượng quan hệ (ORM). JPA được cung cấp bởi Oracle và đã được thừa nhận là một tiêu chuẩn quốc tế.

JPA cung cấp cách tiếp cận trừu tượng cho việc lưu trữ và truy xuất dữ liệu trong cơ sở dữ liệu quan hệ. Sử dụng JPA, chúng ta có thể tập trung vào thiết kế ứng dụng của mình hơn là phải lo về cách lưu trữ và truy xuất dữ liệu như thế nào.
### 1.2 Sự cần thiết của JPA trong ứng dụng Java
Trong quá trình phát triển ứng dụng Java, chúng ta không thể tránh khỏi việc tương tác với cơ sở dữ liệu. Vấn đề là, làm thế nào để lưu trữ và truy xuất dữ liệu một cách dễ dàng và hiệu quả?

Một giải pháp là sử dụng **JDBC (Java Database Connectivity)**, một API được cung cấp bởi Sun Microsystems để tương tác với cơ sở dữ liệu. Tuy nhiên, với JDBC, chúng ta cần phải viết rất nhiều code để thực hiện các hoạt động đơn giản như việc lấy và cập nhật dữ liệu. Điều này có thể làm cho ứng dụng của chúng ta trở nên phức tạp và khó bảo trì.

Vì vậy, JPA ra đời nhằm giải quyết vấn đề này bằng cách giảm bớt sự phức tạp của việc tương tác với cơ sở dữ liệu. Với JPA, ta có thể sử dụng đối tượng Java để lưu trữ và truy xuất dữ liệu. Hơn nữa, JPA sẽ tự động chuyển đổi các đối tượng thành các câu truy vấn SQL để thực hiện việc lưu trữ và truy xuất, giúp lập trình viên chúng ta tập trung vào việc phát triển tính năng của ứng dụng mà không phải lo nhiều về SQL phức tạp.
### 1.3 Vai trò của JPA trong phát triển ứng dụng
JPA là một công nghệ quan trọng trong việc phát triển ứng dụng Java, bởi vì nó giúp:

- **Viết code ít hơn**: Sử dụng JPA, chúng ta có thể sử dụng các đối tượng Java thay vì phải viết các câu truy vấn SQL hoặc sử dụng JDBC trực tiếp. Điều này làm cho code trở nên ngắn gọn hơn và dễ đọc hơn, giảm thiểu tối đa boilerplate code.
- **Dễ bảo trì**: Với JPA, việc thay đổi cấu trúc cơ sở dữ liệu trở nên dễ dàng hơn. Chúng ta chỉ cần thay đổi định nghĩa của đối tượng Java và JPA sẽ tự động áp dụng các thay đổi này vào cơ sở dữ liệu tương ứng.
- **Hiệu suất cao**: JPA giúp tối ưu hóa việc truy xuất và lưu trữ dữ liệu trong cơ sở dữ liệu. Nó sử dụng các kỹ thuật như lazy loading và caching để giảm thiểu số lần truy cập vào cơ sở dữ liệu.
## 2. ORM là gì
### 2.1 Một số ORM framework hỗ trợ JPA
**ORM (Object-Relational Mapping)** là một kỹ thuật trong lập trình để ánh xạ các đối tượng Java vào cơ sở dữ liệu quan hệ. Khi sử dụng ORM, chúng ta có thể làm việc với cơ sở dữ liệu thông qua các đối tượng Java, thay vì phải sử dụng các câu truy vấn SQL trực tiếp.
![alt text](images/orm.png)
Có nhiều framework hỗ trợ ORM và JPA được liệt kê phía dưới:

- **Hibernate**: Hibernate là một trong những framework ORM phổ biến nhất và cũng là backend mặc định cho JPA. Nó cung cấp các tính năng mạnh mẽ và linh hoạt cho việc tương tác với cơ sở dữ liệu quan hệ.
- **EclipseLink**: EclipseLink là một framework ORM mạnh mẽ khác được sử dụng rộng rãi trong cộng đồng Java. Nó cũng hỗ trợ đầy đủ JPA.
- **OpenJPA**: OpenJPA là một framework ORM được phát triển bởi Apache. Nó cung cấp một loạt các tính năng để làm việc với cơ sở dữ liệu quan hệ và hỗ trợ đầy đủ JPA.
## 3. Java Persistence API cơ bản
### 3.1 Kiến trúc JPA

Để hiểu rõ cách JPA hoạt động, chúng ta cần tìm hiểu về một số thành phần chính của nó.

### Các thành phần cơ bản trong JPA

#### 3.1.1 Định nghĩa Entity

Trong JPA, một **Entity** là một đối tượng Java ánh xạ vào một bảng trong cơ sở dữ liệu. Chúng ta có thể hiểu là mối quan hệ 1-1. Đối tượng Entity chứa thông tin về cấu trúc và dữ liệu của bảng.

Để định nghĩa một Entity trong JPA, chúng ta có thể sử dụng các annotation như `@Entity`, `@Table`, `@Column`, vv.

#### Ví dụ

```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name")
    private String name;
    
    // getters and setters
}
```

Ở ví dụ trên, `Employee` được đánh dấu là một Entity bằng cách sử dụng annotation `@Entity`.
Sẽ ánh xạ vào bảng `"employees"` trong cơ sở dữ liệu.
Thuộc tính `id` được đánh dấu là khóa chính bằng cách sử dụng annotation `@Id`.
Các thuộc tính khác được ánh xạ vào các cột của bảng bằng cách sử dụng annotation `@Column`.

## 4. Tại sao nên dùng JPA

### 4.1 No-code Repository

Một trong những lợi ích lớn nhất của JPA là chúng ta **không cần phải viết mã SQL hay DAO (Data Access Object)** để làm việc với cơ sở dữ liệu.
Thay vào đó, JPA cung cấp **Repository Interfaces** để thực hiện các hoạt động CRUD.

**JPA cung cấp các phương thức thao tác với cơ sở dữ liệu**

#### Ví dụ

```java
@Repository
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    List<Employee> findAll();
}
```

Điều này giúp giảm thiểu rất nhiều công việc lặp lại và **mã boilerplate code** mà chúng ta thường phải viết khi sử dụng JDBC trực tiếp.



### 4.2 Khả năng mở rộng

Với JPA, chúng ta có thể **thay đổi cơ sở dữ liệu mà không cần sửa đổi mã nguồn ứng dụng**.
Điều này làm cho việc di động giữa các hệ thống cơ sở dữ liệu khác nhau dễ dàng hơn.

Ngoài ra, JPA còn cho phép chúng ta mở rộng ứng dụng dễ dàng bằng cách **thêm mới các Entity và Repository** mà không cần phải sửa đổi mã nguồn đã có.



## 5. So sánh JPA và Hibernate


JPA và Hibernate là hai khái niệm liên quan đến **ánh xạ quan hệ đối tượng (ORM)** trong Java.
Với ORM, như đã đề cập ở phía trên, là một kỹ thuật cho phép **chuyển đổi các đối tượng Java thành các bản ghi trong cơ sở dữ liệu quan hệ và ngược lại**.

Từ đó, chúng ta có thể thấy được **JPA** và **Hibernate** có một số điểm khác nhau như sau:

| Tiêu chí               | JPA                                            | Hibernate                                                      |
| - | - | -- |
| **Bản chất**           | Là một tiêu chuẩn (Specification)              | Là một triển khai (Implementation) của JPA                     |
| **Ngôn ngữ truy vấn**  | JPQL (Java Persistence Query Language)         | HQL (Hibernate Query Language)                                 |
| **Tính mở rộng**       | Tuân thủ tiêu chuẩn, cung cấp tính năng cơ bản | Mở rộng tiêu chuẩn JPA, cung cấp thêm nhiều tính năng nâng cao |
| **Bộ nhớ đệm (Cache)** | Hỗ trợ 2 loại: Entity Cache, Query Cache       | Hỗ trợ thêm Collection Cache                                   |
| **Tính tương thích**   | Được hỗ trợ bởi tất cả các nhà cung cấp JPA    | Chỉ hỗ trợ riêng cho Hibernate                                 |

Tóm lại, **JPA** và **Hibernate** là hai khái niệm liên quan nhưng khác nhau về ORM trong Java:

* JPA là **một tiêu chuẩn cho ORM**.
* Hibernate là **một triển khai của JPA**, đồng thời mở rộng JPA với nhiều tính năng mạnh mẽ hơn.

Tuy nhiên, việc sử dụng các tính năng dành riêng cho Hibernate có thể gây ra **sự phụ thuộc vào nhà cung cấp** và **khó chuyển đổi** sang các triển khai JPA khác.
Do đó, chúng ta nên cân nhắc kỹ trước khi sử dụng Hibernate hoặc JPA cho dự án của mình.

## 6. Hibernate

### 6.1 Hibernate là gì?

**Hibernate** là một thư viện **ORM (Object Relational Mapping)** mã nguồn mở giúp lập trình viên viết ứng dụng Java có thể **map các objects (POJO)** với **hệ quản trị cơ sở dữ liệu quan hệ**,  
và hỗ trợ thực hiện các khái niệm **lập trình hướng đối tượng** với **cơ sở dữ liệu quan hệ**.


### 6.2 Hibernate 

#### 6.2.1 Persistence Object
Chính là các **POJO object** map với các **table** tương ứng của cơ sở dữ liệu quan hệ.  
Nó như là những “thùng xe” chứa dữ liệu từ ứng dụng để ghi xuống database, hay chứa dữ liệu tải lên ứng dụng từ database.

#### 6.2.2 Session Factory
Là một **interface** giúp tạo ra **session** kết nối đến database bằng cách đọc các cấu hình trong Hibernate configuration.  
Mỗi một database phải có một **session factory**.

Ví dụ:  
Nếu ta sử dụng **MySQL** và **Oracle** cho ứng dụng Java của mình thì ta cần có một `SessionFactory` cho MySQL, và một `SessionFactory` cho Oracle.

#### 6.2.3 Hibernate Session
Mỗi một đối tượng `Session` được `SessionFactory` tạo ra sẽ tạo **một kết nối** đến database.

#### 6.2.4 Transaction
Là **transaction** đảm bảo tính toàn vẹn của phiên làm việc với cơ sở dữ liệu.  
Tức là nếu có một lỗi xảy ra trong transaction thì tất cả các tác vụ thực hiện sẽ thất bại.

#### 6.2.5 Query
Hibernate cung cấp các câu truy vấn **HQL (Hibernate Query Language)** tới database và map kết quả trả về với đối tượng tương ứng của ứng dụng Java.




### 6.3 Tại sao phải dùng Hibernate thay JDBC

#### 6.3.1 Object Mapping
Với **JDBC** ta phải map các trường trong bảng với các thuộc tính của Java object một cách **“thủ công”**.  
Với **Hibernate**, việc này được hỗ trợ **“tự động”** thông qua các file cấu hình map XML hoặc sử dụng **annotation**.

#### JDBC sẽ map Java object với table như sau:

```java
// rs là ResultSet trả về từ câu query get dữ liệu bảng user.
List users = new ArrayList();
while (rs.next()) {
     User user = new User();
     user.setUserId(rs.getString("userNo"));
     user.setName(rs.getString("firstName"));
     user.setEmail(rs.getString("lastName"));
     users.add(user);
}
```

####  Cũng với table user đó, sử dụng annotation để Hibernate map “tự động”:

```java
@Entity
@Table(name = "user")
public class UserModel {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private BigInteger userNo;

    @NotEmpty
    @Column(name = "lastName")
    private String email;

    public BigInteger getUserNo() {
        return this.userNo;
    }
    public void setUserNo(BigInteger userNo) {
        this.userNo = userNo;
    }
    public String getLastName() {
        return lastName;
    }
    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
```



#### 6.3.2 HQL

Hibernate cung cấp các câu lệnh truy vấn tương tự SQL.
HQL của Hibernate hỗ trợ đầy đủ các truy vấn đa hình: **kế thừa (inheritance)**, **đa hình (polymorphism)**, và **liên kết (association)**.



#### 6.3.3 Database Independent

Code sử dụng Hibernate là **độc lập với hệ quản trị cơ sở dữ liệu**,
nghĩa là ta không cần thay đổi câu lệnh HQL khi chuyển từ MySQL sang Oracle hay các hệ quản trị khác.
Chỉ cần thay đổi thông tin cấu hình trong file cấu hình.

```properties
# used MySQL
com.mysql.jdbc.Driver

# used Oracle
oracle.jdbc.driver.OracleDriver
```

#### Ví dụ:

Khi muốn lấy 10 bản ghi dữ liệu của một table từ 2 CSDL khác nhau:

**Với JDBC:**

```sql
# MySQL
SELECT column_name FROM table_name ORDER BY column_name ASC LIMIT 10; 

# SQL Server 
SELECT TOP 10 column_name FROM table_name ORDER BY column_name ASC;
```

**Với Hibernate:**

```java
session.createQuery("SELECT E.id FROM Employee E ORDER BY E.id ASC")
       .setMaxResults(10)
       .list();
```

Câu truy vấn **không thay đổi** dù bạn dùng MySQL hay SQL Server.



#### 6.3.4 Minimize Code Changes

Khi ta thay đổi (thêm) cột vào bảng:

**Với JDBC, ta phải thay đổi:**

* Thêm thuộc tính vào POJO class.
* Thay đổi method chứa câu truy vấn “select”, “insert”, “update” để bổ sung cột mới.
* Có thể có rất nhiều method, nhiều class chứa các câu truy vấn như trên.

**Với Hibernate, ta chỉ cần:**

* Thêm thuộc tính vào POJO class.
* Cập nhật Hibernate XML mapping file để thêm map column – property.

→ Chỉ thay đổi **2 file** là xong.



#### 6.3.5 Lazy Loading

Với những ứng dụng Java làm việc với cơ sở dữ liệu lớn hàng trăm triệu bản ghi, việc sử dụng **Lazy Loading** mang lại lợi ích rất lớn.
Nó giống như việc ta có thể **bẻ từng chiếc đũa** của bó đũa to thay vì bẻ cả bó cùng lúc.

Ví dụ:
Bảng `user` có quan hệ **một-nhiều** với bảng `document`.
Nếu dữ liệu document rất lớn, việc load toàn bộ cùng lúc sẽ tốn bộ nhớ.
Hibernate cho phép ta **chỉ tải dữ liệu khi cần**:

```java
@OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
private Set documents = new HashSet();

// Lấy user
User user = (User) session.get(User.class, new Integer(100));

// Lúc này Hibernate mới fetch documents cho user 100
documents = user.getDocuments();
```



#### 6.3.6 Loại bỏ Try-Catch Blocks

Khi thao tác với JDBC, nếu lỗi xảy ra sẽ có `SQLException` bắn ra, nên ta phải dùng `try-catch` để xử lý.
Hibernate xử lý việc này bằng cách **override toàn bộ JDBC exception** thành **Unchecked Exception**,
vì vậy ta **không cần viết try-catch** trong code nữa.



#### 6.3.7 Quản lý Commit/Rollback Transaction

**Transaction** là nhóm các hoạt động với database của một tác vụ.
Nếu một hoạt động thất bại thì toàn bộ tác vụ cũng thất bại.

**Với JDBC:**

* Lập trình viên phải chủ động gọi `commit` hoặc `rollback`.

**Với Hibernate:**

* Hibernate **tự động quản lý transaction**, giúp đảm bảo tính toàn vẹn dữ liệu.



#### 6.3.8 Hibernate Caching

Hibernate cung cấp cơ chế **bộ nhớ đệm (cache)** giúp **giảm số lần truy cập database**,
từ đó **tăng hiệu năng** cho ứng dụng.

* Hibernate lưu trữ các đối tượng trong session khi transaction được kích hoạt.
* Khi một query được thực hiện liên tục, giá trị đã lưu trong session được sử dụng lại.
* Khi một transaction mới bắt đầu, dữ liệu được lấy lại từ database và được lưu vào session mới.

Hibernate cung cấp **hai cấp độ cache**:

* **Level 1 Cache** (mặc định, theo session)
* **Level 2 Cache** (chia sẻ giữa các session)

## C. Cấu hình DataSource cơ bản (MySQL) trong Spring Boot 3.x

### 1. Maven dependency (MySQL driver + Spring Data JPA)

#### a. pom.xml tối thiểu

```xml
<dependencies>
    <!-- Spring Web nếu bạn viết REST API -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL JDBC driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Test (tuỳ chọn) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2. Cấu hình trong application.properties

#### a. Cấu hình cơ bản

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/demo_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# (Tuỳ chọn) Cấu hình JPA/Hibernate
spring.jpa.hibernate.ddl-auto=update   # create | create-drop | update | validate | none
spring.jpa.show-sql=true               # In SQL ra console
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
```

#### b. Giải thích nhanh

* `spring.datasource.url`

  * `jdbc:mysql://host:port/db_name`
  * Thêm params `useSSL=false`, `serverTimezone=UTC` để tránh warning.
* `spring.datasource.username` / `password`:

  * Tài khoản kết nối DB.
* `spring.jpa.hibernate.ddl-auto`:

  * `create`: mỗi lần chạy **drop bảng + tạo lại**.
  * `update`: cập nhật schema theo entity, không xoá bảng (dùng nhiều khi dev).
  * `validate`: chỉ kiểm tra schema, không thay đổi.
  * `none`: không làm gì cả.

### 3. Cấu hình tương đương trong application.yml

#### a. Cấu trúc YAML

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/demo_db?useSSL=false&serverTimezone=UTC
    username: root
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQLDialect
```

