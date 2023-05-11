# 開発環境構築手順

# バージョン

* Rocky Linux
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
# dnfパッケージを最新にする
$ sudo dnf update -y

# vimをインストール
$ sudo dnf install -y vim
```

## 【任意】よく使用するコマンドをaliasに登録

```
vim ~/.bash_profile

alias ll='ls -l'
alias la='ls -a'

HISTSIZE=10000
HISTFILESIZE=10000
HISTTIMEFORMAT='%Y-%m-%d %H:%M:%S '


$ source ~/.bash_profile
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

# デバイスを確認
$ nmcli device
ens160  ethernet  connected  ens160
lo      loopback  unmanaged  --

# XX.XX.XX.XX には上記で調べたIPアドレスを入力する
$ sudo nmcli connection modify ens160 ipv4.addresses XX.XX.XX.XX/24

# ゲートウェイ設定
$ sudo nmcli connection modify ens160 ipv4.gateway XX.XX.XX.1

# IP固定の設定
$ sudo nmcli connection modify ens160 ipv4.method manual

# 設定を反映する
$ sudo nmcli connection down ens160; nmcli connection up ens160

# 設定の確認
$ sudo nmcli device show ens160
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
# rbenv をインストール
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
exec $SHELL -l

rbenv -v

# ruby-build をインストール
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
ls ~/.rbenv/plugins/ruby-build/bin
<出力結果>
rbenv-install  rbenv-uninstall  ruby-build

# 開発に必要なパッケージをインストール
sudo dnf install -y gcc openssl-devel readline-devel zlib-devel

# Rubyをインストール
rbenv install -l
rbenv install 3.X.X

rbenv global 3.X.X

ruby -v
```

## Docker, Docker-Compose をインストール

```
# リポジトリの追加
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf repolist
> docker-ce-stableが表示されていればOK
docker-ce-stable        Docker CE Stable - aarch64

# Docker/Docker Composeのインストール
sudo dnf -y install device-mapper-persistent-data lvm2
sudo dnf install docker-ce docker-ce-cli containerd.io docker-compose-plugin

vi /etc/NetworkManager/NetworkManager.conf
[main]
#plugins=keyfile,ifcfg-rh
dns=none # <-追加

sudo systemctl restart NetworkManager

docker --version

docker compose version

# Dockerの起動
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker

# sudoを使わずにDockerを起動する設定
sudo groupadd docker
whoami

sudo usermod -aG docker <current_user>

groups <current_user>

# docker コマンドを叩いて、permission errorが出なければ反映済み
docker ps
```

## その他必要なものをインストール
```
# nodejs をインストール
sudo dnf upgrade --refresh
sudo dnf install curl -y
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo -E bash -

sudo dnf install nodejs -y
node -v
npm -v

# yarn をインストール
npm install --global yarn 

yarn -v

# bundlerのインストール
gem install bundler

# Graphvizのインストール
$ sudo dnf install -y graphviz
```

## docker MySQL構築

```
# リポジトリをクローン
git clone https://github.com/katz0726/development_env.git ~/

# docker起動
docker compose up -d

# docker起動確認
docker compose ps

# MySQLへの接続確認
$ docker container ls
$ docker exec -it [container ID] bash
$ mysql -u railsdev -p

※接続情報はdocker-compose.ymlを参照
```

