## 薄樱鬼正经逆向

<br>

### 0. 一些废话

本篇是 [上一篇](https://kyan0s.github.io/2022/03/05/hakuouki-reverse-0.html) 的续集，并且因还没逆完而持续更新中。以及上一篇忘了提逆向的程序是薄樱鬼风之章的可执行程序，`MD5 (HakuokiWin.exe) = 2d674d5e156309e1a756e131e1e456a`，毕竟华之章我还没打通怎么能自己剧透自己。~~顺便调试时用的伊庭八郎小甜饼 DLC 已经看吐了~~

<br>

### 1. 逆向过程

#### 1.1 寻找音频名称对应的数据结构

思路非常简单粗暴，从上一篇提到的断点入手，寻找如 `/voice/096006.hca` 等文件名第一次被设置时的函数，并逆向其设置文件名所需要的数据结构。于是从断点所在的函数一层层往上找，直到遇到了这个：

```c
snprintf (ebx, size, "/voice/%06d.hca", esi);
```
那一刻我感受到了身为傻逼而且自知的深深悲伤。至少这证明我没找错，于是顺着寻找这个 `esi` 寄存器值赋值源头。又一层层往上翻之后找到了 `fcn.0x0043f190`，只有在这里代表文件名的立即数不是由其调用者传递参数而来，而是来自于其所调用函数返回值：

```assembly
0x0043f1e4      call    fcn.0043f740
0x0043f1ec      mov     dword [var_8h], eax
; ...
0x0043f207      mov     eax, dword [var_8h]
0x0043f211      push    0
0x0043f213      push    ecx
0x0043f21b      push    1        
0x0043f21d      push    eax        ; instant value for hca index
0x0043f21e      call    fcn.0043afe0
```

对该函数的进一步分析发现音频名称数据结构为（名字是自己瞎起的）：

```goland
type HCAName struct {
    Six   uint32   // value = 6
    Four  uint32   // value = 4
    Index uint32   // in hex
}
```
如声音文件 `096013.hca`，其对应的数据结构为：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/hca-data-structure.png)

<center>图一</center>

<br>

#### 1.2 寻找写入该数据结构的函数

其实逆向出结构体之后第一反应是通过 `bgrep` 在 `DLC_JP00000.pac` 解包出的各个文件中寻找上图所示的字符串，不过没有找到。或许游戏厂商认为打包的文件大小比内存值钱多了吧。不过如果单独搜索 hca 文件索引如 `0D 77 01 00`，则能够在 `.dat` 文件中找到对应字符串，且字符串前疑似有 16 字节长度 [data header](https://github.com/u3shit/neptools/blob/master/doc/formats/stcm.md) 引导：

```shell
[0x00042c50]> x32
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00042c50  0000 0000 0100 0000 0100 0000 0400 0000  ................
0x00042c60  0d77 0100 0000 0000 0100 0000 0100 0000  .w..............
```

推测 `HCAName` 其实为对 `.dat` 或者什么文件解析所得。开发者应该不会每次播放新语音时都解析一遍文件，所以对于某一章节内播放顺序连续的音频文件，其在内存中的位置应该也是连续的。如果需要切换新的待解析文件，其解析结果也有可能存放于上一份文件解析结果所在的内存区域中。于是从一角色 DLC 章节中退出并点选另一位角色 DLC 章节，并分别在内存中搜索两位角色的前三句语音对应的 `HCAName`，可以看到基本证实了这一假设：

```shell
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 c1 da 00 00
1cfb3e20  06 00 00 00 04 00 00 00-c1 da 00 00 00 00 00 00  ................
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 c2 da 00 00
1cfb5820  06 00 00 00 04 00 00 00-c2 da 00 00 00 00 00 00  ................
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 c3 da 00 00
1cfb8620  06 00 00 00 04 00 00 00-c3 da 00 00 00 00 00 00  ................
0:000> g
Breakpoint 10 hit
eax=00017701 ebx=38ec6f01 ecx=1cfb4020 edx=00000000 esi=00000004 edi=1cfb4020
eip=00eaf304 esp=00d3f5f8 ebp=00d3f60c iopl=0         nv up ei pl zr na pe nc
cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b             efl=00000246
HakuokiWin!CAsyncReader::GetPin+0x16534:
00eaf304 84db            test    bl,bl
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 01 77 01 00
1cfb4020  06 00 00 00 04 00 00 00-01 77 01 00 00 00 00 00  .........w......
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 02 77 01 00
1cfb5c20  06 00 00 00 04 00 00 00-02 77 01 00 00 00 00 00  .........w......
0:000> s 1a330000 L10000000 06 00 00 00 04 00 00 00 03 77 01 00
1cfb9620  06 00 00 00 04 00 00 00-03 77 01 00 00 00 00 00  .........w......
```

于是随便找一个看上去大概率会被新文件覆盖到的地址（如 `0x1cfb5000`），对其下写断点。前几次触发写断点的函数均为 `memset`，于是将 `memset` 返回值作为软件断点。

```shell
14 d Enable Clear  1cfb5000 w 4 0001 (0001)  0:**** 
15 e Disable Clear  00e777ad     0001 (0001)  0:**** HakuokiWin+0x77ad
16 e Disable Clear  00eaf54f     0001 (0001)  0:**** HakuokiWin!CAsyncReader::GetPin+0x1677f
17 e Disable Clear  00eaf5b4     0001 (0001)  0:**** HakuokiWin!CAsyncReader::GetPin+0x167e4
```

这样尝试几次后一个有意思的现象发生了：会连续触发断点 15，且每次断点 15 触发后，`eax` 寄存器中均存放了下一块将要写入的内存区域地址；该内存区域通常为 `0x100` 字节，且在写入之前会通过 `memset` 先将该区域清空。某次写入后对应内存区域如下：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/hitotsu-write.png)

<center>图二</center>

<br>

并且，对每一内存区域的写入不是一步即可的。下图中，正在写入的内存区域起始位置为 `0x1cfb6420`，下一块内存区域（待清空）起始位置为 `0x1cfb6520`。在本块内存区域中，先写入了块结尾处的 `Batch_item`，然后才写入了红色标识的 `20 62 fb 1c` 等。

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/toaru_write.png)

<center>图三</center>

<br>

 通过对比图一图二，可以推测 `HCAName` 的前两个固定值字段实际位于块头部分。经过虽然谈不上漫长却依旧非常痛苦的调试之后，确认了写入 `HCAName` 的函数为 `0x00458ce0`(`HakuokiWin!CAsyncReader::GetPin+0x2fdb7`)。事实上还有复数个函数能够写入 `Batch_item` 块，这些函数所对应的块头也各个不同；但大体上这些函数都包括如下逻辑：
 
 + 通过调用 0x0043f5f0 写入块尾 `Batch_item` 字符串
 + 通过调用 0x004590dc 写入块头第一字节
 + 通过调用 0x004590e6 写入块头第二子节
 + 对块其他部分的处理

函数 `0x00458ce0` 写入块头前两字节代码如下：
 
 ```assembly
0x004590d7      push    6          ; 6
0x004590d9      push    ebx
0x004590da      mov     esi, dword [eax]
0x004590dc      call    fcn.0043f850
0x004590e1      push    4          ; 4
0x004590e3      push    0
0x004590e5      push    ebx
0x004590e6      call    fcn.0043f870
 ```

调用这些写块函数的分发函数（`0x4d8420，HakuokiWin!CAsyncReader::GetPinCount+0x1978`）似乎应该是后续逆向文件结构的重中之重。

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/switch-func.png)

<center>图四</center>

<br>

顺便记录一下 `main` 到分发函数的调用栈：

+ 0x004d9e40
+ 0x004d9d40
+ 0x004ca830
+ 0x004069e0 (call eax, from block 0x00406a52)
+ 0x00406a9b in 0x004069e0 call 0x004069e0
+ 0x004c5260
+ 0x004c5ae0  main

<br>

### 2. 遇到的一些安全保护措施

本来还想着作为商业软件有很多保护措施也不意外，结果似乎除了类似于 `if not always_return zero() then` 这种不知道算不算混淆的东西外，正经的保护措施只找到了一个栈 cookie。希望 Idea Factory 家新的乙游也能保持这样的风格。

<br>

#### 2.1 Canary

```assembly
; function starts
0x004bce59      mov     eax, dword [0x5e75f4c]
0x004bce5e      xor     eax, ebp
0x004bce60      mov     dword [var_4h], eax
; ...
; function ends
0x004bcfc4      mov     ecx, dword [var_4h]
0x004bcfcc      xor     ecx, ebp
0x004bcfd1      call    fcn.0050ab40
```

function \__security\_check\_cookie:

```assembly
fcn.0050ab40 (int32_t arg_4h, HINSTANCE arg_8h);
0x0050ab40      cmp     ecx, dword [0x5e75f4c]
0x0050ab46      jne     0x50ab4a
0x0050ab48      ret
0x0050ab4a      jmp     loc.0050b3b1 ; trigger fastfail
```
<br>

### 3. 下集预告与碎碎念

虽然还没有对分发函数进行逆向，但看到上溯其调用栈时能够一路回溯到 `main` 函数时还有点安心：大概是觉得只需要花时间逆出函数逻辑即可，而不需要再像没头苍蝇一般到处乱窜吧。下集预告自然是带解析文件的文件结构说明；当然如果被证实逆向得到的结果与[现有资料](https://github.com/u3shit/neptools/blob/master/doc/formats/stcm.md)完全一致，那这一系列将 ~~尴尬地~~ 宣布在此告结。

虽然说很久前就听前辈提到逆向本质体力活，但正因为劳动的不可避免，所谓技巧甚至熟能生巧才显得弥足珍贵。之后还是应该勤快逆向吖。

<br>


