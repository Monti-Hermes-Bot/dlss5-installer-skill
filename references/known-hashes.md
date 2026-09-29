# Known hashes — the neural runtime (`nvngx_dlssnr.dll`)

**Policy (updated 2026-09-29): the SHA-256 is the trust anchor, not the signature.**

There are now several `nvngx_dlssnr.dll` builds in circulation, and two of them are
legitimately *modified* — which means they fail Authenticode by design. So:

- A file whose SHA-256 is in the **pinned** table below is accepted for the GPUs listed,
  whatever its Authenticode status says.
- A file whose hash is **not** in the table is refused, even if someone says it is "the
  new version" — stop, tell the user, and ask them to report it (see SKILL.md Step 5) so
  the table can be updated after review.
- Never accept a file on size, version string, or filename alone.

```powershell
Get-FileHash "path\to\nvngx_dlssnr.dll" -Algorithm SHA256
Get-AuthenticodeSignature "path\to\nvngx_dlssnr.dll" | Select-Object Status
```

## Pinned — accepted

| SHA-256 (full) | Build | Authenticode | GPUs | Notes |
|---|---|---|---|---|
| `E16BCF15E16E13F527491CDF7845B2FE6521A738D8F7C9C721866A8496E1FC8E` | NVIDIA 310.8.0 (original signed) | **Valid** | **RTX 50 only** | The reference build. On RTX 20/30/40 it fails with `feature 18 create failed with 0xbad00001`. |
| `E67DEE209320CDAFE0E93E45675D7AA34323A53ACC57A72B2E40A181581C989A` | ShortFuse cross-generation 310.8 (`310.8.SF-v2`) | HashMismatch **(expected)** | **RTX 20 / 30 / 40 / 50** | Picks an FP16 path on 20/30, an Ada path on 40, leaves the 50 path unchanged. Hash published by the wilsjo2 OptiScaler DLSS-NR fork's install guide. |

Tell a 20/30/40 user plainly: the SF-v2 build is a modified NVIDIA binary; its hash is
the *only* thing vouching for it, which is why an exact match is non-negotiable.

Expect RTX 20/30 to be slow — community reports put an RTX 3080-class card around 20–40
fps at 1440p in heavy games. RTX 40 costs roughly 40% on a 4090.

## Seen, not pinned — refuse unless the user supplies an independently verified hash

| Build | Where seen | Why not pinned |
|---|---|---|
| `310.8.Lecram` (RHI lists it as "310.8.3 (50xx)") | `RankFTW/rhi-repo` release `dlssnr-310.8.Lecram`; **the Feeder's installer downloads this by default** | Modified, RTX 50 only, no published DLL hash, no documented advantage over the signed original. Release zip digest is `sha256:ddb64e55…9650ca54` if you need to identify it. |
| `310.8.SF` (v1) and `310.8.0-RTX40` | rhi-repo tags `dlssnr-310.8.SF`, `dlssnr-310.8.0-RTX40` (both 2026-08-30) | Earlier cross-generation builds, superseded by SF-v2 within a day. (Early community ports ran an FP8 path on RTX 20/30 that was unusably slow — a 3080 fell to 4 fps — before the FP16 conversion.) Use SF-v2. |
| Any "310.8.0" from a manager/one-click tool | e.g. a DLSS-5-MANAGER bundle reported HashMismatch (issue #58, 2026-09-24) | A HashMismatch "310.8.0" that is not SF-v2 is an unknown modification. |

## Known-bad

| SHA-256 (full) | Verdict |
|---|---|
| `4B8D19BC3EFF58A084F5ECA7489C921501C203450169FB82FF4F649A4482BA05` | HashMismatch copy labelled 310.8.0 — causes STANDBY/FAILED and `0xBAD00002`. Circulated widely; found pre-seeded in half a dozen games on one machine. |

## Where a good copy comes from

In order of preference:

1. **Already on the machine.** Sweep the library (below). RHI caches under
   `%LOCALAPPDATA%\RHI\DLSS-NR\<version>\`.
2. **`RankFTW/rhi-repo` releases** — the source RHI and the Feeder installer both use.
   Acceptable **only** when the extracted DLL's hash is pinned. Ask before downloading
   (~110 MB). Check the zip digest first, then extract, hash the DLL, then deploy:

   | GPU | Release tag | Zip digest (from GitHub) |
   |---|---|---|
   | RTX 50 | `dlssnr-310.8.0` | `sha256:388c0a7912e15ec911b9c9e11a692142b11fe387ddf2b637d8c358138fffb3ac` |
   | RTX 20/30/40 | `dlssnr-310.8.SF-v2` | `sha256:1da35941894994eb087e017577829e492454e9bae3a6a9397027069ceb74955c` |

   ```powershell
   gh release download dlssnr-310.8.SF-v2 -R RankFTW/rhi-repo -D "$env:TEMP\dlssnr"
   ```
3. Never a YouTube-linked archive, a "one-click" site, or a Discord re-upload you cannot
   hash-match.

> Some guides say the runtime "comes from a driver package" (616.56+). On a machine with
> driver 616.92 installed it was **not** present in the DriverStore or the NGX model
> cache, so do not send users hunting for it there — treat that claim as unverified.

When using the Feeder's `Install-DLSS5Feeder.ps1`, pass the pinned file explicitly so it
does not fetch the unpinned Lecram build: `-DlssNrDll "C:\path\to\nvngx_dlssnr.dll"`.

## Library sweep

```powershell
$pinned = @{ 'E16BCF15E16E13F527491CDF7845B2FE6521A738D8F7C9C721866A8496E1FC8E'='OK 310.8.0 signed (RTX 50)';
             'E67DEE209320CDAFE0E93E45675D7AA34323A53ACC57A72B2E40A181581C989A'='OK SF-v2 (RTX 20-50)';
             '4B8D19BC3EFF58A084F5ECA7489C921501C203450169FB82FF4F649A4482BA05'='BAD - replace' }
Get-ChildItem "D:\SteamLibrary\steamapps\common","C:\Program Files (x86)\Steam\steamapps\common","$env:LOCALAPPDATA\RHI" `
  -Recurse -Filter "nvngx_dlssnr.dll" -Depth 5 -ErrorAction SilentlyContinue |
  ForEach-Object {
    $h=(Get-FileHash $_.FullName -Algorithm SHA256).Hash
    $v= if ($pinned[$h]) { $pinned[$h] } else { 'UNKNOWN - do not use' }
    "{0,-12} {1,-26} {2}" -f $h.Substring(0,12),$v,$_.FullName
  }
```

Notes:

- The RenoDX DLSS5 addon prints `Runtime sha256: E16BCF15... (reference match)` in its
  overlay when it sees the reference build — strong corroborating evidence.
- A `.original` backup next to a bad copy has been seen carrying the **same bad hash** —
  never assume a backup is clean; hash it.

## Other components — verify by hash, not size

Feeder releases have shipped **byte-identical in size** across versions (v0.2.0 and
v0.3.0 `dlss5-feed.addon64` are both 83,968 bytes), and a log banner can keep printing an
old version after an upgrade. `Get-FileHash` is the only reliable identity check.

For the Feeder, the first line of `dlss5-feed.log` names the exact build
(`dlss5-feed 1.17.0 commit 03710dd`); the bridge logs the SHA-256 of every loaded addon at
the top of `dlss5-bridge.log`. Record the hash of whatever you deploy so "did the upgrade
actually land?" is answerable.

GitHub now shows a `sha256:` digest per release asset (`gh release view <tag> -R <repo>
--json assets`). Compare a downloaded zip against it before extracting.
