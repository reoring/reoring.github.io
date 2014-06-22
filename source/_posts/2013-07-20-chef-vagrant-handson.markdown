---
layout: post
title: "Chef Vagrant Handson"
date: 2013-07-20 22:20
comments: true
categories: 
---

# 前提条件

 * XCode
   * Command Line Tools


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


# Sahara plugin

```
git clone https://github.com/ryuzee/sahara.git

cd sahara

bundle install
bundle exec rake build

vagrant plugin install pkg/sahara-0.0.14.gem
```

# Chefのインストール

```
curl -L http://www.opscode.com/chef/install.sh | sudo bash

% knife --version
Chef: 11.4.4
```


# knife solo

```
gem i knife-solo --no-ri --no-rdoc
rbenv rehash
```


## VagrantにBoxを追加

```
vagrant box add centos_63_x86_64 https://dl.dropbox.com/s/ajk7omgla8i4qi6/centos_63_x86_64_for_vb4.1.box
```


# Vagrant_Momoxoを持ってくる

```
git clone git://github.com/reoring/Momoxo_Vagrant.git
```

# vagrant-hostsupdater

```
vagrant plugin install vagrant-hostsupdater
```

# vagrant up

```
vagrant up --no-provision

vagrant sandbox on

vagrant ssh-config --host momoxo >> ~/.ssh/config

cd chef-repo

knife solo prepare momoxo
knife solo cook momoxo

失敗したら, vagrant sandbox rollback
```

# serverspecでテストするよん

```
gem install serverspec

rake spec
..............

Finished in 0.49342 seconds
14 examples, 0 failures
```

