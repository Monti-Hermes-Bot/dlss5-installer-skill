# dlss5-installer-skill

**A copy-paste skill that turns Claude (or any capable LLM with file access) into a
guided DLSS 5 Neural Rendering installer for your games.**

You paste one prompt, tell it which game, and it does what an experienced modder would:
checks your hardware, refuses anti-cheat games, figures out the game's graphics API,
installs the right stack (RenoDX DLSS5 + ReShade + DLSS5-Feeder or the DX11 bridge),
and then **verifies from the log files** instead of just saying "done!". If something
breaks, it debugs one variable at a time — and it can cleanly remove everything.

> 🌐 **Start here: [the project page](https://thunderruler.github.io/dlss5-installer-skill/)** —
> it has the copy-paste prompt and a plain-English explanation.

> **Never done this before?**
> [`references/first-run.md`](references/first-run.md) walks through what actually happens on a
> machine with none of this installed — what you supply, what the assistant handles, and how long
> it takes.

## Quick start

### Option 1 — paste this into Claude (or your LLM of choice)

> Fetch https://raw.githubusercontent.com/ThunderRuler/dlss5-installer-skill/main/SKILL.md and
> follow it exactly. This whole session is for installing DLSS 5 into my games.
> First game: **<game name>**

Works in Claude Code, Claude Desktop (with file access), or any assistant that can fetch
a URL and run commands on your PC.

### Option 2 — install it properly (one command)

```bash
npx skills add ThunderRuler/dlss5-installer-skill
```

Installs into Claude Code and ~20 other agents (Cursor, Codex, Copilot, Gemini CLI,
Cline, Windsurf, Zed…) via the [Agent Skills](https://skills.sh) standard. Then just ask
to "get DLSS 5 working on <game>" and it triggers on its own — or invoke
`/dlss5-installer` directly in Claude Code.

Prefer git? `git clone` this repo into `%USERPROFILE%\.claude\skills\dlss5-installer`
works too.

**Staying up to date:** an installed skill is a snapshot, so run `npx skills update` to
pull the latest version. The copy-paste prompt above always fetches the current `SKILL.md`
straight from this repo, so it never goes stale.

## What you need

| Requirement | Why |
|---|---|
| **NVIDIA RTX GPU** — 50-series + a current driver is the documented config | RTX 20/30/40 work with a community-modified runtime the skill verifies by hash — slower, and not yet documented here, so [tell us](https://github.com/ThunderRuler/dlss5-installer-skill/issues/new/choose) how it goes! |
| Windows + Steam/GOG/Epic games | Game Pass installs are locked and can't be modded this way |
| A game **without anti-cheat** | The skill scans and hard-refuses anti-cheat titles — injecting DLLs gets you banned. For multiplayer games with none found, it stops and asks you to verify first |
| DirectX 9/10/11/12, Vulkan or OpenGL | Almost everything has a path now — only 64-bit DirectX 10 (beta route, untested) and 32-bit DirectX 12 don't |
| Not already officially supported | NVIDIA shipped DLSS 5 on 2026-09-03 (NBA 2K27 first). If your game has a "DLSS Neural Rendering" setting, use that instead |

## How the community loop works

**Read first, then write back.** Before installing, your assistant checks
[`references/game-results.md`](references/game-results.md) for your game — if someone
already got it working (or proved it can't), you start from their answer instead of from
scratch.

Once *you've* confirmed how your install turned out, the assistant writes the report
itself — game, engine, real runtime API, scenario, the proof line from the log — and asks
whether to file it. **You're approving a finished report, not filling in a form.**

**Please say yes.** This is the only mechanism by which the log gets better, and it takes
you about fifteen seconds. That includes the boring outcomes:

- **"It installed fine and looked identical"** is one of the most valuable reports there
  is. Nobody files these, so everyone keeps rediscovering the same dead ends.
- **A failure with a known cause** saves someone an entire evening.
- **An RTX 20/30/40 result** would be the first documented one here, and yours would
  become the reference.

Nothing is sent without your say-so, and no NVIDIA DLLs are ever attached or linked.

## What's in the repo

```
SKILL.md                     the procedure the LLM follows (the skill itself)
references/
  first-run.md               starting from nothing: what happens, in order, and how long
  components.md              every tool in the ecosystem and where to get it
  decision-tree.md           picking the right stack from the game's API + bitness
  install-playbook.md        exact file layouts for scenarios A/B/C/D
  troubleshooting.md         error codes, symptom → cause, real solved cases
  config-reference.md        every config key for every component
  known-hashes.md            good/bad neural-runtime hashes (a corrupt copy is common!)
  motion-vectors.md          image quality for games without DLSS: providers, smearing, ghosting
  game-results.md            per-game results log — PRs welcome
  sources.md                 primary sources for everything claimed here
```

## Contributing game results

Got it working — or definitively *not* working — in a game that isn't logged yet?

**[Open a game result issue](https://github.com/ThunderRuler/dlss5-installer-skill/issues/new/choose)**
and fill in the form. No git required, and it gets added to
[`references/game-results.md`](references/game-results.md) for you. Prefer to edit the file
yourself? That works too — see [CONTRIBUTING.md](CONTRIBUTING.md).

Failure reports are just as valuable as successes: knowing a game *can't* work saves the
next person an evening. Please include the proof line from your log rather than just
"it worked" — that's what makes the table trustworthy.

Two things that get declined, and they're why this project can exist publicly: **no
binaries** (above all `nvngx_dlssnr.dll`) and **no new links to runtime downloads** (the only
source named is the hash-pinned rhi-repo release used by RHI itself).

## Credits & provenance

This skill orchestrates other people's excellent work:
[RenoDX](https://github.com/clshortfuse/renodx) (clshortfuse),
[DLSS5-Feeder](https://github.com/jlrouzies-fr/DLSS5-Feeder) (jlrouzies-fr),
[dlss5-bridge](https://github.com/NIGos/dlss5-bridge) (NIGos),
[OptiScaler DLSS-NR](https://github.com/wilsjo2/OptiScaler-DLSSNR-PreSR-Multipass) (wilsjo2, on Dagherbou's OptiScaler_DLSSNR),
[LumeniteFX](https://github.com/umar-afzaal/LumeniteFX) (umar-afzaal),
[iMMERSE LaunchPad](https://github.com/martymcmodding/iMMERSE) (Marty McFly),
[ReShade](https://reshade.me) (crosire), and
[RHI](https://github.com/RankFTW/RHI) (RankFTW).

**No NVIDIA binaries are included in this repo, ever.** The `nvngx_dlssnr.dll` runtime is
NVIDIA code that was never released standalone (the signed build leaked from a game's
early-access build; the RTX 20/30/40 build is a community modification of it). The skill
accepts a copy only if its SHA-256 matches a short pinned list — preferring one already on
your machine, otherwise fetching the named build from RHI's release repo after asking you
— and refuses anything else.

Use at your own risk. Modding game files can break games; Steam's *Verify integrity of
game files* restores them.

## License

Documentation and skill text: [MIT](LICENSE). The tools it references have their own
licenses.
