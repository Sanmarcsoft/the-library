# Doctor — Validate Library Health

## Context
Run a health check on the library catalog. Validates structure, dependencies, and source URLs.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before validating:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Validate YAML Structure
- Parse `library.yaml` — if YAML is invalid, report the error and stop
- Verify top-level keys: `default_dirs` and `library`
- Verify `library` has sections: `skills`, `agents`, `prompts` (each can be empty but must exist)
- Verify each entry has required fields: `name`, `description`, `source`
- Verify optional fields are valid: `requires` (array of typed refs), `tags` (array of strings)

### 3. Validate Dependencies
For each entry with a `requires` field:
- Parse typed references (`skill:name`, `agent:name`, `prompt:name`)
- Verify each referenced item exists in the catalog under the correct section
- Check for circular dependencies:
  - Build a dependency graph
  - Walk the graph from each node
  - If any node is visited twice in a single walk, report the cycle
- Report: missing dependencies, circular dependencies

### 4. Validate Sources
For each entry:
- **Local paths** (`/` or `~`): Check if the file exists (warn if not, don't fail — could be a different machine)
- **GitHub browser URLs**: Verify format matches `https://github.com/<org>/<repo>/blob/<branch>/<path>`
- **GitHub raw URLs**: Verify format matches `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
- **Other URLs**: Warn as unsupported format

### 5. Validate Triggers (if triggers.yaml exists)
- Parse `triggers.yaml` — if invalid, report error
- Verify each intent has: `name`, `patterns` (non-empty array), `items` (non-empty array), `confidence`
- Verify all items in triggers reference valid catalog entries
- Verify confidence values are: `high`, `medium`, or `low`
- Check for duplicate intent names
- Check for overlapping patterns across intents (warn, don't fail)

### 6. Check Tags Coverage
- Report entries missing `tags` field
- Report entries with empty `tags` array
- Suggest tags for untagged entries based on their `description` and `name`

### 7. Display Results

```
## Library Health Report

### Structure: PASS/FAIL
- [details]

### Dependencies: PASS/WARN/FAIL
- [details]

### Sources: PASS/WARN
- [details]

### Triggers: PASS/WARN/FAIL (or SKIP if no triggers.yaml)
- [details]

### Tags Coverage: X/Y entries tagged
- [untagged entries listed]

### Summary
- Total entries: X
- Issues found: Y (Z critical)
- Warnings: W
```

### 8. Auto-Fix Option
If non-critical issues are found (missing tags, formatting):
- Ask: "Want me to auto-fix the non-critical issues?"
- If yes: apply fixes, commit with `library: doctor auto-fix — [summary]`, push
