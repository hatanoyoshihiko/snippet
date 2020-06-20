# gitメモ

## 利用事例
複数人でプレイブックの編集を行う場合を想定しています。

1.  ローカルPC-Aでansibleプレイブックを作成
2.  ansibleサーバにプレイブックをアップロードする
3.  アップロードされたファイルをPC-Bでダウンロードして編集したい
4.  PC-A, PC-Bで相互に編集したファイルをansibleサーバにアップロードしたい
5.  以上のファイル変更点を履歴として残して、変更点ごとに復元出来るようにしたい

## 仕組の概要
プレイブックやディレクトリを **リポジトリ** という入れ物で管理します。  
ローカルで編集したファイルをリモートにアップロードしたり、
リモートのファイルをローカルにダウンロードしたりし、編集したファイルは修正内容含め履歴として管理することが出来ます。

---

## 用語
##### ワークツリー
実際に作業しているディレクトリ。

##### インデックス
ワークツリーの変更内容を記録しておく場所です。  
ここに追加された情報をもとにリポジトリにコミットしていきます。
インデックスにないファイルはコミット出来ません。

##### リポジトリ
ファイルやディレクトリの状態を記録する場所です。ワークツリーのことです。
ローカルとリモート用があります。リポジトリの中に **ブランチ** という履歴の流れを分岐して管理するものがあります。

##### ブランチ
統合ブランチとトピックブランチがあります。  
ローカル、リモートいずれにも **master** というブランチが存在します。これを統合ブランチとします。  masterは全てのプレイブックが集約されている状態です。  

トピックブランチは機能追加やバグ修正など課題単位で作成する場合等に利用します。
例えば一部のプレイブックを試験的に修正し、テストする場合、ローカルにtestというブランチを作成し、そこでファイルの修正を行います。

その後、testをmasterにマージすることで最終的な反映とする、といった利用法になります。
統合ブランチには全てのファイル・ディレクトリを集約させ、トピックブランチにはその部品となるファイル・ディレクトリが配置されているというイメージです。

統合・トピックブランチを分ける利点は、別ブランチで編集を行うことで統合ブランチに直接的な影響を与えずにファイルの編集を行えることです。  
また修正起点がブランチごと(編集ファイル群)に細かく分かれるため、修正箇所を追跡しやすくなります。

##### HEAD
今いるブランチの最新のコミットのことです。

##### コミット  
ファイルやディレクトリの追加・変更等をリポジトリに記録することです。

##### プッシュ
ファイルやディレクトリの変更点を対向のブランチにアップロードさせることです。
例えば、ローカルブランチの内容をリモートブランチに反映させます。

##### プル
ファイルやディレクトリの変更点を対向のブランチからダウンロードすることです。
例えば、リモートブランチの内容をローカルブランチに反映させます。

##### マージ
ローカルのブランチAとブランチBを統合させることです。

---

## 環境準備
- windows
[git for windows](https://gitforwindows.org/)
<br>
- linux
`$ sudo yum install git` #RHEL
`$ sudo apt install git` #Debian

---

## 利用方法
利用事例ごとにまとめました。

### リモートgitリポジトリのプレイブックをまとめてローカルPCにダウンロード
```
$ cd GIT_DIR
$ git clone ssh://root@172.17.0.120/var/www/html/git/ansible.git
```

### ローカルPCで書いたコードをansibleサーバにアップロードする
```
$ cd GIT_DIR
$ git add <FILE> or DIR_NAME #インデックスに追加
$ git commit -m "COMMIT_COMMENT" #変更点のコメントを記載する
$ git push origin master #リモートリポジトリにアップロード
```
`$ git push リモートリポジトリ　ローカルブランチ名` #上記はoriginがリモートリポジトリのエイリアス

### リモートgitリポジトリの変更点をダウンロードする
`$ git pull`

### 強制的にpullする
git pull後に誤ってファイルを削除してしまった場合等。
```
$ git fetch origin master
$ git reset --hard origin/master
```
---

### ローカルPCのファイル状態を確認
- **modified** ローカルだけの変更点
- **Untracked files** まだアップロードしていないファイルorディレクトリ

```
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   inventory/inventory.ini
        modified:   roles/common/tasks/main.yml

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        document/
        site.retry
        site.yml

no changes added to commit (use "git add" and/or "git commit -a")
```

### 特定ファイルの過去の状態を確認
- ファイルのコミット履歴を確認
`$ git log <FILE>`
- あるコミット時点での差分を表示
`$ git diff <HASH> <FILE>`
`$ git diff HEAD^` #直前のコミットとその1つ前のコミットの比較
- 特定のコミット時点でのファイル内容を表示
`$ git blame <HASH> <FILE>`

### git addしたファイルを取り消す
commitする前にaddしたファイルを取り消したいときに。
`$ git status` #**new file** が追加されたファイル
`$ git reset head <FILE>` #取り消す処理

### git commitの打ち消し
取り消した履歴(git log)が残ります。
コミットログはコマンド実行後、直書き出来ます。

- 特定のコミットだけを打ち消す場合
`$ git revert head` #直前のコミットの打ち消し
`$ git revert <HASH>` #git logのハッシュ値を指定して打ち消し
`$ git revert --no-edit` #コメントなしでコミットする場合

- 2つ以上を打ち消す場合
`$ git revert -n <HASH>` #indexに戻すだけでコミットはしない
`$ git commit -m "まとめてコミット"` #revertしたファイルをまとめてコミット
``

### git commitの取り消し
履歴(git log)からも取り消したログが消えてしまうので注意。
`$ git reset --hard head^` #直前のコミット取り消し・ローカルリポジトリの内容も戻したいとき
`$ git reset --soft head^` #直前のコミット取り消し・ローカルリポジトリの内容はそのまま
`$ git reset <HASH>` #git logの**commit* に記載のハッシュ値指定でもOK。指定したハッシュ値のコミット状態に戻ります。

`$ git reflog` #コミット履歴を表示
`$ git reset --hard HEAD@{1}` #HEAD{1}の時点まで戻す

---
## ブランチ関連

### ブランチの変更
ルールとしてリモートのmasterブランチに全てのファイルが集約される運用だとします。
まずローカルのmasterブランチをコピーして、ローカル上の別ブランチで作業して、リモートのmasterブランチにマージしたりします。

- ブランチの作成とマージ
1.  `$ git branch BRANCH_NAME` #ブランチの作成 $git  branch -b BRANCH_NAMEでチェックアウトも同時に行う
2.  `$ git checkout BRANCH_A` #ブランチの変更
3.  `何かしらファイルをadd,commit`
4.  `$ git merge BRANCH_B` #カレントブランチBRANCH_AにBRANCH_Bがマージされる。
<br>

- ブランチのリベース

<br>

- ブランチの削除
```
$ git branch -a #ローカルとリモートのブランチを表示
$ git branch -d BRANCH_NAME #ローカルブランチを削除
$ git push -d origin BRANCH_NAME #リモートブランチを削除
```

### ローカルで削除したファイルをリストア
- 直前のコミット状態に戻す
`$ git checkout -f`

- ファイルを指定して戻す
`$ git checkout -f <FILE>`

### 作業中の状態を一時保存する
別ブランチで作業するため、現在の変更点を一時保存する場合に利用します。  
一時保存せずに別ブランチに移動してしまった場合、保存していなかった内容が別ブランチに反映されてしまいます。
※運用としては一時保存せずに一旦コミットしてpushしてしまう方法もOKです。

```
$ git stash #一時保存。コミット中の情報も保存される。
$ git stash list #一時保存された状態リストの表示
$ git stash show #一時保存されたファイルの一覧表示
$ git stash list -p #変更されたファイルの内容表示
$ git stash apply stash@{0} #一時保存から復帰。{}の中はgit stash listで確認した値を指定
$ git stash drop stash@{0} #一時保存情報を削除。  
$ git stash pop stash@{0} #復帰と一時保存情報削除を同時に行う。
```

---
### 特定のファイルをgit管理から除外したい場合
```
$ cd GIT_DIR
$ vi .gitignore
*.iso
*.DS_Store
*.retry
*.log
*.tmp
```

### git管理対象から削除したいとき
`$ git rm <FILE>`
`$ git rm -r DIR_NAME`
`$ git rm --cached <FILE>` #ローカルのファイルを残したまま管理対象から外す場合

---

## リモートリポジトリ関連
- リモートリポジトリの一覧を表示
`$ git remote -v`
<br>

- 接続するリモートリポジトリを追加
`$ git remote add origin ssh://USER@192.168.0.1/var/www/html/ansible.git`
<br>

- 接続しているリモートリポジトリを解除
`$ git remote rm origin`
<br>

- リモートリポジトリの情報を表示
`$ git remote show origin`
<br>

- ローカルの追跡ブランチの情報を更新（リモートブランチの情報と同期）
`$ git remote update -p` #-pはprune?
<br>

---

## ログ関連
- HEAD(最新コミット)の遷移を確認
`$ git reflog` #HEADの遷移を表示
git resetで戻すときに便利 # git reset --hard HEAD@{1}

<br>

- コミットログ
コミットのログをコメント付きで表示します。
`$ git log` #過去のコミットログを表示。コミットコメント付き。
`$ git log --name-status` #変更したファイルを表示
`$ git log -- DIR or FILE` #特定のディレクトリやファイルのみ
`$ git log --author='z00080'` #コミットした人の名前で表示
`$ git log --merges` #マージのみ。 --no-mergesはマージを除外。
<br>

- コミットログの詳細表示
コミットログの詳細を表示します。
`$ git show` #最新コミットとの差分表示
`$ git show COMMIT_NO` # $git reflogで確認した値

- インデックスとの差分を表示
`$ git diff` #カレントブランチと最新コミットの差分
`$ git diff HEAD^1` #インデックス指定も可能
`$ git diff --name-only` #変更点のあるファイルを表示
`$ git diff --cached` #git addした後にインデックスと最新のコミットの差を確認
<br>

- リモートサーバとの変更分を確認
```
$ git fetch
$ git log origin/master
```

---

## マージの衝突を解消する
ブランチtest1とtest2があり、その中に同じ編集ファイルがあり、編集内容が衝突（競合）することがあります。その際は下記のような対応を行います。

### git mergeを使う場合
##### ローカルブランチで衝突発生。
- 衝突発生
```
$ git merge test2
Auto-merging test2
CONFLICT (content): Merge conflict in test2
Automatic merge failed; fix conflicts and then commit the result.
```

- 衝突ファイルの調査
test2が衝突していることが分かります。

```
$ git status
On branch test1
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:   test2
```

- 衝突内容を表示

===で区切られた上がtest1ブランチ、下がtest2ブランチの内容です。

##### 修正前
```
$ vi test2
<<<<<<< HEAD
add from test1
=======
add from test2
>>>>>>> test2
```

##### 修正後
```
$ vi test2
add from test1
add from test2
```

##### コミット
ブランチtest1で。
```
$ git add test2
$ git commit -m "merge test1 and test2"
```

---

##### push時に衝突発生
ローカル→リモートへのpushで衝突発生。
リモートの方が最新であるため、ローカルでpullせずにpushしてしまった状況です。

- git pushで衝突発生
```
$ git push origin master
root@172.17.0.120's password:
To ssh://172.17.0.120/var/www/html/git/test.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'ssh://root@172.17.0.120/var/www/html/git/test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```

- 衝突解消
「test」というファイルで衝突が発生しています。
```
$ git fetch
$ git merge origin/master
Auto-merging test
CONFLICT (content): Merge conflict in test
Automatic merge failed; fix conflicts and then commit the result.
```

- 衝突内容の修正
`$ vi test`
<br>
- 改めてプッシュ
```
$ git add test
$ git commit "fix conflict"
$ git push origin master
```

### git rebaseを使う場合
ローカル→リモートへのpushで衝突発生。
リモートの方が最新であるため、ローカルでpullせずにpushしてしまった状況です。

```
$ git push origin master
root@172.17.0.120's password:
To ssh://172.17.0.120/var/www/html/git/test.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'ssh://root@172.17.0.120/var/www/html/git/test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```


## git hook
git commitやpushをトリガーにリモートリポジトリサーバでスクリプトを実行します。
下記は例としてgit pushされたら、リモートリポジトリサーバでgit pullを実行します。
templateの設定は行わなくても構いません。

- テンプレートディレクトリの作成(リモートサーバ)
```
$ cd GIT_DIR
$ git config --global init.templatedir /var/www/html/test.git/hooks
$ git config --global -l
$ vi hooks/post-receive
#!/bin/bash
cd /root/work/test
git --git-dir=.git pull
```

- git push(クライアント)
```
$ git add <FILE>
$ git commit -m "add date"
$ git push origin master
Counting objects: 4, done.
Writing objects: 100% (3/3), 270 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: From ssh://localhost/var/www/html/test
remote:    1b2d49c..b7c3592  master     -> origin/master
remote: Updating 1b2d49c..b7c3592
remote: Fast-forward
remote:  date | 1 +
remote:  1 file changed, 1 insertion(+)
remote:  create mode 100644 date
To ssh://root@192.168.189.130/var/www/html/test.git
   1b2d49c..b7c3592  master -> master
```
##設定
-   基本的な設定
```
$ cd GIT_DIR
$ git config user.name XXX
$ git config user.email XXX

$ git config --system 全ユーザの全リポジトリに反映
$ git config --global ログイン中ユーザの全リポジトリに反映
```
- 設定値確認
`$ git config -l`

<br>

- リモートリポジトリを追加
```
$ git remote -v #現在の設定値を確認
$ git remote add ssh://root@192.168.0.1/var/www/html/test.git
```
