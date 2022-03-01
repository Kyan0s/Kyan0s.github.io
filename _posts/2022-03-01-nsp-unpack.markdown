### Switch 游戏解包练习

<br>

#### 0. 废话之缘起

只是一篇毫无技术含量的工具记录。

缘起请见[这篇](https://kyan0s.github.io/2022/02/28/hakuouki-script-sound-extraction.html)的 "缘起" 部分。解包目标为 “虔诚之花的晚钟 1925”，网上找到了个英文版的 `nsp` [文件]( https://drive.google.com/file/d/1wr5o6Xd0YhY_lrVjlwHZoV4X1Fbx4eBl/view?usp=sharing)。从游戏卡带中直接提取游戏文件就姑且作为之后某次 blog 的任务吧。

<br>

#### 1. 解包过程记录

使用集成了 [hactool](https://github.com/SciresM/hactool) 的批处理工具 [NCA-NSP-XCI_TO_LayeredFS](https://gbatemp.net/threads/extract-nsp-nca-xci-update-all-in-one-tool-for-layeredfs.511156/) 将该 `.nsp` 文件解包为下述文件：

```shell
ricordo-de-nsp>ls -alh
total 4.9G
drwxr-xr-x 1 PC 197121    0 Mar  1 18:09 .
drwxr-xr-x 1 PC 197121    0 Mar  1 18:09 ..
-rw-r--r-- 1 PC 197121 1.8K Mar  1 18:03 01009240117a2000000000000000000b.cert
-rw-r--r-- 1 PC 197121  704 Mar  1 18:03 01009240117a2000000000000000000b.tik
-rw-r--r-- 1 PC 197121 135K Mar  1 18:03 1bee509c0d0662e23055e046817062b1.nca
-rw-r--r-- 1 PC 197121 3.5K Mar  1 18:03 26f5040e62bc7008c21b0b5ca1b42624.cnmt.nca
-rw-r--r-- 1 PC 197121 4.9G Mar  1 18:03 487911a628680865bf83ed203f94d219.nca
-rw-r--r-- 1 PC 197121 182K Mar  1 18:03 a8d72150a48761bc03d7f240bd9b1c87.nca
``` 
根据该批处理工具的引导，体积最大的 `487911a628680865bf83ed203f94d219.nca` 将作为后续解包的目标。解包该文件需要 `.tik` 文件中的 `titlekey`，该 `titlekey` 位于偏移 `0x180` 处。下文中所展示的 `titlekey` 为 `6B6BC0EA1C0EEADE4242A04E8C8CBD54`。

```shell
$ r2 01009240117a2000000000000000000b.tik
 -- Sarah Connor?
[0x00000180]> x16
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00000180  6b6b c0ea 1c0e eade 4242 a04e 8c8c bd54  kk......BB.N...T
[0x00000180]>
```

然而，输入 `titlekey` 后报错 `Invalid NCA header!`。搜索这一报错字符串，得到的[结果](https://www.reddit.com/r/SwitchPirates/comments/a6sgfe/invalid_nca_header_are_keys_correct/)大多是：`titlekey` 错了，或这个文件真的 `invalid`。在转而搜索怎么折腾自己的亲卡带之前，我有幸看到了这篇[教程](https://shipengliang.com/games/switch-%E8%A7%A3%E5%8C%85-%E5%9B%BE%E6%96%87%E6%95%99%E7%A8%8B.html)。将教程附件中的 `keys.ini, keys.txt, prod.keys` 三个文件拷贝至批处理文件夹内 `hactool` 同级文件夹内后，对 `.hca` 文件解包成功。

解包后的文件被分别放置在 `exefs` 及 `romfs` 中，后者为资源文件所存放位置：

```shell
D:\Extracted_NCA\romfs\CONTENTS>ls
GAME.cpk  MOVIE.cpk  SOUND.cpk  STORY.cpk  SYSTEM.cpk
```
使用 [CriPakTools](https://github.com/esperknight/CriPakTools) 解包 `.cpk` 文件。声音文件与脚本文件存放位置及格式与解包薄樱鬼时所见大同小异，目前所发现的唯一区别似乎是晚钟 `SCRIPT` 文件夹中无以 `.smsg` 为后缀的文件：

```shell
192:SCRIPT $ tree
.
├── 100.DAT
├── 101.DAT
├── ...
├── 806.DAT
├── 900.DAT
└── init
    ├── 0_init.DAT
    ├── 1_iback.DAT
    ├── 2_ichara.DAT
    ├── 3_ichara.DAT
    ├── 4_iflag.DAT
    └── 6_ichara.DAT
```

<br>

#### 2. 碎碎念

因为觉得声音文件珍贵异常（好吧作为声优劳动成果确实是）而手动 check 了对应关系的我好憨。

通过进一步地[查询](https://bbs.pediy.com/thread-259962-1.htm)得知，对主机游戏的逆向难度远大于对 PC 游戏的逆向。那么，薄樱鬼的 flag 又重新插了起来。

<br>
