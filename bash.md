# 運用に使えそうなshellメモ

## あるセグメントにping(1)
```shell
$ echo 192.168.0.{1..254} | xargs -n 1 ping -c 1 -t 1
```

## あるセグメントにping(2)
```shell
$ seq 1 10 | sed 's/^/192.168.1.'/g > test.csv
```

## あるセグメントにping(3)
```shell
$ vi test.csv
192.168.0.1
192.168.0.2
192.168.0.3   
$ for i in `cat test.csv`; do ping -c 1 -t 1 $i; done
```

## 連番生成
```shell
$ touch test.log-20180{1..9}0{1..9}
```

## 複数階層のzipを生成
```shell
$ touch 1
$ zip -r 1.zip 1
$ for i in `seq 1 9`; touch 1 && zip 1.zip 1 && do zip -r $((i+1)).zip $i.zip;done
```

## lsの結果に対してtar.gz作成する
* 1ライナーでやる場合

```shell
$ for i in `ls`; do tar --remove-files zcvf $i.tar.gz $i; done
```

* アーカイブ作成と元ファイル削除を分けた場合

```shell
$ for i in `ls`; do tar zcvf $i.tar.gz $i; done;
$ for i in `ls | grep -v .tar.gz`; do rm -i $i; done;
```

## ダミーファイルでログローテート（ファイルの中身あり+.gz）
```shell
$ echo touch test.log-20180{1..9}0{1..9}
$ touch test.log-20180{1..9}0{1..9}
$ for i in `ls`; do date > $i; done
$ for i in `ls`; tar --remove-files zcvf $i.tar.gz $i; done
# logrotate -f /etc/logrotate.conf
```

## ファイルのコメント行と空白行を非表示
* コメントが#の場合

```shell
$ grep -v '^\s*#' /etc/httpd/conf/httpd.conf | grep -v '^\s*$'
$ grep -v -e '^\s*#' -e '^\s*$' /etc/httpd/conf/httpd.conf
```

* コメントが;の場合

```shell
$ grep -v '^\s*;' /etc/httpd/conf/httpd.conf | grep -v '^\s*$'
```

## あるディレクトリ配下の*.conf内のファイルにある文字列を検索
```shell
$ find /etc/httpd -name \*.conf | xargs grep -i xxx
```

## ファイル内の空行・コメントを除外してdiff
```shell
# diff -yb <( grep -v "^¥s*#" FILE_NAME_A | grep -v "^¥s*$" | sort -n ) <( grep -v "^¥s*#" FILE_NAME_B | grep -v "^¥s*$" | sort -n ) 

```


---

# SQLスニペット
* mysqlコマンドでログインする際のパスワード入力を省略する

```
# vi /root/.my.cnf
[client]
password="password"
# chmod 400 $!
```

以下、mysqlコマンドでSQLを実行する場合は、最初のmysql -u rootにだけ-pオプションをつけて、後は-pを省略すると良い。省略しないと処理の度にパスワードを尋ねられてしまう。

* -eの後に任意のSQLを指定して実行

```
# mysql -u root -p -e 'SELECT * FROM mysql.user;'
```

* インストール時のmysql,information_schema,performance_shcema以外のDBを削除

```
# mysql -u root -p -e 'show databases'
# mysql -u root -p -N -e 'show databases' | grep -v -w -e mysql -e information_schema -e performance_schema | xargs -n 1 -I {} mysql -u root -e "DROP DATABASE {}"
```

* root以外のユーザを削除

```
# mysql -u root -p -e "SELECT user,host FROM mysql.user WHERE not user='root';"
# mysql -u root -N -p -e "SELECT user,host FROM mysql.user WHERE not user='root';" | awk '{ print $1 }' | xargs -n 1 -I {} mysql -u root -e "DROP USER '{}'@'localhost';"
```

* ユーザのパスワード変更

```
# mysql -u root -p -e 'SELECT user,password FROM mysql.user;'
# mysql -u root -e "SET PASSWORD FOR 'ansible'@'localhost' = PASSWORD('ansible');"
```
