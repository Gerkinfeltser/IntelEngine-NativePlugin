# IntelEngine-SoldierPack.esp (Spriggit source)

This YAML is the source of `IntelEngine-SoldierPack.esp`. The `.esp` is built from it, so edit the YAML (or the CK, then serialize back), never only the `.esp`.

`build.ps1` builds it into `<DataDir>\Optional\IntelEngine-SoldierPack\`, with `factions.d/companions_soldiers.yaml` beside it in Data layout, and `verify.ps1` rebuilds the esp to a temp file and checks both staged files by hash. That folder is the installer's "Companions soldiers" option (`fomod/ModuleConfig.xml` in GamePlugin); `build.ps1` also installs it on the test targets. Both need `SpriggitCli` (path to `Spriggit.CLI.exe`) as a parameter or in `build.local.psd1`.

By hand, Spriggit CLI 0.40.0, `Spriggit.Yaml.Skyrim`:

```powershell
# YAML -> esp
Spriggit.CLI deserialize -i Plugins/IntelEngine-SoldierPack -o <out>/IntelEngine-SoldierPack.esp -p Spriggit.Yaml.Skyrim -v 0.40.0
# esp -> YAML: run right after saving in the CK, before anything deserializes, or the next build overwrites the CK work
Spriggit.CLI serialize -i <path>/IntelEngine-SoldierPack.esp -o Plugins/IntelEngine-SoldierPack -g SkyrimSE -p Spriggit.Yaml.Skyrim -v 0.40.0
```

## Layout

- Companions soldiers for IntelEngine battles: `soldier_template: "IESoldier_LCharCompAny"` (record 0x000802). Use the name: `SpawnBattleSoldiers` and the quest spawns look templates up by name only, so `"IntelEngine-SoldierPack.esp|0x000802"` falls back to bandits there (seen in game 2026-11-17).
- Top list: 1H x5, 2H x5, archers x2. Six tiers each (player level 1/5/9/14/19/25, all levels at or below the player).
- Faces live only on the one-handed NPCs `IESoldier_EncComp0<tier>Melee1H<Race><Sex>` (72 records: 12 race/sex slots per tier). Two-handed and archer NPCs take Traits from the 1H NPC of the same tier, race and sex.
- 1H draws per tier (20): Nord M x6, Nord F x4, and one each of Redguard M/F, Orc M/F, Dark Elf M/F, Wood Elf M/F, Imperial M, Breton M. Imperial M, Orc F, Breton M and Redguard F (0x8BA-0x8D1) are one-handed only.
- Master: Skyrim.esm only. FormIDs stay in the ESL range (0x800-0xFFF); the header is not ESL-flagged.

## Rules

- Never renumber existing records: baked FaceGen is named by FormID.
- New face: copy a 1H NPC of that tier, give it the next free FormID and a new editor ID, change only the face fields, add it to that tier's `IESoldier_SubCharComp0<tier>Melee1H` list, then bake FaceGen for that NPC.
- No FaceGen is shipped: the installer option names Face Discoloration Fix (Nexus 42441), which builds the faces at run time (seen working in game 2026-11-18).
- FaceGen (`meshes/.../facegeom/IntelEngine-SoldierPack.esp/*.nif`, `textures/.../facetint/IntelEngine-SoldierPack.esp/*.dds`) is binary and not part of this YAML.

## Status (2026-11-20: on hold)

Branches: `feat/soldier-pack-yaml` (this repo) and `feat/soldier-pack-option` (GamePlugin). Both include the 3.8.3 merge. No PR is open.

Seen in game:
- The esp loads and its soldiers spawn from the console (2026-11-20, build before the aggression change). Such a soldier attacked the player: the cause was the bandit template's VeryAggressive, fixed in `7ca9141`, not yet seen in game.
- Face Discoloration Fix builds the faces at run time (2026-11-18, earlier faces).
- Templates are looked up by name only (2026-11-17).

Built and checked offline only, nobody has seen these in game:
- The 72 flame-lab faces on the 1H NPCs (own-race warpaint only).
- Aggressive instead of VeryAggressive on the six tier templates and the 24 archers (`7ca9141`).
- The installer option and its Companions `factions.d` fragment, which raises the Companions from `IESoldier_LCharCompAny`.
- Survivors walk home: `home_marker` in `factions.d` (DLL + `IntelEngine_Battle.psc`, `WayOff`). The Companions use `Skyrim.esm|0x0BC0BF` (C00JorrvaskrFightCenterPoint); that this marker is outdoors behind Jorrvaskr is not checked. Only full battles (`DisperseSoldiers`) use it; story ambushes and assassinations still calm and remove their actors as before.
- Offline checks: the esp round-trips clean through Spriggit, `verify.ps1` passes (95 checks), the DLL builds and both scripts compile with the real compiler.

Still to do before release: see the faces, the aggression and the walk home in a Companions battle; then a PR to fork `dev` and a changelog entry.
