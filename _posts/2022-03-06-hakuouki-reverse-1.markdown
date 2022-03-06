### 薄樱鬼正经逆向（更新 ing）

#### 0. 一些废话

本篇是 [上一篇](https://kyan0s.github.io/2022/03/05/hakuouki-reverse-0.html) 的续集，并且因还没逆完而持续更新中。以及上一篇忘了提逆向的程序是薄樱鬼风之章的可执行程序，`MD5 (HakuokiWin.exe) = 2d674d5e156309e1a756e131e1e456a`，毕竟花之章我还没打通怎么能自己剧透自己。~~顺便调试时用的伊庭八郎小甜饼 DLC 已经看吐了~~

<br>

#### 1. 逆向过程

##### 1.1 寻找音频名称对应的数据结构

思路非常简单粗暴，从上一篇提到的断点入手，寻找如 `/voice/096006.hca` 等文件名第一次被设置时的函数，并逆向其设置文件名所需要的数据结构。于是从断点所在的函数一层层往上找，直到遇到了这个：

```c
snprintf (ebx, size, "/voice/%06d.hca", esi);
```
那一刻我感受到了身为傻逼而且自知的深深悲伤。至少这证明我没找错，于是顺着寻找这个 `esi` 寄存器值究竟如何从何而来，又一层层往上翻之后找到了 `fcn.0x0043f190`，只有在这里代表文件名的立即数不是由其调用者传递参数而来，而是来自于其所调用函数返回值：

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

对该函数的进一步分析发现音频名称数据结构为：

```goland
type HCAName struct {
    Six   uint32   // value = 6
    Four  uint32   // value = 4
    Index uint32   // in hex
}
```
如声音文件 `096013.hca`，其对应的数据结构为：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/hca-data-structure.png)

<br>

#### 2. 遇到的一些安全保护措施

本来还想着作为商业软件有很多保护措施也不意外，结果似乎除了类似于 `if not always_return zero() then` 这种不知道算不算混淆的东西外，正经的保护措施只找到了一个栈 cookie。希望 Idea Factory 家新的乙游也能保持这样的风格。

<br>

##### 2.1 Canary

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
