# IntelEngine-SoldierPack.esp (Spriggit source)

This YAML is the source of `IntelEngine-SoldierPack.esp`. The `.esp` is built from it, so edit the YAML (or the CK, then serialize back), never only the `.esp`.

`build.ps1` builds it into `<DataDir>\IntelEngine-SoldierPack.esp`, and `verify.ps1` rebuilds it to a temp file and checks the staged copy by hash. Both need `SpriggitCli` (path to `Spriggit.CLI.exe`) as a parameter or in `build.local.psd1`.

By hand, Spriggit CLI 0.40.0, `Spriggit.Yaml.Skyrim`:

```powershell
# YAML -> esp
Spriggit.CLI deserialize -i Plugins/IntelEngine-SoldierPack -o <out>/IntelEngine-SoldierPack.esp -p Spriggit.Yaml.Skyrim -v 0.40.0
# esp -> YAML: run right after saving in the CK, before anything deserializes, or the next build overwrites the CK work
Spriggit.CLI serialize -i <path>/IntelEngine-SoldierPack.esp -o Plugins/IntelEngine-SoldierPack -g SkyrimSE -p Spriggit.Yaml.Skyrim -v 0.40.0
```

## Layout

- Companions soldiers for IntelEngine battles: `soldier_template: "IntelEngine-SoldierPack.esp|0x000802"` (`IESoldier_LCharCompAny`).
- Top list: 1H x5, 2H x5, archers x2. Six tiers each (player level 1/5/9/14/19/25, all levels at or below the player).
- Faces live only on the one-handed NPCs `IESoldier_EncComp0<tier>Melee1H<Race><Sex>` (48 records). Two-handed and archer NPCs take Traits from the 1H NPC of the same tier, race and sex.
- Master: Skyrim.esm only. FormIDs stay in the ESL range (0x800-0xFFF); the header is not ESL-flagged.

## Rules

- Never renumber existing records: baked FaceGen is named by FormID.
- New face: copy a 1H NPC of that tier, give it the next free FormID and a new editor ID, change only the face fields, add it to that tier's `IESoldier_SubCharComp0<tier>Melee1H` list, then bake FaceGen for that NPC.
- FaceGen (`meshes/.../facegeom/IntelEngine-SoldierPack.esp/*.nif`, `textures/.../facetint/IntelEngine-SoldierPack.esp/*.dds`) is binary and not part of this YAML.
