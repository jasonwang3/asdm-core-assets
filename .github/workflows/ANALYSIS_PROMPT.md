You are analyzing a cloned repository in the current working directory. Your job is to produce onboarding-oriented context space documentation.

IMPORTANT — Output location:
- Write all markdown files ONLY under this directory (it already exists; do not create a different result root):
  {RESULT_DIR}
- Use absolute paths when calling the Write tool, e.g. {RESULT_DIR}/project-overview.md

IMPORTANT — How to write files:
- Use the **Write** tool to create or overwrite files. Do not rely on "write_to_file" or other names that do not exist in this environment.
- Prefer the Write tool over shell redirection (avoid `echo > file` or heredocs unless Write is unavailable).
- Use UTF-8 plain text. File extension must be `.md`.

Create the following files:

1) **project-overview.md**
   - One short executive summary (what this repo is for).
   - Tech stack and runtime requirements (languages, major frameworks, package managers).
   - Top-level directory layout: what each main folder is for.
   - Main entry points (e.g. main app, CLI, server bootstrap) and how to run/build if documented.
   - Key features or modules and how they relate.
   - Configuration: important env vars or config files.
   - Testing/CI pointers if present.
   - Any notable conventions from README or AGENTS.md if present.

2) **architecture.md** (if the codebase is non-trivial; otherwise still create it with a concise section stating "small/simple project" and listing core modules)
   - High-level architecture (layers, major packages, data flow).
   - Important abstractions, patterns, or integrations.
   - External services/APIs the code depends on.

3) **glossary.md**
   - Domain terms, acronyms, and project-specific names a new developer must know.

Quality bar:
- Be accurate; base claims on files you have read. If something is unknown, say so briefly instead of guessing.
- Use clear Markdown headings (##, ###) and bullet lists where helpful.

When finished:
- Add a final short message listing the absolute paths of every file you wrote under {RESULT_DIR}.