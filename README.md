# Rails environment of Docker on Vagrant for Mac

Mac上で動作するVagrantとDockerを使用したRails開発環境のテンプレートです。  
Docker for Macと比較して、実行速度の低下を防ぎながらソースコードの即時反映を実現しています。  
メンテコストを極力下げるため、必要最低限の構成にしてあります。必要に応じてカスタマイズして使ってください。

## Getting Started

**既存のRailsプロジェクトで使用する場合**

既存のRailsプロジェクトで使用する場合は、下記3ファイルをプロジェクトのドキュメントルートにコピーしてください。
- Vagrantfile
- Dockerfile
- docker-compose.yml

Rubyのバージョンを設定する

```
#Dockerfile
FROM ruby:2.3.7  #バージョン番号を変更
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
#    image: mysql:5.6  #例：MySQLの場合
    volumes:
      - /data/db:/var/lib/postgresql/data  #例：PostgreSQLの場合
#      - /data/db:/var/lib/mysql  #例：MySQLの場合
```

ブラウザでアクセスする際のホスト名を設定

```
#Vagrantfile
  config.vm.hostname = "your.host.name" #任意のホスト名に変更
```

※以下のコマンドは、プロジェクトのドキュメントルートで実行してください

Vagrant VMの起動（初回はVMの作成を行います）

```
YourMac$ vagrant up
```

Vagrant VMにログイン

```
YourMac$ vagrant ssh
```

以下のフォルダにソースがマウントされているので移動

```
Vagrant$ cd /vagrant
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

Dockerコンテナの実行

```
Vagrant$ sudo docker-compose up
```

**新規Railsプロジェクトで使用する場合**

※検証中…

## Requirement

事前に以下のツールのインストールを行う必要があります。

- [VirtualBox](https://www.oracle.com/technetwork/server-storage/virtualbox/downloads/index.html)
- [Vagrant](https://www.vagrantup.com/downloads.html)

上記2つは[Homebrew Cask](http://caskroom.io/)でのインストールを推奨

```
YourMac$ brew cask install virtualbox
YourMac$ brew cask install vagrant
```

- vagrant-bindfsプラグイン

```
YourMac$ vagrant plugin install vagrant-bindfs
```

- vagrant-hostsupdaterプラグイン

```
YourMac$ vagrant plugin install vagrant-hostsupdater
```

共有フォルダのマウントにnfsを使用するため、Macのnfsdを有効にする必要があります。

```
YourMac$ sudo touch /etc/exports
YourMac$ sudo nfsd enable
```

## Authors

[Ryoji Hayasaka](https://github.com/hayakasa)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
