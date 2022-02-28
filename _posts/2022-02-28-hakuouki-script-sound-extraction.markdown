### 解包薄樱鬼风之章语音集

<br>

#### 0. 废话之缘起

塑料日语全靠游戏语音学来的我被总司的声音麻得酥酥的。再一想游戏安全好歹也是安全重要分支，想入门不如直接从扒了老婆（？）下手…… 于是就有了这篇满纸荒唐言一把辛酸泪的血泪之书。

说明一下，目标是摘出游戏中的语音和日语字幕，并且全自动地将其对应上。因为各种探索实在花了太多时间了，“对应”这个目标并没有实现，只做到了从源文件中摘出了出现的可能是字幕的字符串，然后手动 check 是否和音频顺序对应。菜，太菜了，虽然 check 过程中听力水平突飞猛进但还是觉得好丢人啊…… 下一个想拆的游戏是虔诚之花的晚钟（我永远喜欢尼古拉~~斯·赵四~~），到时一定要实现这个目标 (＞﹏＜)

<br>

#### 1. 一些准备工作

##### 1.1 获取资源文件

拆包的是 PC 上 steam 下载在其默认游戏安装目录内的游戏，位置在 `C:\Program Files (x86)\Steam\steamapps\common\Hakuoki Kyoto Winds\data`。

目录内文件列表如下（此处以设置为中文字幕时的文件列表为例，~~steam 你又把我的日语语音包删了吗岂可修~~）：

```shell
C:\Program Files (x86)\Steam\steamapps\common\Hakuoki Kyoto Winds\data>ls -alh
total 18G
drwxr-xr-x 1 PC 197121    0 Feb 26 19:29 .
drwxr-xr-x 1 PC 197121    0 Feb 26 19:29 ..
-rw-r--r-- 1 PC 197121 1.9K Feb 18 22:31 BGM.xsb
-rw-r--r-- 1 PC 197121 101M Feb 18 22:04 BGM.xwb
-rw-r--r-- 1 PC 197121  415 Feb 18 22:31 CL3SE.xsb
-rw-r--r-- 1 PC 197121 553K Feb 18 22:04 CL3SE.xwb
-rw-r--r-- 1 PC 197121 5.9K Feb 18 22:31 DLCVoice.xsb
-rw-r--r-- 1 PC 197121  20M Feb 18 22:04 DLCVoice.xwb
-rw-r--r-- 1 PC 197121 204K Feb 26 18:43 GAME_CN.cpk
-rw-r--r-- 1 PC 197121 1.5G Feb 26 18:43 GAME_CN00000.pac
-rw-r--r-- 1 PC 197121 1.5G Feb 26 18:43 GAME_CN00001.pac
-rw-r--r-- 1 PC 197121 1.5G Feb 26 18:43 GAME_CN00002.pac
-rw-r--r-- 1 PC 197121 1.5G Feb 26 18:43 GAME_CN00003.pac
-rw-r--r-- 1 PC 197121  34M Feb 26 18:43 GAME_CN00004.pac
-rw-r--r-- 1 PC 197121  547 Feb 18 22:59 HakuokiWindSteamSound.xgs
-rw-r--r-- 1 PC 197121 2.3K Feb 18 22:58 LOOPSE.xsb
-rw-r--r-- 1 PC 197121  17M Feb 18 22:04 LOOPSE.xwb
-rw-r--r-- 1 PC 197121  97K Feb 18 22:59 MOVIE_CN.cpk
-rw-r--r-- 1 PC 197121 1.6G Feb 26 18:43 MOVIE_CN00000.pac
-rw-r--r-- 1 PC 197121 551M Feb 26 18:43 MOVIE_CN00001.pac
-rw-r--r-- 1 PC 197121 1.3M Feb 18 22:04 SOUND.cpk
-rw-r--r-- 1 PC 197121 1.2G Feb 18 22:04 SOUND00000.pac
-rw-r--r-- 1 PC 197121  13K Feb 18 22:44 SSE.xsb
-rw-r--r-- 1 PC 197121  21M Feb 18 22:04 SSE.xwb
-rw-r--r-- 1 PC 197121 108K Feb 26 19:21 STORY_CN.cpk
-rw-r--r-- 1 PC 197121  22M Feb 26 18:43 STORY_CN00000.pac
-rw-r--r-- 1 PC 197121 115K Feb 18 22:45 SYSTEM.cpk
-rw-r--r-- 1 PC 197121 163M Feb 18 22:04 SYSTEM00000.pac
-rw-r--r-- 1 PC 197121 114K Feb 26 19:28 SYSTEM_CN.cpk
-rw-r--r-- 1 PC 197121  56M Feb 26 18:43 SYSTEM_CN00000.pac
-rw-r--r-- 1 PC 197121 107K Feb 18 22:44 SYSTEM_COMMON.cpk
-rw-r--r-- 1 PC 197121 3.1M Feb 18 22:04 SYSTEM_COMMON00000.pac
-rw-r--r-- 1 PC 197121  573 Feb 18 23:12 SystemSE.xsb
-rw-r--r-- 1 PC 197121 1.2M Feb 18 22:04 SystemSE.xwb
-rw-r--r-- 1 PC 197121 619K Feb 18 22:04 VOICE.xsb
-rw-r--r-- 1 PC 197121 481M Feb 18 22:04 VOICE.xwb
```

可以看到文件夹内基本只有两类后缀，且每类后缀里第二个后缀（以下简称`胖后缀`）的文件体积要（远）大于第一种后缀（以下简称`瘦后缀`）的文件体积：

+ `xsb` and `xwb` 
+ `cpk` and `pac`

推测胖后缀的是游戏资源而瘦后缀的是游戏资源索引。然而查了一下基本网上的资料全围绕胖后缀展开，让一个刚入门的萌新在找不到资料的情况下独立研究瘦后缀又过于残忍，于是我就先尝试了对胖后缀的解包。另外之后通过十六进制编辑器查看某瘦后缀时发现其确实像是对应胖后缀索引的证据（请见 1.3 节），但并未深入确认这一点。

<br>

##### 1.2 解包血泪碎碎念

根据我的浅薄理解，游戏公司似乎对游戏后缀的设置相当自由，不同游戏公司（甚至同一游戏公司内部的不同游戏中）同样后缀名的文件未必能通过同一种方式解包。因此**搜索解包方式时注意带上游戏名称**是至关重要的。

本文中对资源文件的解包用到了通用解包器引擎 `QuickBMS` 及网络热心大佬们提供的对应 `.bms` 脚本。实话说通用解包器引擎这种想法非常吸引我，但囿于时间有限我还没来得及研究这个。如果仅考虑其用法的话，似乎可以用下述比较简单的方式查询某一类型的文件对应的解包脚本（当然能不能成功解包还得看是不是真正适用）：

+ 获取文件格式幻数。通常（应该没有例外……吧？）位于文件开头。以后缀名为 `.cl3` 的资源文件为例，其幻数为 `CL3B`。(至于这种文件具体是干啥的，标准回答：我不知道 o(TヘTo)）

```shell
$ r2 1000.cl3
 -- You need some new glasses
[0x00000000]> x48
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00000000  434c 3342 0000 0000 0000 0003 0000 0002  CL3B............
0x00000010  0000 0040 0000 0000 ffff ffff 0000 0000  ...@............
0x00000020  0000 0000 0000 0000 0000 0000 0000 0000  ................
[0x00000000]>
```

+ 查询幻数对应的 `.bms` 脚本。事实上脚本文件中通过 `idstring` 指定了期望文件幻数：

```shell
endian big
idstring "CL3B"
```

<br>

##### 1.3 解包音频部分

`.wxb` 文件为音频文件的压缩包，根据文件名选择 `VOICE.xwb` 作为解包目标。使用 [TConvert](https://github.com/trigger-segfault/TConvert) 进行解包，得到一摞 `.wav` 文件。不过解包出来的文件是没有真实文件名的，并且该可爱小工具会对前 41 个文件命名为默认文件名。~~哦我的上帝，这真是太可怕了~~

```c#
private static string[] TrackNames = {
    "01 Overworld Night",
    "02 Eerie",
    ...
    "40 Sandstorm",
    "41 Old One's Army"
};
```

没有文件名显然不利于我寻找文字和语音的对应关系。然而意外的是，解包另一种 `.pac` 资源文件时发现其包含有声音文件（虽然是 `.hca` 格式），且上述声音文件带有文件名。`VOICE.xwb` 中解包出的声音文件数量与解包 `.pac` 后得到的声音文件数量相等；随机从两边各抽了位于同样位置的几个音频文件，听了下内容也是相同的。因此解包 `xwb` 文件得到的音频文件最终惨遭无视。（对 `.pac` 文件的解包过程请见 1.4 节）

从 `.pac` 中解包的语音文件画风如下：

```shell
D:\gameDe\kaze\de_pac\voice>ls | head
010001.hca
010002.hca
010003.hca
010004.hca
010005.hca
010006.hca
010007.hca
010008.hca
010009.hca
010010.hca
```
语音文件的编号似乎遵循如下规则：

+ 最左两位表示角色编号。如 `01` 表示土方岁三，`12` 表示风间千景。
+ 右边四位的编号则较为混乱。理论上说第三位有 `0` 和 `3` 的差别，编号为 `3` 的语音似乎较为特殊（如为经过处理的声音，表示该角色从门外 / 墙外发声等）。但如果某一角色语音过多时，右面整个四位都会被用来编号，比如某话痨副长编号一路猛冲到了 `011271`。

顺便一提，使用 [CriTools](https://github.com/kohos/CriTools) 将 `.hca` 文件转化为了 `.wav` 文件。

基于上文提到的“瘦后缀是胖后缀的索引”这一猜测，拿到文件名后开始在 `VOICE.xsb` 搜索，果然有所收获。推测该文件中音频文件出现的顺序能够帮助我将音频和文字进行对应，可惜囿于时间有限并未深挖。

```shell
$ r2 VOICE.xsb
 -- r2 is for the people
[0x00084940]> x48
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00084940  ffff 3031 3030 3036 0030 3130 3030 3300  ..010006.010003.
0x00084950  3939 3031 3139 0030 3130 3030 3100 3031  990119.010001.01
0x00084960  3030 3032 0030 3130 3030 3400 3031 3030  0002.010004.0100
[0x00084940]>
```
<br>

##### 1.4 解包文字部分

目标为 `STORY_CN00000.pac`，所用 `QuickBMS` 脚本为 [neptunia_rebirth1.bms](https://aluigi.altervista.org/bms/neptunia_rebirth1.bms)。该文件解包后画风如下：

```shell
.
├── jp
│   ├── 100.dat
│   ├── 101.dat
│   ├── ...
│   ├── 913.dat
│   └── 99.dat
├── init
│   ├── 0_init.dat
│   ├── 1_iback.dat
│   ├── 2_ichara.dat
│   ├── 3_ichara.dat
│   └── 4_iflag.dat
└── msg
    ├── file_list.txt
    ├── scriptmsg_099.smsg
    ├── scriptmsg_100.smsg
    ├── ...
    ├── scriptmsg_530.smsg
    ├── scriptmsg_531.smsg
    └── scriptnamelist.smsg
```

 `jp`（或对应的语言名称）及 `msg` 文件夹下同样编号的文件成对应关系。令人疑惑的是，在 .`dat` 与 `.smsg` 文件中都能搜索到字幕台词，而查到的一些本地化大佬的[问答](https://gbatemp.net/threads/need-help-with-text-extraction-insertion-from-file-in-psp-game.369980/)却没有提到对 `smsg` 文件的修改。于是我对照了序章及 `99.dat` 中出现的台词，发现该 `.dat` 文件并未出现台词缺失的情况。考虑到资料多寡，最终选择对 `.dat` 文件进行研究。
 
`.dat` 文件幻数为 `STCM2L`。有趣的是，[neptools](https://github.com/u3shit/neptools/blob/master/doc/formats/stcm.md) 称 `STCM files store bytecode for a VM used by the Re;Births`。这个说法最终让我意识到**想精确地解读这份文件可能少不了对游戏程序的逆向**。可惜一开始我并未选择这么做；事实上本文最终也不涉及逆向的部分。简单来说，在（暴躁地）啃完文档啃代码后，我并没有找到精确的语音（名称或什么索引）与文本的对应的数据结构。于是我决定采用最暴力的直接提取文字然后手动对应这种方式。

根据 [hkki](https://github.com/lnz/hkki/blob/master/README)，字符串对应的头结构如下：

```shell
* Strings:
    Local pointers usually point to larger blocks of data that you want 
    to pass. This most likely will be a string. A string type in STCM2L 
    looks like this:
        32 bit: zero (0x00000000)
        32 bit: amount of 4 byte blocks used (strlen/4)
        32 bit: one (0x00000001)
        32 bit: string length
        followed by the string
```
以下为某英文脚本资源文件内某字符串，其中 `0x52fb4 至 0x52fc4` 的部分为字符串头，该字符串头指示字符串部分长度为 `0x2c` 字节：

```shell
$ r2 099.dat
 -- Coffee time!
[0x00052fb0]> x64
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00052fb0  0000 0040 0000 0000 0b00 0000 0100 0000  ...@............
0x00052fc0  2c00 0000 736d 696c 6564 2c20 616e 6420  ,...smiled, and
0x00052fd0  6c65 6420 7573 2064 6f77 6e20 7468 6520  led us down the
0x00052fe0  7374 7265 6574 2ee3 8080 e380 8000 0000  street..........
```

可惜的是，所有字符串均通过这种方式存储，包括角色名及选择肢等；直接提取字符串画风如下：

```shell
原田 左之助
「……それが俺たちの仕事だ」
雪村 #Name[1]
「……はい」
hara_e
原田さんに言われると、
なんだか少しだけ安心できた。
きっと、大丈夫……。
私と原田さんは、
皆さんの無事を信じて待ち続けた。
switch
sure36
switch
```
此外，注意到能够从字符串本身区分对话与旁白（日语中角色发言会被`「」`包裹）。接下来的提取就是纯工程问题了， 此处不再赘述。

<br>

#### 2. 碎碎念

可以的话请用游戏默认角色名，不然老婆含情脉脉地念你的名字时会由 千鶴ちゃん 变成 おまえさん。doge

<br>

