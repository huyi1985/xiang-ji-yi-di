# I made an NES emulator. Here’s what I learned about the original Nintendo.

> [阅读原文](https://medium.com/@fogleman/i-made-an-nes-emulator-here-s-what-i-learned-about-the-original-nintendo-2e078c9b28fe)

I recently created my own NES emulator. I did it mostly for fun and to learn about how the NES worked. I learned some interesting things, so I wrote this article to share. There is a lot of documentation already out there, so this is just meant to highlight some interesting tidbits. Warning: this will be very technical!


![My emulator can record animated GIFs. Here I am playing Donkey Kong.](https://d262ilb51hltx0.cloudfront.net/max/800/1*pM_a_s1eGZDw2Dz9-Q9U3A.gif)

> My emulator can record animated GIFs. Here I am playing Donkey Kong.

## The CPU
The NES used the MOS 6502 (at 1.79 MHz) as its CPU. The 6502 is an 8-bit microprocessor that was *designed* in 1975. (Forty years ago!) This chip was very popular — it was also used in the Atari 2600 & 800, Apple I & II, Commodore 64, VIC-20, BBC Micro and more. In fact, a revision of the 6502 (the 65C02) is still in production today.

The 6502 had relatively few registers (A, X & Y) and they were special-purpose registers. However, its instructions had several addressing modes including a “zero page” mode that could reference the first 256 words ($0000 — $00FF) in memory. These opcodes required fewer bytes in program memory and fewer CPU cycles during execution. One way of looking at this is that a developer can treat these **256 slots** like “registers.”

The 6502 had no multiply or divide instructions. And, of course, no floating point. There was a BCD (Binary Coded Decimal) mode but this was disabled in the NES version of the chip — possibly due to patent concerns.

The 6502 had a 256-byte stack with no overflow detection.

The 6502 had 151 opcodes (of a possible 256). The remaining 105 values are illegal / undocumented opcodes. Many of them crash the processor. But some of them perform possibly useful results by coincidence. *As such*, many of these have been given names based on what they do.

The 6502 had at least one hardware bug, with indirect jumps. JMP (<addr>) would not work correctly if <addr> was of the form $xxFF. When reading two bytes from the specified address, it would not carry the FF->00 overflow into the xx. For example, it would read $10FF and $1000 instead of $10FF and $1100.

## Memory Map
The 6502 had a 16-bit address space, so it could reference up to 64 KB of memory. But, the NES had just 2 KB of RAM, at addresses $0000 to $0800. The rest of the address space was for accessing the PPU, the APU, the game cartridge, input devices, etc.

Some address lines were unwired, so large blocks of the address space actually mirror other addresses. For example, $1000 to $1800 mirrors the RAM at $0000 to $0800. Writing to $1000 is equivalent to writing to $0000.

![IT’S DANGEROUS TO GO ALONE! TAKE THIS.](https://d262ilb51hltx0.cloudfront.net/max/800/1*TDBeD2Oc_BRd_XI4bJ3Arw.gif)

> IT’S DANGEROUS TO GO ALONE! TAKE THIS.

## The PPU (Picture Processing Unit)
The PPU generated the video output for the NES. Unlike the CPU, the PPU chip was specially-built for the NES. The PPU ran at 3x the frequency of the CPU. Each cycle of the PPU output one pixel while rendering.

The PPU could render a background layer and up to 64 sprites. Sprites could be 8x8 or 8x16 pixels. The background could be scrolled in both the X and Y axis. It supported “fine” scrolling (one pixel at a time). This was kind of a big deal back then.

Both the background and sprites were made from 8x8 tiles. *Pattern tables* in the cartridge ROM defined these tiles. The patterns only specified two bits of the color. The other two bits came from an attribute table. A nametable specified which tiles go where in the background. All in all, it seems convoluted compared to today’s standards. I had to explain to my coworker that it wasn’t “just a bitmap.”

The background was made up of 32 x 30 = 960 of these 8x8 tiles. Scrolling was implemented by rendering more than one of these 32 x 30 backgrounds, each with an offset. If scrolling in both the X and Y axis, up to four of these backgrounds could become visible. However, the NES only supported two, so different mirroring modes were used for horizontal or vertical mirroring.

The PPU contained 256 bytes of OAM — Object Attribute Memory — that stored the sprite attributes for all 64 sprites. The attributes include the X and Y coordinate of the sprite, the tile number for the sprite and a set of flags that specified two bits of the sprite’s color, specified whether the sprite appears in front of or behind the background layer and allowed flipping the sprite vertically and/or horizontally. The NES supported a DMA copy from the CPU to quickly copy a chunk of 256 bytes to the entire OAM. This direct access was about four times faster than manually copying the bytes.

Although the PPU supported 64 sprites, only 8 could be shown on a single scan line. An overflow flag would be set so that the program could handle a situation with too many sprites on one line. This is why the sprites flicker when there is a lot of stuff going on in the game. Also, there was a hardware bug that caused the overflow flag to sometimes not work properly.

*Many games would make changes mid-frame so that the PPU would do one thing for one part of the screen and something else for the other* — often used for split scrolling or rendering a score bar. This required precise timing and knowing exactly how many CPU cycles each instruction used. Things like this make emulation hard.

The PPU had a primitive form of collision detection — if the first (zeroth) sprite intersected the background, a flag would be set indicating a “sprite zero hit.” Only one such hit could occur per frame.

The NES had a built-in palette of 54 distinct colors — these were the only colors available. It wasn’t RGB; the colors in the palette basically spit out a particular chroma and luminance signal to the TV.

![The NES color palette.](https://d262ilb51hltx0.cloudfront.net/max/800/1*e10q3LHPiFB-LyCPXOET5g.png)

> The NES color palette.


## The APU (Audio Processing Unit)
The APU supported two square wave channels, a triangle wave channel, a noise channel and a delta modulation channel.

To play sounds, the game program would write to specific registers (addresses in memory) to configure these channels.

The square wave channels supported frequency and duration control, frequency sweeps and volume envelopes.

The noise channel used a linear feedback shift register to generate pseudo-random noise.

The delta modulation channel could play samples from memory. The SMB3 music has a metal drum sound that used the DMC. TMNT3 had voices like “cowabunga” that used the DMC.

![Balloon Fight.](https://d262ilb51hltx0.cloudfront.net/max/800/1*9hgaviVQ12iOiGLNVyiBCw.gif)

> Balloon Fight


## Mappers
The address space reserved for the cartridge restricted games to 32KB of program memory and 8KB of character memory (pattern tables). This was pretty limiting, so people got creative and implemented mappers.

A mapper is hardware on the cartridge itself that can perform *bank switching* to swap new program or character memory into the addressable memory space. The program could control this bank switching by writing to specific addresses that pointed to the mapper hardware.

Different game cartridges implemented this bank switching in different ways, so there are dozens of different mappers. Just as an emulator must emulate the NES hardware, it must also emulate the cartridge mappers. However, about 90% of all NES games use one of the six most common mappers.

## ROM Files

An .nes ROM file contains the program memory banks and character memory banks from the cartridge. It has a small header that specifies what mapper the game used and what video mirroring mode it used. It also specifies whether battery-backed RAM was present on the cartridge.

## Conclusion

It’s been fun learning about the NES. I’m impressed with what people were able to accomplish with such constrained hardware. It makes me want to write an 8-bit style game now…

I wrote my emulator in Go using OpenGL + GLFW for video and PortAudio for audio. The code is all on GitHub, so check it out:

https://github.com/fogleman/nes


![My favorite: Super Mario Bros. 3](https://d262ilb51hltx0.cloudfront.net/max/800/1*-tgctijO2lfJYqYgkZ3cgg.gif)
> My favorite: Super Mario Bros. 3

## Learn More
* [NES Documentation (PDF)](http://nesdev.com/NESDoc.pdf)
* [NES Reference Guide (Wiki)](http://wiki.nesdev.com/w/index.php/NES_reference_guide)
* [6502 CPU Reference](http://www.obelisk.demon.co.uk/6502/)