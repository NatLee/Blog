---
title: Hexo在Windows上的安裝心得
categories: blog
tags:
  - hexo
  - windows
  - github
  - blog
  - nodejs
abbrlink: 2264945776
date: 2015-08-11 00:00:00
---

## 前言
----------
其實在幾年之前就有想過建立Blog記錄一些瑣事
但之前對於GitHub還不是很熟悉（現在也還在努力中），所以一直弄失敗。
現在好不容易弄成功了，我想把我的步驟記錄下來，或許能幫助到一些人XD

<!--more-->

## 步驟說明
----------

 1. 確認電腦有是否有node.js與Git

	想安裝Hexo需要git與node.js，而Github的桌面版程式已有附Git shell
	所以只需要到[Node.js官網](https://nodejs.org/)下載安裝環境即可
	> **Note:** 建議安裝完後重新啟動，確保環境變數已更新

 2. 開始安裝Hexo

	打開Github的Git shell，然後輸入以下code

	    npm install hexo-cli -g
	    npm install hexo --save
		npm install hexo-deployer-git --save

	直接輸入hexo，測試是否安裝成功，若成功再繼續往下

 3. 以Hexo建置自己的Blog

	首先，你必須到GitHub上面建立一個新的Repository

	> 這個名字之後會是你Blog網址

	建立完後，就可以用Github桌面版程式建立一個clone到自己電腦中
	然後再回到Git shell切換到剛建立的那個clone資料夾中

	    hexo init

	再來可依個人喜好安裝一些插件以幫助Blog營運

	    npm install hexo-generator-index --save
	    npm install hexo-generator-archive --save
	    npm install hexo-generator-category --save
	    npm install hexo-generator-tag --save
	    npm install hexo-server --save
	    npm install hexo-deployer-git --save
	    npm install hexo-deployer-heroku --save
	    npm install hexo-deployer-rsync --save
	    npm install hexo-deployer-openshift --save
	    npm install hexo-renderer-marked@0.2 --save
	    npm install hexo-renderer-stylus@0.2 --save
	    npm install hexo-generator-feed@1 --save
	    npm install hexo-generator-sitemap@1 --save

	這樣Blog的建置大致上完成了，輸入以下code可產生Blog並檢視

	    hexo g # g 為 generator 的縮寫，建置 Blog
	    hexo s # s 為 server 的縮寫，可以開啟 localhost ，方便檢視 Blog
    
   此時，只要在[localhost:4000](http://127.0.0.1:4000)，就可以看到自己的Blog了

 4. 試著建立文章吧！

	先來建立個文章看看吧！

	    hexo n "title" # n 為 new 的縮寫，此指令可以建立一個標題為「title」的文章

	至於內容則要到`/source/_posts/`中找到**.md檔**以**markdown**的格式編輯

	編輯方法可以參考 http://markdown.tw/
	而網路上也有一些線上編輯器，像是 https://stackedit.io/

 5. 上傳到GitHub

	上傳到GitHub有兩種方法
 
	第一種是回到Git shell執行以下command

	    hexo g

	這時資料夾底下可以發現**public**資料夾

	我們複製裡面的所有檔案貼到資料夾內（也就是**public**上一層）

	再回到GitHub的桌面版工具，上傳並總結這次修改的commit

	最後按下右上角的<i class="icon-refresh"></i>Sync同步

	等待一點時間並確認無誤後，這樣就完成上傳了！
	
    另外一個方法是修改資料夾中**_config.yml**檔案裡的**deploy**

	    deploy: 
	      type: git
	      repository: 你的repository網址
	      branch: gh-pages

    接下來找到**URL**並更改url和root

		# URL
		## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
		url: http://[yourAccount].github.io/Blog
		root: /Blog/
		
    最後都儲存完後，在Git shell上輸入以下command

		hexo d # d 為 deploy 的縮寫

    這樣也就完成大部份的建置了！


## 參考資料
----------

[佈署 | Hexo](https://hexo.io/zh-tw/docs/deployment.html)

[使用hexo以及github-page建立自己的部落格](http://eva0919.github.io/2013/04/21/%E4%BD%BF%E7%94%A8hexo%E4%BB%A5%E5%8F%8Agithub-page%E5%BB%BA%E7%AB%8B%E8%87%AA%E5%B7%B1%E7%9A%84%E9%83%A8%E8%90%BD%E6%A0%BC/)

[Hexo安裝教學、心得筆記](https://wwssllabcd.github.io/blog/2014/12/22/how-to-install-hexo-on-window/)

[使用GitHub和Hexo搭建免费静态Blog](http://wsgzao.github.io/post/hexo-guide/)

