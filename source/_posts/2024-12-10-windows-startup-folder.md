---
title: Windows 啟動資料夾(Startup Folder)
date: 2024-12-10 21:22:25
updated:
categories:
tags:
  - Windows
---

<div style="width: 100%; max-width: 500px; margin: 10px 0;">
  <img src="https://i.imgur.com/0zYpbCM.png">
</div>

當你每天開機時，是否希望某些程式能自動啟動，例如工作需要的文書工具、行事曆軟體或是聊天程式？其實常見的程式像是LINE、Discord等，本身就能設定開機自啟動，非常方便；但並不是所有程式都有該功能。而 Windows 的「啟動資料夾」就是一個非常方便的替代方案。透過這個資料夾，你可以輕鬆設定開機時自動執行的程式，讓日常工作更有效率。

在這篇文章中，我就來談談 Windows 中的啟動資料夾、如何找到它、以及如何新增或移除啟動程式。

<!-- more -->

<br>

### 什麼是啟動資料夾？

啟動資料夾（Startup Folder）是一個特殊的資料夾，裡面的程式捷徑會在 Windows 開機時自動執行。Windows 其實有兩個啟動資料夾：

個人使用者的啟動資料夾：僅適用於目前登入的使用者。
所有使用者共用的啟動資料夾：適用於所有帳號。

### 如何找到啟動資料夾？

#### 個人使用者的啟動資料夾

  1. 按下 Win + R 開啟執行視窗。
  2. 輸入以下指令：

     ```ts
     shell:startup
     ```

#### 所有使用者共用的啟動資料夾

  1. 按下 Win + R 開啟執行視窗。
  2. 輸入以下指令：

     ```ts
     shell:common startup
     ```

### 新增程式到啟動資料夾

如果你想讓某個程式在開機時自動執行，將程式捷徑拖曳到資料夾中。  
完成後，下次開機時，該程式就會自動啟動惹。  
如果不想讓某個程式自動啟動，也只要將其從資料夾移除就好囉。  

這邊舉個簡單例子，如果我們希望每次自動開啟Youtube網頁，  
我們可以先建立一個網頁捷徑，並把目標設成下面這樣:

  ```ts
  "C:\Program Files\Google\Chrome\Application\chrome.exe" --profile-directory="Profile 1" https://www.youtube.com/
  ```

再把它放進啟動資料夾就行了~
<div style="width: 100%; max-width: 500px; margin: 10px 0;">
  <img src="https://i.imgur.com/iOSITKu.png">
</div>
