
# DNS

* chroot
	* bind (package)
* zone file
	* table of 
		* ip address
		* host name


http://www.sea-bird.org/pukiwiki/index.php?CentOS6%20%2F%20BIND%A4%CE%C0%DF%C4%EA%A4%CE%B4%AC


## 参考

* 小悪魔女子大生のサーバエンジニア日記

https://co-akuma.directorz.jp/blog/


* DNSサーバーBind（バインド）の設定。（CentOS）

http://www.ohoclick.com/archives/13



* DNSサーバー設定

DigitalOceanのチュートリアル。他の日本語サイトが簡素な説明しかないのに比べかなりよくまとめてくれているように思う。

https://www.digitalocean.com/community/tutorials/how-to-install-the-bind-dns-server-on-centos-6




* DNSサーバーの作り方

いい感じ。

https://nextstageone.g.hatena.ne.jp/satakesatake/20100923/1285229023






## BINDのインストール

BINDをインストールします。

```
# yum -y install bind bind-utils bind-chroot
```

そして各種設定。

>まとめると「/etc/host.conf」「/etc/nsswitch.conf」共にリゾルバ用設定ファイルです。
名前解決する際の、参照先の優先順序を書いておきます。
「/etc/host.conf」が旧型で「/etc/nsswitch.conf」が新型です。
>via: http://piyopiyocs.blog115.fc2.com/blog-entry-678.html

いくつか設定ファイルを確認しておく。


## chrootの設定

>BINDのchroot化 することで万が一ハックされても /var/named より上位にはアクセスすることができなくなるためセキュアな環境を構築することができます。ここでは、chrootを入れた環境設定を紹介します。
http://www.sea-bird.org/pukiwiki/index.php?CentOS6%20%2F%20BIND%A4%CE%C0%DF%C4%EA%A4%CE%B4%AC#qaefa724


<!-- TODO

<script src="https://gist.github.com/totzyuta/2bdb7a6a445511805c83.js"></script>

-->


* /var/named/chroot/var/named/sea-bird.org.wan 外部の正引き情報



```
; /var/named/chroot/var/named/coffeeokadai.com.wan
$TTL    86400
@       IN      SOA     ns.coffeeokadai.com. root.sea-bird.org. (
                                20030812 ; Serial
                                10800    ; Refresh after 3 hours
                                3600     ; Retry after 1 hours
                                604800   ; Expire after 1 week
                                86400    ; Minimum TTL of 1 day
                                       )
;
        IN      NS      ns.coffeeokadai.com.
        IN      NS      dns1.open-circuit.ne.jp
;
        IN      A       153.122.35.112        ; TODO: 自分のグローバルIP
        IN      MX      10      mail.coffeeokadai.com.
;
localhost       IN      A       127.0.0.1
ns      IN      A       153.122.35.112        ; TODO: 自分のグローバルIP
;
www     IN      A       153.122.35.112       ; TODO: 自分のグローバルIP
mail    IN      A       153.122.35.112       ; TODO: 自分のグローバルIP
blog    IN      CNAME   ns.coffeeokadai.com.
```



* /var/named/chroot/var/named/sea-bird.org.lan 内部の正引き情報


```
; /var/named/chroot/var/named/coffeeokadai.com.lan

$TTL    86400
@       IN      SOA     coffeeokadai.com.   root.coffeeokadai.com. (
                                        2004031901 ; Serial
                                        28800      ; Refresh
                                        14400      ; Retry
                                        3600000    ; Expire
                                        86400 )    ; Minimum
        IN      NS      coffeeokadai.com.
        IN      MX      xx      coffeeokadai.com.
@       IN      A       192.168.0.xx    ; TODO: 公開するサーバアドレス(もちろんローカルIP)
*       IN      A       192.168.0.xx    ; TODO: 公開するサーバアドレス(もちろんローカルIP)
```


* /var/named/chroot/var/named/xxx.168.192.in-addr.arpa 内部の逆引き情報

```
; /var/named/chroot/var/named/1.168.192.in-addr.arpa
$TTL    86400
@       IN      SOA     coffeeokadai.com.   root.coffeeokadai.com. (
                                        2004031901 ; Serial
                                        28800      ; Refresh
                                        14400      ; Retry
                                        3600000    ; Expire
                                        86400 )    ; Minimum
        IN      NS      coffeeokadai.com.
        IN      MX      xx      coffeeokadai.com.
1      IN      PTR     coffeeokadai.com.      ; TODO: 1は、公開するサーバアドレス(もち >ろんローカルIP)の下位
1      IN      PTR     window-shop.biz.   ; TODO: 1は、公開するサーバアドレス(もちろん >ローカルIP)の下位
```


### 文法チェック

ここまでできたら、BINDを起動する前に文法をチェックします。

http://www.aconus.com/~oyaji/dns/bind_linux.htm

named-checkconfの使い方は以下がわかりやすかったです


* named-checkconf コマンドで BIND の設定をチェック

http://futuremix.org/2010/03/bind-named-checkconf-error-check-command




#### エラー集

(これらのエラーの修正点は先ほどの設定ファイルに反映してます)

エラーが出ました。

##### DNSSEC

DNSSECというDNSのUDP通信をセキュアにするための設定。

```
[root@sub0000539504 ~]# named-checkconf
/etc/named.conf:13: unknown option 'dnssec-validation'
/etc/named.conf:14: expected 'trust-anchor' near ';'
```


どうやらDNSSECの書き方がダメだったらしく、CentOS6と5で違うみたい。自分のCentOSは5だった。。。

```
dnssec-enable yes;
dnssec-validation yes;
dnssec-lookaside auto;
```

の代わりに、

```
dnssec-lookaside . trust-anchor dlv.isc.org.; // above won't work on CentOS 5
```

と書くと良いみたいです。

##### 'bindkeys-file' option

```
unknown option 'bindkeys-file'
```

>* dnssec-lookaside

>bindkeys-fileで定義された/etc/named.iscdlv.keyのキーを使用して、DNSSECルックアサイド検証(DLV)を有効にするかどうかを示します。
https://docs.oracle.com/cd/E39368_01/e48214/ch13s03.html

わからなかったのでここの設定は削除して対応した...。



##### /etc/named.rfc1912がない

```
/etc/named.conf:49: open: /etc/named.rfc1912.zones: file not found
```

的なエラー。

[ここ](https://nextstageone.g.hatena.ne.jp/satakesatake/20100923/1285229023)にたどり着いた。	

>bind-chrootをインストールすると、/var/named/chroot/etc以下にnamed.rfc1912.zonesが配置されている。
bind-chrootをインストールしない場合、/usr/share/doc/bind*/sample以下のサンプルファイルをコピーし、雛形とする。
特に編集は必要ない。
named.confには、named.rfc1912.zonesを読み込む設定が記載されている。
zone設定は直接named.confには記載しない。
via: https://nextstageone.g.hatena.ne.jp/satakesatake/20100923/1285229023


ということでこちらをコピペしてくる。

```
# cat /usr/share/doc/bind-9.3.6/sample/etc/named.rfc1912.zones > /etc/named.rfc1912.zones
```

もうひとつファイルを`named.conf`でincludeしているので、そちらのサンプルファイルもコピーする。



### BINDの起動















# Note

## 名前解決の方法

### 最初のDNSのつなぎ先

DHCPサーバーなら、参照すべきDNSサーバーの情報ももらえる。

`/etc/nsswitch.conf`の、`hosts:`

files, dnsの順番に見る。filesっていうのは`/ets/hosts`のことなので、先にこちらを参照する。

```
#hosts:     db files nisplus nis dns
hosts:      files dns
```

コメントアウトしてるやつらも。いろいろ重要なものがある。

`/etc/resolv.conf`に、最初に問い合わせる内容が書いてある。

例えば。

```
# Automatically generated by GMO Cloud Public
domain xxx.com
nameserver xx.xx.xx.xx.xx
nameserver 180.222.191.16
```


### 問い合わせ先

さっきみたローカルにない場合、ルートDNSサーバー(世界に13個！)に問い合わせる。

google.co.jpだったら、

ルート < jpの場所がここだよー
jpのやつ < coの場所はここだよー
coのやつ < googleの場所はここだよー

IPアドレスがわかる！

### Protocol

UDPというプロトコル。

昔はDNSサーバーは処理が大変だったので、UDPはハンドシェイクとかしなかった。

=> 中間攻撃ができる！ドメイン乗っ取り。

