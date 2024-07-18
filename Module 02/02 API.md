# Модуль 2.
# Лабораторнная 2: Использование API

### Сценарий

Для управление zabbix требуется использовать автоматизацию. Использование web интерфейса, не повзоляет производить управление Zabbix без человека, а так же сделать интеграцию с другими система. Для автоматизации управления zabbix решено использовать Zabbix API.

### Цели

После прохождения лабораторной работы вы сможете:
```
- Создавать токен для подключения к API.

- Выполнять запросы к zabbix через API.

```

### Настройка лабораторной

Виртуальные машины: Zabbix

Пользователь: **root**

Пароль:  **Pa$$w0rd**

**Настройка Лабораторной**

Для этой лабораторной работы вы будете использовать доступную среду виртуальной машины. Прежде чем приступить к лабораторной работе, необходимо выполнить следующие шаги:

1. На главном компьютере запустите **Диспетчер Hyper-V**.

2. В **Диспетчере Hyper-V** выберите **Zabbix**, а затем на панели **Действия** выберите **Запустить**.

3. Запустите приложение Putty. В сохраненых подключения выберите **Zabbix** и выберите **открыть**

4. Войдите в систему, используя следующие учетные данные:

 - Имя пользователя: **root**

 - Пароль: **Pa$$w0rd**

5. Для доступа в интернет, запустите  **Gate**.


## Задание 1: Использование API Zabbix

#### Шаг 1: Получение API токена

1. На главном компьютере запускем Chrome и в браузере вводим **http://zabbix.corp1.ru**

2. На странице привествия вводим логин **Admin** и пароль **zabbix**

3. Переходим на быстром меню на вкладку **Users** - **Api tokens**

4. Выбираем **Create API token**
```
      Name: api test
      User: select Admin
      Set expiration date and time: disable
```    
4. Выбираем **Add**

5. В новом окне выбираем **copy to clipboard**

6. Сохраняем у себя данный ключ в блокноте

#### Шаг 2: Использование API


1. Установка утилити для обращения к API. Для примеров работы с API.

```
apt install curl
```

2. Создание переменной для API

Создаем переменную для bash и записывает значение токен в нее   
```
export AUTH=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
3. Получение узлов из zabbix

Создание скрипта
```
nano /root/zab_get_hosts_env.sh
```
```
#!/bin/sh

curl -s -k -X POST -H 'Content-Type: application/json-rpc' -d "
{
    \"jsonrpc\": \"2.0\",
    \"method\": \"host.get\",
    \"params\": {
       \"output\": [\"hostid\", \"host\"],
       \"selectInterfaces\": [\"interfaceid\", \"ip\"]
    },
    \"auth\": \"${AUTH}\",
    \"id\": 2
} " http://<имя сервера>/api_jsonrpc.php
```
```
chmod +x /root/zab_get_hosts_env.sh
```

```
# /root/zab_get_hosts_env.sh | jq
```
Создание скрипта
```
nano /root/zab_get_hosts.sh
```
```
#!/bin/sh

curl -s -k -X POST -H 'Content-Type: application/json-rpc' -d "
{
    \"jsonrpc\": \"2.0\",
    \"method\": \"host.get\",
    \"params\": {
    },
    \"auth\": \"${AUTH}\",
    \"id\": 2
} " http://<имя сервера>/api_jsonrpc.php
```
```
chmod +x /root/zab_get_hosts.sh
```

```
# /root/zab_get_hosts.sh | jq '.result | .[] | .name'
```

4. Получение списка карт и их элементов из Zabbix

```
# nano /root/zab_get_maps.sh
```
```
#!/bin/sh

curl -s -k -X POST -H 'Content-Type: application/json-rpc' -d "
{
    \"jsonrpc\": \"2.0\",
    \"method\": \"map.get\",
    \"params\": {
        \"selectLinks\": \"extend\",
        \"selectSelements\": \"extend\"
    },
    \"auth\": \"${AUTH}\",
    \"id\": 2
} " http://<имя сервера>/api_jsonrpc.php

```
```
chmod +x /root/zab_get_maps.sh
```
```
# /root/zab_get_maps.sh | jq -c '.result | .[] | {name: .name, id: .sysmapid}'
```

5. Пример изменения конфигурации через Zabbix API

Добавление элемента

```
# nano /root/additem.sh
```
```
#!/bin/sh

curl -s -k -X POST -H 'Content-Type: application/json-rpc' -d "
{
 \"jsonrpc\": \"2.0\",
 \"method\": \"item.create\",
 \"params\": {
    \"name\": \"Free disk space on $1\",
    \"key_\": \"vfs.fs.size[/home/seanwasere/,free]\",
    \"hostid\": \"10084\",
    \"type\": 0,
    \"value_type\": 3,
    \"interfaceid\": \"1\",
    \"delay\": 30s
    },
    \"auth\": \"api token\",
    \"id\": 2
}" http://zabbix.corp1.ru/api_jsonrpc.php
```
```
chmod +x /root/additem.sh
```
Изминение имени карты

```
# nano /root/zab_set_map_name.sh
```

```
#!/bin/sh

MAPID=$1
MAPNAME=$2

curl -s -k -X POST -H 'Content-Type: application/json-rpc' -d "
{
    \"jsonrpc\": \"2.0\",
    \"method\": \"map.update\",
    \"params\": {
        \"sysmapid\": \"${MAPID}\",
        \"name\": \"${MAPNAME}\"
    },
    \"auth\": \"${AUTH}\",
    \"id\": 2
} " http://<имя сервера>/api_jsonrpc.php
```
```
chmod +x /root/zab_set_map_name.sh
```
```
# /root/zab_set_map_name.sh <id> <Name map>
```
