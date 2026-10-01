# falcon-pdf-strings

Runtime localization catalog for **Falcon PDF**. The app downloads these JSON
files on demand (via the jsDelivr CDN) to localize its UI in languages that are
not bundled in the binary.

This repo is the **translation source-of-truth** for Falcon PDF. It holds:

- `sources/<module>/values[-<locale>]/strings.xml` — the per-module Android string
  resources. Crowdin downloads translations here; the base `values/strings.xml` is
  the English source Crowdin uploads. This repo is wired into the main PDFViewer
  repo as a submodule at `libs/falcon-pdf-strings/`.
- `v1/*.json` — the **generated** runtime catalog the app downloads. Produced from
  `sources/` by `scripts/localization/build_string_catalog.py` in the main repo.
  Do not hand-edit the JSON.

## Layout

```
sources/<module>/values/strings.xml           English source (per module)
sources/<module>/values-<locale>/strings.xml  translations (per module, per locale)
v1/manifest.json      locale index: per-locale revision, byte size, sha256, coverage
v1/en-rUS.json        English source catalog (100% coverage)
v1/<tag>.json         one generated catalog file per locale (Android -r tag form)
```

Module ids: `app`, `common`, `editor`, `tools`, `scanner`, `auth`.

`v<N>/` is the catalog **schema** version. The app pins its own schema version and
reads `v<N>/manifest.json` first, then each `v<N>/<tag>.json`, verifying every
locale file against the sha256 recorded in the manifest.

## How the app resolves a URL

```
https://cdn.jsdelivr.net/gh/avrapps/falcon-pdf-strings@<ref>/v1/manifest.json
https://cdn.jsdelivr.net/gh/avrapps/falcon-pdf-strings@<ref>/v1/<tag>.json
```

`<ref>` is a git tag/commit. The current app release tracks the floating tag
**`v1-latest`**. Pin a release to an immutable commit/tag for maximal CDN caching.

## Compatibility contract

Within a given `v<N>/` the catalog is **additive-only**: only ADD keys, never
rename or remove them, so older app binaries keep resolving every key they request.
A breaking structural change cuts a new `v<N+1>/`.

If a key is missing from a locale, the app falls back to bundled English, then to
the key name — so partial translations are safe to publish.

## Updating

Translations flow: English `sources/<module>/values/strings.xml` → Crowdin →
translated `sources/<module>/values-<locale>/strings.xml` (committed here) →
`build_string_catalog.py` regenerates `v1/`.

From the main PDFViewer repo (this repo checked out at `libs/falcon-pdf-strings/`):

```bash
# 1. Pull latest translations from Crowdin into libs/falcon-pdf-strings/sources/
crowdin download

# 2. Regenerate the catalog from sources/ into v1/
python3 scripts/localization/build_string_catalog.py \
  --out libs/falcon-pdf-strings/v1

# 3. Commit in the submodule, then move the floating v1-latest tag:
cd libs/falcon-pdf-strings
git add sources v1 && git commit -m "Update translations + catalog"
git push origin main
git tag -f v1-latest && git push -f origin v1-latest
```
