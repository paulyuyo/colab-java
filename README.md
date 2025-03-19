# colab-java
在colab，裡面使用java和spring boot。

Google Colab 原生支援 Python，但如果你想在 Colab 裡使用 Java，可以透過以下方法來實現：

---

## 方法 1：使用 `apt` 安裝 OpenJDK（適用於簡單 Java 程式）
1. 在 Colab 的 Notebook 中執行以下命令，安裝 OpenJDK：
   ```python
   !apt-get install openjdk-17-jdk -y
   ```
2. 確認 Java 已安裝：
   ```python
   !java -version
   ```
3. 在 Colab 中執行 Java 程式：
   - 建立 Java 檔案並編譯、執行：
     ```python
     %%writefile HelloWorld.java
     public class HelloWorld {
         public static void main(String[] args) {
             System.out.println("Hello, Google Colab!");
         }
     }
     ```
     - 編譯 Java 程式：
       ```python
       !javac HelloWorld.java
       ```
     - 執行 Java 程式：
       ```python
       !java HelloWorld
       ```

---

## 方法 2：使用 `jupyter-java`（適用於交互式執行）
Colab 其實支援 Jupyter Notebook，所以我們可以透過 `IJava` 來讓 Colab 支援 Java：
1. 安裝 `IJava`：
   ```python
   !apt-get install openjdk-17-jdk -y
   !pip install jupyter
   !git clone https://github.com/SpencerPark/IJava.git
   !cd IJava && ./gradlew installKernel
   ```
2. 重新啟動 Colab Runtime（點選 "Runtime" → "Restart runtime"）。
3. 在 Notebook 第一個 cell 中執行：
   ```java
   System.out.println("Hello, Java in Colab!");
   ```
   **注意**：此方法可能需要較長的安裝時間，但能讓 Colab 直接支援 Java 的互動式執行。

---



### 選擇建議：
- **如果只是執行 Java 程式**，方法 1 比較簡單。
- **如果希望像 Python 一樣交互執行 Java**，方法 2 會比較合適。

如果你有更特定的需求，可以告訴我，我會根據你的需求推薦更合適的做法！ 😊


如果你想在 **Google Colab** 內運行 **Spring Boot**，並且連接 **MySQL** 或 **PostgreSQL**，這裡有詳細的步驟來實現這個需求。

---

## **步驟 1：安裝 Java & Maven**
先安裝 **OpenJDK 17**（或其他版本），以及 **Maven**：
```python
!apt-get install openjdk-17-jdk -y
!apt-get install maven -y
```
確認 Java 和 Maven 是否安裝成功：
```python
!java -version
!mvn -version
```

---

## **步驟 2：建立 Spring Boot 專案**
我們使用 **Spring Initializr** 來生成 Spring Boot 專案：
```python
!curl https://start.spring.io/starter.zip -d dependencies=web,data-jpa,mysql,postgresql -d type=maven-project -d language=java -o spring-boot-app.zip
!unzip spring-boot-app.zip -d spring-boot-app
%cd spring-boot-app
```
這樣會產生一個 **Spring Boot** 專案，並包含：
- `Spring Web`（提供 REST API）
- `Spring Data JPA`（用來與資料庫交互）
- `MySQL` 和 `PostgreSQL` 依賴（可以二選一使用）

---

## **步驟 3：啟動 MySQL 或 PostgreSQL**
Colab 內部沒有內建資料庫，所以我們需要：
1. **使用 Docker 安裝 MySQL 或 PostgreSQL**
2. **連接外部雲端 MySQL/PostgreSQL（如 AWS RDS、ElephantSQL）**

### **選項 1：使用 Docker 啟動本地 MySQL**
```python
!apt-get install docker.io -y
!service docker start
!docker run --name mysql-db -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=testdb -p 3306:3306 -d mysql:latest
```
> MySQL 的連線資訊：
> - **host:** `localhost`
> - **port:** `3306`
> - **database:** `testdb`
> - **user:** `root`
> - **password:** `root`

### **選項 2：使用 Docker 啟動本地 PostgreSQL**
```python
!docker run --name postgres-db -e POSTGRES_USER=root -e POSTGRES_PASSWORD=root -e POSTGRES_DB=testdb -p 5432:5432 -d postgres:latest
```
> PostgreSQL 的連線資訊：
> - **host:** `localhost`
> - **port:** `5432`
> - **database:** `testdb`
> - **user:** `root`
> - **password:** `root`

### **選項 3：連接雲端 MySQL/PostgreSQL**
如果你有 **雲端資料庫**（如 **AWS RDS, ElephantSQL, Heroku Postgres**），可以直接設定遠端連線，修改 **`application.properties`**（詳見步驟 4）。

---

## **步驟 4：設定 `application.properties`**
根據你的資料庫選擇不同的設定：

### **MySQL 設定**
在 `src/main/resources/application.properties` 內新增：
```
spring.datasource.url=jdbc:mysql://localhost:3306/testdb
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.database-platform=org.hibernate.dialect.MySQL8Dialect
spring.jpa.hibernate.ddl-auto=update
```

### **PostgreSQL 設定**
如果你用的是 **PostgreSQL**，請改成：
```
spring.datasource.url=jdbc:postgresql://localhost:5432/testdb
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
```

如果你使用 **雲端 MySQL/PostgreSQL**，請把 `localhost` 改成你的遠端資料庫 IP。

---

## **步驟 5：建立 JPA Entity 和 Repository**
在 `src/main/java/com/example/demo/model/User.java` 新增：
```java
package com.example.demo.model;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    public User() {}

    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}
```

在 `src/main/java/com/example/demo/repository/UserRepository.java` 新增：
```java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {}
```

---

## **步驟 6：建立 REST Controller**
在 `src/main/java/com/example/demo/controller/UserController.java` 新增：
```java
package com.example.demo.controller;

import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @GetMapping
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userRepository.save(user);
    }
}
```
這樣我們就建立了一個 **REST API**：
- `GET /users` → 取得所有使用者
- `POST /users` → 新增使用者

---

## **步驟 7：執行 Spring Boot 應用程式**
回到專案目錄後執行：
```python
!mvn spring-boot:run
```
你應該會看到 Spring Boot 啟動成功，並運行在 **port 8080**。

---

## **步驟 8：使用 `ngrok` 來公開 API（可選）**
由於 Colab 沒有真正的公開 IP，我們可以使用 **ngrok** 來將 **Spring Boot API** 暴露給外部：
```python
!./ngrok http 8080
```
這樣你就會獲得一個 **外部可訪問的 URL**（例如 `https://abcd1234.ngrok.io`），你可以使用 **Postman 或瀏覽器測試 API**：
```sh
curl https://abcd1234.ngrok.io/users
```

---

## **結論**
✅ **這套方案可以讓你在 Colab 內部執行 Spring Boot 並連接 MySQL 或 PostgreSQL！**
- 如果你只想 **本地測試**，使用 **Docker 安裝 MySQL/PostgreSQL**。
- 如果你要 **在雲端運行**，請連接 **遠端 MySQL/PostgreSQL（AWS RDS, ElephantSQL）**。

如果你還有其他需求，像是 **使用 Redis、Kafka、Docker Compose**，或是想要 **部署到雲端（如 AWS, GCP, Heroku）**，可以再問我，我可以提供更進一步的教學！ 🚀🚀
