---
date_created: 2025-09-28
date_modified: 2025-09-28
document_type: resource
tags: resource 
---
[[Resources/Resources Dashboard|Resources]] / **[[Resources/SQL MariaDB|SQL MariaDB]]**
# SQL MariaDB
**Overview**
Description:: SQL short curse

## Note

# MariaDB
Ми навчимося створювати бази та таблиці, пов'язувати їх та оптимізувати, додавати данні а також з цими даними взаємодіяти.<a name="top"></a>

1. [Cтруктура-мови](#структура-мови)

2. [Оператори визначення даних (CREATE, ALERT, DROP)](#оператори-визначення-даних-data-definition-language)
   - [Add createdAt and updatedAt columns](#add-createdat-and-updatedat-columns)

3. [Index and Constraint](#index-and-constraint)
   - [INDEX](#index)
   - [PRIMARY KEY](#primary-key)
   - [UNIQUE KEY](#unique-key)
   - [FOREIGN KEY](#foreign-key)

4. [Database Relationship Types](#database-relationship-types)
   - [One-to-One 1:1 relationship](#1️⃣-one-to-one-11-relationship)
   - [One to Many 1:M relationship](#2️⃣-one-to-many-1m-relationship)
   - [Many to Many M:M relationship](#3️⃣-many-to-many-mm-relationship)

5. [Оператори маніпуляції даними](#оператори-маніпуляції-даними-data-manipulation-language)
   - [INSERT](#insert)
   - [LOAD](#load)
   - [DELETE](#delete)
   - [UPDATE](#update)
   - [SELECT](#select)
        - [DISTINCT](#distinct)
   - [Функції агрегації (агрегати) SUM, COUNT, MIN, MAX](#функції-агрегації-агрегати)

6. [Clauses](#клаузули)
   - [ORDER BY](#order-by)
   - [LIMIT](#limit)
   - [OFFSET](#offset)
   - [WHERE](#where)
   - [BETWEEN](#between)
   - [LIKE](#like)
   - [IN](#in)
   - [IS NULL](#is-null)
   - [REGEXP](#regexp)
   - [GROUP BY](#group-by)
   - [HAVING](#having)

7. [Об'єднання таблиць (JOIN, LEFT JOIN, RIGHT JOIN)](#обєднання-таблиць)

8. [Підзапити (UNION, IN, ANY, SOME, NOT)](#обєднання-запиту-та-підзапити)

9. [View](#view-перегляди)

10. [Процедури, що зберігаються](#процедури-що-зберігаються)
    - [PROCEDURE](#procedure)
    - [Змінні](#змінні-variables)
    - [Error HANDLER](#визначення-обробників-error-handler)
    - [CURSOR](#cursor)
    - [FUNCTION](#функції-що-зберігаються-function)
    - [PREPARE (динамічний SQL)](#prepare-динамічний-sql)
    - [TRIGGER](#trigger)

11. [Керуючі конструкції](#керуючі-конструкції)
    - [IF](#оператор-if)
    - [CASE](#оператор-case)
    - [LOOP](#оператор-loop)
    - [REPEAT](#оператор-repeat)
    - [WHILE](#оператор-while)
    - [FOR](#оператор-for)

12. [Оптимізація та обслуговування](#оптимізація-та-обслуговування)
    - [EXPLAIN](#explain)

# Installation

[Installation](https://docs.vultr.com/how-to-install-mariadb-on-ubuntu-24-04)

```SQL
sudo apt install mariadb-server
```

Access db from console with password:

```SQL
mariadb -u root -p my_password [останнім параметром може бути ім'я бази даних]
```

```bash
mysql -u root -panyhow06 quizdb
```
Export table to csv file:

```bash
mysql -uroot -panyhow06 -e "SELECT * from tiers" quizdb | sed 's/\t/","/g;s/^/"/;s/$/"/' > /home/andrii/GoIT/mariadb/tiers_.csv
```

# Структура мови

## Оператори визначення даних (DDL)

Потрібні для:

    CREATE DATABASE - Створення бази даних

    DROP DATABASE - Видалення бази даних

    CREATE TABLE - Створення таблиці

    DROP TABLE - Видалення таблиці

    ALTER DATABASE Модифікація бази

    ALTER TABLE Модифікація таблиці

Як ми бачимо основні оператори - це CREATE, DROP та ALTER

## Оператори маніпуляції даними (DML)
Це всюдисущі оператори, з якими ви найчастіше працюватимете:

    SELECT - Отримати дані з таблиці

    UPDATE - Оновити дані у таблиці
    
    INSERT - Додати дані до таблиці

    DELETE - Видалити дані з таблиці

## Оператори визначення доступу до даних (DCL)
Управління правами доступу, як до користувача так і до таблиці, також транзакції вважаються контролем даних.

    GRANT - права користувачів

    CONNECT – з'єднання з сервером

    LOCK TABLE – блокування таблиці

    UNLOCK TABLE – розблокувати таблицю

    COMMIT - так само як і в git додати зміни
    
    ROLLBACK - скасувати зміни в транзакції (якщо COMMIT не було)

## Синтаксис
Як я писав вище всі команди великими літерами.

Роздільник для запиту - це крапка з комою ";"

Ось які бувають коментарі:

"--" два мінуси - це однорядковий коментар, стандартний, який точно працює у всіх базах    

<a href="#top">⬅️ Back to top</a>

# Оператори визначення даних (Data Definition Language)
DDL (Data Definition Language)

[Query examples](https://www.edu.goit.global/uk/learn/5265254/1346906/1346908/textbook)

```SQL
SHOW DATABASES; 
```

*Create db*

```SQL
CREATE DATABASE quizdb 
  CHARACTER SET = 'utf8'
  COLLATE = 'utf8_general_ci';
```

*Connect to db:*
```SQL
USE quizdb; 
```

*Delete db:*
```SQL
DROP quizdb; 
```

*Create table*
```SQL
CREATE TABLE IF NOT EXISTS products (
    id INT UNSIGNED AUTO_INCREMENT NOT NULL COMMENT 'Код товару',
    name CHAR(50) NOT NULL DEFAULT '' COMMENT 'Назва товару',
    CONSTRAINT products_PK PRIMARY KEY (id)
);
```

*Describe table:*
```SQL
DESC products; 
```

*Make changes to the table*
```SQL
ALTER TABLE quizdb.questions ADD question_type ENUM('MULTIPLE', 'TEXT', 'TEXT NUMERIC', '') NULL;
```
*Search in all databases for column createdAt or label*
```SQL
SELECT column_name, table_schema, table_name
  FROM information_schema.columns
  WHERE column_name = 'createdAt'
  OR column_name = 'label'
```

```SQL
-- Схема БД сервера
USE information_schema;

-- Таблиці схеми
SHOW TABLES;

-- Таблиці БД
DESC tables;

-- Список БД
SELECT *
    FROM information_schema.schemata \G

-- Список таблиць зазначеної БД
SELECT table_name, table_comment
    FROM information_schema.tables
    WHERE table_schema = 'module2';
    
-- Специфікація стовпчиків для таблиці table2 БД module2
SELECT column_name, data_type, column_comment
    FROM information_schema.columns
    WHERE table_schema = 'module2'
    AND table_name = 'table2';
```

### *Add createdAt and updatedAt columns*
[video tutor](https://www.youtube.com/watch?v=1CjNUdhjl0A&list=PLViULGko0FdhaIH_pXXAF8qcRRfgWKhU2&index=10)
```SQL
ALTER TABLE quizdb.questions 
ADD COLUMN createdAt TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP;

ALTER TABLE quizdb.questions 
ADD COLUMN updatedAt TIMESTAMP NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;
```

<a href="#top">⬅️ Back to top</a>

# INDEX and CONSTRAINT

[Doc](https://textbook.edu.goit.global/mysql-qhg4na/v1/uk/docs/module3/docs/doc1/) | [Generate dummy db with data](https://filldb.info/) | [Query examples](https://textbook.edu.goit.global/mysql-qhg4na/v1/uk/docs/module3/docs/doc4)

## INDEX

Індекси служвть для багаторазоаого пришвидчення вибірки з бази по полю (стовбчику) на якому створюється індекс. 

Є різні типи індексів BTree (збалансоване бінарне дерево), Hash, Full Text, у PostgreSQL взагалі індексів дуже багато, плюс там можна створювати свої індекси.

Але в 99% випадків ви використовуватимете збалансоване дерево, воно створюється за замовчуванням і більшість навіть не знає, що є інші типи індексів.

Сам індекс створюється трьома шляхами.

Під час створення таблиці.

```SQL
CREATE TABLE table1
(
    id INT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'Код товару',
    code char(4) NOT NULL DEFAULT 'AAAA',
    name VARCHAR(50) NOT NULL DEFAULT '' COMMENT 'Назва товару',
    description TEXT NOT NULL DEFAULT '' COMMENT 'Опис товару',
    price FLOAT NOT NULL DEFAULT 0 COMMENT 'Ціна товару',
    CONSTRAINT table1_id_pk PRIMARY KEY (id),
    INDEX table1_name_ix(name),
    INDEX table1_price_ix(price)
) COMMENT 'Таблиця товарів з ключами та індексами';
```

Змінюючи таблицю через ALTER:

```SQL
ALTER TABLE table1 ADD INDEX table1_name_ix(name), ADD INDEX table1_price_ix(price);
```

І додати індекс через CREATE:

```SQL
CREATE INDEX table1_name_ix ON table1 (name);
```
<a href="#top">⬅️ Back to top</a>

## CONSTRAINT

### PRIMARY KEY
PRIMART KEY - це також індекс, як і всі обмеження. Коли ми говоримо серверу PRIMART KEY, ми ставимо обмеження первинного ключа, тобто ми пояснюємо серверу, що обов'язково перевіряй на унікальність (AUTO_INCREMENT).

```SQL
CREATE TABLE table1
(
    id INT UNSIGNED NOT NULL AUTO_INCREMENT COMMENT 'Код товару',
    code char(4) NOT NULL DEFAULT 'AAAA',
    name VARCHAR(50) NOT NULL DEFAULT '' COMMENT 'Назва товару',
    description TEXT NOT NULL DEFAULT '' COMMENT 'Опис товару',
    price FLOAT NOT NULL DEFAULT 0 COMMENT 'Ціна товару',
    CONSTRAINT table1_id_pk PRIMARY KEY (id)
) COMMENT 'Таблиця товарів із ключами';
```
<a href="#top">⬅️ Back to top</a>

### UNIQUE KEY
Це ключ кандидат, має всі властивості первинного (ми пояснюємо серверу, що обов'язково перевіряй на унікальність), але при цьому первинним не є.

```SQL
ALTER TABLE persons ADD CONSTRAINT persons_itn_un UNIQUE KEY (ITN);
```

```SQL
CREATE TABLE `questions` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `label` char(15) DEFAULT NULL, -- Unique key, repetition not allowed
  `image` char(15) DEFAULT NULL,
  `pass_allowed` tinyint(1) DEFAULT 1,
  `answer_options` char(100) DEFAULT NULL,
  `answer` char(100) DEFAULT NULL,
  `question_description` varchar(200) DEFAULT NULL,
  `question_type` enum('MULTIPLE','TEXT','TEXT NUMERIC','') DEFAULT NULL,
  `createdAt` timestamp NULL DEFAULT current_timestamp(),
  `updatedAt` timestamp NULL DEFAULT current_timestamp() ON UPDATE current_timestamp(),
  PRIMARY KEY (`id`),
  UNIQUE KEY `questions_unique` (`label`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```
<a href="#top">⬅️ Back to top</a>

### FOREIGN KEY
Служить для зв'язку двох таблиць. Потрібно запам'ятати основні принципи:

- Завжди повинна опиратися на первинний ключ або ключ кандидат.
- Колонка, яка буде пов'язана, повинна бути з тим самим типом, що і батьківська колонка.

Рядок таблиці `tiers` зв'язується з рядком таблиці `questions` по полю `boundQuestion` в `tiers`. Це поле містить одне із значень в колонці `label` таблиці `questions`. При видаленні рядка в `questions` в `boundQuestion` запишеться NULL. При зміні значення `label` воно зміниться і в `boundQuestion`.

```SQL
CREATE TABLE `tiers` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `idx` int(11) DEFAULT NULL,
  `legend` char(20) DEFAULT NULL,
  `boundQuestion` char(15) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `tiers_questions_FK` (`boundQuestion`),
  CONSTRAINT `tiers_questions_FK` FOREIGN KEY (`boundQuestion`) REFERENCES `questions` (`label`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

Speed query example that use complex index

```SQL
EXPLAIN
SELECT * FROM cliets c USE INDEX (clients_gender_salary_IDX)
  WHERE c.gender = 'WOMAN' and c.salary BETWEEN 8000 and 9000;
```
```SQL
SELECT COUNT(*) FROM questions q  
WHERE q.label REGEXP '^b';
```
```SQL
SELECT COUNT(*) FROM questions q  
WHERE q.label LIKE 'b%';
```

Спробуємо зробити запит і отримати останній запис.
```SQL
SELECT id, name, phone
  FROM users
  ORDER BY id DESC
  LIMIT 1;
```
<a href="#top">⬅️ Back to top</a>

# Database Relationship Types
## 1️⃣ One-to-One (1:1) Relationship
Definition: Each record in Table A is related to one record in Table B.

Use Case: A user has one profile.

__Example Schema:__
```SQL
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE profiles (
    id INT PRIMARY KEY,
    user_id INT UNIQUE,  -- Ensures only one profile per user
    bio TEXT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

__Insert data__
```SQL
INSERT INTO users (id, name) VALUES (1, 'Alice'), (2, 'Bob');

INSERT INTO profiles (id, user_id, bio) VALUES 
(1, 1, 'Software Engineer'),
(2, 2, 'Data Scientist');
```
__Select Data (Join Users with Profiles)__

```SQL
SELECT users.id, users.name, profiles.bio 
FROM users
JOIN profiles ON users.id = profiles.user_id;
```
__Returns__
```SQL
+----+-------+-------------------+
| id | name  | bio               |
+----+-------+-------------------+
| 1  | Alice | Software Engineer |
| 2  | Bob   | Data Scientist    |
+----+-------+-------------------+
```
<a href="#top">⬅️ Back to top</a>

## 2️⃣ One-to-Many (1:M) Relationship
Definition: Each record in Table A can be linked to many records in Table B, but records in Table B can be linked to only one record in Table A.

Use Case: A customer places multiple orders.

__Example Schema:__

```SQL
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    customer_id INT,  -- Many orders can have the same customer
    amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(id) ON DELETE CASCADE
);
```
✅ Each customer can place multiple orders, but each order belongs to only one customer.

__Insert data__

```SQL
INSERT INTO customers (id, name) VALUES (1, 'Alice'), (2, 'Bob');

INSERT INTO orders (id, customer_id, amount) VALUES 
(1, 1, 100.50),
(2, 1, 75.20),
(3, 2, 50.00);
```

__Select Data (Join Customers with Orders)__

```SQL
SELECT customers.id, customers.name, orders.id AS order_id, orders.amount 
FROM customers
JOIN orders ON customers.id = orders.customer_id;
```
або так

```sql
SELECT customers.id, customers.name, orders.id AS order_id, orders.amount 
    FROM customers, orders
    WHERE customers.id = orders.customer_id;
```

<a href="#top">⬅️ Back to top</a>

## 3️⃣ Many-to-Many (M:M) Relationship
Definition: Each record in Table A can be related to multiple records in Table B, and vice versa.

Use Case: Students enroll in multiple courses, and each course has multiple students.

Solution: Use a junction table to manage relationships.

__Example Schema:__

```SQL
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE courses (
    id INT PRIMARY KEY,
    title VARCHAR(100)
);

-- Junction Table (Links students & courses)
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);
```
__Insert data:__

```SQL
INSERT INTO students (id, name) VALUES 
(1, 'Alice'),
(2, 'Bob');

INSERT INTO courses (id, title) VALUES 
(1, 'Math'),
(2, 'Science');

-- Student-Course Relationships
INSERT INTO student_courses (student_id, course_id) VALUES 
(1, 1), -- Alice -> Math
(1, 2), -- Alice -> Science
(2, 1); -- Bob -> Math
```

__Select Data (Join Students with Courses)__

```SQL
SELECT students.name, courses.title 
FROM students
JOIN student_courses ON students.id = student_courses.student_id
JOIN courses ON courses.id = student_courses.course_id;
```

__✅ Returns:__

```SQL
+-------+---------+
| name  | title   |
+-------+---------+
| Alice | Math    |
| Alice | Science |
| Bob   | Math    |
+-------+---------+
```
<a href="#top">⬅️ Back to top</a>

# Оператори маніпуляції даними (Data Manipulation Language)

## INSERT

[MariaDB Doc](https://mariadb.com/kb/en/insert/)

Також є таке поняття як прибрати конфліктуючий ряд, просто прибрати помилку під час запису. Один ряд конфліктує, але решта 6 будуть додані.

```SQL
-- Ігнорувати помилки конкретного ряду
INSERT IGNORE INTO table1 (id, name, price) VALUES 
(21, 'Товар', 12.5),
(42, 'Товар', 123.5),
(43, 'Товар', 12.5),
(44, 'Товар', 123.5),
(45, 'Товар', 12.5),
(46, 'Товар', 12.5),
(47, 'Товар', 123.5);
```
І є ще один різновид, зберегти за результатом вибірки з іншої таблиці.

```SQL
INSERT INTO table1 (name, price)
    SELECT user, 123 as prise
        FROM mysql.user;
```
<a href="#top">⬅️ Back to top</a>

## LOAD
Load data from file

[Документація](https://mariadb.com/kb/en/load-data-infile/)

```SQL
LOAD DATA LOCAL
    INFILE '/home/andrii/GoIT/mariadb/questions.csv'
    INTO TABLE questions
    FIELDS TERMINATED BY ',' -- чим розділяються поля
    LINES TERMINATED BY '\r\n' -- чим розділяються лінії 
    (label, question_type, image, answer_options, answer);
```
Він дуже швидко завантажить без зайвої мороки, таким чином можна завантажувати CSV файл тощо.

Приклад файлу, що завантажується

```SQL
a1,MULTIPLE,a1.png,�;�;�;�,�
d250,MULTIPLE,d250.png,�;�;�;�,�
e349,TEXT,e349.png,,Ĳ�;��
149,MULTIPLE,�148.PNG,�;�;�,�
b85,TEXT,b85.png,,�� ����� �������
d259,MULTIPLE,d259.png,�;�;�;�;�,�
e314,MULTIPLE,e314.png,�;�;�;�,�
e347,MULTIPLE,e347.png,�;�;�;�,�
a23,MULTIPLE,a23.png,�;�,�
b69,TEXT,b69.png,,���� ��'�;�� ��'�;��'�
e283,MULTIPLE,e283.png,�;�;�;�,�
e321,MULTIPLE,e321.png,�;�;�;�,�
b96,TEXT,b96.png,,�����
e309,MULTIPLE,e309.png,�;�;�;�,�
a50,MULTIPLE,a50.png,�;�;�;�,�
```
<a href="#top">⬅️ Back to top</a>

## DELETE
Видалення запису з бази, достатньо простий оператор.

Видалити всі записи з таблиці table1

```sql
DELETE FROM table1
```

Видалити запис з id=1 у таблиці table1

```sql
DELETE FROM table1
    WHERE id = 1;
```
Можна, наприклад, так видалити останній запис
```sql
-- Видалення останнього запису
DELETE FROM table1
    ORDER BY id DESC
    LIMIT 1;
```

Розберемо деякі модифікатори

WHERE – це фільтрація, умови.

ORDER BY – порядок видалення DESC навпаки (правила сортування).

LIMIT – скільки записів видалити.

Не всі знають, що можна видаляти відразу з кількох таблиць одночасно.

```sql
DELETE t1, t2 FROM t1, t2, t3
    WHERE t1.id = t2.id AND t2.id = t3.id;
```
<a href="#top">⬅️ Back to top</a>

## UPDATE
Оновлення запису

```sql
UPDATE table1
    SET 
        name = CONCAT(name, ' *'),
        price = price * 1.5
    WHERE id BETWEEN 10 AND 30;
```
Оновіть ті записи id, які в діапазоні 10 і 30, додати до імені зірочку і збільшити суму на 1.5.

Розберемо сам запит UPDATE ім'я_таблиці далі SET колонка = нове_значення [, колонка2 = нове_значення] і необов'язково WHERE умова

Якщо не задати WHERE, то оновляться всі записи в таблиці.

```sql
--  Auto-generated SQL script #202503061336
UPDATE quizdb.questions
	SET answer='A',answer_options='A;B;C;D'
	WHERE id=18;
```

Також можна оновлювати записи відразу у кількох таблицях.

```sql
UPDATE items, months
    SET 
        items.price = months.price
    WHERE items.id = months.id;
```
<a href="#top">⬅️ Back to top</a>

## SELECT
Один з найрізносторонніших операторів, який дозволяє нам отримувати дані.

Посилання на документацію https://mariadb.com/kb/en/select/

Також в мові SQL є змінні, вони завантажуються, використовуючи конструкцію SET та існують до кінця сеансу.
```sql
SET @a = "Привіт світ";
SELECT @a;
```

Можна замість колонок передати *, це поверне всі колонки, які є в таблиці, але це погана практика і точно не варто так робити в продакшені.

```sql
SELECT * FROM mytable;
```
### DISTINCT 
якщо ми не хочемо отримувати записи, що повторюються

```sql
-- Виведення без повторення записів.
SELECT DISTINCT room FROM lessons;
```

Пару прикладів запитів із таблицями
```sql
SELECT last_name, first_name 
    FROM mytable;
```
```sql
SELECT CONCAT(last_name,', ',first_name) AS full_name
    FROM mytable;
```
<a href="#top">⬅️ Back to top</a>

## Функції агрегації (агрегати).
Відмінність агрегатних від скалярних функцій у тому, що вони працюють не з одним значенням, і з набором даних. Як тільки ми у SELECT напишемо функцію агрегації, то у нас буде збиратися багато записів в один.

Посилання на документацію https://mariadb.com/kb/en/aggregate-functions/

    AVG() – середнє значення

    COUNT() – кількість записів

    COUNT(DISTINCT results) – кількість без повторень

    GROUP_CONCAT() – склеювання рядків, конкатенує записи.

    MIN() – мінімальне з набору

    MAX() – максимальне з набору

    STD() – стандартні відхилення, статистична функція.

    SUM() – складає значення (чисельне) в одне з усіх зазначених полів.

    VARIANCE() – розкид, відхиляючись від середнього.

Порахуємо кількість записів без повторення імен людей.
```sql
SELECT COUNT(DISTINCT name) as count
    FROM persons;
```
<a href="#top">⬅️ Back to top</a>

## Клаузули
Такі оператори як ORDER BY у запиті називаються предикати або клаузули. Клаузули працюють не тільки з SELECT, але і з іншими, UPDATE, DELETE.

Якщо бути буквоїдом, то предикат - це фільтр, отже предикатом є лише WHERE, решта - клаузула (ORDER BY тощо).

### ORDER BY
Сортує відповідь.

```sql
SELECT college, region AS r, seed AS s
    FROM tournament
    ORDER BY r, s;
```

Для ORDER BY є:

ASC – це за зростанням

DESC – за спаданням від слова descent (десант, падають вниз).

```sql
-- в порядку зростання
SELECT id, name, phone
    FROM persons
    ORDER BY id ASC;
```
<a href="#top">⬅️ Back to top</a>

### LIMIT
Повертає обмежену кількість записів. Часто використовується з ORDER BY

### OFFSET
Зміщує видачу, скажімо, наприклад, щоб зробити пагінацію. Майже завжди використовується з LIMIT

```sql
SELECT t.id, t.idx , t.legend FROM tiers t  LIMIT 5 OFFSET 5;
```
По індексованим полям зміщення працює швидше (якщо зміщення > 10000)
```sql
SELECT t.id, t.idx , t.legend FROM tiers t  WHERE id > 55 LIMIT 5;
```
<a href="#top">⬅️ Back to top</a>

### WHERE
Це умова як if працює просто, сервер перевіряє всі записи, до якого підходить вираз у WHERE, у ньому може бути дорівнює, більше, менше тощо.

```sql
SELECT col_name
    FROM tb1_name
    WHERE col_name > 0;

SELECT *
    FROM tb1
    WHERE col1 > 0
    AND col2 = 'hello‘;
```

### BETWEEN 
– це в діапазоні між одним (and) та іншим включно.
```sql
SELECT name, birthday
    FROM persons
    WHERE birthday BETWEEN '2010-01-12' AND '2010-06-22';
```
<a href="#top">⬅️ Back to top</a>

### IS NULL 
 пошук за порожнім значенням.
 ```sql
 SELECT name, birthday, salary
    FROM persons
    WHERE salary IS NULL;
```

### IN
 – пошук у множині, IN (1, 3, 5) замінює 1 OR 3 OR 5
 ```sql
 SELECT id, name, birthday, salary
    FROM persons
    WHERE id IN (1, 3, 5);
```
<a href="#top">⬅️ Back to top</a>

### LIKE 
– вибірка за збігом

% - замінює будь-яку кількість символів як '*' у reqExp. Під Jack% потрапляє і Jackson, і Jackson Smit тощо.

_ - замінює один символ.

```sql
SELECT name, birthday, salary
    FROM persons
    WHERE name LIKE 'Jack%';
```

### REGEXP 
– регулярний вираз, за всіма правилами регулярних виразів [0-9] тобто POSIX регулярки.
```sql
SELECT name, birthday, salary
    FROM persons
    WHERE birthday REGEXP '1971-[0-9]{2}-16';
```
<a href="#top">⬅️ Back to top</a>

### GROUP BY
Це групування даних, наприклад, ми робимо звіт і нам потрібна загальна сума за роками, днями тощо.

```sql
SELECT YEAR(date) as year, SUM(profit)
    FROM reports
    GROUP BY year;
```

Count Occurrences of Each ENUM Value 
```sql
SELECT question_type, COUNT(*) AS count
FROM questions q 
GROUP BY q.question_type;	
```

Коли пишеться комбінація GROUP BY, агрегація створюється для кожної з побудованих груп. Тобто він всі записи за 1970 злив в одну і для нього порахував суми тощо. І ми отримали відсортовану відповідь за роками із загальною сумою за рік.

Те, за чим ми групуємо, обов'язково повинно бути на виході. Тобто не можна групувати по колонках, які не виводяться, для інших клаузул це не важливо.

### HAVING

HAVING – це фільтр після групування, іноді нам потрібно після того, як інформація була відібрана та згрупована, зробити повторну фільтрацію за результатами цього групування.

```sql
SELECT YEAR(date) as year, SUM(profit) as profit
    FROM reports
    GROUP BY year
    HAVING profit > 2000000000;
```
Вибираємо роки, в яких дохід більший, ніж 2 мільярди.

<a href="#top">⬅️ Back to top</a>

### Порядок написання

    WHERE

    GROUP

    HAVING

    ORDER

    LIMIT

Це не обов'язково, але настійно рекомендується, оскільки саме в цьому порядку сервер виконує їх.

<a href="#top">⬅️ Back to top</a>

## Об'єднання таблиць

### JOIN

Output only rows satisfied condition under ON

```sql
SELECT  ROW_NUMBER() OVER (ORDER BY t.idx) AS row_num, t.idx, t.legend, t.boundQuestion, q.label, q.question_type FROM tiers t 
	JOIN questions q ON t.boundQuestion = q.label
```

### LEFT JOIN

Output all rows from tier table and  matched ON condition rows from questions

```sql
SELECT  ROW_NUMBER() OVER (ORDER BY t.idx) AS row_num, t.idx, t.legend, t.boundQuestion, q.label, q.question_type FROM tiers t 
	LEFT JOIN questions q ON t.boundQuestion = q.label
```

### RIGHT JOIN

Output all rows from questions table and  matched ON condition rows from tiers

```sql
SELECT  ROW_NUMBER() OVER (ORDER BY t.idx) AS row_num, t.idx, t.legend, t.boundQuestion, q.label, q.question_type FROM tiers t 
	LEFT JOIN questions q ON t.boundQuestion = q.label
```

<a href="#top">⬅️ Back to top</a>

## Об'єднання запиту та підзапити

### UNION
  
Output similar data from differnt table in one table

```sql
SELECT prod.id, prod.product, prod.type 
    FROM (
        (SELECT id, product, 'pc' as type FROM pc)
        UNION
        (SELECT id, product, 'mobile' as type FROM mobile)
    ) as prod
```

### IN, ANY, SOME, NOT

```sql
SELECT e.id as employeeId, e.firstName, e.lastName, e.departmentId
    FROM employees AS e
    WHERE e.departmentId IN ( SELECT id FROM departments WHERE id = 1 );
```

```sql
 -- Пошук країн, у яких немає міст мільйонників
SELECT country.name
    FROM country
    WHERE country.code NOT IN -- або <> ALL
(
    SELECT city.countryCode
    FROM city
    WHERE city.population >= 1000000
)
LIMIT 10;
```

Розберемо ще підзапити, ми хочемо порахувати скільки міст у кожній із країн.

```sql
SELECT country.name, (SELECT COUNT(*) FROM city WHERE city.countryCode = country.code) as cityCount
    FROM country
    ORDER BY cityCount DESC
    LIMIT 10;
```

Порахуємо скільки входжень кожного типу запитання є серед щаблів із зв'язаними запитаннями:
```sql
SELECT question_type, COUNT(*) AS count
FROM (SELECT  ROW_NUMBER() OVER (ORDER BY t.idx) AS row_num, t.idx, t.legend, t.boundQuestion, q.label, q.question_type FROM tiers t 
	JOIN questions q ON t.boundQuestion = q.label) as tq
GROUP BY tq.question_type;
```
<a href="#top">⬅️ Back to top</a>

## VIEW (перегляди)

VIEW - це запит, який ми зберегли як об'єкт бази даних, він такий самий об'єкт як і таблиця. Тобто фіктивна таблиця, насправді це не таблиця, а запит, який поводиться як таблиця. Сервер ніби імітує запит під таблицю. Працює швидше, ніж звернення до реальних таблиць. І вони можуть бути оновлюваними.

```sql
CREATE VIEW employeeDepartments AS
    SELECT e.id as employeeId, e.firstName, e.lastName, e.departmentId, d.id as idInDepartments, d.department
    FROM employees AS e
    INNER JOIN departments AS d ON d.id = e.departmentId;
```

I can create redy for play view from tiers and questions tables. It will contain tiers idx and legend fields along with questions filds bound to tier. Then I can reffer to view instead doing joined query.

VIEW доступний тільки для читання, якщо:

Функції агрегації (SUM(), MIN(), MAX(), COUNT())

Підзапити

DISTINCT

GROUP BY

HAVING

UNION

JOIN

ALGORITHM = TEMPTABLE

<a href="#top">⬅️ Back to top</a>

# Процедури, що зберігаються

 In MariaDB, procedures, functions, and triggers exist permanently after creation unless explicitly dropped.

## PROCEDURE

Посилання на документацію https://mariadb.com/kb/en/create-procedure/

Замість зберігання запиту, що часто використовується, клієнти можуть посилатися на відповідну процедуру, що зберігається. Під час виклику процедури, що зберігається, її вміст відразу ж обробляється сервером. Один єдиний оператор дозволяє викликати складний сценарій, який міститься в процедурі, що зберігається, що дозволяє уникнути пересилання через мережу сотень команд.

Процедури можуть мати параметри двох типів: IN, OUT

IN – вхідний параметр, якщо ми забули його вказати, саме він і матиметься на увазі.

OUT – вихідні параметри, дані, які будуть повернуті. Ми можемо під час виклику підставити туди змінну і в неї запишеться результат. Це класична передача даних за посиланням.

Також, якщо SELECT повертає якесь скалярне значення, ми можемо записати його не на вихід, а в змінну, використовуючи модифікатор INTO

```sql
SELECT COUNT(*) INTO record_count
    FROM course_by_dates_results;
```

Процедурам, що зберігаються, прийнято давати імена, які починаються з sp або st

```sql
-- Які заняття були у вказаний місяць та рік
DELIMITER |
CREATE OR REPLACE PROCEDURE sp_course_by_dates
    (IN date_start DATE, IN date_end DATE, OUT record_count INT) -- <- Parameters
BEGIN
  -- Створимо тимчасову таблицю запиту
  CREATE TEMPORARY TABLE course_by_dates_results
    SELECT courses.id, courses.title
      FROM courses
      INNER JOIN lessons ON courses.id = lessons.courseId
    WHERE lessons.lesson_date BETWEEN date_start AND date_end;
  -- Підрахуємо, скільки в ній записано в змінну.
  SELECT COUNT(*) INTO record_count
    FROM course_by_dates_results;
    -- Повернемо результат
  SELECT *
    FROM course_by_dates_results;
  -- Видалимо тимчасову таблицю
  DROP TEMPORARY TABLE course_by_dates_results;
END;
|
DELIMITER ;
```
А тепер зробимо запит до процедури, що зберігається.

```sql
CALL sp_course_by_dates('2021-09-15','2021-09-21',@count);
SELECT @count;
```
## Змінні (Variables)

В мові SQL є змінні, вони завантажуються, використовуючи конструкцію SET та існують до кінця сеансу.

```sql
SET @a = "Привіт світ";
SELECT @a;
```

Також можна створювати не глобальні змінні, а локальні за допомогою команди DECLARE.
Всі змінні в процедурі без собачки (@) поза процедурою з собачкою.

```sql
DECLARE xname VARCHAR(5) DEFAULT 'bob';
```

<a href="#top">⬅️ Back to top</a>

## Визначення обробників (error HANDLER)

В SQL немає обробки помилок, як наш улюблений try...catch. І тому його зробили за допомогою спеціальних обробників.

Посилання на документацію https://mariadb.com/kb/en/declare-handler/

```sql
DECLARE handler_type HANDLER
    FOR condition_value [, condition_value] ...
    statement

handler_type:
    CONTINUE
  | EXIT 
  | UNDO

condition_value:
    SQLSTATE [VALUE] sqlstate_value
  | condition_name
  | SQLWARNING
  | NOT FOUND
  | SQLEXCEPTION
  | mariadb_error_code
```

condition_value:

Помилки прив'язуються до коду --https://mariadb.com/kb/en/mariadb-error-codes/ - коди помилок.

Приклад обробника, ми зробимо обробник на SQLSTATE '23000', помилка 23000 - це дублікат унікального ключа.

```sql
-- Drop and recreate the table
DROP TABLE IF EXISTS t;
CREATE TABLE t (s1 INT PRIMARY KEY);

-- Create a table for logging errors
DROP TABLE IF EXISTS error_log;
CREATE TABLE error_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    error_message VARCHAR(255),
    error_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER |

-- Create a stored procedure with error logging
CREATE OR REPLACE PROCEDURE handlerdemo ()
BEGIN
    DECLARE CONTINUE HANDLER FOR SQLSTATE '23000' 
    INSERT INTO error_log (error_message) VALUES ('Duplicate key error in table t');

    INSERT INTO t VALUES (1); -- First insert (should succeed)
    INSERT INTO t VALUES (1); -- Second insert (should fail and log the error)
    INSERT INTO t VALUES (2); -- Third insert (should succeed)
END;
|

DELIMITER ;

-- Call the procedure
CALL handlerdemo();

-- Check the table contents
SELECT * FROM t;         -- To see successfully inserted records
SELECT * FROM error_log; -- To check logged errors
```

<a href="#top">⬅️ Back to top</a>

## CURSOR

In MariaDB, a cursor is a database object used to retrieve and process rows from a result set one at a time within a stored procedure. Cursors are mainly used when you need to handle large datasets or perform row-by-row operations, which cannot be easily achieved with a standard SELECT statement.

Визначаються так

```sql
DECLARE cursor_name CURSOR FOR якийсь_запит;
```

Також важливо зробити обробник, коли записи закінчилися.

```sql
DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
```

✅ Cursor + Dynamic SQL (Prepared Statements)

```sql
DELIMITER |

CREATE PROCEDURE dynamic_update()
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE emp_id INT;
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE sql_query VARCHAR(255);

    DECLARE emp_cursor CURSOR FOR 
    SELECT id, salary FROM employees;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    OPEN emp_cursor;

    read_loop: LOOP
        FETCH emp_cursor INTO emp_id, emp_salary;
        
        IF done = 1 THEN 
            LEAVE read_loop;
        END IF;

        -- Variant 1 Construct dynamic SQL
        SET sql_query = CONCAT('UPDATE employees SET salary = ', emp_salary * 1.10, ' WHERE id = ', emp_id);
        
        -- Prepare and execute dynamic SQL
        PREPARE stmt FROM sql_query;
        EXECUTE stmt;

       -- Variant 2 Apply a conditional update
        IF emp_salary < 6000 THEN
            UPDATE employees SET salary = emp_salary * 1.10 WHERE id = emp_id;
        END IF;

        DEALLOCATE PREPARE stmt;
    END LOOP;

    CLOSE emp_cursor;
END;
|

DELIMITER ;

-- Run the procedure
CALL dynamic_update();

-- Check updated salaries
SELECT * FROM employees;

```

🔹 When to Use Cursors in MariaDB

✅ When processing rows sequentially (e.g., batch updates).

✅ When complex row-by-row logic is needed (e.g., calculations per row).

✅ When a simple UPDATE or JOIN is not enough.

⚠️ When NOT to Use Cursors

❌ When a single SQL query can achieve the same result efficiently.

❌ When working with large datasets (cursors can be slow).

❌ When using MariaDB’s GROUP BY, HAVING, JOIN can replace row-by-row processing.

🔹 Alternative Approach (Without Cursor)
Instead of using a cursor, you could simply update all salaries in one query:

```sql
UPDATE employees SET salary = salary * 1.10;
```

<a href="#top">⬅️ Back to top</a>

## Функції, що зберігаються (FUNCTION)

Якщо процедура, що зберігається, повертає таблицю або нічого, то функція, що зберігається, повертає скаляр.

Створюється також як і процедура, що зберігається, але обов'язково потрібно вказати RETURNS і тип, наприклад INT


***Як бачимо, вже використовується не CALL для виклику, а SELECT, а значить ми можемо це використовувати всередині запитів.***

```sql
DELIMITER |
CREATE OR REPLACE FUNCTION hello (s VARCHAR(20)) RETURNS VARCHAR(100)
  BEGIN
    RETURN CONCAT('Hello, ',s,'!');
  END |
DELIMITER ;

SELECT hello('world');
```

**Example 1: Creating and Using a Function in a Query**

*Step 1: Create the employees Table*

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10,2)
);

INSERT INTO employees VALUES
(1, 'Alice', 5000),
(2, 'Bob', 7000),
(3, 'Charlie', 4500);
```
*Step 2: Create a Function calculate_bonus*

This function calculates a bonus as 15% of salary if the salary is above 5000, otherwise 10%.

```sql
DELIMITER |

CREATE FUNCTION calculate_bonus(salary DECIMAL(10,2)) 
RETURNS DECIMAL(10,2) DETERMINISTIC
BEGIN
    DECLARE bonus DECIMAL(10,2);
    IF salary > 5000 THEN
        SET bonus = salary * 0.15;
    ELSE
        SET bonus = salary * 0.10;
    END IF;
    RETURN bonus;
END;
|

DELIMITER ;
```

*Step 3: Use the Function in a Query*

Now, we can use calculate_bonus() inside a SELECT statement.

```sql
SELECT id, name, salary, calculate_bonus(salary) AS bonus
FROM employees;
```

✅ Function is called for each row dynamically

✅ Works like a built-in function

*🔹 Example 2: Using a Function in WHERE Clause*

You can also use the function inside a WHERE condition.

```sql
SELECT * FROM employees 
WHERE calculate_bonus(salary) > 1000;
```
✅ User-defined functions allow custom calculations inside queries.

✅ They can be used in **SELECT, WHERE, GROUP BY, ORDER BY, etc.**

✅ Functions must be deterministic (return the same result for the same input).

<a href="#top">⬅️ Back to top</a>

## PREPARE (динамічний SQL)

Так само як у JS ми можемо викликати eval, щоб виконати рядок як код, такий самий оператор є і в SQL.

Як воно працює

PREPARE – готуємо.

EXECUTE – виконуємо.

DEALLOCATE PREPARE – звільняємо процес. Можна також написати DROP PREPARE

Подивимося на прикладі

```sql
PREPARE ім'я_процесу FROM @ім'я_змінної;
EXECUTE ім'я_процесу;
DEALLOCATE PREPARE ім'я_процесу;
```

Спробуємо створити процедуру з PREPARE

```sql
DROP TABLE IF EXISTS t1;
CREATE TABLE t1 (s1 INT, PRIMARY KEY (s1));
INSERT INTO t1 VALUES
    (1),
    (2),
    (3);

DELIMITER |
CREATE OR REPLACE PROCEDURE `test_prepare`(IN t TEXT)
BEGIN
    SET @i = CONCAT('SELECT count(*) FROM ',t);
    PREPARE i FROM @i;
    EXECUTE i;
    DEALLOCATE PREPARE i;
END |
DELIMITER ;

CALL test_prepare('t1');
```
<a href="#top">⬅️ Back to top</a>

## TRIGGER

Тригер (англ. trigger) - це процедура, що зберігається, особливого типу, яку користувач не викликає безпосередньо, а виконання якої обумовлено певною модифікацією даних в заданій таблиці або стовпці реляційної бази даних.

Тригер запускається сервером автоматично при спробі зміни даних таблиці, з якою він пов'язаний.

Усі здійснені ним модифікації даних розглядаються як одна транзакція. У разі виявлення помилки або порушення цілісності даних відбувається відкат цієї транзакції.

Момент запуску тригера визначається за допомогою ключових слів **BEFORE** (тригер запускається до виконання пов'язаної з ним події; наприклад, до додавання запису) або **AFTER** (після події).

Давайте створимо тригер, перед цим необхідно створити тестові таблиці.

```sql
DROP TABLE IF EXISTS product;
CREATE TABLE product
(
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    title VARCHAR(30) NOT NULL DEFAULT '',
    price FLOAT NOT NULL DEFAULT 0,
    que FLOAT NOT NULL DEFAULT 0,
    PRIMARY KEY (id)
);

DROP TABLE IF EXISTS product_log;
CREATE TABLE product_log
(
    now DATETIME,
    user VARCHAR(100) NOT NULL DEFAULT '',
    `id` INT UNSIGNED,
    `old price` FLOAT,
    `new price` FLOAT,
    process VARCHAR(30)  NOT NULL DEFAULT ''
);

INSERT INTO product
    VALUE
    (1,'Товар 1',123,200),
    (2,'Товар 2',150,200),
    (3,'Товар 3',187,300),
    (4,'Товар 4',300,400),
    (5,'Товар 5',225,500);
```

А тепер створимо тригер

```sql
-- тригер на оновлення
DROP TRIGGER IF EXISTS tg_product_update;
DELIMITER |
CREATE TRIGGER tg_product_update AFTER UPDATE ON product 
    FOR EACH ROW
    BEGIN
    IF OLD.price <> NEW.price THEN
    INSERT INTO product_log
      VALUES(NOW(), USER(),OLD.id, OLD.price, NEW.price, 'UPDATE');
    END IF;
    END|
DELIMITER ;

-- тригер на додавання
DROP TRIGGER IF EXISTS tg_product_insert;
DELIMITER |
CREATE TRIGGER tg_product_insert AFTER INSERT ON product 
    FOR EACH ROW
    BEGIN
    INSERT INTO product_log
      VALUES(NOW(), USER(),NEW.id,NULL,NEW.price, 'INSERT');
    END|
DELIMITER ;

-- тригер на видалення
DROP TRIGGER IF EXISTS tg_product_delete;
DELIMITER |
CREATE TRIGGER tg_product_delete BEFORE DELETE ON product 
    FOR EACH ROW
    BEGIN
    INSERT INTO product_log
      VALUES(NOW(), USER(),OLD.id,OLD.price,NULL,'DELETE');
    END|
DELIMITER ;
```

Почнемо змінювати записи

```sql
-- Оновлення
UPDATE product SET price = 200 WHERE id = 2;

-- Додавання
INSERT INTO product(title, price, que) VALUE ('Новий Товар',123,200);

-- Видалення
DELETE FROM product
    ORDER BY id DESC
    LIMIT 1;
    
-- перевіряємо лог
SELECT * FROM product_log;
```
<a href="#top">⬅️ Back to top</a>

# Керуючі конструкції

## Оператор IF

```sql
IF done THEN
    LEAVE read_loop;
END IF;
IF x < y THEN
    INSERT INTO c3 VALUES (x);
ELSE
    INSERT INTO c3 VALUES (y);
END IF;
```

## Оператор CASE

```sql
CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list] ...
    [ELSE statement_list]
END CASE
```

## Оператор LOOP

Це цикл без умов.

Ми даємо мітку, наприклад start_loop , але це не обов'язково, і між LOOP та END LOOP буде цикл, доки ми його не зупинимо оператором LEAVE

```sql
start_loop: LOOP
  IF done THEN
    LEAVE start_loop;
  END IF;
  IF x < y THEN
   INSERT INTO c3 VALUES (x);
  ELSE
    INSERT INTO c3 VALUES (y);
  END IF;
END LOOP;
```

## Оператор REPEAT

Класичний приклад циклу з пост-умовою (do while). Якщо умова стає істинною, викидає із циклу.

```sql
CREATE PROCEDURE dorepeat(p1 INT)
BEGIN
    SET @x = 0;
    REPEAT SET @x = @x + 1; UNTIL @x > p1 END REPEAT;
END
```

## Оператор WHILE

Цикл із передумовою. Працює, доки умова істинна, до хибності.

```sql
CREATE PROCEDURE dowhile()
BEGIN
  DECLARE v1 INT DEFAULT 5;

  WHILE v1 > 0 DO
    ...
    SET v1 = v1 - 1;
  END WHILE;
END
```

## Оператор FOR

```sql
FOR i IN 1..3
DO
  INSERT INTO t1 VALUES (i);
END FOR;
```

<a href="#top">⬅️ Back to top</a>

# Оптимізація та обслуговування

## EXPLAIN

Давайте оптимізуємо таблицю, я вже показував приклад з великою таблицею і коли ми додали їй індекс, ми побачили приріст продуктивності, давайте ж визначимося коли індекс потрібен.

Для цього є оператор EXPLAIN.

```sql
EXPLAIN
SELECT  ROW_NUMBER() OVER (ORDER BY t.idx) AS row_num, t.idx, t.legend, t.boundQuestion, q.label, q.question_type FROM tiers t 
 JOIN questions q ON t.boundQuestion = q.label
 WHERE t.boundQuestion = 'd250';
 ```

 |id|select_type|table|type|possible_keys|key|key_len|ref|rows|Extra|
|--|-----------|-----|----|-------------|---|-------|---|----|-----|
|1|SIMPLE|q|const|questions_unique|questions_unique|61|const|1|Using temporary|
|1|SIMPLE|t|ref|tiers_questions_FK|tiers_questions_FK|61|const|1|Using index condition|

***Type – спосіб вибірки, index:***

    Const – таблиця має лише один відповідний рядок, який проіндексований. Це найшвидший тип з'єднання, тому що таблиця читається лише один раз, і значення рядка може сприйматися при подальших з'єднаннях як константа.

    ref → The ref access type means MariaDB is using an indexed lookup (good performance).

    all – Для знаходження відповідних рядків використовується сканування всієї таблиці. Це найгірший тип з'єднання і зазвичай вказує на відсутність відповідних індексів у таблиці.

***Possible_key*** – можу використовувати індекси.

***Key*** – який індекс використовується.

***key_len*** - Key length in bites. 

***ref: const*** → Since we are filtering by t.boundQuestion = 'd250', it becomes a constant lookup.

***rows: 1*** → Only one row needs to be checked ( very efficient!).

***Extra: Using temporary*** → A temporary table is used, likely due to the ROW_NUMBER() function.

***Extra: Using index condition*** → The condition (t.boundQuestion = 'd250') is evaluated directly using the index, improving performance.

✅ Potential Optimization

The temporary table is created due to ROW_NUMBER(), which might affect performance for large datasets.

### Основні концепції оптимізації
Як розставляти індекси

Зв'язок. Будь-який зв'язок повинен бути проіндексований. Але не забуваємо, що первинний ключ сам по собі вже індекс. Коли ми пишемо JOIN, це вже зв'язок, він у нас на схемі, тож потрібен індекс. Тобто, за логікою, якщо ми розуміємо, що це зовнішній ключ, то індексуємо.

Ставимо індекси скрізь де є WHERE, але пам'ятаємо, що REGEXP і LIKE не індексуються. Але якщо WHERE = (< >), то індексуємо. Аналогічно GROUP BY та ORDER BY
А потім кожен наш запит проганяти EXPLAIN і одразу дивимось на possible_keys, якщо індекс використовується, ставимо собі мітку де – те, що використовується.

Ось який індекс не отримав підтвердження, такий сміливо дропаємо.

<a href="#top">⬅️ Back to top</a>


---
[[Resources/Resources Dashboard|Resources]] / **[[Resources/SQL MariaDB|SQL MariaDB]]***