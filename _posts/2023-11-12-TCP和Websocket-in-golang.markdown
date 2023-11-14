---
layout: post
title: (筆記) TCP 和 Websocket in golang
date: 2023-11-12 22:10:00 +0800
image: 02.jpg
tags: Resources
---

分享一下 Golang 的 TCP 跟 Websocket

go 想要用 tcp connection 的功能需要 [net](https://pkg.go.dev/net)函式庫, 負責去跟作業系統溝通, 創建 connection, 一般簡稱 conn, conn 是 file descriptor 的實作, 所以可以 Read / Write / Close

### [程式碼](https://github.com/cbot918/blogcodes/tree/main/tcp-and-ws)

## Simple TCP

以下是簡單的 tcp server 及 client 程式碼
server.go

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net"
)

const (
	network = "tcp"
	port    = ":8888"
)

func main() {
  // 監聽 localhost:8888
	fmt.Println("listening ", port)
	listener, err := net.Listen(network, port)
	if err != nil {
		log.Fatal(err)
	}

  // 起個 infinite loop 去 Accept 連線
	for {
    // Accept 會 block
    // 回傳的 conn, 就是 socket fd, 可以 Read, Write, Close
		conn, err := listener.Accept()
		if err != nil {
			fmt.Println("connect failed")
			fmt.Println(err)
			return
		}

    // 每個連線進來起一個 goroutine 去處理
		go func(conn net.Conn) {
			fmt.Println(conn) 							// 印出連線看一下
			buf := make([]byte, 1024) 			// 讀取用的 buffer
			for { 													// 持續讀取 socket 來的東西
				n, err := conn.Read(buf) 			// 讀資料, Read 也會 block
				if err != nil {
					if err == io.EOF { 					// client 主動結束會發送 EOF過來, 那就印出來
						fmt.Println("client disconnected")
						break
					}
					fmt.Println(err)
				}
				fmt.Println(string(buf[:n])) 	// 將client來的訊息印出
			}
		}(conn)

	}

}


```

client.go

```go
package main

import (
	"fmt"
	"log"
	"net"
	"time"
)

const (
	network = "tcp"
	port    = ":8888" 				// 接到 server 的 port
	wait    = 3 * time.Second // 等 3 秒, 不然訊息會刷太快
	users   = 5 							// 5個使用者建立 socket 連線
)

func main() {

	fmt.Printf("launch %d users", users)

	// 迴圈將 goroutine 發出去
	for i := 0; i < users; i++ {
		go runClient(i)
	}


	select {} // 要有這行程式才正常執行
}

func runClient(number int) {
	// 連線的函式, client 直接 dial 就會回傳 conn, aka socket fd了
	conn, err := net.Dial(network, port)
	if err != nil {
		log.Fatal(err)
	}

	for {
		message := fmt.Sprintf("hello from user %d", number)
		_, err = conn.Write([]byte(message))				// write 的寫法
		if err != nil {
			log.Fatal(err)
		}
		time.Sleep(wait)
	}
}

```

## Websocket

websocket 我們先看 client 端
index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>ws</title>
  </head>
  <body>
    <script>
      // 其實主要是這一行, 位置正確就會發出一個 http 請求, 裡面會帶著連線需要的資訊
      // Sec-WebSocket-Key 是一個字串, 伺服器端需要用這個字串做一些編碼的工作 再傳回來完成連線升級
      // 就是從 HTTP 連線 變成 Websocket 連線
      const ws = new WebSocket("ws://localhost:8889");

      ws.onopen = () => {
        // 監聽連線
        console.log("socket connected");
      };

      ws.onmessage = (data) => {
        // 監聽訊息
        console.log(data);
      };

      ws.onerror = (e) => {
        // 監聽錯誤, 方便除錯 (但我目前看不太懂)
        console.log("socket error");
        console.log(error);
      };

      ws.onclose = () => {
        // 監聽連線關閉
        console.log("socket close");
      };
    </script>
  </body>
</html>
```

接著看 server 端
main.go

```go
package main

import (
	"crypto/sha1"
	"encoding/base64"
	"fmt"
	"io"
	"log"
	"net"
	"regexp"
	"strings"
)

const (
	network = "tcp"
	port    = "localhost:8889"
)

func main() {
	fmt.Println("listeninging: ", port)
	listener, err := net.Listen(network, port)
	if err != nil {
		log.Fatal(err)
	}

	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Fatal(conn)
		}

		// tcp 連線主體都差不多, 這邊的 handleConnection 把他拉出去寫function, 比較容讀
		go handleConnection(conn)

	}

}

func handleConnection(conn net.Conn) {
	defer conn.Close()
	for {

		// 讀取 http request 的資訊
		buf := make([]byte, 1024)
		n, err := conn.Read(buf)
		if err != nil {
			if err == io.EOF {
				fmt.Println("client disconnetced")
				return
			}
			fmt.Println("read socket failed")
			return
		}
		// 這邊做個 log 比較容易了解情況, 可以看到 http request 的內容
		data := buf[:n]
		fmt.Println(string(buf[:n]))

		// 升級連線的核心函式
		// 其實滿容易理解的, 把 Sec-WebSocket-Key 的值抓出來存到 key 裡面
		// 再把 key 傳入下一個函式去做編碼加工得到 retKey
		// 把這個 retKey, 指定為 Sec-WebSocket-Accept 的值回傳給網頁端
		key := getWebSecKey(data)
		retKey := getReturnSec(key)

		// 把 response 字串透過 conn 寫回去給 client 端, 完成升級連線
		// 這邊重點看 response 字串裡面的內容, 格式要完全正確 \r\n 也是一個不能少, 不然連線會失敗
		// 此 socket 之後就可以用 websocket 的方式 傳送資料(也就是需要按照ws格式編解碼)
		response := fmt.Sprintf("HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-WebSocket-Accept: %s\r\n\r\n", retKey)
		// 這邊將 response 寫出去, 理論上 web 端的 ws.onopen 應該會監聽到連線成功
		_, err = conn.Write([]byte(response))
		if err != nil {
			log.Fatal(err)
		}
	}
}

func getWebSecKey(data []byte) string {
	// 這邊用 regex 簡單的把 key 值抓出來, 回傳出去
	pattern := `Sec-WebSocket-Key: ([^\r\n]+)`
	re := regexp.MustCompile(pattern)
	match := re.FindStringSubmatch(string(data))
	return strings.TrimSpace(match[1])
}

func getReturnSec(webSecSocketkey string) string {
	// 這邊加個 GUID 做編碼, 編碼完再回傳出去
	var keyGUID = []byte("258EAFA5-E914-47DA-95CA-C5AB0DC85B11")
	h := sha1.New()
	h.Write([]byte(webSecSocketkey))
	h.Write(keyGUID)
	secWebSocketAccept := base64.StdEncoding.EncodeToString(h.Sum(nil))
	return secWebSocketAccept
}

```

上面的話， 就是 websocket 的 upgrade 升級連線了, 篇幅太長, 下一篇繼續講傳送訊息

## References

1. [build-a-simple-ws-from-scratch-in-ruby](https://www.honeybadger.io/blog/building-a-simple-websockets-server-from-scratch-in-ruby/)
