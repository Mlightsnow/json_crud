# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development commands

```bash
# Run all tests
python3 -m unittest tests.test_roundtrip -v

# Run a single test class
python3 -m unittest tests.test_roundtrip.PathParserTests -v

# Run a single test
python3 -m unittest tests.test_roundtrip.CrudTests.test_set_string -v
```

No external dependencies — uses Python 3.8+ stdlib only.

## Architecture

Five scripts share a common foundation in `scripts/_common.py`:

```
_common.py          # Core utilities (path parsing, traversal, atomic I/O)
    ├── inspect.py  # Structure summary renderer
    ├── get.py      # Read value at path
    ├── set.py      # Create/update value at path
    └── delete.py   # Remove key/element at path
```

**Path parsing** (`parse_path`, `format_path`): Converts between path expressions like `users[0].name` and segment lists like `['users', 0, 'name']`. Supports dot notation, bracket indexing (including `-1` for last element), and quoted keys for keys containing dots: `["weird.key"]`.

**Traversal** (`get_value`, `walk_to_parent`): Walk nested dicts/lists following a path. `walk_to_parent` stops one level before the end and optionally creates missing intermediate containers (used by set/delete).

**Atomic I/O** (`dump_json`): Writes go to a temp file then `os.replace()` — failed writes cannot corrupt the original file.

**Key invariants**:
- All writes use `indent=2` formatting. Original spacing of rewritten subtrees is not preserved.
- `parse_path` and `format_path` are inverses for valid paths.
- Array indices are normalized: `-1` maps to `len-1`, out-of-range raises `IndexError`.
