---
layout: post
title: (線上分享筆記) 試著兩小時學會 JS
date: 2023-11-14 15:10:00 +0800
image: 02.jpg
tags: Resources
---

此系列紀錄前端 web 的新手教學素材

# Getting Started

前端工程師是很多人轉職的選項, 作為軟體工程領域的敲門磚, 學會 javascript 一門程式語言以及 HTML/CSS 作為輔助, 就可以進入軟體工程的領域！這系列的開篇, 我們就用 javascript 作為第一隻箭, 來開始了解什麼是網頁前端開發吧！

- 第一個小時 for fe entry 入門

- 第二個小時 for 想碰點後端的朋友

# Hour 1 第一隻程式

1. 第一隻程式並執行

   ```js
   // 點開瀏覽器 F12
   // 新增程式碼
   console.log("hello world");
   // 按下確定

   // 會看到顯示 hello world
   ```

2. 第一個函式並執行

   ```js
   // 新增程式碼
   function Hello(name) {
     return "hello " + name;
   }
   // 按下確定

   // 新增程式碼
   Hello("Timmy");
   // 按下確定

   // 會看到顯示 hello Timmy
   ```

3. 什麼是變數

   ```js
   // 新增程式碼
   let name = "jack";

   Hello(name);
   // 按下確定

   // 會顯示 hello jack
   ```

4. 判斷式 if-else

   ```js
   // 新增程式碼
   if (name != "") {
     Hello(name);
   } else {
     console.log("忘了設定名字唷!");
   }
   ```

5. 迴圈 loop 及 iterator

   ```js
   // 新增程式碼

   for (let i = 0; i < 10; i++) {
     console.log(i);
   }
   // 會顯示 1 ~ 10

   let users = ["jack", "kasper", "yale", "node"];

   users.forEach((name) => {
     console.log(name);
   });
   // 會顯示 jack, kasper, yale, node
   ```

6. 認識網頁開發三本柱 html / css / js

   6.1 vscode 開個專案

   6.2 index.html

   6.3 style.css

   6.4 main.js

7. 做個 表單頁面

   7.1 div span

   7.2 form

   7.3 action method

   7.4 submit button

8. git 教學 ( 申請 / 推第一個 repo 上去 / gitpage)

# Hour 2 第一個 app

1. express server

   1.1 gpt3.5 唸個咒

   1.2 起一下 express server

   1.3 設定一下 前後端路由串接

   1.4 確定前端 post 資料到後端 正確印出來

2. mongodb

   1.1 regist mongo-atlas

   1.2 get connect string

   1.3 write getDB function

   1.4 module getDB to db.js

   1.5 test code

3. .env config

   1.1 put dsn to .env file

   1.2 install dotenv

   1.3 read data from .env

   1.4 setup .gitignore ( .env, node_modules/)

4. insert data to mongodb

   4.1 write code

   ```js
   try {
     const db = client.db("main"); // Specify your database name
     const collection = db.collection("sheepjs"); // Specify your collection name

     const user = {
       name: name,
       email: email,
       password: password,
     };

     // Insert the document into the collection
     const result = await collection.insertOne(user);
     console.log("Inserted a user:", result);
   } catch (error) {
     console.error("Error inserting user:", error);
   } finally {
     // await client.close(); // Close the connection
   }
   ```

   4.2 進 mongo atlas 看資料有沒有進來

5. 部屬 Zeabur SAAS

   5.1 推專案到 github

   5.2 註冊 Zeabur

   5.3 部屬專案

   5.4 debug time !
