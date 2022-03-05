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

根据看到的[教程](https://bbs.pediy.com/thread-259962-1.htm)，逆向游戏时可以先搜索屏幕上看到的字符串，然后对字符串所在的内存区域下硬件断点。这点其实让我非常疑惑，理论上说存放字符串的 Buffer 又不会写死在程序里，怎么能保证下一次该函数还是从同样地方读取 / 写入数据呢？不过后续的尝试证明这个找函数真的是一个非常好的方式，或许对于类似 size 的 Buffer  会得到复用吧。我决定从搜索字幕和音频文件文件名入手；顺便一提为了方便检索，游戏语言被设置为英语。

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
运行发现这三个断点被断到的顺序分别 `1, 2, 0`，第一次 hit 断点 1 时指令为 `MSVCR120!_snprintf+0x95`。让我略干到疑惑的是，在 hit 断点 2 时修改其对应内存位置中字符串的值，当本轮中断结束后三处内存里的字符串都改变了。（原台词为 "Ah, I guess so"）

```shell
0:014> s -a 1a330000 L10000000 "A1, I guess"
2070ad61  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
2071843d  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
21818c25  41 31 2c 20 49 20 67 75-65 73 73 20 73 6f 2c 20  A1, I guess so, 
```

这修改也如实地反应在了屏幕上。总之纪念一下：

![image](https://raw.githubusercontent.com/Kyan0s/Kyan0s.github.io/main/assets/img/serifu-change.png)


