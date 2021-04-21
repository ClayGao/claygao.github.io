---
title: 簡單理解什麼是 JavaScript Engine 與 AST
date: "2021-03-29-24T22:40:32.169Z"
template: "post"
draft: true
slug: "JavaScript-Engine-and-AST"
category: "JavaScript"
tags:
  - "JavaScript"
  - "Web Development"
  - "AST"
  - "eslint"
description: "講解何謂編譯和直譯，並且帶到 V8 與 Eslint"
socialImage: "/media/42-line-bible.jpg"
---

## 前言https://v8.dev/blog/lazy-unlinking

研究緣由其實也沒什麼特別的原因，主要就是開發時仍然不懂瀏覽器底層更詳細的運作方式，在工作與自身開發時總有一種見樹不見林的感覺，趁最近時間比較多，想把查到的資料筆記並整理一番，僅此而已。

主要架構就是和標題一樣，研究 JavaScript Engine 與 AST，這邊會帶大家來看平常我們開發時，JavaScript Engine 大致運作上的方式，然後帶到 V8，最後則是再回頭來講 AST 的部分。

不過在開始之前，需要先從最基本的概念，也就是編譯和直譯開始講解。

## 編譯的 (Compiled) 與直譯的 (interpreted)

很多的教學與概念文章，都會在編譯式與直譯式後面加上「語言」，比如說 C 是屬於編譯式語言，JavaScript 是屬於直譯式語言，然後，然後就沒有然後了

其實，比起去記什麼語言是屬於編譯還是直譯，更重要的是要理解「編譯」與「直譯」的基本概念，然後再用這個概念回頭思考自己所使用的語言，才能真正融會貫通。

這邊會介紹最近看到一個很棒的解釋，並融合一些自己的話語：

### 編譯 (Compiled)

編譯的意思，就是**先把我們平常在寫的 code，變成另外一種具體的文本形式**，也就是另一種語言。

你可以這樣理解：我們有一位「文本翻譯員」，將一本用中文寫的書，翻譯並另外抄寫成另一本英文書，然後再拿給懂英文的人看。

而這位「文本翻譯員」，在程式的世界中，我們稱呼他為「編譯器」(Compiler)。

翻譯成另外一種文本之後，因為已經是目標物看得懂的語言了 (比如說懂英文的人直接看已翻譯成英文的書籍)，所以在「執行上」就會比較快速。

另一種概念則是直譯

### 直譯 (interpreted)

直譯的意思，就是直接在執行我們平常在寫的 code 時，**一邊執行一邊翻譯**。

用類似上述舉例的方式再理解的話，可以形容為：我們現在有一位「口説翻譯員」，將一本中文的書籍，直接一邊看過，一邊翻譯給懂英文的人聽。

這位 「口説翻譯員」在程式的世界中，我們稱呼他為「直譯器」(Interpreter)。

直譯在執行上會比較慢一點，畢竟是一個一邊翻譯，一邊執行的過程。

### 各自差異

先大致了解上述的概念後，我們才去定義：

- 需要先經過編譯才能夠執行的語言，稱為編譯式語言。
- 若其撰寫出的程式可以一邊執行一邊直譯的，則稱為直譯式語言。
  
但這不是絕對的，下面會提到。

大概整理出幾點差異：

- 在執行時，編譯式會比直譯式來得更快速，前者因為已經編譯過了，不用像直譯需要一邊執行一邊翻譯
- 在靈活性上，編譯式會比直譯式來得更不靈活，比如說編譯式沒辦法開發到一半就馬上執行，而後者可以直接執行除錯
- 在獨立性上，編譯式語言的程式碼可以獨立執行，而直譯式則需要一個「執行環境」，比如說在 node 與瀏覽器中各自執行 JavaScript，可用的 API 就會有所差異
- 編譯式語言大多為「靜態語言」，也就是「強型別語言」，直譯式語言則多為「動態語言」，也就是「弱型別語言」

關於最後一點，一個很主要的關鍵是前者需要在開發時就先宣告好型別，宣告好之後就無法再轉型

但直譯式語言不是，我可以先宣告 `let a = 'string';`，然後下一秒再寫 `a = 123` 而不會報錯

而關於靈活性，自己的理解是，編譯式語言畢竟是得先編譯整份程式，而你編譯出來的東西可能也是特定的機器才能看懂，故有其依賴性

相反的，直譯式則是可以依照自己身處的執行環境，去選擇自己要去直譯成哪一種語言

就如維基百科淺顯易懂的介紹：

> 一般而言，用編譯語言寫成的程式，在執行期的執行速度，通常比用直譯語言寫的程式快。因為程式在編譯期，已經被預先編譯成機器碼，可以直接執行，不用像直譯語言一樣，還要多一道直譯程式。

> 但是要先進行編譯，之後才能執行程式，這也造成了編譯語言的缺點。一般而言，編譯語言的程式開發速度，以及除錯時間，都是比較長的。因為它不像直譯語言可以寫完一行，或一小段程式之後，馬上執行，馬上除錯。直譯語言通常讓程式開發的整體時間變少，在開發過程中，程式師也可以更彈性、快速的測試自己的想法。

不過儘管編譯式語言在執行上比直譯式快，但在整體開發、靈活性與除錯性種種因素考慮之下，直譯式語言的優點反而突現了出來。隨時可執行，隨時可除錯的特點，這點在寫 JavaScript 的開發者都不陌生。

再次整理一下：

|特性  | **編譯式 (Compiled)** | **直譯式(Interpreted)** |
|:----------------|:------------------|:-------------|
|概覽|編輯 -> 編譯 -> 執行 -> 除錯|編輯 -> 執行 & 直譯 -> 除錯|
|靈活性|低|高|
|型別強度|強|弱|
|獨立性|大多不需執行環境|大多需要執行環境|

所以現在再去回想 JavaScript 的特性，其實就很清楚了，與上述關於直譯式語言的描述多有符合。

總而言之，自己去思考在開發什麼語言時，用的是什麼樣的開發方式與執行過程，比起去死記編譯式與直譯式語言還要快速且有效得多。

關於編譯和直譯的介紹，就先講到這裡，接下來來談談 JavaScript。

---

## 直譯式語言的繼續探討

接下來，我們繼續來看看直譯式語言本身可能產生的問題：

### 直譯式語言的痛點

從上一段我們可以知道，在**執行階段**，直譯式語言在執行上會比較慢，因為他執行到哪一行，就直譯到哪一行，如果還是不好理解，可以看看以下這個例子：

```javascript{numberLines: true}
for (i = 0; i < 1000; i++) {
    sum += 1;
}
```

如果是編譯式語言，因為會先經過編譯了，所以在執行時，大概就是單單執行以下的內容：

```js
sum += 1
sum += 1
sum += 1
sum += 1
sum += 1
sum += 1
sum += 1
...
```

那如果是直譯式語言呢？

```js
// 編譯
sum += 1

// 編譯
sum += 1

// 編譯
sum += 1

// 編譯
sum += 1

// 編譯
sum += 1

// 編譯
sum += 1

// 編譯
sum += 1
...
```

因為是每執行一次，就編譯一次，所以既耗時也耗能。

除了上面這個問題以外，直譯式語言的編譯是交給直譯器 (Interpreter)，也就是上述的口說翻譯員，所以你在什麼樣的環境中執行，那麼那個環境本身就應該要有個直譯器，這也相當合理。

關於直譯的部分，就先講到這邊，現在請先容許我來介紹一下 JavaScript 引擎

---

## JavaScript 引擎？

或許你很少聽過什麼具名的直譯器，但我想你一定有聽過 JavaScript Engine，其中更常聽到的應該是 Google 所開發的 V8，不過除了 V8 以外，還有以下幾種 JS 引擎：

- SpiderMonkey (Mozilla Firefox)
- JavaScriptCore (Apple Safari)
- Chakra (Microsoft IE)

從上列可以看到，不同的引擎，其實也對應到不同的瀏覽器，以 SpiderMonkey 這個引擎來說，就屬 Mozilla 的 Firefox 瀏覽器所使用。而 V8 引擎大家應該都知道是屬於 Chromium 相關應用 (如 Google Chrome and Election) 或 Node.js 執行環境所使用。

那麼，引擎本身就是直譯器嗎？並不完全算是，直譯器本身應該説是 JavaScript 引擎中的其中一個環節。以下就簡單講一下當 JavaScript 被執行時，JS 引擎幫我們做了什麼樣的事。

下圖是大多 JS 引擎執行 JavaScript 程式碼時，所經歷的幾個階段：

![JS Engine](/media/20200330/01.png)
(圖片取自 [淺淡 JS Engine 機制](https://medium.com/walkout/%E6%B7%BA%E6%B7%A1-js-engine-%E6%A9%9F%E5%88%B6-77391b4dd3db))

現在，讓我們拆成兩個部分來看：

### 1. 從 Source Code 到 AST

![Parser](/media/20200330/02.png)

首先，**Parser** 會將 JavaScript 程式碼轉為 Abstract Syntax Tree，也就是俗稱的 **AST**，中文為**抽象語法樹**，將我們的 JavaScript 做詞法和語義分析，一些 Plugin，比如說 Babel，Eslint 相關的 Plugin，就是藉由 AST 來做相對應的處理。

舉例來說，Babel 在這個階段，可以針對節點去做語法修改 (比如說 ES6 的 arrow function 轉換為 `function`)，而 ESLint Plugin 在這個時候會針對每個節點去做規則檢測。

### 2. Interpreter - 直譯器登場，不過怎麼有 Compiler?

![Interpreter](/media/20200330/03.png)

AST 經由 **Interpreter** 轉換為 Bytecode 並且執行，不過有一個很酷的地方，那就是當 JS 引擎發現某個函式被重複執行時，會將這段 Bytecode 會被傳至 Optimizing Compiler 中做優化，類似一種快取 (cache) 的概念。

這種優化手段，稱為 [just-in-time compilation](https://zh.wikipedia.org/wiki/%E5%8D%B3%E6%99%82%E7%B7%A8%E8%AD%AF)，簡稱為 JIT。JIT 綜合了編譯與直譯的優點，這樣的技術目前已經也已經被廣泛使，所以 JIT 並非 V8 - 或是 JS 引擎所特有，事實上這樣的概念在 Java 程式設計中也有出現，這裡就不多做贅述了。

那 V8 呢？其實 V8 大致上的架構和上述差不多，可以看一下以下這張圖：

![Interpreter](/media/20200330/04.png)

JavaScript Source Code 到 AST 的流程其實是差不多的，只是 Interpreter 在 V8 中被稱為 **Ignition**，Compiler 被稱為 **TurboFan**，不過這邊還是講解一下流程：

Ignition 除了將 AST 轉化為 bytecode 之外，也負責收集資訊給 TurboFan。比如説，當 Ignition 察覺到函式被執行了 1 次以上，那麼這個資訊就會被傳至 TurboFan 產出最佳化 Machine Code 做快取 (這段程式碼被稱作 HotSpot)。反之，如果函式只執行了一次 (這邊我理解為第一次時)，Ignition 會直接編譯成 Bytecode，而不會交予 TurboFan。從這一點可以看出，Ignition 除了編譯以外，本身還會收集執行相關的訊息，以判斷要不要交給 TurboFan 做優化處理。

至於圖中的紅線，其實是一種特殊的狀況，就是當同一個函式在重複執行過程時被帶入了不同的參數，且這個參數與原本處於快取機制所假想的型別不同時，就會由 De-optimize 返回 Bytecode 自身來執行。

比如說前三次都執行了 `foo(1,2)`，而最後一次執行了 `foo('bar', 'baz')` 這種狀況，因為這與原本 Optimized Machine Code 定義 `foo` 只帶入 `Number` 的假想不同，故只能重回上一個階段。這種的現象就稱為 **De-optimization**。

```js{5}{numberLines: true}
foo(1, 2);
foo(1, 2);
foo(1, 2);

foo('bar', 'baz'); // 這次呼叫函式，參數從 Number 變為 String 
```

V8 的介紹就先簡單到這邊，事實上，V8 Engine 的內容探討還相當龐大，這邊提出的僅僅是冰山一角，這邊就留待以後有機會再慢慢介紹。

總結來說，因為 JS 引擎本身有直譯器，又有 JIT 機制上的實作所以包含了 Compiler，所以會讓人覺得怎麼 JavaScript 在執行過程中個別經歷了編譯與直譯兩種階段，不過以現代的開發應用上，JavaScript 應該仍算是直譯式語言沒錯。

但其實到這邊我還是覺得有點奇怪，於是就想要尋求一些比較公信力的資料，不斷尋覓之後，直到自己在 Wiki 上看到了以下這一段敘述，才豁然開朗：

In computer science, an interpreter is a computer program that directly executes instructions written in a programming or scripting language, without requiring them previously to have been compiled into a machine language program. An interpreter generally uses one of the following strategies for program execution:

1. Parse the source code and perform its behavior directly;
2. Translate source code into some efficient intermediate representation and immediately execute this;
3. **Explicitly execute stored precompiled code made by a compiler which is part of the interpreter system.**

回到稍早對於直譯式與編譯式語言的定義，其實仔細想想應該是沒有絕對的。語言就是語言，我們定義一個語言是直譯或是編譯，其實應該是在描述在「大多數」**執行**情況下，而不是這個語言本身才對。今天如果你功力深厚，搞不好也可以寫一款專門為 JavaScript 專用的編譯器或是專門為 C++ 專用的直譯器 (其實這東西也已經有了)。

如果真的要類比的話，就像你明明知道蘿蔔很常被拿來煮湯，但你不會說蘿蔔只能拿來煮湯是一樣的意思，一切就是看你想怎樣料理，事在人為嘛。

## 從 Source Code 到 AST 是不是少講了什麼 ?

現在，回歸到 JS Engine 一開始的執行步驟，也就是 Source Code 轉換為 AST 的階段，這階段假若還要再拆分，則會是以下幾個步驟：

1. Lexical Analyzer (詞法分析)
2. Syntactic Analysis (語法分析)
3. Semantic Analysis (語義分析)

這邊不會講得太細，因為網路上太多資料了，這邊用一個簡單的方式來比喻：

在第一個詞法分析階段，就是將你的程式碼拆成一些比較小的單位，這些單位若要更精確的描述，則可以稱之為各個 token。

舉白話來比喻，比如說 `小明是一個小淘氣` 這一句，大概就會拆成：`小明`、`是`，`一個`，`小淘氣` 這樣。



如果真的要描述的話，可以理解

<!-- 沒錯，就是在 Source Code 經由 Parser 解析為 AST 的過程之中，在上述的流程圖中，只有簡單講到 -->

在看完 [這篇](https://zhuanlan.zhihu.com/p/46993552) 文章之後，才清楚明白到，其實 V8 引擎做的就是在 Call Stack 中 function call pop up 出去之後的部分

看完 [這篇](https://medium.com/edge-coders/you-dont-know-node-6515a658a1ed) 知道 Call Stack 是 V8 的一部分

參考資料：

- [JavaScript深入浅出第4课：V8引擎是如何工作的？](https://cloud.tencent.com/developer/article/1463973)
- [一點都不深入的了解 Compiler、 Interpreter 和 VM](https://www.spreered.com/compiler_for_dummies/)
- [JavaScript 引擎有幾種？](https://www.html.cn/qa/javascript/18486.html)
- [淺淡 JS Engine 機制](https://medium.com/walkout/%E6%B7%BA%E6%B7%A1-js-engine-%E6%A9%9F%E5%88%B6-77391b4dd3db)
- [JavaScript engine fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)
- [JavaScript 引擎原理：外形与内联缓存](https://www.yexiaochen.com/%E3%80%90%E8%AF%91%E3%80%91JavaScript-engine-fundamentals-Shapes-and-Inline-Caches/)
- [JavaScript 工作原理](https://lyn-ho.github.io/posts/b2f2d94f/)