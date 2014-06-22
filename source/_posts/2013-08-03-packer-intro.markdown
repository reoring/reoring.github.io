---
layout: post
title: "Packerを使ってOSの基礎イメージを作る"
date: 2013-08-03 03:44
comments: true
categories: 
---

PackerはOSの基礎イメージを様々な環境向けに構築できるコマンドラインツール。
Vagrantを作成している、hashicorp社が開発しているみたい。

テンプレートというJSONファイルを書き、それを基にEC2のAMIイメージや、Digital Oceanに基礎イメージを構築してくれる、もちろんVagrantのBoxファイルも作成できる。

Vagrantの基礎イメージ構築にはVeeWeeなどを使っていたが、PackerであればVagrant以外のプロバイダにも同様に構築ができる。

PackerはChefやPuppetのような構成管理ソフトウェアを置き換えるようなものではなく、それらのが動作するための基礎イメージをシンプルな手順で構築できるものらしい。

Packerは各プラットフォーム向けにバイナリを提供しているので、ダウンロードして適当なディレクトリに配置する。PATHを通しておくと便利。

http://www.packer.io/downloads.html

早速AWSにイメージを作ってみる。

まずはJSONファイルを書く

## example.json

```json
{
  "builders": [{
    "type": "amazon-ebs",
    "access_key": "AWSのアクセスキー",
    "secret_key": "AWSのシークレットキー",
    "region": "us-east-1",
    "source_ami": "ami-de0d9eb7",
    "instance_type": "t1.micro",
    "ssh_username": "ubuntu",
    "ami_name": "packer-example {{.CreateTime}}"
  }]
}
```

つぎに、JSONが正しいかを確認する。

```
packer validate example.json
```

Template validated successfully.と表示されれば問題ない。


## 構築を開始

```
packer build example.json
```

正常に構築されると、構築されたAMIのIDを教えてくれる。
```
mazon-ebs output will be in this color.

==> amazon-ebs: Creating temporary keypair for this instance...
==> amazon-ebs: Creating temporary security group for this instance...
==> amazon-ebs: Authorizing SSH access on the temporary security group...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Waiting for instance (i-7c70b010) to become ready...
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!
==> amazon-ebs: Stopping the source instance...
==> amazon-ebs: Waiting for the instance to stop...
==> amazon-ebs: Creating the AMI: packer-example 1375470030
==> amazon-ebs: AMI: ami-c21357ab
==> amazon-ebs: Waiting for AMI to become ready...
==> amazon-ebs: Terminating the source AWS instance...
==> amazon-ebs: Deleting temporary security group...
==> amazon-ebs: Deleting temporary keypair...
Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:

us-east-1: ami-c21357ab
```


## VagrantのBoxを作成する

Packerには、post-processorという仕組みが用意されていて、構築後の処理が行えるようだ。
標準で提供されている、vagrant用のpost-processorを使用すると、構築されたAMIイメージを立ち上げるためのVagrant Boxファイルが作成できる。

```json
{
  "builders": [{
    "type": "amazon-ebs",
    "access_key": "AWSのアクセスキー",
    "secret_key": "AWSのシークレットキー",
    "region": "us-east-1",
    "source_ami": "ami-de0d9eb7",
    "instance_type": "t1.micro",
    "ssh_username": "ubuntu",
    "ami_name": "packer-example {{.CreateTime}}"
  }],

  "post-processors": ["vagrant"]
}
```

これを実行すると、boxファイルができがるので、vagrant box add で追加する。

その後、vagrant init で取り込んだBox名を指定して実行する。

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    config.vm.box = "packer_test"

    config.vm.provider :aws do |aws, override|
        aws.access_key_id = "AWSのアクセスキー"
        aws.secret_access_key = "AWSのシークレットキー"

        aws.keypair_name = "packer"

        override.ssh.username = "ubuntu"
        override.ssh.private_key_path = "~/packer_test/packer.pem"
    end
end
```

そして、vagrant up --provider=aws を実行すると、Packerで構築した環境が立ち上がる。

あとはいつものようにvagrant sshすれば、ログインできる。
