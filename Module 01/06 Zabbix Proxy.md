# Настройка zabbix proxy



Для начала стоить отметить что zabbix-proxy сервер не имеет веб интерфейса, соответственно использует крайне мало системных ресурсов.

Устиановка MySQL для сервера Zabbix
```
apt update

apt install default-mysql-server
```

Далее идем на сайт заббикса и выбираем нужный дистрибутив
```
https://www.zabbix.com/ru/download?zabbix=6.4&os_distribution=debian&os_version=12&components=proxy&db=mysql&ws=
```

Шифрованное подключение

Прежде чем менять файл с настройками сгенерируем на своем компе psk фразу командой и записываем ее в файл
```
openssl rand -hex 32 > /etc/zabbix/zabbix_proxy.psk
```
Далее выводим этот ключ на экран командой 
```

tail /etc/zabbix/zabbix_proxy.psk
```
Сохраняем где нибудь у себя данный ключ он нам пригодиться при настройки серверной части заббикса
И вот теперь конфигурируем файл с настройками

```
nano /etc/zabbix/zabbix_proxy.conf
```

Меняем следующие значения:\
Server=    ваш ip adresss\
Hostname=  такой же нужно будет прописать на основном сервере\
Proxy=1
DBPassword=password
TLSAccept=psk
TLSConnect=psk\
TLSPSKFile=/etc/zabbix/zabbix_proxy.psk\
TLSPSKIdentity=test  - тут меняйте значение на свое - такое же должно использоваться на серверной части\

```
systemctl restart zabbix-proxy
systemctl enable zabbix-proxy
```

