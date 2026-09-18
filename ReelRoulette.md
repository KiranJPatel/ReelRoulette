# ReelRoulette - Random Snippet Player — README

ReelRoulette is a single-file, offline HTML app that turns any folder of local videos into an endless stream of random movie snippets. It scans a folder you choose, picks a random video, jumps to a random point inside it, plays a random-length clip, and then moves on to the next one — like channel-surfing through your own library.

Nothing is uploaded. All file access happens locally in your browser.

Turn any folder into your own endless movie channel.
> Stop scrolling. Start surfing.
> Point it at a folder. It scans every video, picks a random one, and drops you into a random scene.
> You get a fresh snippet every time — 30 seconds, 3 minutes, your call.
> No playlists. No repeats. No endless "what should I watch?"
> Crossfade between clips like a late-night film channel.
> Tap into Mini mode for a story-style, one-thumb experience.
> Fullscreen, Picture-in-Picture, keyboard shortcuts — everything you'd expect.
> 100% local. Your files never leave your device.
> One HTML file. Zero install. Infinite snippets. Press play and let the library surprise you.

---

## 1. Requirements

| Requirement | Detail |
|---|---|
| **Browser** | Chrome or Edge (desktop). Requires the **File System Access API** (`showDirectoryPicker`). |
| **Firefox / Safari** | Not supported — you'll get an alert telling you to use Chrome or Edge. |
| **Internet** | Not needed. The app is fully offline. |
| **Install** | None. Just open `RandomSnippetPlayer-BoltV4.html`. |
| **File formats** | Whatever your browser can decode. Default list: `mp4, mkv, webm, mov, m4v, avi, wmv, flv, ts, m2ts, mts, mpg, mpeg, vob, ogv, 3gp, asf`. In practice, MP4/H.264, WebM and some MOV files work best — MKV/AVI/WMV support is spotty because it depends on the browser's built-in codecs. |

---

## 2. Quick Start

1. Open the HTML file in Chrome or Edge.
2. Click **📁 Select Folder** (in the header) or **📁 Choose Video Folder** (in the empty-state panel).
3. Pick a folder containing videos. The app scans it **recursively**, including subfolders.
4. Once the scan finishes, the player starts automatically (or press ▶ / `Space`, or enable **Start automatically after scan**).
5. Sit back. Snippets play back-to-back. Press `N` to skip to the next one at any time.

The browser will ask for read permission on the folder. The app requests `mode: "read"` only — it can never modify your files.

---

## 3. Interface Tour
### Header
- **📁 Select Folder** — choose / re-choose the video folder.
- **🌙 / ☀** — toggle dark / light theme (saved to `localStorage`).
- **⚙** — open the Settings modal.

### Player area
- **Empty state** — shown until a folder with videos is loaded.
- **Loading spinner** — appears while scanning a folder.
- **Transition overlay** — black fade used for crossfade switching.
- **Overlay bar** (bottom of video): play/pause button, seek slider, `current / total` time.
- **Fullscreen actions** (right edge, only in fullscreen): a vertical stack of ⏸/▶, ⏭ (accent-colored), and ✕ to exit.
- **Story overlays** (only in Mini mode): progress bars at top, tap zones, center flash icon, heart burst, and a gradient title bar at the bottom.

### Controls bar (below the video)
`▶` Play/Pause · `⏭` Next · `🔊` Mute · volume slider · **Mini** (story mode) · **PiP** · `⛶` Fullscreen.

### Info bar
Current filename, the chosen snippet duration, and the time remaining in the current snippet (counts down live).

### Sidebar
- **Playback** — snippet length range, start-position range, and behavior toggles.
- **Randomization** — how the shuffler avoids repeats.
- **Library** — folder name, file count, cycle progress, status badge, and a **pool bar** (dots showing which videos have been played this cycle; the current one is highlighted).
- **Recently Played** — last 15 snippets with their start offset and duration.

---

## 4. How It Works

1. **Scan** — `showDirectoryPicker()` returns a directory handle. The app walks it recursively via `entries()` and collects every file whose extension matches your allowed list. The `File` objects are obtained lazily via `handle.getFile()` when a video is actually played, so nothing is read into memory up front.
2. **Pick** — a candidate pool is built from all files, minus recently-played ones (last *N*), minus already-played-this-cycle ones (if "Exhaust library before repeating" is on). One is chosen at random. If the pool empties, the cycle resets.
3. **Load** — the file is turned into a Blob URL (`URL.createObjectURL`), assigned to the `<video>` element, and its metadata is awaited. The previous URL is revoked to free memory.
4. **Snippet math**:
   - `desired` = random value between Min and Max seconds (or Max if randomizing is off).
   - If the video is shorter than Min and **Skip videos shorter than minimum** is on, it's skipped.
   - `desired` is clamped to `duration − 0.5s`.
   - `maxStart = duration − desired`, and the start point is `rand(startMin%, startMax%) × maxStart`.
5. **Play** — `currentTime` is set to the start, a `timeupdate` listener watches for `currentTime >= snippetEnd`, and then the next video is triggered (if **Automatically play next** is on; otherwise it pauses).
6. **Crossfade** — a black overlay fades in before the source swaps and fades out after, hiding the load flicker.
7. **Failure handling** — if a file fails to load or reports no duration (typical for unsupported codecs), it's skipped automatically after ~400 ms. After 5 consecutive failures the status badge warns about codec issues.

---

## 5. Settings Reference

### Playback card

| Setting | Default | What it does |
|---|---|---|
| **Minimum (seconds)** | 30 | Lower bound of the snippet duration. Also the minimum video length considered when skipping short videos. |
| **Maximum (seconds)** | 180 | Upper bound of the snippet duration. |
| **Start range** slider | 5–85% | Visual indicator of the start-position window. The actual values are edited in the ⚙ Settings modal. |
| **Randomize snippet duration** | On | Pick a random length between Min and Max. Off = always use Max. |
| **Automatically play next** | On | Chain snippets without intervention. Off = pause at the end of each snippet. |
| **Start automatically after scan** | Off | Begin playing as soon as folder scanning completes. |

### Randomization card

| Setting | Default | What it does |
|---|---|---|
| **Avoid last N videos** | 5 | Recently played files are excluded from the candidate pool. Also controls how many entries are kept in the played-history used for pool-bar highlighting. |
| **Exhaust library before repeating** | On | Tracks a "cycle" set of played file paths. Nothing repeats until every file has been played once. |
| **Skip videos shorter than minimum** | On | Ignores files shorter than the Min seconds value. |
| **Crossfade transitions** | On | Fades to black between snippets instead of hard-cutting. |

### Library card

| Field | Meaning |
|---|---|
| **Folder** | Name of the selected directory. |
| **Videos found** | Total supported files discovered (recursive). |
| **Played this cycle** | How many unique files are in the current exhaustion cycle. |
| **Status** | `Ready` / `Playing` / `Scanning…` / `No videos` / `Skipping unsupported file` / `Codec issues`. |
| **Pool bar** | Up to 40 dots. Green = played this cycle, blue = currently playing. |
| **↻ Rescan Folder** | Re-walks the same folder handle to pick up added/removed files and resets the cycle. |

### Settings modal (⚙)

| Field | Default | Notes |
|---|---|---|
| **Start min (%)** | 5 | Lower bound of the random start position, as a % of the available range. |
| **Start max (%)** | 85 | Upper bound. 85% means snippets rarely start in the last 15% of a movie. |
| **Video extensions** | see list | Comma-separated whitelist used during scanning. Edit to add exotic formats or to speed up scanning. |

All settings persist in `localStorage` under the key `randomSnippetPlayerV2`, along with volume, theme, and Mini-mode state.

---

## 6. Keyboard Shortcuts

| Key | Action |
|---|---|
| `Space` | Play / Pause |
| `N` | Next snippet |
| `F` | Toggle fullscreen |
| `M` | Toggle mute |
| `T` | Toggle theme |
| `S` | Toggle Mini (story) mode |
| `Esc` | Close the Settings modal / exit fullscreen |

Shortcuts are ignored while focus is inside an input, select, or textarea.

---

## 7. Mini (Story) Mode

Toggled with the **Mini** button or `S`. It restyles the player like a social-media story viewer:

- **Progress bars** at the top — one segment per snippet, up to 12 retained, filling left-to-right as the snippet plays.
- **Tap zones** — tap the left half to restart the current snippet from its start point; tap the right half to jump to the next video. A single tap is delayed ~250 ms so double-taps can be detected.
- **Double-tap anywhere** — spawns an animated ❤ at the tap position.
- **Center flash** — a ⏮ or ⏭ icon pulses briefly on tap.
- **Bottom gradient bar** — shows the filename, current position, and time left in the snippet.
- The standard overlay bar is hidden in this mode.

The Mini-mode state is saved and restored on reload.

---

## 8. Fullscreen & Picture-in-Picture

- **Fullscreen** (`F`, `⛶`, or double-click the video) puts the whole player shell into fullscreen so overlays stay visible.
- A vertical stack of quick actions appears on the right: play/pause, next (accent-colored), and exit.
- The cursor and controls auto-hide after 3 seconds of no mouse movement in fullscreen, and reappear on move.
- **PiP** (button only) uses the browser's native Picture-in-Picture. Note that the snippet timer keeps running in PiP, so auto-advance still works.

---

## 9. Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| Alert: *"This browser does not support the required local-folder API."* | You're on Firefox or Safari. Use Chrome or Edge. |
| "No supported videos found" | Your files' extensions aren't in the whitelist. Add them in ⚙ → *Video extensions*. |
| A file is skipped instantly | The browser can't decode that container/codec (common with MKV, AVI, WMV, and some VOB/TS files). Convert to MP4/H.264 or WebM. |
| "Codec issues — skipping" after several failures | Multiple unplayable files in a row. Same fix as above. |
| Videos won't play at all | Some browsers block autoplay with sound. Click the video once to grant playback, or lower/keep the volume — the app already attempts `play()` silently and ignores rejection. |
| Folder permission seems revoked after reopening | The File System Access API requires re-granting permission per session. Re-select the folder. |
| Start-position slider doesn't change anything | That slider is a display of the range; edit the actual values in ⚙ Settings → *Start min/max (%)*. |
| Rescanning finds no new files | Some browsers cache directory entries; re-select the folder if a rescan doesn't pick up changes. |

---

## 10. Privacy & Limitations

**Privacy**
- 100% client-side. No network requests, no analytics, no uploads.
- Only read permission is requested on the chosen folder.
- Settings live in your browser's `localStorage`; clearing site data resets everything.

**Limitations**
- Folder handles don't persist across reloads — you must re-select the folder each session.
- Only the formats your browser can natively decode will play.
- Seeking is unavailable for some containers until enough data is buffered.
- The history/pool visualization caps at 40 dots and 15 history rows for performance.
- Large folders (tens of thousands of files) may take a while to scan, since traversal is recursive and single-threaded.

---

## 11. File Overview

Everything — markup, styles, and logic — lives in `RandomSnippetPlayer-BoltV4.html`. Key script sections:

| Section | Responsibility |
|---|---|
| **STATE / SETTINGS** | Defaults, `localStorage` load & save. |
| **THEME** | Dark/light switching via `data-theme`. |
| **HELPERS** | `toast`, `setStatus`, `fmt`, `rand`, range label. |
| **FOLDER SCANNING** | `scanDir`, `chooseFolder`. |
| **VIDEO SELECTION** | `eligible`, `pick`. |
| **LOAD & PLAY** | `loadAndPlay`, `nextVideo` — the core loop. |
| **UI UPDATES** | `updateUI`, `renderHistory`, `renderPoolBar`. |
| **PLAYBACK CONTROLS** | `togglePlay`, `toggleMute`, `pip`, `fullscreen`. |
| **STORY MODE** | Story bars, tap zones, flashes, hearts. |
| **EVENT BINDINGS** | All listeners and keyboard shortcuts. |

---

*Local-only. Your files never leave your device.*