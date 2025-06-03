---
layout: post
title: Firecracker MicroVM
date: 2025-6-3 15:10:00 +0800
image: 02.jpg
tags: Resources
---
# Firecracker MicroVM Survey

## 起源
因為最近在看 AWS 的服務 (團隊的決定)，看到 ECS Fargate，想起一個之前看過的專案 Firecraker，這是一個 Rust 寫的輕量級 micro kernel 專案。簡單來說就是，隔離性好，啟動快，資安風險低，輕量省資源！


## Firecracker
Firecracker 是一個 VMM 應用，他是使用了 Linux kernel KVM 的功能，去創造 micro kernel，對標的應用是 QEMU，只不過 QEMU 是創造一個完整的環境，Firecracker 把很多不需要用到的功能例如 bios, 匯流排省略了，程式碼只有數萬行，然後也更安全(畢竟Rust)。

## 後續
這種技術對一兩年前的我還是有點太難了，看過之後輾轉知道 render.app 的機房也有用這個，然後 Zeabur 的創辦人也有提過這個虛擬化技術，後來我就乖乖的修煉 web 技能了，直到今天調查 AWS 服務的時候，又把他拿出來看了一下。我就做了個假設：有沒有可能我拿一台機器，自己用 Firecracker 技術來架個雲呢？當然這是一個省略了一堆重點的不成熟提問，但現在有 GPT 可以做 Deep Research。

## GPT Deep Research 
提問：大概預算 5萬台幣 的主機 我想要自己架雲

GPT動作回應：了解，我會幫你深入研究 Firecracker 的特性、適用場景、架設需求與限制，並評估它是否適合用在你預算約 5 萬台幣的主機上自架雲服務。我會從效能、隔離性、維護複雜度、安全性與對外提供服務的可行性等角度綜合分析。

稍後我會整理完整報告給你參考。

Deep Research 研究結果：
```
Firecracker MicroVM 技術分析與自架雲服務可行性評估
架構特點與傳統虛擬化技術比較
Firecracker 架構概要
Firecracker 是由 AWS 開發的 輕量級虛擬機監控器（VMM），專為大規模多租戶環境設計
github.com
。它運行於 Linux 主機，利用 KVM (Kernel-based Virtual Machine) 內核模組創建隔離的 微虛擬機（microVM）
github.com
。Firecracker 採取極簡主義設計，僅提供最基本的虛擬硬體設備（例如虛擬網路介面、虛擬磁碟），避免傳統虛擬化中許多不必要的裝置與功能
github.com
。由於砍掉了 BIOS、PCI 匯流排等複雜組件，它大幅降低了每個虛擬機的記憶體開銷（每個 microVM < 5 MB）和啟動延遲（最快 ~125 毫秒即可啟動）
e2b.dev
。Firecracker 使用 Rust 語言撰寫，程式碼量只有數萬行，相較於 QEMU 等傳統 VMM 更小且安全（攻擊面積更小）
e2b.dev
。
與傳統 KVM/QEMU 虛擬機比較
傳統的 KVM/QEMU 是通用虛擬機模擬器，支援各種 CPU 架構、完整 BIOS 啟動流程和大量虛擬硬體裝置，功能豐富但相對「笨重」。相比之下，Firecracker 精簡了虛擬硬體模型，不支援舊式或多餘的裝置（如 BIOS、PCI 匯流排、軟碟機等），也不提供圖形介面或 GPU 模擬
academia.edu
e2b.dev
。這意味著 Firecracker 無法直接啟動 Windows 等需要 BIOS/UEFI 的作業系統，僅能載入 Linux kernel 或類似 unikernel 的映像檔作為客體
academia.edu
。由於精簡，Firecracker 啟動 VM 的速度比傳統 QEMU 快數倍，資源佔用更低；測試顯示 Firecracker 啟動 VM 的速度可比 QEMU MicroVM 模式快約3倍，較一般 QEMU 快約10倍
e2b.dev
。每個 Firecracker microVM 的額外記憶體負擔不到 5MB
e2b.dev
 ，而典型 QEMU VM 則需要較高的開銷。QEMU 擁有龐大的C語言代碼庫（約數百萬行），歷史上曝露不少安全漏洞；Firecracker 則藉由小型Rust代碼庫和精簡介面降低了攻擊面，安全性更高
e2b.dev
。然而，傳統 QEMU 的彈性更大：支援各種裝置模擬、進階功能（如即時遷移 VM）等，而 Firecracker 出於設計目的不支援 VM 熱遷移等複雜功能
academia.edu
。總而言之，Firecracker 犧牲部分通用性來換取極致的啟動速度、資源效率與安全隔離
e2b.dev
。
與 Docker 容器比較
容器（例如 Docker）和 Firecracker 都能提供輕量級的應用運行環境，但實現方式截然不同。隔離層級方面，Firecracker 基於硬體虛擬化（KVM）提供類似傳統 VM 的強隔離，每個 microVM 有獨立的客體內核，安全隔離更嚴密，不易受其他工作負載干擾
medium.com
。Docker 容器則共享宿主機的 Linux 核心，以進程與名稱空間技術實現隔離，開銷更低但理論上隔離不如 VM 完全
medium.com
。效能開銷方面，容器由於共用宿主核心，幾乎沒有虛擬化損耗，資源效率極高；Firecracker microVM 因有一層硬體虛擬化，開銷略高於容器，但已經非常輕量，幾乎接近裸機效能
medium.com
。啟動時間上，Docker 容器可以在幾乎瞬間啟動（通常秒級甚至更快），而 Firecracker 啟動 microVM 需要 ~100 多毫秒，雖稍慢於容器但仍遠比傳統 VM 快
medium.com
。在使用場景上，Firecracker 適合需要強隔離、多租戶安全的情況，例如無伺服器函式運算（FaaS）或多使用者的服務環境；Docker 則更適用於一般微服務、CI/CD pipeline 和開發環境，強調部署速度和資源共享效率
medium.com
。管理介面方面，Firecracker 提供簡單的 REST API（需要透過上層工具整合），而 Docker 具有成熟的 CLI 和豐富的編排生態（如 Kubernetes）的支援
medium.com
。總之，當安全隔離是首要考量時，Firecracker 提供接近 VM 安全性的同時維持接近容器的速度與輕量；若追求最大化的便利性與性能，且接受共用內核的風險，Docker 容器仍是更簡單的選擇。
適用場景與限制
適用場景
Firecracker 的設計初衷就是在多租戶環境中以高密度執行工作負載，同一台實體機上可併發大量彼此隔離的微VM。這非常適合雲端的 Function as a Service（FaaS，如 AWS Lambda）或 容器即服務（如 AWS Fargate）場景：每個函式或容器群被封裝在自己的微VM中，達到安全隔離但幾乎不影響啟動速度與資源利用率
firecracker-microvm.github.io
github.com
。在 CI/CD、自動化測試等需要短暫且高並發啟動環境的場合，Firecracker 也能快速啟動隔離的輕量VM來執行任務，避免傳統VM過慢的啟動延遲。對於邊緣計算或需要在眾多地點部署小型服務的情境，Firecracker 也是理想選擇：它提供接近容器的啟動速度與密度，且因為是硬體級隔離，更能防範惡意用戶嘗試跨租戶攻擊
e2b.dev
。簡而言之，凡是需要同時運行大量隔離實例且每個實例資源需求不大的情況（例如無伺服器函式、多租戶微服務、沙盒執行不信任程式碼等），都是 Firecracker 大顯身手的場合
medium.com
。
不適用場景／限制
儘管 Firecracker 非常強大，但在某些情境下並非最佳選擇。首先，如果需要模擬完整 PC 環境或特殊硬體裝置（例如 GPU 加速、USB 裝置、聲音/圖形介面），Firecracker 無法滿足：它不支援 PCI-E 裝置，因此無法直通 GPU 等硬體，相關開發工作已暫停
e2b.dev
。同樣地，傳統 VM 平台支援的許多裝置（如光碟、軟碟、各類匯流排）在 Firecracker 中一概不存在
academia.edu
。其次，Firecracker 僅支援載入 Linux 核心的客體；無 BIOS/UEFI 意味著無法直接啟動 Windows 等非Linux系統
academia.edu
。對於需要 Windows VM 或完整通用OS支援的工作負載，仍需採用傳統 KVM/QEMU。再次，如果應用對啟動速度不敏感且需要長期穩定運行的大型服務，使用常規 VM 或容器可能更簡便，因為 Firecracker 雖輕量但要求您自行處理不少管理細節（見下節）。另外，Firecracker 不支援 VM 即時遷移，因此在需要高可用性（如在不同主機間遷移運行中 VM）時，它不如傳統虛擬化靈活
academia.edu
。總體而言，Firecracker 適用於特定類型的雲端工作負載，但不適合需要完整虛擬硬體支援、圖形介面或特殊設備的傳統 VM 場景。如果只是單租戶環境且容器已能滿足需求，則引入 Firecracker 可能增加不必要的複雜度。
安裝、部署與維護
安裝與部署難度
Firecracker 本身安裝相對簡單：官方提供靜態編譯的執行檔，可直接下載並在 Linux 上運行
medium.com
。啟動 Firecracker 後，透過其 REST API 或命令列即可配置並啟動微VM。例如，您需要提供一個客體 Linux kernel 映像檔（vmlinux）以及 根檔案系統映像（如 .ext4檔案）給 Firecracker，由它載入啟動微VM
medium.com
medium.com
。部署 Firecracker 微VM涉及一些手動設定工作：
準備 Kernel 和 RootFS：使用合適的Linux核心映像（需為未壓縮的 ELF 格式）和一個根檔案系統映像作為 microVM 的作業系統環境
medium.com
。AWS 官方提供了參考的 kernel 和 minimal rootfs，可供測試使用。
配置網路：Firecracker 微VM使用 Linux tap 裝置作為虛擬網卡接口
blog.cloudflare.com
。部署時需要在宿主機建立對應的 tap 介面並設定 IP。例如可將 tap 接到 Linux bridge，使 microVM 獲得與宿主相同網路的 IP，或透過 NAT 方式讓 microVM 的流量經宿主轉發
blog.cloudflare.com
。這些都需由管理者自行透過 ip tuntap、brctl 或 iptables 等指令設定（如建立一個名為 tap0 的介面並配置 IP/路由)
blog.cloudflare.com
。Firecracker 並沒有內建高階的網路配置，需由使用者按需求設定主機的轉發或埠轉映規則。
使用 Jailer 強化隔離：Firecracker 附帶一個 "jailer" 工具，可在啟動 microVM 時將其進程禁錮在嚴格的 Linux 沙盒環境（使用 namespace、cgroup、seccomp 等機制）
github.com
。在生產環境建議透過 jailer 啟動 Firecracker，確保每個 microVM 進程只能存取被授權的資源，進一步提升安全隔離。這增加了一點部署複雜度（例如需為每個 VM 準備獨立的資料夾、裝置節點權限等），但對多租戶安全非常重要
github.com
。
總的來說，安裝 Firecracker 的門檻不算高，但部署 microVM 的流程相對手動：不像 Docker 那樣拉取映像即可執行，您需要準備 kernel/rootfs、設定網路、調用 API 啟動 VM 等步驟。官方也提供了一些整合工具（例如 firecracker-containerd、Weave Ignite）來簡化這些流程，把 Firecracker VM 管理類似容器化，但這額外引入了其他元件複雜度。
系統維護需求
由於 Firecracker 僅提供輕量級 VMM 功能，它本身不包含完整的 VM 管理或編排系統
academia.edu
。因此，在自架環境中長期運行多個 microVM，維護上需要考慮：
編排與管理工具：若需要同時管理多個微VM的生命週期、資源分配，建議搭配容器編排/管理工具。例如，使用 containerd 的 Firecracker 插件（firecracker-containerd）可以將 Firecracker 微VM 納入類似容器的管理流程
academia.edu
。也有社群工具如 Weave Ignite，可用 Docker 映像直接啟動 Firecracker VM，簡化映像製作與管理。若不使用這類工具，則可能需要撰寫腳本或程式透過 Firecracker API 來管理多個 VM，涵蓋開機、關機、監控等功能
academia.edu
。
主機系統維護：Firecracker 微VM 的安全依賴於宿主 Linux 的安全配置
github.com
。需要確保宿主機啟用了適當的安全強化（例如只允許可信用戶訪問 /dev/kvm，防火牆隔離微VM流量等）來維持多租戶隔離。AWS 官方提供了生產環境主機設定的指南以滿足安全基線
github.com
。
資源監控與優化：在長期運行中，要監控每個 microVM 的CPU、記憶體使用。Firecracker 支援對網路和磁碟IO進行速率限制
firecracker-microvm.github.io
（Rate Limiter），可以配置每個 VM 的頻寬或IOPS上限，防止單一 VM 壟斷資源
firecracker-microvm.github.io
。另外，Firecracker 目前對記憶體回收（ballooning）的支援有限，一旦 microVM 分配了記憶體，即使釋放也不會主動還給宿主
news.ycombinator.com
。因此需要在啟動時合理分配VM記憶體大小，或透過重啟VM來回收資源。
更新與相容性：Firecracker 大約每兩三個月會有新版本發布
github.com
。維護時需關注其更新日誌，評估是否需要升級以獲得安全修補或新功能。由於 Firecracker社群相對較新且生態較小，遇到問題可能需要閱讀文件或 GitHub issue 尋找解決方案
e2b.dev
。
總體而言，維護 Firecracker 平台需要具備一定的 Linux 系統和虛擬化知識。相較於使用成熟的雲平台或容器平臺，自行架設 Firecracker 雲服務會增加一些初始開發整合成本。但在搭建完成並驗證穩定後，其日常運維負擔並不沉重：微VM 本身穩定運行所需的干預很少，重點在於上層的管理編排和宿主機環境的維護。
支援平台與硬體需求
Firecracker 對硬體平台的要求是現代的64位元處理器且支援虛擬化技術。官方表示目前支援 x86_64（Intel/AMD）以及 Arm64 架構的CPU，只要啟用了硬體虛擬化指令集（如 Intel VT-x/EPT 或 AMD-V/RVI）皆可運行
firecracker-microvm.github.io
。這意味著一般的伺服器級或桌上型處理器（近十年的型號）都能符合要求。宿主作業系統必須是 Linux（因為 Firecracker 利用 Linux KVM 模組），主流發行版的新版內核通常都包含 KVM 支援。需注意在 Linux 核心中載入 kvm 模組，以及使用者需具有開啟 /dev/kvm 的權限（通常需要 root 或kvm群組）。另外，網路功能依賴 Linux 的 tun/tap 模組（通常預設提供），以建立虛擬網卡。 在客體（微VM）方面，目前 Firecracker 僅能運行Linux內核的系統。一方面因為 Firecracker 要求提供一個 Linux kernel 映像檔直接啟動（不經過 BIOS），另一方面 Firecracker 的虛擬硬體（virtio裝置等）在 Linux 內建良好支援。理論上其他類Unix內核若能以類似方式啟動也可運行，但 Windows 等系統則不支援
academia.edu
。 Firecracker 自身對硬體資源的額外消耗很小，但為了讓多個微VM 高效運行，建議配置充足的記憶體和CPU核心數。每個 idle 的 microVM 大約只佔用 <5MB 記憶體
firecracker-microvm.github.io
（不含客體系統本身的使用量），因此主要記憶體開銷來自您為每個 VM 分配的 RAM 大小總和。CPU方面，Firecracker 可讓多個 VM 共享核心並支援超額訂閱（Oversubscription）
github.com
——也就是說，您可以開啟的 microVM 數量不必被實體核心嚴格限制，只是若同時有多個 VM 繁忙運算，需要靠 Linux 排程去切分CPU時間。
網路與對外服務支援
Firecracker 微VM 能夠透過虛擬網路介面與外部通信，但需要在宿主機進行一些網路配置。每個 Firecracker microVM 啟動時，您可以透過 API 為其新增一個或多個網路介面，對應到宿主的一個 tap 裝置
github.com
。Tap 裝置是一種虛擬乙太網介面，Firecracker 將 microVM 的虛擬網卡流量經由 tap 送達宿主機。宿主機內核則可像處理實體網卡一樣處理該 tap 流量——通常宿主會扮演路由器或防火牆的角色，將 microVM 的封包轉發或進行 NAT 出去
blog.cloudflare.com
。 常見的對外網路接通方式有兩種：
橋接模式：將 Firecracker 的 tap 介面加到宿主的一個網路橋（bridge）中，使 microVM 仿佛直接連接在橋接網路上。這樣可賦予 microVM 一個與宿主同網段的 IP 位址（可以是透過 DHCP 或手動設定），外界就能直接與該 IP 通信，如同這 VM 是本地網路中的一台機器。這種方式下對外綁定 IP 非常直接，microVM 自己擁有IP並開放埠，即可被外部存取。
NAT模式：宿主機為 microVM 提供內部私有 IP（例如 192.168.x.x），透過 iptables 設定進行來源位址轉換（MASQUERADE）將 microVM 的出站流量轉發至外網，並將外部流量轉發回對應 microVM（目的位址轉換）
medium.com
。管理員可以設置埠轉發規則，例如將宿主機的某個埠映射到 microVM 的內部服務埠，從而讓外部流量透過宿主轉送到 microVM 上運行的服務。這種方式類似 Docker 的 --publish 埠轉發，實現微VM 在無直接IP的情況下提供對外服務。
無論橋接還是 NAT，Firecracker 本身不會自動完成這些網路設定，需要管理者在宿主機設定好相應網路。如果需要微VM 之間互通或與宿主通信，也可以配置 Linux bridge 或使用 vsock (虛擬socket) 等機制。總的來說，Firecracker 微VM 的網路能力與一般 KVM 虛擬機類似，只要正確配置宿主網路，它們就能擁有對外的連線能力、綁定獨立 IP 或透過特定埠對外提供服務。
在中階硬體上部署多租戶微服務的適用性
假設您擁有一台約新台幣 5 萬元等級的 x86_64 實體主機（可能配備如 8-16 核心 CPU、32~64GB 記憶體），使用 Firecracker 來打造自架雲服務平台是可行且適合的，但需注意平衡效益與複雜度。Firecracker 的特性使其能在單機上運行大量隔離的微服務實例：每個 microVM 的額外開銷極低，理論上可在一台伺服器上啟動上千個微VM（AWS 實測每台主機每秒可啟動高達 150 個 microVM）
firecracker-microvm.github.io
firecracker-microvm.github.io
。實際可同時穩定運行的數量取決於硬體資源，但相較於傳統 VM，Firecracker 能以更高密度承載微服務而不犧牲安全隔離。這非常適合多租戶微服務：不同客戶或模組的服務部署在各自的微VM中，彼此安全隔離，而宿主機的 CPU/記憶體可透過 KVM 超額分配機制充分利用，提升成本效益
github.com
。 在中階硬體長期運行 Firecracker 微VM 一般是穩定的。Firecracker 已被廣泛應用於 AWS 內部的 Lambda 和 Fargate 平台，證明了其在長時間多租戶負載下的可靠性
e2b.dev
。例如，Edge 平臺 Fly.io 也採用 Firecracker 來啟動隔離的輕量 VM 為使用者運行應用
e2b.dev
。因此，即使您的硬體規模不及超大資料中心，依然可以藉助 Firecracker 建立小型的「雲」服務環境：將這台主機視作一個 mini 雲節點，在其上部署多個微服務 VM 對外提供 API。 需要強調的是，雖然 Firecracker 非常高效，但自架多租戶平臺仍有一些考驗：您需要建立適當的管理機制，防止資源爭用（可利用 Firecracker 的 IO/網路限速功能
firecracker-microvm.github.io
來避免某個 VM 佔滿頻寬）、監控各 VM 的健康狀態，並確保整臺主機的安全更新。由於 Firecracker 缺乏傳統虛擬化的一些功能（如即時遷移），在單機部署時要做好故障應變預案（例如定期備份快照，VM 故障時快速重啟服務於新 VM）。 綜合評估，若您的目標是在有限預算的硬體上最大化地運行多個隔離微服務，Firecracker 是相當適合的選擇。它能提供接近裸機的性能和容器般的敏捷，同時賦予每個服務強大的安全沙箱隔離
e2b.dev
。只要對其架構特性和限制有清晰認識並做好相應的配置管理，一台 5 萬元級別的 Linux 主機完全可以藉助 Firecracker 成為一套可靠的自建雲服務平台，用於部署多租戶的輕量虛擬機以對外提供各種 Web API 或服務。
```