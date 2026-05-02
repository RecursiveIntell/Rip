# z.py

`z.py` is a standalone source/context archive certifier. It creates a ZIP file
plus audit sidecars that describe what was included, what was excluded, and
which validation checks passed or failed.

It is a single Python file and uses only the standard library.

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

## What It Writes

By default, a run writes:

- `<archive>.zip`
- `<archive>.manifest.json`
- `<archive>.report.md`
- `<archive>.excluded.json`
- `<archive>.findings.json`

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

- `0`: archive written or dry run completed
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

## Review Checklist

Before using an archive as a handoff artifact:

1. Read `<archive>.report.md`.
2. Confirm there are no unexpected validation findings.
3. Review `<archive>.excluded.json` for unexpected omissions.
4. Confirm any external path dependencies are intentional.
5. Keep the generated sidecars with the ZIP.
