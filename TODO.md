# Next session

The top of this file is the next session's work; the history of sessions
is at the bottom. Read the top, not the history.

## Where the port stands

| | |
|---|---|
| the disc on the kit | 846 files, 165 streams (54 of a new form), 127 cels, 19 AIFF, battery clean (`docs/01`) |
| the OS | Escape from Monster Manor's images, byte for byte: Portfolio 21.10, GRAPHIX 20.45.419, AUDIOFOLIO 21.10, OPERAMATH 21.10 |
| recompiled | `launchme` 1,056 functions, `frontovl` 575, `lmadm` 94, `format` 35; no seeds; self-test 0 failures (1,365 functions, 17,862 vectors) |
| `pfboot build/disc --boot --nvram build/nvram` | logos, intro, title, the attract loop with the hosts' clips, on for ever with no stop |
| `... $(cat replays/to-the-race.txt)` | Start to the title and the menu, Start into the default race (Diablo against 512TR, City), A the throttle and R first gear: the car drives off, the clock runs (`docs/02`) |
| speed, headless | the race about 2.5 times the console's (26,000 fields in 173 s with frames every 30 fields) |
| the kit | **bebc7dc** in every port (`docs/10`): regressed on every port, committed and pushed with the user's word |
| this repository | published at the end of session 1 (vs-sr-dev/pc-theneedforspeed), the user's word: "when we see something" |

```sh
# the port's build from the kit's working tree (session 1's scratchpad build.sh does this)
cd D:/Homebrew6 && PYTHONIOENCODING=utf-8 python -m 3dokit.recomp --out pc-theneedforspeed/build/recomp --optest \
  "launchme=pc-theneedforspeed/build/disc/LaunchMe" "frontovl=pc-theneedforspeed/build/disc/frontovl" \
  "lmadm=pc-theneedforspeed/build/disc/System/Programs/LMADM" "format=pc-theneedforspeed/build/disc/System/Programs/FORMAT"
export PATH=/c/msys64/mingw64/bin:$PATH                 # for cmake/ninja/clang only
cmake -S build/recomp -B build/recomp-build -G Ninja -DCMAKE_CXX_COMPILER=clang++ && ninja -C build/recomp-build
cp build/nvram-formatted.bin build/nvram/nvram.bin     # FORMAT's fresh NVRAM, kept aside
build/recomp-build/pfboot build/disc --boot --trace 0 --nvram build/nvram --max-calls 2500000 \
  $(cat replays/to-the-race.txt) --frames build/frames --frames-at 2200-30000/30
```

Session 1's scratchpad (`0d1f8f8d-.../scratchpad`): `build.sh [--selftest]`,
`discall.sh KITPARENT OUT` (discovery over the 19 programs of the six
discs), `regress.sh VARIANT KITPARENT` (the other ports' builds and
reference traces; `reg/out-old` is 42a44c2's, `reg/out-new` the commit's),
`callseq.py` (two traces compared call by call), `odis.py IMAGE ADDR [N]`
(a raw image disassembled, `BASE=10000` for the kernel), `g419.dis`,
`af2110.dis`, `k2110.dis`, `eb2110.dis` (GRAPHIX 419, AUDIOFOLIO, the
kernel and the event broker 21.10 whole), `relocleaves.py`,
`neighbours.py`, `dbg_leaf.py` (discovery's pointed leaves looked at),
`poorprof.sh`, `poorprof2.sh` (a profile by gdb's backtraces;
`profbuild/` the build with symbols), `sheet.py`, `ppm2png.py`.

## The work, in order

### 1. Play it in the window

The race runs headless; next is the user at the pad (`--window`, the keys
in the README): the front end's menus and options (the car, the track, the
transmission), a whole race with steering, the police, the traffic, the
replay; record the presses (`--record`) and keep them in `replays/` to
guard later changes. Every stop on the way is read as before. Worth
watching: the sound in the race (the engine is `varmono16`, `multiply`,
`mixer8x2`, `halfmonosample` and `dcsqxdstereo` through the DSP's
interpreter: no hand-written model matches their code), the frame rate
against the console's, the dashboard's needles, the rear-view mirror.

### 2. Phoenix

The user's captures of the same moments (the title, the menu, the race's
start) against the runtime's frames, as for the other ports.

### 3. The race's speed in the window

Headless the race runs at about 2.5 times the console's; the cost is the
cel engine's projector (half the time) and the DSP's interpreter (a third)
-- `poorprof.sh`. If the window falls behind, models of the engine's DSP
instruments (checked with `--dsp-check` as the others are) and a faster
path in `pf_cel` (the frame buffer's words read and written directly) come
first.

### 4. On from there

* The cel engine's time (pc-escapefrommonstermanor's TODO #1): this game
  draws hundreds of cels a frame; its pace on the console is the one to
  keep.
* `SendIO` from the "Sim" thread on an IOReq of `LaunchMe`'s main task
  gives NOTOWNER, as 21.10's kernel does (0x13e8c compares the owner with
  the current task alone); the game draws its full background over the
  frame anyway. Worth a look on the console whether anything is missing.
* `DisplayScreen` from the "Sim" thread passes; GRAPHIX 20.45's own test
  (0x37d8) and a thread's own resource table say why only in part -- read
  the kernel's thread creation (0x16860 on) for what a thread inherits.
* The display control tags of a VDL (`CREATEVDL_TAG_*` 11 to 22) and
  `ModifyVDL` stop with "not yet" when reached.

## Answered

* (Session 1) The kit's commit and this repository: the user's word in
  the session's brief ("repo BYOA pc-theneedforspeed con topic quando
  vediamo qualcosa ... e aggiornamento di 3dokit").

## Questions for the user

* The NVRAM: a console's comes formatted; the port formats a blank one
  with the disc's own FORMAT (`docs/02`). Fine as the way, or should
  `pfboot` do it when the NVRAM is blank?

## Keep in mind

* **Chat in Italian, every file in English.** No game data, no OS bytes in
  any repository; hashes, sizes and addresses are fine.
* `PYTHONIOENCODING=utf-8` for the Python tools; Python with the normal
  PATH (msys's has no capstone); mingw64 on PATH only for cmake, ninja,
  clang. C++ or Python with `\n` or `\\` through Write or Edit, never a
  shell heredoc.
* Always `--max-calls`; never `--trace 1` a long run into a file (the
  race's cel lines run to hundreds of megabytes) -- grep the stream.
* A frame is written only when the picture changes: a still race (the
  start, waiting for the throttle) writes none, and is not a stall.
* `frontovl` is loaded where `LaunchMe`'s allocator puts it (0x67ff0 in
  the runs so far); its functions' addresses in a trace are that plus the
  module's.

# History

## Session 1 (2026-10-20) -- the repository, the disc on the kit, the boot to the race

* **The repository**: git, the BYOA `.gitignore` of the sister ports (and
  the 7z the disc came in), MIT, the README, 3dokit as a submodule.
* **The disc** (`docs/01`): the user's ISO and its own .cue; 846 files,
  `--verify` clean but for one mastering slip in `System/Drivers`; the
  battery: 40 AIF images, 0 disagreements with capstone, discovery with 0
  problems; 54 films in a DataStream form the kit did not read, read now.
  The OS: Escape from Monster Manor's, byte for byte.
* **The recompiler**: `LaunchMe`, `frontovl`, LMADM, FORMAT, no seeds;
  self-test 0 failures. Discovery learnt the arctangent's and the
  decompressor's computed jumps, a hook behind a pointer variable, the
  overlay's services table, wrappers and frameless callbacks.
* **The boot** (`docs/02`): to the attract loop, the menu and the race;
  the stops answered in the kit: the game's own VDLs, the event broker's
  queue, the overlay entered at its AIF header, FindItem's debug hook,
  CreateItem of a bitmap, the NVRAM (formatted by the disc's FORMAT),
  Pause/ResumeInstrument, GetDirectory's "/CD-ROM", DefineSampleHere,
  DisplayScreen from a thread, Cross3_F16.
* **The kit**: regressed on Crash 'n Burn, Immercenary, OMF2097, Doctor
  Hauzer and Escape from Monster Manor (`docs/10`); committed and pushed,
  every port's submodule moved.
* **The repository** published on GitHub.
