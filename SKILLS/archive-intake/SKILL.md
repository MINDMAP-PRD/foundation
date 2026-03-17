---
name: archive-intake
description: Use when the source material arrives as a zip or other archive and the agent must preserve the archive, unpack it into a readable folder, save the extracted contents, and then read the right files in a controlled order.
---

# Archive Intake

Handle zipped or archived source material as a proper intake step before planning, auditing, or building.

## When to Use

- the user provides a `.zip` archive
- the user asks to unzip, extract, save, or ingest an archive
- the project source arrives as a compressed handoff instead of an open folder
- the next task depends on reading files that are trapped inside an archive

## Core Rules

1. Preserve the original archive unless the user explicitly asks to delete it.
2. Extract into a sibling folder with a clear deterministic name.
3. Keep the extracted contents readable and stable for later agents.
4. After extraction, switch from archive handling to normal repo/doc intake.
5. If the environment cannot extract archives, say so clearly and ask for an unzipped folder or a Git source instead of pretending the archive was read.

## Extraction Workflow

### 1. Save the archive safely

- keep the original file
- do not overwrite it
- use a clear extracted-folder name based on the archive name

Example:

- `handoff.zip` -> `handoff/`
- if that exists already, use `handoff-extracted/` or a timestamped variant

### 2. Unpack the archive

- extract the full contents
- preserve the internal folder structure
- avoid flattening nested folders
- avoid mixing extracted files into unrelated repo roots

### 3. Normalize the intake point

After extraction:

- if the archive contains one top-level folder, use that folder as the intake root
- if the archive spills files directly at root, keep them in the extracted container folder

### 4. Read in the right order

After extraction, look for the canonical entry points first:

- `AGENTS.md`
- `README.md`
- `README_FOR_LOVEABLE.md`
- `FOUNDATION/README.md`
- `SKILLS/SKILL_INTERCONNECTION_MAP.md`
- `BUILD-PLAN/README.md`

Then hand off to the correct skill for the actual job:

- `foundation-module-factory` for governed packaging/build intake
- `foundation-extension-auditor` for ownership/audit work
- `project-mapper` for rich project mapping
- `coding-standards`, `api-design`, `backend-patterns`, or others after intake is complete

## Loveable / Limited-Tool Fallback

If the agent runner cannot extract archives directly:

- do not claim the zip was read
- tell the user extraction is blocked in that environment
- ask for one of:
  - the unzipped folder contents
  - a Git repo
  - the key files pasted directly

## Output Expectation

A successful archive-intake pass should leave:

- the original archive still present
- a saved extracted folder
- a clear identified intake root
- the next-step reading order and handoff target identified
