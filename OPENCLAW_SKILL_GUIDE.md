# OpenClaw Skill Installation Guide

## The Secret: What Makes a Skill Work

After extensive debugging, here's what's required to get a custom skill working in OpenClaw:

### 1. The `metadata` Field is Required

The skills CLI **will not recognize your skill** without the `metadata` field in your SKILL.md frontmatter:

```yaml
---
name: your-skill-name
description: Your skill description here
metadata:
  tags: tag1, tag2, tag3
---
```

Without `metadata`, you'll get: `No valid skills found. Skills require a SKILL.md with name and description.`

### 2. Directory Structure

Your GitHub repo must have this structure:

```
your-repo/
└── skills/
    └── your-skill-name/
        ├── SKILL.md          # Required
        └── references/       # Optional
            └── *.md
```

The skill folder name doesn't have to match the `name` in frontmatter, but it's recommended.

### 3. SKILL.md Format

```yaml
---
name: lowercase-with-hyphens
description: Single line description (max 1024 chars). Explain what it does AND when to use it.
metadata:
  tags: comma, separated, tags
---

# Skill Title

Your skill instructions in markdown...
```

**Name rules:**
- Lowercase letters, numbers, hyphens only (`a-z`, `0-9`, `-`)
- No spaces or underscores
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Max 64 characters

## Installation Steps

### Step 1: Create GitHub Repo

```bash
# Create repo with proper structure
mkdir -p my-skill/skills/my-skill-name/references
cd my-skill

# Create SKILL.md with required fields
cat > skills/my-skill-name/SKILL.md << 'EOF'
---
name: my-skill-name
description: What this skill does. Use when users want X, Y, or Z.
metadata:
  tags: relevant, tags, here
---

# My Skill

Instructions for the AI agent...
EOF

# Initialize and push to GitHub
git init
git add .
git commit -m "Initial commit"
gh repo create username/my-skill --public --source=. --push
```

### Step 2: Install via Skills CLI

```bash
npx skills add username/my-skill -g -y
```

This installs to `~/.agents/skills/my-skill-name/`

### Step 3: Create OpenClaw Symlink

```bash
ln -s ../../.agents/skills/my-skill-name ~/.openclaw/skills/my-skill-name
```

### Step 4: Verify

```bash
# Check skills CLI
npx skills list -g

# Check OpenClaw
openclaw skills list
openclaw skills info my-skill-name
```

## How It Works Under the Hood

### Two Skill Systems

1. **Agent Skills (`~/.agents/skills/`)** - Used by Claude Code, Codex, Cursor, etc.
   - Registry: `~/.agents/.skill-lock.json`
   - Installed via: `npx skills add`

2. **OpenClaw Skills (`~/.openclaw/skills/`)** - Used by OpenClaw
   - Reads from directory directly
   - Can symlink to Agent Skills

### Discovery Flow

```
GitHub Repo
    ↓
npx skills add (clones, validates, installs)
    ↓
~/.agents/skills/skill-name/
    ↓
Symlink to ~/.openclaw/skills/skill-name/
    ↓
OpenClaw discovers skill
```

## Troubleshooting

### "No valid skills found"
- Add `metadata:` field with `tags:` to your SKILL.md
- Ensure SKILL.md is in `skills/skill-name/` folder structure

### Skill not showing in `openclaw skills list`
- Create symlink: `ln -s ../../.agents/skills/name ~/.openclaw/skills/name`
- Check symlink resolves: `cat ~/.openclaw/skills/name/SKILL.md`

### Name validation errors
- Use only lowercase, numbers, hyphens
- Match folder name to `name` field in frontmatter

## Example Working Skill

Repository: https://github.com/banjtheman/travel-companion-skill

```
travel-companion-skill/
└── skills/
    └── travel-companion/
        ├── SKILL.md
        └── references/
            ├── agentmail.md
            └── browser.md
```

SKILL.md frontmatter:
```yaml
---
name: travel-companion
description: Plan trips, search flights/hotels on Expedia, discover destinations via TikTok, Instagram, and Google Maps, check weather, and email itineraries. Uses the OpenClaw-managed browser. Use when users want travel planning help, destination recommendations, flight/hotel searches, or trip itineraries.
metadata:
  tags: travel, trip, itinerary, flights, hotels, planning
---
```

## Quick Reference

| Requirement | Value |
|------------|-------|
| Required frontmatter | `name`, `description`, `metadata` |
| Name format | lowercase, hyphens, numbers only |
| Folder structure | `skills/skill-name/SKILL.md` |
| Install command | `npx skills add user/repo -g -y` |
| OpenClaw symlink | `~/.openclaw/skills/` → `~/.agents/skills/` |
