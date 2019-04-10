# Rails environment of Docker on Vagrant

Mac/Windows両対応のVagrantとDockerを使用したRails開発環境のテンプレートです。  
共有ディレクトリにNFSを使用し、ソースコードの即時反映を実現しながらDocker for Mac/Windowsと比較して高速に動作します。  
メンテコストを極力下げるため、必要最低限の構成にしてあります。必要に応じてカスタマイズして使ってください。

## Requirement

事前に以下のツールのインストールを行う必要があります。

- [VirtualBox](https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html)
- [Vagrant](https://www.vagrantup.com/downloads.html)

{Macの場合}上記2つは[Homebrew Cask](http://caskroom.io/)でのインストールを推奨（公式サイトに最新版が掲載されるのにラグがあるため）

```
YourPC$ brew cask install virtualbox
YourPC$ brew cask install vagrant
```

- vagrant-bindfsプラグインをインストール

```
YourPC$ vagrant plugin install vagrant-bindfs
```

- vagrant-hostsupdaterプラグインをインストール

```
YourPC$ vagrant plugin install vagrant-hostsupdater
```

{Macの場合}共有ディレクトリのマウントにnfsを使用するためnfsdを有効にする必要があります。

```
YourPC$ sudo touch /etc/exports
YourPC$ sudo nfsd enable
```

{Windowsの場合}vagrant-winnfsdプラグインをインストール

```
YourPC$ vagrant plugin install vagrant-winnfsd
```

## ネットワーク設定の確認

VirtualBoxマネージャーを起動し、「ファイル」-「ホストネットワークマネージャー」を開く。  
Virtual Host-Only Ethernet Adapter #{任意の数字}をダブルクリックし、

- 「アダプターを手動で設定」を選択
- IPv4アドレスを「192.168.33.1」
- IPv4ネットマスクを「255.255.255.0」

に変更する。

## Getting Started

**既存のRailsプロジェクトで使用する場合**

既存のRailsプロジェクトで使用する場合は、あなたのプロジェクトをmyappディレクトリ（任意の名前に変更可）内にコピーしてください。

/  
├ myapp/  
│  ├ docker-compose.yml  
│  ├ Dockerfile  
│  └ {your rails project files}  
└ Vagrantfile

database.ymlの以下の項目を当環境に合わせて編集

```
#database.yml
development:
  host: db
  username: postgres #postgresqlの場合
    .
    .
    .
```

Rubyのバージョンを設定する

```
#Dockerfile
FROM ruby:2.5.3  #バージョン番号を変更
ENV LANG C.UTF-8
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /app
WORKDIR /app

```

使用するRDBMSを設定する

```
#docker-compose.yml
version: '3'
services:
  db:
    image: postgres:10.4  #例：PostgreSQLの場合
    volumes:
      - /data/db:/var/lib/postgresql/data  #例：PostgreSQLの場合
    ports:
      - "5432:5432" #例：PostgreSQLの場合
```

ブラウザでアクセスする際のホスト名を設定

```
#Vagrantfile
  config.vm.hostname = "myapp.localhost" #任意のホスト名に変更
```

Vagrant VMの起動（初回はVMの作成を行います）  
※{Windowsの場合}コマンドプロンプトまたはPowerShellを「管理者として実行」する

```
YourPC$ vagrant up
```

Vagrant VMにログイン

```
YourPC$ vagrant ssh
```

以下のディレクトリにソースがマウントされているので移動

```
Vagrant$ cd /vagrant/myapp
```

Vagrant VM上でDockerコンテナに対してbundle install実行

```
Vagrant$ sudo docker-compose run app bundle install --path vendor/bundle
```

DB構築

```
Vagrant$ sudo docker-compose run app bundle exec rake db:create
Vagrant$ sudo docker-compose run app bundle exec rake db:migrate
```

初期設定に使用したコンテナの削除

```
Vagrant$ sudo docker-compose down
```

Dockerコンテナの実行  
※実行してから実際にアクセス可能になるまで時間がかかるので気長にお待ち下さい

```
Vagrant$ sudo docker-compose up
```

ブラウザで次のURLにアクセス  
http://myapp.localhost (Vagrantfileのconfig.vm.hostnameで設定したホスト名)

**新規Railsプロジェクトで使用する場合**

※検証中…

## 検証に使用した環境

- MacBook Air Mid2012 macOS High Sierra
- BTO PC Windows10 Pro

## Authors

[Ryoji Hayasaka](https://github.com/hayakasa)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
