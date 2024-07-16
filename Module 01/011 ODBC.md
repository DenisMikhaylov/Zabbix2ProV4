Настройка MySQL с ODBC

Установка компонентов на сервер
```
yum install unixODBC mariadb-connector-odbc

```
Настройка ODBC мониторинг MYSQL

Установка ODBC драйвера на сервер c базой данных MYSQL
В зависимости от операционной системы найдите инструкцию по установке ODBC drive
База данных Zabbix находиться на сервере Zabbix.
Подключаемся на сервер Zabbix через putty.
Предварительно необходимо узнать ваш ip адресс использую команду
```
# Ip a
```
Установка Mysql odbc driver
```
# apt install odbc-mariadb
```
Проверка настройки ODBC
```
# odbcinst -J
```
В результат выполнения команды 
Находим два пути настроек
```
DRIVERS............: /etc/odbcinst.ini
SYSTEM DATA SOURCES: /etc/odbc.ini
```
1.	Настройка ODBC выполняется изменением odbcinst.ini и odbc.ini файлов. Эти файлы конфигурации можно найти в /etc папке. 
Файл odbcinst.ini может отсутствовать. Если его нет создается в ручную.
Открываем или создаем файл odbcinst.ini
```
# nano /etc/odbcinst.ini
```
Настройка для MySQL. Изменяем название драйвера
```
[MariaDB Unicode] на [madb] для упрощения настроек
```
Открываем файл odb.ini
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
Настройка мониторинга на стороне Zabbix
Подключиться на браузере к серверу Zabbix
```
http://<ip zabbix>
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



