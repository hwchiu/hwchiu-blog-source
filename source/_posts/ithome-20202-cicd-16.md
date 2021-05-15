---
title: 鐵人賽系列文章- Day 16 - GitOps 的介紹
keywords: 'Kubernetes,DevOps,Linux,K8s'
tags:
  - ITHOME
  - DevOps
  - Kubernetes
description: ITHOME-2020 系列文章
abbrlink: 29451
adate: 2020-12-16 21:57:35
---

之前章節我們介紹過關於 CD 的種種議題，同時也有提到 `CNCF End User Technology Radar` 關於 CD 的使用者報告內，有兩個關於 GitOps 的軟體，其中一個就是 Flux 而另外一個則是 ArgoCD。

其中 Flux 還是報告中被個使用者廠商推薦為推薦使用的專案技術，那到底是什麼是 GItOps

本篇文章就來跟大家介紹到底什麼是 GitOps 以及這個概念相對於過往的 CD pipeline 使用起來有什麼差異



# 介紹

GitOps 的概念源自於 `weaveworks` 於 2017 所提出的一個想法，希望透過 GitOps 帶來一個針對 Cloud Native 的全新 CD 方式

這個概念中，希望可以使用大家已經熟悉且穩定的工具來搭建出一套良好的 CD 方式，這兩個工具就是

1. Git
2. 任何好用的 Continuous Delivery 工具



# 核心概念

GitOps 的核心概念不會太難，分別是

1. Git 作為單一來源
2. 狀態同步
3. 更新方式單一來源

## Git 作為單一來源

GitOps 中強調，所有的資源描述檔案，都集中放於 Git，不論是原生的 Yaml，Kustomize 或是 Helm。 這些內容都要放到 Git 裡面

同時也只能有這個來源，當有人問起你這個 Kubernetes 資源的描述檔案在哪裡，唯一的答案就是 Git 身上

透過使用 Git 帶來一些好處

1. 任何檔案的變化都可以使用 Git History 來觀察，藉此追蹤每個版本的差異
2. 如果有任何修改有問題，想要修復的話，都可以透過 Git 的指令操作，譬如 Revert, 或是再次修正

> 你要用哪一套 Git 其實沒差，其實概念源自於 VCS 版本控制系統

此外， Git 中所放置的資源描述檔案都希望是基於 Declarative 的概念，一種宣告式描述希望狀態的格式，擁有這個要求才可以滿足第二個核心概念

## 狀態同步

第二個核心概念是完全建築在第一個概念的實現，要先完成第一個核心狀態的建置，接下來才可以處理這個

探討這個概念前，我們要先定義兩個資源狀態

1. 使用者渴望的資源狀態
   這個狀態指的是 Git 內所維護的狀態，譬如使用者希望我的 Deployment 有 3 個副本，同時 image 的版本是 1.2.4。
   這也是為什麼前述有說 Git 專案內要使用的是 Declarative 的格式，透過這類型的概念來描述開發者渴望的狀態
2. 正在運行的實際狀態
   這個狀態指的是目標資源目前於 Kubernetes(舉例)內運行的狀態，譬如當前運行的 Deployment 有 2 個副本，使用的 image 版本是 1.2.3

GitOps 會希望有一個代理人(Controller)，這個代理人權責很重，他左邊觀看(1)的渴望狀態，同時右邊監控(2)系統上的運行狀態

這個代理人的最終目標就是要確保 (1) 與 (2) 的狀態一致，大部分的情況下都是把 (1) 的狀態給覆蓋到系統內，讓(2)最後會成為(1)所描述的樣子。

> 部分情況下，管理人員會直接使用一些工具來直接對運行的 Kuberentes 資源進行修改，譬如 kubectl patch, kubectl edit 等工具來修改其運行狀態。一但這種事情發生，就會導致最初描述這些資源的 Yaml 檔案與運行狀態不一致



## 更新方式單一來源

最後要講的則是 GitOps 的更新方式，鑑於前面兩個核心概念的組合，所有的更新都要從 Git 出發

舉例來說，我今天想要更新 Deployment 的 image tag, 我就針對該檔案進行修改，並且遞交一個修正的 Git Commit.

當一切都合併完畢後， `GitOps` 內的代理人接下來就會負責將 Git 上面的狀態資源給同步到目標的 Kubernetes 叢集中，藉此更新 Kubernetes 內的資源。

這種方式帶來幾個好處

1. Git Commit 是唯一的更新來源，禁止其他人透過 kubectl 等工具直接對 Kubernetes 進行部署與修改。這樣當問題發生的時候也比較好追蹤問題來源與除錯
2. 今天版本有問題想要進行退版的時候，可以直接對 Git 進行版本的處理，譬如修正，退版等。只要 Git 這邊搞定，後續就等待代理人將 Kubernetes 叢集內的狀態修正成符合 GIt 上面的格式
3. 就算今天有任何繞過規則，手動對 Kubernetes 內的資源進行手動修改，這些修改都可以被代理人追蹤，可以自動更新回去，迫使所有運行資源都要與 Git 所描述的一致



上述三個核心概念組建出 GitOps 的操作模式，然而這邊都只是概念上的敘述，下一篇會再用圖片跟大家介紹 Kuberentes 架構下的 GitOps 實作方式，當然實作方式也是百百種，不同的開源作案做法也都不一樣。



# 缺點

當然每個技術都不可能完美無瑕沒有任何缺失，接下來將列舉一些別人於 GitOps 實戰中遇到的痛點以及一些領悟，由於內文過長，對於詳細內容有興趣以參閱 [GitOps 帶來的痛點與反思](https://www.hwchiu.com/gitops-bad-and-ugly.html) 內的分析與介紹。

以下列舉文章內的缺點

1. 不適合使用於程式化的更新
2. Git Repo 增長帶來的問題

3. 缺乏視覺化
4. Secret 的管理問題依然沒有解決
5. 缺少檔案資源的驗證性



最後，其實 GitOps 的概念並沒有侷限於 Kubernetes 身上，畢竟 GitOps 就是一個概念，不是一個實作的規格，用任何的工具都有辦法

打造出符合這個核心概念的工作流程，甚至目標部署不是 Kubernetes 也沒有問題。

# 個人資訊
我目前於 Hiskio 平台上面有開設 Kubernetes 相關課程，歡迎有興趣的人參考並分享，裡面有我從底層到實戰中對於 Kubernetes 的各種想法

線上課程詳細資訊: https://course.hwchiu.com/
另外，歡迎按讚加入我個人的粉絲專頁，裡面會定期分享各式各樣的文章，有的是翻譯文章，也有部分是原創文章，主要會聚焦於 CNCF 領域
https://www.facebook.com/technologynoteniu

如果有使用 Telegram 的也可以訂閱下列頻道來，裡面我會定期推播通知各類文章
https://t.me/technologynote

你的捐款將給予我文章成長的動力
<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="hwchiu" data-color="#000000" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#fff" data-font-color="#fff" data-coffee-color="#fd0" ></script>