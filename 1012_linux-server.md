_2015/10/12_

# The Introduction of Linux Server

sshの設定を変える

* 公開鍵認証方式でsshできるようにする
* パスワードでのssh接続を向こうにする
* port番号を変える


## Note

CentOS7ではサービスの起動方法が変わる。

* CentOS7でのサービス（デーモン）の起動・停止方法

http://www.server-memo.net/centos-settings/centos7/service-start.html

`/etc/init.d/httpd start`は環境変数を引き継ぐので、`service httpd start`とかを
使う方がベター。

`service ~~~`はsytemdでの管理導入後でも使えるが、systamctlコマンドに
リダイレクトされるようになる。

* サービスの起動　　

```
systemctl start サービス名.service
```

* サービスの停止　　

```
systemctl stop サービス名.service
```

* サービスの再起動　

```
systemctl restart サービス名.service;w
```

## sshd_config

* PermitRootLoginどっちにする？
  * sudoするときにどちらにせよパスワードいれなきゃいけないじゃん！
  * 考えて、自分の中でポリシーをもっておくのが大事！

* むずかしいパスワードを覚える
  * 歌の頭文字とか

## RSAAuthentication / PasswordAuthentication


## Password

RPCが111ポート

https://ja.wikipedia.org/wiki/RPC

* 攻撃者側から見た侵入前の事前調査（下見） (2/2)

http://www.atmarkit.co.jp/ait/articles/0210/11/news001_2.html

とじておこう！

* 111番ポートが何故か開いてて困った困った

http://hogem.hatenablog.com/entry/20061217/1166368658




CentOS 5.5 + nginx + Ruby

普通にinstallできた。

Rubyはrbenv使いたいのでここを参考にする

http://qiita.com/inouet/items/478f4228dbbcd442bfe8

```
$ nginx -v
nginx version: nginx/0.8.55

$ ruby -v
ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-linux]

$ mysql -V
mysql Ver 14.12 Distrib 5.0.95, for redhat-linux-gnu (x86_64) using readline 5.1
```

### rubygems

How To Install Ruby on Rails on CentOS 6

https://www.digitalocean.com/community/tutorials/how-to-install-ruby-on-rails-on-centos-6





## Linuxのディレクトリ構成

* /home, /root 各ユーザ。ぐにゃぐにゃ変わる=>デフラグの対象
* /var ログ用、システム関連。ぐにゃぐにゃ
* /etc 環境設定管理 config 
* /bin 実行ファイル 
* /sbin 管理者(system)の実行ファイル
* /usr ユーザ用


## group

$ gropus

$ vi /etc/group

$ vi /etc/passwd


3つの8進数！



## その他参考

* 初心者の頃に知っておきたかった rpm と yum の違いと使い分け

http://blog.inouetakuya.info/entry/20111006/1317900802

