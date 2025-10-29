📁 Структура проекта

vet_helper/
│
├── db.php
├── register.php
├── login.php
├── logout.php
├── appointments.php
├── new_appointment.php
├── admin.php
├── style.css
└── index.php


⸻

💾 1. База данных (MySQL)

Создай базу, например, vet_helper, и выполни этот SQL:

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  username VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL
);

CREATE TABLE appointments (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  pet_name VARCHAR(50),
  animal_type VARCHAR(30),
  service_type VARCHAR(50),
  date DATE,
  symptoms TEXT,
  status VARCHAR(30) DEFAULT 'Заявлена',
  FOREIGN KEY (user_id) REFERENCES users(id)
);


⸻

⚙️ 2. db.php (подключение к базе)

<?php
$conn = new mysqli("localhost", "root", "", "vet_helper");
if ($conn->connect_error) {
    die("Ошибка подключения: " . $conn->connect_error);
}
session_start();
?>


⸻

🏠 3. index.php (главная – просто ссылки)

<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<link rel="stylesheet" href="style.css">
<title>Ветеринарный помощник</title>
</head>
<body>
<h2>Добро пожаловать в «Ветеринарный помощник»</h2>
<a href="login.php">Войти</a> | 
<a href="register.php">Регистрация</a>
</body>
</html>


⸻

📝 4. register.php (регистрация)

<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Регистрация</title></head>
<body>
<h2>Регистрация</h2>

<form method="post">
  Логин: <input type="text" name="username" required><br>
  Пароль: <input type="password" name="password" required><br>
  <button type="submit" name="register">Зарегистрироваться</button>
</form>

<p>Уже есть аккаунт? <a href="login.php">Войти</a></p>

<?php
if (isset($_POST['register'])) {
    $username = $_POST['username'];
    $password = password_hash($_POST['password'], PASSWORD_DEFAULT);

    $sql = "INSERT INTO users (username, password) VALUES ('$username', '$password')";
    if ($conn->query($sql)) {
        echo "Регистрация успешна. <a href='login.php'>Войти</a>";
    } else {
        echo "Ошибка: пользователь уже существует.";
    }
}
?>
</body>
</html>


⸻

🔑 5. login.php (авторизация)

<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Вход</title></head>
<body>
<h2>Авторизация</h2>

<form method="post">
  Логин: <input type="text" name="username" required><br>
  Пароль: <input type="password" name="password" required><br>
  <button type="submit" name="login">Войти</button>
</form>

<p>Новый пользователь? <a href="register.php">Регистрация</a></p>

<?php
if (isset($_POST['login'])) {
    $username = $_POST['username'];
    $password = $_POST['password'];
    $result = $conn->query("SELECT * FROM users WHERE username='$username'");
    if ($result->num_rows > 0) {
        $user = $result->fetch_assoc();
        if (password_verify($password, $user['password'])) {
            $_SESSION['user_id'] = $user['id'];
            header("Location: appointments.php");
            exit;
        }
    }
    echo "Ошибка авторизации.";
}
?>
</body>
</html>


⸻

🚪 6. logout.php

<?php
session_start();
session_destroy();
header("Location: index.php");
exit;
?>


⸻

📜 7. appointments.php (история заявок)

<?php include 'db.php';
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit; }
$user_id = $_SESSION['user_id'];
$result = $conn->query("SELECT * FROM appointments WHERE user_id=$user_id");
?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>История обращений</title></head>
<body>
<h2>Ваши заявки</h2>
<a href="new_appointment.php">Создать заявку</a> |
<a href="logout.php">Выход</a>
<table border="1" cellpadding="5">
<tr><th>Питомец</th><th>Дата</th><th>Услуга</th><th>Статус</th></tr>
<?php while($row = $result->fetch_assoc()): ?>
<tr>
<td><?= $row['pet_name'] ?></td>
<td><?= $row['date'] ?></td>
<td><?= $row['service_type'] ?></td>
<td><?= $row['status'] ?></td>
</tr>
<?php endwhile; ?>
</table>
</body>
</html>


⸻

🐾 8. new_appointment.php (создание заявки)
<?php include 'db.php';
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit; }
?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Новая заявка</title></head>
<body>
<h2>Создать заявку</h2>
<form method="post">
Кличка питомца: <input type="text" name="pet_name" required><br>
Вид животного: 
<select name="animal_type">
<option>Кот</option><option>Собака</option><option>Грызун</option><option>Птица</option><option>Другое</option>
</select><br>
Тип услуги: 
<select name="service_type">
<option>Консультация</option><option>Вакцинация</option><option>Чистка</option><option>Стрижка</option><option>Осмотр</option>
</select><br>
Дата приема: <input type="date" name="date" required><br>
Пожелания/симптомы: <br><textarea name="symptoms" rows="3"></textarea><br>
<button type="submit" name="submit">Отправить заявку</button>
</form>
<?php
if (isset($_POST['submit'])) {
  $pet = $_POST['pet_name'];
  $type = $_POST['animal_type'];
  $service = $_POST['service_type'];
  $date = $_POST['date'];
  $symptoms = $_POST['symptoms'];
  $uid = $_SESSION['user_id'];
  $conn->query("INSERT INTO appointments (user_id, pet_name, animal_type, service_type, date, symptoms) VALUES 
  ($uid, '$pet', '$type', '$service', '$date', '$symptoms')");
  echo "Заявка отправлена! <a href='appointments.php'>Вернуться</a>";
}
?>
</body>
</html>


⸻

🧑‍⚕️ 9. admin.php (панель администратора)

<?php include 'db.php';
$admin_login = "admin";
$admin_pass = "12345";

if (isset($_POST['login'])) {
  if ($_POST['username'] == $admin_login && $_POST['password'] == $admin_pass) {
    $_SESSION['admin'] = true;
  } else echo "Неверные данные";
}

if (!isset($_SESSION['admin'])): ?>
<!DOCTYPE html>
<html><body>
<h2>Вход администратора</h2>
<form method="post">
Логин: <input type="text" name="username"><br>
Пароль: <input type="password" name="password"><br>
<button name="login">Войти</button>
</form>
</body></html>
<?php exit; endif; ?>

<!DOCTYPE html>
<html><body>
<h2>Панель администратора</h2>
<a href="logout.php">Выход</a>
<table border="1" cellpadding="5">
<tr><th>ID</th><th>Пользователь</th><th>Питомец</th><th>Дата</th><th>Услуга</th><th>Статус</th><th>Действие</th></tr>
<?php
$result = $conn->query("SELECT a.*, u.username FROM appointments a JOIN users u ON a.user_id=u.id");
while ($row = $result->fetch_assoc()):
?>
<tr>
<td><?= $row['id'] ?></td>
<td><?= $row['username'] ?></td>
<td><?= $row['pet_name'] ?></td>
<td><?= $row['date'] ?></td>
<td><?= $row['service_type'] ?></td>
<td><?= $row['status'] ?></td>
<td>
<form method="post">
<input type="hidden" name="id" value="<?= $row['id'] ?>">
<select name="status">
<option>Заявлена</option>
<option>Подтверждена</option>
<option>Прием завершен</option>
</select>
<button name="update">Изменить</button>
</form>
</td>
</tr>
<?php endwhile; ?>

<?php
if (isset($_POST['update'])) {
  $id = $_POST['id'];
  $status = $_POST['status'];
  $conn->query("UPDATE appointments SET status='$status' WHERE id=$id");
  header("Location: admin.php");
}
?>
</table>
</body></html>


⸻

🎨 10. style.css (минимум оформления)

body {
  font-family: Arial, sans-serif;
  margin: 20px;
}
table {
  border-collapse: collapse;
}
th, td {
  padding: 5px 10px;
}








📁 Структура проекта

sportstyle/
│
├── db.php
├── register.php
├── login.php
├── logout.php
├── bookings.php
├── new_booking.php
├── admin.php
├── style.css
└── index.php


⸻

💾 1. База данных (MySQL)

Создай базу данных sportstyle и выполни:

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  login VARCHAR(50) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  lastname VARCHAR(50) NOT NULL,
  firstname VARCHAR(50) NOT NULL,
  middlename VARCHAR(50) NOT NULL,
  phone VARCHAR(20) NOT NULL,
  email VARCHAR(100) NOT NULL
);

CREATE TABLE bookings (
  id INT AUTO_INCREMENT PRIMARY KEY,
  user_id INT,
  activity_type VARCHAR(50),
  trainer VARCHAR(50),
  date_time DATETIME,
  comment TEXT,
  status VARCHAR(30) DEFAULT 'Ожидает подтверждения',
  FOREIGN KEY (user_id) REFERENCES users(id)
);


⸻

⚙️ 2. db.php

<?php
$conn = new mysqli("localhost", "root", "", "sportstyle");
if ($conn->connect_error) {
    die("Ошибка подключения: " . $conn->connect_error);
}
session_start();
?>


⸻

🏠 3. index.php

<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<link rel="stylesheet" href="style.css">
<title>СпортСтайл</title>
</head>
<body>
<h2>Добро пожаловать в фитнес-портал «СпортСтайл»</h2>
<a href="login.php">Войти</a> |
<a href="register.php">Регистрация</a>
</body>
</html>


⸻

📝 4. register.php

<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Регистрация</title></head>
<body>
<h2>Регистрация нового участника</h2>

<form method="post">
Логин: <input type="text" name="login" pattern="[A-Za-z0-9]{4,}" required><br>
Пароль: <input type="password" name="password" minlength="6" required><br>
Фамилия: <input type="text" name="lastname" pattern="[А-Яа-яЁё\s]+" required><br>
Имя: <input type="text" name="firstname" pattern="[А-Яа-яЁё\s]+" required><br>
Отчество: <input type="text" name="middlename" pattern="[А-Яа-яЁё\s]+" required><br>
Телефон: <input type="text" name="phone" pattern="\+375\([0-9]{2}\)[0-9]{3}-[0-9]{2}-[0-9]{2}" required><br>
Email: <input type="email" name="email" required><br>
<button type="submit" name="register">Стать членом клуба</button>
</form>

<p>Уже зарегистрированы? <a href="login.php">Войти</a></p>

<?php
if (isset($_POST['register'])) {
    $login = $_POST['login'];
    $password = password_hash($_POST['password'], PASSWORD_DEFAULT);
    $lastname = $_POST['lastname'];
    $firstname = $_POST['firstname'];
    $middlename = $_POST['middlename'];
    $phone = $_POST['phone'];
    $email = $_POST['email'];

    $sql = "INSERT INTO users (login, password, lastname, firstname, middlename, phone, email)
            VALUES ('$login', '$password', '$lastname', '$firstname', '$middlename', '$phone', '$email')";
    if ($conn->query($sql)) {
        echo "Регистрация успешна! <a href='login.php'>Войти</a>";
    } else {
        echo "Ошибка: логин уже используется.";
    }
}
?>
</body>
</html>


⸻

🔑 5. login.php

<?php include 'db.php'; ?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Вход</title></head>
<body>
<h2>Авторизация</h2>
<form method="post">
Логин: <input type="text" name="login" required><br>
Пароль: <input type="password" name="password" required><br>
<button type="submit" name="login_btn">Войти</button>
</form>

<p>Нет аккаунта? <a href="register.php">Зарегистрируйтесь!</a></p>

<?php
if (isset($_POST['login_btn'])) {
    $login = $_POST['login'];
    $password = $_POST['password'];
    $result = $conn->query("SELECT * FROM users WHERE login='$login'");
    if ($result->num_rows > 0) {
        $user = $result->fetch_assoc();
        if (password_verify($password, $user['password'])) {
            $_SESSION['user_id'] = $user['id'];
            header("Location: bookings.php");
            exit;
        }
    }
    echo "Неверные учетные данные.";
}
?>
</body>
</html>


⸻

🚪 6. logout.php

<?php
session_start();
session_destroy();
header("Location: index.php");
exit;
?>


⸻

📅 7. bookings.php (мои записи)
<?php include 'db.php';
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit; }
$user_id = $_SESSION['user_id'];
$result = $conn->query("SELECT * FROM bookings WHERE user_id=$user_id");
?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Мои записи</title></head>
<body>
<h2>Мои записи</h2>
<a href="new_booking.php">Записаться на тренировку</a> |
<a href="logout.php">Выход</a>
<table border="1" cellpadding="5">
<tr><th>Дата и время</th><th>Тип занятия</th><th>Тренер</th><th>Статус</th></tr>
<?php while($row = $result->fetch_assoc()): ?>
<tr>
<td><?= $row['date_time'] ?></td>
<td><?= $row['activity_type'] ?></td>
<td><?= $row['trainer'] ?></td>
<td><?= $row['status'] ?></td>
</tr>
<?php endwhile; ?>
</table>
</body>
</html>


⸻

🏋️ 8. new_booking.php (запись на тренировку)

<?php include 'db.php';
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit; }
?>
<!DOCTYPE html>
<html lang="ru">
<head><meta charset="UTF-8"><title>Новая запись</title></head>
<body>
<h2>Записаться на тренировку</h2>
<form method="post">
Тип занятия:
<select name="activity_type" id="activity" required>
<option value="Йога">Йога</option>
<option value="Персональная тренировка">Персональная тренировка</option>
<option value="Бассейн">Бассейн</option>
<option value="Кардиозона">Кардиозона</option>
</select><br>

Тренер:
<select name="trainer" required>
<option>Иванова Ольга</option>
<option>Сидоров Алексей</option>
<option>Ким Анастасия</option>
<option>Петров Игорь</option>
</select><br>

Дата и время: <input type="datetime-local" name="date_time" required><br>
Дополнительные пожелания:<br>
<textarea name="comment" rows="3"></textarea><br>
<button type="submit" name="submit">Забронировать</button>
</form>

<?php
if (isset($_POST['submit'])) {
  $user_id = $_SESSION['user_id'];
  $activity = $_POST['activity_type'];
  $trainer = $_POST['trainer'];
  $date_time = $_POST['date_time'];
  $comment = $_POST['comment'];

  $conn->query("INSERT INTO bookings (user_id, activity_type, trainer, date_time, comment)
                VALUES ($user_id, '$activity', '$trainer', '$date_time', '$comment')");
  echo "Заявка отправлена! <a href='bookings.php'>Мои записи</a>";
}
?>
</body>
</html>


⸻

🧑‍💼 9. admin.php (панель администратора)

<?php include 'db.php';
$admin_login = "admin";
$admin_pass = "12345";

if (isset($_POST['login'])) {
  if ($_POST['username'] == $admin_login && $_POST['password'] == $admin_pass) {
    $_SESSION['admin'] = true;
  } else echo "Неверные данные";
}

if (!isset($_SESSION['admin'])): ?>
<!DOCTYPE html>
<html><body>
<h2>Панель администратора</h2>
<form method="post">
Логин: <input type="text" name="username"><br>
Пароль: <input type="password" name="password"><br>
<button name="login">Войти</button>
</form>
</body></html>
<?php exit; endif; ?>

<!DOCTYPE html>
<html><body>
<h2>Все записи клиентов</h2>
<a href="logout.php">Выход</a>
<table border="1" cellpadding="5">
<tr><th>ID</th><th>Клиент</th><th>Тип занятия</th><th>Тренер</th><th>Дата и время</th><th>Статус</th><th>Действие</th></tr>
<?php
$result = $conn->query("SELECT b.*, u.lastname, u.firstname FROM bookings b JOIN users u ON b.user_id=u.id");
while ($row = $result->fetch_assoc()):
?>
<tr>
<td><?= $row['id'] ?></td>
<td><?= $row['lastname'] ?> <?= $row['firstname'] ?></td>
<td><?= $row['activity_type'] ?></td>
<td><?= $row['trainer'] ?></td>
<td><?= $row['date_time'] ?></td>
<td><?= $row['status'] ?></td>
<td>
<form method="post">
<input type="hidden" name="id" value="<?= $row['id'] ?>">
<select name="status">
<option>Ожидает подтверждения</option>
<option>Подтверждено</option>
<option>Отменено</option>
</select>
<button name="update">Изменить</button>
</form>
</td>
</tr>
<?php endwhile; ?>

<?php
if (isset($_POST['update'])) {
  $id = $_POST['id'];
  $status = $_POST['status'];
  $conn->query("UPDATE bookings SET status='$status' WHERE id=$id");
  header("Location: admin.php");
}
?>
</table>
</body></html>


⸻

🎨 10. style.css
body {
  font-family: Arial, sans-serif;
  margin: 20px;
}
table {
  border-collapse: collapse;
}
th, td {
  padding: 6px 10px;
}
