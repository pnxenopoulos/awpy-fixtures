# awpy-fixtures

Test-fixture **Counter-Strike 2 demo files** for [Awpy](https://github.com/pnxenopoulos/awpy).

Demos are large (tens to hundreds of MB) and can't live in the main Awpy
repository, so this repo distributes them as **GitHub release assets**. Awpy's
test suite downloads the fixtures it needs on demand and caches them locally.

The goal is coverage across demo **sources**, because each records in slightly different ways. 
Testing against all of them keeps the parser honest and lets us validate
Awpy's entity-derived datasets (`blinds`, `item_events`, the snapshot economy)
against the ground-truth events where they exist.

## Related repositories

- **[Awpy](https://github.com/pnxenopoulos/awpy)** — the Counter-Strike 2 demo
  parser these fixtures test.
- **[awpy-data](https://github.com/pnxenopoulos/awpy-data)** — map artifacts
  (geometry meshes, nav meshes, radar images, `map_data.json`), distributed as
  GitHub release assets in the same way. This repo is the demo counterpart.

## Available fixtures

One release per demo. The asset is named `<source>-<map>-<match-id>.dem` (the
match id keeps future demos of the same source + map distinct), and the release
**tag** matches the asset name without `.dem`. "Source" is what the parser's
coverage hinges on — each platform records a different subset of the demo
protocol.

| Asset | Source | Map | Result | SHA-256 | Match |
| --- | --- | --- | --- | --- | --- |
| `valve-de_mirage-472514809.dem` | Valve matchmaking | de_mirage | 13–0 | `2e0a4f9aa257…` | [csstats.gg](https://csstats.gg/match/472514809) |
| `faceit-de_mirage-6de46e85.dem` | FACEIT | de_mirage | 16–14 | `b1295d828786…` | [FACEIT room](https://www.faceit.com/en/cs2/room/1-6de46e85-0b21-420d-8c3b-df2737fc0067) |
| `esea-de_dust2-2c5bdd12.dem` | ESEA Season | de_dust2 | 6–13 | `734173f2fc14…` | [FACEIT room](https://www.faceit.com/en/cs2/room/1-2c5bdd12-6a65-4823-b774-fcf0e257a059) |
| `hltv-de_inferno-2395492.dem` | HLTV (Perfect World GOTV) | de_inferno | 13–10 | `054d633e5ec0…` | [HLTV](https://www.hltv.org/matches/2395492/9z-vs-parivision-xse-pro-league-guangzhou-2026) |

The `SHA-256` column shows the first 12 hex characters as a fingerprint; the
**full checksums** (for verifying a download) live in
[`manifest.json`](manifest.json). Every fixture is SourceTV-recorded, but the
recording configs differ (e.g. matchmaking/broadcast GOTV strip `player_blind`,
chat, and round events that some league configs keep).

[`manifest.json`](manifest.json) is the machine-readable index — for each
fixture it records the download URL, the full SHA-256 checksum, and the
**ground-truth scoreboard** (final score plus per-player kills / deaths /
assists, ADR, HS%, KAST, and flashes) that the stats-module tests assert
against.

## Publishing a fixture

Each demo is its **own** GitHub release — one release per fixture, with the tag
equal to the asset name (minus `.dem`). No compression is needed: a CS2 demo is
well under GitHub's 2 GB per-asset limit, and release assets don't count against
repo size. (Compress with `zstd`/`bzip2` only if you specifically want smaller
downloads — the raw `.dem` keeps the Awpy-side downloader dependency-free.)

### 1. Name the asset

Use `<source>-<map>-<match-id>.dem`, all lowercase. The **match id** (from the
source URL — a csstats/HLTV match number, or the first segment of a FACEIT room
id) keeps future demos of the same source + map distinct:

```
valve-de_mirage-472514809.dem
faceit-de_mirage-6de46e85.dem
hltv-de_inferno-2395492.dem
```

Keep the name **stable** once released — the test suite references assets by
name, so renaming one breaks it (mint a new name instead). Start from a `.dem`
you have the right to redistribute.

### 2. Publish the release

With the [`gh`](https://cli.github.com/) CLI (attach the demo as the positional
asset; the tag — the asset name minus `.dem` — is created on the fly):

```sh
gh release create valve-de_mirage-472514809 \
  --repo pnxenopoulos/awpy-fixtures \
  --title "valve-de_mirage-472514809" \
  --notes "Valve matchmaking · de_mirage · 13–0. https://csstats.gg/match/472514809" \
  ./valve-de_mirage-472514809.dem
```

(You can also create the release and drag the file onto it in the GitHub web
UI.) The download URL is then stable and predictable — the asset name repeats in
the path because the tag matches it:

```
https://github.com/pnxenopoulos/awpy-fixtures/releases/download/valve-de_mirage-472514809/valve-de_mirage-472514809.dem
```

### 3. Record it in `manifest.json`

Add an entry with the asset name, download URL, its **SHA-256** (`sha256sum
<file>`), and the match's **final score and per-player scoreboard**. Those are
the values Awpy verifies against — the checksum guards the download, and the
more of the scoreboard you capture (K / D / A, ADR, HS%, KAST, multikills,
flashes) the more the fixture can test. Mirror the entry as a row in the
**Available fixtures** table above.

## License

Demo files remain the property of their respective owners and are included here
solely for automated testing. Only add demos you have the right to redistribute.
