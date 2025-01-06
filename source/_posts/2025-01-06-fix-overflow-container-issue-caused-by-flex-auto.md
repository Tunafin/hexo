---
title: "flex-auto 造成 overflow 可滾動元素跑版的解決方法"
date: 2025-01-06 23:29:11
updated:
categories:
tags: HTML CSS
---

<div style="width: 100%; max-width: 600px; margin: 10px 0;">
  <img src="https://i.imgur.com/qOXlcRS.png">
</div>

在使用 CSS 建立 flex 彈性佈局時，經常遇到一個情境：父容器大小固定的情況下，我們會讓其中某個子元素使用`flex: auto`來讓其填滿父容器的空間，並給這個子元素添加`overflow: auto`，讓該元素過長時能顯示 scrollbar 而不會超出容器範圍。

以上是父容器大小固定的情況下運作正常，但如果這個父容器其實也是另一個flex容器的子元素且被設為`flex:auto`，那麼結果界與預期不同了...

<!-- more -->

---

先來看簡單的情境，下面範例的黑框是flex容器，  
其中藍框元素高度固定，綠框元素高度也固定。  
而綠框元素本身也是flex容器，其中灰底子元素被設為`flex: auto`和`overflow: auto`。  
結果可以看到灰底元素的內容過長時，並不會綠框的範圍。

[CodePen](https://codepen.io/tunafin/pen/mybpRLx)

<div style="width: 100%; max-width: 500px; margin: 10px 0;">
  <img src="https://i.imgur.com/dB7CZUt.png">
</div>

---

現在我們想要綠框元素能填滿剩下的空間阿，於是我們調整一下，  
將籃框元素的高度改成`flex: 0 0 40px`，  
綠框元素的高度改成`flex: auto`。  
此時就能發現不對勁了，綠框元素竟然超出父容器的範圍了!?

[CodePen](https://codepen.io/tunafin/pen/vEBpgGZ)

<div style="width: 100%; max-width: 500px; margin: 10px 0;">
  <img src="https://i.imgur.com/GzRLZ4t.png">
</div>

其實解決方法也很簡單，  
只要將灰底子元素的 'flex auto' 改成 `flex: 1 1 0` 就行了。  
最終結果如下圖。

<div style="width: 100%; max-width: 500px; margin: 10px 0;">
  <img src="https://i.imgur.com/o1Qf9iF.png">
</div>

註: 有在網路上找到另一方法，套用到上述範例的話，  
就是將綠框元素補上`min-height: 0`，也可解決內容跑板問題。
