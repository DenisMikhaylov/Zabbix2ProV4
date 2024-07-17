Задание 1. Создание зависимых элементов

Шаг 1. Создание мастер элемента

Подключиться к Zabbix по SSH

Подключиться под root.
Выполнить команду.

```
chmod +r /var/log/nginx/access.log
```

```
Host: Zabbix server
```
```
  Items
    Name: HTTP Access Log
    Type: agent zabbix (active)
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

Задание 2. Зависимые элементы  и предобработка для файлов в папке

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


Задание 3. Внешние скрипты

Шаг 1. Простой скрипт

Проверка расположения внешних скриптов на Zabbix server
```
# zabbix_server --help | grep ExternalScripts
```
```
# nano /etc/zabbix/zabbix_server.conf
```
Настройка работы с внешними скриптами
```
Timeout=30
```
```
ExternalScripts=/etc/zabbix/externalscripts
```
```
# mkdir /etc/zabbix/externalscripts
```

```
systemctl restart zabbix-server
```
```
# nano /etc/zabbix/externalscripts/ping_avg.sh
```
```
#!/bin/sh
ping -c"$1" "$2" | tail -n1 | cut -d'/' -f5
```

```
chmod +x /etc/zabbix/externalscripts/ping_avg.sh
```
Настройка элемента сбора информации
```
>Hosts->zabbix
  Items
    Name: Ping AVG
    Type: External Check
    Key: ping_avg.sh[3,"{HOST.CONN}"]
    Type of information: Numeric (float)
    Units: ms
```

Шаг 2. Скрипт расширенный

Создание проверки speedtest
Переходим на сервер zabbix
Устанавливаем приложение
```
apt install speedtest-cli
```
Тестирование работы приложения
```
# time speedtest-cli

# speedtest-cli --csv-header

# speedtest-cli --csv

```
Создаем второй скрипт собирающий данные по speedtest
```
# nano /etc/zabbix/externalscripts/speedtest.sh
```
```
#!/bin/sh

if [ "x$1" = xupload ]
then
        A="--no-download"
        F=8
elif [ "x$1" = xdownload ]
then
        A="--no-upload"
        F=7
else
        exit 1
fi

speedtest-cli --csv $A | cut -d',' -f $F
```

```
chmod +x /etc/zabbix/externalscripts/speedtest.sh
```
Тестируем работу скрипта
```
# /etc/zabbix/externalscripts/speedtest.sh upload

# /etc/zabbix/externalscripts/speedtest.sh download
```
Создаем item
```
Hosts->zabbix
  Items
    Name: speedtest download
    Type: External Check
    Key: speedtest.sh[download]
    Type of information: Numeric (float)
    Units: bit/s
    Update interval: 30m
    Name: speedtest upload
    Type: External Check
    Key: speedtest.sh[upload]
    Type of information: Numeric (float)
    Units: bit/s
    Update interval: 30m
```
Задание 4. Создание проверки на основе zabbix trapper

Cоздаем item на zabbix
```
Hosts->zabbix->Items
  Name: my item
    Type: Zabbix trapper
    Key:  my.item
    Allowed hosts: 127.0.0.1,192.168.10.0/24
```
Исталируем приложение
```
# apt install zabbix-sender
```
Тестируем работу приложения
<Name host> - имя хоста в zabbix web интерфейсе
```
zabbix_sender -z 192.168.10.41 -p 10051 -s <Name host> -k my.item -o 1
```
Переделываем работы получение данных из speedtest

Создаем новый скрипт
```
# nano /root/speedtest.sh
```
```
#!/bin/sh

### speedtest-cli ### result bits/s
MY_RES=`speedtest-cli --csv`
MY_DOWNLOAD=`echo $MY_RES | cut -d',' -f7`
MY_UPLOAD=`echo $MY_RES | cut -d',' -f8`

### speedtest ### result Bytes/s (use preprocess Custom multiplier)
#MY_RES=`speedtest -f csv`
#MY_DOWNLOAD=`echo $MY_RES | cut -d',' -f6`
#Y_UPLOAD=`echo $MY_RES | cut -d',' -f7`

zabbix_sender -z 127.0.0.1 -p 10051 -s <Name host> -k speedtest.download -o $MY_DOWNLOAD
zabbix_sender -z 127.0.0.1 -p 10051 -s <Name host> -k speedtest.upload -o $MY_UPLOAD
```
```
/etc/zabbix/externalscripts/speedtest.sh
```
Создаем новые Item
```
zabbix->Items
  Name: speedtest download trap
    Type: Zabbix trapper
    Key:  speedtest.download
    Type of information: Numeric (float) или Numeric (unsigned)
    Units: bit/s
    Allowed hosts: 127.0.0.1
...
  Name: speedtest upload trap
    Type: Zabbix trapper
    Key:  speedtest.upload
    Type of information: Numeric (float) или Numeric (unsigned)
    Units: bit/s
    Allowed hosts: 127.0.0.1
```
Запускаем скрипт
```
chmod +x /etc/zabbix/externalscripts/speedtest.sh
```

Задание 5. Пользовательские параметры

```
nano /etc/zabbix/zabbix_agentd.d/hdd.conf
```
```
UserParameter=custom.vfs.dev.read.ops[*],cat /sys/class/block/$1/stat | awk '{print $$1}'
UserParameter=custom.vfs.dev.read.merged[*],cat /sys/class/block/$1/stat | awk '{print $$2}'
UserParameter=custom.vfs.dev.read.sectors[*],cat /sys/class/block/$1/stat | awk '{print $$3}'
UserParameter=custom.vfs.dev.read.ms[*],cat /sys/class/block/$1/stat | awk '{print $$4}'
UserParameter=custom.vfs.dev.write.ops[*],cat /sys/class/block/$1/stat | awk '{print $$5}'
UserParameter=custom.vfs.dev.write.merged[*],cat /sys/class/block/$1/stat | awk '{print $$6}'
UserParameter=custom.vfs.dev.write.sectors[*],cat /sys/class/block/$1/stat | awk '{print $$7}'
UserParameter=custom.vfs.dev.write.ms[*],cat /sys/class/block/$1/stat | awk '{print $$8}'
UserParameter=custom.vfs.dev.io.active[*],cat /sys/class/block/$1/stat | awk '{print $$9}'
UserParameter=custom.vfs.dev.io.ms[*],cat /sys/class/block/$1/stat | awk '{print $$10}'
UserParameter=custom.vfs.dev.weight.io.ms[*],cat /sys/class/block/$1/stat | awk '{print $$11}'
UserParameter=my.disks.discovery,/bin/lsblk -dJ | /bin/sed -e 's/blockdevices/data/' -e 's/name/{#NAME}/g' -e 's/type/{#TYPE}/g'
```
```
systemctl restart zabbix-agent
```


Проверяем работу нового параметра , переключаемся на zabbix

```
apt install zabbix_get
apt install jq
```

Ip address указываем агента.
Если расширялся агент на Zabbix server, указывается ip 127.0.0.1
```
# zabbix_get -s <ip server> -k my.disks.discovery | jq
```


Шаг 2. Создание Item
Создайте item на остальные userparameter
Создаем шаблон 

```
Templates->Create template
  Template name: My Template Linux disks utilization
  Groups: Templates/Server hardware

  Discovery rules->
    Name: my disks discovery
    Key: my.disks.discovery
    Filters->
      {#TYPE} matches disk
```

создание item prototypes

```
    Item prototypes->
      Name: disk {#NAME} read bytes
      Key: custom.vfs.dev.read.ops[{#NAME}]
      Type of information: Numeric (float)

      Name: disk {#NAME} write bytes
      Key: custom.vfs.dev.write.ops[{#NAME}]
      Type of information: Numeric (float)
```
по аналогии добавить все остальные параметры.


Настройка мониторинга Windows  с помощью активного агента.

Настройка авторегистрации систем с агентами, работающими в активном режиме

```
Alert - Actions - Auto registration 
  Name: Add Windows clients                                          # or Add Linux clients
  Conditions: Host name contains CLIENT                              # or client (lowercase) for linux
  Action operations:
    Add host
    Add to host groups: Windows clients                              # or Linux clients
    Link to templates: Windows by Zabbix agent active                # or Linux by Zabbix agent active
                 
  Set host inventory mode: Automatic
```
Подключиться к Windows сервер

Настройка агента на активный режим

Отредактировать файл конфигурации агента 

C:\Program Files\Zabbix Agent\zabbix_agentd.conf

```
#Server=<ip address zabbix or Name>
ListenIP=0.0.0.0
ServerActive=<ip address zabbix or Name>
Hostname=CLIENTN
```
перезапустить службу агента

```
get-service 'zabbix agent' | Restart-Service
```
В веб интерфейсе zabbix открывает "Data collection" - 'Host'


Удаляем хост от windows.

Ждем автоматичекой регистрации активного клиента.
Проверяем, создание нового хоста с именем CLIENTN

Создание элемента мониторинга 

В веб интерфейсе zabbix открывает "Data collection" - 'Host'

```
Host: CLIENTN
...
  Items
    Name: Processor total
    Type: agent zabbix active
    Key: perf_counter_en["\Processor(_Total)\% Processor Time",60]
```
```
Host: CLIENTN
...
  Items
    Name: Event 50 
    Type: agent zabbix active
    Key: eventlog[System,,"Warning",,50,,skip]
```
```
Host: CLIENTN
...
  Items
    Name: Error drive
    Type: agent zabbix active
    Key: eventlog[System,,"Warning",,153,,skip]
```
```
Host: CLIENTN
...
  Items
    Name: WMI object disk freespace
    Type: agent zabbix active
    Key: wmi.get[root\cimv2,SELECT FreeSpace FROM Win32_LogicalDisk WHERE DeviceID='C:']

```
```
Host: CLIENTN
...
  Items
    Name: WMI object version bios
    Type: agent zabbix active
    Key: wmi.get[root\cimv2,SELECT version FROM Win32_bios]

```
```
Host: CLIENTN
...
  Items
    Name: WMI object product baseboard
    Type: agent zabbix active
    Key: wmi.get[root\cimv2,SELECT product FROM Win32_baseboard]
    Populates host inventory field: model
```
```
Host: CLIENTN
...
  Items
    Name: WMI object serial number baseboard
    Type: agent zabbix active
    Key: wmi.get[root\cimv2,SELECT serialnumber FROM Win32_baseboard]
    Populates host inventory field: Chassis
```



Задание 6. Глобальные скрипты

Создание 

```
    Alert -> Scripts -> Create Script
      Name: Время работы
      Scope: Manual host action
      Type: Script
      Execute on : Zabbix server
      Commands: secs=`zabbix_get -s {HOST.CONN} -p 10050 -k 'system.uptime'`
      printf '%d дней, %d:%d:%d\n' $((secs/86400)) $((secs%86400/3600)) $((secs%3600/60)) $((secs%60))
```
```
    Alert -> Scripts -> Create Script
      Name: Кто на Linux
      Scope: Manual host action
      Type: Script
      Execute on : Zabbix agent
      Commands: who -dsTu
```
