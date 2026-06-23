# assets

Static game assets for the [talus-engine](https://github.com/talus-engine)
project, served directly via `raw.githubusercontent.com` (plain git —
**no LFS**, see `.gitattributes`).

## Layout

The tree mirrors the engine's old `public/` layout so persisted asset
references resolve unchanged:

```
assets/sprites/<pack>/...   sprite packs (PNG/GIF/WebP; spritesheet.json atlases;
                            <name>.pyxel/ directories are pre-expanded archives)
assets/audio/<group>/...    OGG/WAV/MP3
manifest.<source>.json      remote-asset manifests, one per source (see below)
```

The engine consumes this repo two ways:

1. **Direct fetch** — `assetUrl()` in the engine points at
   `https://raw.githubusercontent.com/talus-engine/assets/main/`,
   so every `assets/...` path here is a live URL.
2. **Remote-manifest sources** — each `manifest.*.json` is a
   `foxtrot-asset-manifest`-format document (the engine's wire-format
   id) the in-app file manager registers under `/Remote/` for
   browsing + drag-import.

## Regenerating after asset changes

From the engine repo checkout:

```sh
# If .pyxel archives were added, expand them in place first:
npx tsx scripts/expandPyxelArchives.ts ~/gh/assets/assets/sprites

# Regenerate the affected manifest(s). Only the manifest(s) whose subtree
# you touched need rebuilding — except sprites-core (see note below).

# Dedicated per-pack manifests (each scopes one --subdir):
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-modern-exteriors --title="Modern Exteriors" --subdir=assets/sprites/modernexteriors-win --out=manifest.modern-exteriors.json
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-modern-interiors --title="Modern Interiors" --subdir=assets/sprites/moderninteriors-win --out=manifest.modern-interiors.json
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-modern-farm --title="Modern Farm" --subdir=assets/sprites/Modern_Farm_v1.2 --out=manifest.modern-farm.json
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-atomic-realm-tiles --title="Atomic Realm Tiles" --subdir=assets/sprites/atomic_realm_tiles --out=manifest.atomic-realm-tiles.json
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-character-template --title="Character Template" --subdir=assets/sprites/character-template --out=manifest.character-template.json

# Audio:
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-audio --title="Audio" --subdir=assets/audio --out=manifest.audio.json

# Catch-all over assets/sprites MINUS the three modern packs (which have their
# own manifests). It does NOT exclude atomic_realm_tiles / character-template,
# so those two subtrees are intentionally double-indexed — once in their
# dedicated manifest above, once here. Regenerate sprites-core for ANY change
# under assets/sprites (new packs like vfx_slash/, weapons/ are picked up here),
# AND additionally rebuild the dedicated manifest if you touched a modern pack
# or atomic_realm_tiles / character-template.
npx tsx scripts/buildAssetManifest.ts ~/gh/assets --slug=talus-sprites-core --title="Core Sprites" --subdir=assets/sprites --exclude=assets/sprites/modernexteriors-win --exclude=assets/sprites/moderninteriors-win --exclude=assets/sprites/Modern_Farm_v1.2 --out=manifest.sprites-core.json

# Regenerate the engine's committed asset-index JSONs and commit them there:
npx tsx scripts/buildAssetIndex.ts ~/gh/assets
```

Then commit + push both repos.
