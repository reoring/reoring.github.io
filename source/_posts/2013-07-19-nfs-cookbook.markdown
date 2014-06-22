---
layout: post
title: "NFS CookbookをWheezyに対応させる"
date: 2013-07-19 19:36
comments: true
categories: 
---

NFSのCookbookがGitHubで公開されているのだけど、Wheezyで実行するとportmapあたりでエラーになる。これはWheezyではportmapはrpcbindというサービスに変更されているから発生する。

https://github.com/atomic-penguin/cookbook-nfs

forkしたレポジトリ
https://github.com/reoring/cookbook-nfs


## Berksfileにcookbookの場所を追加
```ruby
site :opscode

metadata

cookbook 'nfs', git: 'git://github.com/reoring/cookbook-nfs.git'
```

## Cookbookを取得しなおす

```
rm Berksfile.lock
berks install
```

## 行った変更

attributes/default.rb に下記変更を追加。
nodeのplatform_versionでOSのバージョンが取得できるので、それを使って判断する。
```ruby
# Debian 7.0+
if node['platform'] == "debian" and node['platform_version'].to_i >= 7
    default['nfs']['service']['lock'] = "nfs-common"
    default['nfs']['service']['portmap'] = "rpcbind"
    default['nfs']['packages'] = %w{ nfs-common rpcbind }
end
```

本家にプルリクしたら既に同様のプルリクがあった…orz


## nodeってなに？

node変数には、Chef Soloを実行している環境の様々な変数が入っています。
これは Ohai というミドルウェアが頑張って収集していきているデータです。

その他にも、attriburesで設定した変数などもnodeに入ってきます。

platform_versionの他にも、下記の情報が入っています。

 * node['platform']
 * node['ipaddress']
 * node['macaddress']
 * node['fqdn']
 * node['hostname']
 * node['domain']
 * node['recipes']
 * node['roles']
 * node['ohai_time']

詳しくは、Opscodeのサイトに書いてあります。http://docs.opscode.com/ohai.html
