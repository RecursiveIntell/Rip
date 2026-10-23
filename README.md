# z.py

<!-- last-verified: 2026-10-23 -->

`z.py` is a standalone source/context archive certifier. It creates a ZIP file
plus audit sidecars that describe what was included, what was excluded, and
which validation checks passed or failed.

It is a single Python file and uses only the standard library. A successful archive check covers the enabled archive rules; it does not certify that the packaged software works or that every secret was detected.

Use Python 3.11+ to enable the standard-library TOML parser used for Cargo metadata checks. Older-Python fallback parsing is less complete.

## Quick Start

Run it from the directory that contains `z.py`:

```bash
python3 z.py --root /path/to/project
```

Create a package with an explicit output path:

```bash
python3 z.py --root /path/to/project --output project-context.zip
```

Run validation and write reports without creating the ZIP:

```bash
python3 z.py --root /path/to/project --dry-run
```

Show the full CLI:

```bash
python3 z.py --help
```

## Source-tree effects and scope

Normal runs enable `--archive-codex-runs` by default. Before packaging, the script can move stale Codex-run files into `docs/codex-runs/archive`, remove active duplicates already preserved there, and write supersession/index records. Run on a clean working copy or inspect the planned changes with `--dry-run` first. Dry runs skip those moves but still write report sidecars.

Use `--no-archive-codex-runs` when you want to disable that normalization. External Cargo path dependencies are included by default when detected; inspect `include_roots` and use `--no-include-external-path-deps` when the intended boundary is only `--root`. The selected profile/mode determines which project surfaces are relevant. Archive inclusion is not permission to share files with someone else.

Relative `--output` paths are resolved under `--root`, not necessarily the shell's current directory.

## What It Writes

By default, a run writes:

- `<archive>.zip`
- `<archive>.manifest.json`
- `<archive>.report.md`
- `<archive>.excluded.json`
- `<archive>.findings.json`
- a Codex archive-normalization report when that workflow is enabled

The Markdown report is the human review surface. The JSON files are for
automation and audit trails.

## Strict Mode

Strict mode is enabled by default. If validation errors are found, `z.py` exits
with code `2` and does not write the ZIP. It still writes the sidecar reports so
the failure can be inspected.

Use `--no-strict` for a diagnostic package when you knowingly want output even
with validation errors:

```bash
python3 z.py --root /path/to/project --no-strict
```

Exit codes:

- `0`: requested operation completed (including dry-run or archive-only modes; inspect the report for what was actually written)
- `2`: validation failed in strict mode
- `1`: unexpected operational failure

## Common Options

- `--profile <name>`: choose the project profile, or use `auto`
- `--mode <name>`: choose the archive policy mode
- `--include-external-path-deps`: include Cargo path dependencies outside root
- `--no-include-external-path-deps`: keep the package under `--root`
- `--include-doc-binaries` / `--exclude-doc-binaries`
- `--include-images` / `--exclude-images`
- `--include-logs` / `--exclude-logs`
- `--follow-symlinks`: follow symlinks whose targets stay inside root
- `--max-file-size-mb <number>`: cap included file size
- `--compresslevel <0-9>`: set ZIP compression level
- `--preserve-mtime`: preserve file mtimes instead of deterministic timestamps

## Built-In Checks

When enabled, `z.py` checks for:

- missing expected project surfaces for the selected profile
- Rust `include_str!` and `include_bytes!` references
- Cargo path dependency self-containment
- conservative shell script references to `.sh` and `.py` files
- high-risk secret-like content in included text files
- stale active Codex-run artifacts when archive hygiene checks are requested

Use the matching `--no-check-*` flags only for diagnostic runs.

## Review the generated archive

Start with a dry run and keep its reports outside the source tree:

```bash
python3 z.py --root /path/to/project --output /tmp/project-context.zip --dry-run
```

In the disposable-fixture preview, the command printed `dry-run: zip not written` and reported the Markdown sidecar. For the `project-context.zip` output shown above, the default report and excluded-file paths are `/tmp/project-context.report.md` and `/tmp/project-context.excluded.json`, unless their corresponding `--report-out` or `--excluded-out` options override them. Read the report and inspect the excluded-file JSON before producing or sharing an archive. The dry run does not create the ZIP, but it does write sidecar reports beside the requested output path. Those reports describe the proposed package; they do not prove the packaged project works or that every secret was detected.

The complete set of CLI flags and their parser defaults is intentionally kept in `python3 z.py --help` rather than duplicated here; this README highlights the common options and the source-tree and inclusion defaults that materially affect package scope.

## Review Checklist

Before using an archive as a handoff artifact:

1. Read `<archive>.report.md`.
2. Confirm there are no unexpected validation findings.
3. Review `<archive>.excluded.json` for unexpected omissions.
4. Confirm any external path dependencies are intentional.
5. Keep the generated sidecars with the ZIP.
