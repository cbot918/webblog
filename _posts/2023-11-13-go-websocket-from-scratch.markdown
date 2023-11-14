---
layout: post
title: (演講分享筆記) Go Websocket From Scratch
date: 2023-11-13 22:10:00 +0800
image: 02.jpg
tags: Resources
---

此系列文分享 golang building websocket app from scratch (without websocket library)

## 動機

研究一些網路機制(如 tcp, multithread, protocol 編碼等等)

## 希望的收穫

學會 websocket 怎麼實作, 以及 chatapp 設計

## 希望投稿

- [x] GDG Devfest
- COSCUP

## Getting Started

1.  tcp server

    1.1 組塞的 list = net.Listener()

    1.2 監聽的 for {}

    1.3 組塞的 lis.Accept()

    1.4 封裝 handleConn 函式, 在裡面 print conn 出來看

    1.5 封裝一個 printJSON 函式, 方便印 conn.RemoteAddr 出來看

    1.6 當有連線進來時, printJSON(conn.RemoteAddr)

    1.7 多執行緒 go handleConn()

    1.8 用瀏覽器測試

2.  封裝 Websocket Class

    ```go
    type Websocket struct {
    		Count int
    		Conns map[int]*Client
    }
    ```

    2.1 需要一個使用者數 Counter

    2.2 把建立的連線紀錄起來 Conns

    2.3 函式包好程式寫好, 瀏覽器連線測試一下

<!-- 3.  pre: server 多吐一下靜態頁面, 避免 cors 問題 -->

4.  前端 Websocket 程式碼
    1.
