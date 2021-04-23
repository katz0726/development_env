# 開発環境構築手順

# バージョン

* CentOS
* Docker(1.13.1)
* docker-compose(1.16.1)
* Ruby(2.5.1)
* Rails(5.2.0)

# Linuxの設定

ホスト側の設定については、以下を参照。

TODO 設定手順を追記

* https://qiita.com/trkrcafeaulate/items/3dfa8d5ec1075848f1e9
* https://kitigai.hatenablog.com/entry/2018/06/28/233000

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
IPADDR=10.0.XX.YY
NETMASK=255.255.255.0
GATEWAY=10.0.XX.1
DNS1=8.8.8.8
DNS2=8.8.4.4

# 設定を反映する
$ sudo systemctl daemon-reload
$ sudo systemctl restart network
```

## firewallの無効化

```
$ sudo systemctl stop firewalld
$ sudo systectl disable firewalld
$ sudo systectl status firewalld
```

## 必要なライブラリをインストール

```
# yumパッケージを最新にする
$ sudo yum update -y

# vimをインストール
$ sudo yum install -y vim

# git2をインストール
$ sudo yum remove git*
$ sudo yum -y install https://packages.endpoint.com/rhel/7/os/x86_64/endpoint-repo-1.7-1.x86_64.rpm
$ sudo yum install git

# git2.xがインストールされていることを確認
$ git -- version
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
                    ImageMagick-devel

# Dockerをインストール
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce

# Docker を sudo なしで実行できるように設定
sudo usermod -aG docker $USER
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "bip":            "172.30.0.1/16",
  "live-restore":   true
}
EOF
sudo systemctl enable docker
sudo systemctl start docker

# docker composeをインストール
sudo curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# nodejs をインストール
curl --silent --location https://rpm.nodesource.com/setup_12.x | sudo bash -
sudo yum install -y nodejs
curl --silent --location https://dl.yarnpkg.com/rpm/yarn.repo | sudo tee /etc/yum.repos.d/yarn.repo
sudo yum install -y yarn

# python3 をインストール
sudo yum install -y python3
sudo pip3 install --upgrade pip

# rbenvをインストール
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

echo '# rbenv' >> ~/.bashrc
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

source ~/.bashrc

# Rubyインストール
mkdir -p ~/src
pushd ~/src
  rm -Rf ruby-2.7.2*
  wget https://cache.ruby-lang.org/pub/ruby/2.7/ruby-2.7.2.tar.gz
  tar zxf ruby-2.7.2.tar.gz
  cd ruby-2.7.2
  ./configure --disable-install-rdoc
  make
  sudo make install
popd
sudo gem update --system --no-document
sudo gem update -f rdoc --no-document
sudo gem update -f bundler --no-document

# railsのインストール
sudo gem install rails
```