---
layout: post
title:  (想法紀錄) 練習設計個 Linode 平台
date:   2023-09-2 12:59:00 +0800
image:  02.jpg
tags:   Resources
---

## 簡稱

Host Machine 簡稱為 Host (就是自己主機的作業系統)

Guest Machine 簡稱為 Guest (就是安裝的Vbox VM)

## 前情提要
這篇主要是講 從 Guest 往 Host 做 http request, 如果需要 Host to Guest, 請參考[這一篇](https://cbot918.github.io/webblog/2023/08/29/new-vagrantvm-ssh-setup/)

因為自己有一些 app 跑在 [Linode平台](https://www.linode.com/) 上, 自己也喜歡 linode的設計(左邊sidebar功能就看完了, 簡單有效適合自學), 加上最近也有在練習， Object storage / Log service 等等, 所以我就在想, 我能不能自己寫一個極簡化版的 Linode ?(暫時叫他Menode好了) 

## 架構 & 選型
1. VM 我可以用 vbox / vagrant 
2. 其他服務, 例如 mysql , minio, loki 等等的都有現成可以使用, 打算用 docker 架
3. 通訊先使用 JSON API, 不然怕太複雜做不出來
4. 希望這個平台可以開源出來, 讓別人一鍵部署起來, 可以作為快速自學的一個很好的方式. 

## 沒想清楚的地方
1. 我本來預設是, 整個app會放在一台 VM 裡面, 畢竟用 docker 開了一堆 application 起來, 會污染環境(其中一部份也是 port 的考量), 但我忘了一件事情：「 Vbox 本身就是 VM 了 」, 如果把 VM 放在 VM 裡, 除了需要 VM 軟體本身有特別支援, 沒意外應該還會有效能問題, 這樣就不是希望做到的 simple and efficient 了
2. 接下來就想其他方案, 因為 Menode 上的 VM 服務, 就是直接裝在作業系統上的 vbox engine 了, 如果以這個為前提的話, 那其他的application服務, 可以用 docker 裝在 Host 就好？雖然這樣還是會污染到 Host 的環境, 但好像沒有什麼其他辦法了(?)

## 後來想到的方案
- 把 application 都放到 K8S 環境裡(我打算用 [kind](https://kind.sigs.k8s.io/)), 這樣起碼是做到隔離了, 這樣只要我能確定, Guest 能對 Host 做呼叫, 就相當於 VM 內外可以做通訊(因為外往內已經做過) , 這樣就能夠存取到 K8S 裡面的服務了

## 呼叫測試
1. 開個VM, 設定好 private_ip, 要確保VM跟 Host 在同一個網段

Vagrantfile
```
Vagrant.configure("2") do |config|

  config.vm.define "db"

  config.vm.network "private_network", ip: "192.168.56.7"

  config.vm.box = "ubuntu/focal64"

  config.vm.disk :disk, size: "20GB", primary: true

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 3
  end

end
```

2. 在 Guest 裡面查 gateway
```go
netstat -rn
/*
vagrant@ubuntu-focal:~$ netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG        0 0          0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 enp0s3
10.0.2.2        0.0.0.0         255.255.255.255 UH        0 0          0 enp0s3
172.17.0.0      0.0.0.0         255.255.0.0     U         0 0          0 docker0
192.168.56.0    0.0.0.0         255.255.255.0   U         0 0          0 enp0s8
*/
```
這樣我就知道 我可以使用 10.0.2.2 連到外面的 Host

3. 在 Host 起個 go echo servier

main.go
```go
pacakge main
package main

import (
    "net/http"
)

func main() {
    f := 
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("hihi"))
    })

    http.ListenAndServe(":8887", nil)
}
```

4. 在 Host 把 防火牆打開
```bash
sudo su
vim /etc/services
```
加入以下內容, 小心不要覆蓋到其他資料
```bash
test    8887/tcp
```
5. 在 Guest 裡面做 curl 測試
```bash
curl 10.0.2.2:8887
# 應該會收到 hihi
```

## 最後還是放棄
上面提到可以呼叫, 我認為實做出最簡陋版的應該沒問題, 但我還是決定放棄了.
因為後來發現, 如果想做在自己主機上還ok, 頂多就是做自動安裝給使用者(但會裝不少東西, 所以已經不太理想了). 但如果我想佈署到雲端就真的沒辦法了, 因為所需要的環境就是一台主機, 而不是一台虛擬主機. 因此就算我做完這個專案, 還是只有我自己可以使用, 別人不能使用, 更別說 simple and efficient了QQ,  所以只能放棄

## 後記
放棄也不是壞事, 因為時間可以流著做其他事情, 而且我覺得經過這整個思考流程, 對一些 infra 網路 的東西, 又有了更多了解！那就 keep going！謝謝觀看的朋朋^^"