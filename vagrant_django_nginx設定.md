# vagrant設定
## 初期設定
~~~bash
> vagrant box add (ボックス名をつける) (使用したいOSのボックス）
vagrantを作成したいディレクトリまで移動
(ここではユーザー直下にtestディレクトリとする)

test> vagrant init bento/centos-7.3
test> vagrant up
＊ここでエラーが出るため、SSHを行い、修正する
test> vagrant ssh
これでvagrantというユーザー名でvmにsshでアクセス可能になる
$ su -
# yum -y install ftp://ftp.riken.jp/Linux/cern/centos/7/updates/x86_64/Packages/kernel-devel-3.10.0-514.26.2.el7.x86_64.rpm
# exit
$ exit
test> vagrant reload
test> vagrant ssh
これで使用可能になる
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

### Nginxの導入
~~~
[]$ sudo vi /etc/yum.repos.d/ngnix.repo
~~~
- ngnix.repo
~~~
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/$basearch/
gpgcheck=0
enabled=1
~~~
~~~
[]$ su -
[]# yum -y install nginx
[]# systemctl enable nginx
[]# systemctl start nginx
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