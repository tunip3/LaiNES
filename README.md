LaiNES
======

Compact, cycle-accurate NES emulator in ~1000 lines of C++ (well, originally).

![File Browser](http://i.imgur.com/2tuDlCw.png)
![Super Mario Bros. 3](http://i.imgur.com/Gm4QWsE.png)
![Kirby's Adventure](http://i.imgur.com/xA2vwim.png)

![Star Wars](http://i.imgur.com/j3MmRba.png)
![Super Mario Bros.](http://i.imgur.com/yal0ps1.png)
![The Legend of Zelda](http://i.imgur.com/OLO02ij.png)

## Requirements
LaiNES should run on any Unix system that is compatible with the following tools.
- SCons
- C++11 compatible compiler (e.g. clang++)
- SDL2 (including sdl2-ttf and sdl2-image)

## Building and running
Install the dependencies:
```sh
# Arch Linux:
sudo pacman -S clang scons sdl2 sdl2_image sdl2_ttf

# Debian-based systems:
sudo apt-get install clang scons libsdl2-dev libsdl2-image-dev libsdl2-ttf-dev

# Mac OS X:
brew install scons sdl2 sdl2_image sdl2_ttf
```

Compile and run:
```sh
git clone --recursive https://github.com/AndreaOrru/LaiNES && cd LaiNES
scons
./laines
```

## Usage
The emulator comes bundled with a simple GUI to navigate the filesystem and set preferences. Use arrow keys and Enter to operate it. ESC toggles between emulation and menu.

The size of the window and the controls are customizable. LaiNES supports multiple controllers and should work with joysticks as well. The default controls for the first player are as follows:

![Controller Settings](http://i.imgur.com/ERQ2nmJ.png)

## Compatibility
LaiNES implements the most common mappers, which should be enough for a good percentage of the games:
- NROM (Mapper 000)
- MMC1 / SxROM (Mapper 001)
- UxROM (Mapper 002)
- CNROM (Mapper 003)
- MMC3, MMC6 / TxROM (Mapper 004)

You can check the compatibility for each ROM in the following list:
http://tuxnes.sourceforge.net/nesmapper.txt

## Technical notes
The 6502 CPU and the PPU are implemented in just 219 and 283 lines of code respectively.
Meta-programming is used extensively to keep the codebase compact.
Here is a good example of how that is achieved:
```c++
/* Cycle emulation.
 *     For each CPU cycle, we call the PPU thrice, because it runs at 3 times the frequency. */
#define T   tick()
inline void tick() { PPU::step(); PPU::step(); PPU::step(); ... }
...

/* Addressing modes.
 *     These are all the possible ways instructions can access memory. */
typedef u16 (*Mode)(void);
inline u16 imm() { return PC++; }
...
inline u16 zpx() { T; return (zp() + X) % 0x100; }
...

/* Fetch parameter.
 *     Get the address of the opcode parameter in a, and the value in p. */
#define G  u16 a = m(); u8 p = rd(a)
...

/* Instruction emulation (LDx where x is in registers {A, X, Y}).
 *     upd_nz, not shown, just updates the CPU flags register. */
template<u8& r, Mode m> void ld() { G; upd_nz(r = p); }
...

/* Execute a CPU instruction.
 *     Opcodes are instantiated with the right template parameters
 *     (i.e. register and/or addressing mode).*/
void exec()
{
    switch (rd(PC++))  // Fetch the opcode.
    {
        // Select the right function to emulate the instruction:
         ...
         case 0xA0: return ld<Y,imm>();  case 0xA1: return ld<A,izx>();
         ...
    }
}
```

## Known issues
* If you're experiencing audio issues on Linux, try typing `export SDL_AUDIODRIVER=ALSA` before running the emulator.

## Contributors
* [Jeff Katz](https://github.com/kraln) - Mapper 002 & 003, configuration.
* [PudgeMa](https://github.com/PudgeMa) - Scrollable menu and bug fixes.
* [tyfkda](https://github.com/tyfkda) - Show error message instead of segfault for unsupported mappers.

## References and credits
- Special thanks to [Ulf Magnusson](https://github.com/ulfalizer) for the invaluable [PPU diagram](http://wiki.nesdev.com/w/images/d/d1/Ntsc_timing.png) and for his [excellent implementation](https://github.com/ulfalizer/nesalizer) which was a big source of inspiration.
- blargg's APU sound chip emulator: http://blargg.8bitalley.com/libs/audio.html#Nes_Snd_Emu
- Complete hardware reference: http://problemkaputt.de/everynes.htm
- Tick-by-tick breakdown of 6502 instructions: http://nesdev.com/6502_cpu.txt
