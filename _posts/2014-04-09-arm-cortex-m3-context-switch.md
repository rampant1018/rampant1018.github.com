---
layout: post
title: "ARM Cortex M3 Context Switch"
description: ""
category: Embedded
tags: [ASM, C, OS]
---
{% include JB/setup %}

#ARM Cortex-M3 #
### Execution Modes ###
Cortex-M3執行分成兩種模式：Thread mode、Handler mode

* Thread mode
    * 會在reset以及從exception跳出時進入
    * code可以在Priviledge或Unpriviledge下執行
* Handler mode
    * 會在發生exception時進入
    * code一定會以Priviledge執行，所以當exception發生時會自動跳轉至Priviledge mode
* 切換方式
    * 可以使用`MSR`將`CONTROL[0]`清空，從Priviledge Thread mode轉到User Thread mode
    * 無法在沒有進入exception的情況下，直接由Unpriviledge mode轉到Priviledge mode

### Main Stacks and Process Stacks ###
Cortex-M3支援兩個不同的堆疊：Main Stack、Process Stack，R13在同一個時間點只會指向其中一個`sp`，但是可以利用`MSR`及`MRS`指令取得這兩個`sp`。

* 在reset及進入Handler mode時會切換至Main stack
* 只有在Thread mode下可以將PSP視為當前的SP
* 在Thread mode下有兩種方法可以切換main stack及process stack，其中之可以利用`MSR`指令將`CONTROL[1]`寫入達成main stack與process stack間的切換

### System Calls ###
透過轉換Thread mode以及Handler mode可以達成系統呼叫的功能，下面是trace rtenv的系統呼叫`write`過程：

1. 呼叫write()
2. 將r8堆進stack
3. 將0x3寫入r8(syscall number)
4. 呼叫svc，產生exception，由SVC_Handler處理，產生exception時會將依序將xPSR、PC、LR、R12、R3、R2、R1、R0堆入stack
    SVC_Handler()
    1. 將user state堆入process stack中
    2. 從main stack中取出kernel state
    3. branch到lr，此處的lr是call activate進入activate後存的function return address，所以會branch回到call activate的下一句指令
5. kernel中處理syscall經過迴圈後call activate
    activate()
    1. 將kernel state存入main stack
    2. 將r0寫進psp，並切換至process stack(CONTROL)
    3. 從process stack中將user state取出
    4. branch到lr，此處的lr是call svc進入svc_handler後存的function return address，所以會branch回到call svc的下一句指令
6. 結束write

下圖是以`sbrk`為範例呼叫
![System call sbrk](https://farm8.staticflickr.com/7101/13734597853_f0a70eb985_o.jpg)
