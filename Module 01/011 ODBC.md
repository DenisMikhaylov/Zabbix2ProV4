Настройка MySQL с ODBC
Цель Лабораторной:
Настройть возможность собирать информацию с базы данных по средствам подключения ODBC.

Сценарий:
Вы администратор 

Настройка лабораторного стенда:

Virtual machines: Zabbix, Gate

User name: root

Password: Pa$$w0rd

Основные задачи для лабораторной:

1. Установить ODBC Dricer для MySQL
2. Создать элемент мониторинга для проверки корректной настройки.

Шаг 1. Установка ODBC Driver

Установка ODBC драйвера на сервер c базой данных MYSQL.
В зависимости от операционной системы найдите инструкцию по установке ODBC drive.

Для мониторинга будем использовать базу данных Zabbix.

Подключаемся на сервер Zabbix через putty.
Для подключения используем ip адрес 192.168.10.41

Установка Mysql odbc driver
```
# apt install odbc-mariadb
```
Проверка настройки ODBC
```
# odbcinst -j
```
В результат выполнения команды 
Находим два пути настроек
```
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
```
Шаг 2. Настройка ODBC Drive

Настройка ODBC выполняется изменением odbcinst.ini и odbc.ini файлов. Эти файлы конфигурации можно найти в /etc папке. 

Файл odbcinst.ini может отсутствовать. Если его нет создается в ручную.

Открываем или создаем файл odbcinst.ini
```
# nano /etc/odbcinst.ini
```
Настройка для MySQL. Изменяем название драйвера
```
[MariaDB Unicode] на [madb] для упрощения настроек
```
Открываем файл odbc.ini
```
# nano /etc/odbc.ini
```
Добавляем настройку для MySQL.

В поля user и Password вводим свои значения.
В нашем стенде используются значения 
```
User = zabbix
Password = Pa$$w0rd
```
```
[mysql]                       
Description = MySQL database 2                 
Driver  = madb                                
User = zabbix                                    
Port = 3306                                    
Password = Pa$$w0rd                           
Database = zabbix                             
Server = 127.0.0.1
```
Тестирование подключения 
```
# isql -v mysql
SQL> show tables;
SQL> quit;
```
Шаг 3. Настройка на Zabbix Тестового элемента.

Подключиться на браузере к серверу Zabbix
```
http://zabbix.corp1.ru
```
Настройка элемента мониторинга
```
Host: Zabbix
  Items
    Name: Mysql host count
    Type: database monitor
    Key: db.odbc.select[mysql-simple-check,mysql]
    Script: select count(*) from hosts
```
Проверка значений 
```
Monitors -> Latest data-> Mysql host count
```


Results: После выполнения данной лабораторной, вы успешно создадите подключение Zabbix к MySQL по средствам ODBC Driver.
И убедитесь в корректной работе по средствам тестового элемента. Тестовый элемент получит информацию о количестве host.


