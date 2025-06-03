---
layout: post
title: Write Your Own Docker
date: 2025-6-2 15:10:00 +0800
image: 02.jpg
tags: Resources
---
# Write Your Own Docker

## 寫在前面
非教學只是自己凌散的一些筆記，有些概念甚至可能說得不太對，主要是邊了解邊紀錄，如果是學習角度不會特別建議閱讀，比較適合本身就在做相關程式研究的大大。

## 起源
最近因為另一個小小的計畫，所以想先試試看自己寫 docker，看了一下網路上原理文章也是滿多的，現在還能搭配 GPT 一起 pair code，所以滿方便的。

## 原理-docker image
Docker image 的本質是一些檔案集合(其實就是一個 linux 的檔案)，我們可以藉由`docker pull alpine` 然後 `docker save alpine -o alpine.jar` 這兩指令，可以得到一個壓縮檔，把他解開會有 meta data 包括 manifest.json，這邊的 Layer 部份紀錄了一個 hash value，拿這個 hash value 去 blob 資料夾裡面找到那個 hash value，對他做 `tar -xvf hash_value` 就會發現，解開了一個 linux 的檔案系統。

## 是一些檔案然後呢
我好奇平常我們做 `docker run ` 不是感覺會進到另外一個作業系統的感覺，後來在實作的過程知道了是使用`/proc/self/exe` 去做執行，就會跳進這個 shell 裡面了 例如 `/proc/self/exe my_image /bin/sh`

## 實作
先請 GPT 生一些程式碼
```
cmd.go // 目前支援 ducker load 跟 ducker run 

fs.go // 負責把上面提到的 alpine.tar 複製到目前環境的某個地方集中管理 (ducker load 做的事情)

runtime.go //負責 /proc/self/exe my_image  的部份 (ducker load 做的事情)

```

先就這兩個功能，用到能動就可以了

## 採坑
我們常常在操作 docker，都會去執行 /bin/bash 或是 /bin/sh，我在 fs.go 做檔案複製的時候，發現這些檔案是有類型的
```bash
1. 資料夾
2. 檔案
3. 軟連結/硬連結
```

所以我一直以為複製完去執行就會進去了，但我後來直接去看，確實 bin 資料夾裡面沒有 sh

## Busybox
busybox 是一個 linux 工具箱，裡面提供了 shell 環境裡面的工具，包括 sh。我發現複製過去的檔案，在 bin 裡面有 busybox 這個東西，所以我們就可以 ducker run busybox /bin/sh 這樣做


## 一些沒有提到的 docker 核心概念
chroot: linux 提供的功能，讓使用者可以把某個地方變成根目錄（封起來）
namespace, cgroup 這部份網路上文章滿多的可以直接搜尋來看

## docker 的 layer 概念
我們在 pull docker image 的時候，會發現很多 hash值，其實就是前面提到的 layer 內的 hash 值。docker 在最開始可能是個 alpine，然後裝入 nginx，然後可能使用者在裡面新增了一些東西又再包一次，所以他是一個層層打包的概念，layer1 alpine, layer2 nginx, layer3 新增的檔案。

## podman
在整個過程中，我原本想先 build podman 來看他原始碼，但後來因為 podman 原生是 debian 還是 centos 去做的，最後沒有在 ubuntu 上面 build 起來，所以我就放棄，直接去跟 GPT 一起實作。

## 後記
這算是自己一個 project 裡面的子 project，就滿好玩的但專案真的很大就需要時間慢慢累積。