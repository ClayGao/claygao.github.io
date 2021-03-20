---
title: "[記錄] 從零打造一個屬於你自己的 Hugo Blog !"
date: "2020-02-19T22:40:32.169Z"
template: "post"
draft: false
slug: ":D"
category: "hugo"
tags:
  - "Hugo"
  - "Blog"
  - "Web Markdown"
  - "Go"
description: "我也不知道為什麼自己要在 Gatsby 建構的 blog 中放 hugo 的教學 ..."
---

## 前言

我自己原本是有寫部落格的習慣，可能是工作上的體悟，或者是技術上的筆記，但大多都存在 Medium 與 Coderbridge 上，這兩個平台都相當好用，也是我目前的主力。

但其實最早養成筆記習慣的，還是在 vscode 上面自己寫 .md，然後 push 放在 GitHub repo 上面，這樣做的好處算是相當簡單，做法單純，而缺點則是 SEO 不佳，自己的筆記和文章不好被搜尋到，這時候還能怎麼做呢？

感謝同學千，安利了 Hugo (好啦應該也不算安利，只是碰巧知道他也有在玩)，所以就埋頭來研究要怎麼用 Hugo 架設一個部落格。

相比 Medium 與 Coderbridge 這些充滿種多功能的平台來說，的確少了一些功能，但畢竟是自己從零打造的，趣味性就相當高了，這也是今天我想要介紹架設過程的原因之一。

其實我知道相關的工具還有 Hexo 與 jekyll，但在研究 Hugo 之前其實也對他們完全不熟，不知道他們是如何運作的，所以如果你像我一樣完全不懂，那也沒有關係，可以照這一篇來一步一步架設自己的部落格。

在架設 Hugo Blog 之前，有一些工具是我們需要具備的：

- Git 基本指令操作 (如 `pull`，`push` 與 `commit` 等)
- GitHub 基本知識
- Command Line 的使用

最後，本篇文章將以 macOS 為預設作業系統來進行介紹。
## 一、安裝 Hugo

Hugo 本著由 Go 語言開發而成的靜態網站產生器，讓使用者可以快速建立自己的靜態網站，而其實部落格只是**其中一種應用形式**而已。所以如果要利用 Hugo 產生履歷 CV，甚至是專案的 Release Note 當然也都沒有問題。

在 Mac 上安裝 Hugo 的方式相當簡單，只要先安裝 [Homebrew](https://brew.sh/index_zh-tw) ，並在 Terminal 執行以下指令，如果你的 mac 已經安裝過 Homebrew，那麼直接執行以下指令來安裝 Hugo 即可：

```cli
brew install hugo
```

又或者如果你要親自編輯 hugo 框架，可以先安裝 Go 語言環境，再將 Hugo 以環境套件的形式安裝到套件包，這邊就先不多加描述了。

## 二、直接產生本地靜態網站

這一個部分也可以看成直接架設本地部落格，結果會生成資料夾。

假設我們這個專案叫做 `hugo-blog`，那麼可以在你想要的路徑執行：

```cli
hugo new site hugo-blog
```

這時候你應該就會看到 `hugo-blog` 被建立了，同時你也可以在 Terminal 上看到以下敘述：

{{<figure 
    src="/images/2021/build-your-hugo-blog/01.png" 
    title="貼心指引下一步" 
>}}

基本上就是 Hugo 很貼心的在指引我們下一步應該怎麼做，並且依據步驟 1 的描述，我們可以來開始挑選喜歡的 theme 囉，也就是部落格佈景啦，不過這對選擇障礙的人來說，其實是最難的一個步驟呢 🥺

不過在挑選喜歡的主題之前，還是可以先來釐清一下專案架構：

```cli
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
```

大概可以來談幾個重要的部分：

- config.toml
	
	是一個檔案，看到 `config` 這個關鍵字就知道，相關的參數都是在這裡設置，不論是之後的 Domain 指向，網站 title，甚至是一些功能等等，都需要在這裡做配置，不過通常我們不用從零開始做，這邊下一個章節會談到。
	
	順帶一提，其實檔案格式也可以是 `.yaml` 或是我們熟悉的 `.json`，功能上完全相同，但大多數都會是 `toml` 就是了。
	
- archetypes

	可以看到這個資料夾包裹著 `default.md`，我們可以在 `default.md` 設置一些格式，讓我們在每次建立新文章時，就自動寫入這些內容，打開 `default.md` 可以看到以下內容，這就代表我們每次建立新文章時，都會將上述的設置帶到我們的文章之中。
	
	一句話簡單來說，就是模板文件。：
	
```md
---
title: "{{ replace .Name "\-" " " | title }}"
date: {{ .Date }}
draft: true
---
```
- layouts

	查詢資料為 .html 的模板，不過我這邊沒有用到，先不介紹。
	
- data

	如果模板文件需要取用資料，可以統一放在這裡，比如語系，不過我目前也沒有用到。
	
- content

	主要的文章與 `.md` 檔案的存放位置，內中的子資料夾會作為網站的路由，比如說在裡面新增 `about` 資料夾並建立 `index.html`，屆時就可以切換至 `/about` 這個 router 以下。
	
- static

	滿重要的一個資料夾，可以在內中存放 `favicon.icon` 等圖示，甚至是我們待會兒部署會用到的 `CNAME` 檔案
	
- themes 

	部落格的主題放置處，形式是以一種主題以一個資料夾為單位存放在內
	

上述我們會比較常處理的應該是 content、static 與 themes 三個資料夾，這邊先看過就好，之後的操作我們會再提到他們，現在，我們就先來建立第一篇文章，並挑選喜歡的主題吧！  

## 三、建立第一篇文章並挑選主題

### 建立文章

現在，我們可以開始來建立第一篇文章了。

儘管我們可以在 content 資料夾中新增 `.md` 檔案，但這邊還是建議使用以下方式來建立：

```cli
$ hugo new posts/first.md
```

順帶一提，你可能會看到有些使用方法會以 `post` 來命名而不是 `posts`，我是覺得都可以，當然這邊可能會根據你所使用的 theme 不同而調整，但也是可以自己修改的。

輸入上述指令之後，這個 `posts` 就會在我們剛剛的 content 資料夾底下。

同理，如果今天你輸入 `$ hugo new haha` 的話，那麼 content 底下也會有一個叫做 haha 的資料夾。

現在，切換到 `content/posts` 底下，開啟 `first.md`，可以看到裡面已經有一些內容，這些內容就是我們上面提到的 `archetypes/default.md` 中的模板。

可以留意 `draft: true` 這一個參數，代表目前這個文章是草稿狀態，之所以需要做這個參數設置的原因，是因為這篇文章你還沒寫完，還不希望別人看到，刪除這個參數或者設置為 `false`，就會被認為是 Published。

一但被認為是 Published，那屆時就會顯示在你的文章列表上，否則則不會出現。

### 挑選主題

現在到了大家最期待的環節了，不過在這邊我也花了最多時間，大概整整花了八個小時才找到自己喜歡的 theme 🥺

不過因為各個 theme 本身其實都是 GitHub repo，我們需要下 Git 相關指令將其下載到我們的 `themes` 資料夾，所以在這之前我們必須在根目錄安裝 `git` :

```cli
// 在 hugo-blog 根目錄底下
$ git init
```
安裝好後，別忘記要進行首次的 `git add .` 與 `git commit` 哦！

接著，我們可以到 Hugo 的官方網站底下的 [Themes](https://themes.gohugo.io/)，這裡存放著各式各樣的 Hugo 主題 (超感謝各種熱心的開發者！)，看上一個喜歡的主題之後，點選進去，可以看到有 Download 與 Demo 兩個按鈕，也可能會有 HomePage，代表作者為這個主題所做的介紹頁面。

而 Download 這個按鈕通常會連到這個主題的 GitHub repo 頁面，通常關於這個主題的設置方式與相關資訊通常會寫在 README.md 當中。

{{<figure 
    src="/images/2021/build-your-hugo-blog/02.png" 
    title="以 hello-friend-ng 這個 theme 為例" 
>}}

可以先點選 Demo，看一下這個主題大概長什麼樣子，並且玩看看有什麼功能，這裡真的可以說是精神時光屋啊。

順帶一提，之所以會說「功能」，是因為有某些主題僅僅只是 CSS style，而有些則是新增了一些有趣的互動等等，隨著功能的多寡，主題包的大小也各有不同，這邊就看個人喜好。

以 [hello-friend-ng](https://github.com/rhazdon/hugo-theme-hello-friend-ng) 這個主題為例，先點擊 Demo 進入 repo，複製 `.git` 網址，這邊介紹兩種安裝方式，不過建議使用第二種：

第一種，是切換至 themes 這個資料夾底下，然後 

```cli
$ git clone https://github.com/rhazdon/hugo-theme-hello-friend-ng.git
```

第二種，是在根目錄輸入以下指令：

```cli
$ git submodule add https://github.com/rhazdon/hugo-theme-hello-friend-ng.git themes/hello-friend-ng
```

不論使用哪一種方式，你的 themes 底下都會多出以這個主題為名的資料夾：

```cli
.
├── archetypes
│   └── default.md
├── config.toml
├── content
├── data
├── layouts
├── static
└── themes
	└── hello-friend-ng <-- 就是它 
```

這邊建議使用第二種方式，使用第二種方式之後，會自動在根目錄建立一個 `.gitmodules` 的檔案，

內容大概是像這樣的：

```.gitmodules
[submodule "themes/hello-friend-ng"]
	path = themes/hello-friend-ng
	url = https://github.com/rhazdon/hugo-theme-hello-friend-ng.git
```

至於為什麼要這樣做呢？

大家可以理解為，因為我們的根目錄已經有一個 `.git` 檔案在控制這個專案的版本了，當裡面的子資料夾 (也就是我們下載的主題) 同時也有一個 `.git` 時，就會形成版本控制下還有一個版本控制。

簡單來說，就是一個 repo 裡面還有一個 repo。

這時候就可以看作子 repo 是主 repo 的 Sub Module，Sub Module 本身若有更改，則需要在 Sub Module 本身進行 `add` 與 `commit` 等動作，再回到主要的 repo 進行 `add` 與 `commit` 才行。

如此，由於主題就是 Sub Module，哪一天若這個主題有更新功能時，我們就可以依照上述的邏輯來同時更新我們的部落格。

其實這邊自己在實務上也沒有遇過，如果上述有敘述不清或者錯誤，再麻煩 Twitter 敲我，不勝感激 🙏 🙏 🙏

順帶一提，如果你是使用第一種方式，將主題 `git clone` 在 `theme` 底下，那麼當回到根目錄執行 `git add` 來儲存這次變更時，也會有訊息跳出來，要你將該主題加入為 `gitsubmodule` ：

```cli
warning: adding embedded git repository: themes/hugo-theme-hello-friend-ng

hint: You've added another git repository inside your current repository.
hint: Clones of the outer repository will not contain the contents of
hint: the embedded repository and will not know how to obtain it.
hint: If you meant to add a submodule, use:
hint:
hint: 	git submodule add <url> themes/hugo-theme-hello-friend-ng
hint:
hint: If you added this path by mistake, you can remove it from the
hint: index with:
hint:
hint: 	git rm --cached themes/hugo-theme-hello-friend-ng
hint:
hint: See "git help submodule" for more information.
```

這時候按照上述步驟來處理，應該就沒有問題了，或者按照敘述執行 `git rm --cached themes/hugo-theme-hello-friend-ng -r` 後，再使用第二種方式下載主題。

### 主題設置

前面我們有提過 `config.toml` 這個檔案，關係到我們的部落格設置，但同時我們也提過，如果搭配主題使用的話，我們其實不太需要自己去設置細項，因為我們下載的眾多主題其實本身都有他們獨有的一些配置，我們只要複製一份，並修改部分可以客製的參數即可。

不管是哪一種主題，在其 repo 底下應該都會有一個 `exampleSite`，以 hello-friend-ng 這個主題為例，切換至 `/themes/hello-friend-ng` 可以看到 `exampleSite`。

切換至 `exampleSite` 底下之後，可以發現底下也有一個 `config.toml` 檔案，同時也可以看到一個 `content` 資料夾，以我這兩天的觀察，這個 `content` 資料夾所包含的 `.md` 檔案與 Section，通常都與主題的 Demo 相符合。

複製 `config.toml` 到根目錄，取代原本的 `config.toml` (如果是 `yaml` 或 `json` 也是如此)，然後依照主題的文件調適你想要的參數，通常 `config.toml` 中也會有註解，而文件部分則是看主題作者個人，以 [Anatole](https://github.com/lxndrblz/anatole/) 這個主題來說，就將文件很清楚的寫在 README.md，而也有某些主題自己架設了靜態網站當作說明書。

目前看來，其實每種主題配置同一項參數的功能不一樣，比如說瀏覽器標籤的 Icon，Anatole 主題的做法是可以在 `config.toml` 設置 `favicon.ico` 路徑，而 hello-friend-ng 則是會請使用者將是配各種瀏覽器裝置的 Icon 放在 static 資料夾中。

當然也是有一些不管哪種主題都會調整的參數，通常就是 `config.toml` 前面那幾項：

```toml
baseURL = "https://blog.claygao.tw"
title   = "Clay Gao"
languageCode = "en-us"
theme = "hello-friend-ng"

...
```

其中 `theme` 這個參數滿重要的，如果沒有設置的話，當 building 或是跑在本機 server 時就會需要另外帶參數 `-t` 或 `--theme` 來指定主題名稱才能跑起。

由於每個主題配置的參數都不太一樣，這邊就留給讀者自己去找到喜歡的主題來研究，不過當然了，功能越簡單的主題，配置通常也越少就是了。

## 跑起來！產生部落格

終於到了關鍵的時刻了，現在就讓我們把部落格跑起來吧！

如果是第一次接觸相關應用的新手，可以先了解兩個概念，第一是在測試環境上運行，第二則是正式打 build 出我們的網站。

為了避免 build 出來的成品不如我們的預期，其實我們可以先在測試環境上面跑，在根目錄輸入：

```cli
$ hugo server -D
```

這時候會出現一串描述，代表你可以在相對應的 port 上跑起你的網站，比如照以下描述，我的測試 port 就是 `localhost:59397`，所以只要打開瀏覽器，在網址列輸入就可以看到我們的部落格了！

{{<figure 
    src="/images/2021/build-your-hugo-blog/03.png" 
    title="反白的就是我們的測試 port，趕緊複製貼上網址列吧！" 
>}}

另外，也可以看到 Hugo 列出了生產了哪些類型的檔案：

```cli
                   | EN
-------------------+-----
  Pages            | 24
  Paginator pages  |  0
  Non-page files   |  0
  Static files     | 27
  Processed images |  0
  Aliases          | 12
  Sitemaps         |  1
  Cleaned          |  0
```

> 在看部落格之前先提醒幾點，因為我們已經有在 config.toml 加上參數 `theme = [我們要的主題]`，如果你原本沒有帶上這個參數，那麼在執行 `hugo server` 或是 `hugo` 時可能會有問題，這時候就會需要加上參數 `--theme=主題名稱` 或是 `-t 主題名稱` (注意 -t 與主題名稱中間有空格)，才能正常跑起。

在網址列貼上剛剛的 `localhost:port 號` 之後，就可以看到部落格啦！大多數的人應該都是 `localhost:1313` 這個 url，不過也有可能出現別的 port，所以如果跑起來是 404，記得檢查一下 Terminal 給的資訊哦！

{{<figure 
    src="/images/2021/build-your-hugo-blog/04.png" 
    title="跑起來囉，以這個範例來說，port 是 59397" 
>}}

> 然後順便提一下 `-D` 這個參數，前面有提到，在文章標頭會有 `draft: true`，當帶入 `-D` 後，測試環境會將性質為草稿的文章也列出在文章列表，做編輯對應之用。

另外，當我們的部落格跑起來後，由於支援 Hot rendering，只要編輯文章並儲存，就可以看到網站內容會自動刷新，不用再按 F5 重新整理囉！

現在，你可以開始編輯自己的文章，或者按照之前說的使用 `hugo new` 產生一篇新的文章，也可以在自己在 context 資料夾加入你想要的子資料夾，並搭配參數豐富你的網站選項，這邊就讓讀者自己玩看看囉！因為根據不同的主題，也會有不同的效果，故不一言以蔽之。

等到玩得差不多之後，就可以正式產生，也就是 build 出我們的成品了，只要簡單輸入：

```cli
hugo
```

沒錯，就是這麼簡單，這時候應該 Terminal 中應該也會告訴你生產了哪些靜態檔案，這些靜態檔案會放在一個在根目錄新產生的資料夾 `public` 之中，`public` 裡面會有一個 `index.html`，他就是我們的成品啦，整個網站的入口點！

另外你應該也有發現到，輸入 `hugo` 到產生 `public`，時間也是超爆快的，這也是 Hugo 強大的地方。

如果你有點好奇心，也可以探索 `public` 這個資料夾內的內容，應該也可以注意到我們在編輯的 `.md` 文章也已經被轉化成資料夾與各自的 `index.html` 了，就是各種靜態網頁的概念。

## Build 了，然後呢？談 GitHub Page 部署第一種部署方式

當然是把我們的網站部署在線上，讓親朋好友一起瀏覽囉！

如果你已經有架設過 AWS / GCP 等雲端伺服器的經驗，那應該不用我多做說明了，因為是靜態網頁，所以只要簡單把 build 出來的內容放上去就好。

不過這邊提供一個大多數人會選擇的方案，那就是使用 GitHub Page，GitHub Page 允許我們將專案中的 index.html 呈現在與我們用戶名稱相同的 github.io domain name 上，而且完全免費，只是要注專案本身必須是公開的 (Public)，而不能是私有的 (Private)
  
> 由於這邊對 GitHub Page 研究沒有太多，所以關於私有無法運行的敘述，如果有錯誤再煩請推特聯絡我，我還沒裝留言系統 (也可能不會裝)

首先，我們創造一個新的專案 repo，名為 `[用戶名].github.io`，用戶名就是你的 GitHub 用戶名，比如說我的用戶名是 `claygao`，創建的專案名稱就是 `claygao.github.io`，記得要設立成公開的哦！

當然如果想另外自己設計路由，而不想叫做 `[用戶名].github.io` 應該也是 OK，不過這邊就先不舉例了。

**注意，以下是第一種部署方式，本人建議使用第二種，所以以下建議先看過了解原理就好，實作上採取之後第二種方式**

開好 repo 之後，repo 還是空的，這時候回到我們的本地部落格資料夾根目錄，進入剛剛 build 成的 public 資料夾，我們現在要將該資料夾的所有資料都上到剛剛創立的 `github.io` repo 上。

切換到 `public`，輸入 `git init`，`git add` 並且做好初次 `git commit` 之後，將剛剛的遠端 repo 與本地 public 牽上線。

牽上線很簡單，以我的例子來說，只要輸入：

```cli
$ git remote add origin https://github.com/ClayGao/claygao.github.io.git
```

> repo 的主頁可以快速複製網址

牽上線後，再把本地資料 push 到遠端 repo：

```
$ git push origin master
```

push 成功後，遠端 repo 已經有資料了，這時候讓我們點選上排的 `Settings` 標籤頁：

{{<figure 
    src="/images/2021/build-your-hugo-blog/05.png" 
    title="點選 Settings" 
>}}

點選之後，拉到最下面，觀看 GitHub Pages 的部分，確定 Branch 的地方選擇 master，並按下 Save

{{<figure 
    src="/images/2021/build-your-hugo-blog/06.png" 
    title="確定 Branch 的地方選擇 master" 
>}}

然後，你就可以在網址列輸入 `https://[用戶名].github.io`，看到你的網頁囉！以我麼案例來說的話，我的部落格網址就會是 `https://claygao.github.io`

另外如果沒有成功看到部落格，可以再稍等一下，應該十五分鐘之內會成功，如果沒有成功，可能要看一下是不是遺漏掉什麼步驟，或者是 repo 根目錄中沒有 `index.html` 這個檔案，這可能是 `config.toml` 遺漏了一些參數沒在 `hugo` 時補上就會出現這種狀況。

這樣感覺好像就成功了，但其實我們還有兩個部分要優化。

第一個部分就是 `https://[用戶名].github.io` 這個網址實在不太美觀，也不夠個人化，我們能否客製化成自己的網域了？當然沒問題！

第二個部分就是，這樣感覺每次修改完文章，我都必須要重新執行 `hugo` 這個指令來產生更新過後的 `public`，然後每次又都要在 `public` 裡面 commit 變更，然後再 push 到遠端 repo 上，感覺超麻煩。

所以接下來，我們就要來完成上述兩件事情囉！就先從建立自己的網域開始吧！




