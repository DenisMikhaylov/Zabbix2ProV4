Мониторинг SSH

Настройка мониторинга на стороне Zabbix
Подключиться на браузере к серверу Zabbix
```
http://<ip zabbix>:8080
```
Настройка элемента мониторинга
```
Host: Zabbix
  Items
    Name: CPU Usage
    Type: SSH agent
    Key: ssh.run[cpu]
    Executed script: top -b -n 10 -d.2 | grep 'Cpu' |  awk 'NR==3{ print($2)}'
    Tag: ssh
```
Настройка элемента мониторинга
```
Host: Zabbix
  Items
    Name: Passwd MD5
    Type: SSH agent
    Key: ssh.run[md5]
    Executed script: md5sum /etc/passwd |awk '{print $1}'
    Tag: ssh
```
Проверка значений 
```
Monitors -> Latest data-> Passwd MD5
Monitors -> Latest data-> CPU Usage
Monitors -> Latest data-> tag: ssh
```
``
