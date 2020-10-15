# How to tune apache

## about multi processing module (MPM)

```shell
# httpd -V
Server version: Apache/2.4.6 (CentOS)
Server MPM:     prefork
```

### in case of prefork

- not use thraeds.
- child process forked are in charge of one session.
- use a lots of memory (not efficiently)

```shell
# vim /etc/httpd/conf.modules.d/00-mpm.conf
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
<IfModule mpm_prefork_module>
    StartServers 10　#The number of child server processes created at startup. The default is 5.
    MinSpareServers 20　←The number  アイドル状態にいる子サーバプロセスの最小（希望）個数。デフォルトは5。
    MaxSpareServers 400　←アイドル状態にいる子サーバプロセスの最大（希望）個数。デフォルトは10。
    ServerLimit 400　←MaxClientsに指定可能な値の上限。（MaxClientより大きな数にする）
    MaxClients 400　←応答できる同時リクエスト数。デフォルトは256。
    MaxRequestsPerChild 80　←個々の子サーバプロセスが扱うことのできるリクエストの総数。デフォルトは10000。0にすると無制限
</IfModule>

```

### in case of workes

### in case of event

## Configure kernel parameters
