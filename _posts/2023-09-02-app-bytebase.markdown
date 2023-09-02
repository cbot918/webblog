---
layout: post
title:  (分享) bytebase 開源資料庫自動化管理工具 
date:   2023-09-2 12:31:00 +0800
image:  02.jpg
tags:   Resources
---

[Github](https://github.com/bytebase/bytebase)

[官網](https://www.bytebase.com/)

bytebase 是一個資料庫管理界面, 有別於傳統的桌面app, 這款GUI是基於web, 而且反應速度靈敏, 操作起來算舒服.

他有一些特別的功能(可能其他也有, 但不一定有發現或沒這麼普遍)
1. sql editor 有指令提示
2. 除了可以連 rdb 還有 redis
3. 界面上可以開 console
4. 可以創建 project 去管理 db instance
5. 有 schema 管理的功能
6. 可以監看 slow query 

還有其他這個專案在推廣的 DB_CICD自動化 的功能, 就在請有興趣的朋友自己試試看囉, 因為我太菜還看不太懂xD

其中最喜歡的是 1-4, 覺得很不錯的 app 推薦給大家

安裝是我是使用 [docker安裝](https://www.bytebase.com/docs/get-started/self-host/#docker), 直接上面的指令裝好, 瀏覽 localhost:5678, 就可以看到 web 界面囉, 真的覺得是很棒棒的 app 呢！
