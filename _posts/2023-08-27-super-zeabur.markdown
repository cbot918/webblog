---
layout: post
title:  (分享) 個人覺得最棒的部署平台之一
date:   2023-08-27 17:53:00 +0800
image:  02.jpg
tags:   Resources
---

身為一個菜鳥Web工程師, 常常需要練習或做一些debug, 免不了把網頁部署起來, 用過不少平台: AWS, GCP, Heroku, Vercel, Render, Flyio, Linode, Zeabur

使用者體驗前三名是
1. Zeabur
2. Heroku
3. Linode

## 起因

我想特別介紹 [Zeabur](https://zeabur.com/)

起因是Heroku收費後, 想找個地方放應徵的作品(兩個MERN網站), 一番努力後我放上了Render, 但卻發現啟動速度竟然比原本的Heroku 10秒啟動還慢

偶然看到有人推薦Zeabur, 雖然沒聽過這個平台, 但想說試試看, 失敗了就啟動速度慢就算了.

一用下去, 我就黏住了

我部署了兩個MERN, 後來教別人寫code, 部署了java的ws demo上去, 常常在幫人測試debug, 我就砍掉舊的部署新的, 一個帳號不夠用我開了兩個, 三個xD

今天又幫別人debug, 要抓取server domain, 我在local測試過了, 我查了一下Zeabur部署Sveltekit, 直接clone template下來, 改一改, 推code, 還能設定cname, 大概5分鐘就成功了

我只能說, 省了不少肝xD

## 開源
有次在水球DC, 竟然有Zeabur老闆的演講, 我直接手刀. 他把Zeabur的架構做分享, 讓大家有個脈絡可以實做自己的SAAS平台, 我第一個想法是 OMG怎麼可以這麼佛心, 而且我是真的聽過一遍, 我就覺得我大概知道怎麼蓋了, 尤其是有個很酷的技術 https://github.com/zeabur/zbpack,  這個專案可以根據使用者上傳的repo, 去判斷是哪種語言, 然後幫你包成一個docker image, 然後部署上去, 也就是說, 真正的無痛, 因為新手連build script怎麼寫都不知道, 但不用知道, Zeabur會處理好一切.

這個專案我有clone下來玩過, 試過一次上癮, 後來我簡單的測試container, 都用 zbpack 直接幫我生成image, 我就不用寫dockerfile了, 賺爛!

## Zeabur有多好
1. 預設深色畫面眼睛舒服
2. 畫面按鈕很少, 但都知道應該壓哪裡
3. 內建github 的自動部署, 推完code, 平台就會更新app
4. 網址最前面的cname可以自訂, 還能修改
5. 網頁啟動速度超快

## 後記
目前還沒有產品穩定運行在上面, 不過我三不五時會使用一下, 希望之後能寫出有人用的網站, 放到上面一段時間看看, 希望更多人有機會可以試試看這個很酷的平台！