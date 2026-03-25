---
name: proofshot-issue-resolution
description: Document GitHub issue resolution with visual before/after evidence using
  ProofShot. Use when closing a bug fix issue, visual regression, or any UI-related
  GitHub issue that benefits from screenshot proof of the fix. Captures the broken
  state, captures the fixed state after deployment, and posts structured evidence
  directly to the GitHub issue before closing it.
allowed-tools: Bash(proofshot:*), Bash(agent-browser:*), Bash(playwright:*), Bash(gh:*)
---

# ProofShot Issue Resolution — Visual Evidence for Bug Fixes

Close GitHub issues with proof, not promises. This skill captures before/after screenshots of a bug fix and posts them as structured evidence on the GitHub issue before closing it.

## When to use

Use this skill when:
- Fixing a UI bug tracked in a GitHub issue
- Resolving a visual regression
- Closing any issue where the fix has a visible, screenshot-able outcome
- The team or project values visual proof of resolution

Do NOT use when:
- The fix is purely backend with no visual component
- The issue is a feature request (use the base `proofshot` skill for PR proof instead)
- The deployed site is not accessible for screenshots

## Prerequisites

- `proofshot` CLI installed and on PATH
- `gh` CLI authenticated with repo access
- The bug must be reproducible on the deployed/running site before the fix
- The fix must be deployable (or already deployed) for the "after" screenshot

## The Workflow

### Phase 1: Capture the broken state (BEFORE fixing)

Before writing any code, document what is broken.

```bash
# Start a proofshot session against the live/deployed site
# No --run flag needed — we are screenshotting an already-running site
proofshot start --description "BEFORE fix: issue #ISSUE_NUMBER — brief description of bug"
```

Navigate to the broken page and capture evidence:

```bash
proofshot exec open https://your-site.example.com/affected-page
proofshot exec screenshot before-bug-state.png
```

If the bug requires interaction to reproduce (e.g., clicking a button, submitting a form):

```bash
proofshot exec snapshot -i                          # See interactive elements
proofshot exec click @e5                             # Trigger the bug
proofshot exec screenshot before-bug-triggered.png   # Capture the broken result
```

Stop the session and collect artifacts:

```bash
proofshot stop
```

Save the proof directory path — you will need the screenshot paths later.

```bash
# The screenshots land in the latest proof directory
BEFORE_DIR=$(ls -td /tmp/proofshot-*/screenshots 2>/dev/null | head -1)
```

### Phase 2: Fix the bug

Implement the fix using your normal development workflow. Commit and deploy.

### Phase 3: Capture the fixed state (AFTER deploying)

After the fix is deployed to the target environment:

```bash
proofshot start --description "AFTER fix: issue #ISSUE_NUMBER — verifying resolution"
```

Navigate to the same page and capture the fixed state:

```bash
proofshot exec open https://your-site.example.com/affected-page
proofshot exec screenshot after-fix-state.png
```

If the bug required interaction, repeat the same steps:

```bash
proofshot exec snapshot -i
proofshot exec click @e5                             # Same interaction as before
proofshot exec screenshot after-fix-verified.png     # Should show correct behavior
```

Stop and collect:

```bash
proofshot stop
```

```bash
AFTER_DIR=$(ls -td /tmp/proofshot-*/screenshots 2>/dev/null | head -1)
```

### Phase 4: Post evidence to the GitHub issue

Upload screenshots and post a structured comment on the issue.

First, upload the images to a location GitHub can render. The simplest method is to use the `gh` issue comment body with base64-embedded images or to reference images from the repo. The most reliable approach is to let GitHub handle the upload by attaching images via the API:

```bash
# Define variables
OWNER="org-name"
REPO="repo-name"
ISSUE_NUMBER=123

# Post the structured comment with before/after evidence
gh issue comment "$ISSUE_NUMBER" \
  --repo "$OWNER/$REPO" \
  --body "$(cat <<'EVIDENCE'
## Resolution Evidence

### Before (bug present)
<!-- Attach before screenshot below -->
![Before fix](BEFORE_SCREENSHOT_URL)

**Observed behavior:** Describe what was broken — e.g., "Image fails to load, returns 403 from S3."

### After (fix deployed)
<!-- Attach after screenshot below -->
![After fix](AFTER_SCREENSHOT_URL)

**Expected behavior confirmed:** Describe the fix — e.g., "Image loads correctly via signed URL proxy."

### Fix details
- **Branch:** `fix/branch-name`
- **Commit:** COMMIT_SHA
- **Deployed to:** staging / production
- **Verified on:** YYYY-MM-DD

---
*Evidence captured with [ProofShot](https://github.com/nicholasgriffintn/proofshot)*
EVIDENCE
)"
```

#### Uploading screenshots to GitHub

GitHub Markdown in issue comments supports image uploads. The easiest agent-friendly method:

**Option A — Repo assets (recommended for private repos):**
Commit screenshots to a `docs/evidence/` directory in the repo and reference them with raw GitHub URLs:

```bash
mkdir -p docs/evidence
cp "$BEFORE_DIR/before-bug-state.png" docs/evidence/issue-${ISSUE_NUMBER}-before.png
cp "$AFTER_DIR/after-fix-state.png" docs/evidence/issue-${ISSUE_NUMBER}-after.png
git add docs/evidence/
git commit -m "docs: add resolution evidence for issue #${ISSUE_NUMBER}"
git push
```

Then reference in the comment:
```
![Before](https://raw.githubusercontent.com/OWNER/REPO/BRANCH/docs/evidence/issue-123-before.png)
![After](https://raw.githubusercontent.com/OWNER/REPO/BRANCH/docs/evidence/issue-123-after.png)
```

**Option B — Inline base64 (small images only, <1 MB):**
```bash
BEFORE_B64=$(base64 -w0 "$BEFORE_DIR/before-bug-state.png")
# Use: ![Before](data:image/png;base64,${BEFORE_B64})
# Note: GitHub may strip data URIs in some contexts. Option A is more reliable.
```

### Phase 5: Close the issue with evidence

Only close after the evidence comment is posted:

```bash
gh issue close "$ISSUE_NUMBER" \
  --repo "$OWNER/$REPO" \
  --reason completed \
  --comment "Resolved with visual evidence above. Fix deployed and verified."
```

## Fallback: Playwright Direct (when ProofShot is unavailable)

If `proofshot` CLI or `agent-browser` is not installed, use Playwright directly:

```bash
# Install if needed
bunx playwright install chromium

# Take a screenshot
bunx playwright screenshot \
  --browser chromium \
  "https://your-site.example.com/affected-page" \
  before-bug-state.png

# After deploying the fix
bunx playwright screenshot \
  --browser chromium \
  "https://your-site.example.com/affected-page" \
  after-fix-state.png
```

Then post using the same `gh issue comment` workflow from Phase 4.

## Limitation: WebGL / GPU-Heavy Sites

Sites using WebGL, GPU-accelerated rendering (e.g., CesiumJS, Three.js, Mapbox GL), or canvas-heavy visualizations may render incorrectly or as blank in headless browsers. Headless Chromium lacks GPU acceleration by default.

**Workarounds:**
1. **Test against the deployed site, not localhost** — some hosted environments provide software rendering that works better than local headless.
2. **Use `--headed` mode** if the agent host has a display (e.g., via Xvfb):
   ```bash
   proofshot start --description "..." --headed
   ```
3. **Capture the non-WebGL parts** — screenshot the surrounding UI, error states, or network panel evidence instead of the 3D canvas itself.
4. **Document the limitation** — in the GitHub comment, note that WebGL content cannot be captured headlessly and reference manual verification or link to a screen recording.

## Complete Example

Resolving GitHub issue #87: "Dashboard map tiles fail to load after auth change."

```bash
# 1. Capture broken state on staging
proofshot start --description "BEFORE: issue #87 — map tiles 403 on staging dashboard"
proofshot exec open https://staging.example.com/dashboard
proofshot exec screenshot before-map-broken.png
proofshot stop
BEFORE_DIR=$(ls -td /tmp/proofshot-*/screenshots 2>/dev/null | head -1)

# 2. Implement fix, commit, deploy to staging
# ... (normal development workflow) ...

# 3. Capture fixed state
proofshot start --description "AFTER: issue #87 — map tiles loading after proxy fix"
proofshot exec open https://staging.example.com/dashboard
proofshot exec screenshot after-map-fixed.png
proofshot stop
AFTER_DIR=$(ls -td /tmp/proofshot-*/screenshots 2>/dev/null | head -1)

# 4. Commit evidence
mkdir -p docs/evidence
cp "$BEFORE_DIR/before-map-broken.png" docs/evidence/issue-87-before.png
cp "$AFTER_DIR/after-map-fixed.png" docs/evidence/issue-87-after.png
git add docs/evidence/
git commit -m "docs: add resolution evidence for issue #87"
git push

# 5. Post to GitHub issue
BRANCH=$(git branch --show-current)
COMMIT=$(git rev-parse --short HEAD)

gh issue comment 87 \
  --repo "myorg/myrepo" \
  --body "$(cat <<EOF
## Resolution Evidence

### Before (bug present)
![Before fix](https://raw.githubusercontent.com/myorg/myrepo/${BRANCH}/docs/evidence/issue-87-before.png)

**Observed behavior:** Map tiles return 403 after authentication token change. Dashboard shows blank map area with network errors in console.

### After (fix deployed)
![After fix](https://raw.githubusercontent.com/myorg/myrepo/${BRANCH}/docs/evidence/issue-87-after.png)

**Expected behavior confirmed:** Map tiles load correctly through the authenticated proxy. All tile layers render without errors.

### Fix details
- **Branch:** \`${BRANCH}\`
- **Commit:** ${COMMIT}
- **Deployed to:** staging
- **Verified on:** $(date +%Y-%m-%d)

---
*Evidence captured with [ProofShot](https://github.com/nicholasgriffintn/proofshot)*
EOF
)"

# 6. Close the issue
gh issue close 87 \
  --repo "myorg/myrepo" \
  --reason completed \
  --comment "Resolved with visual evidence above. Fix deployed and verified on staging."
```

## Checklist

Before closing an issue with this workflow, confirm:

- [ ] "Before" screenshot clearly shows the bug
- [ ] Fix is deployed to the target environment
- [ ] "After" screenshot shows the correct behavior
- [ ] Evidence comment is posted on the issue with both screenshots
- [ ] Fix details (branch, commit, environment) are documented
- [ ] Issue is closed with `--reason completed`
