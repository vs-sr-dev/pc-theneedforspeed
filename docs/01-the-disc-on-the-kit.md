# 01. The disc on the kit

What 3dokit reads on The Need for Speed's disc, tool by tool, and what it
did not until this port taught it. Every number below was produced in
session 1 with 3dokit at 42a44c2 plus the changes of `10-3dokit.md`, the
commands as in the README, `PYTHONIOENCODING=utf-8` throughout. The
battery's outputs are kept in `build/battery/`.

## The disc

The user's image is the US release as an ISO (2048-byte blocks) and a
.cue in a 7z archive; the .cue names another file than the one in the
archive, so the port writes its own (`iso/nfs.cue`: `FILE "nfs.iso"
BINARY`, `TRACK 01 MODE1/2048`). The ISO is 671,703,040 bytes, SHA-1
e9117a7dddfd4eba9c053a5b952a85a52a2f324e.

```
nfs.cue: iso 2048, 327680 blocks of 2048 (image 327980), label 'CD-ROM',
  root 1 block(s) x7 copies; 846 files, 31 directories, 530.0 MiB
  rom tag 0f.0d  v0.0   -> System/Kernel/boot_code (5050 bytes, tag says 5168)
  rom tag 0f.07  v0.16  -> System/Kernel/os_code (85896 bytes)
  rom tag 0f.0c  v0.0   offset 0xaae3b77e size 0x0
  rom tag 0f.02  v0.0   -> LaunchMe (296720 bytes, tag counts blocks from block 0)
```

* **`--verify`**: the root's seven copies identical; 33 entries stored more
  than once, 64 extra copies -- 55 identical, 6 directory copies that
  differ only in the case of names, 1 `rom_tags` copy with its own relative
  offsets, and **2 different**: the second and third copies of
  `System/Drivers` give `CPORT49.ROM` 1,400 bytes where the first gives
  1,332, the same block for all three. A mastering slip on a driverlet the
  game does not load; the extracted file is the first copy's.
* **The OS is Escape from Monster Manor's, byte for byte**: `boot_code`,
  `os_code`, `misc_code`, GRAPHIX, AUDIOFOLIO, OPERAMATH, GRAFMATH, the
  shell, the event broker, LMADM and FORMAT all have that disc's SHA-1s --
  Portfolio 21.10, GRAPHIX 20.45 build 419 (pc-escapefrommonstermanor's
  `docs/02`). The unpacked images in `build/os/` are that port's.
* **The tree**: `LaunchMe` (the race engine and the boot), **`frontovl`**
  (the front end, an overlay `LaunchMe` loads itself: `02-the-boot.md`),
  `FrontEnd/` (the menus' pictures in EA's own `.3sh` shapes, the hosts'
  and the cars' pages, the `Trixie.3fn` font), `DriveData/` (`CarData`:
  each car's dashboards, specifications and slides; `DriveArt`; `tracks`:
  nine `.trk` files of 5 to 10 MB, the three tracks' three segments; `aiff`:
  engine, horn and music; and a `dsp/` directory of its own, the System's
  63 instruments and three more), `Movies/` (165 films: the hosts'
  commentary clips, the cars' showcases, the attract films, the logos).
  `AppStartup` is the stock empty one.

## The executables

`aif --scan`: **40 AIF images, 5 compressed, 12 signed, 0 failing**.

| | `/LaunchMe` | `/frontovl` |
|---|---|---|
| file | 296,720 | 120,100 |
| read-only | 0x3db4c | 0x1ae84 |
| read-write | 0x99c4 | 0x1f14 |
| zero-init | 0x14944 | 0x4cc |
| relocations | 1,105 | 436 |
| 3DO binary header | none (node 0) | none (node 0) |
| stack (header) | 0x4e20 | 0x7d00 |
| `arm60 --check` | 62,517 words agree with capstone, 0 disagree | 26,960 agree, 0 disagree |
| discovery | 1,056 functions, 3 switches, 2 computed jumps, 0 problems | 575 functions, 15 switches, 0 problems |
| embedded names | 0 | 0 |

* **No compiler names at all**: both programs were linked without them, so
  the library code (the DataStream, Cinepak, the sound spooler) is known
  only by its shapes. The films are played by `LaunchMe`'s own linked code,
  as on the other discs.
* **Hand-written code**: an arctangent that picks its octant with `add pc,
  pc, ip` (0x3d11c) and a decompressor, EA's, whose unrolled copy is
  entered with `add pc, pc, r1, lsl #1` (0x3d578) -- the two computed jumps
  discovery now follows (`10-3dokit.md`); and the overlay's entry, which
  patches the overlay's AIF header (0x3c0b4).
* **The OS surface** (`portfolio --sites`): `LaunchMe` 250 SWI sites to 38
  entry points and 123 vector sites, every one attributed; `frontovl` 259
  SWI sites to 37 entry points. Worth noting: `SubmitVDL` and `SetVDL` (the
  game's own display lists), `PauseInstrument`, `DefineSampleHere`,
  `CreateFile` and `DeleteFile` (the NVRAM), `Cross3_F16`, the "mac" device
  looked for.

## Pictures, films, sound

* **`cel --survey`/`--check`: 127 files, 127 frames, 127 decoded**: 112 16-bit
  uncoded literal (56 with BGND), 10 16-bit uncoded packed, 4 4-bit and 1
  8-bit coded packed; `XTRA` after 15 CCBs. Most of the game's pictures are
  in its own containers (`.3sh`, the `*Fam` families, the tracks), which
  the game reads itself.
* **`stream --scan`: 165 streams** (111 before this port): **54 of the films
  open with a whole block of `FILL`** -- 32 KiB of nothing, the stream's
  first `CTRL` in the second block -- a DataStream with no header of a form
  no other disc has; the kit now reads it (`10-3dokit.md`). 27,881 Cinepak
  frames in all (14,344 before), 320x240 or 320x192 at a tick scale of 15
  or 30, SDX2 stereo at 22,050 Hz in all but one.
* **`audio --scan`: 19 AIFF files, 0 failing**: the cars' engine notes and
  the music (`*.22k.mw.aif` 16-bit mono, `*.44K.sw.aifc` SDX2 stereo), and
  `sinewave.aiff` (one sustain loop). The engines' and the front end's
  effects are inside `*.AIFFfam` files, families of AIFF images the game
  hands the audio folio from memory (`DefineSampleHere`).
* **`dsp --verify`**: `System/Audio/dsp` 63 instruments, 219 knobs, 669
  relocations; `DriveData/dsp` 66 (the same and `setmem`, `sinesamp`,
  `thru`); every file walked to its last byte.
* The runtime's check, in Python and in C (`check`, `tdkcheck`): the same
  127 frames and 165 streams, 0 lines differ.

## The recompiler

```
launchme   1056 functions    59480 instructions
frontovl    575 functions    24452 instructions
lmadm        94 functions     4002 instructions   (the disc's System/Programs/LMADM, 21.10)
format       35 functions     1085 instructions   (System/Programs/FORMAT)
self-test: 1,365 functions, 17,862 vectors, 0 failures (optest 891/10,580, launchme 316/4,796,
           frontovl 128/2,016, lmadm 22/344, format 8/126)
```

No seeds: every entry discovery did not see at first (the C library's
`return 0` stub behind a hook, the services table `LaunchMe` hands its
overlay, wrappers and frameless callbacks named only by pointers) is now
found by a rule of the kit's (`10-3dokit.md`).

## What the kit does not read here

The game's own formats, which the recompiled game reads itself: `.3sh`
(EA's shapes), `.3fn`, the `*Fam` families, `.trk`, `.spec` and the other
car data, `AllConstants.data`, `xtracar.cfg`, `FrontEnd/InitHS`.
