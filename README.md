# Vice-Tay

Vice-Tay runs an Xbox Series title natively on Windows, from files you supply,
through [VIctory](https://github.com/Marvisak/VIctory), a compatibility layer:
it loads the console executable, answers its system calls and translates its
GPU work to Direct3D 12. This tree is tuned and fixed on one PC, an RTX 4090
with a Ryzen 9 9950X3D and 48 GB of RAM, toward one goal: the open world at
30 frames a second, with the picture right, fully playable.

**No game files, code or assets are in this repository.** You need your own
copy of the game. Vice-Tay is not affiliated with or endorsed by the game's
developer or publisher.

> [!TIP]
> **[View the gameplay screenshots](https://github.com/TaynanGT/Vice-Tay-Screenshots#readme)**
>
> Ten captures from an earlier Series S playtest, on build `672cf831`.

## At a glance

- **It plays.** On the Series S profile the prologue runs from the prison
  through the boat section to an island, driven by hand, with failed attempts
  picked up again from checkpoints ([Gameplay](#gameplay)).
- **It is not fast enough yet.** 21.1 fps at the median over the latest
  30-minute session, short of the 30 fps goal ([Tests](#tests)).
- **Every change is tested twice:** by CI, and by playing it on this PC with
  the frame rate, memory and card measured from the game's own frame lines.

## Gameplay

What a session plays, on the Series S profile, as in the latest 30-minute
session (minutes in the world):

1. **Loading.** The start screen after 72 s, the world after 120 s.
2. **The prison and the ride out** (0-4 min). Cutscenes carry the story from
   the prison to a boat on open water.
3. **The boat, by hand** (4-25 min). Following a plane to the package it drops,
   with Focus to find it, then the route the map draws: under a road bridge and
   an iron bridge, along a mangrove channel, past a broken pier and across a bay
   to an island with a crashed plane.
4. **Failing and trying again.** Running aground ends the mission ("the boat is
   no longer usable"). The game offers to repeat the checkpoint, skip it or
   restart the mission; repeating and skipping both work, and the run goes on.
5. **On foot** (25-30 min). Off the boat on the island's beach, with the
   inventory tutorial; the pause menu opens and closes.

The HUD, minimap, route, mission prompts and subtitles work throughout. Since
[#57](https://github.com/TaynanGT/Vice-Tay/pull/57) a performance panel sits in the top-left corner of every session: FPS and
frame times with a graph, resolution, draws, GPU and CPU with their load, VRAM
and RAM. **F3** hides it, `-NoPanel` starts without it. Every session is
recorded to video on the card's encoder.

### Series S gameplay clip

The boat section on the RTX 4090 test PC, with the performance panel visible.

https://github.com/user-attachments/assets/bd07891c-c53f-4f46-9b7e-80463c379b84

## Tests

Every change goes through the same two steps before it is merged:

1. **CI.** Clippy with and without the diagnostics feature, every test in both
   configurations, and the repository policy check (`cargo xtask`), locally and
   on GitHub.
2. **A session in the game.** The Series S profile in full screen, played by
   hand or by a scripted route (walk, run, turn the camera, a capture every
   minute). Nothing compiles meanwhile. A change that could cost frames is run
   against a control of the same base, the same route and the same length.

### Latest sessions

Newest first. fps from the game's once-a-second frame lines in the world (over
1,000 draws a frame); memory is the game's private bytes, sampled every 20 s.

| Session | Build | Played | In the world | fps: median (p10-p90; 1% low) | Game memory, median / max | What it showed |
|---|---|---|---|---|---|---|
| Performance panel | `5963f415` ([#57](https://github.com/TaynanGT/Vice-Tay/pull/57)) | scripted route | 5 min | 20.8 (16.8-23.9; 1.4) | 26.2 / 27.1 GB | The panel on screen at no cost in frame rate; GPU load read from Windows (87% in the boat scene) |
| The boat section | `bc090416` | by hand | 30 min | 21.1 (19.1-23.2; 17.0) | 23.5 / 24.1 GB | The prologue through the boat section; frame rate eased from 21.7 to 20.6 fps and memory grew by about 1 GB from the first ten minutes to the last |

Each session's timeline, the frame broken into its parts and the card's
readings: [docs/tests.md](docs/tests.md) and
[docs/measurements.md](docs/measurements.md). What earlier sessions found and
fixed: [docs/findings-rtx4090.md](docs/findings-rtx4090.md).

### Under test, not merged yet

| Change | From | Status |
|---|---|---|
| Title textures uploaded at the start of each submission (frame order, flickering) | [Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD) `5d6bd841` | CI green; Series S session against a control |
| Wave64 groups over 64 threads keep exact barriers through divergence (translator version 3) | the C01 translator batch | Reviewed; Series S session against a control |
| PC FSR 4.1.2 to 3840×2160 (`-Fsr4K`, Series X profile) | Belobug, from [DexNom/VIctory](https://github.com/DexNom/VIctory) `ab34a8b2` | Ported; needs AMD's signed upscaler DLL |
| DLSS 4.5 Super Resolution (`-Dlss4K`) on the same bridge | this tree | Being written |

### Test reliability

Four `victory-gpu` tests failed CI now and then. Three shared global state with
tests running beside them (the one asking of `watch::ask_once`, the heap-sync
thread) and were fixed in [#55](https://github.com/TaynanGT/Vice-Tay/pull/55);
the fourth assumed where a heap would land and was fixed with the panel. Over
the whole `victory-gpu` suite run 15 times per commit: 3 failures before, none
in 30 runs after.

## Test PC

| Component | Configuration |
| --- | --- |
| CPU | AMD Ryzen 9 9950X3D, 16 cores and 32 threads |
| Motherboard | MSI MAG X870 TOMAHAWK WIFI (MS-7E51), revision 1.0 |
| BIOS | MSI 1.A93 |
| GPU | NVIDIA GeForce RTX 4090, 24 GB GDDR6X |
| NVIDIA driver | 621.40 |
| Integrated graphics | AMD Radeon Graphics |
| RAM | 48 GiB DDR5 (2 × 24 GiB), configured at 8,000 MT/s; 43.4 GiB visible to Windows |
| Page file | 128 GiB |
| System SSD | Samsung 990 PRO 4 TB NVMe |
| Additional SSDs | Samsung 990 PRO 2 TB NVMe and Samsung 970 PRO 1 TB NVMe |
| Monitor | Samsung Odyssey OLED G8 (G81SF), 3840 × 2160, 240 Hz |
| OS | Windows 11 Pro 26H2 x64, build 26300.10075 |

## Known problems

- **Frame rate.** Below 30 fps in the world. Of a 47 ms frame, about 34 ms go to
  preparing the GPU's work on the CPU and 13 ms to copying data back.
- **Memory.** The game holds 23-27 GB on the Series S profile. On a PC with less
  free RAM than that, Windows pages part of it out and frames stall.
- **Picture.** Skin, hair and some walls show flat bands of colour; some objects
  (tanks, tarps, plants, a skylight) glow in flat purple, pink or blue; some faces
  show scrambled colour patches and some eyes are lit white; clouds and reflections
  break into square blocks; a few shapes render as flat black.
  For banding that varies between launches, pass `-More @{VICTORY_RTV_FORMAT_LOG='1'}`
  to `scripts\play.ps1` on a diagnostics build. Compare the `RTV format check`
  totals and `RTV format mismatch` examples at the same checkpoint in a clean and a banded run.
- **Ray tracing.** The ray-traced structures are not built by default, so a few
  hundred dispatches a frame are skipped and some light is missing; `-RayTracing`
  builds them.
- **Loading.** The first run of a new build translates every shader while it plays;
  translate them ahead (below).
- **Measuring.** On the Series S profile the log's GPU-time counter reads near
  zero, so the tables leave it out; the panel's GPU load comes from Windows instead.

## Getting started

You need Windows 10 or 11 x64, a CPU with AVX2, a Direct3D 12 GPU with shader
model 6.6 and mesh shaders, Rust (the toolchain is pinned in the repository),
the MSVC build tools with a Windows SDK, and your own dump of the title.

```powershell
git clone https://github.com/TaynanGT/Vice-Tay
cd Vice-Tay
copy victory.example.toml victory.toml      # then set your paths, outside this folder
cargo build --locked --profile diag -p victory --features diagnostics
```

From the repository root, translate the shaders and warm the pipelines (each can
be stopped and resumed). With `game_dir` set in `victory.toml`, the shader
command scans that directory. To scan one archive, pass its quoted path, for
example `.\target\diag\victory.exe shaders "D:\My Game Dump\shaders.rpf"`.

```powershell
.\target\diag\victory.exe shaders       # scans game_dir in victory.toml
.\target\diag\victory.exe warm --used   # pipelines earlier runs drew with
```

Then play on the Series S profile, the one every session above uses:

```powershell
scripts\play.ps1 -Exe <path of your copy's executable> -Console series-s
```

`Play-VIctory.cmd` takes the same options from a Command Prompt; on a double-click
PowerShell asks for the executable's path.

| Option | What it does |
|---|---|
| (none) | Full screen, Series X profile, settings picked for your graphics card, performance panel on |
| **F11** or **Alt+Enter** | Switch between full screen and a window while it runs |
| **F3** | Hide or show the performance panel |
| `-NoPanel` | Start without the performance panel |
| `-Windowed` | Start in a window |
| `-Console series-s` | Run as a Series S: a smaller picture, lighter on the card, faster; the profile every session uses |
| `-Machine 4090` / `7900xtx` | Force the settings for NVIDIA (measured on the RTX 4090) or AMD |
| `-Machine laptop` | The settings for an NVIDIA PC with 16 GB of RAM (measured on an RTX 3050 6 GB laptop, [docs/laptop-16gb.md](docs/laptop-16gb.md)) |
| `-Safe` | Leave out the changes still on trial; try it first when something goes wrong |
| `-RayTracing` | Build the ray-traced structures, so the title's ray-traced indirect light runs |
| `-Build release` | Run `target\release\victory.exe` (`cargo build --release --locked -p victory`); it has no panel |

Every `VICTORY_...` switch is described in a comment where the code reads it.
More commands and debugging switches:
[docs/README-fxceless.md](docs/README-fxceless.md) and [docs/findings.md](docs/findings.md);
recording one frame and running it again: [docs/replay.md](docs/replay.md).

## How this tree is built

Vice-Tay is published as a repository of its own, not as a GitHub fork, with the
whole history of the trees it comes from. It started from codec-d/cool_fox (now
taken down), which is [avatarmanz/VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX)
with work for NVIDIA cards on top. New work from avatarmanz is merged as it lands,
and changes from other forks and our own fixes come in through pull requests,
each tested as described under [Tests](#tests) before it is merged. What each
change does: [docs/this-fork.md](docs/this-fork.md). How the layer works:
[docs/architecture.md](docs/architecture.md).

## Credits

This is other people's work with more on top. The history is kept, and commits
taken from elsewhere name the original commit and its author. Everyone with a
commit here is listed in [docs/credits.md](docs/credits.md).

- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and its
  contributors: the loader, the runtime services, the Direct3D 12 back end and
  the shader recompiler.
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)** and the
  people whose pull requests it carries.
- **[avatarmanz/VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX)**:
  list calls recorded on the title's threads and translated by workers,
  submissions prepared on threads of their own, cheaper write guards, textures on
  a copy queue, frame record and replay.
- **codec-d/cool_fox** (taken down by its author): two-plane composition, fallback
  heaps, table plans, the residency guard, fence wakes on `WaitOnAddress`, wave32
  lowering for NVIDIA, movie-plane uploads and XMA audio fixes.
- **[DexNom/VIctory](https://github.com/DexNom/VIctory)**: ray tracing fixes,
  cube gradients, sparse textures.
- **[Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD)**:
  the performance panel, the ambient light idea, pipeline workers, and work by
  romainhedouin.
- **[FQY55kai/ZJOU-VICTORY](https://github.com/FQY55kai/ZJOU-VICTORY)**.
- **[myokex/victory-amd](https://github.com/myokex/victory-amd)**: the guard over
  every view of a page (with Spexivus, joroyt and tightfall), and Joshua Odom's
  reads into title memory.
- **Geckomacabre** and **cxy0072019-sketch**: the texture read-back.
- Pull requests to this repository by **meshahid973** (relocation checks),
  **nullexxx** (file logging; an empty pipeline library on vkd3d-proton),
  **cxy0072019-sketch** (saved pipeline scheduling left out on request) and
  **welisonzika14-jpg** (placed-filler bookkeeping kept out of default builds).
- **[XCL](https://github.com/ivall/xcl)** (GPL-2.0-only) was read for how the same
  title behaves under another layer; no code of it is here.

## License

GPL-3.0-or-later, as VIctory. See [LICENSE](LICENSE); the one file it does not
cover is named in [docs/asset-policy.md](docs/asset-policy.md).
