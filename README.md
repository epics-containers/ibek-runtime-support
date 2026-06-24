# ibek-runtime-support

A central **library of runtime support patterns** for otherwise-generic EPICS IOCs.
Each pattern is a self-contained set of `ibek` support files (entity definitions, and
any EPICS DB/template they instantiate) that an IOC instance can **vendor at a pinned
version** using `ibek pattern`.

Vendoring copies a pattern's files into an IOC instance's `config/` directory and
records exactly which version was taken, with per-file integrity hashes. The IOC then
loads that extra support **at runtime**, with no image rebuild — a *generic* IOC image
gains additional entities/DB without a purpose-built container.

This repo is consumed by the `ibek pattern` subsystem in
[epics-containers/ibek](https://github.com/epics-containers/ibek). It is one of two
central pattern libraries; its sibling
[`ibek-runtime-streamdevice`](https://github.com/epics-containers/ibek-runtime-streamdevice)
holds StreamDevice device-support patterns (protocol + DB + support). This repo holds
the **non-StreamDevice** runtime additions — entity/DB bundles that layer onto a
generic IOC.

## What a "pattern" is — the file-set contract

A pattern is **a top-level folder named after the pattern**. The folder name is the
pattern name used on the command line.

A pattern holds an **arbitrary file-set**, not a fixed shape. The only required member
is the `ibek` support definition; everything else is whatever that definition
references:

| File | Role | Required |
|---|---|---|
| `*.ibek.support.yaml` | `ibek` support module: declares the `entity_models`, their parameters, and which DB/template files to instantiate | **yes** |
| `*.template` / `*.db` | EPICS database/template instantiated by the entity | optional |
| `*.req` | autosave request file(s) | optional |
| `*.pvi.device.yaml` | PVI device descriptor (UI/PV generation) | optional |
| anything else the support yaml references | … | optional |

There is **no hard-coded file shape**. The vendoring lock simply hashes a **file list**,
so it already generalises to any combination of the above (or files not yet invented).
Add the files your support yaml needs and they are vendored together as a unit.

This repo currently provides one pattern:

```
detectorPlugins/
  detectorPlugins.ibek.support.yaml   # entity_models "gdaPlugins" and "detectorPlugins"
```

`detectorPlugins` is the **standard AreaDetector plugin set** used DLS-wide. Its
support yaml declares two entity models, each a curated collection of AreaDetector
plugins wired together:

- **`gdaPlugins`** — the full plugin set for GDA (ROIs, stats, std-arrays, process,
  overlay, TIFF/HDF5 file writers, PVA, ffmpeg stream, …).
- **`detectorPlugins`** — the lighter plugin set for Ophyd (PVA, stats, ROI-stat,
  HDF5).

It is a **single `*.ibek.support.yaml`** with no proto/template/DB of its own — the
plugins it references are compiled into the AreaDetector IOC image; this pattern only
adds the `ibek` entities that instantiate and connect them at runtime.

## Pristine storage and the vendored header

**Files in this repo are stored PRISTINE.** They contain exactly the `ibek` support
content, with **no provenance header**.

When `ibek pattern` vendors a file into an IOC instance, it injects a deterministic
provenance header as the first line **before hashing**:

```
# Vendored from github.com/epics-containers/ibek-runtime-support@v0.1.0 — DO NOT EDIT
```

(note: em dash `—`). Because the header is added at vendor time and is part of the
content that is hashed into the instance's lock file, integrity checking on the
consumer side is a trivial `sha256(file_as_written) == lock`. The header is
deterministic (no timestamps or absolute paths) so it is reproducible.

**Do not commit this header here.** It is a consumer-side artifact. Patterns in this
repo must stay header-free and pristine; the header you see in a vendored copy belongs
to the IOC instance, not to this library.

## Versioning

Releases are cut as **repo-wide semantic-version git tags** (`v0.1.0`, `v0.2.0`, …). A
tag is an **immutable point** covering every pattern in the repo at that revision, so
`name@<tag>` always resolves to the same bytes.

There is intentionally **no per-pattern version**: a single tag versions the whole
library. Bump the tag when any pattern changes; consumers opt in to the new content by
moving their pin.

On the consumer side, [Renovate](https://docs.renovatebot.com/) tracks the pin using
the **`github-releases`** datasource and raises a PR to bump the pinned version when a
new tag is published, keeping IOC instances current without manual edits.

## Consumer pin / update workflow

All consumer commands run against an **IOC instance directory** (`services/<instance>/`
in a services repo). `ibek pattern` writes the vendored files into the instance's
`config/`, and writes `runtime-lock.yaml` + `ioc.schema.json` at the **instance root**
(not in `config/`, which is the ≤ 1 MiB K8s ConfigMap and holds runtime inputs only).

### Add a pattern (pin a version)

The qualified name `<library>:<pattern>@<tag>` selects the library, pattern and version:

```bash
ibek pattern add ibek-runtime-support:detectorPlugins@v0.1.0 services/bl01t-di-cam-01
```

This:

1. fetches the pattern's file-set at tag `v0.1.0`,
2. writes each file into `services/bl01t-di-cam-01/config/` with the vendored header,
3. records `version`, `source` and a per-file `sha256` in
   `services/bl01t-di-cam-01/runtime-lock.yaml`,
4. merges the pattern's entity models into the instance's
   `services/bl01t-di-cam-01/ioc.schema.json` (the image's published schema with the
   vendored entities grafted in) and points `config/ioc.yaml`'s first line at it
   (`# yaml-language-server: $schema=../ioc.schema.json`).

You then reference the entity in `config/ioc.yaml` (the entity type is
`<module>.<entity_model>`):

```yaml
entities:
  - type: detectorPlugins.gdaPlugins
    P: BL01T-DI-CAM-01
    CAM: DET.DET
    PORTPREFIX: D
```

The resulting `runtime-lock.yaml` looks like:

```yaml
detectorPlugins:
  version: v0.1.0
  source: github.com/epics-containers/ibek-runtime-support
  files:
    detectorPlugins.ibek.support.yaml: "sha256:…"
```

(a hash value may be `"DIRTY # <reason>"` for a deliberately locally-modified file).

### Update a pin

```bash
ibek pattern update detectorPlugins services/bl01t-di-cam-01 -v v0.2.0
# or update every pinned pattern to its latest resolvable version:
ibek pattern update services/bl01t-di-cam-01
```

### Check integrity (CI / pre-commit)

```bash
ibek pattern check services/bl01t-di-cam-01
```

Re-hashes each vendored file and fails if it drifts from `runtime-lock.yaml`. Allow
intentional local edits with `--allow-dirty` (or `IBEK_ALLOW_DIRTY=1`); such files must
be marked `DIRTY` in the lock.

### Restore vendored files

```bash
ibek pattern restore detectorPlugins services/bl01t-di-cam-01   # one pattern
ibek pattern restore services/bl01t-di-cam-01                   # all patterns
```

Rewrites the vendored files from the locked version, discarding local edits.

### Inspect the merged schema

```bash
ibek pattern schema services/bl01t-di-cam-01
```

Regenerates the instance's self-contained `ioc.schema.json` (the image's published base
schema with the vendored/local entities merged in). If the image has not published a
schema, this is reported and skipped — it is not an error.

## Local beamline support

Beamline-local support that is **not** shared across instruments (e.g. a one-off
`<beamline>-support` module) does **not** belong in this library. Drop its
`*.ibek.support.yaml` straight into the consuming instance's `config/`: it is **not**
recorded in `runtime-lock.yaml` (it is editable, not vendored) but **is** auto-merged
into that instance's `ioc.schema.json` alongside the vendored patterns.

## Adding a new pattern

1. Create a top-level folder named after the pattern, e.g. `mydetector/`.
2. Drop in the file-set:
   - `mydetector.ibek.support.yaml` — the `ibek` support module (module name, one or
     more `entity_models` with their parameters and `sub_entities`/`databases`),
   - any `*.template`/`*.db`/`*.req` files that support yaml references.
3. Keep every file **pristine** — no vendored header (that is added by `ibek pattern`
   at vendor time).
4. Validate locally by vendoring into a scratch IOC instance:
   `ibek pattern add ibek-runtime-support:mydetector@HEAD <instance>` (or test against a
   branch) and run `ibek pattern check`.
5. Open a PR. Once merged, **cut a new semver tag** (`vX.Y.Z`) so consumers can pin the
   new pattern; Renovate will then offer the bump downstream.

## License

[Apache License 2.0](LICENSE).
