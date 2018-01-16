# Crowi Plus構築

## ホストOS要件

- Vagrant 2.0.1

## イメージ図

![image](https://github.com/kmd2kmd/ansible_playbooks/blob/master/workshop001-local/img/infrastructure.png?raw=true)

## 使い方

### インストール・起動
```bash
git clone https://github.com/kmd2kmd/ansible_playbooks
cd workshop001-local
vagrant up
```

### アンインストール
```bash
vagrant destroy -f
```


### 接続
```bash
vagrant ssh crowiplus
vagrant ssh mongodb
vagrant ssh redis
vagrant ssh elasticsearch
vagrant ssh controller
```

### PlayBook再実行

#### 方法1
```bash
vagrant up controller --provision
```

#### 方法2
```bash
vagrant ssh
cd /vagrant/
ansible-playbook -i playbook/hosts/local playbook/site.yml
```


## 説明

- Vagrant の Provisioner [ansible_local ](https://www.vagrantup.com/docs/provisioning/ansible_local.html)を利用して環境を構築する
- 各VMにAnsibleはインストールせず、VM:controllerに集約する

1. VagrantでVMを5つ作成(1~5)
2. controllerのvagrant provisionが実行(5)
  1. 各VMの秘密鍵コピー
  2. 秘密鍵のパーミッション、オーナー設定
  3. ansible_playbook を実行(6~9)

## 苦労したこと

- controllerからansible_localを実行する時に、秘密鍵のパーミッション777で警告がでる。
  - vagrantのマウント機能でパーミッションを600に制限すると何故かvagrantユーザでアクセスできなくなった...
    ```ruby
      config.vm.synced_folder ".", "/vagrant", mount_options: ['dmode=600','fmode=600']
    ```
  - 事前にprovision.file,shellで秘密鍵をコピーしパーミッションを変更することで対応した

    ```ruby
    controller.vm.provision "file", source: ".vagrant/machines/crowiplus/#{VAGRANT_PROVIDER}/private_key", destination: "/home/vagrant/.ssh/crowiplus.private_key"
    controller.vm.provision "file", source: ".vagrant/machines/mongodb/#{VAGRANT_PROVIDER}/private_key", destination: "/home/vagrant/.ssh/mongodb.private_key"
    controller.vm.provision "file", source: ".vagrant/machines/redis/#{VAGRANT_PROVIDER}/private_key", destination: "/home/vagrant/.ssh/redis.private_key"
    controller.vm.provision "file", source: ".vagrant/machines/elasticsearch/#{VAGRANT_PROVIDER}/private_key", destination: "/home/vagrant/.ssh/elasticsearch.private_key"

    controller.vm.provision "shell", inline: <<-SHELL
      chmod 600 -R /home/vagrant/.ssh/*.private_key
      chown vagrant:vagrant -R /home/vagrant/.ssh/*.private_key
    SHELL
    ```
