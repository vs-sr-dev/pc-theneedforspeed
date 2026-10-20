# pc-theneedforspeed

Toward a native PC port of **The Need for Speed** (Electronic Arts Canada
/ Pioneer Productions, 3DO, 1994): the first Need for Speed, made for the
3DO and on the 3DO before it came to any other machine -- eight real cars
of 1994 (the Diablo, the 512TR, the Viper, the NSX, the Supra, the RX-7,
the Corvette ZR-1, the 911) with Road & Track's data and films, three
point-to-point tracks of three segments each, traffic and the police. The
goal is the game running natively on PC by **static recompilation**: the
3DO original's own ARM60 code turned into C++ and run on a
reimplementation of the 3DO's OS, booting the disc as the console boots
it, in a window in real time, with its sound.

It is the fifth port built on
[3dokit](https://github.com/vs-sr-dev/3dokit), the game-agnostic 3DO
toolkit -- after [pc-immercenary](https://github.com/vs-sr-dev/pc-immercenary),
[pc-crashnburn](https://github.com/vs-sr-dev/pc-crashnburn),
[pc-doctorhauzer](https://github.com/vs-sr-dev/pc-doctorhauzer) and
[pc-escapefrommonstermanor](https://github.com/vs-sr-dev/pc-escapefrommonstermanor)
-- and the first whose game loads code of its own into memory and enters
it. 3dokit is taken here as a submodule: clone with `--recursive`, or run
`git submodule update --init`.

## BYOA — Bring Your Own Assets

This repository contains **documentation and tools only**. No game data, no
executables, no assets, no OS code. You need your own original disc. The
work is done on the US release, one data track, as an ISO of 2048-byte
blocks (327,980 blocks; the port writes its own .cue for it, `iso/nfs.cue`).
Hashes, sizes and addresses are written down here; bytes of the product
are not.

## Layout

    docs/            what the kit reads on this disc, the boot to the race, what the kit got
    replays/         presses (pfboot --pad options), to replay
    3dokit/          game-agnostic 3DO toolkit (submodule)
    iso/, build/     your disc and everything derived from it (ignored by git)

## The route

The route the four other ports have walked: **recompile** the programs
(`3dokit.recomp`, checked by its self-test against an ARM60 interpreter),
**boot** the disc on the Portfolio runtime (`pfboot`) one OS call at a
time, each call the runtime does not answer read in the disc's own OS code
and put in the kit, and **compare and play**, recording the presses to
guard later changes.

The game is two programs: `LaunchMe` (the boot, the race) and `frontovl`
(the front end), which `LaunchMe` reads into memory and enters at its AIF
header itself; both are recompiled, and the runtime recognises the
overlay's image where the game put it (`docs/02`). The disc's own `LMADM`
and `FORMAT` are recompiled too, for the console's NVRAM.

## Tools

The Python tools need Python 3.8+, and capstone for the code; run them from
the repository root. Building the C++ needs CMake, Ninja, clang and SDL3
(MSYS2's mingw64).

```sh
D=iso/nfs.cue        # FILE "nfs.iso" BINARY / TRACK 01 MODE1/2048 / INDEX 01 00:00:00
export PYTHONIOENCODING=utf-8

# the disc: volume, ROM tags, every copy compared; extract it
python -m 3dokit.disc "$D"
python -m 3dokit.disc "$D" --verify
python -m 3dokit.disc "$D" --extract build/disc

# the executables, the pictures, the films, the sound
python -m 3dokit.aif --scan build/disc
python -m 3dokit.cel --check build/disc
python -m 3dokit.stream --scan build/disc
python -m 3dokit.audio --scan build/disc
python -m 3dokit.dsp build/disc/DriveData/dsp --verify

# the code
python -m 3dokit.portfolio build/disc/LaunchMe --sites
python -m 3dokit.recomp.discover build/disc/LaunchMe --report
python -m 3dokit.recomp.discover build/disc/frontovl --report

# the recompiler and its self-test
python -m 3dokit.recomp --out build/recomp --optest \
  "launchme=build/disc/LaunchMe" "frontovl=build/disc/frontovl" \
  "lmadm=build/disc/System/Programs/LMADM" "format=build/disc/System/Programs/FORMAT"
python -m 3dokit.recomp.selftest --image launchme=build/disc/LaunchMe --auto \
  --out build/recomp/selftest/launchme.txt          # and frontovl, lmadm, format the same way
cmake -S build/recomp -B build/recomp-build -G Ninja -DCMAKE_CXX_COMPILER=clang++
ninja -C build/recomp-build
build/recomp-build/selftest build/recomp/selftest/*.txt

# the console's NVRAM, formatted once by the disc's own FORMAT (a console's comes so)
build/recomp-build/pfboot build/disc/System/Programs/FORMAT --disc build/disc \
  --nvram build/nvram --cmdline "format ram 3 0 nvram"

# the disc as the console starts it; to the race, headless, its frames
build/recomp-build/pfboot build/disc --boot --trace 0 --nvram build/nvram --max-calls 2500000 \
  $(cat replays/to-the-race.txt) --frames build/frames --frames-at 2200-30000/30

# in a window, with the pad (arrows; Z X C for A B C; Enter Start; Q W for L R: Z the throttle, W up a gear)
build/recomp-build/pfboot build/disc --boot --trace 0 --nvram build/nvram --window --record build/play.txt
```

## Status

**Session 1**: the repository; the disc read by the kit (846 files; 54
films in a form of DataStream the kit did not read); the OS the same as
Escape from Monster Manor's, Portfolio 21.10; `LaunchMe`, `frontovl`,
`LMADM` and `FORMAT` recompiled with no seeds, the self-test at 0 failures;
and the boot, stop by stop, **from the first call to the race**: the
Pioneer and EA logos, the intro film, the title, the attract loop with the
hosts, the main menu, the loading, and the race -- the chase camera's
fly-by, the cockpit, the car pulling away under A and R. On the way the
kit learnt the game's own display lists (`SubmitVDL`), its overlay entered
at its AIF header, the bitmaps it makes, the audio folio's pause, samples
from AIFF images in memory, the cross product, and the disc's name in
`GetDirectory`. See `TODO.md`.

## Documentation

| | |
|---|---|
| [TODO](TODO.md) | the next session's work, and the history of sessions |
| [01-the-disc-on-the-kit](docs/01-the-disc-on-the-kit.md) | what 3dokit reads on this disc, and what it learnt from it |
| [02-the-boot](docs/02-the-boot.md) | the run to the race; the stops and what they asked; the overlay, the VDLs, the NVRAM |
| [10-3dokit](docs/10-3dokit.md) | what this port gave 3dokit |

## Licence

MIT -- see [LICENSE](LICENSE). The Need for Speed is © 1994 Electronic
Arts; this repository contains none of it.
