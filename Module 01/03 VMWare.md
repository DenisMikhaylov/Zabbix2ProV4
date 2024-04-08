Подключение VMware

Настройка встроенного шаблона

Подключиться на браузере к серверу Zabbix
```
http://<ip zabbix>
```
Создать новый хост для подключения шаблона VMWare
```
Host name: VMWare
  interface agent: ip 127.0.0.1
  New group: Virtual
  Template: VMWare
```
Настройка шаблона 
```
{$VMWARE.PASSWORD} = Pa$$w0rd
{$VMWARE.URL} = https://vcenter.corp1.ru/sdk
{$VMWARE.USERNAME} = student@corp1.ru
```
Настройка конфигурации zabbix
```
nano /etc/zabbix/zabbix_server.conf
```
```
StartVMwareCollectors=1
```
Перезапускаем cсервер
```
systemctl restart zabbix-server
```
