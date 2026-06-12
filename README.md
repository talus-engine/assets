# foxtrot-assets

Static game assets for [foxtrot](https://github.com/talus-engine/foxtrot),
served directly via `raw.githubusercontent.com` (plain git — **no LFS**,
see `.gitattributes`).

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
   `https://raw.githubusercontent.com/talus-engine/foxtrot-assets/main/`,
   so every `assets/...` path here is a live URL.
2. **Remote-manifest sources** — each `manifest.*.json` is a
   `foxtrot-asset-manifest` the in-app file manager registers under
   `/Remote/` for browsing + drag-import.

## Regenerating after asset changes

From the engine repo checkout (`talus-engine/foxtrot`):

```sh
# If .pyxel archives were added, expand them in place first:
npx tsx scripts/expandPyxelArchives.ts ~/gh/foxtrot-assets/assets/sprites

# Regenerate the affected manifest(s):
npx tsx scripts/buildAssetManifest.ts ~/gh/foxtrot-assets --slug=foxtrot-modern-exteriors --title="Modern Exteriors" --subdir=assets/sprites/modernexteriors-win --out=manifest.modern-exteriors.json
npx tsx scripts/buildAssetManifest.ts ~/gh/foxtrot-assets --slug=foxtrot-modern-interiors --title="Modern Interiors" --subdir=assets/sprites/moderninteriors-win --out=manifest.modern-interiors.json
npx tsx scripts/buildAssetManifest.ts ~/gh/foxtrot-assets --slug=foxtrot-modern-farm --title="Modern Farm" --subdir=assets/sprites/Modern_Farm_v1.2 --out=manifest.modern-farm.json
npx tsx scripts/buildAssetManifest.ts ~/gh/foxtrot-assets --slug=foxtrot-sprites-core --title="Core Sprites" --subdir=assets/sprites --exclude=assets/sprites/modernexteriors-win --exclude=assets/sprites/moderninteriors-win --exclude=assets/sprites/Modern_Farm_v1.2 --out=manifest.sprites-core.json
npx tsx scripts/buildAssetManifest.ts ~/gh/foxtrot-assets --slug=foxtrot-audio --title="Audio" --subdir=assets/audio --out=manifest.audio.json

# Regenerate the engine's committed asset-index JSONs and commit them there:
npx tsx scripts/buildAssetIndex.ts ~/gh/foxtrot-assets
```

Then commit + push both repos.
