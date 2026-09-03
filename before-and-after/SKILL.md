---
name: before-and-after
description: Captures before/after screenshots of web pages or elements for visual comparison. Use when user says "take before and after", "screenshot comparison", "visual diff", "PR screenshots", "compare old and new", or needs to document UI changes. Accepts two URLs (file://, http://, https://) or two image paths.
allowed-tools:
  - Bash(npx @vercel/before-and-after *)
  - Bash(before-and-after *)
  - Bash(which before-and-after)
  - Bash(npm install -g @vercel/before-and-after)
  - Bash(*/upload-and-copy.sh *)
  - Bash(curl -s -o /dev/null -w *)
  - Bash(gh pr view *)
  - Bash(gh pr edit *)
  - Bash(vercel inspect *)
  - Bash(vercel whoami)
  - Bash(which vercel)
  - Bash(which gh)
---

# Before-After Screenshot Skill

> **Package:** `@vercel/before-and-after`
> Never use `before-and-after` (wrong package).

## Agent Behavior Rules

**DO NOT:**
- Switch git branches, stash changes, start dev servers, or assume what "before" is
- Use `--full` unless user explicitly asks for full page / full scroll capture
- **Commit or push the before/after PNG (or any other media) files to the repo.**
  The PNGs are PR attachments, not source. See "Never commit the images" below.

**DO:**
- Use `--markdown` when user wants PR integration or markdown output
- Use `--mobile` / `--tablet` if user mentions phone, mobile, tablet, responsive, etc.
- Assume current state is **After**
- If user provides only one URL or says "PR screenshots" without URLs, **ASK**: "What URL should I use for the 'before' state? (production URL, preview deployment, or another local port)"

## Never commit the images

Before/after screenshots are **PR description content**, never repo content.
The repo keeps code; the PR keeps the proof.

- Capture locally (default: `~/Downloads/`), then upload via
  `./scripts/upload-and-copy.sh before.png after.png --markdown` — that
  produces hosted URLs and a markdown table ready to paste into the PR
  body.
- **Never** `git add` the `.png` (or `.jpg`/`.webp`/`.mp4`/`.webm`) files.
  The PR description gets the markdown; the commit gets the code change.
- Keep raw captures under a gitignored folder (`.artifacts/`,
  `evidence/`, `screenshots/`) so they can be reused as input to a later
  run without ever reaching git.
- `.gitignore` should include at least: `.artifacts/`, `evidence/`,
  `screenshots/`, `before-after/`. The `before-and-after` CLI's default
  output dir (`~/Downloads/`) is already outside the repo, so commits
  inside the repo aren't a risk from there — but explicit gitignore lines
  protect against the agent dropping captures into the project by mistake.
- Exceptions (e.g. an image baked into a docs page, a fixture read by
  tests) must be justified in the PR body — visual proof of a UI change
  is not an exception.

## Execution Order (MUST follow)

1. **Pre-flight** — `which before-and-after || npm install -g @vercel/before-and-after`
2. **Protection check** — if `.vercel.app` URL: `curl -s -o /dev/null -w "%{http_code}" "<url>"` (401/403 = protected)
3. **Capture** — `before-and-after "<before-url>" "<after-url>"`
4. **Upload** — `./scripts/upload-and-copy.sh <before.png> <after.png> --markdown`
5. **PR integration** — optionally `gh pr edit` to append markdown

**Never skip steps 1-2.**

## Quick Reference

```bash
# Basic usage
before-and-after <before-url> <after-url>

# With selector
before-and-after url1 url2 ".hero-section"

# Different selectors for each
before-and-after url1 url2 ".old-card" ".new-card"

# Viewports
before-and-after url1 url2 --mobile    # 375x812
before-and-after url1 url2 --tablet    # 768x1024
before-and-after url1 url2 --full      # full scroll

# From existing images
before-and-after before.png after.png --markdown

# Via npx (use full package name!)
npx @vercel/before-and-after url1 url2
```

| Flag | Description |
|------|-------------|
| `-m, --mobile` | Mobile viewport (375x812) |
| `-t, --tablet` | Tablet viewport (768x1024) |
| `--size <WxH>` | Custom viewport |
| `-f, --full` | Full scrollable page |
| `-s, --selector` | CSS selector to capture |
| `-o, --output` | Output directory (default: ~/Downloads) |
| `--markdown` | Upload images & output markdown table |
| `--upload-url <url>` | Custom upload endpoint (default: 0x0.st) |

## Image Upload

```bash
# Default (0x0.st - no signup needed)
./scripts/upload-and-copy.sh before.png after.png --markdown

# GitHub Gist
IMAGE_ADAPTER=gist ./scripts/upload-and-copy.sh before.png after.png --markdown
```

## Vercel Deployment Protection

If `.vercel.app` URL returns 401/403:

1. Check Vercel CLI: `which vercel && vercel whoami`
2. If available: `vercel inspect <url>` to get bypass token
3. If not: Tell user to provide bypass token, take manual screenshots, or disable protection

## PR Integration

```bash
# Check for gh CLI
which gh

# Get current PR
gh pr view --json number,body

# Append screenshots to PR body
gh pr edit <number> --body "<existing-body>

## Before and After
<generated-markdown>"
```

If no `gh` CLI: output markdown and tell user to paste manually.

## Error Reference

| Error | Fix |
|-------|-----|
| `command not found` | `npm install -g @vercel/before-and-after` |
| `could not determine executable` | Use `npx @vercel/before-and-after` (full name) |
| 401/403 on .vercel.app | See Vercel protection section |
| Element not found | Verify selector exists on page |
