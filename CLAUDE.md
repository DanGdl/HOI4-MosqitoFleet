# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Mod Is

**MosquitoFleet** (v1.7.6, for HOI4 1.19.\*) adds small/cheap naval vessels to Hearts of Iron IV: torpedo/artillery fast attack craft (FAC), corvettes, frigates, midget submarines, hydrofoils, auxiliary ships, and merchant carriers. The mod is published on Steam Workshop (ID `3167195673`).

All mod-specific identifiers are prefixed with `mdg_`.

## How to Test / Run

There is no build system or test runner. Testing is done by launching HOI4 with the mod enabled:

- Copy or symlink the mod folder into `~/.local/share/Paradox Interactive/Hearts of Iron IV/mod/`
- The `descriptor.mod` in the root declares the mod metadata (name, version, supported HOI4 version)
- Launch HOI4, enable MosquitoFleet in the launcher, start a game
- HOI4 prints errors and warnings to `logs/error.log` (in the HOI4 user data directory) — check this after each change

The game reloads most script files on a new game start; entity/GFX assets require a full game restart.

## Code Architecture

### File naming convention
- Files inside `common/` that belong to this mod are prefixed `mdg_`
- Country-specific AI equipment files use the country tag prefix: `ENG_naval.txt`, `GER_naval.txt`, etc.
- History files follow the vanilla pattern: `TAG mdg CountryName.txt`

### Ship types added by this mod

| Type | Hull archetype | Unit type | Role IDs |
|------|---------------|-----------|----------|
| FAC (Fast Attack Craft) | `mdg_ship_hull_fac` | `mdg_fac` | `naval_screen_fac`, `naval_escort_fac`, `naval_mine_layer_fac` |
| Corvette | `mdg_ship_hull_corvette` | `mdg_corvette` | `naval_screen_corvette`, `naval_escort_corvette`, `naval_mine_sweeper_corvette`, `naval_mine_layer_corvette` |
| Frigate | `mdg_ship_hull_frigate` | `mdg_frigate` | `naval_screen_frigate` |
| Midget Submarine | `mdg_ship_hull_midget_submarine` | `mdg_midget_submarine` | `naval_submarine_midget` |
| Hydrofoil | `mdg_ship_hull_hydrofoil` | `mdg_hydrofoil` | `naval_screen_hydrofoil` |
| Auxiliary | `mdg_ship_hull_auxiliary` | `mdg_auxiliary` | `naval_auxiliary` |
| Merchant Carrier (MAC) | `mdg_ship_hull_mac` | `mdg_mac` | `naval_mac` |

Each ship type has 1–4 hull variants (`_1` through `_4`) that gate off vanilla tech milestones: `early_ship_hull_light` → `basic_ship_hull_light` → `improved_ship_hull_light` → `advanced_ship_hull_light`.
Ships of types `mdg_mac`, `mdg_auxiliary` are created via decisions in game, `mdg_hydrofoil` - from special project. They have single sub-type (not 4 like others).

### Directory structure

```
common/
  ai_equipment/	   # AI ship design templates (generic + per-country for ITA, JAP, USA, ENG, FRA, GER)
  ai_strategy/		# AI naval role ratios (mdg_naval.txt) and doctrine strategies
  decisions/		  # Refit decisions (convert convoys/civilian ships to warships)
  equipment_groups/   # MIO equipment groups
  ideas/			  # Navy spirits (mosquitos_navy_spirit)
  on_actions/		 # Event hooks (mdg_00_on_actions.txt)
  special_project	 # Special projects. Mod adds a special project to enable mdg_hydrofoil.
  script_enums.txt	# Declares mdg_ equipment types for MIO/equipment_bonus system
  technologies/	   # Tech tree additions grafted onto MTG_naval.txt and MTG_naval_Support.txt
  technology_tags/	# Custom tech tags
  terrain/			# Terrain modifiers
  unit_leader/		# Naval commander traits (mdg_00_traits.txt)
  units/			  # Sub-unit definitions (battalions)
  units/equipment/	# Hull archetypes and concrete hull variants
  units/equipment/modules/  # Custom weapon modules (FAC guns, light cannons)
gfx/
  entities/		   # 3D model references (.gfx + .asset) — DO NOT redistribute models without permission
  texticons/		  # Small ship-type icons for UI
interface/			# GUI layout files (.gfx, .gui) for equipment designer, navies view, traits window
history/countries/	# Starting equipment grants for ~42 countries (one file per tag)
localisation/		 # YML localisation files; english/ and russian/ are primary, others are machine-translated
```

### Technology integration

`common/technologies/MTG_naval.txt` replaces/extends the vanilla light ship tech tree. The hidden tech `mdg_hidden_tech` (always blocked, set at game start) enables all base hull/module variants. Corvette/frigate/FAC hulls are unlocked in parallel with vanilla destroyer hulls so nations can choose either path.

### AI equipment designs

`common/ai_equipment/mdg_*.txt` — generic designs blocked for major naval powers (ITA, JAP, USA) that get their own files. Each country-specific file (`ITA_mdg_corvettes.txt`, etc.) contains tailored module selections. The `blocked_for` / `available_for` keys control which countries use a design.

### Custom modules

`common/units/equipment/modules/mdg_01_fac.txt` — FAC-specific gun categories (`mdg_ship_light_guns`, used in FAC fixed_ship_battery_slot).  
`common/units/equipment/modules/mdg_02_cannons.txt` — Light cannon categories (`mdg_ship_light_cannon`, `mdg_ship_light_cannon_dp`) for corvettes and frigates.

### Localisation

Primary language is English (`localisation/english/`). All other languages are community/AI translations. YML files must be UTF-8 with BOM. Key files:
- `mdg_equip_naval_l_english.yml` — hull and equipment names
- `mdg_ship_modules_l_english.yml` — module names/tooltips
- `mdg_research_l_english.yml` — technology names
- `mdg_traits_l_english.yml` — admiral traits
- `mdg_decisions_l_english.yml` — refit decision text

### Interface / GFX

`interface/mdg_equipmentdesignerview*.gfx` — equipment designer layout, one file per major nation (plus a generic). If adding a module slot to a hull, update both the archetype in `units/equipment/` and the corresponding `.gfx` view file.

`interface/mdg_unitleaderwindow.gfx` — trait tree layout for naval commanders.
