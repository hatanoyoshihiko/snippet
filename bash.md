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
