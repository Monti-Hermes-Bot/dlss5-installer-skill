# Sources

First researched 2026-08-30; **refreshed 2026-09-29**. The GitHub READMEs and release notes
are the authoritative documentation — news coverage is context only. `gh release list -R
<repo>` is the fastest way to see whether anything below has moved again.

## Primary tools

| Source | Good for |
|---|---|
| https://github.com/jlrouzies-fr/DLSS5-Feeder | The Feeder (v1.17.0). Still the single best document in the ecosystem: API matrix, file layouts, full cfg table, consumer compatibility table by driver, troubleshooting, and `tools/Install-DLSS5Feeder.ps1` / `Verify-DLSS5Feeder.ps1`. |
| https://github.com/NIGos/dlss5-bridge | The bridge (renamed from `dlss5-dx11-bridge`; old URL redirects). README: cfg table, confirmed games, `VERIFYING-DOWNLOADS.md` with SHA-256s, the `dlss5bridge.com` malware warning. |
| https://github.com/RankFTW/rhi-repo/releases | **Where versioned builds of the RenoDX DLSS5 addon (`renodx-dlss5-<ver>`), ShortFuse's `renodx-dlss-SF-<date>`, and the runtime builds (`dlssnr-<ver>`) are published**, with per-asset SHA-256 digests. Used by RHI and by the Feeder installer. |
| https://github.com/RankFTW/RHI · [releases](https://github.com/RankFTW/RHI/releases) | The installer/manager (2.7.8). Release notes document the Neural Rendering section, version pinning, and the runtime manifest. |
| https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass | OptiScaler DLSS-NR fork (Scenario E). `INSTALL-DLSSNR.md` is excellent — and publishes the SHA-256 of both accepted runtime builds. |
| https://github.com/Dagherbou/OptiScaler_DLSSNR | The original OptiScaler + DLSS-NR build the fork descends from (v0.2.0, idle since 09-04). Upstream OptiScaler PR #1116 is still open. |
| https://github.com/jlrouzies-fr/OptiScaler-DLSSNR-PreSR-Multipass/releases/tag/v0.8.92 | Feeder author's flicker-fixed build of the fork. |
| https://github.com/clshortfuse/renodx | The RenoDX framework. Releases are now `nightly-YYYYMMDD` plus the older `snapshot` tag — **neither carries the DLSS5 addon**; it is distributed via the RenoDX Discord, Nexus and the rhi-repo mirror. |
| https://github.com/PEQHUB/RenoDX-DLSS5-Generic | A source copy of the RenoDX DLSS5 addon (v8.5.0-rc10). **Unconfirmed provenance** — useful for config key names, not as a download. |
| https://github.com/umar-afzaal/LumeniteFX | LumeniteFX Kernel / QuantMotion motion vectors. Default branch is `mainline`, not `main`. |
| https://github.com/NIGos/dlss5-d3d12-fix | Mip-chain fix. Obsolete (superseded by NR v3.3.5) but documents the `0xBAD00005` / guides-`0x0` symptom set. |
| https://github.com/markitzeroo/dltb-dlss5-fix | The per-game typeless→typed fix pattern. Read before writing any new per-game fix. |
| https://github.com/kayle2203/dlssnr-signature-repair | Replaces a HashMismatch runtime from a local Valid copy (RTX 50 only — see known-hashes). |
| https://github.com/martymcmodding/iMMERSE | LaunchPad motion vectors — now optional (provider 1). |
| https://reshade.me | ReShade. **Must be the Addon build** (`ReShade_Setup_X.X.X_Addon.exe`). |
| https://github.com/beeradmoore/dlss-swapper | Clean source for `nvngx_dlss.dll`. |

## Official NVIDIA DLSS 5

| Source | Notes |
|---|---|
| https://www.nvidia.com/en-us/geforce/news/nba-2k27-dlss-5-3d-guided-neural-rendering-geforce-game-ready-driver/ | 2026-09-03: DLSS 5 launches in NBA 2K27 with driver 616.64. |
| https://www.pcguide.com/news/dlss-5-games-list/ | The announced fall-2026 game list. |
| https://www.notebookcheck.net/Nvidia-confirms-DLSS-5-is-coming-to-RTX-40-series-GPUs-but-Ada-Lovelace-owners-have-no-release-date-yet.1388828.0.html | RTX 40 official support promised, no date. No plans announced for RTX 20/30. |
| https://www.notebookcheck.net/First-DLSS-5-game-tested-DLSS-5-humbles-RTX-5090-with-over-50-performance-loss.1391978.0.html | Digital Foundry numbers: ~51% loss on a 5090 at 4K (~8 ms/frame), up to 731 MB VRAM. |
| https://heldgames.com/guides/dlss-5-official-vs-mod | Official vs leaked: similar cost; the leak lacks engine-level masks and developer-tuned models. |

## Community / context

| Source | Notes |
|---|---|
| https://www.nexusmods.com/site/mods/2224 | "Applying RR and DLSS 5 RenoDX for the games" — the main community how-to. **Blocks automated fetching (HTTP 403).** |
| https://www.nexusmods.com/control/mods/140 | Control — the first game DLSS 5 was ported to. |
| https://www.nexusmods.com/deathstranding/mods/95 | RenoDX HDR fix for Death Stranding DC; warns OptiScaler / SpecialK / PureDark FG may conflict. |
| https://github.com/zhubaohi/FF7R-DLSS5 | One-click installer for FF7 Rebirth — a per-game packaging example. |
| RenoDX Discord | Where the newest consumer builds and the RTX 20/30/40 runtime (ShortFuse's pinned thread) circulate. Not indexable. |
| https://www.tomshardware.com/pc-components/gpus/exclusive-dlss-5-has-already-been-ported-to-work-on-rtx-4000-series-graphics-cards-incompatible-cuda-instructions-get-patched-to-work-on-previous-gen-hardware | RTX 40 port: 4090 135 → 82 fps (~-39%). |
| https://wccftech.com/dlss-5-now-works-on-rtx-20-30-gpus-older-graphics-apis-and-even-pcsx2/ | RTX 20/30 via FP16 conversion (2026-08-31). |
| https://github.com/NODIX-TECH/DLSS-5-MANAGER/issues/58 | A manager bundling an unidentified HashMismatch "310.8.0" (2026-09-24) — why hashes are pinned. |
| https://github.com/danielblnc/DLSS-NR-on-AMD | AMD RDNA 3/4 port (alpha). Out of scope for this skill. |
| https://videocardz.com/newz/breaking-modders-unlock-experimental-nvidia-dlss-5-hours-after-dll-discovery | Origin story: the DLL discovery in NBA 2K27. |

## Key statements worth keeping (paraphrased, with short quotes)

- **GPU floor** — the Feeder README (2026-09) states the signed 310.8.0 and the Lecram build
  create the neural feature on RTX 50 only; older cards need a GPU-specific modified runtime.
  This supersedes the Aug-30 dlss5-d3d12-fix claim that nothing worked below RTX 50.
- **API coverage** — the Feeder README now lists D3D11, D3D12, Vulkan, OpenGL, 32-bit,
  DirectX 10 and DirectX 9. Early-August statements that DX9/DX10/Vulkan "won't work" are
  historical. NGX itself is still 64-bit only, hence the `host64` helper.
- **No upscaling in the Feeder** — quality modes are "not as a setting, and not as a future one either"; the
  Feeder stays DLAA-only.
- **Fail-safe** — the bridge README: on any failure it disables itself and the game renders
  normally.
- **No redistribution** — Dagherbou's OptiScaler_DLSSNR notes: the runtime is NVIDIA's,
  "not mine to redistribute".

## Provenance caveat

The feeders, bridges, OptiScaler fork and fixes are open source and readable. The
`nvngx_dlssnr.dll` runtime is **not** an official standalone NVIDIA distribution: the signed
310.8.0 leaked from an early-access game build, and the RTX 20/30/40 build is a community
modification of it. Some guides say the runtime "comes from a driver package" — on a
616.92 install it was not present in the DriverStore or the NGX cache, so that is
unverified. Trust is by exact SHA-256 per `known-hashes.md`.
