# README

網址: https://tunafin.net/  
這是一個使用 Hexo 建立的靜態網站。  
Hexo 是一個快速、簡單且高效的靜態網站生成器，詳細請見 [Hexo官網](https://hexo.io/)。

## 關於部屬

Hexo 官方關提供的關於部屬的教學: [One-Command Deployment](https://hexo.io/zh-tw/docs/one-command-deployment)  
其中需要安裝到 `hexo-deployer-git`套件。  

但本專案並未使用以上方法，  
而是使用 `Github Actions` 來操作，  
當推到 `main` 分支時會觸發工作流程，並自動化完成部屬。  
(相關文件路徑: .github\workflows\pages.yml)
