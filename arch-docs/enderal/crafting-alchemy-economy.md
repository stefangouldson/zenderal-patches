# Crafting, alchemy and the economy

Enderal keeps Bethesda's crafting *plumbing* — bench keywords, COBJ records, alchemy keywords — and
builds a much larger and differently-balanced system on top of it. This is the friendliest system to
patch, because the mechanism is familiar.

## Bench keywords are vanilla, and that's genuinely useful

**[verified]** Enderal retained the vanilla crafting-bench keywords with their vanilla FormIDs:

| Keyword | FormKey | Base recipes | FS recipes |
|---|---|---:|---:|
| `CraftingSmithingForge` | `088105:Skyrim.esm` | 246 | 247 |
| `CraftingSmithingArmorTable` | `0ADB78:Skyrim.esm` | 169 | 184 |
| `CraftingSmithingSharpeningWheel` | `088108:Skyrim.esm` | 132 | 105 |
| `CraftingTanningRack` | `07866A:Skyrim.esm` | 55 | 43 |
| `CraftingCookpot` | `0A5CB3:Skyrim.esm` | 21 | 2 |
| `CraftingSmelter` | `0A5CCE:Skyrim.esm` | 16 | 3 |

Also present: `isAlchemy`, `isBlacksmithForge`, `isBlacksmithAnvil`, `isBlacksmithWorkbench`,
`WICraftingAlchemy`, `WICraftingEnchanting`, `WICraftingSmithing`, `WICraftingSmithingTempering`.

> **A recipe patch is one of the few Skyrim patterns that ports almost unchanged to Enderal.** Point
> a `ConstructibleObject` at `088105:Skyrim.esm` and it appears at the forge, exactly as in Skyrim.
> The *items* are Enderal's, but the mechanism is Bethesda's.

## Recipe inventory

**639 ConstructibleObjects in base Enderal, 1220 in Forgotten Stories** — 1859 total. **[verified]**

FS more than doubles the recipe count, and the reason is a single system:

| FS bench keyword | Recipes | What it is |
|---|---:|---|
| `InvisibleDismantling` `02F336:Enderal - Forgotten Stories.esm` | **587** | the dismantle/salvage system |
| `CraftingSmithingForge` `088105` | 247 | |
| `CraftingSmithingArmorTable` `0ADB78` | 184 | |
| `CraftingSmithingSharpeningWheel` `088108` | 105 | |
| `_00E_Phasmalist_CraftingSummoningWorkbench` `01E945` | 48 | Phasmalist summoning workbench |
| `CraftingTanningRack` `07866A` | 43 | |
| `_00E_FS_MQ18c_WorkbenchElixir` `01C675` | 1 | one-off quest bench |

### The dismantle system

Nearly half of FS's recipes are **dismantling** — turning gear back into materials. They use the
`InvisibleDismantling` keyword and `_00E_Dismantle*` EditorIDs, e.g.
`_00E_DismantleArmor_Circlet01ToSteelIngot` (`02E7A0`). **[verified]** Each is a COBJ whose condition
is a `GetItemCount >= 1` on the source item.

Supporting records: `_00E_DismantleList`, `_00E_DismantleItemList`, `_00E_DismantleResetList`
FormLists, the `CraftingSmelterDismantling` (`02F333`) and `InvisibleDismantling` (`02F336`)
keywords, and `_00E_SE_DismantlePerk` (`014CFA`). **[verified]**

> **Patch note.** Any mod that adds gear should consider whether it needs dismantle recipes. Gear
> without them is an outlier the player will notice — everything else in the game can be broken down.
> Conversely, a mod that changes *materials* must check the 587 dismantle recipes for outputs that
> no longer make sense.

## Alchemy keywords are vanilla too

The `MagicAlch*` family is intact **[verified]**: `MagicAlchBeneficial`, `MagicAlchDamageHealth`,
`MagicAlchDamageMagicka`, `MagicAlchDamageStamina`, `MagicAlchDurationBased`,
`MagicAlchFortify{Alchemy,Alteration,Block,CarryWeight,Conjuration,Destruction,Enchanting,HealRate,…}`.

So potion-effect classification works the vanilla way. **55 Ingredients and 66 Ingestibles** ship in
FS alone. **[verified]**

## Arcane Fever

Enderal's signature survival mechanic, and the one most likely to be broken by a careless potion or
healing patch.

**It is stored in the repurposed vanilla ActorValue `LastFlattered`, negated.** **[verified]**

```papyrus
; _00E_FS_AlchAddArcaneFever
float fArcaneFeverAdd = Self.GetBaseObject().GetArea()
akTarget.ModAV("LastFlattered", -fArcaneFeverAdd)
_00E_Player_sArcaneFeverIncreased.Show(fArcaneFeverAdd, -1*(akTarget.GetAV("LastFlattered")))
if bVisuals
    _00E_ArkanistenfieberIMOD.Apply()
endif
_00E_FS_IncreaseArcaneFeverM.Play(akTarget)
```

Three things to notice:

1. **The magnitude comes from the magic effect's `Area` field** (`GetBaseObject().GetArea()`) — not
   from magnitude or duration. A patch that "tidies up" an unused Area value on an Arcane-Fever MGEF
   silently changes how much fever the potion inflicts.
2. **The sign is inverted.** Higher fever = more negative `LastFlattered`.
3. **The visual and sound are part of the contract.** `_00E_ArkanistenfieberIMOD` is the player's
   only feedback that fever went up. A blanket imagespace override removes the signal while leaving
   the mechanic.

Related records and scripts **[verified]**: `_00E_MagicAlchArcaneFever` keyword,
`_00E_ArkanistenfieberEffect`, `_00E_ArkanistenfieberTriggerbox`,
`_00E_ArkanistenfieberBlitzheilungSCN`, `_00E_ArkanistenfieberWohltatSCN`, `_00E_AmbrosiaEffect`,
`_00E_PeaceweedPlayerAliasScript`, `_00E_FS_IronPathFeverBoost` (global).

Perks that reduce it: `_00E_Class_Theriantrophist_P04c_LessArcaneFever_01/_02`. **[verified]**

## Class crafting (Forgotten Stories)

Both FS classes gate crafting behind tiered perks **[verified]**:

- **Phasmalist** — `_00E_Class_Phasmalist_P02_CraftTier1`, `P05_CraftTier2`, `P06_CraftTier3`,
  `P07_CraftTier4`, `P09_CraftTier5`. Uses the summoning workbench (`01E945`) and consumes soul gems
  and ectoplasm per `_00E_Phasmalist_SpectralizeSoulGemCost` /
  `_00E_Phasmalist_SpectralizeBonemealEctoplasmaCost`.
- **Theriantrophist** — `_00E_Class_Theriantrophist_P02_Talent_Laboratory_01/02/03`, the
  `_00E_Theriantrophist_Chymikum*` keyword family, and `_00E_Theriantrophist_BlockCraftingSC`
  (blocks crafting in werewolf form).

## Economy

- **Crafting Points (`Handwerkspunkte`)** are a distinct currency from Learning Points and Memory
  Points — see [`progression-and-classes.md`](progression-and-classes.md). Crafting skills are bought,
  not trained by use.
- **114 LeveledItems in FS** **[verified]** — these are the records the Spriggit 0.40.0 pin exists to
  protect. Leveled-list entries carrying owner ExtraData are exactly what Spriggit 0.41.0 corrupts.
  See CLAUDE.md → "Why Spriggit 0.40.0 is pinned".
- Vendor gating uses keywords like `_00E_FS_NQ07_VendorNoSale` and `VendorItemFSBlueprintContainer`.
  **[verified]**

## Enemy loot

Verified against `reference/base/` on 2026-08-02. The headline is that **enemy loot is
hand-authored**, so most of what you'd reach for in Skyrim is not there:

- **Zero `LeveledNpc` (LVLN) records** in either plugin — no `LeveledNpcs/` folder exists. Every
  actor is placed by hand. **[verified]**
- **Zero NPCs inherit `Inventory` from a template** — 652 carry `Template:`, none list `Inventory`
  under `Configuration.TemplateFlags`. **[verified]**
- Most hostiles hold a **direct weapon record** rather than a list (`_04E_Magier01_Range1100`
  `04CFDB` → `Items: [0028D2 _01E_02_IronDagger]`). **[verified]**

**The three lists Enderal's enemies actually share** — note they split by *level band*, not faction:

| FormKey | EditorID | NPCs | Population | Winning source |
|---|---|---:|---|---|
| `04C982:Skyrim.esm` | `00E_MOB_Bandit` | 109 | `_10E_`–`_20E_` bandits, Skaragg, Sunborn, Plunderers, bounty targets | `Skyrim.esm` (FS does not override) |
| `02F32B:Enderal - Forgotten Stories.esm` | `_00E_FS_DeathItem_Human` | 106 | `_03E_`/`_05E_` bandits, FS quest bandits | FS |
| `03AD7F:Skyrim.esm` | `DeathItemDraugr` | 186 | Lost Ones of every variant, Zu'Sherath, forest elementals | `Skyrim.esm` (FS does not override) |

Union **353** base NPC records; 46 carry the first two both. Magier (59), Vatyr (56), Arpsplitter
(36), Skelett (98) and Hexe (12) are reachable **only** by overriding the NPC records themselves.

Two traps found the hard way and documented in full in CLAUDE.md → "Adding loot without taking any
away":

1. **Appending to a leveled list dilutes it.** A list resolves to one entry, so your addition
   *replaces* the original loot a fraction of the time. Wrap the original in a `UseAll` parent
   instead — Enderal's own `LootSmokingPipePeaceweed1` (`04815B`) is that pattern. **[verified]**
2. **Never append to a list with no `Flags:` and no `ChanceNone:`.** Those are single-pick **weapon**
   lists (`00E_MOB_BanditWeapon01` `014EBA`, `03E_MOB_SkelettWeapon01` `090A14`) — adding loot makes
   the enemy spawn holding it *instead of a weapon*. **[verified]**

### Potions

Potions in Enderal are an alchemist purchase, not a combat reward: `01E_Traenke` `0028E3` is on
**8** NPC records, seven of them corpse props, and `01E_FS_ClutterUseful` `02E68C:EFS` — the one
list holding all 15 restore potions properly level-banded — sits on **75 citizens**. **[verified]**
`Zenderal - Enemy Potions.esp` closes that gap via the three hooks above.

The three restore families, by tier (all `:Skyrim.esm`). **Stamina is `Morgenlufttrank`, not
`Ausdauertrank`** — the latter is *fortify* stamina, a false friend. Base effects are
`00E_AlchRestoreHealth` `0028C3`, `00E_AlchRestoreMagicka` `0028DD`, `00E_AlchRestoreStamina`
`0028DC`. **[verified]**

| Tier | Health (`…Genesungstrank`) | Mana (`…Manatrank`) | Stamina (`…Morgenlufttrank`) |
|---|---|---|---|
| 01E (Rancid) | `0028C8` | `0028DB` | `0028DE` |
| 02E (Cheap) | `0028C5` | `019E3B` | `085668` |
| 03E (Standard) | `0028C6` | `090892` | `09B6CA` |
| 04E (Quality) | `0028C7` | `09B6CB` | `1037F5` |
| 05E (Exclusive) | `0028C9` | `1037F7` | `1037F6` |

FS overrides the five `Genesungstrank` ALCH records but not the other ten — irrelevant if you only
*reference* them, since references resolve to the winning version. Enderal's own banding, from
`01E_FS_ClutterUseful`, is level **1 / 10 / 20 / 30 / 40** with
`CalculateFromAllLevelsLessThanOrEqualPlayer` so lower tiers stay in circulation. **[verified]**

## Checklist for a crafting/economy patch

- [ ] Does the recipe point at a real bench keyword? (vanilla FormIDs above)
- [ ] If it adds gear, does it need dismantle recipes to match everything else?
- [ ] If it changes materials, do the 587 dismantle outputs still make sense?
- [ ] Does it touch an Arcane-Fever MGEF's **Area** field? That's the fever magnitude.
- [ ] Does it write `LastFlattered` for any reason? That's Arcane Fever, not a vanilla stat.
- [ ] Does it edit leveled lists? Confirm the Spriggit version pin before building.
- [ ] Is it *adding* to a leveled list? Check the target's `Flags:` first — append only to `UseAll`
      lists, wrap anything else, and never touch a no-flags `*Weapon*` list.
- [ ] Is it distributing to enemies? There is no LVLN layer and no template inheritance — only the
      three shared lists above reach more than a handful of NPCs.
- [ ] Does it assume alchemy skill rises from use? It doesn't.
