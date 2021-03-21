---
title: 理解基礎的 JavaScript Prototype
date: "2021-03-24T22:40:32.169Z"
template: "post"
draft: false
slug: "JavaScript-Prototype"
category: "JavaScript"
tags:
  - "JavaScript"
  - "Web Development"
description: "講解基本的 ProtoType，幫助新手能夠更快理解"
socialImage: "/media/42-line-bible.jpg"
---

## 前言

之前在學習 Prototype 的時候算是斷斷續續在理解的，怎麼說呢 ? 一開始是在上課的時候學習，那時候懂了一半，而接續兩次作業之後又認為自己懂了，這是第二次以為自己理解。
直到最近準備面試題目，學習 JavaScript 內的 OOP 時，發現對於 Prototype 的理解還是不太記得，於是就想說這次一氣呵成整理一下。
因為是複習，所以不算非常全面，若有錯誤也希望能夠被指正，這樣的機會是可遇不可求的。

---

## 一切的一切，都從 JavaScript 中的物件導向開始

什麼是物件導向 ? 我相信你已經爬過一些文章了，這些文章大多用狗和汽車來做舉例，也就是我有一張設計圖，然後利用這張設計圖來創造一些實體 (instance)，在 JavaScript ES5 中，看起來會是這個樣子：

```js
function Car(color) {
    this.color = color
}
Car.prototype.getName = function() {
    return this.color
}


const ford = new Car("red")
console.log(ford.color) // red
console.log(ford.getName()) // red
```

都一樣印出 red，但不要誤會了，第一個 `red` 是因為我們利用 `Car` 這個建構子 (中國翻作構造函式)，實作出了 `ford` 這個物件，然後我們去印出 `ford.color` 這個屬性。而第二個 `red` 是我利用 prototype 定義了 `getName` 這個函式來回傳 instance 的 `color` 屬性。

這也是我第一次看到 prototype，從上面的例子理解的話，它就有點像是一個設定檔，我可以添加一個方法，然後來做我想要做的事情，如果就使用上來講，了解到這樣的確就足夠了，但感覺好像還是哪裏怪怪的。

---

我寫過幾萬次的 function 了，怎麼都沒察覺 prototype 這個東西 ?
我再貼一次剛剛的例子 :

```js
function Car(color) {
 this.color = color
}
Car.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")
console.log(ford.color) // red
console.log(ford.getName()) // red
```

先假設你不知道如何在 JavaScript 中實作 OOP，那麼 `Car` 在你眼中看來就不過是一個普通的 Function 而已。
另外就是 Car 第一個字母是大寫，也不會改變它是一個 Function 的事實，裡面的 `this` 也一樣，不會影響到這個 Function 的本質。
好，現在請你恢復記憶，想起我們現在在講 OOP，一切都沒有改變，`Car` 之所以第一個字母大寫，是一個做為建構子的習慣用法，`this` 指的則是 instance，大概就是這樣，`Car` 依舊是一個函式，它仍然作為一個身為 Function 的職責。
那麼既然 `Car` 依然是一個普通的 Function，但它可以使用 `prototype` 來定義方法，那我們是不是可以推論，其實 Function 都帶有 prototype ?
為了驗證上述說法，我們寫一段 Code :

```js
function Car(color) {
 this.color = color
}
function test() {
    console.log('ya')
}
console.log(Car.prototype) // Car {}
console.log(test.prototype) // test {}
```

這邊我宣告了一個普通函式叫做 `test`，然後我印出兩者的 prototype，可以看到分別是 `Car { }` 與 `test { }` 兩個物件，為了驗證 prototype 就是我印出的這個 `{ }`，我幫他們的 prototype 都加上了新的屬性 and 方法：

```js
function Car(color) {
 this.color = color
}
function test() {
    console.log('ya')
}
// 加入新方法
Car.prototype.getName = function() {
    return this.color
}
// 加入新屬性
test.prototype.sayHi = 'Hi'
// { } 內有東西了

console.log(Car.prototype) // Car { getName: [Function] }
console.log(test.prototype) // test { sayHi: 'Hi' }
```

大家在學習最基礎的物件時，應該都記得如果要幫物件增加屬性的話，直接用 . 運算子就好，所以其實 `Car.prototype.getName` 與 `test.prototype.sayHi` 也是在做一樣的事情，既然針對 `prototype` 增加屬性會得到 `{ getName: [Function] }` 與 `{ sayHi: 'Hi' }` 的結果，那我們也可以明白，一開始印出的 `{ }` 就是 Prototype

也就是說 Prototype 本身就是個物件，而且宣告函式時會自動附帶，跟我這個函式要不要作為建構子一點關係都沒有，那之所以過去宣告 Function 時都沒有用到，是因為我們沒有利用 Function 作為建構子，也就是說，我們沒有 new 出 instance，所以在使用時機上就不太用得到。
但其實你早就接觸過了。

---

## `__proto__` 與 `Object.getPrototypeOf()`

要看一個 instance 是會找向哪一個 Prototype，用 `__proto__` 就可以看出來了，用我們最初的範例來舉例：

```js
function Car(color) {
 this.color = color
}
Car.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")
console.log(Car.prototype) // Car { getName: [Function] }
// __proto__
console.log(ford.__proto__) // Car { getName: [Function] }
console.log(ford.__proto__ === Car.prototype) // true

```

而比起 `__proto__` ，其實更推薦使用 `Object.getPrototypeOf()` 這個語法，它會 return 該 instance 指向的 Prototype，這個用法感覺也比較技術性：

```js
function Car(color) {
 this.color = color
}
Car.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")
console.log(Car.prototype) // Car { getName: [Function] }
// 使用 Object.getPrototypeOf(instance)
console.log(Object.getPrototypeOf(ford)) // Car { getName: [Function] }
console.log(Object.getPrototypeOf(ford) === Car.prototype) // true

```

那為什麼會提 `__proto__` ? 如果用白話來表示，我會習慣稱呼它為「該物件所指向的原型」，「原型」指的就是 Prototype， 因為 `ford` 是 new 自於 `Car` 的，所以 `ford.__proto__` 就會指向 `Car.prototype`，也就是都指向同一個記憶體位置。
既然物件可以用 `__proto__` 來找到它的原型，那我們來試試看下列幾個：

```js
function Car(color) {
 this.color = color
}
Car.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")

console.log(Car.prototype.__proto__)  // { }
console.log(Car.prototype.__proto__ === Object.prototype) // true
console.log(Car.prototype.__proto__.__proto__) // null 
console.log(Car.__proto__) // [Function]
console.log(Car.__proto__ === Function.prototype) // true
console.log(Car.__proto__.__proto__) // { }
console.log(Car.__proto__.__proto__.__proto__) // null
```

上面分別對 `Car.prototype` 與 Car 做了 `__proto__` 的探測，比對結果簡述以下：
首先，`Car.prototype` 是一個物件，這是我們剛剛得到的結論，這個物件的存在表面上與 `function Car() { }` 本身沒有關係，在程式碼中他們是不同的兩個部分 (我當初也以為 `Car.prototype` 這一段是把 function 中的程式碼寫到外面，以為是 function 的一部分，這種想法是錯的)

但是在底層，`Car.prototype` 就像是顆衛星一樣環繞在 `Car` 身邊，這點是沒錯，畢竟我們剛剛也說了，宣告函式時會自帶一個 `prototype`。
接下來一一講解 :

```js
console.log(Car.prototype.__proto__)  // { }
```

`Car.prototype.__proto__` 就是 Car.prototype 所指向的原型，因為是物件，所以印出 `{ }`

```js
console.log(Car.prototype.__proto__ === Object.prototype) // true
```

`Car.prototype.__proto__` 與 `Object.prototype` 指向同一個記憶體位置，因為剛剛說過了，它是物件嘛，所以源自於物件

```js
console.log(Car.prototype.__proto__.__proto__) // null
```

之所以回傳 `null`，代表再往上找已經沒有東西了，可以確定 `Object.prototype` 就是最頂層無誤。

```js
console.log(Car.__proto__) // [Function]
```

`Car.__proto__` 就是 `Car` 所指向的原型，因為是函式，所以印出 `[Function]`

```js
console.log(Car.__proto__ === Function.prototype) // true
```

`Car.__proto__` 與 `Function.prototype` 指向同一個記憶體位置，前面也說過了，它是函式嘛

```js
console.log(Car.__proto__.__proto__) // { }
```

那為什麼 `Car.__proto__.__proto__` - 也就是 `Function.prototype.__proto__` 是印出 `{ }` ? 因為 Prototype 本身是物件，所以源自於物件無誤。

所以接下來的 `Car.__proto__.__proto__.__proto__` 已經找到了最頂層以外了，所以回傳 `null`：

```js
console.log(Car.__proto__.__proto__.__proto__) // null
```

等等，你有沒有發現察覺到了什麼 ?

如果在最初的例子，`ford` 是 `Car` 的 instance，所以 `ford.__proto__` 指涉到了 `Car.prototype`，那麼回過頭來說，`Car.__proto__` 之所以會指涉到 `Function.prototype`，是不是代表我宣告 `Car` 這個函式時，也等於是我 new 了一個新函式 ?

換句話說，`Car` 是 Function 的 instance 嗎 ?

不只是函式，讓我們也針對物件(Object) 與陣列(Array) 實驗看看：

```js
// new 一個 Function
var count = new Function('a','b','return a + b')
// new 一個 Object
var objA = new Object()
var objB = { a: 'a', b:'b' } // 一般宣告物件方式
objA.a = 'a'
objA.b = 'b'
// new 一個 Array
var arrA = new Array()
var arrB = [1,2] // 一般宣告陣列方式
arrA.push(1)
arrA.push(2)
console.log(count(3,4)) // 7
console.log(objA) // a: 'a', b: 'b'
console.log(objB) // a: 'a', b: 'b'
console.log(arrA) // [1, 2]
console.log(arrB) // [1, 2]
```

上面我 new 了一個 Function 並代入了一些參數，最後一個參數是函式內容，所以實際上與我們一般宣告函式無異。(這是宣告函式的其中一種方法)
物件的部分我也是 new 了一個 Object，並利用 . 加入屬性，得到的結果與下面一般宣告物件方式相同。
陣列的部分與物件差不多，就不多做說明。
由上述可以了解，我們一般宣告函式、物件與陣列，實際上都是 new 一個新的 instance，所以 JavaScript 的 OOP 實現方式，一直以來我們早就在做了，只是我們沒有察覺而已，所以前面才說我們其實早就接觸過了。
有了以上的緣故，我們就可以理解為什麼 `Car.__proto__` 會指向 `Function.prototype` 了。

---

## 一起來拆解 new

上一段的最後一句，寫道 `Car.__proto__` 會指向 `Function.prototype` ，但我比較習慣稱兩者是指向同一個記憶體位置，原因是因為 new 本身做了一些事情。

現在先假設我們不知道有 new，然後我寫一些 code，以模擬讓 Car 這個函式實體化一個 instance

```js
function Car(color) {
 this.color = color
}
Car.prototype.getColor = function() {
    console.log(this.color)
}
function newCar(color) {
    var obj = {} // 建立一個空物件 
    obj.__proto__ = Car.prototype // 指派空物件的 __proto__ 為 Car.prototype 
    Car.call(obj, color) // 傳入空物件為 this
    return obj // 回傳該物件
}
var ford = newCar('black')
ford.getColor() // black
```

首先會建立一個空物件
並將 `Car.prototype` 指派給該物件的 `__proto__` ，使兩者指向相同
之後利用 `.call` 將空物件傳入 Car 裡面的 this 並指定了 `color` (也就是 `obj.color`)，並指派了參數 `color` 為值
最後回傳這個 obj

以上大概就是 new 做的事情，由於最後是回傳物件，所以 ford 也是一個物件沒錯。
而第二步的 `obj.__proto__ = Car.prototype`，也證明了兩者是指向同一個記憶體位置沒錯，當然你也可以說 `obj.__proto__` 指向 `Car.prototype`，這並不衝突。

---

## 原型鍊總結 (Prototype Chain)

最後，我們再重溫一下一開始的範例

```js
function Car(color) {
 this.color = color
}
Car.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")
console.log(ford.color) // red
console.log(ford.getName()) // red
```

`ford` 本身沒有 `getName` 這個函式，但你知道為什麼可以這樣呼叫，因為 new 幫我們做了一切，如果不使用 new， `Car` 本身就只是一個不回傳值的普通函式而已，`Car.prototype` 也沒有上場的機會。

假設 `Car.prototype` 自已也沒有 `getName`，但我若寫在更上層的 `Object.prototype.getName = function ...`，那麼 `ford` 也可以存取得到，因為 `ford` 會透過 `__proto__` 繼續往上查找，我們在稍早也模擬過這個過程。

```js
function Car(color) {
 this.color = color
}
// 設在 Object.prototype
Object.prototype.getName = function() {
 return this.color
}
const ford = new Car("red")
console.log(Car.prototype.__proto__ === Object.prototype) // true
console.log(ford.getName()) // red
```

透過 `ford.__proto__` 往上查找 `Car.prototype` 發現沒有 `getName`，於是又透過 `Car.prototype.__proto__` 往上查找 `Object.prototype`，終於找到了 `getName` 這個方法並執行，這樣的查找路線，我們就稱之為原型鍊 ( Prototype Chain )

---

參考資料：

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
- [桑莫的 Prototype 介紹](https://cythilya.github.io/2018/10/26/prototype/)
- [該來理解 JavaScript 原形鍊了](https://blog.techbridge.cc/2017/04/22/javascript-prototype/)
