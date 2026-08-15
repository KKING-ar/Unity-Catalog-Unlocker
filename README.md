# 🔓 Unity Catalog Unlocker
### Addressables CRC Patcher by AnyThing

A standalone, offline tool that disables the built-in CRC / integrity checks in Unity Addressables catalogs, so you can freely edit or replace asset bundles without the game rejecting them as "corrupted". No installation, no coding, no internet required — just drop the file and it works.

---

## The Problem

Unity Addressables catalogs (`catalog.bin` or `catalog.json`) store a CRC checksum for every asset bundle. When you modify a bundle — for localization, modding, or asset replacement — the game recomputes the bundle's CRC at load time, finds it doesn't match the stored value, and refuses to load it as "corrupted", even though your edit is perfectly valid.

**Without this tool:**
```
edited bundle → CRC mismatch → Addressables refuses to load it
```
**With this tool:**
```
edited bundle → CRC check disabled → loads normally
```

---

## How to Use

1. **Open** the app — after installation
2. **Pick the format** — `catalog.bin` or `catalog.json` (also auto-detected from the dropped file)
3. **Drop** your catalog file — every bundle entry is scanned and listed with its current CRC and size
4. **Click** "Disable All CRC Checks" — every CRC field is zeroed and a new `catalog.hash` is computed
5. **Download** the patched catalog and the new hash file, and replace the originals in your game's `StreamingAssets` folder

That's it. No coding. No hex editor required.

---

## Features

- **Two catalog formats** — handles both binary (`catalog.bin`) and text (`catalog.json`) Addressables catalogs
- **Binary structure parsing** — scans the raw `BinaryStorageBuffer` layout to locate every bundle's hash, CRC, and size fields directly, without needing Unity source access
- **JSON field patching** — renames every `"m_Crc"` key to `"d_Crc"` in the JSON catalog, a field name Unity's Addressables loader ignores, disabling the check without touching the JSON structure
- **Bundle entry table** — lists every detected bundle with its name, hash, current CRC status (active or disabled), and size before you commit to patching
- **Automatic hash recompute** — generates a correct `catalog.hash` (MD5) for the patched file, since Unity validates the catalog against this hash before it even reads the bundle entries
- **Bilingual UI** — English / عربي toggle for the interface and the in-app guide, with automatic RTL layout in Arabic
- **Fully offline** — everything runs on you machine; no file is ever uploaded anywhere

---

## What Gets Patched

| Format | What changes | Why it works |
|--------|--------------|---------------|
| **catalog.bin** | Every bundle's 4-byte CRC field is zeroed in place, directly in the binary buffer | Unity treats a CRC of `0` as "no check" for that bundle |
| **catalog.json** | Every `"m_Crc"` key is renamed to `"d_Crc"` | The Addressables loader only recognizes the exact field name `m_Crc`; renaming it makes the check a no-op |
| **catalog.hash** | Regenerated as the MD5 of the patched catalog file | Unity checks this hash before loading the catalog at all, so it must match the patched file, not the original |

---

## After Patching

Replace these two files in your game's `StreamingAssets` folder (or wherever your build stores Addressables catalogs), overwriting the originals:

- The patched `catalog.bin` (or `catalog.json`)
- The new `catalog.hash`

`settings.json`, if present, needs no changes — its embedded hash relates to Addressables settings, not catalog content.

---

## Technical Details

- Pure HTML — zero external runtime dependencies
- Binary parsing scans for Unity's `BinaryStorageBuffer` hash-string pattern (a 32-byte hex hash string followed by reference offsets, a CRC `uint32`, and a size `uint32`), validated against plausible bundle-size and CRC-value ranges to avoid false positives
- Bundle names are recovered by scanning backwards from each matched entry for a `.bundle`-suffixed string
- JSON catalogs are patched via direct string field renaming; catalogs with an embedded `m_ExtraDataString` (base64-encoded binary blob) are also decoded and scanned for bundle entries for visibility
- MD5 is implemented in pure JavaScript (no external crypto library) to compute `catalog.hash` for the patched output
- Fonts: [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono) and Space Mono, both open source (OFL 1.1)

---

## License

MIT — see [LICENSE](LICENSE)

---

## Credits

Built by **اي حاكَة (AnyThing)** for the Unity modding and localization community — to make CRC unlocking safe, visible, and reversible-by-inspection, without needing a hex editor or the original Unity project.

Fonts: [JetBrains Mono](https://github.com/JetBrains/JetBrainsMono) by the JetBrains Mono Project Authors and Space Mono by Colophon Foundry (both SIL Open Font License 1.1)
