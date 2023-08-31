---
layout: post
title:  (開發紀錄) Apps Linode 專案
date:   2023-08-29 12:59:00 +0800
image:  02.jpg
tags:   Resources
---

紀錄一下這個專案, 是由幾個service集合而成的一個專案, 架構上目前打算都做在一台 VM 裡面, 這樣資源存取走內部比較方便一點( 缺點是耦合性會高一些 )

## services
### ghcui ( go-http-client-ui ):
- 類似 postman 的 gui client ,有時候想快速測試 http request, postman比較慢一點, curl 要存起來比較不方便, 有找到一款 [Hoppscotch](https://github.com/hoppscotch/hoppscotch), 比較接近, 但因為自己想把資料存到自己的資料庫, 因此打算自己開發一款可以在web上, 也可以在local簡單自用的
### sm ( service-monitor ):
- 這是一個會輪詢服務, 查看健康狀況的服務, 如果有服務down了, 就會notify給我(目前是發送到slack上), 我會手動去啟動, phase2再讓 sm 具有自動重啟服務功能 
### getsub
- [這篇](https://cbot918.github.io/webblog/2023/08/29/getsub-service/)介紹過的 github 子目錄下載器, 可以 oneliner 下載某個 github repo 下面的子目錄

### [portainer](https://github.com/portainer/portainer)
- docker 監控頁面
### [minio](https://github.com/minio/minio)
- 知名的開源 Object Storage, 目前頁面導轉有點問題, workaround 另開port處理

## 進度規劃
1. 設定一下 proxy: nginx
2. 把各個服務放到 proxy 裡
3. 測試 domain 路由會正確導轉
4. 實作或啟動各服務

## 未來預計加入等等服務做練習
- PostgreSQL
- [Grafana Loki](https://grafana.com/docs/loki/latest/installation/)
- [NSQ](https://github.com/nsqio/nsq)

## 專案網址
- 目前先不對外開放, 畢竟亂到沒法看xD