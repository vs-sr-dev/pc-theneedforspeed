# What this port gave 3dokit

3dokit (`3dokit/`, a submodule from
[vs-sr-dev/3dokit](https://github.com/vs-sr-dev/3dokit)) was started by
Immercenary's port and grew on Crash 'n Burn's, Doctor Hauzer's and Escape
from Monster Manor's; The Need for Speed is its fifth game, the second on
Portfolio 21.10 (the same OS images as Monster Manor's), and the first
that loads code of its own into memory and enters it. The submodule was
taken at **42a44c2**; it is at **2ae02a2**.

## Session 1

One commit, **bebc7dc**, made and pushed with the user's word; every
port's submodule then moved to it (pc-crashnburn 8b7d2d3, PC-Immercenary
356a02d, pc-doctorhauzer 730ceb2, pc-escapefrommonstermanor adcbdc8, all
pushed). What it adds:

| file | What |
|---|---|
| `stream.py`, `check.py`, `runtime/tdk_stream.c`, `tdk_stream.h`, `tdkcheck.c` | a DataStream whose first block is one `FILL`, a power of two long, the next block opened by `CTRL`, `FILM` or `SNDS` -- a stream with no header, its block size that `FILL`'s (54 of this disc's films, 32 KiB) |
| `recomp/discover.py` | `mov rN, pc` before a write to pc is a call returning through rN (the overlay's entry); the computed jump `add pc, pc, rX[, lsl #s]`, rX made in the straight line before from `mov #k` or `and rY, #mask` and constants or'd, added, subtracted, and'd or eor'd in, each value a case (an arctangent's octants, a decompressor's unrolled copy); pointed leaves also from a pointer variable whose address a literal pool holds (`ldr r2, =hook; ldr pc, [r2]`), from a table of functions in the read-write data (a relocated word beside one naming a function: the overlay's services), and wherever code a pointer names opens as a function does (`stmfd sp!, {..., lr}`, or a few data-processing words and a `b` to an entry) |
| `recomp/emit.py` | the computed jump as a `switch` on rX |
| `recomp/__main__.py`, `runtime/arm60.h`, `runtime/arm_core.cpp` | each module's crc32 of its read-only area past the AIF header (`crc_body`), and `arm_identify_body` |
| `runtime/pf_os.cpp` | a jump to no entry and no trap that lands on a module's image in memory: the module loaded there (moved, when it was elsewhere) and the image's AIF header run word by word (no-op, BL, `mov pc, rN`, the exit) |
| `runtime/pf_graphics.cpp` | GRAPHIX 20.45's CreateItem of a VDL (`SubmitVDL`, 0x5d80, 0x52e0, 0x4d7c) with the proof and copy of the program's list (0x56ec); `SetVDL` (SWI 47, 0x4e60); CreateItem of a Bitmap (0x5e80); the display types (gf_DefaultDisplayType, gf_DisplayTypeMask, the tables at 0x6e00/0x6e80/0x6ea0); `DisplayScreen` letting through a screen the caller has open, else GRAFERR_NOTOWNER (SWI 45, 0x37d8) |
| `runtime/pf_event.cpp` | `cr_QueueMax` 1 to 20 a listener's queue (`strleb` at +0x60 in all three brokers read; the old note of a word store was wrong) |
| `runtime/pf_kernel.cpp` | `FindItem` of subsystem 0: nothing found, the tags unread |
| `runtime/pf_audio.cpp` | `PauseInstrument`, `ResumeInstrument` (21.10's 0x1bd4, 0x1c24, 0x79b8, 0x7920); `DefineSampleHere` (0x26ec: CreateSample 0x2808 of an image); the IFF reader on an image in memory as well as a file (0xa898 on) |
| `runtime/pf_math.cpp` | `Cross3_F16` (OPERAMATH 21.10's Green 0x1b48) |
| `runtime/pf_file.cpp` | `GetDirectory` names the disc's filesystem as the console mounts it ("/CD-ROM", from the disc's label), and that name at the root is the disc's root |
| `runtime/pf_main.cpp` | `pfboot PROGRAM --cmdline LINE` (FORMAT on a blank NVRAM) |
| `README.md` | this port among the ports; the checks and the gaps |

### Discovery, checked

Over the 19 programs of the six discs (the 17 of `pc-escapefrommonstermanor`'s
`docs/10`, and this disc's `LaunchMe` and `frontovl`), every rule was run
before and after (`discall.sh` in the session's scratchpad): **only this
disc's two programs change** (`LaunchMe` +12 functions and 2 computed
jumps, `frontovl` +2), problems 0 everywhere. A first, wider form of the
table rule took literal pools' words side by side (Crash 'n Burn's
0x26fec, one of its own hand seeds, Doctor Hauzer's `sramtools` 0xec04,
Immercenary's `StorageTuner` 0x70e0); kept to the read-write data it takes
only this disc's table.

### Regression

The other ports built from 42a44c2 and from the working tree
(`regress.sh` in the session's scratchpad), their reference traces run on
both:

* **The same byte for byte**: Crash 'n Burn's `launchme` to call 234 (529
  lines) and `--boot --max-calls 60000 --pad a@1300x1` (125,270 lines);
  Immercenary's `p` (539,774 lines); OMF2097's `LaunchMe` (107 lines).
* **Moved by `GetDirectory`, the console's way**: Immercenary's boot
  (300,000 calls, its replay's presses): the same calls, a few times a
  microsecond later (its directory's path is longer now); Escape from
  Monster Manor's boot (400,000 calls, `--pad a@9000x1`): `progdir` is
  "/CD-ROM" where it was "", the same calls, the allocations after it 4
  bytes on; Doctor Hauzer's boot (400,000 calls, its two Starts): the same
  calls but one -- `launchme` 0x25608 changes to `$boot` only when
  GetDirectory gives "", which the console does not.
* **The generated C++** of every other port: the same but each module's
  table (`crc_body`).
* **The readers**: `stream --scan` the same on the four other trees;
  `tdkcheck` from 42a44c2 and now, and `check.py`, the same on Immercenary's
  and Escape from Monster Manor's images.

Then, after the user's first race in the window and with their word,
**2ae02a2**: a blank NVRAM that `pfboot --nvram DIR` keeps is formatted by
the disc's own FORMAT before the boot ("format ram 3 0 nvram", the line
LMADM gives it), as a console comes from the factory -- 21.10's LMADM does
not format a blank one --, and `--nvram-blank` keeps it blank (`pf_nvram`,
`pf_file`'s `pf_shell_boot`, `pf_main`); the README's ports table mended.
Every port's reference traces the same byte for byte as bebc7dc's (none
keeps an NVRAM); Doctor Hauzer with a blank kept NVRAM: FORMAT, then its
LMADM 20.x finds it valid. Every port's submodule moved (pc-crashnburn
637d3bf, PC-Immercenary 9e0d3fa, pc-doctorhauzer caeec3e,
pc-escapefrommonstermanor e813778, all pushed).
