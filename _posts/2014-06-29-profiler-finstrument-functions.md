---
layout: post
title: "Profiler: finstrument functions"
description: "GCC提供一個編譯選項，可以用來trace以及profile函式"
category: learning
tags: [C]
---
{% include JB/setup %}

前幾天在看 profiler 的時候發現 GCC 提供一個很有趣的編譯選項 `finstrument-functions`，附上 GNU 手冊中的說明[連結](https://gcc.gnu.org/onlinedocs/gcc-4.5.1/gcc/Code-Gen-Options.html#index-finstrument_002dfunctions-2114)。

加上這個參數以後，GCC 會自動在進入函式後以及離開函式前產生 instrumentation call，接著分析用的函式就能取得目前的函式位址以及呼叫者。函式定義：`void __cyg_profile_func_enter(void *this_fn, void *call_site)`、`void __cyg_profile_func_exit`，第一個參數就是目前函式的起始位址，可以從 symbol table 查表對應。

另外，可以替函式增加`no_instrument_function`屬性避免產生 instrumentation，這可以被用在任何無法安全呼叫分析函式的地方，例如 signal handler。`-finstrument-functions-exclude-file-list=file,file...`會在產生 instrumentation 時排除 file 檔案內有宣告的函式，對照檔名是依照子字串的方式，例如排除`/sys`，則所有路徑中包含`/sys`的檔案都會在排除清單中。

## 實際測試 ##

{% highlight c linenos %}
/* main.c */
#include <stdio.h>

void fun1();
void fun2();
void fun3();

void fun1()
{
	printf("Fun1 call Fun2\n");
	fun2();
	printf("Fun1 call Fun3\n");
	fun3();
}

void fun2()
{
	printf("Fun2 call Fun3\n");
	fun3();
}

void fun3()
{
	printf("In fun3\n");
}

int main()
{
	fun1();

	return 0;
}
{% endhighlight %}

先進行編譯：`gcc -g -c -finstrument-functions main.c -o main.o`，透過 readelf 觀察產生出的物件檔

	$ readelf -r main.o
	Relocation section '.rela.text' at offset 0x7c0 contains 28 entries:
	  Offset          Info           Type           Sym. Value    Sym. Name + Addend
	00000000000c  00090000000a R_X86_64_32       0000000000000000 fun1 + 0
	000000000011  000a00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_ent - 4
	000000000016  00050000000a R_X86_64_32       0000000000000000 .rodata + 0
	00000000001b  000b00000002 R_X86_64_PC32     0000000000000000 puts - 4
	000000000025  000c00000002 R_X86_64_PC32     0000000000000050 fun2 - 4
	00000000002a  00050000000a R_X86_64_32       0000000000000000 .rodata + f
	00000000002f  000b00000002 R_X86_64_PC32     0000000000000000 puts - 4
	000000000039  000d00000002 R_X86_64_PC32     000000000000008c fun3 - 4
	000000000045  00090000000a R_X86_64_32       0000000000000000 fun1 + 0
	00000000004a  000e00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_exi - 4
	00000000005c  000c0000000a R_X86_64_32       0000000000000050 fun2 + 0
	000000000061  000a00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_ent - 4
	000000000066  00050000000a R_X86_64_32       0000000000000000 .rodata + 1e
	00000000006b  000b00000002 R_X86_64_PC32     0000000000000000 puts - 4
	000000000075  000d00000002 R_X86_64_PC32     000000000000008c fun3 - 4
	000000000081  000c0000000a R_X86_64_32       0000000000000050 fun2 + 0
	000000000086  000e00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_exi - 4
	000000000098  000d0000000a R_X86_64_32       000000000000008c fun3 + 0
	00000000009d  000a00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_ent - 4
	0000000000a2  00050000000a R_X86_64_32       0000000000000000 .rodata + 2d
	0000000000a7  000b00000002 R_X86_64_PC32     0000000000000000 puts - 4
	0000000000b3  000d0000000a R_X86_64_32       000000000000008c fun3 + 0
	0000000000b8  000e00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_exi - 4
	0000000000cf  000f0000000a R_X86_64_32       00000000000000be main + 0
	0000000000d4  000a00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_ent - 4
	0000000000de  000900000002 R_X86_64_PC32     0000000000000000 fun1 - 4
	0000000000ef  000f0000000a R_X86_64_32       00000000000000be main + 0
	0000000000f4  000e00000002 R_X86_64_PC32     0000000000000000 __cyg_profile_func_exi - 4

編譯器自動在進入函式後以及離開函式前加上 profile 函式，再來就實作 profile 函式。

{% highlight c linenos %}
#include <stdio.h>
#include <time.h>

void __cyg_profile_func_enter(void *func, void *caller)
{
	printf("e %p %p %lu\n", func, caller, time(NULL));
}

void __cyg_profile_func_exit(void *func, void *caller)
{
	printf("x %p %p %lu\n", func, caller, time(NULL));
}
{% endhighlight %}

編譯完後將兩個物件檔連結，執行後就能看到分析結果：

	$ gcc -c trace.c -o trace.o
	$ gcc trace.o main.o -o main
	$ ./main
	e 0x400726 0x7f93682e576d 1403981337
	e 0x400668 0x40074a 1403981337
	Fun1 call Fun2
	e 0x4006b8 0x400691 1403981337
	Fun2 call Fun3
	e 0x4006f4 0x4006e1 1403981337
	In fun3
	x 0x4006f4 0x4006e1 1403981337
	x 0x4006b8 0x400691 1403981337
	Fun1 call Fun3
	e 0x4006f4 0x4006a5 1403981337
	In fun3
	x 0x4006f4 0x4006a5 1403981337
	x 0x400668 0x40074a 1403981337
	x 0x400726 0x7f93682e576d 1403981337

目前產生的位址必須到 symbol table 中找出對應的函式，例如：

	$ readelf -s main | grep 400726
	    68: 0000000000400726    67 FUNC    GLOBAL DEFAULT   13 main

`binutils`中有提供一個工具`addr2line`，這個工具可以用來輸出一些除錯資訊：

	$ addr2line -e main -f 0x400726
	main
	/home/rampant/workspace/testcode/prof/main.c:27

透過上面這個 profile 函式的機制，加上自訂的紀錄方式，就能簡單的做出程式的 profiler。

## 參考資料 ##

* http://balau82.wordpress.com/2010/10/06/trace-and-profile-function-calls-with-gcc/
