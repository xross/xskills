# xskills

Personal OpenCode skills repository synced via `@jgordijn/opencode-remote-config`.

## Skills

### fix-repo-checks

Automatically fixes `test_repo_checks` failures in XMOS library repositories.

**Triggers:**
- "fix repo checks"
- "repo_checks failed"
- "test_repo_checks"
- "infr_apps checks"

**What it does:**
1. Runs `pytest test_repo_checks.py` to identify failures
2. Sets up infr_apps environment (clones if needed, creates venv)
3. Auto-fixes checks with update scripts:
   - `xmos_source_check` - Copyright headers
   - `xmos_license_check` - LICENSE.rst file
   - `xmos_changelog_check` - CHANGELOG.rst formatting
4. AI-fixes checks without update scripts:
   - `xmos_readme_check` - README.rst structure
   - `xmos_settings_check` - settings.yml schema
   - `xmos_versions_check` - Version consistency
   - `xmos_metadata_check` - Metadata in README.rst
   - `xmos_doc_check` - Documentation structure
   - `xmos_deps_check` - Dependency versions
5. Verifies all fixes and reports results (all changes left uncommitted)

**Safety features:**
- Validates `xmos_public_v1` license appropriateness
- Never commits changes automatically
- Uses repo-specific venv (`.venv_<repo_name>`)

---

## Setup

### Prerequisites

- **OpenCode** v1.0.0 or later
- **Bun** runtime (comes with OpenCode)
- **Git** installed and configured
- **npm** or **bun** for installing plugins

### 1. Install opencode-remote-config Plugin

**Option A - npm (Recommended):**
```bash
npm install -g @jgordijn/opencode-remote-config
```

**Option B - bun:**
```bash
bun add -g @jgordijn/opencode-remote-config
```

### 2. Register the Plugin

Add the plugin to your OpenCode configuration.

**Global config** (`~/.config/opencode/opencode.json`):
```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@jgordijn/opencode-remote-config"]
}
```

**Or project config** (`.opencode/opencode.json`):
```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@jgordijn/opencode-remote-config"]
}
```

### 3. Create remote-config.json

**Global config** (`~/.config/opencode/remote-config.json`):
```json
{
  "$schema": "https://raw.githubusercontent.com/jgordijn/opencode-remote-config/main/remote-skills.schema.json",
  "repositories": [
    {
      "url": "https://github.com/xross/xskills.git",
      "ref": "main"
    }
  ]
}
```

**Or project config** (`.opencode/remote-config.json`):
```json
{
  "$schema": "https://raw.githubusercontent.com/jgordijn/opencode-remote-config/main/remote-skills.schema.json",
  "repositories": [
    {
      "url": "https://github.com/xross/xskills.git",
      "ref": "main"
    }
  ]
}
```

### 4. Restart OpenCode

The plugin syncs on startup. Quit OpenCode and restart it.

**Verify installation:**
```bash
# In OpenCode, the skill should now be available
# You can test by typing: "use the fix-repo-checks skill"
```

---

## Configuration Options

### Selective Import

Import only specific skills:
```json
{
  "repositories": [
    {
      "url": "https://github.com/xross/xskills.git",
      "ref": "main",
      "skills": {
        "include": ["fix-repo-checks"]
      }
    }
  ]
}
```

### Pin to Specific Version

Pin to a tag or commit:
```json
{
  "repositories": [
    {
      "url": "https://github.com/xross/xskills.git",
      "ref": "v1.0.0"
    }
  ]
}
```

### Local Development

For testing changes before pushing to GitHub:
```json
{
  "repositories": [
    {
      "url": "file:///Users/ross/sandboxes/libs/xskills"
    }
  ]
}
```

**Note:** Restart OpenCode after changing `remote-config.json`.

---

## Usage

Once set up, OpenCode will automatically load skills from this repository.

### Using fix-repo-checks

From any XMOS library repository (e.g., `lib_sw_pll`):

```
You: fix repo checks

OpenCode will:
1. Detect the fix-repo-checks skill is relevant
2. Run test_repo_checks.py
3. Fix all failures
4. Report results
```

Or be more explicit:
```
You: use the fix-repo-checks skill to fix test_repo_checks failures
```

---

## Development

### Adding New Skills

1. Create a new directory under `skill/`:
   ```bash
   mkdir -p skill/my-new-skill
   ```

2. Create `SKILL.md` with frontmatter:
   ```markdown
   ---
   name: my-new-skill
   description: Use when... (describe when to trigger and what it does)
   ---
   
   # My New Skill
   
   (skill instructions here)
   ```

3. Commit and push:
   ```bash
   git add skill/my-new-skill/SKILL.md
   git commit -m "Add my-new-skill"
   git push
   ```

4. Restart OpenCode to sync changes

### Testing Skills Locally

Use `file://` URL in `remote-config.json` to test before pushing:

```json
{
  "repositories": [
    {
      "url": "file:///Users/ross/sandboxes/libs/xskills"
    }
  ]
}
```

This allows rapid iteration without committing/pushing.

---

## Repository Structure

```
xskills/
├── README.md                      # This file
└── skill/                         # Skills directory
    └── fix-repo-checks/
        └── SKILL.md               # Skill definition
```

**Why this structure?**
- `@jgordijn/opencode-remote-config` scans for `skill/*/SKILL.md`
- Each skill gets its own directory
- Can add agents, commands, plugins later using same pattern:
  - `agent/*.md` - Custom agents
  - `command/*.ts` - Slash commands
  - `plugin/*.ts` - OpenCode plugins

---

## How It Works

### Sync Process

1. On OpenCode startup, `@jgordijn/opencode-remote-config` reads `remote-config.json`
2. Clones (or fetches updates for) each repository to `~/.cache/opencode/remote-config/repos/`
3. Scans for skills in `skill/*/SKILL.md`
4. Symlinks skills to `~/.config/opencode/skill/_plugins/xskills/`
5. OpenCode loads all skills including remote ones

### Priority

If multiple repositories define the same skill name:
1. Local config (highest priority)
2. First repository in `remote-config.json`
3. Subsequent repositories (skipped, warning logged)

### Updates

Skills are synced on every OpenCode startup. To get latest changes:
1. Push updates to GitHub
2. Restart OpenCode

---

## Troubleshooting

### Plugin Not Loading

**Check OpenCode config:**
```bash
cat ~/.config/opencode/opencode.json
# Should contain: "plugin": ["@jgordijn/opencode-remote-config"]
```

**Check plugin installation:**
```bash
npm list -g @jgordijn/opencode-remote-config
```

### Skills Not Syncing

**Check remote-config.json:**
```bash
cat ~/.config/opencode/remote-config.json
```

**Check cache:**
```bash
ls -la ~/.cache/opencode/remote-config/repos/
# Should contain cloned xskills repo
```

**Check symlinks:**
```bash
ls -la ~/.config/opencode/skill/_plugins/
# Should contain symlinked skills from xskills
```

**Force re-sync:**
```bash
rm -rf ~/.cache/opencode/remote-config/repos/
# Then restart OpenCode
```

### Git Authentication Issues

**For GitHub:**
- Ensure SSH key is added: `ssh -T git@github.com`
- Or use HTTPS with token: `https://github.com/xross/xskills.git`

**For private repositories:**
Add GitHub personal access token or use SSH keys.

### Skill Not Triggering

**Check skill description:**
The `description` field in SKILL.md frontmatter is critical. It must:
- Describe BOTH what the skill does AND when to trigger it
- Front-load trigger keywords ("repo_checks", "fix repo checks")
- Be specific enough to match user requests

**Test skill loading:**
```bash
# In OpenCode logs, search for skill loading messages
# Skills without descriptions are filtered out
```

---

## References

- **opencode-remote-config**: [GitHub](https://github.com/jgordijn/opencode-remote-config)
- **OpenCode Docs**: [opencode.ai/docs](https://opencode.ai/docs)
- **OpenCode Skills Guide**: [opencode.ai/docs/skills](https://opencode.ai/docs/skills)

---

## License

Personal repository - use at your own risk.

## Maintenance

This repository is actively maintained. Skills will be updated as XMOS tooling evolves.

**Feedback & issues:** Open issues on GitHub or modify skills directly.
