# Regenerating the Apocalypse conversion

`src/Apocalypse/ApocalypseESP/` is Enai Siaion's plugin with our changes applied. It is **committed
in full** so CI can build it, but it is *derived* — these scripts are how it was produced, and how it
gets reproduced against a new Apocalypse version.

They are also the reason a version bump is a re-run rather than a re-investigation. Every one asserts
what it changed and throws on zero matches, per CLAUDE.md guardrail 11.

## Why this is a replacement plugin and not a patch

**Enderal SE runs Skyrim SE 1.5.97, and that engine silently refuses any plugin whose `HEDR` form
version is 1.71** — no warning, no log line, the plugin is simply absent from the game. Apocalypse
ships at 1.71, so it never loaded in Enderal at all. A patch plugin cannot fix that from outside,
because it has to declare Apocalypse as a master and the engine has already skipped it — which
produces a null-pointer CTD during data load. The plugin itself must be rebuilt at **1.70**.

That single fact drives the whole architecture. Do not "simplify" this back into a patch.

## Order

Run against a fresh `reference/mods/Apocalypse/esp/` produced by `/spriggit-decompile-reference`.

| # | Script | Does |
|---|---|---|
| 1 | `01-gen-renames.ps1` | Elder Scrolls proper nouns → Enderal equivalents, across every user-visible string |
| 2 | `02-gen-distribution.ps1` | Builds the six tome/scroll sublists and injects them into Enderal's nine vendor/loot lists |
| 3 | `03-forward-leveled-lists.ps1` | Rebuilds those nine host lists from the **winning** record — Forgotten Stories overrides eight of them, and building from base Enderal silently reverts FS's edits |
| 4 | `04-forward-worldspace.ps1` | Replaces Apocalypse's `Tamriel` override of `00003C` with Enderal's `MQP01Home`, **keeping Apocalypse's three persistent refs** — a quest and a faction still point at them |
| 5 | `05-merge-tree.ps1` | Merges Enai's tree with our edits, drops the 67 staff recipes, re-homes our six new records into Apocalypse's own FormID space, writes the header at form version 1.7 |

The AddonNode re-index (`WB_IllusionNightmare_MPS_Seidsigil` 110 → 746) is a single committed record,
not a script. `verify-addonnode-indices.ps1` is what found the collision.

## Verify

| Script | Checks |
|---|---|
| `verify-plugin-census.ps1 <orig.esp> <built.esp>` | record counts by signature, FormID set, masters, `HEDR`. Expect −67 `COBJ`, +15 `LVLI`, `HEDR 1.7`, no `Dragonborn.esm` |
| `verify-dangling-diff.ps1` | unresolved references **relative to Enai's original**. Apocalypse already points at hundreds of vanilla FormIDs Enderal lacks; only *new* ones are ours. Expect **0 new** |
| `verify-addonnode-indices.ps1` | no `ADDN` index shared with Enderal |
| `verify-plugin-structure.ps1 <esp>` | header, masters, group/record framing |
| `debug-make-masters.ps1` | builds a hand-written plugin with a chosen master list and no records — the control that isolates a load crash to the header rather than the records |

## FormID allocation

Our six new records live in **Apocalypse's own space at `1C1E71`–`1C1E76`** (its highest own FormID
is `1C1E70`). This is not an ESL block — the merged plugin has ~3,890 records and is a full ESP.

## Gotchas

- The plugin **must** declare `Enderal - Forgotten Stories.esm` as a master. The forwarded leveled
  lists contain 63 FS FormKeys; without it Spriggit fails with "Could not map FormKey to a master
  index".
- Set `ModHeader.Stats.Version: 1.7` explicitly. Mutagen defaults to **1.71**, which is exactly the
  value that makes the plugin invisible to Enderal.
- Never re-add a `Dragonborn.esm` master. After step 5 the tree should contain zero matches for it.
