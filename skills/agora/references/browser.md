# Browser and desktop

## Mental model

- One Chromium per workspace, shared by all cursors. Each `browser_open` gives you your own tab in its own window, sharing the workspace login profile (cookies, sessions). `isolated=true` gives a fresh incognito-like context; `same_window=true` opens a tab in an existing window.
- Every tab tool takes `target` (from `browser_open` or `browser_tabs`). Browser tools act through CDP on that tab, so they work on background tabs and never steal another cursor's focus.
- `desktop_*` tools use your own MPX pointer and keyboard: real X input, independent of other cursors.

## Reading a page

| Need | Tool | Cost |
| --- | --- | --- |
| What is on screen after an action | already in the action result (`now visible:`) | free |
| Whole page incl. offscreen, all refs | `browser_snapshot` (`max_text` caps text, default 3000) | medium |
| Structured data (tables, lists, JSON) | `browser_eval` with an expression returning JSON | low |
| Looks, layout, images, charts | `browser_screenshot` with `max_width: 800` | high |
| UI quality review | `browser_audit` (overflow, contrast, tap targets, labels, alt, headings, meta, LCP/CLS, broken links) | one call |

Refs look like `[e12] button "Sign in"`. They stay valid until the page re-renders that element; if a ref fails, use the refs from the latest observation.

## Acting

- `browser_click(ref)`; `browser_click(x, y)` in viewport coordinates only for canvases, cross-origin iframes and CAPTCHA widgets.
- `browser_fill(ref, text, submit)` replaces a field's value; `browser_type(text, ref?)` types key by key (autocomplete, editors that ignore value changes).
- `browser_press(keys)`: `Enter`, `Tab`, `Escape`, `ctrl+a`, `ArrowDown`.
- `browser_select(ref, values)` for `<select>`.
- `browser_scroll(dy)` in pixels; `browser_hover`, `browser_drag` for menus and drag-and-drop.
- `browser_upload(files, ref?)`: absolute paths on the Agora host; no ref sets files on a file chooser that just opened.
- `browser_action`: `back`, `forward`, `reload`, `dialog` (`accept`, `text`), `challenge` (detect CAPTCHA), `focus`, `info`, `close`.

## Waiting

- `browser_navigate` waits for `load` by default (`wait: domcontentloaded | none` to go faster).
- `browser_wait` with `selector`, `text` or `url_contains` (`timeout_ms`, default 15000) for SPA transitions and async results.
- `events_wait` for daemon events (`browser.page.loaded`, `browser.dialog.opened`, `browser.challenge.detected`, `process.exited`, `cursor.ended`, `control.returned`) instead of polling.

## Recipes

Log in and extract:
1. `browser_open(url)` → target + visible login form.
2. `browser_fill(email_ref, ...)`, `browser_fill(password_ref, ..., submit: true)` in one turn → visible dashboard.
3. `browser_eval("[...document.querySelectorAll('tr')].map(r => r.innerText)")`.

Responsive check:
1. `browser_emulate(width: 390, height: 844, mobile: true)` → `browser_audit` → one `browser_screenshot(max_width: 600)` if something looks off.
2. `browser_emulate(width: 0, height: 0)` resets.

Testing a local dev server: open `http://127.0.0.1:<port>`; the browser runs on the Agora host, so the server must listen there.

## Desktop apps

1. `desktop_launch(command, args)` starts a program unprivileged with `DISPLAY` set (`restore: true` relaunches it on resume).
2. `desktop_windows` lists windows with ids and geometry; `desktop_screenshot(window: id)` captures one window as a 1024 px JPEG. Pass positions from your last screenshot to `desktop_click`, `desktop_move`, `desktop_drag` and `desktop_scroll` as they are: Agora maps them to the desktop, so never scale them yourself. The same holds for `browser_click(x, y)` after a `browser_screenshot`.
3. `desktop_click`, `desktop_type(text, window?)`, `desktop_key("ctrl+s")`, `desktop_scroll`, `desktop_drag`, `desktop_clipboard`.
4. `desktop_window(window, action)`: `raise`, `move`, `resize`, `release`, `close`.
5. `control_surface(op: acquire, surface: "win:<id>")` when other cursors must not type into a window you are using; release it when done.
