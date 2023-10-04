---
layout: post
title:  (工具) dbeaver-cassandra
date:   2023-09-17 06:34:00 +0800
image:  02.jpg
tags:   Resources
---

因為想玩一下 cassandra, 發現有些 db gui 沒有連線 cassandra的選項, servey 了一下最後的免費方案 dbeaver 安裝 driver,  以下紀錄一下步驟

### 快速參考:
1. [教學影片](https://www.youtube.com/watch?v=OFST8Bt9J5Q) (我基本上是參考這個)
2. [driver 下載位置](https://github.com/ing-bank/cassandra-jdbc-wrapper/releases/download/v4.10.0/cassandra-jdbc-wrapper-4.10.0-bundle.jar)
3. [cassandra docker](https://hub.docker.com/_/cassandra)

## Getting Started

### Setup Cassandra docker
啟 cassandra instance
```bash
docker run -p 9042:9042 --name cassandra -it cassandra:latest
```
先到 docker 內
```bash
docker exec -it cassandra bash
```
cqlsh
```bash
cqlsh -u cassandra
# 密碼輸入 cassandra
```
看到 cassandra@cqlsh>   
ok, docker cassandra 沒問題

### Setup DBeaver 
1. 先下載 driver
```bash
curl -OL https://github.com/ing-bank/cassandra-jdbc-wrapper/releases/download/v4.10.0/cassandra-jdbc-wrapper-4.10.0-bundle.jar
```

2. 設定
在 dbeaver 界面
1.  [Database] > [Driver Manager] > [New]
2. 在 Settings 標籤頁
輸入設定
```
Driver Name:   Cassandra
Class Name:   jdbc class: com.ing.data.cassandra.jdbc.CassandraDriver
URL Template:   jdbc:cassandra://{host}[:{port}]
Default Port:   9042
```
壓個 ok

3. 在 Libraries 標籤頁
4. [Add File] 把剛剛下載好的 .jar檔案加入, 按下ok, 按下close

#### 測試連線 
1. 新增連線, 找到 cassandra
2. 連線方式用 url ( Connected By 那邊要點一下)
3. 我是用這樣去連 `jdbc:cassandra://localhost:9042`
4. 左下角按一下 Test Connection, 得到ok
5. 右下角按 finish 就ok了



### 後記
開始玩一下 cassandra, 後面如果有相關知識統整起來再來文章好了, 目前大概 0