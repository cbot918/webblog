---
layout: post
title:  (分享) go三層式架構做個poc專案的流程
date:   2023-08-29 12:59:00 +0800
image:  02.jpg
tags:   Resources
---

最近有新專案在做poc demo, 把專案流程做個紀錄, 算是個人目前的best practice, 紀錄並分享有相關需要的朋友

## stacks
1. golang
2. fiber http framework
3. mysql 
4. docker-compose

## 目錄結構
```
├─ internal/
│  ├─ api.go          ( api routing )
│  ├─ config.go       ( config structure)
│  ├─ controller.go   ( layer1 handle http request)
│  ├─ repository.go   ( layer3 access database)
│  ├─ service.go      ( layer2 business logic )
│  ├─ types.go        ( data types in here aka model)
│  └─ types.go        ( some helper function )
├─ docker-compose.yml ( IaC deploy )
├─ fso.sql            ( sql schema )
├─ main.go            ( entry point )
└─ req.sh             ( testcase )

```
## Flow
1. 設定檔 .env
- 先改一下.env內的設定
- 注意一下 sql user, password, port, dbname 設定正確

2. 起 MySQL  
注意 docker-compose 裡面設定的port 跟 .env裡面一致
`docker-compose up  `

3. 到 MySQL container 裡面新增 sql, 並做測試
```bash
sudo docker start fso && docker exec -it fso bash
# 會進到container裡面
mysql -u root -p
# 密碼輸入 12345(或自己設定過的)
# 會進入mysql cli裡面
use database fso;
```
貼上 fso.sql 的內容後按下enter (以下節省時間幫貼上來)
```sql
CREATE TABLE users(
  id        int NOT NULL AUTO_INCREMENT PRIMARY KEY,
  email     varchar(255) NOT NULL,
  name      varchar(255) NOT NULL,
  password  varchar(255) NOT NULL
);
```
會新增成功, 接著做個快速測試
```sql
SELECT * FROM users;
INSERT INTO users(name, email, password) VALUES('test','test@gmail','12345');
SELECT * FROM users;
```
應該會看到一筆資料

4. 做個API測試 (已經準備好測試資料)
```
./req.sh
```
預期結果 
`{"email":"test123@example.com","Name":"test123"}`


## repo ( 因為是repo下的folder, 所以用特別的方式抓取)
```bash
curl -o a.tar https://getsub.fiveplanet.online/?url=https://github.com/cbot918/template/tree/main/go-three-layer-poc \
 && tar -xvf a.tar && rm a.tar
```

## 後記
因為這個專案的repo是某個專案的子目錄, git沒法直接下載, 所以寫到一半跑去寫下載github子目錄的service, 算是寫出來了, 但還不夠完善, 下載repo的時候需要整串指令, 但起碼堪用, 希望可以對有相關需求的大大有幫助^^