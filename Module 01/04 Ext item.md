Задание 1. Создание зависимых элементов
Шаг 1. Создание мастер элемента
```
Host: Zabbix server
```
```
  Items
    Name: HTTP Access Log
    Type: agent zabbix active
    Key: log[/var/log/nginx/access.log,"^.*",,,skip,\0,,,]
    Type of Information:	Log
```
Шаг 2. Создание дочерних элементов
```
Host: Zabbix server
```
```
  Items
    Name: Status Code
    Type: Dependent Item
    Key: HTTPStatusCode
    Type of Information:	Numeric (unsigned)
    Master item: HTTP Access Log
Preprocessing:
Regular Expression:
^(\S+) (\S+) (\S+) \[([\w:\/]+\s[+\-]\d{4})\] \"(\S+)\s?(\S+)?\s?(\S+)?\" (\d{3}|-) (\d+|-)\s?\"?([^\"]*)\"?\s?\"?([^\"]*)\"
Output: \8
Type : Numeric (unsigned)
```
```
  Items
    Name: Path
    Type: Dependent Item
    Key: HTTPPath
    Type of Information:	text
    Master item: HTTP Access Log
Preprocessing:
Regular Expression:
^(\S+) (\S+) (\S+) \[([\w:\/]+\s[+\-]\d{4})\] \"(\S+)\s?(\S+)?\s?(\S+)?\" (\d{3}|-) (\d+|-)\s?\"?([^\"]*)\"?\s?\"?([^\"]*)\"
Output: \6
Type : text
```
Задание 2. Предобработки и троттлинг

Шаг 1. Измените настройки предобработки простой проверки RDP availability:
```
Data collection > Hosts > Local Windows server > Items > RDP availability > Preprocessing
```
```
Preprocessing steps: Add
	Name: Discard unchanged
```

Задание 3. Зависимые элементы  и предобработка для файлов в папке
Шаг 1. Создание мастер элемента
```
Host: Zabbix server
```
```
  Items
    Name: Content folder
    Type: agent zabbix
    Key: vfs.dir.get[/var/log/nginx/]
    Type of Information:	Text
```
Шаг 2. Создание discovery

```
Host: Zabbix server
```
```
  Discovery rules:
    Name: Discover file in folder
    Type: Dependent item
    Key: discovery.files
    Master item: Content folder

LLD macros: 
LLD macro:{#STATUSFILE}
     JSONPATH: $.basename 
	    
```
Шаг 3. Создание прототипа элемента
```
Host: Zabbix server
```
```
  Items
    Name: Last modifi file {#STATUSFILE}
    Type: Dependent item
    Key: status.file[{#STATUSFILE}]
    Master item: Content folder
Preprocessing steps: Add
Name: JSONPATH
Parameter: $..[?(@.basename == "{#STATUFILE}")].timestamp.modify
Preprocessing steps: Add
Name: LeftTrim
Parameter: [
Preprocessing steps: Add
Name: RightTrim
Parameter: ]
```
