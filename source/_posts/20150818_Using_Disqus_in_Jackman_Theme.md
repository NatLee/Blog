title: 在Hexo的主題jacman中使用Disqus
date: 2015-08-18
categories: Tech
tags: [Hexo, Disqus, Github, Comment]
---

## 前言 ##
----------

之前裝hexo的時候，留言一直弄不好，還是先記錄一下步驟，以免之後忘記XD

<!--more-->

## 步驟 ##
----------
先註冊[Disqus](https://disqus.com/ "Disqus")帳號並登入到主頁面


1. 點選右上角的選項->**Add Disqus To Site**

	![](http://i.imgur.com/LJYmpVy.png)

2. 點選**Start Using Engage**

	![](http://i.imgur.com/w7A8ZVw.png)

3. 在**Site name**輸入你要的名字（這裡以**justTryTest**為例）

	![](http://i.imgur.com/2x04Lcz.png)

4. 按下**Universal Code**，把內容貼至
   **.\themes\jacman\layout\_partial\post\comment.ejs**
```js
    	<% if (theme.disqus_shortname && page.comments){ %>
			<section id="comments" class="comment">
				<div class="ds-thread" data-thread-key="<%- page.path %>" data-title="<%- page.title %>" data-url="<%- page.permalink %>"></div>
				<div id="disqus_thread"></div>
				<script type="text/javascript">
					/* * * CONFIGURATION VARIABLES * * */
					var disqus_shortname = 'justtrytest';

					/* * * DON'T EDIT BELOW THIS LINE * * */
					(function() {
			    		var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
			    		dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
	        		(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
					})();
				</script>
				<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
			</section>
		<% } %>
```
5. 再到**.\themes\jacman\_config.yml**，在最後一行加入

		#### Comment

		duoshuo_shortname:
		disqus_shortname: justtrytest
	> 　　**justtrytest**是你在**步驟3**所輸入的名字

6. 回到剛剛的網頁，按下**Advance**

	![](http://i.imgur.com/2uH5tyZ.png)

7. 在**Trusted Domains**加入自己Blog的位置

	![](http://i.imgur.com/Zd2W1zg.png)

8. 按下**Save Changes**後，重新啟用Blog即可以啟用Disqus

## 追記 ##
----------
安裝時，注意一下站點與theme本身需求的**_config.yml**的配置
似乎可以省一些麻煩XD
