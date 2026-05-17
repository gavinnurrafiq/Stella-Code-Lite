# StellaCode Lite

A low-resource Python code editor for machines **without a GPU** and with
modest CPUs. Forked from StellaCode with the video background, alpha
compositing, and other GPU-heavy features stripped out, replaced with
solid colors, software rendering, and longer debounces.

Built with PyQt6. No web engine. No autocomplete daemons. No language
server. Just an editor that opens, runs, and lints Python.

---

## Features

- **Multi-tab editor** — open many files at once, drag to reorder,
  middle-click or `Ctrl+W` to close.
- **Inline tab rename** — double-click a tab to rename it (display label
  only, the file on disk is untouched).
- **Python syntax highlighting** — regex-based, single-pass, can be
  disabled on very weak hardware.
- **Live linting** — `pyflakes` + `ast` running in a background thread,
  debounced at 1.2s. Errors appear as a tinted line plus wavy underline.
- **Run / Stop** — runs the active tab's file via `QProcess`, captures
  stdout/stderr live, color-coded.
- **Format with Black** — optional. The button gracefully says
  "not installed" if `black` isn't available.
- **No GPU required** — solid colors only, raster paint engine forced,
  OpenGL paths disabled before `QApplication` is created.
- **Small footprint** — ~60 MB resident memory on Windows.

---

## Requirements

- **Python 3.10+**
- **PyQt6** (required)
- `pyflakes` (optional — without it, only syntax errors are reported)
- `black` (optional — without it, the Format button is a no-op)

Install everything:

```powershell
pip install PyQt6 pyflakes black
```

Or just the minimum:

```powershell
pip install PyQt6
```

---

## Running from source

```powershell
python "C:\Users\Admin\Documents\Programs\Stella Code Lite\app.py"
```

Or open a file directly:

```powershell
python app.py path\to\some_script.py
```

---

## Building the EXE

A prebuilt EXE lives in `dist\StellaCode Lite\`. To rebuild:

```powershell
pip install pyinstaller
cd "C:\Users\Admin\Documents\Programs\Stella Code Lite"
python -m PyInstaller --noconfirm --windowed ^
    --name "StellaCode Lite" ^
    --icon "icon.ico" ^
    --add-data "style.qss;." ^
    --add-data "icon.ico;." ^
    app.py
```

The output is a **one-folder build** under `dist\StellaCode Lite\`. Ship
the entire folder, not just the .exe — PyQt6 needs its bundled DLLs and
plugin directories.

### Why one-folder, not one-file?

One-file PyInstaller builds repack everything into a single 60–100 MB
.exe, then unpack it to `%TEMP%` on every launch. On the kind of low-end
hardware this app targets, that disk extraction adds noticeable startup
delay. One-folder starts in well under a second.

If you really want a single .exe (e.g. for portable distribution on a
USB stick), swap `--windowed` for `--onefile --windowed`. Expect ~2 s
slower startup.

### Distributing

Zip `dist\StellaCode Lite\` and ship the zip. Recipients extract and
run `StellaCode Lite.exe` — no install needed, no Python on the target
machine required.

---

## Keyboard shortcuts

| Action               | Shortcut       |
| -------------------- | -------------- |
| New tab              | `Ctrl+N`       |
| Open file            | `Ctrl+O`       |
| Save                 | `Ctrl+S`       |
| Save As              | `Ctrl+Shift+S` |
| Close tab            | `Ctrl+W`       |
| Rename current tab   | `F2`           |
| Quit                 | `Ctrl+Q`       |
| Undo / Redo          | `Ctrl+Z` / `Ctrl+Y` |
| Cut / Copy / Paste   | `Ctrl+X` / `Ctrl+C` / `Ctrl+V` |
| Run                  | `F5`           |
| Stop run             | `Shift+F5`     |
| Lint now             | `F7`           |
| Format with Black    | `Ctrl+Alt+F`   |

---

## Tab UX

- **Add tab:** click the `+` button at the top-right, or `Ctrl+N`.
- **Switch tab:** single click. Drag to reorder.
- **Close tab:** click the `×` on the tab, or `Ctrl+W`. Modified tabs
  ask before discarding.
- **Rename tab:** **double-click** the tab label. An input appears in
  place — press `Enter` to commit, `Esc` to cancel. Or press `F2` to
  rename the current tab.

Renaming changes the **display label only**. The file on disk keeps its
original name. Use **Save As** if you want to rename the actual file.

If you `Open` a file that's already open in another tab, that tab is
focused instead of opening a duplicate.

---

## Tweaks for very weak hardware

Two environment variables turn off the heaviest optional work:

```powershell
$env:HIGHLIGHT_OFF = "1"   # disable syntax highlighting
$env:LINT_OFF      = "1"   # disable background linting
& ".\dist\StellaCode Lite\StellaCode Lite.exe"
```

With both off, the editor is essentially a colored text box with line
numbers — virtually zero CPU when idle. You can still trigger a one-shot
lint with `F7` even when `LINT_OFF=1`.

---

## What got cut vs the full StellaCode

| Feature              | Full | Lite               |
| -------------------- | ---- | ------------------ |
| Video background     | Yes  | **Removed**        |
| Translucent panels   | Yes  | Solid colors       |
| Multi-tab editing    | No   | **Added**          |
| Tab rename           | No   | **Added**          |
| Lint debounce        | 500ms | 1200ms            |
| Highlighter passes   | 5+ regex sweeps | 4 (combined) |
| OpenGL paths         | Default | Forced software |
| Default window size  | 1200×780 | 900×600        |
| Output buffer        | Unbounded | 2000 lines    |

The file ops, run/stop pipeline, linter, formatter, and editor key
bindings are otherwise identical.

---

## File layout

```
Stella Code Lite/
├── app.py            # entry point (sets env vars, applies stylesheet)
├── __main__.py       # for `python -m` invocation
├── window.py         # MainWindow + LintWorker + tab orchestration
├── tabs.py           # EditorTabs + EditorTabBar + inline rename
├── editor.py         # CodeEditor + LitePythonHighlighter
├── linter.py         # ast + pyflakes wrapper
├── style.qss         # red dark theme (solid colors, no alpha)
├── icon.ico
├── requirements.txt
└── dist/             # PyInstaller output (after build)
```

---

## License

StellaCode Lite is **free of charge but not open source**. See
[LICENSE.txt](LICENSE.txt) for the full terms. Short version:

| You may                                       | You may NOT                                          |
| --------------------------------------------- | ---------------------------------------------------- |
| Use it for personal projects, study, evaluation | Use it commercially (paid work, business, products) |
| Install on as many of your own devices as you like | Redistribute the program or source, modified or not |
| Read the source files to understand the code  | Publish the source on GitHub, mirrors, package registries |
| Make private modifications for your own use   | Sell, sublicense, or rent it                         |

The source is visible because Python ships as source — that visibility
does **not** make it open source. If you want commercial use, contact
the copyright holder to negotiate a commercial license.

Replace `StellaCode Lite Authors` in `LICENSE.txt` with your real name
or company before distributing.

---

## Troubleshooting

**The EXE opens then closes immediately.**
Run it from a terminal to see the error:
```powershell
& ".\dist\StellaCode Lite\StellaCode Lite.exe"
```
Most common cause: missing `style.qss` or `icon.ico` next to the EXE.
PyInstaller bundles them, but if you copied the EXE alone they'll be
absent. Ship the whole `dist\StellaCode Lite\` folder.

**Tab close button is invisible.**
This is a known quirk with custom QSS on some Qt themes. The button is
still clickable in the same spot. To restore the default look, remove
the `QTabBar::close-button` rule from `style.qss`.

**Linter is slow on a huge file.**
Either bump `LINT_DEBOUNCE_MS` in `window.py:39`, or set
`LINT_OFF=1` and hit `F7` manually when you want a check.
