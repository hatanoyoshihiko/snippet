# SQL スニペット

## mysql コマンドでログインする際のパスワード入力を省略する

```shell
# vi /root/.my.cnf
[client]
password="password"
# chmod 400 $!
```

以下、mysql コマンドで SQL を実行する場合は、最初の mysql -u root にだけ-p オプションをつけて、後は-p を省略すると良い。省略しないと処理の度にパスワードを尋ねられてしまう。

## -e の後に任意の SQL を指定して実行

```shell
# mysql -u root -p -e 'SELECT * FROM mysql.user;'
```

## インストール時の mysql,information_schema,performance_shcema 以外の DB を削除

```shell
# mysql -u root -p -e 'show databases'
# mysql -u root -p -N -e 'show databases' | grep -v -w -e mysql -e information_schema -e performance_schema | xargs -n 1 -I {} mysql -u root -e "DROP DATABASE {}"
```

## root 以外のユーザを削除

```shell
# mysql -u root -p -e "SELECT user,host FROM mysql.user WHERE not user='root';"
# mysql -u root -N -p -e "SELECT user,host FROM mysql.user WHERE not user='root';" | awk '{ print $1 }' | xargs -n 1 -I {} mysql -u root -e "DROP USER '{}'@'localhost';"
```

## ユーザのパスワード変更

```shell
# mysql -u root -p -e 'SELECT user,password FROM mysql.user;'
# mysql -u root -e "SET PASSWORD FOR 'ansible'@'localhost' = PASSWORD('ansible');"
```

## root ユーザのパスワードを忘れてしまったとき

```shell

```

## SQL 実行時の枠線やヘッダー非表示

- ヘッダなし

```shell
# mysql -u root -p -N -e "show databases;"
```

- ヘッダ・枠線なし

```shell
# mysql -u root -p -s -e "show databases;"
```

## ある DB 内テーブルのレコード数を調査する

- SQL を実行

```shell
$ mysql -u root -ppassword -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

- カラム名非表示

```shell
$ mysql -u root -ppassword -N -e "show databases;"
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```

- 枠線非表示

```shell
$ mysql -u root -ppassword -s -e "show databases;"
Database
information_schema
mysql
performance_schema
```

- カラム名、枠線非表示

```shell
$ mysql -u root -ppassword -N -s -e "show databases;"
information_schema
mysql
performance_schema
```

- 各 DB のテーブル内レコード数をカウント

```shell
#!/bin/bash

DB_USER=root
PASSWORD=password
DB_NAME=mysql
TABLES=$(mysql -u $DB_USER -p$PASSWORD -D $DB_NAME -sN -e "show tables;")
SQL=""

for TBL in ${TABLES[@]}; do
	SQL="${SQL} SELECT '${TBL}' AS table_name, COUNT(*) AS table_count FROM ${TBL} UNION ALL"
done

SQL="$(echo $SQL | sed -e 's/ UNION ALL$//')"

mysql -u $DB_USER -p$DB_USER -p$PASSWORD -D $DB_NAME -e "${SQL}"

```

実行結果

```sql
+---------------------------+-------------+
| table_name                | table_count |
+---------------------------+-------------+
| columns_priv              |           0 |
| db                        |           0 |
| event                     |           0 |
| func                      |           0 |
| general_log               |           0 |
| help_category             |          39 |
| help_keyword              |         464 |
| help_relation             |        1028 |
| help_topic                |         508 |
| host                      |           0 |
| ndb_binlog_index          |           0 |
| plugin                    |           0 |
| proc                      |           0 |
| procs_priv                |           0 |
| proxies_priv              |           2 |
| servers                   |           0 |
| slow_log                  |           0 |
| tables_priv               |           0 |
| time_zone                 |           0 |
| time_zone_leap_second     |           0 |
| time_zone_name            |           0 |
| time_zone_transition      |           0 |
| time_zone_transition_type |           0 |
| user                      |           3 |
+---------------------------+-------------+

```
