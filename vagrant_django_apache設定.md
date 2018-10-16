# vagrant設定
## 初期設定
~~~bash
> vagrant box add (ボックス名をつける) (使用したいOSのボックス）
# vagrantを作成したいディレクトリまで移動
# (ここではユーザー直下にtestディレクトリとする)

test> vagrant init (ボックス名)
test> vagrant up
# ここでエラーが出るため、SSHを行い、修正する
test> vagrant ssh
# これでvagrantというユーザー名でvmにsshでアクセス可能になる
$ su -
# yum -y install ftp://ftp.riken.jp/Linux/cern/centos/7/updates/x86_64/Packages/kernel-devel-3.10.0-514.26.2.el7.x86_64.rpm
# exit
$ exit
test> vagrant reload
test> vagrant ssh
# これで使用可能になる
~~~

## 基本コマンド
### 立ち上げ
~~~
vagrant up
~~~
### SSH接続
~~~
vagrant ssh
~~~
### シャットダウン
~~~
vagrant halt
~~~
### 初期化(削除のこと）
~~~
vagrant destroy
~~~

# CeotOs
## vagrant初期化後の設定
### Python3.6、Django2.0を入れる
~~~
[]$ sudo yum install -y https://centos7.iuscommunity.org/ius-release.rpm
[]$ sudo yum install -y python36u python36u-pip python36u-devel
[]$ python3.6 -m venv (作成したいvenvの名前)
[]$ source (venv名)/bin/activate
(venv名)[]$ pip install django
(venv名)[]$ deactivate
~~~

### Apacheの導入
~~~
[]$ su -
[]# yum -y install httpd
[]# systemctl start httpd
[]# systemctl enable httpd
(確認コマンド)
[]# systemctl is-enabled httpd
[]# exit
~~~

### git、wgetの設定
~~~
(ライブラリをインストールしないとgitを入れられない)
[]$ sudo yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-ExtUtils-MakeMaker autoconf
(wgetを入れないとこのあとの処理ができない）
[]$ sudo yum -y install wget
[]$ wget https://www.kernel.org/pub/software/scm/git/git-2.18.0.tar.gz
[]$ tar vfx git-2.18.0.tar.gz
[]$ cd git-2.18.0
[git-2.18.0]$ make configure
[git-2.18.0]$ ./configure --prefix=/usr
[git-2.18.0]$ make all
[git-2.18.0]$ sudo make install
(バージョンの確認）
[git-2.18.0]$ git --version
[]$ cd
~~~
# デプロイ
## 必要なライブラリをインストール
~~~
[]$ sudo yum -y install httpd-devel python36u-mod_wsgi
~~~

## デプロイするDjango側の設定
### (Appの名前)/setting.py
~~~
# IPをすべて許可する
ALLOWED_HOSTS = ['*']
~~~

## デプロイするCentOS側の設定
### githubからcloneを入手する
~~~
保存したいディレクトリ名を作る
[]$ mkdir (ディレクトリ名）
[]$ cd （ディレクトリ名）
[ディレクトリ名]$ git clone https://github.com/(githubのアカウント名)/(取るディレクトリ名).git
[ディレクトリ名]$ cd
~~~

### githubからpullする
~~~
[ディレクトリ名]$ git pull https://github.com/(githubのアカウント名)/(取るディレクトリ名).git
[ディレクトリ名]$ cd
[]$ su -
[]# systemctl restart httpd
~~~

### migrateし、DBを有効化する
~~~
[]$ source (venv名)/bin/activate
(venv名)[]$ cd (ディレクトリ名)/(githubからのディレクトリ名)
(一応、lsしてmanage.pyがあるか確認する)
(venv名)[githubからのディレクトリ名]$ ls
(venv名)[githubからのディレクトリ名]$ python3.6 manage.py migrate
(ここでエラーが出た場合は「DBへのアクセス権限」を行うこと)
(venv名)[githubからのディレクトリ名]$ python3.6 manage.py migrate
(OKが出たら終了)
~~~

### Apacheの設定変更
- /etc/httpd/conf.d/django.conf
~~~
WSGIScriptAlias / /home/(vagrantなら：vagrant)/(ディレクトリ名)/(gitから取得したディレクトリ名))/(project名)/wsgi.py
WSGIPythonPath /home/(vagrantなら：vagrant)/(ディレクトリ名)/(gitから取得したディレクトリ名):/home/(vagrantなら：vagrant)/(作成したvenv名)/lib/python3.6/site-packages
<Directory /home/(vagrantなら：vagrant)/(ディレクトリ名)/(gitから取得したディレクトリ名))/(project名)>
<Files wsgi.py>
Require all granted
</Files>
</Directory>
~~~
- Aliasの最初の「/」はルートURLの指定である
- この設定の場合、http://(IP Address)/がルートURL
- 変える場合「/(なんでも可)/」とすれば、http://(なんでも可)/とできる

### アプリの位置を教える
・/etc/httpd/conf.d/userdir.conf
~~~
<IfModule mod_userdir.c>
    #
    # UserDir is disabled by default since it can confirm the presence
    # of a username on the system (depending on home directory
    # permissions).
    # UserDir disabled
    UserDir enable

    #
    # To enable requests to /~user/ to serve the user's public_html
    # directory, remove the "UserDir disabled" line above, and uncomment
    # the following line instead:
    #
    #UserDir public_html
</IfModule>
~~~

### ユーザーディレクトリの権限変更
・デフォルトならばユーザー名はvagrant
~~~
[]$ cd /home
(権限を確認する)
[home]$ ls -la
(vagrantが755でなければ以下を実行)
[home]$ sudo chmod 755 (vagrantならば：vagrant)
~~~

### httpd(Apache)をリスタートする
~~~
[]$ sudo systemctl restart httpd
~~~

### DBへのアクセス権限
・Web参照のものが別の方法が良いかもと言っているので場合分けする
#### やり方1
~~~
[]$ cd （ディレクトリ名)
[ディレクトリ名]$ sudo chmod 757 (gitから取得したディレクトリ名)
[ディレクトリ名]$ cd (gitから取得したディレクトリ名)
[gitから取得したディレクトリ名]$ sudo chmod 756 db.sqlite3
[gitから取得したディレクトリ名]$ cd
[]$ sudo systemctl restart httpd
~~~
### staticファイルを反映させる(ある場合は)
- settings.py(場所はもう察して）
~~~
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
DEBUG = False
~~~

- /etc/httpd/conf.d/django.conf
~~~
Alias /static/ /home/(vagrant)/(ディレクトリ名)/(githubからのディレクトリ名)/static/

<Directory /home/(vagrant)/(ディレクトリ名)/(githubからのディレクトリ名)/static>
Require all granted
</Directory>
~~~

- CeotOS側
~~~
(venv名)[(githubからのディレクトリ名)]$ ./manage.py collectstatic
(venv名)[(githubからのディレクトリ名)]$ sudo systemctl restart httpd
~~~

# 参照先
- [gitの設定](https://qiita.com/Qiita/items/c686397e4a0f4f11683d)
- [デプロイまで](https://qiita.com/slt666666/items/8b5ac6f30a8310391cda)