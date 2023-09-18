# Postgresql
Запуск postgresql
```bash
    psql
```
Выход
```sql
=> \q
```
pg_ctlcluster запуск и останов сервера
```bash
    sudo pg_ctlcluster 12 main stop // Остановить сервер
    sudo pg_ctlcluster 12 main start // Запустить сервер
    sudo pg_ctlcluster 12 main restart // Перезапустить
    sudo pg_ctlcluster 12 main reload // Обновить конфигурацию
```
## Каталог установки PostgreSQL:
/usr/lib/postgresql/12 - owner root
## $PGDATA
Кластер баз данных автоматически инициализируется при установке из пакета и находится в каталоге /var/lib/postgresql/12/main ($PGDATA) - owner postgres
## Журнал сообщений
https://postgrespro.ru/docs/postgresql/12/runtime-config-logging
```bash
    tail -n 10 /var/log/postgresql/postgresql-12-main.log
```
## Посмотреть конфигурационные параметры
```bash
    psql
=> SHOW work_mem;
```
## Настройки сервера
- Основной конфигурационный файл — postgresql.conf, он редактируется вручную
```bash
    sudo sed '/^work_mem/d' -i /etc/postgresql/12/main/postgresql.conf
    //echo 'work_mem = 16MB' >> /etc/postgresql/12/main/postgresql.conf
    sudo nano /etc/postgresql/12/main/postgresql.conf
```
добавить в самый низ файла: work_mem = 64MB 

Ctrl+x -> y -> Enter
```bash
    tail -n 5 /etc/postgresql/12/main/postgresql.conf
    sudo pg_ctlcluster 12 main reload
```
или
- postgresql.auto.conf — предназначен для изменения специальной командой ALTER SYSTEM. Параметры, установленные через ALTER SYSTEM, имеют приоритет над параметрами в postgresql.conf. Файл лежит в директории $PGDATA
```bash
    psql
=> ALTER SYSTEM SET work_mem TO '64MB';
ALTER SYSTEM
=> \! sudo pg_ctlcluster 12 main reload
=> \q
```
TЕсли не обновлять конфигурацию - то изменения будут только на время текущего сеанса. Обновить конфигурацию
```bash
    sudo pg_ctlcluster 12 main reload
```
При указании SET LOCAL новое значение действует только в текущей транзакции.

## Команды Posgresql
```sql
=> \conninfo // узнать, куда мы подключились
```
### Форматирование вывода
Клиент psql умеет выводить результат запросов в разных форматах:
- формат с выравниванием значений;
- формат без выравнивания;
- расширенный формат.
Формат с выравниванием используется по умолчанию (Ширина столбцов выровнена по значениям. Также выводится строка заголовков и итоговая строка):
```sql
=> SELECT name, setting, unit FROM pg_settings LIMIT 7;
```
Команды psql для переключения режима выравнивания:
```
\a — переключатель режима: с выравниванием/без выравнивания.
\t — переключатель отображения строки заголовка и итоговой строки.
```
Отключим выравнивание, заголовок и итоговую строку:
```sql
=> \a \t
Output format is unaligned.
Tuples only is on.
=> SELECT name, setting, unit FROM pg_settings LIMIT 7;
allow_system_table_mods|off|
application_name|psql|
archive_cleanup_command||
archive_command|(disabled)|
archive_mode|off|
archive_timeout|0|s
array_nulls|on|
=> \a \t
Output format is aligned.
Tuples only is off.
```
Такой формат неудобен для просмотра, но может оказаться полезным для автоматической обработки.

Расширенный формат удобен, когда нужно вывести много столбцов для одной или нескольких записей, при этов названия столбцов выводятся в столбик, а в соседний столбик выводятся их значения. Для этого вместо точки с запятой указываем в конце команды \gx:
```
\gx — переключатель расширенного режима для одного запроса.
```
```sql
=> SELECT name, setting, unit, category, context, vartype, min_val, max_val, boot_val, reset_val FROM pg_settings
WHERE name = 'work_mem' \gx
```
Если расширенный формат нужен не для одной команды, а постоянно, можно включить его переключателем \x.
```
\x — переключатель расширенного режима на постоянно.
```
Все возможности форматирования результатов запросов доступны через команду \pset.
```sql
=> \pset // узнать все возможности форматирования результатов запросов
```
## Взаимодействие с ОС и выполнение скриптов
Из psql можно выполнять команды shell:
```sql
=> \! pwd
/home/student
```
С помощью запроса SQL можно сформировать несколько других запросов SQL и записать их в файл, используя команду \o[ut] (Сначала отключим форматирование (\a \t), заменим разделитель столбцов  с | на " "(пусто), отправим вывод в файл dev1_psql.log (\o dev1_psql.log) и выполним запрос):
```sql
=> \a \t
Output format is unaligned.
Tuples only is on.
=> \pset fieldsep ''
Field separator is "".
=> \o dev1_psql.log
=> SELECT format('SELECT %L AS tbl, count(*) FROM %I;', tablename, tablename)
FROM pg_tables LIMIT 3;
```
На экран ничего не попало. Посмотрим в файле:
```sql
=> \! cat dev1_psql.log
SELECT 'pg_statistic' AS tbl, count(*) FROM pg_statistic;
SELECT 'pg_type' AS tbl, count(*) FROM pg_type;
SELECT 'pg_foreign_server' AS tbl, count(*) FROM pg_foreign_server;
```
Теперь файл dev1_psql.log можно считать скриптом (в нём записаны команды sql).

Вернем вывод на экран и восстановим форматирование по умолчанию.
```sql
=> \o \t \a
Tuples only is off.
Output format is aligned.
```
Выполним теперь эти команды из файла с помощью \i[nclude]:
```sql
=> \i dev1_psql.log
     tbl      | count 
--------------+-------
 pg_statistic |   422
(1 row)

   tbl   | count 
---------+-------
 pg_type |   406
(1 row)

        tbl        | count 
-------------------+-------
 pg_foreign_server |     0
(1 row)
```
Впрочем, то же самое можно получить за один шаг, используя команду **\gexec**:
```sql
=> SELECT format('SELECT %L AS tbl, count(*) FROM %I;', tablename, tablename)
FROM pg_tables LIMIT 3 \gexec
     tbl      | count 
--------------+-------
 pg_statistic |   422
(1 row)

   tbl   | count 
---------+-------
 pg_type |   406
(1 row)

        tbl        | count 
-------------------+-------
 pg_foreign_server |     0
(1 row)

```
Другие способы выполнить команды из файла:
- psql < имя_файла
- psql -f имя_файла
- psql -c 'команда' (работает только для одной команды)

## Переменные psql
По аналогии с shell, psql имеет собственные переменные.

Установим переменную:
```sql
=> \set TEST 'Hi everyone!'
```
Чтобы получить значение, надо предварить имя переменной двоеточием:
```sql
=> \echo :TEST
Hi everyone!
```
Значение переменной можно сбросить:
```sql
=> \unset TEST
=> \echo :TEST
:TEST
```
Переменные можно использовать, например, для хранения текста часто используемых запросов. Вот запрос на получение пяти самых больших по размеру таблиц:
```sql
=> \set top5 'SELECT tablename, pg_total_relation_size(schemaname||''.''||tablename) AS bytes FROM pg_tables ORDER BY bytes DESC LIMIT 5;'
Для выполнения запроса достаточно набрать:
=> :top5
   tablename    |  bytes  
----------------+---------
 pg_depend      | 1130496
 pg_proc        | 1015808
 pg_attribute   |  688128
 pg_rewrite     |  679936
 pg_description |  573440
(5 rows)
```
Присвоение значения переменной top5 лучше записать в стартовый файл .psqlrc в домашнем каталоге пользователя. Команды из .psqlrc будут автоматически выполняться каждый раз при старте psql.
### Выполнить серию запросов
Результат запроса можно записать в переменную с помощью **\gset** при условии, что этот запрос возвращает ровно одну строку. 

Все системные переменный пишутся заглавными буквами. Свои переменные мы дорлжны писать строчными буквами.

В нашем примере функция current_setting('work_mem') возвращает одно значение параметра work_mem. Создадим аллиас current_setting('work_mem') AS current_work_mem и в конце ставим не ; , а \gset. Эта кострукция заставит команду выполниться, и результат этой команды присвоится в переменную current_work_mem. Через запятую таких выражений могло быть несколько, тогда несколько переменных было бы создано.
```sql
=> SELECT current_setting('work_mem') AS current_work_mem \gset
=> \echo Значение work_mem: :current_work_mem
Значение work_mem: 64MB
```
Без параметров \set выдает значения всех переменных, включая встроенные.
```sql
=>\set
AUTOCOMMIT = 'on'
COMP_KEYWORD_CASE = 'preserve-upper'
DBNAME = 'student'
ECHO = 'none'
ECHO_HIDDEN = 'off'
ENCODING = 'UTF8'
ERROR = 'false'
FETCH_COUNT = '0'
HIDE_TABLEAM = 'off'
HISTCONTROL = 'none'
HISTSIZE = '500'
HOST = '/var/run/postgresql'
IGNOREEOF = '0'
LAST_ERROR_MESSAGE = ''
LAST_ERROR_SQLSTATE = '00000'
ON_ERROR_ROLLBACK = 'off'
ON_ERROR_STOP = 'off'
PORT = '5432'
PROMPT1 = '%n/%/%R%x%# '
PROMPT2 = '%n/%/%R%x%# '
PROMPT3 = '>> '
QUIET = 'off'
ROW_COUNT = '1'
SERVER_VERSION_NAME = '12.5 (Ubuntu 12.5-1.pgdg20.04+1)'
SERVER_VERSION_NUM = '120005'
SHOW_CONTEXT = 'errors'
SINGLELINE = 'off'
SINGLESTEP = 'off'
SQLSTATE = '00000'
USER = 'student'
VERBOSITY = 'default'
VERSION = 'PostgreSQL 12.5 (Ubuntu 12.5-1.pgdg20.04+1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 9.3.0-17ubuntu1~20.04) 9.3.0, 64-bit'
VERSION_NAME = '12.5 (Ubuntu 12.5-1.pgdg20.04+1)'
VERSION_NUM = '120005'
current_work_mem = '64MB'
```
Справку по встроенным переменным можно получить так:
```sql
=> \? variables
```