# CLAUDE.md — kaster

Shared concepts live in the [root CLAUDE.md](../../CLAUDE.md) and [docs/vocabulary.md](../../docs/vocabulary.md). This file only covers what is specific to kaster.

## 1. Role

Kaster is the runik trinket that renders the contents of a spell's `glyphs:` block as a **separate** ArgoCD source, in parallel with the spell's primary source. It is a thin dispatch chart — its own `templates/` contains a single 14-line file; the real work lives in the bundled glyph subcharts.

## 2. How it is registered

Book (or chapter) `index.yaml`:

```yaml
trinkets:
  kaster:
    key: glyphs
    repository: ...
    path: .
    revision: ...
```

The trinket name (`kaster`) is cosmetic. Only `key: glyphs` is meaningful — librarian routes any spell that has a `glyphs:` key through whichever trinket was registered under that key.

## 3. The dispatcher

- **Entry**: `charts/kaster/templates/kaster.yaml` (14 lines).
- **Gate**: runs only when `$root.Values.glyphs` is present.
- **Loop**: for each bundled subchart name, iterate `$root.Values.glyphs.<subchart>` entries; for each entry, merge the entry name in as `.name` and include `<subchart>.<type>` passing `(list $root $glyphWithName)`.
- Same calling convention as summon's internal dispatcher, so one glyph template works from both dispatchers.
- Fails with a Helm error if an entry's `.type` is missing or points to a non-existent template; it does not silently skip the way summon's dispatcher does.

## 4. What kaster receives

Librarian passes only the `glyphs:` block plus injected context (`spellbook`, `chapter`, `lexicon`). See [librarian/CLAUDE.md §5.8](../../librarian/CLAUDE.md). `charts/kaster/values.yaml` ships empty defaults; kaster does not merge defaults of its own.

## 5. Bundled subcharts

`charts/kaster/charts/` is a git submodule of `glyphs.git`; contents are identical to `charts/summon/charts/`.

- **Dispatchable (14)**: `argo-events` · `aws` · `certManager` · `crossplane` · `external-secrets` · `freeForm` · `gcp` · `istio` · `keycloak` · `pinniped` · `postgresql` · `s3` · `vault` · `workflow`
- **Helpers (not dispatched)**: `common` · `runic-system` · `summon`

Edit rule: edit only in the canonical path `charts/glyphs/`; then bump the submodule reference in `charts/kaster/`.

## 6. Testing

Kaster is the chart through which glyphs are rendered in tests. The Make targets pass kaster as the Helm chart and a glyph example file as values:

```
make render   glyph <name> [file]
make snapshot glyph <name> [file]
make test     glyph <name> [file]
```

`charts/kaster/examples/` holds cross-subchart smoke tests (`argo-events-test.yaml`, `summon-serviceaccount-test.yaml`). Per-glyph examples live under each subchart's own `examples/` directory.

## 7. Related docs

- [Root CLAUDE.md](../../CLAUDE.md) — two dispatchers, source table.
- [docs/vocabulary.md](../../docs/vocabulary.md) — glossary.
- [librarian/CLAUDE.md](../../librarian/CLAUDE.md) — what reaches kaster.
- [charts/summon/CLAUDE.md](../summon/CLAUDE.md) — sister dispatcher.
- [charts/glyphs/CLAUDE.md](../glyphs/CLAUDE.md) — authoring glyphs (to be written).
- [docs/design/kaster.md](../../docs/design/kaster.md) — code walkthrough.
- [docs/usage/glyphs.md](../../docs/usage/glyphs.md) — spell-author guide to the `glyphs:` key.
