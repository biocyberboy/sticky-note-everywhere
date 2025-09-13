# Sticky Note Everywhere (Chrome/Chromium Extension, MV3)

A draggable, resizable sticky note overlay that can pin across all tabs and syncs via `chrome.storage.sync`.

- Position: fixed; default bottom-right (24px margin), very high z-index.
- Header: title, Pin toggle, Minimize, and Close.
- Body: textarea with 300ms debounced autosave.
- Draggable and resizable; remembers position and size.
- Global state in `chrome.storage.sync`: `{ pinned, visible, minimized, noteText, pos, size }`.
- Service worker injects content on active tab changes or page loads (when pinned+visible).
- Keyboard shortcut: `Alt+Shift+N` to toggle visibility when pinned is ON.
- Light/Dark theme via `prefers-color-scheme`.
- Skips restricted pages (chrome://, Web Store, Edge Add-ons, etc.).

## Project Structure

- `manifest.json` — MV3 manifest.
- `background.js` — service worker. Injects content via `chrome.scripting` and handles commands.
- `content.js` — overlay injection and UI logic (drag/resize, buttons, sync).
- `styles.css` — overlay styles, minimal and theme-aware.
- `popup.html` / `popup.js` — optional action popup to toggle state quickly.
- `(optional) icons/` — you can add PNG icons if you want.

## Installation (Developer Mode)

1. Open Chrome/Chromium and navigate to `chrome://extensions/`.
2. Enable "Developer mode" (top-right toggle).
3. Click "Load unpacked" and select this folder.
4. Optionally, click the extension’s Details → "Extension options" to pin the action icon.
5. Assign the shortcut: open `chrome://extensions/shortcuts`, set "Toggle sticky note visibility" to `Alt+Shift+N` if not already set.

## Usage

- Click the action icon to open the popup; toggle `Pinned`, `Visible`, and `Minimized`.
- When `Pinned` and `Visible` are true, the overlay auto-appears on active tabs and persists across reloads. By default the panel is closed; click the ☰ button (top-left) to open the sidebar and editor.
- Use `Alt+Shift+N` to globally toggle visibility when `Pinned` is ON.
- Close (X) sets `visible=false` globally—no auto-inject until re-enabled via popup or the shortcut.
- Drag by grabbing the header; resize via the container edges/corner. Position and size are saved.
- Download current note via the ⬇ button on the header (saves HTML; pasted images are preserved).
- Resize the history sidebar by dragging the vertical divider between the sidebar and the editor; width is saved and synced across tabs.

## Domain Restrictions

The extension skips restricted pages (e.g., `chrome://*`, Chrome Web Store, Edge Add-ons) and silently ignores injection errors. Some pages (e.g., Chrome settings) do not allow extensions to inject content—this is expected.

## State Schema

Stored under `chrome.storage.sync` key `sne_state`:

```
{
  pinned: true,
  visible: true,
  minimized: false,
  noteText: "",
  pos: { x: null|number, y: null|number },
  size: { w: 320, h: 240 }
}
```

- `pinned`: when true, background injects into active tabs.
- `visible`: controls whether overlay shows at all; `Alt+Shift+N` toggles this when `pinned` is true.
- `minimized`: hides textarea but keeps header visible.
- `noteText`: the sticky note contents; debounced save (300ms).
- `pos`: top-left position. Defaults to bottom-right if null, then saved.
- `size`: overlay size in pixels.

## Build/Pack (Optional)

You can load unpacked directly. To pack:
- Chrome: chrome://extensions → Pack extension… → select this folder.

## Test Checklist

- Switch tabs with `Pinned=true` and `Visible=true`: overlay persists without flicker.
- Click `X` (Close): overlay disappears on all tabs; stays hidden until toggled/pinned again.
- Type in the note, reload the page: text persists.
- Drag/Resize, then reload or switch tabs: position/size persist.
- Shortcut `Alt+Shift+N` toggles visibility (only when `Pinned=true`).

## Notes

- No frameworks; clean JS/CSS with MV3 APIs only.
- Content only attaches one overlay per tab (`#sticky-note-overlay`).
- Errors on restricted domains are caught and ignored by the background worker.
- If you need icons, add PNGs (16/32/48/128) and reference them in `manifest.json` under `icons`/`action.default_icon`.
