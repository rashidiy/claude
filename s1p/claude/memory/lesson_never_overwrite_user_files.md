---
name: Never overwrite user files from git history
description: Never restore files from git over uncommitted user work — always work on copies in .tmp first
type: feedback
---

NEVER overwrite files in the working directory by restoring from git history without confirming the user has no uncommitted changes.

**Why:** User spent hours manually fixing 14 illustration PNGs (extracting from source, removing backgrounds, cleaning edges). Those fixes were uncommitted. Agent restored "originals" from an old git commit, overwrote all the user's work, then ran processing on the old versions. User's hours of manual work were permanently lost.

**How to apply:**
1. Before touching any user file, check `git status` — if files have uncommitted changes, those changes ARE the user's work
2. NEVER write directly to source files when processing. Always write to `.tmp/` first, show the user, and only copy to final location after confirmation
3. If you need "originals" to process, copy the CURRENT on-disk files to `.tmp/backup/` first — not the git history versions
4. Treat uncommitted changes as MORE valuable than committed ones — they represent recent manual work that can't be recovered
5. Binary files (images, etc.) cannot be recovered from git reflog if overwritten — extra caution required
