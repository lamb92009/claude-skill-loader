---
name: skill-loader
description: Reads the remote skill catalog from GitHub, analyzes project context, recommends relevant skills, and fetches them for the session.
---

# Skill Loader

<!-- ============================================================
  CONFIGURATION — edit these values before using this skill
  ============================================================ -->

<!--
  GITHUB_USERNAME:   your GitHub username (e.g. acme-corp)
  REPO_NAME:         the repo that holds your skills (e.g. my-claude-skills)
  SKILLS_REPO_URL:   raw GitHub URL to your catalog README
  SKILLS_REPO_API:   GitHub API base path for the repo (no trailing slash)
  PERMANENT_SKILLS:  skills that are always locally installed — never fetch,
                     never add to the manifest, never clean up
-->

**GITHUB_USERNAME:** `your-github-username`
**REPO_NAME:** `your-skills-repo`
**SKILLS_REPO_URL:** `https://raw.githubusercontent.com/your-github-username/your-skills-repo/main/README.md`
**SKILLS_REPO_API:** `repos/your-github-username/your-skills-repo`
**PERMANENT_SKILLS:** `skill-loader, skill-cleanup` *(add any others here, e.g. a brand-reference skill)*

<!-- ============================================================ -->

Load only the skills you need for this session from the central skill repository.

## Step 1 — Fetch the catalog

Run this to get the README (substitute your configured `SKILLS_REPO_API`):

```bash
bash -c 'source ~/.bashrc && gh api {SKILLS_REPO_API}/contents/README.md --jq ".content" | base64 -d'
```

Read and understand the full catalog of available skills and their descriptions.

## Step 2 — Collect project context signals

Check each of the following (skip silently if not present):

```bash
# What files exist in the working directory?
ls -la

# Package manager / language signals
cat package.json 2>/dev/null | head -30
cat requirements.txt 2>/dev/null | head -10
cat go.mod 2>/dev/null | head -5
cat Cargo.toml 2>/dev/null | head -5
cat composer.json 2>/dev/null | head -10

# Project instructions
cat CLAUDE.md 2>/dev/null
cat .claude/CLAUDE.md 2>/dev/null

# Infrastructure signals
cat Dockerfile 2>/dev/null | head -20
cat docker-compose.yaml 2>/dev/null | head -20
cat docker-compose.yml 2>/dev/null | head -20
```

Also note:
- What task or goal has the user stated for this session?
- What file extensions dominate the working directory?

## Permanent residents (always available — never fetch, never add to manifest)

These skills (defined in **PERMANENT_SKILLS** above) are installed locally at all times. Do not recommend fetching them and do not write them to `.session-skills`:

*(Replace this list with your configured PERMANENT_SKILLS values)*
- `skill-loader`
- `skill-cleanup`

## Step 3 — Generate recommendations

Based on the catalog descriptions and project signals, select skills that are **directly relevant** to this project and session. For each recommendation, give one sentence of reasoning tied to a specific signal. **Skip any permanent residents — they are always available without fetching.**

**Aim for 3–8 skills.** Be selective — do not recommend everything. Skills you don't recommend are still available if asked.

## Step 4 — Present and confirm

Present in this format:

```
Based on [key signals observed], I recommend fetching these skills:

✓ `skill-name` — reason tied to specific signal
✓ `skill-name` — reason tied to specific signal
...

Also available but not recommended for this project: skill-a, skill-b, skill-c

Confirm this list, adjust it, or add skill names?
```

**Wait for the user to respond before fetching anything.**

## Step 5 — Fetch approved skills

For each confirmed skill name, run (substitute your `SKILLS_REPO_API`):

```bash
# Fetch SKILL.md
bash -c 'source ~/.bashrc && \
  SKILL=<skill-name> && \
  mkdir -p ~/.claude/skills/$SKILL && \
  gh api "{SKILLS_REPO_API}/contents/$SKILL/SKILL.md" --jq ".content" | base64 -d > ~/.claude/skills/$SKILL/SKILL.md && \
  echo "Fetched: $SKILL/SKILL.md"'
```

Then fetch any reference files for that skill:

```bash
bash -c 'source ~/.bashrc && \
  SKILL=<skill-name> && \
  REFS=$(gh api "{SKILLS_REPO_API}/git/trees/main?recursive=1" --jq ".tree[] | select(.path | startswith(\"$SKILL/references/\")) | .path") && \
  if [ -n "$REFS" ]; then
    mkdir -p ~/.claude/skills/$SKILL/references
    echo "$REFS" | while IFS= read -r path; do
      fname=$(basename "$path")
      gh api "{SKILLS_REPO_API}/contents/$path" --jq ".content" | base64 -d > ~/.claude/skills/$SKILL/references/$fname
      echo "Fetched: $path"
    done
  fi'
```

Repeat for each skill in the confirmed list.

## Step 6 — Write the session manifest

```bash
cat > ~/.claude/skills/.session-skills << 'MANIFEST'
<skill1>
<skill2>
<skill3>
MANIFEST
```

Write one skill name per line, listing every skill that was fetched in this session.

## Step 7 — Confirm

Report which skills were fetched and are now active. Remind the user:

> When you're done with this session, invoke `skill-cleanup` to remove these skills and keep `~/.claude/skills/` lean.
