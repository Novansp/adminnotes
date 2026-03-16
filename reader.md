```markdown
# Reading Markdown on an Ubuntu Server via SSH (PuTTY)

When connecting to an Ubuntu server through SSH using PuTTY, reading Markdown (`.md`) files with simple commands such as `cat` can be difficult because Markdown formatting is not rendered. Terminal-friendly tools improve readability by formatting or highlighting the Markdown content.

This guide explains how to configure PuTTY and the best tools for reading Markdown files on a remote Ubuntu server.

---

# 1. Enable UTF-8 in PuTTY (Required)

Before using any Markdown reader, configure PuTTY to display UTF-8 characters correctly. Without this setting, tables and formatting characters may appear as unreadable symbols.

### Steps

1. Open PuTTY.
2. Navigate to:

```

Window → Translation

```

3. Set:

```

Remote character set: UTF-8

```

4. Return to:

```

Session

````

5. Save the profile and connect to the server.

---

# 2. Best Markdown Reader: `glow`

`glow` is a command-line Markdown renderer designed for terminal environments. It properly formats:

- Headings
- Lists
- Tables
- Code blocks
- Syntax highlighting

This produces a clean and readable display directly in the PuTTY terminal.

## Installation (APT Repository Method)

Some servers do not include Snap, so the official APT repository is the most reliable installation method.

```bash
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://repo.charm.sh/apt/gpg.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/charm.gpg

echo "deb [signed-by=/etc/apt/keyrings/charm.gpg] https://repo.charm.sh/apt/ * *" \
| sudo tee /etc/apt/sources.list.d/charm.list

sudo apt update
sudo apt install glow
````

### Usage

```bash
glow README.md
```

### Theme Tip

If colors appear incorrect in PuTTY, force a theme:

```bash
glow -s dark README.md
```

or

```bash
glow -s light README.md
```

### Without sudo

If administrative access is unavailable, download the standalone binary from the project's GitHub releases page and run it from the user directory.

---

# 3. Enhanced File Viewer: `bat`

`bat` is an improved replacement for `cat`. It does not fully render Markdown but provides syntax highlighting, which improves readability over SSH.

Highlighted elements include:

* Markdown syntax characters
* Links
* Code blocks

## Installation

```bash
sudo apt update
sudo apt install bat
```

### Usage on Ubuntu

Ubuntu installs the command as:

```bash
batcat README.md
```

---

# 4. Built-in Tools (No Installation Required)

On restricted servers where installing packages is not allowed, use built-in Linux utilities.

---

## 4.1 Using `less` (Best for Long Files)

`less` allows scrolling through long documents without flooding the terminal screen.

```bash
less README.md
```

### Navigation

| Key              | Function             |
| ---------------- | -------------------- |
| Up / Down arrows | Scroll line-by-line  |
| Space            | Scroll one page down |
| `/text`          | Search for text      |
| `q`              | Quit                 |

---

## 4.2 Using Text Editors

Text editors often provide basic syntax highlighting for Markdown.

### Nano (Beginner-friendly)

```bash
nano README.md
```

Exit:

```
Ctrl + X
```

### Vim (Advanced editor)

```bash
vim README.md
```

Exit:

```
:q
```

---

# Summary Recommendations

### Step 1 — Fix PuTTY

Ensure UTF-8 encoding is enabled:

```
Window → Translation → UTF-8
```

### Step 2 — Choose a Viewer

| Scenario                        | Recommended Tool |
| ------------------------------- | ---------------- |
| Best visual rendering           | `glow`           |
| Syntax highlighted raw Markdown | `bat` (`batcat`) |
| No installation allowed         | `nano` or `less` |

For most SSH users with installation permissions, `glow` provides the best Markdown reading experience in a terminal environment.

```
```
