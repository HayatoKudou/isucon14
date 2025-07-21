# ISUCON14 Nodejs

ref: https://github.com/matsuu/aws-isucon/tree/main/isucon14

### サーバーセットアップ
- 配布されているAMIからEC2インスタンスを立ち上げる
- セキュリティグループにHTTPS 443のインバウンドルール追加
- TODO: 言語別のサービス起動
- hostsファイルに `${サーバのIPアドレス} isuride.xiv.isucon.net` 登録

### githubセットアップ
- TODO: sshキーの設定

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

### ローカルからDB接続

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

## ベンチマーク計測方法

```
sudo -i -u isucon
./bench run . run --addr 127.0.0.1:443 --target https://isuride.xiv.isucon.net --payment-url http://127.0.0.1:12346 --payment-bind-port 12346
```

### DBクライアント確認

```
sudo systemctl list-units | grep -E "(mysql|mariadb|postgres|postgresql)"
```