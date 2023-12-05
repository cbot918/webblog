---
layout: post
title: (筆記) Lionde Remote DB Access
date: 2023-12-5 15:10:00 +0800
image: 02.jpg
tags: Resources
---

紀錄一下 linode 安裝 mysql 直到在本機用 sql editor 連線進去的過程

- 情境：自己的電腦連線到 [Akamai Linode](https://www.linode.com/) VM 裡面的 mysql-server

- 安裝 mysql-server

```bash
sudo apt update && sudo apt install mysql-server
```

- 初始化 mysql-server

```bash
sudo mysql_secure_installation
# VALIDATE PASSWORD component :  N
# root remote access :  N
# 其他看自己
```

- 改 mysql 外部連線設定

```bash
vim /etc/mysql/mysql.conf.d/mysqld.cnf
# bind-address 127.0.0.1  -> 0.0.0.0
```

- 新增遠端存取的使用者

```bash
CREATE USER 'remote'@'%' IDENTIFIED BY '12345';
GRANT ALL PRIVILEGES ON *.* TO 'remote'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

- 開防火牆

```bash
sudo ufw allow 3306
```

- 重開 mysql

```bash
sudo systemctl restart mysql
```

- 本地 sql editor 去連線

```bash
Host: 在 linode dashboard 上面看 ip
Port: 3306
User: remote
Password: 12345
```

- (如果需要的話) 刪除 mysql-server

```bash
sudo apt purge --auto-remove mysql-common mysql-server
sudo rm -rf /etc/mysql rm -rf /var/lib/mysql
sudo killall -9 mysql userdel mysql
```
