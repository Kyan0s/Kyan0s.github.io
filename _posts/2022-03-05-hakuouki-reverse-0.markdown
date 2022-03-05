### 薄樱鬼初步逆向兼 windbg 的使用学习

#### 0. 废话之缘起

话不多说，开搞（bushi

本篇的目的就是做到能让我一边打游戏一边自动记录屏幕上显示的台词和外放的语音文件名称，好在这个目的还是达到了。基本不涉及啥逆向吧，功夫都花在查 windbg 怎么用了。~~不过 windbg 好东西，折磨还是值得的~~

<br>

#### 1. 过程记录

##### 1.0 开幕雷击.jpg

打开游戏。打开 windbg。Attach 进程。Break。一气呵成。然后我就发现我的游戏没法最小化了。

什么鬼这也是由游戏程序控制的吗 (ノ﹏ヽ) 好在可以 Alt + Tab 切换程序。（好久不用 windows 的萌新一开始真的被吓到了……)

<br>

##### 1.1 寻找字符串与下硬件断点

根据看到的[教程](https://bbs.pediy.com/thread-259962-1.htm)，逆向游戏时可以先搜索屏幕上看到的字符串，然后对字符串所在的内存区域下硬件断点。这点其实让我非常疑惑，理论上说存放字符串的 Buffer 又不会写死在程序里，怎么能保证下一次该函数还是从同样地方读取 / 写入数据呢？不过后续的尝试证明这个用以找函数真的是一个非常好的方式；或许类似 size 的 Buffer  会得到复用吧。我决定从搜索字幕和音频文件文件名入手；顺便一提为了方便检索，游戏语言被设置为英语。

windbg 在内存中搜索的命令为 `s`，且该命令必须指定待搜索的内存区域范围。于是反手一个 `!heap` 查看堆信息，然后分别搜索了下伊庭八郎~~老婆~~的台词:

```shell
0:009> s -a 1a330000 L10000000 "Because of how t"
1ca36521  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
2070ad61  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
2071843d  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
21673ad5  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
21806f71  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
2180822d  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
0:009> s -a 43d40000 L10000000 "Because of how t"
45a80d19  42 65 63 61 75 73 65 20-6f 66 20 68 6f 77 20 74  Because of how t
```
推测这些位置的内存里至少有一部分放的是 `.dat` 等文件的内容。于是我想到，可以再搜索一句台词，其出现位置不变的部分或许是 Buffer 所在的位置：

```shell
0:014> s -a 1a330000 L10000000 "It's a lot more"
1ca38121  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
2070ad61  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
2071843d  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
21673ad5  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
2180f46d  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
21810729  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
0:014> s -a 43d40000 L10000000 "It's a lot more"
45a80f31  49 74 27 73 20 61 20 6c-6f 74 20 6d 6f 72 65 20  It's a lot more 
```
可以看到 `2070ad61, 2071843d, 21673ad5` 三个地址是不变的。对这三个地址分别下硬件断点（断点地址需对齐）：

```shell
0:014> ba w4 2070ad60
0:014> ba w4 2071843c
0:014> ba w4 21673ad4
```
运行发现这三个断点被断到的顺序分别 `1, 2, 0`，第一次 hit 断点 1 时指令为 `MSVCR120!_snprintf+0x95`。让我略感到疑惑的是，在 hit 断点 2 时修改其对应内存位置中字符串的值，当本轮中断结束后三处内存里的字符串都改变了。（原台词为 "Ah, I guess so"）

```shell
0:014> s -a 1a330000 L10000000 "A1, I guess"
2070ad61  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
2071843d  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
21818c25  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
```

这修改也如实地反应在了屏幕上。总之纪念一下：

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/serifu-change.png)

用同样的思路找了下音频文件应该存放的内存位置，下了写断点后发现其只被一个函数 hit，其区域如下图所示:

![](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/software-breakpoint.png)

进一步地探究发现，断对音频文件名区域的写断点时，字幕区域内存已更新。因此决定将软件断点断在 `0xf2a18e`，即 `HakuokiWin!CAsyncReader::GetPin+0x913be`。注意到此时音频文件地址存放在 `esi` 寄存器中。

<br>

##### 1.2 Dump 脚本

将断点断在 `0xf2a18e` 时意外发现 `dr0` 寄存器中存放有台词的地址。虽然对该函数并未进行进一步地逆向，但姑且先 dump 该寄存器以作演示：

```shell
0:015> bp HakuokiWin!CAsyncReader::GetPin+0x913be ".printf \"VOICE: %ma, SCRIPT: %ma\\n\", @esi, @dr0;g"
0:015> g
VOICE: /bgm/BGM01.hca, SCRIPT: 
VOICE: /bgm/BGM02.hca, SCRIPT: 
VOICE: /voice/096001.hca, SCRIPT: "Because of how tense the political situation in#nKyoto is becoming, I thought the city would be a#nbit more hectic, but..."
VOICE: /voice/096002.hca, SCRIPT: "It's a lot more peaceful than I had anticipated.#nI'm quite relieved."
VOICE: /voice/096003.hca, SCRIPT: "What's the matter?"
VOICE: /voice/096004.hca, SCRIPT: "Maybe one of those palm readers.#nIf they're good enough to draw a crowd as big#nas that one, the reader must be pretty good."
VOICE: /voice/096005.hca, SCRIPT: "Why don't you get your palm read?"
VOICE: /voice/096006.hca, SCRIPT: "Oh, come on. It'll be a good thing to get our#nminds off of things.#nOr are you not for it?"
```
此外，如果想将 log 重定向到文件中，可使用如下命令：

```shell
.logappend /specify/path/hakuouki.log
.logclose
```
如果想 printf 位于某一地址的字符串，可使用如下命令：

```shell
.printf "%ma",20709760
```
<br>

##### 1.3 与静态分析结合

看着 `0xf2a18e` 这一地址我就在想，一定有什么公式可以让我找到这个在文件中对应的地址与静态分析工具中通常显示的虚拟地址。然而，忘了 (ಥ﹏ಥ)

于是使用最无赖的方式：搜索二进制命令

```shell
[0x004ba100]> /x 8a068d7601884433ff84c075f3f30f104514
Searching 18 bytes in [0x401000-0x528000]
hits: 1
0x004ba100 hit1_0 8a068d7601884433ff84c075f3f30f104514
```

哟西，其所在函数为 `fcn.004ba050`。剩下的逆向就留待下篇了。

<br>


