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

таблица хистори рекомендация сделать по дням
```
ALTER TABLE history_uint PARTITION BY RANGE ( clock)
(PARTITION p2020_12_19 VALUES LESS THAN (UNIX_TIMESTAMP("2020-12-20 00:00:00")) ENGINE = InnoDB,
PARTITION p2020_12_20 VALUES LESS THAN (UNIX_TIMESTAMP("2020-12-21 00:00:00")) ENGINE = InnoDB,
PARTITION p2020_12_21 VALUES LESS THAN (UNIX_TIMESTAMP("2020-12-22 00:00:00")) ENGINE = InnoDB,
PARTITION p2020_12_22 VALUES LESS THAN (UNIX_TIMESTAMP("2020-12-23 00:00:00")) ENGINE = InnoDB
;
```

Для работы параметры времени надо взять по дням от начала курса.
вместо 2020_10-03 подставить свои значения

таблица трендов рекомендация сделать по месяцам
```
ALTER TABLE trends_uint PARTITION BY RANGE ( clock)
(PARTITION p2020_10 VALUES LESS THAN (UNIX_TIMESTAMP("2020-11-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2020_11 VALUES LESS THAN (UNIX_TIMESTAMP("2020-12-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2020_12 VALUES LESS THAN (UNIX_TIMESTAMP("2021-01-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2021_01 VALUES LESS THAN (UNIX_TIMESTAMP("2021-02-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2021_02 VALUES LESS THAN (UNIX_TIMESTAMP("2021-03-01 00:00:00")) ENGINE = InnoDB,
PARTITION p2021_03 VALUES LESS THAN (UNIX_TIMESTAMP("2021-04-01 00:00:00")) ENGINE = InnoDB);
```

Вариант сверху для примера и требует постоянного исправление дат

есть вариант более прогрессивынй при помощи скриптов perl

[Инструкция по настройке](https://blog.zabbix.com/partitioning-a-zabbix-mysql-database-with-perl-or-stored-procedures/13531/)



