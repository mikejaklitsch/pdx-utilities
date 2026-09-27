# pdx-utilities

Shared utilities for PDX modding tools (pdx-nav, pdx-gui, pdx-format,
pdx-lint, pdx-audit, pdx-maint).

This directory is the only copy. Tools import it as an installed package;
nothing is vendored into tool repos, so a change here reaches every tool at
once and the copies cannot drift.

## Layout

```
pdx-utilities/
  pyproject.toml
  pdx_utilities/     # the importable package
```

## Using it from a tool

Tools are pipx installs. Inject this package into the tool's venv as an
editable install, so the tool always runs the current source:

```bash
pipx install -e ./pdx-nav
pipx inject pdx-nav -e ./pdx-utilities
```

pipx records the injection, so `pipx reinstall pdx-nav` keeps it. To link
every tool that uses it:

```bash
for t in pdx-audit pdx-format pdx-gui pdx-lint pdx-maint pdx-nav; do
    pipx inject "$t" -e ./pdx-utilities
done
```

Each tool's `pyproject.toml` depends on this repo by URL
(`pdx-utilities @ git+https://github.com/mikejaklitsch/pdx-utilities.git`),
never by bare name: it is not on PyPI, and a bare name would resolve there.
A fresh `pipx install` of a tool therefore pulls `master` from GitHub; the
inject step swaps in your local checkout. Push here before relying on a
change in fresh installs.

Tools' run-from-clone wrapper scripts (`./pdx-audit`, `./pdx-gui`, ...) add a
sibling `../pdx-utilities` checkout to `sys.path`, so they work without any
install.

## Using it anywhere else

It is an ordinary package, so any environment can use it:

```bash
pip install -e /path/to/pdx-utilities
```

```python
from pdx_utilities.scanner import split_line
```

## Changing it

Edit here. Every editable install sees the change on the next run. When a
signature changes, grep the tools for callers (`grep -rn "pdx_utilities"
../pdx-*`) and run their tests.

## Modules

| Module | Contents |
|--------|----------|
| `scanner.py` | `split_line` (comment/string-aware line parser), `structural_balance` |
| `script_parser.py` | `tokenize`, `parse` (script and GUI text to a node tree; strict by default, `strict=False` skips stray braces, `positions=True` adds character offsets), `RAW_BLOCKS` |
| `paths.py` | `find_mod_root`, `find_mod_root_or_exit`, `vanilla_root`, `find_vanilla_repo`; `native_form`, `canonical_path`, `path_key` (one spelling per folder across WSL and Windows, casefolded on case-insensitive volumes) |
| `fileio.py` | BOM-aware I/O: `read_text_preserve`, `read_with_bom`, `write_text_exact`, `write_with_bom`, `strip_bom_bytes` |
| `constants.py` | `SCAN_TOPDIRS`, `CODE_EXTS`, `EXCLUDE_PARTS` |
| `files.py` | `collect_files` (enumerate script files under a game/mod root) |
| `git.py` | Bare-repo helpers: `git`, `git_tags`, `git_latest_tag`, `git_archive`, `git_extract_files`, `git_read_file` |
| `terminal.py` | `color_enabled` (NO_COLOR/TERM=dumb aware), ANSI constants |
