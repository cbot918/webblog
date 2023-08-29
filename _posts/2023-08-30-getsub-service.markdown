---
layout: post
title:  (專案) github子目錄下載服務getsub
date:   2023-08-29 12:59:00 +0800
image:  02.jpg
tags:   Resources
---

今天需要一個github專案子目錄的下載功能, 社群發問了一下, 感謝朋友幫忙找一些方案
1. [Git Sparce Checkout](https://chat.openai.com/share/ff06d76d-c2af-4bd9-9ccd-b5f4eb28fb13)
2. [Download Git](https://minhaskamal.github.io/DownGit/#/home)
3. [svn checkout](https://medium.com/jeeapex/how-to-download-github-sub-folder-cda8ae2951dd)

一番掙扎下, 試了之前就看過的方法: svn去下載, 但這樣要多安裝一個程式. 參考2 Download Git服務, 想了一下之後有一個想像中的步驟, 覺得自己來實現一次, 當作練習, 之後有機會也要來試試看原生的Git Sparce Checkout

## 目標功能
一行指令, 就把目標subfolder複製下來

## SVN Checkout 原理
1. 安裝 sudo apt install subversion
2. 將想複製的目標連結例如 `https://github.com/cbot918/template/tree/main/go-three-layer-poc`, 
中間的 tree/main 改成 trunk: 
`https://github.com/cbot918/template/trunk/go-three-layer-poc`
這樣就可以使用 svn checkout url 單獨下載下來, 以上就是程式主要邏輯

## Stack 跟主要邏輯部份
1. 語言:      golang
2. http框架:  fiber v2
3. 部屬:      docker
4. 平台:      linode

運用了自己剛做的[bootstrap template](https://github.com/cbot918/template/tree/main/go-three-layer-poc), svn checkout 下來之後, 開始開發.

先把boot template裡不需要的東西都砍掉([使用方式參考](https://cbot918.github.io/webblog/2023/08/29/go-three-layer-poc-project-copy/)), 以下貼一下主要邏輯
```go
func download(oldUrl string) (string, error) {

    // 這邊將傳進來的 oldUrl 做字串取代
	newUrl := strings.Replace(oldUrl, "tree/main", "trunk", 1)

    // regex 將檔案名字取出來, 後面會用到
	folderName := strings.Trim(regexp.MustCompile(`/trunk/(.*)$`).FindString(newUrl), "/trunk/")

	cmd := cmdy.New() // 這邊偷懶使用自己包好的操作 command 的函式庫
	cmd0 := fmt.Sprintf("apk add subversion ") // 這一步其實沒用到, 寫文張的時候發現可以拿掉
	cmd1 := fmt.Sprintf("svn checkout %s", newUrl) // svn checkout newUrl 
	cmd2 := fmt.Sprintf("tar -cvf %s.tar %s", folderName, folderName) // 將下載好的資料夾壓縮成.tar
	cmd3 := fmt.Sprintf("mv %s.tar files", folderName) // 這邊只是把檔案整理一起
	cmd4 := fmt.Sprintf("rm -rf %s", folderName)

    // 以下逐個指令執行
	err := cmd.Run([]string{cmd0})
	if err != nil {
		log.Fatal(err)
	}

	err = cmd.Run([]string{cmd1})
	if err != nil {
		log.Fatal(err)
	}

	err = cmd.Run([]string{cmd2})
	if err != nil {
		log.Fatal(err)
	}

	err = cmd.Run([]string{cmd3})
	if err != nil {
		log.Fatal(err)
	}

	err = cmd.Run([]string{cmd4})
	if err != nil {
		log.Fatal(err)
	}

	return folderName, nil
}

func (ctr *Controller) GetSub(c *fiber.Ctx) error {

	oldUrl := c.Queries()["url"]
	fmt.Println(oldUrl)

    // 執行下載的主要邏輯函式
    name, err := download(oldUrl)
	if err != nil {
		fmt.Println("download error")
		return err
	}
	fmt.Println("name: ", name)


	path := fmt.Sprintf("./files/%s.tar", name)
	nname := fmt.Sprintf("%s.tar", name)
	// 這是fiber框架提供的api, 可以提供client端檔案下載
    return c.Download(path, nname)
}
```

## 測試跟部屬
在local測試完成後, dockerfile 做個 multi-stage-build 縮減image大小(500MB變成50MB), 先試著部屬我的最愛 [Zeabur](https://zeabur.com/) 平台, 自己耍白痴加上後來發現, 應該是我在docker環境裡面做的一些操作, 導致部屬不成功, 所以想說先做個能動就好, 我就去 linode 的 VM 環境部屬看看, 順便測試一下剛買的網址( 在 godaddy 買了一個39元的網址, 開心xD ). 一番折騰後, 終於部屬上去了! 到 cloudflare 設定網域(因為有把 godaddy 轉到cloudflare 上面, 使用他的 https 服務), 新增一筆A record, 指定 cname (自幾決定), 指定目標 ip (去linode看), 就綁好網域了. 這期間還經歷了, 需要把docker的port開到80上面, 如果之後還想增加服務, 預計是需要 Load Balancer 了, 還不太熟, 之後再試(倒)

## 測試指令
適用於 linux like 系統, win 的話指令要另外做, 有機會在 win 上測試再補上來!
```
curl -o a.tar https://getsub.fiveplanet.online/?url=https://github.com/cbot918/template/tree/main/go-three-layer-poc && tar -xf a.tar && rm a.tar
```
預期得到一個 go-three-layer-poc 的資料夾

說明: -o a.tar 算是個 work around, 未來會再優惠, 讓指令不要這麼冗長.

## 後記
再簡單的服務, 菜鳥實際做起來還是還是有些地方會卡卡, 只能多練習, 勤能補拙, 下面附上測試連結, 目前 service 沒有自動 recover, 只能三不五時上去看一下. 這個專案還引出另一個小副本, 在上雲的時候因為很想看 log, 所以就想說, 有機會自己架架看 Grafana Loki, 不過這就是改天再繼續的坑了！

