---
layout: post
title:  (整理) 一個方便的 Go Cli App Bootstrap Template
date:   2023-08-31 23:13:00 +0800
image:  02.jpg
tags:   Resources
---

這不是我寫的專案([原作者](https://github.com/afarid/simple-todo-cobra)), 我只是整理過, 加個安裝script, 做成template來自用

## 解決問題
可以快速上手開發 go cli app

## Feature
1. 簡單明瞭, 一看大概就知道cli app的指令結構
2. 有一個 ~/.todos 的檔案去維護資料(用[viper](https://github.com/spf13/viper)去操作), 可以作為config參考

### 1. 安裝golang
[golang安裝頁面](https://go.dev/doc/install) 

這邊提供 linux 的安裝指令
下載安裝(他會砍掉舊的裝新的, 原本就有可以跳過這步)
```
curl -OL https://go.dev/dl/go1.21.0.linux-amd64.tar.gz && rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
```
設定環境變數
```bash
export PATH=$PATH:/usr/local/go/bin
source ~/.bashrc
go
# 應該看到go指令畫面
```
### 2. 下載專案
```
curl -o a.tar https://getsub.fiveplanet.online/?url=https://github.com/cbot918/template/tree/main/go-cobra-cli && tar -xf a.tar && rm a.tar
```

### 3. 安裝app
```
./install.sh
```

### 4. 執行
```
todos
```

## 後記
感謝原作者讓我省很多研究時間, 並且可以分享給其他用得到的朋友, 送顆星星給作者！