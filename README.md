# ISUCON14 Nodejs

ref: https://github.com/matsuu/aws-isucon/tree/main/isucon14


### サーバーセットアップ
- 配布されているAMIからEC2インスタンスを立ち上げる
- セキュリティグループにHTTPS 443のインバウンドルール追加
- TODO: 言語別のサービス起動
- hostsファイルに `${サーバのIPアドレス} isuride.xiv.isucon.net` 登録

### ISUCONで使いそうなファイルの場所

```
/home/isucon/webapp # アプリケーション
/etc/nginx/nginx.conf # Nginxの設定ファイル
/etc/mysql/mysql.conf.d/mysqld.cnf # MySQLの設定ファイル
/etc/systemd/system/isuride-node.service # アプリケーションのsystemdサービスファイル
```

### githubセットアップ
TODO: sshキーの設定

```bash
cd /home/isucon/webapp

git config --global user.email "@gmail.com"
git config --global user.name ""

git init
git add nodejs/ sql/
git commit -m "Initial commit"
git branch -M main
git remote add origin git@github.com:HayatoKudou/isucon**.git
git push -u origin main
```

### DBサーバの稼働確認

```bash
sudo systemctl list-units | grep -E "(mysql|mariadb|postgres|postgresql)"
```

### ローカルからDB接続
- init.sh の接続情報を参照

```
HOST: 127.0.0.1
PORT: 3306
USER: isucon
PASS: isucon
DATABASE: isuride
SSH: SERVER: EC2のパプリックIP
PORT: 22 # インバウンドルールに追加要
USER: ubuntu
```

### スロークエリログの有効化(MySQL)

- /etc/mysql/mysql.conf.d/mysqld.cnf のコメントアウトを外す
- `sudo systemctl restart mysql` で再起動
- `long_query_time` を0にすることで、全てのクエリをスロークエリとして記録することができる
- `show variables like 'slow%'` で設定の確認

```
# slow_query_log = 1
# slow_query_log_file = /var/log/mysql/mysql-slow.log
# long_query_time = 2
# log-queries-not-using-indexes
```

### スロークエリ解析

pt-query-digest インストール
ref: https://github.com/matsuu/docker-pt-query-digest

```bash
mkdir ~/tools
cd ~/tools
wget https://github.com/percona/percona-toolkit/archive/refs/tags/v3.6.0.tar.gz
tar zxvf v3.6.0.tar.gz
./percona-toolkit-3.6.0/bin/pt-query-digest --version
sudo install ./percona-toolkit-3.6.0/bin/pt-query-digest /usr/local/bin
pt-query-digest --version
```

ベンチマーク計測

```shell
sudo -i -u isucon
./bench run . run --addr 127.0.0.1:443 --target https://isuride.xiv.isucon.net --payment-url http://127.0.0.1:12346 --payment-bind-port 12346
```

pt-query-digest で解析
ref: https://isucon-workshop.trap.show/text/chapter-3/1-SlowQueryLog.html#%E3%82%B9%E3%83%AD%E3%83%BC%E3%82%AF%E3%82%A8%E3%83%AA%E3%83%AD%E3%82%AF%E3%82%99%E3%81%AE%E8%A7%A3%E6%9E%90%E7%B5%90%E6%9E%9C%E3%81%AE%E3%81%BE%E3%81%A8%E3%82%81%E3%82%92%E8%A6%8B%E3%82%8B

```shell
sudo pt-query-digest /var/log/mysql/mysql-slow.log
```

ログテーション設定 (任意)

```shell
mkdir ~/log 
sudo pt-query-digest /var/log/mysql/mysql-slow.log > ~/log/$(date +mysql-slow.log-%m-%d-%H-%M -d "+9 hours")
sudo rm /var/log/mysql/mysql-slow.log
sudo systemctl restart mysql # MySQLのログの出力先を消したため、再起動してログファイルを再生成
```

### DB適用(初期化スクリプト実行)

```shell
cd /home/isucon/webapp/sql
./init.sh
```