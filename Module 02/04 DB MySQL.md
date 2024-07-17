[Создание партиций](https://blog.zabbix.com/partitioning-a-zabbix-mysql-database-with-perl-or-stored-procedures/13531/)

Включение партиций для базы данных

подключаемся к MySQL
```
mysql -u root -p
```

```
use zabbix;
```

```
SELECT FROM_UNIXTIME(MIN(clock)) FROM history_uint;
```
Для работы параметры времени надо взять по дням от начала курса.
вместо 2020_12_19-22 подставить свои значения

таблица хистори по месяцам
```
ALTER TABLE history_uint PARTITION BY RANGE ( clock)
(PARTITION p2024_07 VALUES LESS THAN (UNIX_TIMESTAMP("2024-07-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2024_08 VALUES LESS THAN (UNIX_TIMESTAMP("2024-08-01 00:00:00")) ENGINE = InnoDB);
```

Для работы параметры времени надо взять по дням от начала курса.
вместо 2020_10-03 подставить свои значения

таблица трендов  по месяцам
```
ALTER TABLE trends_uint PARTITION BY RANGE ( clock)
(PARTITION p2024_07 VALUES LESS THAN (UNIX_TIMESTAMP("2024-07-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2024_08 VALUES LESS THAN (UNIX_TIMESTAMP("2024-08-01 00:00:00")) ENGINE = InnoDB);
```

Вариант сверху для примера и требует постоянного исправление даты

есть вариант более прогрессивынй при помощи скриптов perl

[Инструкция по настройке](https://blog.zabbix.com/partitioning-a-zabbix-mysql-database-with-perl-or-stored-procedures/13531/)



