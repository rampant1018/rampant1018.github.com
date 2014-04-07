---
layout: post
title: "GNOME: Desktop files"
description: "添加捷徑至GNOME啟動器"
category: 
tags: [Ubuntu]
---
{% include JB/setup %}

## Overview ##
在GNOME中會自動從註冊的應用程式清單中建立user menu，每一個應用程式的都是透過一個純文字檔(副檔名.desktop)`desktop entry`註冊，桌面會從這個檔案中取得一些資訊來使用：

* 將應用程式放在`Main Menu`
* 在`Run Application`中列出應用程式
* 在`menu`或是`desktop`中建立做適合的啟動器
* 將`name`與`description`與應用程式關聯
* 使用應用程式的icon

要新增一個menu entry需要建立一個desktop file，檔案名必須唯一，這個名字沒有長度限制，所以避免使用縮寫，並且使用UTF-8編碼，下面是範例：

    [Desktop Entry]
    Name=FooCorp Painter Pro
    Exec=foocorp-painter-pro
    Icon=foocorp-painter-pro
    Type=Application
    Categories=GTK;GNOME;Utility;

上面的範例是一個最基本的desktop file，接著將這個檔案放到*/usr/share/applications*讓所有人都能使用，或是放到*~/.local/share/applications*讓單一使用者使用，GNOME會自動監測，所以只要將檔案複製到正確的位置就能註冊。

## Sample

    [Desktop Entry]
    Type=Application
    Encoding=UTF-8
    Name=Sample Application Name
    Comment=A sample application
    Exec=application
    Icon=application.png
    Terminal=false

* [Desktop Entry] - 區段標頭，必須放在第一行用作檔案辨認
* Type=Application - 分辨檔案型態，另外兩種是`Link`、`Directory`
* Encoding=UTF-8 - 檔案編碼
* Name=Sample Application Name - 在`Main Menu`中的應用程式名稱
* Comment=A sample application - 用來描述應用程式，會被用在tooltip中
* Exec=application - 從shell執行應用程式的指令，可以包含arguments
* Icon=application.png - 應用程式的圖示名稱
* Terminal=false - 是否要使用終端機開啟

## Icons
在剛剛的Sample中設定了一個`Icon=application.png`，接著就要將icon放到正確位置讓他正常運作，desktop會從選定的佈景資料夾(*/usr/share/icons*)中尋找icon，最少要有一份解析度為48\*48的圖片，如果desktop在選定的佈景資料夾中找不到icon，他會到*/usr/share/icons/hicolor*中尋找。

新增好圖片後要重新產生icon的快取```$ gtk-update-icon-cache 'path of theme'```

假設我現在是用GNOME預設的佈景主題，則我要做：

1. 將icon放到*/usr/share/icons/gnome/48\*48/apps*
2. ```$ gtk-update-icon-cache /usr/share/icons/gnome```
3. 重新啟動Launcher
