---
layout: post
title:  (流程紀錄) 開新的 Vagrant VM 及 SSH 設定紀錄
date:   2023-08-29 12:59:00 +0800
image:  02.jpg
tags:   Resources
---

每次做這段都會卡SSH設定, 今天總算摸出一個確定能動就好的流程, 紀錄並分享

## 解決問題
開一台 vagrant 機器, 並且設定好 ssh, 連線進去

## 小提醒
這一篇跟 Local Dev VM Setup 這邊有一些地方重複, 不過著重的點不同, 故另開篇

## Getting Started
1. 開 VM
Vagrantfile

```
Vagrant.configure("2") do |config|

  config.vm.define "nginx"

  config.vm.network "private_network", ip: "192.168.56.6"

  config.vm.box = "ubuntu/focal64"

  config.vm.disk :disk, size: "20GB", primary: true

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 3
  end
end
```

2. 啟動 VM 並連線測試
```
vagrant up
vagrant ssh
```
會進到內部, exit是退出VM

3. 回到 Host Machine(外面那台自己的) 產生keypair
```bash
ssh-keygen
# 此例子指定名稱為 /home/yale/.ssh/nginx
```

4. 將 nginx.pub 複製到 VM的 ~/.ssh/authorized_keys 內

先將 nginx.pub 的內容複製起來

在 VM 裡面
```bash
vim ~/.ssh/authorized_keys
# 接著貼上資料, 注意不要覆蓋到原本第一把key, 那個是 vagrant ssh 用的
```

5. 在 Host Machine 綁 /etc/hosts

在 /etc/hosts 新增以下內容(注意新增時不要覆蓋到舊的)
```
nginx.com  192.168.56.6
```

6. 在 Host Machine 設定 ssh config

打開 ~/.ssh/config 加入以下
```
Host nginx
  HostName nginx.com
  User vagrant
  Port 22
  IdentityFile ~/.ssh/nginx
```

7. 測試連線
```
ssh nginx
```
接著輸入yes, 然後應該就會連線進去了


## 除錯筆記
1. 如果不小心沒用好, publickey denied, 希望重製, 我自己是會把 Host 的 ~/.ssh/known_hosts 內容全部刪掉, 然後再 ssh nginx 去連線
2. 如果連線不順利, 可以使用 `ssh -v -i ~/.ssh/nginx vagrant@nginx.com` -v是把debug訊息印出來
3. 如果真的很不順, 歡迎聯絡我 DC yale918#9832, 幫忙debug
