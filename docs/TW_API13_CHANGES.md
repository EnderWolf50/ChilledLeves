# ChilledLeves Taiwan API 13 maintenance

## Lineage

- Upstream: <https://github.com/LeontopodiumNivale14/ChilledLeves> (remote `upstream`, default branch `Main-Branch`). The fork is <https://github.com/EnderWolf50/ChilledLeves> (remote `origin`), default branch `api13`.
- Base: `28d30255c7fb65c3defb27dc36ef7a32325d0be0` ("Alert update", 2025-10-22). No tag. At this commit `ChilledLeves/ChilledLeves.csproj` uses `Dalamud.NET.Sdk/13.0`, `net9.0-windows7.0`, `Version` `1.0.1.18`, and lockfile restore.
- Submodule: `ECommons` at `3aeec9a52b721646445e2828d120dee17591eb35` (2025-09-18, `net9.0-windows7.0`).
- Evidence: the upstream API move is `d01874182d0b3b03b9b6aa3e7a81fc7e206ed409` ("Updated to 14.0 (surprisingly easy)", 2025-12-22). It moves the SDK from `13.0` to `14.0` and the target framework to `net10.0-windows8.0`. Its first parent, `324455c95f6ecb4ce50f443215ebd90c0dfcbe76` ("Update ECommons", 2025-12-22), still declares SDK 13.0, but its only change moves `ECommons` to `c8167d22d8aca6732babc7cb5f39168f676478a8`, which targets `net10.0-windows7.0`. Restoring it against the Taiwan Dalamud fails with `NU1201: Project ECommons is not compatible with net9.0-windows7.0`. The base is therefore `324455c`'s first parent, which differs from it only in the submodule pointer.
- Other remotes compared: `reference` (<https://github.com/cycleapple/ChilledLeves>, branch `api13-tw`, tip `70db34a6b242350a295551ac244204bd7a1c8805`). Its merge base with upstream is this base, and it keeps the same `ECommons` revision. Upstream has no Taiwan branch.

## Changes

None. The branch is the upstream base unchanged.

The reference fork's six commits since the base were reviewed and not ported:

- Interface translation (`65cd15e`, `54da61d`, `7e0473c`, `4c36c28`, and the translated labels in `70db34a`): skipped, the interface is not translated.
- Version and metadata (`a7c46bc` adds its own `RepoUrl`; the `Version` raise to `1.0.1.22`; the translated `Description`, `Punchline` and `Tags`): skipped, the build does not need them.
- Behaviour change (`70db34a`, "Guard vnavmesh IPC startup race"): wraps vnavmesh IPC in `SafeWrapper.AnyException` and gates the Start buttons on vnavmesh reporting ready rather than installed. It is not specific to the Taiwan client, so it was skipped.

The base was searched for Taiwan failure patterns. No plugin code switches on `ClientLanguage` or reads an English sheet. Leve and quest names come from the client-language `Leve` and `Quest` sheets and are compared with addon text, and dialogs are answered by callback index, not by matching game text. Nothing there shows a failure on the Taiwan client, so the remaining assumptions are listed under Runtime verification.

## Build and package

`$DALAMUD_HOME` (machine-local environment variable) points at the Taiwan Dalamud API 13 injector directory. The commands pass it explicitly as `DalamudLibPath`. Initialise the submodule first with `git submodule update --init --recursive`.

Build command, run from the repository root:

```bash
dotnet restore ChilledLeves/ChilledLeves.csproj --locked-mode "-p:DalamudLibPath=$DALAMUD_HOME/" --nologo
dotnet build ChilledLeves/ChilledLeves.csproj -c Release --no-restore "-p:DalamudLibPath=$DALAMUD_HOME/" --nologo
```

Last verified build: 2026-11-27, commit `28d30255c7fb65c3defb27dc36ef7a32325d0be0`. The branch adds only these docs on top, so the source is identical to that commit.

- `DalamudApiLevel`: 13.
- `AssemblyVersion`: 1.0.1.18.
- Package: `ChilledLeves/bin/Release/ChilledLeves/latest.zip`.
- `latest.zip` SHA-256: `40876adf00054136fc73309837619fcf528091c5cb23217653b16046c2b0e5dc`.

Signature scan (`scripts/sigscan.py` in the home repository, against the Taiwan `ffxiv_dx11.exe`): 14 patterns, all in the bundled ECommons, none missing. The plugin defines no fixed memory offsets of its own.

## Runtime verification

Confirmed in game: none.

Pending in the Taiwan client:

- Load and reload ChilledLeves with vnavmesh installed.
- Picking up a leve: the `GuildLeve` list read through fixed `AtkValues` indices (25, 626 onward, 1233) in `ChilledLeves/Utilities/GuildLeve.cs` returns the Taiwan leve names, and the name in `JournalDetail` node 19 matches the `Leve` sheet name.
- The `SelectString` menu indices for each leve NPC (class choice and leave buttons) select the intended entries.
- Turning in: `SelectIconString` entries match the `Quest` sheet names, and the HQ confirmation, `JournalResult` and multi-turn-in steps complete.
- Class change before a leve and the gathering mode loop.

## Rebase policy

Do not move `api13` to `324455c` or later: `324455c` pins a net10 ECommons and `d018741` moves to API 14, so no newer upstream revision builds against API 13 unchanged. A later upstream fix may be cherry-picked on its own once it builds against the Taiwan Dalamud with the ECommons revision above.
