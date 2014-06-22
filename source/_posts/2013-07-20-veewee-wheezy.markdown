---
layout: post
title: "Vagrant Wheezy BoxをVeeWeeで作る"
date: 2013-07-20 20:54
comments: true
categories: 
---

## 構築した環境
* Mac OS X 10.8.4
* XCode 4.6.3
  * 事前にCommand Line Toolsはインストールしておく
* rbenv
* Vagrant 1.2.2
* Debian Wheezy

Vagrnat 1.0以降はパッケージで提供されています。gemからVagrantを入れる必要ありませんので、最新の場合はインストール部分を省略してください。

[Vagrantダウンロード一覧]


# VirtualBoxをインストール

http://download.virtualbox.org/virtualbox/4.2.12/VirtualBox-4.2.12-84980-OSX.dmg


# rbenv

```
git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build

echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

exec $SHELL -l

rbenv install 1.9.3-p429
rbenv global  1.9.3-p429
rbenv rehash

gem install bundler

exec $SHELL -l
```


# Vagrantのインストール

http://www.vagrantup.com/

http://downloads.vagrantup.com/tags/v1.2.2

Vagrant-1.2.2.dmg をダウンロードしてインストール


## VeeWeeのインストール

VeeWeeはBoxファイルのテンプレートを生成してくれるパッケージです。

### Bundlerが入っていなければ入れる
```
gem install bundler
```


### インストール

```
git clone git://github.com/jedi4ever/veewee.git
cd veewee
bundle install
```


## 仮想マシンの定義

Debianの環境を作成する。

このコマンドはveeweeディレクトリ内で実行する。

```
bundle exec veewee vbox define "wheezy" "Debian-7.1.0-amd64-netboot"
```

そうするとdefinationsにテンプレートが作成されるので、それらを編集する。

### definition.rb 編集

vim definitions/wheezy/definition.rb

ミラーの参照先を変更

```
:iso_src => "http://ftp.riken.jp/Linux/debian/debian-cd/7.1.0/amd64/iso-cd/debian-7.1.0-amd64-netinst.iso",
```

## 仮想マシンのビルド

```
bundle exec veewee vbox build wheezy
```

しばらく待つ。途中でisoをダウンロードするか聞いてくるのでyesとする。

自動的に仮想マシンが起動するので、またしばらく放置。

プロンプトに

```
The box wheezy was built successfully!

ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -p 7222 -l veewee 127.0.0.1
```

と表示されていれば、このコマンドで vagrant というパスワードでsshログインが可能。

この状態になったら一度落とします。

## boxファイルへエクスポート

構築した仮想環境をboxファイルにエクスポートします。

```
bundle exec veewee vbox export wheezy
```


## 仮想環境の登録と初期化



veeweeディレクトリから抜けて、仮想マシン用のディレクトリを作成します。

```
mkdir ~/box/wheezy
```

さて、いよいよ。登録して起動です。

### エクスポートしたboxを登録します。

```
vagrant box add 'base_wheezy' 'wheezy.box'
```

```
vagrant init wheezy

# カレントディレクトリにVagrantfileが作成される

vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
[default] Importing base box 'wheezy'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Creating shared folders metadata...
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2222 (adapter 1)
[default] Booting VM...
[default] Waiting for VM to boot. This can take a few minutes.
[default] VM booted and ready for use!
[default] Configuring and enabling network interfaces...
[default] Mounting shared folders...
[default] -- /vagrant
```

起動が完了しました！

## ログインしてみる

```
vagrant ssh

[vagrant@localhost ~]$
```

プロンプトがでましたね。
仮想マシンを起動したカレントディレクトリが、/vagrant として仮想マシン内から参照可能です。

## 仮想マシンの停止

### 作業情報を保持したまま停止

```
vagrant halt
```

### 作業情報をすべて破棄

この方法は仮想マシンのファイルはすべて破棄されます。

```
vagrant destroy
```

一度boxファイルができてしまえば vagrant コマンドだけで、すぐに構築した仮想マシンをすぐに各環境で立ち上げることができます。

複数の環境を自動的にテストする目的には特に有用かと思います。


## Appendix

公開されている box 一覧
http://www.vagrantbox.es/


今回作ったboxファイル

 https://www.dropbox.com/s/ps447hpvfmdjwoy/wheezy.box

[Vagrantダウンロード一覧]:http://downloads.vagrantup.com/

# Chef ＋ Vagrant入門（ハンズオン）やります!!

[ATNDはこちら](http://atnd.org/events/40871/)
