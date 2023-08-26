---
layout: post
title:  (分享) Local Dev VM Setup
date:   2023-08-25 23:28:00 +0800
image:  02.jpg
tags:   Resources
---

之前一直很想要一個隔離開的本地開發環境, 最近花了點時間總算弄了個簡易版, 寫篇文章做個分享, 希望可以給有類似需求的人一點參考方向

## 解決問題
在本地開一個VM的隔離環境, 並且從Host使用VScode Remote-SSH進去做開發

## 小提醒
此篇會需要對開發環境有一點點熟悉度

## 筆者環境
1. Host OS: [Linux Ubuntu 20.04](https://ubuntu.com/download)
2. VM: [Virtual Box](https://www.virtualbox.org/)
3. VM Wrapper: [Vagrant](https://developer.hashicorp.com/vagrant/downloads?product_intent=vagrant)
4. IDE: [VScode](https://code.visualstudio.com/download)

## 步驟

### 1. 開一台VM出來
Vagrantfile
```ruby
Vagrant.configure("2") do |config|

  config.vm.define "fss"

  config.vm.network "private_network", ip: "192.168.56.5"

  config.vm.box = "ubuntu/focal64"

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 3
  end
end
```
啟動VM
```bash
vagrant up
```
值得注意的是, 我使用了 192.168.56.5 作為private ip (我也不知道為什麼, 隨便找的且能用)
vm image使用 ubuntu/focal64 (就是ubuntu 20)
有設定了 Ram 4G 以及 3 CPU (當時忘記設定硬碟大小, 可以再自行設定)
然後因為設定private_network的關係, 啟動會跑比較久屬於正常, 我的電腦大概跑5分鐘左右
如果沒有private_network, 大概10秒左右就起來了

### 2. 進入VM裡面整理一下開發環境
進入 VM
```bash
vagrant ssh
```
這個步驟就看個人有沒有做環境自動化, 之後整理一下, 再把自己自動化的shell script貼上來(大概就是自動裝好go/nodejs/docker之類的)

### 3. 把ip綁定domain name
在 /etc/hosts 這個檔案內, 多新增一行
應該會需要一些權限, 因為算是權限高的操作, 也別不小心刪到裡面的內容了
```bash
192.168.56.5 testvm.com
```
這邊是把剛剛 VM的 private ip, 綁定成 test.com, 等等就可以使用 domain name 做連線, 真是方便了呢！
在這邊建議可以先直接 測試一下 testvm.com
```bash
ping testvm.com
```
如果有回應封包, 就代表有成功, 如果沒成功, 可能可以聯絡我, 協助排除問題看看之類的

### 4. 從Host ssh進入 VM
這個步驟, 需要設定一下ssh (指定好檔案名字後, 本例子為 /home/.ssh/key, 後面我都enter壓到底使用預設)
```bash
ssh-keygen
```
這邊簡單提一下 ssh 基本原理, ssh-keygen 會產生出一組檔案(一般資安稱為key pair), key跟key.pub,  key是 private key, key.pub是 public key, ssh的用法是, 把 key 放在自己 ~/.ssh 這個資料夾, 把 key.pub 的內容放在 VM 的 ~/.ssh/authorized_keys  這個檔案內, 此為基本原理,  其他一些輔助工具, 都是圍繞著這個原理去使用的

接下來通常是會做個ssh的設定檔
/home/.ssh/config 這個檔案內新增
```bash
Host testvm
  HostName testvm.com
  User vagrant
  Port 22
  IdentityFile ~/.ssh/key
```

接下來直接 ssh 做個連線測試
```bash
ssh testvm
```

這樣理論上就會連進去vm, 他會問 yes/no 的問題, 手動回答一下 yes, 應該就能順利進入vm裡面了
但通常應該會遇到不少問題, 筆者就被ssh打敗很多次, 不熟的時候滿容易卡的, 可以多參考別人教學, 熟悉ssh連線操作方式(原理我上面已經有講了, 可以搭配參考)

### 5. VScode Remote SSH
最重要的這一步, 因為做到這一步之後, 就可以實現在local 使用vscode在隔離的VM環境裡面做開發
- 開啟 vscode
- 打開 command pallete (我的快速鍵是 ctrl+shift+p)
- 輸入 ssh, 下方選單選擇 Remote-SSH:Connect to Host
- 選擇我們設定的 testvm 壓下確定鍵
- 原則上會出現一個新的 vscode視窗, 然後左邊folder區就open一下, 選擇vm內的資料夾
- 然後再打開terminal, 會發現路徑是在VM裡面
- 快樂開發

## 後記
這一篇不算是新手友善文章, 但確是可以比較完整知道怎麼設定local開發環境, 希望對有相關需求的大大有幫助
寫得比較匆忙, 沒有寫得很清楚的地方, 之後會慢慢完善