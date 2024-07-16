Перейти на сервер Debian1

Установка приложения 
```
# apt install isc-dhcp-server
```
Настройка DHCP 

```
# nano /etc/dhcp/dhcpd.conf
```
Добавить Scope
```
shared-network LAN1 {
  subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.2 192.168.1.254;
    option routers 192.168.1.1;
  }
}
```

Установить dhcpd-pools

```
# apt install dhcpd-pools
```
Устанавливаем приложение
```
apt install jq
apt install xmlstarlet
```
Создание Статистика DHCP сервера

Создаем скрипты

```
# nano /etc/zabbix/dhcp-pools-discovery.sh
```
```
#!/bin/bash

echo -n '{"data":['

str=`/usr/bin/dhcpd-pools -c /etc/dhcp/dhcpd.conf -f x | \
/usr/bin/xmlstarlet sel -T -t -m '//shared-network' \
-o '{"{#POOLNAME}":"' -v location -o '"},'`

echo -n ${str::-1}

echo -n ']}'
```
Разрешем выполняться скрипту 
```
chmod +x /etc/zabbix/dhcp-pools-discovery.sh
```

Проверяем работу

```
# /etc/zabbix/dhcp-pools-discovery.sh | jq
```

Создание еще одного скрипта

```
# nano /etc/zabbix/dhcp-pools-shared-network.sh
```

```
#!/bin/sh

res_field=2
test "x$2" = "xused" && res_field=3

/usr/bin/dhcpd-pools -c /etc/dhcp/dhcpd.conf -f x | \
  /usr/bin/xmlstarlet sel -T -t -m '//shared-network' \
  -v location -o ' ' -v defined -o ' ' -v used -n | \
  grep $1 | cut -d ' ' -f $res_field
```
Разрешем выполняться скрипту 
```
chmod +x /etc/zabbix/dhcp-pools-shared-network.sh
```

тестирование скрипта

```
# /etc/zabbix/dhcp-pools-shared-network.sh LAN1 defined

# /etc/zabbix/dhcp-pools-shared-network.sh LAN1 used
```

Создаем  новый файл для агента.
```
# nano  /etc/zabbix/zabbix_agentd.conf.d/dhcp_stat.conf
```
```
UserParameter=dhcp.pools.discovery,/etc/zabbix/dhcp-pools-discovery.sh

UserParameter=dhcp.pools.shared-network[*],/etc/zabbix/dhcp-pools-shared-network.sh $1 $2
```
Создаем настройки 

```
Templates->Create template
  Template name: Template App DHCP Pools
  Groups In groups: Templates/Applications

  Macros: {$DHCP.POOLS.MAX.PERCENT}=90
Add


  Discovery rules
    Name: Search DHCP Pools
    Type: Zabbix Agent
    Key: dhcp.pools.discovery
  Add
    Item prototypes
      Name: DHCP Pool {#POOLNAME} max addr
      Type: Zabbix Agent
      Key: dhcp.pools.shared-network[{#POOLNAME},defined]
      
    Add

      Name: DHCP Pool {#POOLNAME} cur addr
      Type: Zabbix Agent
      Key: dhcp.pools.shared-network[{#POOLNAME},used]
      
    Add

    Graph prototypes
      Name: DHCP Pool {#POOLNAME} max cur
      Y axis MIN value: Fixed 0
      Items: 
        Template App DHCP Pools: DHCP Pool {#POOLNAME} cur addr
        Template App DHCP Pools: DHCP Pool {#POOLNAME} max addr

    Trigger prototypes

      Name: On {HOST.NAME} in the DHCP pool {#POOLNAME}
     
      Expression:
                  last(/Template App DHCP Pools/dhcp.pools.shared-network[{#POOLNAME},used])/last(/Template App DHCP Pools/dhcp.pools.shared-network[{#POOLNAME},defined])*100 > {$DHCP.POOLS.MAX.PERCENT}
      Severity: Warning
```
Редактируем файл настройки DHCP

```
nano /etc/dhcp/dhcpd.conf
```
Находим блок с subnet
```
добавлем 

shared-network LAN2 { 

  subnet 192.168.30.0 netmask 255.255.255.0 {
    range 192.168.30.101 192.168.30.109;
    option routers 192.168.30.1;
  }

} 
```
Проверить работы discovery.


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

Создание пользовательских макросов
Глобальные макросы: 

Создание глобальных макросов, перейдите в меню Administration >  Macros 

Нажимаем Add
```
Макрос: {$DISK_NAME}
Значение: c:
Описание: Макрос для диска с:
```
```
Макрос:{$THRESHOLDCPU_WAR} 
Значение: 50
Описание: Предупреждение значение cpu
```
```
Макрос:{$THRESHOLDCPU_CRIT} 
Значение: 70
Описание: Максимальное значение cpu
```
```
Макрос:{$THRESHOLDDISK_LOW} 
Значение: 5
Описание: Минимальное свободное место в процентах
```
Создание макросов для host, перейдите в меню data colliction >  hosts

Выбрать host Gate, перейти на вкладку Macros

Нажать add:
```
Макрос:{$THRESHOLD_WMI_LOW} 
Значение: 10
```

Создание элемента с использованием макроса 
```
Host: CLIENTN
...
  Items
    Name: WMI object disk c: freespace
    Type: agent zabbix active
    Key: wmi.get[root\cimv2,SELECT FreeSpace FROM Win32_LogicalDisk WHERE DeviceID='{$DISK_NAME}']

```
