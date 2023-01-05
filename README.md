# 開発環境構築手順

# バージョン

* CentOS
* Docker(1.13.1)
* docker-compose(1.16.1)
* Ruby(2.5.1)
* Rails(5.2.0)

# VMwareの設定

VMwareの仮想マシンの編集を開き、ネットワークアダプタの設定で以下を入力し、適用する

* 「物理ネットワーク接続の状態を複製」にチェックを入れる
* 「ブリッジ」を選択する。
* アダプタの設定でブリッジ先が一つになっていること

# Linuxの設定

## 【任意】キーボードをUS配列にする

```
$ localectl set-keymap us
$ localectl status
```

## SELINUX無効化

SELINUX=enforcingをdisabledに変更する
```
$ sudo vi /etc/selinux/config
SELINUX=enforcing

$ reboot
```

## sudoをパスワードなしで実行する

XXXX部分にはsudoをパスワードなしにするユーザ名を入力する
```
$ sudo visudo

## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
XXXX    ALL=(ALL)       NOPASSWD: ALL
```

## 必要なライブラリをインストール

```
# yumパッケージを最新にする
$ sudo yum update -y

# vimをインストール
$ sudo yum install -y vim

# git2をインストール
$ sudo yum remove git*
$ sudo yum install \
    https://repo.ius.io/ius-release-el7.rpm \
    https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ sudo yum install pcre2
$ sudo yum install git --enablerepo=ius --disablerepo=base,epel,extras,updates

# git2.xがインストールされていることを確認
$ git --version
```

## 【任意】よく使用するコマンドをaliasに登録

```
vim ~/.bashrc

alias ll='ls -l'
alias la='ls -a'

HISTSIZE=10000
HISTFILESIZE=10000
HISTTIMEFORMAT='%Y-%m-%d %H:%M:%S '


$ source ~/.bashrc
```

## 【任意】vimの設定

```
$ touch ~/.vimrc
$ sudo vim ~/.vimrc
```

※設定は以下のリンクを参照

https://github.com/katz0726/vim-settings

## IPアドレスを固定

```
# 現在のIPアドレスを確認
$ ip addr

# ifcfg-xxxのxxxには、ip addrで調べたデバイス名を入力する
$ sudo vim /etc/sysconfig/network-scripts/ifcfg-ens33

# 以下を追記
BOOTPROTO="none"
IPV6INIT="no"
ONBOOT="yes"
IPADDR=XXX.XXX.XXX.YYY
NETMASK=255.255.255.0
GATEWAY=XXX.XXX.XXX.2
DNS1=8.8.8.8
DNS2=8.8.4.4

# 設定を反映する
$ sudo systemctl daemon-reload
$ sudo systemctl restart network
```

## firewallの無効化

```
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
$ sudo systemctl status firewalld
```

## Rubyをインストール

以下をコピーして実行する

```
#!/bin/bash

#開発に必要なパッケージをインストール
sudo yum -y install epel-release
sudo yum groupinstall -y "Development Tools" 
sudo yum -y install bzip2 \
                    gcc \
                    gcc-c++ \
                    openssl-devel \
                    readline-devel \
                    zlib-devel \
                    sqlite-devel \
                    unzip \
                    zip \
                    zlib-devel \
                    wget \
                    ImageMagick-devel \
                    mysql-devel

# Dockerをインストール
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce

# Docker を sudo なしで実行できるように設定
sudo gpasswd -a $(whoami) docker
sudo chgrp docker /var/run/docker.sock

sudo systemctl enable docker
sudo systemctl start docker

# docker composeをインストール
sudo curl -L https://github.com/docker/compose/releases/download/1.28.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# nodejs をインストール
sudo yum install nodejs npm
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo yum install -y yarn

# rbenvをインストール
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

echo '# rbenv' >> ~/.bashrc
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

source ~/.bashrc

# Rubyインストール
rbenv install 2.7.3

rbenv global 2.7.3
rbenv version

# bundlerのインストール
gem install bundler
```

## docker MySQL構築

```
# リポジトリをクローン
git clone https://github.com/katz0726/development_env.git ~/

# docker起動
docker-compose up -d

# docker起動確認
docker-compose ps

# MySQLへの接続確認
$ docker container ls
$ docker exec -it [container ID] bash
$ mysql -u root -p

※接続情報はdocker-compose.ymlを参照
```

