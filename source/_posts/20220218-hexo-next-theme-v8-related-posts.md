---
title: Hexo NEXT Theme v8.10『相關文章』插件踩坑
categories: blog
tags:
  - blog
  - hexo
  - plugin
abbrlink: a606eb6a
date: 2022-02-18 02:41:06
---

![](https://repository-images.githubusercontent.com/115875606/0a408300-6d51-11ea-8edd-354d8279fa95)

## 前言
----------

最近更新了一波Hexo跟NEXT的版本

發現NEXT的『相關文章』插件沒辦法正常顯示

<!--more-->

## 內容
----------

把blog的NEXT主題更新到8之後

因爲主題可以用npm直接安裝了，專案內的東西少了很多

甚至還支援不修改NEXT原始碼插入元件的`custom file`方法

在測試啓動的時候會吐出炫砲的ANSI圖

```
❯ hexo s
INFO  Validating config
INFO  ==================================
  ███╗   ██╗███████╗██╗  ██╗████████╗
  ████╗  ██║██╔════╝╚██╗██╔╝╚══██╔══╝
  ██╔██╗ ██║█████╗   ╚███╔╝    ██║
  ██║╚██╗██║██╔══╝   ██╔██╗    ██║
  ██║ ╚████║███████╗██╔╝ ██╗   ██║
  ╚═╝  ╚═══╝╚══════╝╚═╝  ╚═╝   ╚═╝
========================================
NexT version 8.10.0
Documentation: https://theme-next.js.org
========================================
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/Blog/ . Press Ctrl+C to stop.
```

看似一切都很美好，但[hexo-related-popular-posts](https://github.com/tea3/hexo-related-popular-posts)，這個插件出了問題

問題就是它沒有被顯示出來！

如剛說的，NEXT在v8可以支援非侵入式的修改

所以我們得自行去新增顯示方法

首先，先找到NEXT的設定檔`./_config.next.yml`，並且把`postBodyEnd`的註解打開

```yml
# Define custom file paths.
# Create your custom files in site directory `source/_data` and uncomment needed files below.
custom_file_path:
  #head: source/_data/head.njk
  #header: source/_data/header.njk
  #sidebar: source/_data/sidebar.njk
  #postMeta: source/_data/post-meta.njk
  postBodyEnd: source/_data/post-body-end.njk
  #footer: source/_data/footer.njk
  #bodyEnd: source/_data/body-end.njk
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  #style: source/_data/styles.styl
```

按照路徑，我們建立一個資料夾`./source/_data`

然後新增檔案`./source/_data/post-body-end.njk`，並且把下面的內容填入

```js
{%- if theme.related_posts.enable and (theme.related_posts.display_in_home or not is_index) %}
    <!-- related posts -->
    {%- set popular_posts = popular_posts_json(theme.related_posts.params, page) %}
    {%- if popular_posts.json and popular_posts.json.length > 0 %}
        <div class="popular-posts-header">{{ theme.related_posts.title or __('post.related_posts') }}</div>
        <ul class="popular-posts">
        {%- for popular_post in popular_posts.json %}
            <li class="popular-posts-item">
            {%- if popular_post.date and popular_post.date != '' %}
                <div class="popular-posts-date">{{ popular_post.date }}</div>
            {%- endif %}
            {%- if popular_post.img and popular_post.img != '' %}
                <div class="popular-posts-img"><img src="{{ popular_post.img }}"></div>
            {%- endif %}
            <div class="popular-posts-title"><a href="{{ popular_post.path }}" rel="bookmark">{{ popular_post.title }}</a></div>
            {%- if popular_post.excerpt and popular_post.excerpt != '' %}
                <div class="popular-posts-excerpt"><p>{{ popular_post.excerpt }}</p></div>
            {%- endif %}
            </li>
        {%- endfor %}
        </ul>
    {%- endif %}
{%- endif %}
```

這個內容是從舊版插件會在文章模板內自動新增的東西，也就是秀出相關文章的程式碼

接着，我們再去把NEXT設定檔`./_config.next.yml`中，秀出『相關文章』的功能打開

```yml
# Related popular posts
# Dependencies: https://github.com/sergeyzwezdin/hexo-related-posts
related_posts:
  enable: true
  title: 相關文章推薦
  display_in_home: false
  params:
    maxCount: 5
    #PPMixingRate: 0.0
    #isDate: true
    #isImage: false
    #isExcerpt: false
```

設定完成後，我們就可以啓動看看結果

```
hexo g && hexo s
```


## 結語
----------

我個人覺得新版Hexo跟NEXT比較好管理

之前用的插件都要侵入式的修改，現在有`custom file`可以直接設定

這樣就不會因爲沒有記錄就升級導致某些檔案忘記一起修改
