# ChilledLeves Taiwan API 13 branch

`api13` keeps LeontopodiumNivale14/ChilledLeves at an earlier Dalamud API 13 revision than its last one for the Taiwan client (game 7.3), plus the changes in `docs/TW_API13_CHANGES.md`. Read that record before changing this branch.

## Invariants

- Keep the `ECommons` submodule at `3aeec9a52b721646445e2828d120dee17591eb35` or another revision that still targets `net9.0-windows`. Upstream's next ECommons bump (`c8167d2`) targets `net10.0-windows` and does not restore against API 13.
