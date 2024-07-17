Настройка макросов

Настройка мониторинга на стороне Zabbix
Подключиться на браузере к серверу Zabbix
```
http://zabbix.corp1.ru
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

```
Макрос: {$SSH_PORT}
Значение: 22
Описание: Порт для подключения SHH
```

Создание или настройка макросов хоста Zabbix server
```
  Открыть меню
Data - collection:
    Hosts:
       Zabbix server:
          Macros:
          Macro: {$HDD_FREE_PERCENT_THRESHOLD_WARNING}
          Value: 20
          Macro: {$HDD_FREE_PERCENT_THRESHOLD_CRITICAL}
          Value: 5
```
Создание или настройка макросов хоста CLIENTN
```
Открыть меню
Data - collection:
    Hosts:
       CLIENTN:
          Macro:{$THRESHOLD_WMI_LOW} 
          Value: 10
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

Создание или настройка макросов Шаблона
```
Открыть меню
Data - collection:
    Template:
       Create template:
          Name: Template App SSH Port Service SSH Port Service
          Macros:
          Macro: {$SSH_PORT_NEW}
          Value: 22
      Create item:
          Name: SSH service is running
          Type: simple check
          Key: net.tcp.service[ssh,,{$SSH_PORT_NEW}]
          Update interval: 30s
         Tag: ssh
```

Добавить новый шаблон на сервер Zabbix
```
Открыть меню
Data - collection:
    Hosts: Zabbix server
    Templates:  Template App SSH Port Service SSH Port Service
```

Проверка значений 
```
Monitors -> Latest data-> SSH service is running
Monitors -> Latest data-> tag: ssh
```
