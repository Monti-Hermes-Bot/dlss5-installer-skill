# First run — starting from nothing

What actually happens on a machine with none of this installed, in order, with the parts
you supply and the parts your assistant handles marked separately.

Read this if you want to know what you're getting into before you start. If you'd rather
just go, paste the prompt from the [project page](https://thunderruler.github.io/dlss5-installer-skill/)
and let the assistant walk you through it — it covers all of the below on its own.

## Before anything: what you need

| | |
|---|---|
| **An NVIDIA RTX GPU** | 50-series is the documented config. See the note below for 40/30. |
| **A current NVIDIA driver** | You install this yourself. Nothing here can do it for you. |
| **A game without anti-cheat** | Non-negotiable. Injecting DLLs is what anti-cheat bans people for. |
| **Disk space** | About 250 MB per game — the neural runtime alone is ~165 MB and it's per-folder. |
| **An assistant with file access** | Claude Code, Claude Desktop with file access, or equivalent. A chat window with no access to your PC cannot do this. |

**On 40- and 30-series:** people report getting this working, but there is **no documented
result here** — the entire log was built on a 5070 Ti. The skill will warn you and then
continue if you want it to. If it works on yours, that report is the single most valuable
thing you could contribute, because it would be the first.

## Step 0 — the shortcut nobody tells you about

On a machine with nothing installed, **install [RHI](https://github.com/RankFTW/RHI/releases)
first.** Its DLSS 5 button deploys three of the four components in one step, including a
hash-verified copy of the neural runtime.

Doing it piecemeal instead means hunting the RenoDX Discord and Nexus for a file that
isn't on any release page, and sourcing a leaked NVIDIA DLL yourself — which is where
people pick up the corrupt copy. **Start with RHI.** Your assistant will suggest this
anyway; it's here so you're not surprised by it.

## Step 1 — components

Your assistant checks for four things and tells you which are missing:

1. **ReShade 6.8+, Addon build** — you download and run the installer (`_Addon.exe`). The
   *standard* build silently cannot load addons, and picking the wrong one is the most
   common first-run mistake.
2. **`renodx-dlss5.addon64`** — RHI provides it. Otherwise: RenoDX Discord or
   [Nexus](https://www.nexusmods.com/site/mods/2224). Not on any GitHub release page.
3. **`nvngx_dlssnr.dll` v310.8.0** — the neural runtime. **This repo never distributes it
   and neither should anyone else.** Your assistant finds a copy already on your machine
   and verifies it cryptographically before use.
4. **`nvngx_dlss.dll`** — standard DLSS. Many games already ship it.

**About that third one.** It is unreleased NVIDIA code that leaked from a game build.
Copies circulating in the wild are frequently corrupt or tampered with, and a bad one
fails in a way that looks exactly like a GPU problem — you'll get `STANDBY` or `FAILED`
and conclude your card isn't supported. Your assistant checks the Authenticode signature
and the SHA256 against known-good and known-bad values before touching anything. If it
can't find a valid copy it will tell you so rather than proceeding.

**Nothing is downloaded behind your back.** Anything fetched from the internet is named
and shown to you first.

## Step 2 — your game gets triaged

The assistant determines the game's **real** runtime API (not what the store page or the
launcher claims), its bitness, where the actual executable lives, and whether it already
has DLSS. That picks one of four install recipes.

This is the step most written guides get wrong, and it's the main reason this is a skill
rather than a blog post. Gunner HEAT PC sat in this log as "D3D11" for days when it is
actually D3D12 — and that one field decides which files get installed.

It also checks the community log here for your game first. If someone already solved it,
you start from their answer.

## Step 3 — install, then prove it

Files go in next to the real executable. You launch the game. The assistant reads the
mod's own log files for the specific line proving frames are being delivered.

There is no "should be working now." Either the proof line is in the log or it keeps
debugging. Expect to launch the game more than once.

## Step 4 — you judge it

**This is your call, not the assistant's.** Three honest outcomes:

- **Working and worth it.**
- **Working and pointless** — it installed cleanly and looks identical. Common, and not a
  failure of the install. See *Which games actually benefit* in
  [`game-results.md`](game-results.md): stylized games mostly land here.
- **Dead end** with a known cause.

Undo is one ask away at any point — the assistant tracks every file it added, so removal
is exact. Worst case, Steam's *Verify integrity* restores the game.

## Step 5 — share what happened

Once you've judged it, the assistant **writes the whole report itself** — game, engine,
real API, scenario, the proof line from your log — and asks whether to file it. You're
approving a finished report, not filling in a form. Nothing is sent without your yes, and
no NVIDIA DLLs are ever attached or linked.

**Please say yes.** It costs you about fifteen seconds and it is the only reason the log
is worth reading. That includes the unglamorous results:

- **"Installed fine, looked identical"** is the most useful and least reported outcome
  there is. Nobody files these, so everyone keeps rediscovering the same dead ends.
- **A failure with a known cause** saves the next person an entire evening.
- **A 40- or 30-series result** would be the first documented one either way.

If you'd rather not, say so once and it won't ask again.

## Roughly how long

First game on a clean machine: **30–60 minutes**, most of it spent locating components
and relaunching the game. Every game after that: **10–20 minutes**, because the components
are already sitting on your disk.

If your game is already in the log, considerably less.
