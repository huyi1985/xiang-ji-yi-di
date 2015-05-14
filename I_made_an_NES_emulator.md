> 我做了一个 NES 模拟器！我从中学到了关于 Nintendo 的那些事 **[阅读原文 »](https://medium.com/@fogleman/i-made-an-nes-emulator-here-s-what-i-learned-about-the-original-nintendo-2e078c9b28fe)** 

> 以上是推荐者写的

----------

# I made an NES emulator. Here’s what I learned about the original Nintendo.

# 我编写了一个 FC 模拟器！我从中学到了关于 Nintendo 的那些事

> Family Computer（简称FC）是任天堂（Nintendo）公司发行的家用游戏主机。日版 FC 机身以红色和白色为主，因此在华人圈中又有“红白机”的俗称；欧美版 FC 在欧美则称 Nintendo Entertainment System（简称NES）。——译者注

I recently created my own NES emulator. I did it mostly for fun and to learn about how the NES worked. I learned some interesting things, so I wrote this article to share. There is a lot of documentation already out there, so this is just meant to highlight some interesting tidbits. Warning: this will be very technical! 

最近我编写了一个 [FC 模拟器](https://github.com/fogleman/nes "FC 模拟器")。制作这样一个模拟器主要是出于兴趣以及为了从中学习 FC 的工作原理。在这个过程中我学到了很多有趣的知识，于是写下这篇文章同诸位分享我所学到的内容。由于相关的文档已经有很多了，所以这里我只打算讲述一些有趣的特性。请注意，接下来都将是些技术方面的内容。

![My emulator can record animated GIFs. Here I am playing Donkey Kong.](https://d262ilb51hltx0.cloudfront.net/max/800/1*pM_a_s1eGZDw2Dz9-Q9U3A.gif)

> My emulator can record animated GIFs. Here I am playing Donkey Kong. 

> 图1 我的模拟器可以将画面录制成 GIF。这是我正在玩《大金刚》（Donkey Kong）的画面。

## The CPU

## CPU

The NES used the [MOS 6502](http://en.wikipedia.org/wiki/MOS_Technology_6502 "MOS 6502") (at 1.79 MHz) as its CPU. The 6502 is an 8-bit microprocessor that was designed in 1975. (Forty years ago!) This chip was very popular — it was also used in the Atari 2600 &amp; 800, Apple I &amp; II, Commodore 64, VIC-20, BBC Micro and more. In fact, a revision of the 6502 (the 65C02) is still in production today.

FC 使用 MOS 6502（主频1.79MHz）作为其CPU。6502 是一枚诞生于 1975 年（距今已有 40 年之久了）的 8 比特微处理器。在当时这款芯片非常流行，不仅应用于 FC，还被广泛应用于雅达利 2600 & 800、Apple I & II、Commodore 64、VIC-20、BBC Micro等。事实上，直到今天6502的修订版（[65C02](http://en.wikipedia.org/wiki/WDC_65C02 "65C02")）还依然在使用。

The 6502 had relatively few registers (A, X & Y) and they were special-purpose registers. However, its instructions had several addressing modes including a “zero page” mode that could reference the first 256 words ($0000 — $00FF) in memory. These opcodes required fewer bytes in program memory and fewer CPU cycles during execution. One way of looking at this is that a developer can treat these 256 slots like “registers.”

6502 的寄存器相对较少，只有寄存器 A、 X 和 Y 等而且它们都是专用寄存器。尽管如此，其指令却有多种寻址模式。这其中包括一种称为“零页”（Zero Page）的寻址模式，使开发人员可以访问内存中最初的256个字（$0000～ $00FF）。6502 的操作码占用的程序内存较少，执行时花费的 CPU 周期也较短。<span style="color: #ff0000;">// 开发人员可以把零页上的 256 个存储单元看作是 256 个寄存器。</span>

The 6502 had no multiply or divide instructions. And, of course, no floating point. There was a BCD (Binary Coded Decimal) mode but this was disabled in the NES version of the chip — possibly due to patent concerns.

6502 中没有乘法和除法指令，当然也没有浮点数运算指令。虽然有 BCD 码模式，但是在 FC 版的6502中，可能是由于专利问题该模式被禁用了。 

> Binary-Coded Decimal，简称BCD，中国大陆称BCD码或二-十进制编码，是一种十进制的数字编码形式。在这种编码下，每个十进制数字用一串单独的二进制比特来存储表示。通常 4 个二进制数表示 1 个十进制数。——译者注

The 6502 had a 256-byte stack with no overflow detection.

6502 还具有一块不带溢出检测的 256 字节的栈空间。

The 6502 had 151 opcodes (of a possible 256). The remaining 105 values are illegal / undocumented opcodes. Many of them crash the processor. But some of them perform possibly useful results by coincidence. As such, many of these have been given names based on what they do.

6502 拥有 151 条指令（理论上有 256 条指令）。剩余的 105 条都是非法或没有文档的指令，多数会使导致处理器崩溃。但是其中也有一些可能会碰巧产生某种作用，于是大部分这样的指令也会有与其作用相应的名称。

The 6502 had at least one hardware bug, with indirect jumps. JMP () would not work correctly if was of the form $xxFF. When reading two bytes from the specified address, it would not carry the FF->00 overflow into the xx. For example, it would read $10FF and $1000 instead of $10FF and $1100.

6502 至少有一个已知的硬件上的缺陷，例如间接跳转指令的缺陷在于，当 `JMP <addr>` 指令的操作数为形如 $xxFF 的地址时就无法正常工作。因为当从这样的地址读出 2 字节的数据时，该指令无法将低字节 FF 加 1 后（FF -> 00）产生的进位加到高字节上。例如，当从 $10FF 读出2字节的数据时，读取的其实是 $10FF 和 $1000 中的数据，而不是 $10FF 和 $1100 中的数据。

## Memory Map

## 内存映射 

The 6502 had a 16-bit address space, so it could reference up to 64 KB of memory. But, the NES had just 2 KB of RAM, at addresses $0000 to <span style="color: #ff0000;">$0800（$0799?）</span>. The rest of the address space was for accessing the PPU, the APU, the game cartridge, input devices, etc. 

6502 拥有 16 位地址空间，寻址能力为 64 KB。但是 FC 实际只有 2 KB的 RAM（Internal RAM），对应的地址范围是 $0000～$0799。而剩余的地址空间则用于访问 PPU、 APU、游戏卡以及输入设备等。 

Some address lines were unwired, so large blocks of the address space actually mirror other addresses. For example, $1000 to $1800 mirrors the RAM at $0000 to $0800. Writing to $1000 is equivalent to writing to $0000. 

6502 上有些地址总线的引脚并没有布线，所以有很大的一块内存空间实际上都映射到了之前的空间。例如 RAM 中的 $1000～$17FF 就映射到了 $0000～$07FF，这意味着向 $1000 写数据等价于向 $0000 写数据。

![IT’S DANGEROUS TO GO ALONE! TAKE THIS.](https://d262ilb51hltx0.cloudfront.net/max/800/1*TDBeD2Oc_BRd_XI4bJ3Arw.gif)

> IT’S DANGEROUS TO GO ALONE! TAKE THIS.

> 图2 “IT’S DANGEROUS TO GO ALONE! TAKE THIS.”（《塞尔达传说》中的游戏对白）

## The PPU (Picture Processing Unit)

## PPU（图形处理器）

The PPU generated the video output for the NES. Unlike the CPU, the PPU chip was specially-built for the NES. The PPU ran at 3x the frequency of the CPU. Each cycle of the PPU output one pixel while rendering.

PPU 为 FC 生成视频输出。与 CPU 不同，PPU 芯片是为 FC 定制的，其运行频率是 CPU 的 3 倍。渲染时 PPU 在每个周期输出1个像素。

The PPU could render a background layer and up to 64 sprites. Sprites could be 8x8 or 8x16 pixels. The background could be scrolled in both the X and Y axis. It supported “fine” scrolling (one pixel at a time). This was kind of a big deal back then.

PPU 能够渲染游戏中的背景层和最多 64 个卡通图形（Sprite）。卡通图形可以由 8 x 8 或 8 x 16 像素构成。而背景则既可以延水平（X轴）方向卷动，又可以延竖直（Y轴）方向卷动。并且 PPU 还支持一种称为微调（Fine）的卷动模式，即每次只卷动 1 像素。<span style="color: #ff0000;">// 这种卷动模式曾是那些年的一项非常重要的技术。</span>

Both the background and sprites were made from 8x8 tiles. Pattern tables in the cartridge ROM defined these tiles. The patterns only specified two bits of the color. The other two bits came from an attribute table. A nametable specified which tiles go where in the background. All in all, it seems convoluted compared to today’s standards. I had to explain to my coworker that it wasn't “just a bitmap.”

背景和卡通图形都是由 8 x 8 像素的图形块（Tile）构成的，而图形块是定义在游戏卡 ROM 中的 Pattern Table 里的。Pattern Table 中的图形块仅指定了其所用颜色中的最后 2 比特，剩余的 2 比特来自 Attribute Table。Nametable 则指定了图形块在背景上的位置。总之，这一切看起来都要比今天的标准复杂得多，所以我不得不和同事解释说“这不是简单的位图”。

The background was made up of 32 x 30 = 960 of these 8x8 tiles. Scrolling was implemented by rendering more than one of these 32 x 30 backgrounds, each with an offset. If scrolling in both the X and Y axis, up to four of these backgrounds could become visible. However, the NES only supported two, so different mirroring modes were used for horizontal or vertical mirroring.

背景的分辨率为 32 x 30 = 960 像素，由 8 x 8 像素的图形块构成。背景卷动的实现方法是再额外渲染一副 32 x 30 像素的背景，且每次渲染都需要一个偏移量。如果同时沿 X 轴和 Y 轴卷动背景，那么最多可以有 4 幅背景处于可见状态。但是 FC 只支持 2 幅背景，因此游戏中经常使用不同的镜像模式（Mirroring Mode）来实现水平镜像或竖直镜像。

The PPU contained 256 bytes of OAM — Object Attribute Memory — that stored the sprite attributes for all 64 sprites. The attributes include the X and Y coordinate of the sprite, the tile number for the sprite and a set of flags that specified two bits of the sprite’s color, specified whether the sprite appears in front of or behind the background layer and allowed flipping the sprite vertically and/or horizontally. The NES supported a DMA copy from the <span style="color: #ff0000;">CPU？</span> to quickly copy a chunk of 256 bytes to the entire OAM. This direct access was about four times faster than manually copying the bytes.

PPU 包含 256 字节的 OAM（Object Attribute Memory）用于存储全部 64 个卡通图形的属性。属性包括卡通图形的 X 和 Y 坐标、对应的图形块编号以及一组标志位。在这组标志位中，有 2 比特用于指定卡通图形的颜色，还有用于指定卡通图形是显示在背景层之前还是之后，是否允许沿水平和/或竖直方向翻转卡通图形的标志位。FC 支持 DMA 复制，可以快速地将 256 字节从 CPU 可寻址的某段内存（通常是$0200-$02FF——译者注）填充到整个 OAM。像这样直接访问比手工逐字节拷贝大约快 3 倍左右。

Although the PPU supported 64 sprites, only 8 could be shown on a single scan line. An overflow flag would be set so that the program could handle a situation with too many sprites on one line. This is why the sprites flicker when there is a lot of stuff going on in the game. Also, there was a hardware bug that caused the overflow flag to sometimes not work properly.

虽然 PPU 支持 64 个卡通图形，但是在一条扫描线（Scan Line）上只能显示 8 个卡通图形。当一条扫描线上有过多的卡通图形时，PPU 的溢出（Overflow）标志位将被置位，程序可以依此做出相应的处理。这也就是当画面中有很多的卡通图形时，这些卡通图形会发生闪烁的原因。另外，由于一个硬件上的缺陷，会导致溢出标志位有时不能正常工作。

Many games would make changes mid-frame so that the PPU would do one thing for one part of the screen and something else for the other — often used for split scrolling or rendering a score bar. This required precise timing and knowing exactly how many CPU cycles each instruction used. Things like this make emulation hard.

很多游戏会使用一种叫做 mid-frame 的技术，使 PPU 可以在屏幕的一部分做一件事而在另一部分做另一件事。这项技术经常用于分屏滚动画面或刷新分数条。这需要精确的时间掐算以及对每条指令所需 CPU 周期的详细了解。实现类似这样的功能将会加大编写模拟器的难度。

The PPU had a primitive form of collision detection — if the first (zeroth) sprite intersected the background, a flag would be set indicating a “sprite zero hit.” Only one such hit could occur per frame.

PPU 具有一个原始形态的碰撞检测机制。如果第 1 个（编号为0的）卡通图形和背景相交，那么一个标志位将会被置位，表示“卡通图形 0 发生了碰撞”。这种碰撞在每一帧只会发生一次。

The NES had a built-in palette of 54 distinct colors — these were the only colors available. It wasn’t RGB; the colors in the palette basically spit out a particular chroma and luminance signal to the TV.

FC 具有一个内置的 54 色调色板，游戏只能使用这里面的颜色。这些颜色不是 RGB 颜色，基本上只会向电视输出特定的色度（Chroma）和亮度（Luminance）信号。

![The NES color palette.](https://d262ilb51hltx0.cloudfront.net/max/800/1*e10q3LHPiFB-LyCPXOET5g.png)

> The NES color palette.

> 图3 FC的调色板。

## The APU (Audio Processing Unit)

## APU（音频处理器）

The APU supported two square wave channels, a triangle wave channel, a noise channel and a delta modulation channel.

APU 支持 5 个声道，包括 2 个方波声道，1 个三角波声道，1 个噪声声道和 1 个增量调制声道（DMC）。

To play sounds, the game program would write to specific registers (addresses in memory) to configure these channels.

游戏程序需要向指定的寄存器（已映射到内存）写入数据以驱动这些声道发出声音。

The square wave channels supported frequency and duration control, frequency sweeps and volume envelopes.

方波声道支持对频率和时值的控制，以及频率扫描（Frequency Sweep）和音量包络（Volume Envelope）。

The noise channel used a linear feedback shift register to generate pseudo-random noise.

噪声声道可以利用线性反馈移位（Linear Feedback Shift）寄存器生成伪随机的噪声。

The delta modulation channel could play samples from memory. The SMB3 music has a metal drum sound that used the DMC. TMNT3 had voices like “cowabunga” that used the DMC.

增量调制声道（DMC）可以播放内存中的声音样本。例如在《超级马里奥3》中金属鼓的敲击声以及《忍者神龟3》中的语音“cowabunga”使用的都是DMC。

![Balloon Fight.](https://d262ilb51hltx0.cloudfront.net/max/800/1*9hgaviVQ12iOiGLNVyiBCw.gif)

> Balloon Fight 

> 图4 打气球游戏

## Mappers

## Mapper

The address space reserved for the cartridge restricted games to 32KB of program memory and 8KB of character memory (pattern tables). **This was pretty limiting, so people got creative and implemented mappers.**

预留给游戏卡的地址空间是有限的，游戏卡的程序内存（Program Memory）被限制在 32 KB，角色内存（Character Memory）被限制在 8 KB。为了突破这种限制，人们发明了 Mapper。

A mapper is hardware on the cartridge itself that can perform bank switching to swap new program or character memory into the addressable memory space. The program could control this bank switching by writing to specific addresses that pointed to the mapper hardware.

Mapper 是游戏卡中的一个硬件，具有存储体空间切换（Bank Switching）的功能，以将新的程序或角色内存引入到可寻址的内存空间。程序可以通过向指向 Mapper 的特定的地址写入数据来控制存储体空间的切换。

Different game cartridges implemented this bank switching in different ways, so there are dozens of different mappers. Just as an emulator must emulate the NES hardware, it must also emulate the cartridge mappers. However, about 90% of all NES games use one of the six most common mappers.

不同的游戏卡实现了不同的存储体空间切换方案，所以会有十几种不同的 Mapper。既然模拟器要模拟 FC 的硬件，也就必须能够模拟游戏卡的 Mapper。尽管如此，实际上 90% 的 FC 游戏使用的都是六种最常见的 Mapper 中的一种。

## ROM Files
## ROM文件 

An .nes ROM file contains the program memory banks and character memory banks from the cartridge. It has a small header that specifies what mapper the game used and what video mirroring mode it used. It also specifies whether battery-backed RAM was present on the cartridge. 

一个扩展名为 .nes 的 ROM 文件包含游戏卡中的一个或多个程序内存 Bank 和角色内存 Bank。除此之外还有一个简单的头部用于说明游戏中使用了哪种 Mapper 和视频镜像模式，以及是否存在带蓄电池后备电源的 RAM。

## Conclusion

## 结尾 

It’s been fun learning about the NES. I’m impressed with what people were able to accomplish with such constrained hardware. It makes me want to write an 8-bit style game now… 

学习关于 FC 的那些事很有意思，当时的人们能够用如此有限的硬件完成这样一款游戏机给我留下了深刻的印象。接下来我都想开始编写一个 8 比特风格的游戏了。 

I wrote my emulator in Go using OpenGL + GLFW for video and PortAudio for audio. The code is all on GitHub, so check it out: 

我使用 Go 语言编写了我的模拟器，利用 OpenGL 和 GLFW 处理视频，PortAudio 处理音频。模拟器的代码都放到了 GitHub 上，欢迎诸位下载： 

[https://github.com/fogleman/nes](https://github.com/fogleman/nes)

![My favorite: Super Mario Bros. 3](https://d262ilb51hltx0.cloudfront.net/max/800/1*-tgctijO2lfJYqYgkZ3cgg.gif)

> My favorite: Super Mario Bros. 3 

> 图5 我的最爱：《超级马里奥3》

## Learn More

*   [NES Documentation (PDF)](http://nesdev.com/NESDoc.pdf)
*   [NES Reference Guide (Wiki)](http://wiki.nesdev.com/w/index.php/NES_reference_guide)
*   [6502 CPU Reference](http://www.obelisk.demon.co.uk/6502/)