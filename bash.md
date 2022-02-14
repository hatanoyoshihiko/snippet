# 運用に使えそうな shell メモ

## あるセグメントに ping(1)

```shell
$ echo 192.168.0.{1..254} | xargs -n 1 ping -c 1 -t 1
```

## あるセグメントに ping(2)

```shell
$ seq 1 10 | sed 's/^/192.168.1.'/g > test.csv
```

## あるセグメントに ping(3)

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

## 複数階層の zip を生成

```shell
$ touch 1
$ zip -r 1.zip 1
$ for i in `seq 1 9`; touch 1 && zip 1.zip 1 && do zip -r $((i+1)).zip $i.zip;done
```

## ls の結果に対して tar.gz 作成する

- 1 ライナーでやる場合

```shell
$ for i in `ls`; do tar --remove-files zcvf $i.tar.gz $i; done
```

- アーカイブ作成と元ファイル削除を分けた場合

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

- コメントが#の場合

```shell
$ grep -v '^\s*\(#\|$\)' /etc/httpd/conf/httpd.conf
$ grep -v -e '^\s*#' -e '^\s*$' /etc/httpd/conf/httpd.conf
```

- コメントが;の場合

```shell
$ grep -v '^\s*\(#\|$\)' /etc/httpd/conf/httpd.conf | grep -v '^\s*$'
```

## あるディレクトリ配下の\*.conf 内のファイルにある文字列を検索

```shell
# find /etc/httpd -name \*.conf | xargs grep -i WORD
```

## ファイル内の空行・コメントを除外して diff

```shell
# diff -yb <( grep -v '^\s*\(#\|$\)' FILE_NAME_A | sort -n ) <( grep -v '^\s*\(#\|$\)' FILE_NAME_B | sort -n )
```

## 特定のプロセス群を kill する

pgrep, pkill は拡張正規表現。

- 対象プロセスの確認
  ps の[COMMAND]列で表示される箇所全体を検索対象とし、PID とコマンド文字列を表示する。

  ```shell
  # vmstat > /dev/null &
  # pgrep -af vmstat
  2482 vmstat 1 #vmstatの引数1まで確認出来る
  ```

- 対象プロセスのコマンド全体を検索対象にし、PID を表示する

  ```shell
  # pgrep -fl vmstat
  2482 vmstat #こっちは vmstat の引数までは表示されない
  ```

- 実際にプロセスを落とす
  pgrep -fl の結果で vmstat の PID を特定できたので、pkill を実行する。

  ```shell
  # pgrep -fl vmstat
  # pkill -ex vmstat #プロセス名と完全に一致したものを対象にする
  vmstat killed (pid 2482)
  ```

- grep を駆使する場合

  ```shell
  # vmstat 1 > /dev/null &
  # ps auxf | head -n 1 && ps auxf | grep -i [v]mstat #vmstatのPIDを確認
  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root      2713  0.0  0.7  42308  3544 pts/2    S    11:45   0:00                      \_ vmstat 1

  # ps auxf | grep -i [v]mstat | awk '{print $2}' #vmstatのPIDだけを抽出
  2713

  # ps auxf | grep -i [v]mstat | awk '{print $2}' | xargs kill -n 15 #killで終了させる
  ```

## 置換

- 単純置換

  ```shell
  # cat test.txt
  http://www.google.co.jp/

  # sed -e 's/google/yahoo/g' test.txt
  http://www.yahoo.co.jp/
  ```

- パターンマッチした文字列を置き換える

  ```shell
  # cat test.txt
  http://www.alessiareya.com/htdocs/dir/index.html

  # sed -e 's/\(\/htdocs\)\(\/dir\)/\2\1/g' test.txt
  http://www.alessiareya.com/dir/htdocs/index.html

  デリミタを/から:に変更
  # sed -e 's:\(/htdocs\)\(/dir\):\2\1:g' test.txt
  http://www.alessiareya.com/dir/htdocs/index.html
  ```

- xml のコメントと空行を削除する

  ```shell
  # cat test.xml
  <--! comment -->
  cooment
  <--! comment -->
  cooment
  <--! comment -->
  cooment

  <--!
  comment
  comment
  comment
  -->

  # sed -e 's/<--!.*-->//' test.xml | sed -e '/<--!/,/-->/d' | sed -e '/^$/d'
  cooment
  cooment
  cooment
  ```

## crontab

- PATH 変数
  crontab 内で PATH 変数を定義しない場合、/bin, /usr/bin しかパスが通っていない。
  特に指定がなければ実行ユーザのワーキングディレクトリとして実行される。

- crontab 内での PATH 変数を調べる
  PATH 変数が/usr/bin:/bin、PWD が/root, SHELL は/bin/sh であることがわかる。

```shell
# crontab -e
*/1 * * * * /bin/env > env.txt
XDG_SESSION_ID=12
SHELL=/bin/sh
USER=root
PATH=/usr/bin:/bin
PWD=/root
LANG=ja_JP.utf8
SHLVL=1
HOME=/root
LOGNAME=root
XDG_RUNTIME_DIR=/run/user/0
_=/bin/env
```

- crontab でパスの通っていないコマンドを実行する(/sbin)

````shell
# crontab -e
*/1 * * * * /sbin/arp > sbin_arp.txt
*/1 * * * * arp > arp.txt```shell
````

/sbin/arp の方だけ実行結果が存在する

```shell
# ls -l
-rw-r--r--  1 root root    0 Jul 17 22:47 arp.txt
-rw-r--r--  1 root root  321 Jul 17 22:47 sbin_arp.txt

# cat arp.txt

# cat sbin_arp.txt
Address                  HWtype  HWaddress           Flags Mask            Iface
gateway                  ether   52:54:00:12:35:02   C                     eth0
ap01                     ether   dc:fb:02:01:ca:9a   C                     eth2
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0
```

- crontab に PATH 変数を定義する
  PATH=\$PATH のように変数展開は出来ない。

````shell
PATH=/sbin:/bin:/usr/bin
*/1 * * * * /sbin/arp > sbin_arp.txt
*/1 * * * * arp > arp.txt```shell
````

- PATH 変数定義後の実行結果
  arp, /sbin/arp の両方が実行されている。

```shell
# ls -l
-rw-r--r--  1 root root  321 Jul 17 22:58 arp.txt
-rw-r--r--  1 root root  321 Jul 17 22:58 sbin_arp.txt

# cat arp.txt
Address                  HWtype  HWaddress           Flags Mask            Iface
gateway                  ether   52:54:00:12:35:02   C                     eth0
ap01                     ether   dc:fb:02:01:ca:9a   C                     eth2
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0

# cat sbin_arp.txt
Address                  HWtype  HWaddress           Flags Mask            Iface
gateway                  ether   52:54:00:12:35:02   C                     eth0
ap01                     ether   dc:fb:02:01:ca:9a   C                     eth2
10.0.2.3                 ether   52:54:00:12:35:03   C                     eth0
```

- ワーキングディレクトリを変更する
  crontab 内で PWD 変数を書き換えても実行出来ない模様。
  cd でごまかすしかない。

```shell
# vi /usr/local/sbin/test.sh
#!/bin/bash
/bin/echo 'change_working_directory' > echo.txt

# chmod +x !$

# crontab -e
PATH=/sbin:/bin:/usr/bin
*/1 * * * * /bin/cd /usr/local/sbin; test.sh

# cat /root/echo.txt
change_working_directory
```

## rsync での文字コード処理

- --iconv=sjis,utf8
  rsync 元ファイル名の文字コードを sjis として認識し、utf8 に変換して rsync 先に同期する。
  rsync を実行する側のオプションが先になる。
  実行サーバ=同期元の場合は--iconv=cp932,utf8。実行サーバ=同期先の場合は--iconv=utf8,cp932

- --protect-args
  ファイル名やディレクトリ名の空白をエスケープなしで処理する。
  通常、rsync で上記ファイルを同期する場合はエスケープ処理が必要。

- よく使う同期例  
  ミラーリング  
  `# rsync --delete -n -azv --protect-args --log-file=/var/log/rsync.log SRC/ DST/`

- 文字コード調査
  意図的に sjis のファイル名を作成して調査する。

```
# echo '変な文字' | iconv -f utf8 -t sjis | xargs touch
# ls -i1
33585221 ????????
# find ./ -inum 33585221 | nkf -g
Shift_JIS
```

## ネットワーク IO 調査

- プロセスの
  `# strace -p 20471`

- read コールが unfinished のため、そのファイルディスクリプタを調べる  
  `# readlink /proc/20471/fd/58`

- netstat で socket 番号を grep して、何のプロセスか調べる  
  `# netstat -ane | grep 11111`
****
下記でも良い。  
`# lsof -i -a -p pid` -a は and 条件  
`# lsof -iTCP4 -a -p pid`  

## 計画シャットダウン

2022/2/15 17:50にシャットダウン例  
`# echo "systemctl poweroff" | at 17:50 02152022`

キューの確認  
```bash
# atq
1 Mon Feb 14 19:10:00 2022 a root
```

キューの削除  
`# at -d 1`
