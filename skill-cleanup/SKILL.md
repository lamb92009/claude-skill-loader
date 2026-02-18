---
name: skill-cleanup
description: Removes session-fetched skills from ~/.claude/skills/ using the .session-skills manifest, keeping the directory lean between sessions.
---

# Skill Cleanup

<!-- ============================================================
  CONFIGURATION — must match your skill-loader configuration
  ============================================================ -->

<!--
  PERMANENT_SKILLS: skills that are always locally installed and must
                    never be removed, even if they appear in the manifest.
                    Must match the PERMANENT_SKILLS list in skill-loader.
-->

**PERMANENT_SKILLS:** `skill-loader, skill-cleanup` *(add any others here, e.g. a brand-reference skill)*

<!-- ============================================================ -->

Remove skills that were fetched for this session, restoring `~/.claude/skills/` to its lean baseline.

## Step 1 — Check for manifest

```bash
cat ~/.claude/skills/.session-skills 2>/dev/null
```

If the file does not exist or is empty, report:

> No session skills found — `~/.claude/skills/` is already clean.

Then stop.

## Step 2 — Read the manifest

Parse the list of skill names (one per line). These are the skills to remove.

## Step 3 — Remove session skills

For each skill name in the manifest:

```bash
rm -rf ~/.claude/skills/<skill-name>
echo "Removed: <skill-name>"
```

**Never remove permanent residents** (defined in **PERMANENT_SKILLS** above) — even if they appear in the manifest. Skip any name that matches a permanent skill.

*(Replace this safeguard list with your configured PERMANENT_SKILLS values)*
- `skill-loader`
- `skill-cleanup`

## Step 4 — Remove the manifest

```bash
rm -f ~/.claude/skills/.session-skills
echo "Manifest removed."
```

## Step 5 — Confirm

Show what remains:

```bash
ls ~/.claude/skills/
```

Report what was removed and what permanent skills remain.
