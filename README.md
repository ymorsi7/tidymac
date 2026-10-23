# tidymac

**Find the disk space macOS's Storage settings can't explain, and get it back safely.**

macOS lumps most of a developer's disk usage into a gray "System Data" bar and then
suggests you delete your photos. The real culprits are usually invisible to it:

- **Docker's virtual disk** keeps growing as you rebuild images and never shrinks on
  its own. One Mac had 87 GB inside Docker when the images you'd actually recognize
  added up to 5 GB.
- **Local AI models** (Ollama, LM Studio) are 4–40 GB each and are often forgotten
  after a weekend of experimenting.
- **ML dataset and model caches** in `~/.cache/huggingface`, `~/.cache/torch` and
  `~/.cache/whisper`. A single HuggingFace dataset can be 40 GB.
- **Package-manager caches** from pip, npm, Yarn, Homebrew, Conda, Cargo, Gradle and Go.
- **Stale copies of Chrome.** Chrome keeps a hidden 1.4 GB copy of itself for every
  update and doesn't always delete the old ones.
- **Developer leftovers**: Xcode DerivedData, Playwright browsers, simulator runtimes,
  and finished app-update downloads.
- **Project build folders**: `node_modules`, `.next`, Python virtualenvs and Rust
  `target` folders spread across your projects.

On the Mac it was built on, tidymac took free space from **2.7 GB to 218 GB** without
touching a single personal file.

## Install

It's a single Python script with no dependencies, and it runs on the Python that
ships with macOS.

```sh
git clone https://github.com/ymorsi7/tidymac.git
ln -s "$PWD/tidymac/tidymac" /usr/local/bin/tidymac   # or anywhere on your PATH
```

## Use

```sh
tidymac              # scan and show what's taking space (deletes nothing)
tidymac clean        # scan, then pick what to delete
```

A scan takes a few seconds and prints a numbered list grouped by category:

```
Docker  (81.2 GB)
    1  safe      79.4 GB  Untagged Docker images (68)
                          leftover layers from rebuilds; no container uses them
    2  review     2.1 GB  Stopped Docker containers (4)
                          anything written inside these containers is lost

AI models  (73.0 GB)
    3  review    42.0 GB  Ollama model llama3.3:70b
                          downloaded 20 months ago; `ollama pull llama3.3:70b` restores it
```

When `tidymac clean` asks what should go, you can answer with:

- item numbers, like `1,3,5-8`
- `safe` to pick every item marked safe
- `all` to pick everything
- Enter to cancel

It shows what you picked and asks once more before deleting anything.

| Option | What it does |
|---|---|
| `--dry-run` | Show exactly what would be deleted or run, without doing it |
| `--select SPEC` | Choose items without the prompt, e.g. `tidymac clean --select safe` |
| `-y`, `--yes` | Skip the final confirmation |
| `--min-size 500MB` | Hide items smaller than this (default 100MB) |
| `--roots DIR ...` | Folders to search for project build output |
| `--no-projects` | Skip the project-folder search |
| `--json` | Machine-readable output |

## Safety rules

- **Videos are never deleted.** Before removing anything, tidymac checks it for video
  files (`.mp4`, `.mov`, `.mkv` and more), including inside folders and zip files. If
  it finds one, it skips that item.
- `~/Movies`, `~/Pictures` and `~/Music` are never scanned or touched, and neither is
  your home folder itself.
- **safe** means a pure cache or leftover that rebuilds itself automatically. Selecting
  it loses nothing except some re-download time.
- **review** means something you might want to keep, such as a downloaded model, a
  project's `node_modules` or a simulator. Each one says how to get it back.
- Whenever a tool has its own cleanup command (`docker image prune`, `ollama rm`,
  `brew cleanup`, `conda clean`, `xcrun simctl`), tidymac uses it instead of deleting
  files directly.
- After Docker cleanup on [Colima](https://github.com/abiosoft/colima), it trims the
  VM's disk so macOS actually gets the space back.

## Requirements

macOS with Python 3.9 or newer. Every tool tidymac checks (Docker, Ollama, Homebrew,
Xcode and others) is optional; it skips whatever you don't have installed.

## License

MIT
