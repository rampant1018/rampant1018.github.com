---
layout: post
title: "The C Programming Language(A Storage Allocator)"
description: ""
category: 讀書筆記 
tags: [C, OS]
---
{% include JB/setup %}

## Overview ##

brk 跟 sbrk 這兩個系統呼叫/函式是用於改變 program break 的位置，program break 定義出執行序的 data segment 結尾(i.e. program break 是在未初始化的資料區段結尾後第一個位置)，增加 program break 的效果就是替執行序配置記憶體，反之則是 deallocate 記憶體。brk 以及 sbrk 在 UNIX manual 的描述如下：

`int brk(void *addr)`，brk 只要滿足下列三個條件就能將 program break 設定到 addr 的位置：

1. 合理的位置
2. 系統有足夠的記憶體
3. 處理序還未超出他的最大 data size

若是成功會回傳 0，失敗則回傳 -1

`void *sbrk(intptr_t increment)`，sbrk 則是將 program break 增加`increment`個位元組，若參數給 0 就可以找到當前的 program break 位置。成功的話回傳前一次的 program break，也就是增加空間後的起始位置，失敗則回傳 -1。

---
malloc 與 free 則是用來配置/釋放動態記憶體，詳細的說明可以從 UNIX manual 中找到：

* `void *malloc(size_t size)`，配置`size`個位元組，並且回傳配置好的記憶體位置
* `void free(void *ptr)`，釋放`ptr`指向的記憶體空間，這個指標必須是由 malloc 所配置出的位置

## Implementations ##

[The C Programming Language Ritchie & Kernighan](http://zanasi.chem.unisa.it/download/C.pdf)提供了一段 malloc 的實作方式，整理於下：

一般來說，malloc 會向作業系統要求空間，但不同的 task 不一定都會使用 malloc，造成分配下來的記憶體空間不連續，所以會儲存一串可用區塊(free block)的序列，每一個區塊會包含大小、下一個區塊的指標、以及空間位置，區塊會依照空間位置作排序，且最後一個區塊會指回第一個區塊。

![malloc block](https://farm6.staticflickr.com/5557/14371849244_b93d5c86ff_c.jpg)

---
配置記憶體時搜尋區塊分兩種演算法：

* first fit - 搜尋到第一塊足夠大小的區塊
* best fit - 搜尋到最適合大小的區塊，也就是最小且滿足需求的區塊

處理方式：

* 區塊大小剛好 - 會將區塊從可用序列中移除，並回傳給使用者
* 區塊過大 - 將區塊切開，將適當大小傳給使用者，剩餘區塊在放回可用序列中
* 沒有足夠大小的區塊 - 向作業系統要求一塊夠大的記憶體，並連結至可用序列

釋放記憶體：

1. 找到最適當的位置將區塊插入可用序列
2. 若此區塊與旁邊的可用區塊相鄰，則將其合併，避免區塊過於片段

---
對齊(Alignment)，每部不同的機器都有最有限制性的型態，當某個最嚴格限制的型態能儲存在某個位置，則其他型態也能存在相同位置，有些機器是double，有些則是int或long就足夠了。

下面是一個處理對齊的範例標頭(Header)：

{% highlight c linenos %}
typedef long Align;         /* for alignment to long boundary */

union header {                /* block header */
    struct {
	    union header *ptr;  /* next block if on free list */
	    unsigned size;          /* size of this block */
    } s;
    Align x;   /* force alignment of blocks */
};

typedef union header Header;
{% endhighlight %}

`Align`可以用來處理上面提到的對齊問題，透過 union 可以將標頭調整到最適當的大小，這邊是使用`long`這個型態，除此之外，為了簡化對齊問題，所有區塊大小都是標頭大小的整數倍。

一般 malloc 都會將使用者要求的大小自動進位到最適合的大小，每個區塊會替標頭多配置一個單位大小，並把整個區塊的大小存在標頭中的 size 欄位，而 malloc 回傳的指標位置會指向可用空間而不是標頭，使用者可以對他要求的空間做任何操作，但如果操作超出範圍，很有可能會發生錯誤。

![malloc return](https://farm6.staticflickr.com/5158/14187927647_f355304002.jpg)

{% highlight c linenos %}
static Header base;    /* empty list to get started */
static Header *freep = NULL;    /* start of free list */
/* malloc: general-purpose storage allocator */
void *malloc(unsigned nbytes) {
    Header *p, *prevp;
    Header *morecore(unsigned);
    unsigned nunits;
    nunits = (nbytes+sizeof(Header)-1)/sizeof(header) + 1;
    if ((prevp = freep) == NULL) {    /* no free list yet */
	    base.s.ptr = freep = prevp = &base;
	    base.s.size = 0;
    }
    for (p = prevp->s.ptr; ; prevp = p, p = p->s.ptr) {
	    if (p->s.size >= nunits) { /* big enough */
		    if (p->s.size == nunits) /* exactly */
			    prevp->s.ptr = p->s.ptr;
		    else {    /* allocate tail end */
			    p->s.size -= nunits;
			    p += p->s.size;
			    p->s.size = nunits;
		    }
		    freep = prevp;
		    return (void *)(p+1);
	    }
	    if (p == freep) /* wrapped around free list */
		    if ((p = morecore(nunits)) == NULL)
			    return NULL;    /* none left */
    }
}
{% endhighlight %}

* Line 8 : Round up to proper size
* Line 9~12 : 若freep為NULL，代表還未執行過malloc，先進行初始化
* Line 13 : 開始搜尋free list
* Line 14 : 搜尋到足夠大的free block 
* Line 15~16 : free block與要求的大小一樣，直接將free block從free list移除
* Line 17~21 : free block過大，從結尾切好要求的大小，並把前段的header size剪去切掉的大小，在將指標p指向新的block，並assign區塊大小
* Line 22~23 : 將free list指向上一個free block，因為p位置是header，所以回傳p + 1的位置
* Line 25 : 搜完整個list都沒找到適合的
* Line 26~27 : 向系統要記憶體，若是失敗則回傳NULL
* morecore(nunits)是向OS要求記憶體，會將新配置的記憶體插進free list，並回傳freep

### Implement on rtenv ###

因為目前 rtenv 還未有記憶體管理，因此直接在 kernel 中宣告一塊足夠大的 heap 空間，sbrk 會直接從這段空間中配置記憶體給使用者。實作的程式碼放在[GitHub](https://github.com/rampant1018/rtenv-1/commit/2cb4c582af8461f93ab0269097d7f30785cb92f4)。

