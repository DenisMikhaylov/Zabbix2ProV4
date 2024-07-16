Настройка макросов

Настройка мониторинга на стороне Zabbix
Подключиться на браузере к серверу Zabbix
```
http://zabbix.corp1.ru
```

Создание глобальных макросов
```
  Открыть меню
Administration:
    Macros:
       Add:
          Macro: {$SSH_PORT}
          Value: 22
          Description: Порт для подключения SHH
```
Создание или настройка макросов хоста
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
