# ReelRoulette — Random Movie Snippet Player

A single-file, offline HTML app that turns any local folder of videos into an
endless stream of random movie snippets — Reels/TikTok-style browsing, but
100% local. Nothing is ever uploaded; every file you play stays on your device.

This build implements the Reels-user enhancement roadmap through **v2.4**,
plus a further round covering theming, full responsive support, and several
of the roadmap's "additional ideas" (monetization and true live
speech-to-text remain excluded — see [Section 3](#3-development-path)).

---

## 1. Requirements & supported devices

- **Browser:** Chrome or Edge. The app relies on the **File System Access
  API** (`showDirectoryPicker`) and the **Web Audio API**, which only those
  two (plus other Chromium browsers) fully support.
- **Laptop / desktop:** full two-column layout (player + sidebar).
- **Tablets** (iPad, Galaxy Tab, Surface, etc.): the sidebar reflows under
  the player in a responsive grid; touch targets are sized for fingers, not
  just a mouse.
- **Phones**, including large modern ones like the **Galaxy S23/S24/S25
  Ultra** and **iPhone 16 and newer**: a horizontally-scrollable, icon-only
  control bar and overlay bar so nothing is ever cut off, ≥44px touch
  targets, safe-area padding around the iPhone Dynamic Island/notch and
  Android gesture bar, and a landscape mode that prioritizes the video.
- ⚠️ **Important iPhone/iPad caveat:** Apple requires *every* browser on iOS
  and iPadOS — including Chrome and Edge — to use Apple's WebKit engine
  under the hood. WebKit does not implement the File System Access API, so
  on an iPhone or iPad, "Chrome" and "Edge" behave like Safari for this
  app: folder selection automatically falls back to a plain file picker
  (still works fine for a session), but **folder auto-reconnect, drag-and-
  drop of directories, and clip persistence across reloads are
  Chromium-desktop/Android-only** — a platform limitation, not a bug in
  this app. Mix Jams, the audio mixer, filters, gestures, and everything
  else work normally on iOS.
- No install, no build step, no server required — just open the `.html`
  file in your browser. No internet connection is ever required; demo
  clips are generated on-device with `<canvas>` + `MediaRecorder`.

---

## 2. Feature list

### Core playback (original)
- Point the app at a local folder; it recursively scans for video files.
- Plays a random-length snippet from a random file at a random start point,
  then jumps to the next one automatically.
- Adjustable snippet-length range, start-position range, avoid-repeats
  window, and "exhaust library before repeating."
- Fullscreen, mute, Mini (story) mode.

### v2.0 — Feel like Reels
| Feature | Details |
|---|---|
| Gestures | Swipe up/down (or `↓` / `↑`) to jump next/previous; double-tap anywhere to like (heart burst); long-press opens a quick-action sheet. Works in normal mode, not just Mini mode. |
| Playback speed | Pill in the corner cycles `0.5× → 0.75× → 1× → 1.25× → 1.5× → 2×` on tap; long-press opens a fine-grained slider (0.25×–3×). Pitch-preservation toggle included. |
| Always-on progress bars | Segmented story-style bars for the last 10 snippets, visible in normal mode, not just Mini mode. |
| Transitions | Cut, Fade-through-black, Crossfade, Slide up, Zoom punch — selectable in Settings. |
| Thumbnail history | "Recently Played" is a 3-column thumbnail grid with duration badges, not a text list. |

### v2.1 — Creative tools & lower drop-off
| Feature | Details |
|---|---|
| Filter carousel | 8 one-tap presets (Vintage, Noir, Vivid, Faded, Cyberpunk, Warm, Cold, Dream) plus an intensity slider. |
| Overlay FX | Toggleable film grain, vignette, scanlines, letterbox bars, light leaks, old-projector flicker. |
| Save / Loop / Replay | 🔖 bookmarks the current snippet to a persistent Clips grid; 🔁 loops the snippet in place; ↺ restarts it. |
| Style presets | Bundle a filter + speed + transition into a named "style," and optionally auto-apply your last style to every new snippet. |
| Drag & drop | Drop video/audio files or a whole folder anywhere on the player. |
| Demo clips | "Load demo clips" generates 5 short animated clips locally — no folder, no download, no internet needed. |
| Folder auto-reconnect | The last folder handle is remembered (IndexedDB); revisit and it reconnects with one click. |
| PWA (best-effort) | Installable manifest + service worker — see [Limitations](#4-known-limitations). |

### v2.2 — The signature "mix" feature
| Feature | Details |
|---|---|
| Background audio mixer | Dual-track Web Audio graph: your snippet's original audio vs. a looping background track. |
| Crossfader | Single equal-power slider, Original ↔ Background. |
| Auto-duck | Optional: background volume dips automatically when the original audio is loud. |
| Track management | Add multiple tracks, pick which is active, set fade-in/fade-out durations and loop behavior. |
| Audio file scanning | The folder scanner also indexes `mp3, m4a, aac, ogg, opus, wav, flac` separately from video files. |
| **Audio files play in the main rotation** | Audio files aren't just for the background mixer or Mix Jams — they shuffle in alongside your videos in the normal random-snippet rotation (random-length snippet, random start point, same transitions, same Save/Loop/Replay, same history and clips). While one is playing, the video area shows **album art** instead of a blank frame: real embedded cover art extracted directly from the file (ID3v2 `APIC` for MP3, native `PICTURE` blocks for FLAC, Vorbis comments for OGG) plus the track's title/artist tags when present, over a blurred-art background. Files without readable embedded art (M4A/AAC, WAV, or ones with no tags) fall back to the filename and a generated music-note badge. The active filter/effects still apply to the art for a consistent look, and exporting a snippet works too — it renders the album art into the clip instead of a blank frame. |

### v2.3 — Power-user queueing
| Feature | Details |
|---|---|
| Playlist import | `.m3u` / `.m3u8`, `.pls`, `.json` (`[{path,start,end,title}]`), and `.xspf`. |
| Timestamp ranges | Entries like `movie.mp4#t=120,180` play that exact clip instead of a random snippet. |
| Shuffle vs. Playlist mode | Toggle between the original random behavior and following the imported queue in order. |
| Drag-to-reorder | Reorder the queue directly in the sidebar. |

### v2.4 — New use cases
| Feature | Details |
|---|---|
| Mix Jams mode | A dedicated audio-only DJ mode: canvas frequency-bar visualizer, DJ-style crossfades between tracks in your audio-file pool, transport controls. |
| Captions | Auto-detects a sibling `.srt`/`.vtt` file with the same name as the video and attaches it as a toggleable `CC` track (converts `.srt` to `.vtt` on the fly). You can also load a subtitle file manually. |

### Photo Slideshow with Music (standalone feature)
A completely separate mode from the random-snippet player — point it at a
photo album and a folder of songs, and it turns into a self-playing
slideshow: photos crossfade on a timer while music plays continuously
underneath.

| Feature | Details |
|---|---|
| Two independent folders | Photos and Songs are picked separately from your main video/audio library, via their own folder pickers — this is a dedicated feature, not mixed into the main shuffle. |
| Tabbed setup | A **📷 Photos / 🎵 Songs** tab switcher keeps the two folder pickers, the photo interval, and the shuffle toggle organized in one small panel instead of a wall of controls. |
| Configurable interval | Photos advance every 3/5/8/15/30 seconds (default 5s), each with a subtle Ken Burns zoom and a crossfade into the next. |
| Reuses the Mix Jams audio engine | Rather than a second, duplicate audio system, the music playback (dual-track crossfading between songs) is the exact same engine built for Mix Jams — genuinely shared code, not a lookalike. The two modes cleanly hand the engine back and forth if you switch between them. |
| Familiar controls | Play/Pause, Previous/Next photo, Next song, and Fullscreen, plus the same keyboard shortcuts as the rest of the app (Space, arrow keys, `F`, `Escape`) scoped to the slideshow while it's open. |
| Convenience shortcut | If you've already scanned a folder with audio files for the main player, a "Use my already-scanned audio files" button skips picking a separate Songs folder. |

### Latest phase — theming, responsiveness & the roadmap's "bonus ideas"
| Feature | Details |
|---|---|
| **Theme system** | Four appearance modes — **Auto** (follows your OS/browser light-dark setting live), **Light**, **Dark**, and **AMOLED Black** (true `#000` backgrounds to save battery and look crisp on OLED phone screens) — plus **5 accent colors** (Blue, Purple, Pink, Green, Orange). Set both in Settings, or tap the header's theme icon to cycle appearance modes quickly. |
| **Fully responsive layout** | Reworked breakpoints for laptop, tablet, and phone; safe-area padding so content clears the iPhone Dynamic Island/notch and Android gesture bars; ≥40px touch targets on small screens; a landscape-phone mode that hides chrome and maximizes the video. |
| **Text overlays** | Add a caption/title card (top, center, or bottom) to the current snippet — included when you export a clip, and saved with a clip if you bookmark it. |
| **Remix a saved clip** | From the Clips grid, 🎛 Remix reloads a saved clip and opens the effects panel so you can apply a different filter/speed/audio, then save the result as a new clip. |
| **Sound discovery** | 🎼 Extract audio pulls the current snippet's own audio track into your local sound library (the Audio Mixer's track list) so you can reuse it as a background track on other snippets. |
| **Session recap** | 📊 A running tally of snippets watched, unique videos seen, session time, and your most-watched file — viewable any time from the sidebar or the tools sheet. |
| **Focus / Study mode** | 🧘 Locks snippets to 90 seconds, turns off auto-jumping, and runs a 15/25/45/60-minute countdown — a Pomodoro-style ambient mode. Your normal settings are restored automatically when it ends. |
| **"Up next" preview** | A brief corner banner names the next snippet (or next playlist entry) about a second before it plays, YouTube-end-screen style. |
| **Export / share a clip** | ⬇ Renders up to 15 seconds of the current snippet — with your active filter, letterbox, and text overlay baked in — to a downloadable video file (MP4 where the browser supports it, otherwise WebM), audio included. The output is correctly sized for the source's own aspect ratio (including portrait phone-shot video — no more oversized or distorted exports). |
| **Batch export a highlight reel** | From the Saved Clips card, 🎬 **Batch Export** lets you pick any set of saved clips and combine them into one downloadable video. Since a combined video needs one consistent frame size, each clip is automatically letterboxed into it — a portrait clip next to a landscape one next to an audio clip's album art all play correctly, none of them stretched or cropped to fit. Shows live progress ("Exporting clip 2 of 5…"), skips and warns about any clip whose file isn't available rather than failing the whole batch, and can be cancelled mid-export. |
| **Fullscreen auto-hide** | While playing in fullscreen, the bottom control bar and all floating buttons/pills fade out after 3 seconds of inactivity (matching the cursor auto-hide) and reappear instantly on any mouse/touch movement or tap. Controls always stay visible while paused or while a menu/sheet is open. Toggle this behavior off in Settings if you'd rather controls stayed put. |
| **Show/hide bottom panel** | A Settings toggle to permanently show or hide the bottom control bar (play, seek, loop, save, mix) — independent of fullscreen or auto-hide, for a fully "clean" video view if you prefer navigating by gestures and keyboard alone. |
| **Flicker fix (found the real cause)** | Three of the optional overlay effects (Grain, Scanlines, Light Leaks) were applying `mix-blend-mode` **unconditionally** on layers that sit on top of the video at all times, and two (Grain, Flicker) ran an **infinite CSS animation unconditionally** — all of that kept running in the background even while those effects were switched off and invisible. A browser has to keep re-compositing a blend-mode layer against the live video underneath it every frame, so this was continuous, pointless GPU overhead sitting directly on top of the video for the entire time you watched, in every session — a very plausible source of the periodic hitching. It's now gated behind each effect's actual on/off state, so there's zero extra compositing work unless you've actually turned an effect on. Two smaller contributors from an earlier pass are also fixed: `backdrop-filter` blur was removed from controls that sit on the video continuously (bottom bar, speed pill, Focus pill, fullscreen buttons), and the video reveal at every snippet switch now waits for the browser's `seeked` event instead of firing the instant a seek is requested. |
| **Responsive overhaul, round 2** | The control bar (Play/Next/Mute/Volume/Effects/Mini/Fullscreen) and the bottom overlay bar (Play/Loop/Replay/Save/seek/Captions/⋯/Mix) both have more controls than reliably fit on a phone screen at once. On screens ≤640px wide they're now tightened (smaller gaps/padding, a narrower volume slider, a truncating Mix pill) and made horizontally scrollable as a safety net, so nothing is ever cut off or unreachable — the worst case is a short side-swipe instead of a missing button. The header's "Select Folder" label collapses to just its icon on phones, matching the app title. |
| **Duplicate-button cleanup** | Several rounds of feature additions had left genuinely redundant controls: a mobile-only bottom action bar (Speed/Effects/Audio/Next/More) that duplicated the header, control bar, and mix pill; and a separate long-press "quick sheet" (Play/Loop/Replay/Save/Effects) that duplicated buttons already sitting permanently in the overlay bar. Both are removed. Long-press on the video now opens the same **⋯ tools sheet** used for Text/Extract audio/Export/Focus/Recap — one consistent sheet instead of three overlapping ones. This also caught and fixed a real bug the previous pass introduced: the plain single-tap-to-play/pause handler still referenced the just-removed sheet and would have thrown an error on every tap; it's now fixed and covered by an automated test that clicks every single button in the app and exercises every gesture (swipe/double-tap/long-press) to confirm nothing is left dangling. |
| **Clean fullscreen (new setting, off by default)** | In fullscreen, the title/duration bar and the "up next" preview are now always hidden in both Normal and Mini/Story mode, for an uncluttered view. A new Settings toggle — **"Show title/duration bar & 'up next' preview in fullscreen"**, unchecked by default — lets you bring them back if you'd rather keep them visible; it applies live even while already in fullscreen. |
| **Control bar layout** | Effects, Mini Player, and Fullscreen moved next to Play/Next/Mute on the left, and now show as icons only (🎨 / 📱 / ⛶) instead of icon+text, matching the rest of the control bar. Picture-in-Picture was removed as a low-value control. |

The new **📝 🎼 ⬇ 🧘 📊** tools live behind a single **⋯** button in the
player's bottom bar, next to Captions, so the core overlay bar doesn't get
cluttered.

### Responsive & interaction-design rewrite
This pass applied concrete techniques from three published design skills
([`responsive-design`](https://github.com/wshobson/agents/tree/main/plugins/ui-design/skills/responsive-design),
[`web-component-design`](https://github.com/wshobson/agents/tree/main/plugins/ui-design/skills/web-component-design),
[`interaction-design`](https://github.com/wshobson/agents/tree/main/plugins/ui-design/skills/interaction-design))
rather than more one-off breakpoint patches:

| From | What changed |
|---|---|
| `responsive-design` | **Fluid type & spacing tokens** (`clamp()`-based `--text-*`/`--space-*` variables) so sizing scales smoothly instead of jumping at breakpoints. **Dynamic viewport units** (`dvh` with a `vh` fallback) on the video area so mobile browser address-bar show/hide no longer changes the layout height — one of the skill's explicitly-named "common issues." **Container queries** on every sidebar card (`container-type: inline-size`), so a card's internal layout (two-column fields, the accent-color row, the recap stats) adapts to *its own* rendered width — correct whether that card ends up in the two-column tablet grid, a resized desktop window, or a phone, instead of only reacting to viewport width. **Auto-fit grids** for the Clips and History thumbnails (`repeat(auto-fill, minmax(...))`) replace three hand-tuned breakpoint overrides with one rule that reflows naturally at any width. **44×44px touch targets**, the skill's explicitly-stated minimum (tightened from an earlier 40px pass). |
| `web-component-design` | **Accessible-by-default modals**: Settings, What's New, and Session Recap now use `role="dialog"`/`aria-modal`/`aria-labelledby`, move keyboard focus into the dialog on open, trap Tab navigation inside it, and restore focus to whatever triggered it on close. **Consistent toggle semantics**: Loop, Mute, Mini Player, and Captions now expose `aria-pressed` so assistive tech reports their on/off state correctly, matching the buttons' visual "on" styling. |
| `interaction-design` | **A real timing/easing scale** (`--duration-micro/small/medium`, `--ease-standard/out/spring`) replaces ad hoc transition durations, following the skill's guidance (micro-feedback ~120ms, toggles ~220ms, modal/page-level ~320ms). **Modals and full-screen sheets now animate symmetrically** — they used to snap open with an entrance animation but close instantly; open and close are now a proper fade+scale both ways. **`prefers-reduced-motion` is respected globally** — every animation and transition collapses to effectively instant for anyone who has that OS/browser preference set, per the skill's explicit accessibility best practice. **Visible keyboard focus rings** (`:focus-visible`) were added to all buttons for keyboard users, without adding an outline for mouse clicks. |

---

## 3. Development path

### ✅ Implemented
Everything in Section 2 — the full roadmap through v2.4, plus theming,
responsive design, and six of the roadmap's "additional ideas" (text
overlays, remix, sound discovery, session recap, focus mode, up-next
preview) and one v3.0 item (export/share via MediaRecorder).

### ⏳ Still pending
- **Live auto-captions** — real speech-to-text (today's captions rely on a
  sidecar `.srt`/`.vtt` file, not transcription). This needs a client-side
  model such as Whisper WASM, which would meaningfully bloat a single HTML
  file and was explicitly scoped out in the original roadmap, the same way
  it scoped out WebCodecs speed-ramps.
- **Theater sync** — a WebRTC "watch together" room. This wasn't
  implemented because a trustworthy version needs either a signaling
  server (which breaks the local-first, zero-backend design) or a fragile
  manual copy-paste SDP exchange that's hard to make reliable and harder to
  verify without live cross-browser testing. Flagging it honestly as
  descoped rather than shipping something unverified.

### 🎨 Explicitly descoped (Tier-3 effects)
Canvas/WebGL-heavy effects — real-time LUT color grading, glitch/RGB split,
kaleidoscope, mirror, speed ramps — were left out, matching the enhancement
doc's own scoping of WebCodecs as too heavy for a single HTML file.

### 💰 Out of scope by request
Monetization (Freemium tiers, lifetime unlock, preset marketplace, B2B
licensing, etc.) was excluded per your original instruction.

---

## 4. Known limitations

- **PWA install** only activates when the file is served over `http(s)` or
  `localhost`. Opened directly as `file://`, browsers won't register a
  service worker or offer an install prompt — a browser-level restriction.
- **iOS/iPadOS**: as noted in Section 1, every browser on iOS uses WebKit,
  which doesn't support the File System Access API. Folder selection still
  works via a fallback file picker, but auto-reconnect, folder drag-and-
  drop, and cross-session clip playback need that API and are effectively
  Chromium-desktop/Android-only.
- **Captions** rely on a matching subtitle file next to the video
  (`movie.mp4` + `movie.srt`/`.vtt`). There's no live speech-to-text.
- **Album art extraction** is implemented directly (no external library) for
  MP3 (ID3v2) and FLAC/OGG (native picture blocks and Vorbis comments) —
  the formats that commonly embed cover art. M4A/AAC and WAV files play
  normally in the rotation but fall back to the filename and a generated
  music badge, since their tagging formats weren't worth the added
  complexity for this pass.
- **Exported clips** bake in your active filter, letterbox bars, and text
  overlay, but not the other overlay effects (grain, scanlines, vignette,
  light leaks, flicker) — those stay visual-only for now to keep the
  export pipeline simple and reliable.
- **Export format** depends on the browser: recent Chrome/Edge can record
  directly to MP4; otherwise it falls back to WebM. Both play natively in
  Chrome/Edge and most modern players.
- **Demo clips** are short (7s) procedurally generated animations meant to
  let you try the app instantly — not real movie footage.
- **Saved clips** whose original file came from drag-and-drop (rather than
  the folder picker) may not survive a page reload, since only real
  `FileSystemFileHandle`s (from `showDirectoryPicker`) persist to
  IndexedDB across sessions.

---

## 5. Production-readiness audit (this pass)

Every feature built so far was re-reviewed end to end — not just re-read,
but re-tested with new automated tests targeting exactly the kind of bug
that hides in a media-heavy, long-running app. This pass found and fixed
real issues rather than just confirming things looked fine:

| Area | What was found | Fix |
|---|---|---|
| **Memory leaks** | Switching the background-mixer track, and every song change in Mix Jams/Slideshow, created a new blob URL without ever releasing the previous one — in a session meant to run for hours, this accumulates indefinitely. | Both now track and revoke the specific URL they're replacing. Verified with a test that switches tracks repeatedly and confirms revocation actually happens. |
| **Race conditions** | Rapid-clicking Next/Previous (photos in Slideshow, or tracks in Mix Jams) could let an older, slower request resolve *after* a newer one and clobber it with stale content. | Both now use a request-token guard — only the most recent request is ever allowed to update the screen, matching the pattern already used for album-art loading. Verified with a test that deliberately resolves an old request after a newer one. |
| **Partial-failure resilience** | If Web Audio setup threw partway through construction, it could leave the "is it ready" flag in a falsely-positive state, so a real failure would be silently reported as success. | Setup is now wrapped so any failure cleanly resets to "not ready" and tells the user, instead of pretending to work. |
| **Storage resilience** | Every settings/clip/preset save wrote straight to `localStorage` with no handling if it's full or disabled — a single failed write could break whatever button triggered it. | All writes now go through one guarded helper; if a save genuinely fails (e.g. storage full), the person doing the saving is told so, rather than seeing a false "saved" confirmation. |
| **Silent UX failures** | The Fullscreen button (main player and Slideshow) did nothing with zero feedback if the browser refused the request. | Now shows a toast explaining fullscreen isn't available, instead of silently ignoring the click. |
| **Settings persistence gap** | Slideshow's photo interval and shuffle preference reset on every reload, unlike every other setting in the app. | Now persisted the same way everything else is. |
| **XSS/injection safety** | Audited every place a filename gets written into the page. | Already consistently safe (correctly escaped everywhere) — no changes needed, confirmed rather than assumed. |
| **Accessibility on newer features** | Checked Slideshow's controls against the same standard applied earlier (labels, keyboard shortcuts, reduced-motion). | Already consistent — no changes needed. |

The full existing test suite (every button, every gesture, theming, the
mixer, playlists, Mix Jams, Slideshow, album art, fullscreen behavior) was
re-run after every fix with zero regressions.

**One thing this pass found but didn't change:** double-tapping to "like" a
snippet has only ever shown a heart animation — it doesn't actually
remember anything. That's exactly what the original spec asked for, but it
reads like it should do more. See the first item in Section 8 below.

**Addendum — aspect ratio, verified end to end.** The player's core
aspect-ratio handling (`object-fit: contain` on the video element, correct
in both normal and fullscreen mode) was already sound — confirmed by
tracing it rather than assuming it. But the single-clip **export** path had
a real bug: it capped the output's *width* but not its *height*, so a
portrait phone-shot clip (e.g. 1080×1920) would export at 960×1706 —
correct proportions, but needlessly oversized. Fixed with a proper
bounded-box calculation that caps whichever dimension is actually larger,
verified with dedicated tests covering landscape, portrait, square, and
already-small sources. The same aspect-safe letterboxing logic now also
powers Batch Export below, where it matters even more: combining clips of
different shapes into one video requires it.

## 6. How to use it

### Getting started
1. Open `ReelRoulette.html` in Chrome or Edge (any device from Section 1).
2. Click **📁 Choose Video Folder** and pick a folder of videos — or click
   **🎞 Load demo clips** to try it instantly with no files of your own.
3. Playback starts automatically (or press **Space** / tap the video).

### Everyday controls
| Action | How |
|---|---|
| Play / Pause | Tap the video, or `Space` |
| Next snippet | Swipe up, `↓`, `N`, or the ⏭ button |
| Previous snippet | Swipe down, `↑` |
| Like | Double-tap anywhere on the video |
| Quick actions | Long-press the video (Play/Pause, Loop, Replay, Save, Effects) |
| Loop this snippet | `L`, or the 🔁 button |
| Replay from start | `R`, or the ↺ button |
| Save a clip | `B`, or the 🔖 button |
| Mute | `M` |
| Fullscreen | `F` |
| Cycle theme | `T`, or the header's theme icon |
| Mini / story mode | `S` |

### Appearance
Open **⚙ Settings** to pick:
- **Theme:** Auto (matches your device), Light, Dark, or AMOLED Black.
- **Accent color:** Blue, Purple, Pink, Green, or Orange.
- **Show bottom control bar:** uncheck to permanently hide the bottom bar
  (play/seek/loop/save/mix) for a cleaner, gesture-and-keyboard-only view.
- **Auto-hide controls after 3s in fullscreen:** on by default — the
  bottom bar and floating buttons fade out after 3 seconds of inactivity
  while playing in fullscreen, and reappear on any movement or tap.
  Controls never hide while the video is paused or a menu is open. Uncheck
  this if you'd rather they stayed on screen.

Or just tap the header's theme icon (🖥/☀/🌙/⚫) to cycle through the four
appearance modes without opening Settings.

### Using effects
1. Click **🎨 Effects** in the control bar to open the filter carousel.
2. Tap a preset to apply it; tap it again to remove it. Use the intensity
   slider to control strength.
3. Toggle overlay effects (grain, vignette, scanlines, etc.) as chips below
   the carousel.
4. Like a look? Click **🎨 Save current style** in the sidebar to bundle
   your current filter + speed + transition, and optionally turn on
   **Auto-apply last style to next snippet**.

### More tools (text, sound discovery, export, focus, recap)
Tap the **⋯** button in the video's bottom bar to open a small tools sheet:
- **📝 Text** — type a caption and choose top/center/bottom placement.
- **🎼 Extract audio** — pull the current snippet's audio into your Audio
  Mixer track list for reuse elsewhere.
- **⬇ Export clip** — download up to 15 seconds of the current snippet as
  a video file, filter/text baked in.
- **🧘 Focus mode** — start/stop a Pomodoro-style ambient session (also in
  the sidebar, where you can pick the session length).
- **📊 Session recap** — see how much you've watched this session (also in
  the sidebar).

### Remixing a saved clip
In the sidebar's **Saved Clips** grid, tap **🎛** on any clip to reload it
with the effects panel open — adjust the look, then tap 🔖 again to save
your remix as a new clip alongside the original.

### Batch-exporting a highlight reel
1. Save a few clips first (🔖 on snippets you like).
2. In the **Saved Clips** card, click **🎬 Batch Export Clips**.
3. Tap clips in the list to include/exclude them (all are selected by
   default), or use **Select all** / **Select none**.
4. Click **Export selected**. You'll see live progress as each clip is
   rendered in turn; when it's done, one combined video downloads
   automatically. Clips of different shapes (portrait phone video,
   landscape movie clips, audio clips with album art) are all letterboxed
   into the same frame size so nothing gets stretched or cropped.

### Mixing in background audio
1. Click the **♪ Original** pill on the video (or **🎚 Open Audio Mixer**
   in the sidebar).
2. Click **＋ Add audio file(s)** and pick a track — it starts playing and
   is applied to every snippet.
3. Drag the crossfader between "Original" and "Background."
4. Turn on **Auto-duck** if you want the background to automatically get
   quieter over loud original audio.

### Using playlists instead of shuffle
1. In the sidebar's **Playlist** card, click **＋ Import playlist** and
   select an `.m3u`, `.m3u8`, `.pls`, `.json`, or `.xspf` file.
2. The app switches to **📃 Playlist** mode automatically. Drag items to
   reorder, or click **🔀 Shuffle** to go back to random snippets.
3. Playlist entries can include a range (`file.mp4#t=120,180`) to play an
   exact clip instead of a random one.

### Mix Jams (audio-only mode)
Click the **🎧** icon in the header to enter a full-screen, audio-only DJ
mode that plays through your folder's audio files with a visualizer and
DJ-style crossfades. Click **✕** to exit back to video mode.

### Photo Slideshow with Music
1. Click the **🖼** icon in the header. This is a separate feature from
   the main player — it doesn't touch your video/audio shuffle.
2. On the **📷 Photos** tab, click **Choose Photo Folder** and pick an
   album; set how often photos should change (default 5 seconds) and
   whether to shuffle their order.
3. Switch to the **🎵 Songs** tab and either **Choose Songs Folder**, or —
   if you've already loaded a folder with audio files for the main
   player — click **Use my already-scanned audio files**.
4. Click **▶ Start Slideshow**. Photos crossfade automatically while music
   plays continuously in the background.
5. Controls: ⏮/⏭ jump photos, the center button plays/pauses everything
   (photos and music together), 🎵⏭ skips to the next song, and ⛶ goes
   fullscreen. Keyboard: `Space` to play/pause, arrow keys for
   previous/next photo, `F` for fullscreen, `Escape` to exit.

### Captions
If a video has a matching `.srt` or `.vtt` file in the same folder, the
**CC** button lights up automatically — click it to toggle captions on/off.
If none is found, clicking **CC** lets you pick a subtitle file manually.

---

## 7. Files

- `ReelRoulette.html` — the entire app (HTML/CSS/JS, no external
  dependencies, no build step).

All settings, saved clips, style presets, session recap stats, and
folder/track handles are stored locally in your browser (`localStorage` +
`IndexedDB`) — clearing your browser data for this file will reset them.

---

## 8. Feature ideas for making the tool more effective

Ideas below, not implemented. Roughly ordered by value for effort. None of
these overlap with what's already flagged pending in Section 3 (live
speech-to-text, WebRTC theater sync) or explicitly out of scope
(monetization, Tier-3 canvas effects).

### High value, moderate effort
1. **Make "like" mean something.** Right now double-tapping shows a heart
   and nothing else — the animation is real, the memory isn't. Turning it
   into an actual lightweight favorites signal would let it bias future
   shuffling toward liked files and away from ones you always skip, plus
   give you a "Liked" filter alongside the existing Clips grid. This is
   the single most natural next feature, since the gesture and the UI
   affordance already exist — only the persistence and weighting are
   missing.
2. **Library search.** With a folder of hundreds of files, there's
   currently no way to jump to a specific one — you can only wait for
   shuffle luck or scroll history. A simple filter box over the file list
   (and a "play this now" action) would close a real gap for larger
   libraries.
3. **Tags/collections.** Let people label snippets or whole files
   ("workout," "chill," "study") and shuffle within a tag instead of the
   whole library. Pairs naturally with the favorites idea above — both
   are about curating a big library down to what you actually want right
   now.
4. **Duplicate-file detection.** Many real libraries have the same file
   copied into more than one folder. A simple size+name (or partial hash)
   check during scanning could flag or auto-skip duplicates so they don't
   eat into the "avoid repeats" pool artificially.

### High value, higher effort
5. **Library Insights dashboard.** Session Recap already tracks
   snippets/unique-files/time for the *current* session; keeping that
   history across sessions (total hours watched, most-played files
   over time, a simple usage calendar) would turn a nice-to-have into a
   genuinely useful long-term view of a library.
6. **Optional "continue where I left off."** When a long file gets
   picked again later, remember roughly where you last stopped in *that
   file* and bias the next random start point nearby, instead of fully
   re-randomizing — useful for long lectures, podcasts, or audiobook-style
   audio files mixed into a library.

### Novel / differentiating
7. **Voice commands.** The browser's built-in speech-recognition API (not
   a heavy model — just simple command-word matching) could drive "next,"
   "pause," "loop" hands-free. Unlike live captioning, this doesn't need
   a large model, since it only has to recognize a handful of short
   commands.
8. **Shareable settings via QR code.** Encode a style preset or a full
   settings snapshot into a QR code so a friend can scan it and get your
   exact setup — no server, no account, fully client-side, consistent
   with the app's local-first design.
9. **PWA install shortcuts.** Once installed, a web app manifest can
    define quick actions (e.g., long-press the home-screen icon → "Shuffle
    now" or "Open Mix Jams") that jump straight past the normal load
    screen.

### Stretch ideas
10. **Lightweight collaborative queue**, as a more tractable alternative
    to full Theater Sync: instead of trying to keep two people's random
    playback in sync (which needs both libraries to match exactly), let
    two open tabs/devices share a *curated queue* over a manual WebRTC
    data-channel pairing — closer to a shared playlist than shared random
    playback, and meaningfully simpler to get right.
11. **Silence/black-frame-aware start points.** A lightweight heuristic
    (sample a few frames or a short audio window before committing to a
    random start point, and nudge away from near-silent or near-black
    moments) would make random starts feel less like a coin flip landing
    on dead air.
