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

新增一個 database(keyspace)
```
CREATE KEYSPACE IF NOT EXISTS history
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 1
};
```
history 是因為我想拿來存聊天的歷史紀錄, 可以自己改名字

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
3. 我是用這樣去連 `jdbc:cassandra://localhost:9042/history?localdatacenter=datacenter1`
4. 左下角按一下 Test Connection, 得到ok
5. 右下角按 finish 就ok了


#### 測試 SQL Editor
滑鼠點開 cassandra 資料庫, 點開 Test Cluster (我是叫這個名字), 右鍵點 history, 開一個 SQL Editor
```sql
CREATE TABLE record (
   id int, 
   to_user int, 
   body text, 
   PRIMARY KEY (id)
);

-- 看 keyspace 裡面有什麼 table
DESCRIBE keyspace;

-- insert
insert into record (id, to_user, body) values (1,1,'hihi');

-- select
select * from record;
```


### golang 連線 cassandra
```go
package main

import (
	"fmt"
	"log"

	"github.com/gocql/gocql"
)

func main() {
	// Create a new cluster configuration
	cluster := gocql.NewCluster("localhost:9042") // Replace with your Cassandra cluster address
	cluster.Keyspace = ""                   // Replace with your keyspace name
	cluster.Consistency = gocql.Quorum            // Set the desired consistency level

	// Create a session to connect to Cassandra
	session, err := cluster.CreateSession()
	if err != nil {
		log.Fatal(err)
	}
	defer session.Close()

	// // Create a keyspace and table (if they don't exist)
	// if err := session.Query(`
	//       CREATE KEYSPACE IF NOT EXISTS mykeyspace
	//       WITH replication = {
	//           'class': 'SimpleStrategy',
	//           'replication_factor': 1
	//       }`).Exec(); err != nil {
	// 	log.Fatal(err)
	// }

	if err := session.Query(`
        CREATE TABLE IF NOT EXISTS mytable (
            id UUID PRIMARY KEY,
            name TEXT
        )`).Exec(); err != nil {
		log.Fatal(err)
	}

	// Insert data into the table
	id := gocql.TimeUUID()
	name := "John Doe"

	if err := session.Query(`
        INSERT INTO mytable (id, name)
        VALUES (?, ?)`, id, name).Exec(); err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Inserted data: ID=%s, Name=%s\n", id.String(), name)

	// Retrieve data from the table
	var retrievedID gocql.UUID
	var retrievedName string

	if err := session.Query(`
        SELECT id, name
        FROM mytable
        WHERE id = ?`, id).Scan(&retrievedID, &retrievedName); err != nil {
		log.Fatal(err)
	}

	fmt.Printf("Retrieved data: ID=%s, Name=%s\n", retrievedID.String(), retrievedName)
}

```


[程式碼連結](https://github.com/cbot918/dbs/tree/main/cassandra)

### 後記
開始玩一下 cassandra, 後面如果有相關知識統整起來再來文章好了, 目前大概 0