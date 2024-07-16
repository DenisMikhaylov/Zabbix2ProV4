Мониторинг JMX

Настройка приложения для мониторинга

Для примера будем использовать приложение tomcat

Подключиться к серверу Debian-1

Настройка и установка TomCat server

Создаем пользователя
```
useradd -m -d /opt/tomcat -U -s /bin/false tomcat
```
Обновляем репозитарий приложений
```
apt update
```
Устанавливаем JDK
```
apt install default-jdk
```

Проверяем версию Java
```
java -version
```
Скачиваем пакет и устанавливаем Apache-TomCat
```
wget https://dlcdn.apache.org/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
```
```
tar xzvf apache-tomcat-10.1.26.tar.gz -C /opt/tomcat --strip-components=1
```
Настраиваем права
```
chown -R tomcat:tomcat /opt/tomcat/
chmod -R u+x /opt/tomcat/bin
```
Переходим в папку 
```
cd /opt/tomcat/bin
```
Запускаем для проверки TomCat server
```
./catalina.sh start
```
В браузере проверяем работу
```
http://[IP address of server running Tomcat]:8080
```
Настройка мониторинга JMX

Переходим в папку
```
cd /opt/tomcat/bin
```
Создаем файл
```
nano setenv.sh
```
Вставляем в файл строку ниже и подставляем ip адрес сервера tomcat
```
CATALINA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9000 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false -Djava.rmi.server.hostname=[IP address of server running Tomcat]"
```
Настройка прав
```
chmod 750 setenv.sh
chown -R tomcat:tomcat setenv.sh
```
Перезапускаем сервер TomCat
```
./catalina.sh stop
./catalina.sh start
```
Проверяем открытый порт
```
ss -ntlp | grep 9000
```
Настройка в Zabbix JMX

Создать хост Debian-1
```
Параметры агента :
Тип: JMX
IP Адрес:  Debian-1
Порт: 9000
Добавить шаблон: Apache Tomcat by JMX
```
Проверка значений 
```
Monitors -> Latest data-> Debian-1
```
