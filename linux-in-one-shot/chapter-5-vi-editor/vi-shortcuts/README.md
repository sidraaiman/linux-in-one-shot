# VI Editor Shortcuts

## Why Learn VI?

`vi` (and its improved version `vim`) is installed on virtually every Linux system — from minimal Alpine containers to enterprise RHEL servers. When you SSH into a remote machine, nano may not be available, but `vi` always is. It is the editor that works everywhere, which makes it a non-negotiable tool for anyone working with Linux systems.

---

## Understanding VI Modes

VI is a **modal editor** — the same keys do different things depending on the active mode. This is its biggest difference from editors like nano or VS Code.

```
┌─────────────────────────────────────────────────────┐
│                   NORMAL MODE                       │
│           (default on startup)                      │
│     Navigate, delete, copy, paste, commands         │
│                                                     │
│   Press i/a/o ──────────────► INSERT MODE          │
│   Press Esc  ◄────────────── (type text here)      │
│                                                     │
│   Press :    ──────────────► COMMAND MODE          │
│   Press Esc  ◄────────────── (save, quit, search)  │
└─────────────────────────────────────────────────────┘
```

| Mode | How to Enter | What It Does |
|------|-------------|--------------|
| **Normal** | `Esc` (from any mode) | Navigation, deletion, copy/paste, executing commands |
| **Insert** | `i`, `a`, `o`, `I`, `A`, `O` | Typing and editing text |
| **Command** | `:` (from Normal mode) | Save, quit, search, replace, split windows |
| **Visual** | `v` (from Normal mode) | Select text for copy/cut operations |

> The most common mistake for beginners: typing text in Normal mode and wondering why the cursor is jumping around. Always check your mode. When in doubt, press `Esc` — it always returns you to Normal mode.

---

## Opening and Closing VI

```bash
vi filename.txt          # Open a file
vi +42 filename.txt      # Open file and jump to line 42
vi +/pattern file.txt    # Open file and jump to first match of pattern
vim filename.txt         # Open with vim (Vi Improved — recommended)
```

---

## 1. Basic Navigation (Normal Mode)

### Character and Line Movement

| Key | Action |
|-----|--------|
| `h` | Move left one character |
| `l` | Move right one character |
| `j` | Move down one line |
| `k` | Move up one line |
| `0` | Move to the very beginning of the line |
| `^` | Move to the first non-blank character of the line |
| `$` | Move to the end of the line |

> You can prefix any motion with a number to repeat it. `5j` moves down 5 lines. `10l` moves right 10 characters.

### Word Movement

| Key | Action |
|-----|--------|
| `w` | Move to the start of the next word |
| `W` | Move to the start of the next WORD (ignores punctuation) |
| `b` | Move to the start of the previous word |
| `B` | Move to the start of the previous WORD |
| `e` | Move to the end of the current word |

### File-Level Navigation

| Key | Action |
|-----|--------|
| `gg` | Go to the first line of the file |
| `G` | Go to the last line of the file |
| `:n` | Go to line number `n` (e.g., `:42` goes to line 42) |
| `Ctrl+d` | Scroll down half a page |
| `Ctrl+u` | Scroll up half a page |
| `Ctrl+f` | Scroll down full page |
| `Ctrl+b` | Scroll up full page |
| `H` | Move cursor to top of visible screen |
| `M` | Move cursor to middle of visible screen |
| `L` | Move cursor to bottom of visible screen |

---

## 2. Insert Mode — Entering Text

All of these enter Insert mode but position the cursor differently:

| Key | Action |
|-----|--------|
| `i` | Insert **before** the cursor |
| `I` | Insert at the **beginning** of the line |
| `a` | Append (insert) **after** the cursor |
| `A` | Append at the **end** of the line |
| `o` | Open a new line **below** and enter insert mode |
| `O` | Open a new line **above** and enter insert mode |
| `s` | Delete character under cursor and enter insert mode |
| `S` | Delete entire line and enter insert mode |
| `Esc` | Exit Insert mode → return to Normal mode |

> Tip: `o` (open line below) is the fastest way to start typing on a new line without navigating to the end of the current line first.

---

## 3. Editing Text (Normal Mode)

### Deleting

| Key | Action |
|-----|--------|
| `x` | Delete character **under** cursor |
| `X` | Delete character **before** cursor |
| `dw` | Delete from cursor to start of next word |
| `de` | Delete from cursor to end of current word |
| `dd` | Delete (cut) entire current line |
| `2dd` | Delete 2 lines |
| `d$` or `D` | Delete from cursor to end of line |
| `d0` | Delete from cursor to beginning of line |
| `dgg` | Delete from current line to start of file |
| `dG` | Delete from current line to end of file |

> Deleted text in VI is placed in a register and can be pasted with `p`. Think of `dd` as "cut line" not just "delete line."

### Copying (Yanking) and Pasting

| Key | Action |
|-----|--------|
| `yy` | Copy (yank) the current line |
| `2yy` | Copy 2 lines |
| `yw` | Copy from cursor to start of next word |
| `y$` | Copy from cursor to end of line |
| `ygg` | Copy from current line to start of file |
| `yG` | Copy from current line to end of file |
| `p` | Paste **after** the cursor (or below current line for full lines) |
| `P` | Paste **before** the cursor (or above current line for full lines) |

### Changing Text

| Key | Action |
|-----|--------|
| `cw` | Delete word and enter Insert mode |
| `cc` or `S` | Delete entire line and enter Insert mode |
| `c$` or `C` | Delete to end of line and enter Insert mode |
| `r<char>` | Replace single character under cursor with `<char>` |
| `R` | Enter Replace mode — overwrites characters as you type |

### Undo and Redo

| Key | Action |
|-----|--------|
| `u` | Undo last change |
| `U` | Undo all changes on current line |
| `Ctrl+r` | Redo (re-apply undone change) |

> In `vim`, undo history is persistent and deep — you can undo dozens of changes. In the original `vi`, undo is limited.

### Indentation

| Key | Action |
|-----|--------|
| `>>` | Indent current line right |
| `<<` | Indent current line left |
| `5>>` | Indent 5 lines right |

---

## 4. Visual Mode — Selecting Text

Visual mode lets you select a range of text before applying an operation.

| Key | Action |
|-----|--------|
| `v` | Enter character-wise Visual mode |
| `V` | Enter line-wise Visual mode |
| `Ctrl+v` | Enter block Visual mode (column selection) |

After selecting text with visual mode:

| Key | Action |
|-----|--------|
| `d` | Delete selected text |
| `y` | Yank (copy) selected text |
| `>` | Indent selected block right |
| `<` | Indent selected block left |
| `~` | Toggle case of selected text |

---

## 5. Search (Normal Mode)

| Key | Action |
|-----|--------|
| `/pattern` | Search **forward** for pattern |
| `?pattern` | Search **backward** for pattern |
| `n` | Jump to **next** match (same direction) |
| `N` | Jump to **previous** match (reverse direction) |
| `*` | Search forward for word under cursor |
| `#` | Search backward for word under cursor |

```
# Example: search for the word "error"
/error    then press Enter
n         move to next occurrence
N         move to previous occurrence
```

To clear search highlighting in vim:
```
:noh
```

---

## 6. Search and Replace (Command Mode)

| Command | Action |
|---------|--------|
| `:%s/old/new/g` | Replace **all** occurrences in the entire file |
| `:%s/old/new/gc` | Replace all, but **confirm** each substitution |
| `:s/old/new/g` | Replace all occurrences on **current line only** |
| `:5,15s/old/new/g` | Replace in lines 5 through 15 |
| `:%s/old/new/gi` | Replace all, **case-insensitive** |
| `:%s/\bold\b/new/g` | Replace whole word only (not substring) |

Confirmation prompt (`c` flag) options:
- `y` — yes, replace this one
- `n` — no, skip this one
- `a` — replace all remaining
- `q` — quit substitution
- `l` — replace this one then quit

---

## 7. Saving and Quitting (Command Mode)

| Command | Action |
|---------|--------|
| `:w` | Save (write) the file |
| `:w filename.txt` | Save to a different filename |
| `:wq` or `:x` | Save and quit |
| `ZZ` | Save and quit (Normal mode shortcut) |
| `:q` | Quit (only if no unsaved changes) |
| `:q!` | Quit **without saving** (force) |
| `ZQ` | Quit without saving (Normal mode shortcut) |
| `:wq!` | Force save and quit (for read-only files you own) |

---

## 8. Working with Multiple Files

### Opening Files

| Command | Action |
|---------|--------|
| `:e filename` | Open a file in the current window |
| `:e!` | Reload current file from disk (discard changes) |

### Split Windows

| Command | Action |
|---------|--------|
| `:split filename` | Split screen **horizontally**, open file |
| `:vsplit filename` | Split screen **vertically**, open file |
| `:sp` | Split horizontally (same file) |
| `:vsp` | Split vertically (same file) |

### Navigating Between Splits

| Key | Action |
|-----|--------|
| `Ctrl+w w` | Switch to next window |
| `Ctrl+w h` | Move to window on the **left** |
| `Ctrl+w l` | Move to window on the **right** |
| `Ctrl+w j` | Move to window **below** |
| `Ctrl+w k` | Move to window **above** |
| `Ctrl+w =` | Make all windows equal size |
| `Ctrl+w +` | Increase current window height |
| `Ctrl+w -` | Decrease current window height |
| `:close` | Close current split |
| `:only` | Close all splits except current |

---

## 9. Useful Miscellaneous Commands

| Command / Key | Action |
|--------------|--------|
| `:set number` | Show line numbers |
| `:set nonumber` | Hide line numbers |
| `:set paste` | Disable auto-indent when pasting from clipboard |
| `:set nopaste` | Re-enable auto-indent |
| `:syntax on` | Enable syntax highlighting |
| `:%d` | Delete all lines in the file |
| `gg=G` | Auto-indent the entire file |
| `:!command` | Run a shell command without leaving vim (e.g., `:!ls`) |
| `Ctrl+z` | Suspend vim and return to shell (`fg` to resume) |

---

## 10. Quick Survival Guide

If you're new to vim and just need to edit a file and get out:

```
1. Open the file:     vim filename.txt
2. Navigate:          use arrow keys or h j k l
3. Start editing:     press i
4. Edit your text
5. Stop editing:      press Esc
6. Save and quit:     type :wq and press Enter
7. Quit without save: type :q! and press Enter
```

---

## Full Cheat Sheet

### Navigation
| Key | Action |
|-----|--------|
| `h / j / k / l` | Left / Down / Up / Right |
| `w / b` | Next / Previous word |
| `0 / ^/ $` | Line start / first char / line end |
| `gg / G` | File start / File end |
| `:n` | Go to line n |
| `Ctrl+d / Ctrl+u` | Half page down / up |

### Insert Mode Entry
| Key | Inserts At |
|-----|-----------|
| `i / I` | Before cursor / Line start |
| `a / A` | After cursor / Line end |
| `o / O` | New line below / above |

### Edit
| Key | Action |
|-----|--------|
| `x / dd / dw` | Delete char / line / word |
| `yy / yw` | Copy line / word |
| `p / P` | Paste after / before |
| `u / Ctrl+r` | Undo / Redo |
| `r<c>` | Replace single char |
| `cw / cc` | Change word / line |

### Command Mode
| Command | Action |
|---------|--------|
| `:w / :q / :wq / :q!` | Save / Quit / Save+Quit / Force quit |
| `/pat / ?pat` | Search forward / backward |
| `:%s/old/new/g` | Replace all |
| `:split / :vsplit` | Horizontal / vertical split |

---

**← Back to [Chapter 4: File Management](../../chapter-4-file-management/file-management/README.md)**  
**Next → Chapter 6: File Permissions**
