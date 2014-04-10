---
layout: post
title: "C: Function Pointer"
description: ""
category: Learning 
tags: [C]
---
{% include JB/setup %}

Function pointer就是一個指向某函式位置的變數，透過函式指標可以讓程式在利用性提高，例如一個Linked List的函式庫提供`foreach(func)`的功能，在使用這個函式庫時只要替換func就能在每次foreach產生不同結果。

假設我的list函式庫中有一個foreach函式，參數是一個函式，他的參數是一份資料

    void list_foreach(void (*operation)(void *)) {
        foreach(list) {
            operation(list->iterator);
        }
    }

接著我在使用這個函式庫時，只要使用我自己的function代入函式，就能在foreach中執行我要的方法

    void my_func(void *data) {
        process(data);
    }

    void main() {
        list_foreach(my_func);
    }

這樣函式庫中就不需要在定義一個`list_foreach_my_func()`的函式

這個[範例](https://github.com/rampant1018/practice/commit/f76bbe16c2ee95ceefad4962faac9d1066c530ab)就是定義好foreach的介面，接著main function在使用foreach時分別使用`dlist_find_max()`與`dlist_sum()`產生結果，如此一來dlist中不需要也不應該提供find_max以及sum功能。

