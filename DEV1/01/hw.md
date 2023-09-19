# Home work

Для выполнения практических заданий нужно войти в операционную систему под пользователем student (пароль student). Для запуска psql в окне терминала наберите psql без параметров. Для подключения будут использованы настройки по умолчанию.
```bash
    student:~$ psql
```
Для выполнения заданий каждой темы удобно создавать отдельную
базу данных:
```sql
student/student=# CREATE DATABASE tools_overview;
CREATE DATABASE
student/student=# \c tools_overview
You are now connected to database "tools_overview" as user
"student".
student/tools_overview=#
```

1. Установите в postgresql.conf для параметра work_mem значение 8 Мбайт. Обновите конфигурацию и проверьте, что изменения вступили в силу.
<summary>Решение</summary>
<details>

Посмотрим конфигурационные параметры

```bash
    psql
=> SHOW work_men;
 work_mem 
----------
 16MB
(1 row)
=> \q
```
Установим 8MB
```bash
    sudo sed '/^work_mem/d' -i /etc/postgresql/12/main/postgresql.conf
    sudo nano /etc/postgresql/12/main/postgresql.conf
```
в самый низ файла добавим: work_mem=8MB -> Ctrl+x -> y -> Enter

Обновим конфигурацию
```bash
    tail -n 5 /etc/postgresql/12/main/postgresql.conf
    sudo pg_ctlcluster 12 main reload
```
Проверим
```bash
    psql
=> SHOW work_mem;
 work_mem 
----------
 16MB
(1 row)
```
Так как в postgresql.auto.conf во время лекции было записано другое значение (которое в приоритете), то мы не добились желаемого результата. Изменим также значение переменной в postgresql.auto.conf:
```bash
    psql
=> ALTER SYSTEM SET work_mem TO '8MB';
ALTER SYSTEM
=> \! sudo pg_ctlcluster 12 main reload
=> SHOW work_mem;
 work_mem 
----------
 8MB
(1 row)
=> \q
```
Done!
</details>

------

2. Запишите в файл ddl.sql команду CREATE TABLE на создание любой таблицы. Запишите в файл populate.sql команды на вставку строк в эту таблицу. Войдите в psql, выполните оба скрипта и проверьте, что таблица создалась и в ней появились записи.
<summary>Решение</summary>

<details>

```bash
cat > ddl.sql <<EOF
CREATE TABLE products (
product_no integer,
name text,
price numeric
);
EOF
cat > populate.sql <<EOF
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99),
    (3, 'Milk', 2.99);
EOF
psql
```
```sql
=> \c tools_overview
=> \l
                                    List of databases
      Name      |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
----------------+----------+----------+-------------+-------------+-----------------------
 bookstore      | student  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres       | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 student        | student  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                |          |          |             |             | postgres=CTc/postgres
 template1      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                |          |          |             |             | postgres=CTc/postgres
 tools_overview | student  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
(6 rows)

=> q
=> \dt
Did not find any relations.

=> \i ddl.sql
=> \dt
          List of relations
 Schema |   Name   | Type  |  Owner  
--------+----------+-------+---------
 public | products | table | student
(1 row)

=> SELECT * FROM products;
 product_no | name | price 
------------+------+-------
(0 rows)

=> \i populate.sql
INSERT 0 3

=> SELECT * FROM products;
 product_no |  name  | price 
------------+--------+-------
          1 | Cheese |  9.99
          2 | Bread  |  1.99
          3 | Milk   |  2.99
(3 rows)

```
Done!
</details>

---

3. Найдите в журнале сервера строки за сегодняшний день.
<summary>Решение</summary>
<details>

```bash
    tail -n 20 /var/log/postgresql/postgresql-12-main.log
```
Записи за сегодняшний день находятся в конце файла

Done!
</details>