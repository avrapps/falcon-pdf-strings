# falcon-pdf-strings

Runtime localization catalog for **Falcon PDF**. The app downloads these JSON
files on demand (via the jsDelivr CDN) to localize its UI in languages that are
not bundled in the binary.

This repo is **generated** — do not hand-edit the JSON. It is produced from the
app's Compose Resources `strings.xml` by
`scripts/localization/build_string_catalog.py` in the main PDFViewer repo.

## Layout

```
v1/manifest.json      locale index: per-locale revision, byte size, sha256, coverage
v1/en-rUS.json        English source (100% coverage)
v1/<tag>.json         one file per locale (Android -r tag form)
```

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

In the main PDFViewer repo:

```bash
python3 scripts/localization/build_string_catalog.py --out /tmp/catalog/v1
# copy /tmp/catalog/v1 into this repo's v1/, commit, then move the v1-latest tag:
git tag -f v1-latest && git push -f origin v1-latest
```
