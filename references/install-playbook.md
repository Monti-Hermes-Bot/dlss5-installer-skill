# Install playbook

**Updated 2026-09-29.** In every scenario, `nvngx_dlssnr.dll` means **a file whose SHA-256
is pinned for this user's GPU** in `known-hashes.md` — the signed original on RTX 50, the
SF-v2 build on RTX 20/30/40. "Consumer" means `renodx-dlss5.addon64` **v6.1+** unless the
user already runs another one (see `components.md`).

## Scenario A — D3D12 game with native DLSS (simplest)

Example: Death Stranding Director's Cut.

Files next to the game exe:
```
game.exe
dxgi.dll                  <- ReShade 6.8+ Addon build, renamed
renodx-dlss5.addon64      <- v6.1+ (v8.x current)
nvngx_dlssnr.dll          <- pinned hash for this GPU
nvngx_dlss.dll
```

1. Install ReShade 6.8+ **Addon build** against the game exe, choosing Direct3D 10/11/12,
   with "Enable loading of add-ons" ticked. Or in RHI, set the game's **RS DLL** dropdown
   to `dxgi.dll`, or use RHI's Neural Rendering → *DLSS5 Tool* method.
2. Drop `renodx-dlss5.addon64` next to the exe.
3. Ensure the runtime and `nvngx_dlss.dll` are present; hash the runtime.
4. Launch. Confirm in `ReShade.log`:
   ```
   Registered add-on "DLSS 5 Neural Rendering" ...
   inline feature 18 evaluation succeeded (...)
   ```
5. **Home** for the overlay → Add-ons → DLSS 5 Neural Rendering. **F6** toggles the effect.

**Force DLAA** unless you choose a different hook point. The pass runs after upscaling by
default, and mismatched render/output resolution causes partial-frame artefacts (see
troubleshooting). Use **NVIDIA App → Graphics → *game* → DLSS Override – Super Resolution →
DLAA**, and leave DLSS *enabled* in the game — the add-on hooks the game's own NGX call.
v8.x's `NRHookPoint=1` (before upscaling) is the alternative for users who want the cost
of NR paid at render resolution; treat it as experimental and verify from the log.

## Scenario B — D3D11 or Vulkan 64-bit game with DLSS (native or modded)

Everything from Scenario A, plus:
```
dlss5-bridge.addon64      <- NIGos/dlss5-bridge, stable v1.4.12
```

- **Upgrading?** Delete any old `dlss5-dx11-bridge.addon64` first — ReShade loads both and
  the old one wins.
- D3D11: ReShade as `dxgi.dll`. **Vulkan**: ReShade as a **layer** (installer → Vulkan →
  pick the exe), no local `dxgi.dll`; add `AddonPath=.\` under `[ADDON]` in `ReShade.ini`.
  The Vulkan mirror is on by default (`vk_mirror=1`).
- It writes `dlss5-bridge.cfg` on first run with defaults that need no adjustment. A cfg from
  a different version is reset to defaults unless its first line is `# dlss5-bridge keep`.
- Diagnostics in `dlss5-bridge.log`: SHA-256 of every loaded addon, the contract read from
  the game, the result of every NGX call, and `[bridge] 600 frames: …` timing lines. The
  consumer shows **INJECTED** when it's working. ReShade overlay → **DLSS 5 Bridge** window
  → tick *Details* for a bug-report dump.
- To prove the bridge is or isn't at fault, set `stage=0`. If the problem persists, it
  isn't the bridge.
- Known per-game values: RDR2 needs `flags=75`; PSO2 `0xBAD00005` fixed by `shape=1`.

## Scenario C — 64-bit game with NO DLSS (Feeder)

### Fast path: the Feeder's own installer (offer this first)

DLSS5-Feeder v1.17.0 ships `Install-DLSS5Feeder.ps1`, which detects bitness and API,
installs ReShade (local DLL or Vulkan layer), the Feeder, ReShade headers, LumeniteFX,
writes `ReShade.ini` / `ReShadePreset.ini` with the provider and technique order, and runs
the verifier at the end. It backs up and merges rather than overwriting.

**Always pass the runtime explicitly** — left alone it downloads RHI's newest `dlssnr-`
build (currently the unpinned Lecram build):

```powershell
powershell.exe -ExecutionPolicy Bypass -File .\Install-DLSS5Feeder.ps1 "C:\path\to\game.exe" `
  -Consumer RenoDX -DlssNrDll "C:\path\to\pinned\nvngx_dlssnr.dll"
```

Useful switches: `-Api D3D|Vulkan|OpenGL|D3D9|D3D8` (override detection), `-Consumer
RenoDX|OptiScaler|DFC`, `-RenoDxAddon <file>`, `-DlssDll <file>`, `-FeederZip <file>`,
`-Yes` (non-interactive), `-NoElevate`. Get the script only from the Feeder's GitHub
Releases/`tools\` folder.

Then **verify independently** — the script is not the proof:

1. Hash `nvngx_dlssnr.dll` next to the exe against `known-hashes.md`.
2. Run `Verify-DLSS5Feeder.ps1` from the release zip: every line `[ OK ]` or an understood
   `[WARN]`.
3. Launch and read the logs (proof lines below).

Record what the script created/overwrote (it keeps backups) so removal stays exact.

### Manual layout (fallback, or when the user wants to see every step)

```
game.exe
dxgi.dll                    (ReShade Addon build; Vulkan = layer instead)
dlss5-feed.addon64
renodx-dlss5.addon64        (the one consumer)
nvngx_dlssnr.dll            (pinned hash)
nvngx_dlss.dll
reshade-shaders\
  Shaders\
    DLSS5_Feed.fx           (same release as the addon)
    ReShade.fxh             (+ the standard headers — a Feeder-only folder won't compile)
    lumenite_Kernel.fx      (provider 3 — recommended)
    include\lumenite_*.fxh
  Textures\
    lumenite_bluenoise256.png
```

1. ReShade 6.8+ Addon build as `dxgi.dll` (D3D11/12), `opengl32.dll` (OpenGL) or a layer
   (Vulkan, plus `AddonPath=.\`).
2. `dlss5-feed.addon64` next to the exe, `DLSS5_Feed.fx` into `reshade-shaders\Shaders\`
   — **addon and shader from the same release**.
3. Install a motion-vector provider (LumeniteFX Kernel recommended — `motion-vectors.md`)
   and set `DLSS5_MV_PROVIDER` in **both** `ReShade.ini` and the preset, game closed.
4. Consumer + both nvngx DLLs next to the exe.
5. In game (Home):
   - enable **Generic Depth** and pick the correct scene depth
   - enable the provider technique (`LUMENITE: Kernel 2.0` for provider 3)
   - enable **DLSS 5 Feed** — **below** the provider in the technique list
   - enable neural rendering in the consumer's panel
   - turn the game's **MSAA/SSAA off**

Pipeline: `provider → DLSS5_Feed → [D3D12: DLSS DLAA + neural] → written back → remaining
effects → present`.

Proof in `dlss5-feed.log`: line 1 `dlss5-feed 1.17.0 commit …`, then `feature ready: WxH
DLAA`, then `[feed] frame N delivered (…)` — the parenthesis names the transport:
`same-device` (D3D12), `Vulkan transport`, `OpenGL transport`; D3D11 instead shows the
work-resolution form `(WxH at P% -> WxH, reset=0 …)`.

## Scenario D — 32-bit game (D3D11 / D3D10 / OpenGL / Vulkan / D3D9 via dgVoodoo2)

```
game.exe                   (32-bit)
dxgi.dll                   (32-bit ReShade; D3D9 games also get dgVoodoo's MS\x86\D3D9.dll)
dlss5-feed.addon32
reshade-shaders\           (Feed shader + provider, as Scenario C)
host64\
  dlss5-feed-host64.exe    (same zip as addon32 — protocol must match)
  dxgi.dll                 (64-bit ReShade)
  renodx-dlss5.addon64
  nvngx_dlssnr.dll
  nvngx_dlss.dll
```

The installer script handles this layout too (and fetches dgVoodoo2 for D3D8/9).

- The helper's window sits hidden behind the game by default (`host_window=0`; set `1` for a
  visible window where Home works). Since v0.12 the DLSS 5 panel is **cast into the game
  window**: enable "Show the DLSS 5 panel in-game" (windowed /
  borderless) or "Show as texture" (works in exclusive fullscreen; not on D3D10). Bind it
  via "Set key" (`cast_key`/`cast_mods`).
- Self-test the helper without the game: `host64\dlss5-feed-host64.exe --test` → expect
  `300/300 evaluates succeeded`.
- Logs are split: `dlss5-feed.log` (game side, line 1 `dlss5-feed32 …`, `host connected in N
  ms (protocol v11 …)`), `host64\dlss5-feed-host.log`, and `host64\ReShade.log` (consumer
  output — the `feature 18` lines live here).
- If the helper dies, the pipe breaks, the addon disables itself, the game keeps rendering.

## Scenario E — OptiScaler DLSS-NR fork (native-DLSS games, no ReShade)

For 64-bit games whose temporal upscaler (DLSS, FSR or XeSS) reaches an OptiScaler D3D12
path. Its main advantage: NR can run **before** super-resolution, at render resolution —
the most effective way to cut DLSS 5's ~50% cost.

1. Game and launcher closed. Back up any existing proxy DLL and OptiScaler install.
2. Extract the **whole** `OptiScaler-NR-v<ver>.zip` (wilsjo2's releases; not `-rtx40-mfg`)
   next to the real exe. (wilsjo2 v0.8.1+ ships no `nvngx.dll_dlssnr.dll` forwarder; if an
   older guide or build left one behind, it is not the model.)
3. Add the pinned `nvngx_dlssnr.dll` beside it.
4. Run `setup_windows.bat` → NVIDIA → pick a proxy name (`dxgi.dll` usually; Cyberpunk
   validated with `dbghelp.dll`). It creates an uninstaller.
5. `OptiScaler.ini`:
   ```ini
   [DlssNr]
   Enabled=true
   RunBeforeSR=true
   Passes=1
   WorkingScale=1.0
   [ProcessFilter]
   TargetProcessName=auto
   ```
   D3D11 games: `[Upscalers] Dx11Upscaler=dlss_12`.
6. Launch, press **Insert**. Proof in `OptiScaler.log`: `OptiScaler ... loaded`, then the
   model initialising. `the model would not initialise` on RTX 20/30/40 → wrong runtime hash.

Rules: **no RenoDX DLSS5 addon in the same folder** (two consumers). Never copy an
`OptiScaler.ini` with another game's `TargetProcessName` (puts it in pass-through: no menu,
no NR). Below 100% model resolution, builds up to v0.8.91 flicker — use the Feeder author's
v0.8.92. RTX 20/30 run a much heavier FP16 path — a reduced `WorkingScale` may be needed.

**Combined with the Feeder** (games with no DLSS, `-Consumer OptiScaler`): rename
`OptiScaler.dll` to **`winmm.dll`** (ReShade keeps `dxgi.dll`), do **not** run
`setup_windows.bat`, set `[Upscalers] Dx12Upscaler=dlss` and keep `FinishedPicture=false`.
The Feeder logs `NGX calls are routed through OptiScaler DLSS-NR (winmm.dll)`.

## Post-install checklist

- [ ] `ReShade.log` exists and names the right DLL and bitness (A–D)
- [ ] exactly one consumer deployed
- [ ] `nvngx_dlssnr.dll` hash is **pinned for this GPU** in `known-hashes.md`
- [ ] consumer is not v4.6/v4.7 if driver ≥ 616.64 (v6.1+ recommended)
- [ ] the proof line for the scenario appears
- [ ] render resolution == output resolution (DLAA) where the hook point requires it
- [ ] no stale files: old `dlss5-dx11-bridge.addon64`, a `dlss5-feed.addon64` inside `host64\`,
      a leftover `nvngx.dll_dlssnr.dll` beside a Feeder+OptiScaler setup that doesn't need it
