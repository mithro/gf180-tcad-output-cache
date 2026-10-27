# gf180-tcad output cache

A shared cache of large generated files of [mithro/gf180-tcad](https://github.com/mithro/gf180-tcad)
that are kept out of that repository. This README is written by `tools/output-cache init` there;
do not edit it here.

## The contract

- **Everything here is regenerable** from https://github.com/mithro/gf180-tcad: each file is a result of a job that
  repository describes and can rerun. Nothing here is original data.
- **This repository may be wiped and recreated at any time** (a new root commit, force-pushed).
  Clone it with `git clone --depth 1`, and **never depend on its history**: only the current
  tree of `main` matters.
- **No provenance lives here.** The primary repository's `results/MANIFEST.sha256` and its
  `provenance.json` and `*.meta.json` files are the truth. This cache only holds bytes whose
  sha256 they name; a fetched file is used only if its sha256 matches (`tools/output-cache pull`
  checks and refuses a mismatch). `index.json` records where a file came from as a convenience.
- No git-LFS: every object is a plain git blob under 95 MB.

## Layout

- `objects/sha256/<first 2 hex>/<sha256>.<ext>[.gz]`: one file per sha256 of the
  **uncompressed** content. `.gz` objects are gzip-compressed deterministically (no file name,
  mtime 0) and must be decompressed before the hash is checked.
- `index.json`: sorted JSON, `objects` maps each sha256 to its `object` path, `ext`,
  `compression`, `size`, `compressed_size`, the results `paths` in the primary repository, the
  `generator_sha256` of its results tree, the `primary_commit` it was pushed from, the `date` and
  the `job`.

Cacheable files:

- `tcad/**/mesh.msh` (gmsh MSH 2.2 mesh of a 2D MOS-capacitor job (docs/limitations/moscap2d.md)): named by `mesh_sha256` in the `cv.meta.json` beside it; stored as `.msh.gz`.

## Use (from a checkout of the primary repository)

```sh
tools/output-cache status                        # which named files the cache has
tools/output-cache pull results/<job dir>/mesh.msh   # fetch one, verified, into place
tools/output-cache pull <sha256> --out FILE
```

Add files after a run (needs write access; `--dry-run` first):

```sh
git clone --depth 1 git@github.com:mithro/gf180-tcad-output-cache.git tmp/output-cache
tools/output-cache push --cache tmp/output-cache --results results --publish
```

First population, while this repository is still empty (a clone of it has no `index.json`):

```sh
tools/output-cache init tmp/output-cache --remote git@github.com:mithro/gf180-tcad-output-cache.git
tools/output-cache push --cache tmp/output-cache --results results --publish
```

Recreate from nothing (every listed file must be present locally):

```sh
tools/output-cache init tmp/output-cache-new
tools/output-cache push --cache tmp/output-cache-new --results results
git -C tmp/output-cache-new push --force origin main
```

## Repository settings

The owner's standard settings, except where the contract needs otherwise:

- Wiki, projects and discussions disabled; merge commits only (no squash or rebase merges);
  delete head branches on merge; secret scanning and push protection enabled.
- **Force pushes to `main` are allowed**, unlike the standard branch protection: a wipe is a
  force push of a new root commit. Deletion of `main` stays disallowed.
- No `v0.0` tag and no tag ruleset: tags would pin history that may be discarded.
- No git-LFS (bandwidth quotas); objects stay under the size cap instead.

Repository: https://github.com/mithro/gf180-tcad-output-cache (`mithro/gf180-tcad-output-cache`).
