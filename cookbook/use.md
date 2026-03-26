# Use a Skill from the Library

## Context
Pull a skill, agent, or prompt from the catalog into the local environment. If already installed locally, overwrite with the latest from the source (refresh).

## Input
The user provides a skill name or description.

## Steps

### 1. Sync the Library Repo
Pull the latest catalog before reading:
```bash
cd <LIBRARY_SKILL_DIR>
git pull
```

### 2. Find the Entry
- Read `library.yaml`
- Search across `library.skills`, `library.agents`, and `library.prompts`
- Match by name (exact) or description (fuzzy/keyword match)
- If multiple matches, show them and ask the user to pick one
- If no match, tell the user and suggest `/library search`

### 3. Trust Check
Before fetching, verify the source is trusted:

1. Parse the `source` URL to extract the GitHub org (e.g., `Sanmarcsoft` from `https://github.com/Sanmarcsoft/repo/...`)
2. Check if the org is in `trusted_orgs` from `library.yaml`
3. Also check the `trusted` field on the entry itself

**If trusted (`trusted: true` AND org in `trusted_orgs`):** Proceed normally.

**If untrusted:** BLOCK and warn:
```
WARNING: This item's source is from an untrusted organization: <org>

Trusted organizations: Sanmarcsoft, smsmatt, disler

To proceed, either:
1. Fork the source repo into the Sanmarcsoft org and update the source URL
2. Add the org to trusted_orgs in library.yaml

Do NOT install from untrusted sources without explicit user approval.
```

If the user explicitly overrides ("install it anyway"), proceed but mark the entry with `trusted: false` in library.yaml as a permanent warning.

### 4. Resolve Dependencies
If the entry has a `requires` field:
- For each typed reference (`skill:name`, `agent:name`, `prompt:name`):
  - Look it up in `library.yaml`
  - If found, recursively run the `use` workflow for that dependency first
  - If not found, warn the user: "Dependency <ref> not found in library catalog"
- Process all dependencies before the requested item

### 5. Determine Target Directory
- Read `default_dirs` from `library.yaml`
- If user said "global" or "globally" → use the `global` path
- If user specified a custom path → use that path
- Otherwise → use the `default` path
- Select the correct section based on type (skills/agents/prompts)

### 6. Fetch from Source (Local-First Resolution)

**Resolution order:** Always try `local` path first, then fall back to `source` URL.

#### Step 6a: Check Local Path
If the entry has a `local` field:
- Check if the file exists at the `local` path
- If it exists: use it as the source (fast, no network needed)
- If it doesn't exist: fall back to Step 6b

**If source is a local path** (starts with `/` or `~`) and no `local` field:
- Resolve `~` to the home directory
- Check if the file exists
- If it exists: use it
- If it doesn't exist: check if `source` is a GitHub URL that can be used instead

#### Step 6b: Fetch from GitHub (Fallback)
**If source is a GitHub URL**:
- Parse the URL to extract: `org`, `repo`, `branch`, `file_path`
  - Browser URL pattern: `https://github.com/<org>/<repo>/blob/<branch>/<path>`
  - Raw URL pattern: `https://raw.githubusercontent.com/<org>/<repo>/<branch>/<path>`
- Determine the clone URL: `https://github.com/<org>/<repo>.git`
- Determine the parent directory path within the repo (everything before the filename)

**First, check if the repo is already cloned locally:**
- Read `local_workspace` from `library.yaml` (default: `/config/workspace/projects/sanmarcsoft`)
- Check if `<local_workspace>/<repo>/` exists
- If it does: use the local clone instead of fetching from GitHub
  - Also offer to set the `local` field: "Found local clone at `<path>`. Want me to add it as the `local` source for faster future access?"

**If no local clone exists, offer to clone:**
- For trusted Sanmarcsoft org repos: "This repo isn't cloned locally. Want me to clone it to `<local_workspace>/<repo>/`? This enables local-first access for future installs."
- If user agrees: clone the full repo (not shallow — it's a workspace copy)
  ```bash
  git clone https://github.com/<org>/<repo>.git <local_workspace>/<repo>/
  ```
  Then use the local clone for this install.

**If user declines or for non-local installs**, do a temporary shallow clone:
```bash
tmp_dir=$(mktemp -d)
git clone --depth 1 --branch <branch> https://github.com/<org>/<repo>.git "$tmp_dir"
```

#### Step 6c: Copy to Target
- Get the parent directory of the referenced file
- For skills: copy the entire parent directory to the target:
  ```bash
  cp -R <parent_directory>/ <target_directory>/<name>/
  ```
- For agents: copy just the agent file to the target:
  ```bash
  cp <agent_file> <target_directory>/<agent_name>.md
  ```
- For prompts: copy just the prompt file to the target:
  ```bash
  cp <prompt_file> <target_directory>/<prompt_name>.md
  ```

**If clone fails (private repo)**, try SSH:
```bash
git clone --depth 1 --branch <branch> git@github.com:<org>/<repo>.git "$tmp_dir"
```

Clean up temporary directories:
```bash
rm -rf "$tmp_dir"
```

### 7. Verify Installation
- Confirm the target directory exists
- Confirm the main file (SKILL.md, AGENT.md, or prompt file) exists in it
- Report success with the installed path

### 8. Confirm
Tell the user:
- What was installed and where
- Any dependencies that were also installed
- If this was a refresh (overwrite), mention that
- Trust status of the source
- Whether local or GitHub source was used
