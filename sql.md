# SQL スニペット

- mysql コマンドでログインする際のパスワード入力を省略する

```shell
# vi /root/.my.cnf
[client]
password="password"
# chmod 400 $!
```

以下、mysql コマンドで SQL を実行する場合は、最初の mysql -u root にだけ-p オプションをつけて、後は-p を省略すると良い。省略しないと処理の度にパスワードを尋ねられてしまう。

- -e の後に任意の SQL を指定して実行

```shell
# mysql -u root -p -e 'SELECT * FROM mysql.user;'
```

- インストール時の mysql,information_schema,performance_shcema 以外の DB を削除

```shell
# mysql -u root -p -e 'show databases'
# mysql -u root -p -N -e 'show databases' | grep -v -w -e mysql -e information_schema -e performance_schema | xargs -n 1 -I {} mysql -u root -e "DROP DATABASE {}"
```

- root 以外のユーザを削除

```shell
# mysql -u root -p -e "SELECT user,host FROM mysql.user WHERE not user='root';"
# mysql -u root -N -p -e "SELECT user,host FROM mysql.user WHERE not user='root';" | awk '{ print $1 }' | xargs -n 1 -I {} mysql -u root -e "DROP USER '{}'@'localhost';"
```

- ユーザのパスワード変更

```shell
# mysql -u root -p -e 'SELECT user,password FROM mysql.user;'
# mysql -u root -e "SET PASSWORD FOR 'ansible'@'localhost' = PASSWORD('ansible');"
```
