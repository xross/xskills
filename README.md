# xskills

Skills repository for XMOS-based sofware development.

## Skills

### fix-repo-checks

Automatically fixes `test_repo_checks` failures in XMOS library repositories.

**What it does:**
- Runs pytest to identify failures
- Sets up infr_apps environment (clones if needed, creates `.venv_<repo_name>`)
- Auto-fixes: copyright headers, LICENSE.rst, CHANGELOG.rst
- AI-fixes: README.rst, settings.yml, version consistency, metadata, docs, deps
- Validates `xmos_public_v1` license appropriateness
- Reports results (changes left uncommitted)

---

## Setup (OpenCode)

How to sync automatically with OpenCode's remote config plugin.

### 1. Install opencode-remote-config plugin

```bash
npm install -g @jgordijn/opencode-remote-config
```

### 2. Configure OpenCode

Add to `~/.config/opencode/opencode.json`:
```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["@jgordijn/opencode-remote-config"]
}
```

Create `~/.config/opencode/remote-config.json`:
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

### 3. Restart OpenCode

Skills sync on startup.

---

## Usage

From any XMOS library repository:
```
You: fix repo checks
```

---

## Development

### Adding Skills

```bash
mkdir -p skill/my-new-skill
# Create skill/my-new-skill/SKILL.md with frontmatter
git add . && git commit -m "Add skill" && git push
```

### Local Testing

Use `file:///path/to/xskills` in `remote-config.json` for rapid iteration.

---

## Structure

```
xskills/
├── README.md
└── skill/
    └── fix-repo-checks/SKILL.md
```

Can add `agent/*.md`, `command/*.ts`, `plugin/*.ts` later.

---

## Troubleshooting

**Plugin not loading:** Check `~/.config/opencode/opencode.json` contains plugin entry

**Skills not syncing:** Delete `~/.cache/opencode/remote-config/repos/` and restart OpenCode

**Git auth issues:** Use SSH (`git@github.com:...`) or HTTPS with token
