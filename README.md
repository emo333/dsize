<img width="728" height="174" alt="dsize-logo" src="https://github.com/user-attachments/assets/7c1e8e9a-a709-4665-ba9d-47b37d949cb8" />

# dsize

[Website Post on dsize](https://www.csimw.com/post/dsize-terminal-tool)

A fast terminal tool that shows human-readable table of directory sizes, with mount point info.

_Because `du` alone will make you forget why you opened the terminal ;)_

## Features

- **Single `du` pass** — gathers all sizes efficiently via `du -h --max-depth=1`. (Yes, just one. Not three. You're welcome.)
- **Color-coded output** — sizes are colorized by magnitude (grey → green → yellow → orange → red).
- **Mount point info** — shows filesystem usage stats for the current partition.
- **Inode deduplication** — skips bind mounts and symlinked duplicates.
- **Pure Bash + GNU coreutils** — no Python, Node, or other runtime dependencies.

## Installation

Make the script executable:

```bash
chmod +x dsize
```

### Making `dsize` a global command

Choose one of the following methods to invoke `dsize` from anywhere.

#### Option 1 — Install to `/usr/local/bin/` (system-wide, requires root)

```bash
sudo cp dsize /usr/local/bin/dsize
sudo chmod +x /usr/local/bin/dsize
```

Then run it simply as `dsize` or `dsize /some/path`.

#### Option 2 — Install to `~/.local/bin/` (user-only, no root needed)

```bash
mkdir -p ~/.local/bin
cp dsize ~/.local/bin/dsize
chmod +x ~/.local/bin/dsize
```

Ensure `~/.local/bin` is on your `PATH`:

```bash
# Add to ~/.bashrc or ~/.zshrc if not already there:
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### Option 3 — Alias or function in your shell RC file

Add one of the following to `~/.bashrc` (or `~/.zshrc`):

1: **Alias (always scans the current directory — handy if you're too lazy to type a path):**

```bash
alias dsize='bash "$(dirname "$0")/dsize"'
```

2: **Function (supports optional path argument):**

```bash
dsize() {
    bash "$(dirname "${BASH_SOURCE[0]}")/dsize" "${1:-.}"
}
```

Then reload your shell:

```bash
source ~/.bashrc
```

### Use Globally ( then you can say "I use dsize, BTW" )

Once installed to your `PATH` (see [Installation](#installation)), run `dsize` from anywhere:

```bash
# Run in the current directory
dsize

# Or specify a target directory
dsize /path/to/dir
```

### Use as a File ( the lame way to use it. lol! )

No installation needed — use the script directly from your filesystem:

```bash
# Run in the current directory (source keeps it in the current shell)
source ./dsize

# Or specify a target directory
bash ./dsize /path/to/dir

# Or execute directly (if chmod +x)
./dsize /path/to/dir
```

## Output

The script produces a formatted table like this:

<img width="498" height="746" alt="dsize-example" src="https://github.com/user-attachments/assets/420bb790-57ab-49c8-bb02-814da5b16de4" />

```text
-------------------------------
| Size       | Folder         |
-------------------------------
|  0.5M      | .              | <-- this shows the total of files only
| 12.3M      | build          |
|  4.7M      | src            |
|  2.1M      | tests          |
| 19.6M      | Total:         |
-------------------------------

----------------------
| Mount: /dev/sda1   |
----------------------
|  256.4G | Total    |
|  128.2G | Used     |
|  128.2G | Avail    |
----------------------
```

## Color Thresholds

| Size    | Color  | ANSI Code  |
| ------- | ------ | ---------- |
| < 1 MB  | Grey   | `38;5;244` |
| ≥ 1 MB  | Green  | `38;5;40`  |
| ≥ 1 GB  | Yellow | `38;5;220` |
| ≥ 5 GB  | Orange | `38;5;208` |
| ≥ 10 GB | Red    | `38;5;160` |

### Mount Point Thresholds

| Usage   | Color Behavior                                |
| ------- | --------------------------------------------- |
| ≥ 90 %  | Used → Red, Avail → Grey                      |
| 75–90 % | Avail → Orange                                |
| 50–75 % | Avail → Yellow                                |
| < 50 %  | No coloring — your disk is doing fine, relax. |

## Configuration

A few constants can be tweaked at the top of the script:

| Variable           | Default | Description                                                      |
| ------------------ | ------- | ---------------------------------------------------------------- |
| `PADDING`          | `2`     | Column padding between cells                                     |
| `MAX_FOLDER_WIDTH` | `65`    | (reserved) — we hope you never need this. Your folders are fine. |

## Requirements

- **Bash** ≥ 4.0
- **GNU coreutils**: `du`, `stat`, `find`, `df`, `awk`, `sed`

## How It Works

1. **Root directory** — direct file sizes are collected via `find . -maxdepth 1 -type f -exec stat ...`.
2. **Subdirectories** — a single `du -h --max-depth=1` pass gathers recursive sizes.
3. **Deduplication** — each path is resolved to its real device:inode pair, skipping duplicates (useful for bind mounts or symlink aliases).
4. **Mount info** — `df -B1 .` provides total/used/available bytes for the current filesystem.
5. **Colorization** — sizes are mapped to ANSI 256-color codes based on magnitude thresholds.
6. **Table formatting** — column widths are computed dynamically, then output is written to a temp file before display (cleaned up via `trap` on exit).

## License

Public domain — do what you want with it. We're not your boss.
_(Please don't break it, though.)_
