---
title: CH08_B：Memory Management
tags: GIS ComputerFundamental
layout: article
aside:
    toc: true
---


## Page table 的實作
前提： `info`{:.info}
- 每一個 process 有自己的 page table！
- page table 放在較慢的 DRAM
- 當 CPU 要 acc addr. 的時候，會需要將 logical addr.(page number) 拿到 page table
  以 $$O(1)$$ 的時間找到映射的 physical addr.(frame number)。==所以有兩次 mem acc==

根據上述前提第1點：
所以未來在做 context switch 的時候，需要有一個變數(register)是跟隨著該 process，並用來儲存該 process 的 page table 起始地址。我們叫做 Page-table base register(PTBR)。
理所當然他應該要被放在 PCB(Process control block) 裡面被 OS 所管理。
所以 PCB 會多出一個欄位專門存放該 process 的 page table 位於 DRAM 的起始位址。
Hence the value holding by PTBR is a physical address.

根據上述前提第2、3點：
這是極大的 performance 問題！速度直接減半！
所以我們可以另外加一個擁有 chche 功能的 TLB(Translation Look-aside Buffer)。他是 full associative 的，所以 lookup time = $$O(1)$$。==它為了速度被設計在 SRAM 裡面，屬於一種硬體支援(只有一個 TLB) 所以沒有辦法像 page table 兩個 process 就有兩個==
從 mem 拿到想要的資料需要對 DRAM 作兩次的 mem acc。
第1次是對 page table。這也是由 paging 所衍生出來的成本(蠻大的成本)。
第2次是對 physical 地址的 acc。
而如果我們可以將整個 page table 放到 SRAM 就好了，這樣我們就從兩個 DRAM acc 變成 一個 SRAM、一個 DRAM 的 acc，幾乎是只有一個 DRAM 的 acc 時間代價。
可惜我們沒有辦法將整個 table 搬到 SRAM。太貴了。
雖然現實不會總是這麼美好，但工程是妥協的藝術。
我們可以把部份的 page table 搬到 SRAM，我們叫他 TLB (translation look-aside buffer)
加了這個功能，我們就可以將原本兩次的 DRAM acc 變成幾乎剩下一次的 DRAM acc。

==根據上述，TLB 需要在 context switch 的時候被 flush。==
==而這也是 context switch 導致的效能下降的主因！==
(除了 context switch 過程中所需要的複製是一種 overhead 之外)
![[Pasted image 20240719151131.png|600]]
話雖如此，現代電腦系統真的可以達到９８％甚至以上的 hit rate 嗎？
其實是可以的...因為 locality。

---

## 功能運作正常，那麼該如何保護 page table？
Memory protection！
valid bit + length 去判斷是否取到 out of memory
- valid bit indicates 有可能目前被放到 disk 裡面
- length indicates 該 page 裡面 alloc 到多少的 mem
  (因為我們的 acc 是以 byte 為單位，不是以 page 為單位)

Shared pages：
Paging allows processes share common code, which must be ==reentrant==(pure code 沒有人可以在執行期間去修改它，如 editor, compiler, web servers, etc.).
- share data, code, or even library. copy on write, 如 dynamic loading。

與其 share memory，不如 share 那些被重複用到的 library 的 page 們。
而這可以透過階層式 paging (page table 中 entry 的 mapping)

```C++
#include<bits/stdc++.h>

int main(){
    cout << "ASD" << '\n';
}
```