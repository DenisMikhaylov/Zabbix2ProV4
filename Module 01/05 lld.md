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

Устанавливаем приложение

```
# apt install dhcpd-pools
```
```
apt install jq
```
```
apt install xmlstarlet
```

Настройка мониторинга DHCP сервера

Создаем скрипт

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
Разрешаем выполняться скрипту 
```
chmod +x /etc/zabbix/dhcp-pools-discovery.sh
```

Проверяем работу

```
# /etc/zabbix/dhcp-pools-discovery.sh | jq
```

Создание 2 скрипт

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
Разрешаем выполняться скрипту 
```
chmod +x /etc/zabbix/dhcp-pools-shared-network.sh
```

Проверяем работу

```
# /etc/zabbix/dhcp-pools-shared-network.sh LAN1 defined

# /etc/zabbix/dhcp-pools-shared-network.sh LAN1 used
```

Создаем файл настроек для агента.
```
# nano  /etc/zabbix/zabbix_agentd.conf.d/dhcp_stat.conf
```
```
UserParameter=dhcp.pools.discovery,/etc/zabbix/dhcp-pools-discovery.sh

UserParameter=dhcp.pools.shared-network[*],/etc/zabbix/dhcp-pools-shared-network.sh $1 $2
```
Настройка в Zabbix 

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
    Trigger prototypes

      Name: On {HOST.NAME} in the DHCP pool {#POOLNAME}
     
      Expression:
```
строчку нужно сгенерировать.Пример возможной строки.
```
 last(/Template App DHCP Pools/dhcp.pools.shared-network[{#POOLNAME},used])/last(/Template App DHCP Pools/dhcp.pools.shared-network[{#POOLNAME},defined])*100 > {$DHCP.POOLS.MAX.PERCENT}
```
```
      Severity: Warning
```
Редактируем файл настройки DHCP

```
nano /etc/dhcp/dhcpd.conf
```
Находим блок с subnet
```
добавлем ниже

shared-network LAN2 { 

  subnet 192.168.30.0 netmask 255.255.255.0 {
    range 192.168.30.101 192.168.30.109;
    option routers 192.168.30.1;
  }

} 
```

Проверить работы discovery.

