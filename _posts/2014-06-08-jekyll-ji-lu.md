---
layout: post
title: "Jekyll紀錄"
description: "紀錄使用 GitHub Pages & Jekyll 寫作的過程"
category: learning
tags: [Jekyll]
---
{% include JB/setup %}

最近要寫新的文章時發現，已經忘記要怎麼發布文章，所以就紀錄一下過程，以免之後又忘記。

## Create a Post and Publish ##

新增 post 的指令如下：

    $ rake post title="title"

文章可以使用 chrome 的 [Markdown Preview](https://chrome.google.com/webstore/detail/markdown-preview/jmchmkecamhbiokiopfpnfgbidieafmd) 進行預覽，在 `.vimrc` 中加入熱鍵，這樣只要按下`F5`就能進行預覽：

    "" Open markdown files with Chrome.
    autocmd BufEnter *.md exe 'noremap <F5> :!/usr/bin/google-chrome-stable %<CR>'

完成文章編修後就能進行發布，發布其實就是將修改後的檔案 commit，接著 push 上去 github 等待更新就好。

    $ git add .
    $ git commit -m "Add new post"
    $ git push origin master

## Markdown ##

GitHub 預設是使用 [Maruku](https://github.com/bhollis/maruku/) 來處理 Jekyll 網站，不過 Maruku 在 2012 年的十月宣佈結束，所以 GitHub 建議使用較新的直譯器，方法是在`_config.yml`檔案中加入`markdown: kramdown`，這樣 GitHub 就會使用 [Kramdown](https://github.com/gettalong/kramdown) 取代 Maruku。

* [Kramdown Documentation](http://kramdown.gettalong.org/quickref.html)
* [Kramdown quick reference](http://kramdown.gettalong.org/quickref.html)
* [Kramdown Syntax](http://kramdown.gettalong.org/syntax.html)

## Jekyll ##

在編修完文章後可以先在本地端用 Jekyll 產生出最後的靜態網頁，確認沒問題在上傳到 GitHub。

    $ jekyll build
    # => The current folder will be generated into ./_site
    $ jekyll serve
    # => A development server will run at http://localhost:4000/

## 參考資料 ##

* <http://jekyllbootstrap.com/usage/jekyll-quick-start.html>
* <http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html>
* <https://help.github.com/articles/migrating-your-pages-site-from-maruku>
* [Jekyll usage](http://jekyllrb.com/docs/usage/)
