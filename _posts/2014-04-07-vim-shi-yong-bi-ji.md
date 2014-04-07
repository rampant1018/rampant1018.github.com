---
layout: post
title: "Vim使用筆記"
description: "一些Vim使用的心得以及設定"
category: application
tags: [Vim]
---
{% include JB/setup %}

## Vimrc
Vim個人設定檔存放在*~/.vimrc*，下面是一些常用的設定

* autoindent(ai) － 換行時會依照目前的縮排
* cindent(cin) － 適合 C 語言的縮排方式
* hlsearch(hls) － 搜尋時會以反白方式顯示關鍵字
* ruler(ru) － 在狀態列顯示目前行列狀態
* shiftwidth(sw) － 使用 &gt;&gt; 移動整行內容的寬度
* tapstop(ts) － Tab 鍵的寬度
* showcmd(sc) － 在狀態列顯示目前所執行的命令
* showmode(smd) － 在狀態列顯示目前的模式
* number(nu) － 顯示行號

其中`softtabstop(sts)`可以將`tab`以`空白鍵`取代，例如：

    set softtabstop=4
    set shiftwidth=4
    按一次tab鍵會跳出四格空白
    若二兩次tab鍵則會補上tab


## Folding
Vim有程式碼折疊的功能：

* zf - 新增折疊
* zo - 開啟折疊
* zc - 關閉折疊
* zr - 關閉全文折疊
* zm - 開啟全文折疊

zf 後可加上物件選擇指令，例如 zfap 是折疊一個段落，或是利用 V 將要折疊的部份選取後使用 zf 指令。

    依據 syntax 自動折疊
    set foldmethod=syntax

    限制最大折疊層數
    set foldnestmax=3
