# SDL2
## Цель
Развернуть окружение с веб-сайтом с использованием базы данных, настроить постоянное хранение и создать сетевое окружение. Проверить получившийся контейнер и образ на безопасность.
## Исходные данные
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/990c0fde-9f85-4776-bf00-d00bbe875abc)
## Код докер-файлов
### Dockerfile
```{r}
FROM php:8.2-apache
RUN a2enmod rewrite
RUN docker-php-ext-install mysqli pdo pdo_mysql
WORKDIR /var/www/html
COPY ./www .
```
### docker-compose.yml
```{r}
version: '3.9'
services:
  webserver:
    container_name: PHP-webServer
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./www:/var/www/html
    ports:
      - 8000:80
    depends_on:
      - mysql-db
  mysql-db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: test_database
      MYSQL_USER: db_user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    links:
      - mysql-db
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql-db
      MYSQL_ROOT_PASSWORD: password
```
### index.php
```{r}
<?php
$host = 'mysql-db';
$user = 'db_user';
$pass = 'password';
$db = 'test_database';
$conn = new mysqli($host, $user, $pass, $db);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "Connected to MySQL successfully";
// Handle POST request
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (isset($_POST['submit'])) {
        $name = $_POST['name'];
        $email = $_POST['email'];
        // Insert data into the database
        $sql = "INSERT INTO users (name, email) VALUES ('$name', '$email')";
       if ($conn->query($sql) === TRUE) {
            echo "<br>Data inserted successfully";
        } else {
            echo "<br>Error inserting data: " . $conn->error;
        }
    }
}
// Handle GET request
if ($_SERVER["REQUEST_METHOD"] == "GET") {
    if (isset($_GET['action']) && $_GET['action'] == 'fetch') {
        // Retrieve data from the database
        $result = $conn->query("SELECT * FROM users");
        if ($result->num_rows > 0) {
            echo "<br><br>Users:<br>";
            while ($row = $result->fetch_assoc()) {
                echo "Name: " . $row["name"] . " - Email: " . $row["email"] . >
            }
        } else {
            echo "<br>No users in the database";
        }
    }
}
$conn->close();
?>
<!DOCTYPE html>
<html>
<head>
    <title>PHP Form</title>
</head>
<body>
    <h2>Submit Form</h2>
    <form method="post" action="<?php echo $_SERVER['PHP_SELF']; ?>">
        Name: <input type="text" name="name"><br>
        Email: <input type="text" name="email"><br>
        <input type="submit" name="submit" value="Submit">
    </form>
    <h2>Retrieve Data</h2>
    <a href="?action=fetch">Fetch Data</a>
</body>
</html>
```
##
Поднимем контейнеры из папки, содержащей файл docker-compose.yml:
```{r}
docker-compose up -d
```
В результате по адресу https://localhost:8000 будет доступен простой сайт с формой ввода имени и e-mail и вывода ранее введённых.
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/2ec351e3-ba28-483d-a751-f665e660c07a)
Установим trivy, первый сканер уязвимостей и запустим проверять образ php-docker_webserver:
```{r}
snap install trivy
trivy image php-docker_webserver
```
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/95785221-c3a1-484c-8d4a-3086be58d74a)
Запустить trivy не получилось, перейдём к другому сканеру, bench security:
```{r}
git clone https://github.com/docker/docker-bench-security.git
cd docker-bench-security/
sh docker-bench-security.sh
```
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/a1cebed5-c124-4aa3-9c33-0304dd7a967f)
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/e3ac3a13-c73e-4f80-868a-dc22555a7438)
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/d460f609-a352-4fa4-ac4e-2e966aad9b31)
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/0493e76b-528a-4dc6-9aef-282f8d0c1e13)
![image](https://github.com/Din0saur0s/SDL2/assets/70744702/c0e11fc4-d574-460d-8f6a-672f2a495cbe)




