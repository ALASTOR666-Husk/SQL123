# Пошаговое выполнение задания MySQL + Docker

Я изучил твой файл `MySQL.md`. Ниже распишу выполнение по порядку: от запуска MySQL в Docker до создания таблиц, выполнения SQL-запросов и сохранения резервной копии базы данных.

Будем работать в Windows через Git Bash (MINGW64) и Docker Desktop. Команды приведены так, чтобы можно было копировать их по очереди.

## Что нужно сделать

Чек-лист выполнения

0 из 10

1\. Подготовить папку проекта и файл my.cnf

2\. Запустить контейнер MySQL

3\. Подключиться к MySQL

4\. Создать базу данных и таблицы

5\. Добавить данные и проверить CRUD-операции

6\. Выполнить фильтрацию, сортировку и агрегирование

7\. Изучить JOIN, подзапросы и UNION

8\. Изменить данные и структуру таблиц

9\. Создать дамп базы данных

10\. Проверить результат и сохранить скриншоты

## Шаг 1. Подготовь окружение

### 1.1. Запусти Docker Desktop

1. Открой Docker Desktop.
2. Дождись, пока Docker Engine запустится.
3. Открой Git Bash.
4. Выполни команду:

```
docker --version
```

Проверь, что Docker отвечает:

```
docker info
```

Если команды выводят информацию о Docker, можно продолжать. Если появляется ошибка подключения к Docker daemon, сначала запусти Docker Desktop.

### 1.2. Создай папку проекта

В Git Bash выполни:

```
mkdir -p ~/my-mysql
cd ~/my-mysql
```

Проверь расположение:

```
pwd
```

Внутри этой папки будут конфигурация MySQL и резервная копия базы данных.

### 1.3. Создай файл `my.cnf`

Выполни:

```
nano my.cnf
```

Вставь следующее содержимое:

```
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4
```

Сохрани файл в `nano`:

1. Нажми `Ctrl + O`.
2. Нажми `Enter`.
3. Нажми `Ctrl + X`.

Проверь, что файл создан:

```
cat my.cnf
```

Зачем это нужно? Настройки `utf8mb4` позволяют корректно хранить и отображать русский текст и другие Unicode-символы.

## Шаг 2. Запусти контейнер MySQL

Находясь в папке `~/my-mysql`, выполни:

```
docker run --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=123 \
  -e LANG=C.UTF-8 \
  -v mysql_data:/var/lib/mysql \
  -v "$(pwd -W)/my.cnf:/etc/mysql/conf.d/my.cnf:ro" \
  -d mysql:latest
```

В этой команде:

- `--name my-mysql` — имя контейнера.
- `MYSQL_ROOT_PASSWORD=123` — учебный пароль пользователя `root`.
- `mysql_data` — хранилище данных, сохраняющееся при удалении самого контейнера.
- `my.cnf` — настройки кодировки.
- `-d` — запуск в фоновом режиме.
- `mysql:latest` — официальный образ MySQL.

Пароль 123 подходит только для учебного примера. Для реального сервера используй надёжный пароль.

### 2.1. Проверь запуск

```
docker ps
```

В списке должен появиться контейнер `my-mysql`.

Затем посмотри журнал запуска:

```
docker logs my-mysql
```

MySQL может запускаться некоторое время. Для проверки готовности выполни:

```
docker exec my-mysql mysqladmin ping -uroot -p123
```

Ожидаемый результат:

```
mysqld is alive
```

### Если контейнер не запускается

Если получишь ошибку `Conflict. The container name "/my-mysql" is already in use`, контейнер с таким именем уже существует. Проверь его:

```
docker ps -a
```

Если он остановлен, попробуй:

```
docker start my-mysql
```

Если контейнер уже существует и работает, не запускай `docker run` повторно — переходи к следующему шагу.

## Шаг 3. Подключись к MySQL

Выполни в Git Bash:

```
docker exec -it my-mysql mysql --default-character-set=utf8mb4 -uroot -p
```

Когда появится запрос:

```
Enter password:
```

Введи пароль:

```
123
```

При вводе пароль может не отображаться — это нормально.

Если подключение прошло успешно, появится приглашение:

```
mysql>
```

Теперь ты работаешь внутри MySQL. Важно различать две среды:

- Команды `docker ...` вводятся в Git Bash.
- Команды `CREATE DATABASE`, `SELECT`, `INSERT` и другие SQL-команды вводятся после появления `mysql>`.

### 3.1. Проверь кодировку

Введи:

```
SHOW VARIABLES LIKE 'character_set%';
```

Убедись, что параметры `character_set_client`, `character_set_connection` и `character_set_results` имеют значение `utf8mb4`.

Затем посмотри существующие базы:

```
SHOW DATABASES;
```

В списке будут системные базы MySQL. Пока не удаляй их.

## Шаг 4. Создай учебную базу данных

Внутри `mysql>` выполни:

```
CREATE DATABASE sql_practice
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
```

Выбери созданную базу:

```
USE sql_practice;
```

Проверь результат:

```
SELECT DATABASE();
```

Ожидаемый результат — `sql_practice`.

Если база уже существует, не создавай её повторно. Просто выполни `USE sql_practice;`.

## Шаг 5. Создай таблицы

Для практики в задании используются две связанные таблицы:

- `customers` — клиенты.
- `orders` — заказы клиентов.

Скопируй весь следующий блок в MySQL:

```
CREATE TABLE customers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    city VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) DEFAULT 'new',
    FOREIGN KEY (customer_id) REFERENCES customers(id)
) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

Что означают основные элементы:

| Элемент          | Назначение                                   |
| ---------------- | -------------------------------------------- |
| `id`             | Уникальный идентификатор записи              |
| `AUTO_INCREMENT` | Автоматическое увеличение ID                 |
| `PRIMARY KEY`    | Первичный ключ                               |
| `NOT NULL`       | Обязательное значение                        |
| `UNIQUE`         | Запрет повторяющихся email                   |
| `FOREIGN KEY`    | Связь заказа с клиентом                      |
| `DECIMAL(10, 2)` | Денежная сумма с двумя знаками после запятой |

### 5.1. Проверь таблицы

```
SHOW TABLES;
```

Должны появиться `customers` и `orders`.

Посмотри структуру каждой таблицы:

```
DESCRIBE customers;
DESCRIBE orders;
```

## Шаг 6. Добавь тестовые данные

Сначала заполним клиентов:

```
INSERT INTO customers (name, email, city) VALUES
('Иван Иванов', 'ivan@example.com', 'Москва'),
('Пётр Петров', 'petr@example.com', 'Санкт-Петербург'),
('Мария Сидорова', 'maria@example.com', 'Москва'),
('Алексей Смирнов', 'alex@example.com', 'Казань'),
('Ольга Кузнецова', 'olga@example.com', 'Новосибирск');
```

Теперь добавим заказы:

```
INSERT INTO orders
    (customer_id, order_date, amount, status)
VALUES
    (1, '2026-10-01', 1500.00, 'paid'),
    (1, '2026-10-05', 2300.50, 'paid'),
    (2, '2026-10-03', 800.00, 'new'),
    (3, '2026-10-07', 4200.00, 'paid'),
    (3, '2026-10-10', 1100.00, 'cancelled'),
    (4, '2026-10-12', 950.00, 'new');
```

В таблице заказов `customer_id` ссылается на `id` из таблицы клиентов. Поэтому сначала добавляются клиенты, а уже потом их заказы.

### 6.1. Проверь, что данные добавились

```
SELECT * FROM customers;
SELECT * FROM orders;
```

Должны отображаться 5 клиентов и 6 заказов.

Если хочешь сохранить результаты для отчёта, сделай скриншот таблиц с данными.

## Шаг 7. Выполни CRUD-операции

CRUD — это четыре основных действия с данными: создание, чтение, обновление и удаление.

### 7.1. Create — добавление записи

```
INSERT INTO customers (name, email, city)
VALUES ('Павел Иванов', 'pavel@example.com', 'Москва');
```

Проверь:

```
SELECT * FROM customers;
```

### 7.2. Read — чтение записей

Получить всех клиентов:

```
SELECT * FROM customers;
```

Получить только имена и города:

```
SELECT name, city FROM customers;
```

Найти клиента по ID:

```
SELECT * FROM customers WHERE id = 1;
```

### 7.3. Update — обновление записи

```
UPDATE customers
SET city = 'Екатеринбург'
WHERE id = 4;
```

Проверь:

```
SELECT * FROM customers WHERE id = 4;
```

### 7.4. Delete — удаление записи

Для проверки можно удалить добавленного ранее клиента:

```
DELETE FROM customers
WHERE email = 'pavel@example.com';
```

Проверь:

```
SELECT * FROM customers;
```

Важно: всегда проверяй условие `WHERE` перед выполнением `UPDATE` и `DELETE`. Если забыть условие, можно изменить или удалить все записи таблицы.

## Шаг 8. Освой фильтрацию, сортировку и ограничение

Следующие запросы выполняй по одному или целым блоком.

### 8.1. Фильтрация через `WHERE`

```
-- Клиенты из Москвы
SELECT * FROM customers
WHERE city = 'Москва';

-- Заказы дороже 1000
SELECT * FROM orders
WHERE amount > 1000;

-- Оплаченные заказы дороже 1500
SELECT * FROM orders
WHERE status = 'paid' AND amount > 1500;

-- Новые или отменённые заказы
SELECT * FROM orders
WHERE status = 'new' OR status = 'cancelled';
```

### 8.2. Сортировка через `ORDER BY`

```
-- Клиенты по имени
SELECT * FROM customers
ORDER BY name ASC;

-- Заказы от самых дорогих к самым дешёвым
SELECT * FROM orders
ORDER BY amount DESC;
```

`ASC` — по возрастанию, `DESC` — по убыванию.

### 8.3. Ограничение через `LIMIT`

```
SELECT * FROM orders
ORDER BY amount DESC
LIMIT 3;
```

Запрос вернёт три самых дорогих заказа.

### 8.4. Поиск через `LIKE`

```
-- Имена, начинающиеся на И
SELECT * FROM customers
WHERE name LIKE 'И%';

-- Email, заканчивающиеся на @example.com
SELECT * FROM customers
WHERE email LIKE '%@example.com';
```

Символ `%` обозначает любую последовательность символов.

### 8.5. Диапазоны и списки

```
SELECT * FROM orders
WHERE amount BETWEEN 1000 AND 3000;

SELECT * FROM customers
WHERE city IN ('Москва', 'Казань');
```

### 8.6. Поиск пустых значений

```
SELECT * FROM customers
WHERE city IS NULL;
```

Пока у всех добавленных клиентов заполнен город, поэтому результат может быть пустым. Это нормально.

## Шаг 9. Выполни агрегатные запросы

Агрегатные функции позволяют рассчитывать показатели сразу по нескольким записям.

```
-- Количество заказов
SELECT COUNT(*) AS total_orders
FROM orders;

-- Общая сумма заказов
SELECT SUM(amount) AS total_sum
FROM orders;

-- Средняя сумма заказа
SELECT AVG(amount) AS avg_amount
FROM orders;

-- Минимальная и максимальная сумма
SELECT MIN(amount) AS min_amount,
       MAX(amount) AS max_amount
FROM orders;
```

### 9.1. Группировка через `GROUP BY`

Количество заказов и их общая сумма для каждого клиента:

```
SELECT customer_id,
       COUNT(*) AS order_count,
       SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id;
```

### 9.2. Фильтрация групп через `HAVING`

```
SELECT customer_id,
       SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 2000;
```

Разница простая:

- `WHERE` фильтрует отдельные строки до группировки.
- `HAVING` фильтрует получившиеся группы после группировки.

## Шаг 10. Изучи соединения таблиц `JOIN`

Соединения позволяют получать информацию сразу из нескольких таблиц. В нашем случае можно узнать имя клиента, дату его заказа, сумму и статус.

### 10.1. `INNER JOIN` — только совпадающие записи

```
SELECT c.name,
       o.order_date,
       o.amount,
       o.status
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;
```

В результате будут клиенты, у которых есть заказы.

### 10.2. `LEFT JOIN` — все клиенты

```
SELECT c.name,
       o.id AS order_id,
       o.amount
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

Запрос показывает всех клиентов, в том числе тех, у кого нет заказов. В полях отсутствующего заказа будет `NULL`.

### 10.3. `RIGHT JOIN` — все заказы

```
SELECT c.name,
       o.id AS order_id
FROM customers c
RIGHT JOIN orders o ON c.id = o.customer_id;
```

Этот запрос сохраняет все строки из таблицы `orders`, даже если соответствующего клиента нет в левой таблице.

### 10.4. Объедини `JOIN` и `GROUP BY`

Посчитай количество заказов и общую сумму для каждого клиента:

```
SELECT c.name,
       COUNT(o.id) AS order_count,
       COALESCE(SUM(o.amount), 0) AS total_amount
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name
ORDER BY total_amount DESC;
```

Здесь `COALESCE` заменяет `NULL` на `0`, если у клиента нет заказов.

## Шаг 11. Подзапросы и `UNION`

### 11.1. Подзапрос с `IN`

Найди клиентов, у которых есть заказ дороже 1000:

```
SELECT name, city
FROM customers
WHERE id IN (
    SELECT customer_id
    FROM orders
    WHERE amount > 1000
);
```

### 11.2. Подзапрос с `EXISTS`

Клиенты, у которых есть хотя бы один заказ:

```
SELECT name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

Клиенты без заказов:

```
SELECT name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```

### 11.3. Объединение результатов через `UNION`

```
SELECT name, city
FROM customers
WHERE city = 'Москва'

UNION

SELECT name, city
FROM customers
WHERE city = 'Казань';
```

`UNION` объединяет результаты запросов и удаляет повторяющиеся строки. `UNION ALL` сохраняет повторы. Количество столбцов в обеих частях должно совпадать.&#x20;

MySQL.md



## Шаг 12. Измени данные и структуру таблиц

### 12.1. Обновление данных

```
UPDATE customers
SET city = 'Екатеринбург'
WHERE id = 4;

UPDATE orders
SET status = 'paid'
WHERE id = 3;
```

Проверь изменения:

```
SELECT * FROM customers WHERE id = 4;
SELECT * FROM orders WHERE id = 3;
```

### 12.2. Добавление нового столбца

```
ALTER TABLE customers
ADD COLUMN phone VARCHAR(20);
```

Заполни его у одного клиента:

```
UPDATE customers
SET phone = '123'
WHERE id = 4;
```

Проверь результат:

```
SELECT * FROM customers WHERE id = 4;
```

### 12.3. Изменение типа столбца

```
ALTER TABLE customers
MODIFY COLUMN phone VARCHAR(30);
```

### 12.4. Удаление столбца

```
ALTER TABLE customers
DROP COLUMN phone;
```

Выполняй удаление столбца только после проверки: данные в нём будут потеряны.&#x20;

MySQL.md



## Шаг 13. Создай резервную копию базы данных

Это одна из главных задач из твоего файла.

Сначала выйди из MySQL:

```
\q
```

Теперь ты снова находишься в Git Bash.

Убедись, что работаешь в папке проекта:

```
cd ~/my-mysql
```

Создай дамп:

```
docker exec -e MYSQL_PWD=123 my-mysql \
  mysqldump -uroot sql_practice > dump.sql
```

Проверь, что файл появился:

```
ls -lh dump.sql
```

Посмотри содержимое:

```
head -n 30 dump.sql
```

В файле должны находиться SQL-команды для восстановления структуры таблиц и их данных.

Дамп сохраняет базу в текстовом SQL-файле. Храни его отдельно от контейнера: это позволит восстановить данные, если контейнер или хранилище будут потеряны.

### 13.1. Проверь, что дамп содержит таблицы

Выполни:

```
grep -E 'CREATE TABLE|INSERT INTO' dump.sql
```

Если команда показывает определения таблиц и вставку данных, это хороший признак, что дамп сформирован.

### 13.2. Как восстановить дамп

Если понадобится восстановление в существующую базу `sql_practice`, можно выполнить:

```
docker exec -i -e MYSQL_PWD=123 my-mysql \
  mysql -uroot sql_practice < dump.sql
```

Внимание: восстановление поверх существующих таблиц может завершиться ошибкой из-за уже имеющихся таблиц и записей. Для полноценной проверки восстановления лучше использовать отдельную тестовую базу и предварительно убедиться, что дамп содержит необходимые данные.

MySQL.md

## Шаг 14. Проверь управление контейнером

Эти команды выполняются в Git Bash, а не в MySQL.

Остановка контейнера:

```
docker stop my-mysql
```

Повторный запуск:

```
docker start my-mysql
```

Просмотр логов:

```
docker logs my-mysql
```

Проверка доступности MySQL:

```
docker exec my-mysql mysqladmin ping -uroot -p123
```

Так ты убедишься, что контейнер можно остановить и запустить снова. Благодаря тому, что используется именованный том `mysql_data`, данные сохраняются при обычном удалении контейнера, если сам том не удалять.&#x20;

MySQL.md



## Шаг 15. Что должно остаться после выполнения

В папке `my-mysql` ожидаются следующие файлы:

my-mysql/

Папка проекта

my.cnf

Конфигурация кодировки MySQL

dump.sql

Резервная копия учебной базы данных

Данные самой базы хранятся в Docker volume `mysql_data`, а не в этой папке.

### Итоговый контрольный список

Проверка перед сдачей

0/11

Docker Desktop запущен, контейнер my-mysql работает

Подключение к MySQL выполняется без ошибок

Создана база sql_practice

Созданы таблицы customers и orders

Добавлены тестовые записи

Проверены INSERT, SELECT, UPDATE и DELETE

Выполнены фильтрация и сортировка

Проверены агрегаты и GROUP BY

Выполнены запросы JOIN, подзапросы и UNION

Проверены ALTER TABLE и управление контейнером

Файл dump.sql создан и содержит SQL-данные

Рекомендация: для отчёта преподавателю сделай скриншоты запущенного контейнера (`docker ps`), таблиц с данными (`SELECT * FROM customers;` и `SELECT * FROM orders;`), результата `JOIN` и созданного файла `dump.sql`.

Важный момент: не выполняй `DROP DATABASE sql_practice;` до создания и проверки резервной копии. Эта команда удаляет учебную базу, и для продолжения работы её придётся создавать заново.
