# 02. The boot, to the race

The disc booted on the Portfolio runtime (`pfboot build/disc --boot`), one
stop at a time, from the first OS call to the race. Each stop was read in
this disc's own OS code (the same 21.10 images as Escape from Monster
Manor's) or the game's, and answered in the kit as that code does it
(`10-3dokit.md` lists the commits' files). The run's trace and frames are
`build/run*.txt` and `build/frames*`.

## The way the disc starts

The shell runs `startopera`, which runs `$c/lmadm -a ram 3 0 nvram`, then
`AppStartup` (empty), then `$boot/LaunchMe`. `LaunchMe` opens the folios,
makes two screen groups (two screens of 320x240 each), and then **builds
its own display lists**: `SubmitVDL` of an 80-word VDLTYPE_FULL list for
each screen, and `SetVDL` to give each screen its list. It makes its
threads ("Joy", "Timer Task"), configures the event broker, reads its
configuration from `/nvram/Drive`, and loads **`frontovl`**, the front end.

## The stops, in order

| call | stop | what it asked | the answer |
|---|---|---|---|
| 31 | `Graphics -168 SubmitVDL` | a program's own VDL | GRAPHIX 20.45's VDL item (0x52e0, 0x4d7c) and its proof (0x56ec): each entry checked and copied into the OS's VRAM, the last link post-display; `SetVDL` (SWI 47); the display types' fields of the folio's node (0x140, 0x144) |
| 139 | event broker, `cr_QueueMax` | a listener's queue of 20 | the broker stores it in a byte (0x107c): the kit's note that it wrote a word was wrong; read in the 20.21, 20.31 and 21.10 brokers |
| 346 | a jump to 0x67ff0 | the overlay, read into memory by `LaunchMe` and entered at its AIF header | below: the image recognised as the `frontovl` module, its header run |
| 425 | a call to 0x32de0 | a service of `LaunchMe`'s the overlay calls | discovery: the services table (0x3ebf4 on), a table of functions in the read-write data |
| 437 | `FindItem` of subsystem 0 | a debug hook (`swi 0x104` with a flag word in r0, its result discarded) | nothing found, the tags unread |
| 4319 | `CreateItem` of a Bitmap | the game's own bitmap over its screen's buffer | GRAPHIX 20.45's (0x5e80): tags, the widths' table, the clip, the REGCTLs |
| -- | "NVRAM Full: Configuration not saved!" | the NVRAM | below |
| 517120 | `ResumeInstrument` | the audio folio's pause | AUDIOFOLIO 21.10's Pause and Resume (0x1bd4, 0x1c24): the instrument's DSP side out of the DSP's program and back |
| -- | `frontend/display/control.3sh` not found | the directory the game came back to | below: `GetDirectory` |
| -- | `audio -164 DefineSampleHere` | AIFF images in memory (the `*.AIFFfam` families) | 21.10's CreateSample (0x2808): the IFF reader on an image (0xa71c), the data copied out by the game's allocator |
| -- | calls to 0x6dc04, 0x17ebc, 0x2a4d8 | callbacks named only by literal pools: a wrapper ending in a `b`, a frameless `stmfd sp!, {lr}` | discovery: code a pointer names that opens as a function does |
| -- | `DisplayScreen` from the "Sim" thread | a screen the thread did not make | GRAPHIX 20.45's SWI 45 (0x37d8): the caller owns it or has it open, else GRAFERR_NOTOWNER -- the runtime returned the error instead of stopping |
| -- | `Cross3_F16` | Operamath SWI 14 | OPERAMATH 21.10's Green 0x1b48: the cross product on MADAM's matrix engine |

From there on nothing stops: the attract films loop on their own, the
front end takes the pad, and a race runs (`replays/to-the-race.txt`).

## The overlay

`LaunchMe` 0x12d00 reads `frontovl` whole into memory it allocated
(0x32b5c: open, size, allocate, read), and calls 0x3c0b4 on it. That
hand-written routine makes the image's fourth word -- the AIF header's `BL`
to the entry -- `mov pc, r7`, sets `r7` to its own next word (`mov r7,
pc`), and jumps to the image's start: the header's first words run (the
self-relocation, the zero-init) and the patched word comes back. Then it
puts the word back, finds the entry's first `BL` (the C startup's call to
`main`) and returns its target, which `LaunchMe` calls with a block of
pointers to its own data and functions (0x3ebf4). The front end returns to
`LaunchMe` when a race is chosen, and the race runs in `LaunchMe`.

For the kit that asked three things (`10-3dokit.md`): `mov rN, pc` before a
write to pc is a call that returns through rN; a jump to an address no
module knows is looked for as an AIF image in memory -- the module whose
read-only area past the header it is (a new crc32 per module, since the
header has been patched) -- loaded there and its header run; and the
services `LaunchMe` hands over are entries.

## Its own VDLs

The game's two VDLs (80 words each) are full ones: an entry of 240 lines
with the screen's buffer and colour words. GRAPHIX checks every DMA control
word against the bits the hardware defines, the lengths, the display mode,
the lines (an entry of 0 lines takes all that are left), refuses AMY
control words and reserved display-control flags, copies each entry into
the OS's VRAM 16-byte aligned, and links the last to post-display. The
display (`pfboot --frames`) then shows the screens through the game's
lists as it does through the folio's own.

## The NVRAM

The 21.10 `LMADM -a` validates the NVRAM through the mounted `/nvram`; a
blank one is "Not a flat linked-memory filesystem" and LMADM ends there
(Doctor Hauzer's 20.x LMADM runs FORMAT itself; this one does not). The
console's own NVRAM is formatted when it leaves the factory, so the port
formats it once with the disc's own FORMAT (`pfboot --cmdline`, new):

    build/recomp-build/pfboot build/disc/System/Programs/FORMAT --disc build/disc \
      --nvram build/nvram --cmdline "format ram 3 0 nvram"
    Device ram unit 3 has 32768 blocks of 1 bytes each ... Filesystem initialization complete.

From then on the File folio mounts `nvram`, LMADM checks it (exit 0), and
the game makes its `/nvram/Drive` (its options) and reads `/nvram/DriveHS`
(the high scores).

## GetDirectory

The game keeps `GetDirectory`'s path, changes to `drivedata/dsp` to load
its instruments, and changes back to the path. The runtime wrote "" for the
disc's root (it is the runtime's "/"), and `ChangeDirectory("")` stays
where it is: every later relative path failed. On the console the root of
the disc is the mounted filesystem named after its label, "/CD-ROM": the
runtime now writes that (the label read from the disc's block 0, the "Disc
label" file) and takes "/CD-ROM" back to the disc's root. The other ports'
traces move with it (`10-3dokit.md`) -- in the console's direction.

## The race

With Start the attract loop gives way to the title and the main menu;
Start again chooses the default race (the Lamborghini Diablo against the
Ferrari 512TR, the City track): the loading page, the car's slides, the
chase camera's fly-by, the cockpit. The transmission is manual: A is the
throttle, R shifts up -- and the race has no start signal: it waits for
the player's first gear, and everything (the car, the rival, the clock)
sets off with it. With `replays/to-the-race.txt` the car pulls away,
the clock runs, and with no steering it ends against the rock wall of the
first bend, the clock going on.

Headless, the race runs at about 2.5 times the console's speed (26,000
fields in 173 seconds with `--frames` every 30 fields); the attract films
and menus far faster.
