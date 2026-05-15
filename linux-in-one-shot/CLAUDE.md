# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a structured Linux learning resource ("linux-in-one-shot") organized as a collection of topic-based folders. Each topic has its own folder containing a `README.md` with educational content. Content is added incrementally — the user provides README files one by one and each gets its own folder committed and pushed to GitHub (`sidraaiman/linux-in-one-shot`).

## Workflow

When the user provides a README for a new topic:
1. Create a new folder named after the topic (kebab-case, e.g., `file-permissions/`, `process-management/`)
2. Place the content as `README.md` inside that folder
3. Commit with message: `add <topic-name> module`
4. Push to `origin master`

## Remote

```
origin  https://github.com/sidraaiman/linux-in-one-shot.git
```

Push with: `git push origin master`

## Folder Naming Convention

- Use lowercase kebab-case: `basic-commands/`, `user-management/`, `networking/`
- Each folder contains exactly one `README.md`
- No subfolders within topic folders unless the topic is large enough to warrant sections
