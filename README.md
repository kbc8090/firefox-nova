# Firefox Nova Tweaks

My personal Firefox setup: a tuned `user.js` built on [Betterfox](https://github.com/yokoffing/Betterfox), and a `userChrome.css` / `userContent.css` that restyle Firefox's **Nova** interface (Firefox 161+).

The theme follows whatever Firefox theme you use. Nova color themes, the new Classic themes, and older themes from addons.mozilla.org all get matching accents, menus and URL bar styling. On top of that, it fine-tunes the tab strip: separators, animations and drag-and-drop.

<!-- Screenshots go here -->

---

## Requirements

- **Firefox 161 or newer** with the Nova interface. `user.js` sets `browser.nova.enabled` so the Nova-only rules always apply.
- **Windows.** Everything was built and tested on Windows 10. Most of it should work elsewhere, but it isn't tested.
- **`toolkit.legacyUserProfileCustomizations.stylesheets` set to `true`.** `user.js` handles this.
- **`ui.prefersReducedMotion` set to `0`** for the tab and menu animations. `user.js` handles this too.
- **Optional fonts:** [Inter](https://rsms.me/inter/) for add-on badges and the zoom indicator. Without it, Firefox falls back to the default font.

## Installation

1. Open `about:support` and click **Open Folder** next to **Profile Folder**.
2. Copy `user.js` into the profile folder itself.
3. Create a folder named `chrome` in the profile folder and copy in the files listed below.
4. Restart Firefox.

```
<profile folder>/
├── user.js
└── chrome/
    ├── userChrome.css     # browser interface
    ├── userContent.css    # new tab, blank and private browsing pages
    ├── Back.svg           # custom back icon
    └── Forward.svg        # custom forward icon
```

> **Note:** Firefox reapplies `user.js` every time it starts. If you change one of its prefs in `about:config`, the change is reverted on the next launch. Edit `user.js` instead.

---

## user.js

Based on Betterfox's four sections (Fastfox, Securefox, Peskyfox, Smoothfox), adjusted for a desktop with a dedicated GPU, a 240Hz monitor and a fast connection. Overall it favors speed and a clean interface. Privacy settings are strict where they don't break sites.

### Performance (Fastfox)
- **GPU rendering:** WebRender forced on, with the layer compositor, precached shaders, and GPU-accelerated canvas with larger canvas caches.
- **Font rendering:** ClearType tuned (gamma, contrast, rendering mode) for crisper text on Windows.
- **Caching:** bigger image cache, immediate image decoding, and longer media read-ahead.
- **Networking:** more connections per server, larger DNS cache, faster TLS session reuse, HTTP/3, and speculative loading (DNS prefetch, predictor, preconnect on hover).
- **Media and formats:** JPEG XL, HEVC, WebCodecs H.265 and WebGPU enabled.
- **Processes:** 4 content processes. Windows "efficiency mode" throttling is disabled for background tabs, and the foreground tab's main thread is raised in priority.
- **Frame rate:** follows the monitor's refresh rate (`layout.frame_rate = -1`).

### Privacy and security (Securefox)
- **Tracking protection:** Enhanced Tracking Protection set to Strict, with a few social sites exempted so embeds keep working.
- **Encrypted DNS:** DNS over HTTPS in strict mode (`network.trr.mode = 3`). Lookups never fall back to plain DNS.
- **Connections:** HTTPS-First. Certificate revocation uses CRLite instead of OCSP, TLS 0-RTT is disabled, and mixed content is blocked or upgraded.
- **Privacy signals:** Global Privacy Control enabled, and cross-origin referrers trimmed.
- **Data collection:** telemetry, studies, Normandy, Nimbus rollouts, the coverage ping and crash report auto-submission all off.
- **Permissions:** geolocation off (BeaconDB set as the provider if it's ever enabled), and notification requests denied by default.
- **Forms and files:** saved addresses and credit cards off, PDF scripting off, and downloads go to a temp folder first.

### AI features and clutter (Peskyfox)
- **AI features blocked:** the AI controls, the ML engine, the AI chatbot sidebar and menus, link previews, smart tab groups and "smart window".
- **Promos off:** sponsored content, Pocket, VPN and sync promos, add-on and feature recommendations, and the welcome and upgrade pages.
- **URL bar:** trimmed to the essentials. No trending, weather, Wikipedia, Yelp or quick suggestions, no top sites, and no one-off search buttons. Calculator and unit conversion stay on.
- **New tab page:** top sites, stories, weather and sponsored items off.
- **Tabs:** tab groups and drop-to-pin off, tab hover previews effectively disabled, unloaded tabs faded out, and Ctrl+Tab ordered by recent use.
- **Cookie banners:** rejected automatically where Firefox knows how.
- **Fullscreen:** no transition animation and no warning popup.

### Interface prefs
Nova enabled, compact density available, focus rings removed, GTK-style scrollbars that follow the theme accent, a minimum tab width of 57px, a 750ms tooltip delay, and the Alt key no longer focusing the menu bar.

### Smooth scrolling (Smoothfox)
Physics-based ("MSD") smooth scrolling with custom spring constants and a faster mouse-wheel multiplier, tuned for a snappy but smooth feel on a high refresh rate monitor.

### Things to be aware of
- **Strict encrypted DNS:** with `network.trr.mode = 3`, sites won't load at all if the DNS-over-HTTPS provider is unreachable, for example on some captive-portal Wi-Fi. Set it to `2` to allow a fallback.
- **IPv6 is disabled.**
- **Hardware video:** `media.wmf.zero-copy-nv12-textures-force-enabled` forces a hardware video path that isn't enabled for every GPU by default. Remove it if videos glitch.
- **Leftover MaterialFox block:** the commented-out MaterialFox section is from an older setup and isn't used.

---

## userChrome.css

### Theme-aware colors
The goal is that any theme looks intentional, with no stray Windows accent color.

- **Nova color themes** (Sun, Spark, Flame, Dusk, Lagoon and the rest): an accent color is generated from the theme's own frame color by shifting its hue and boosting its saturation. Teal and green themes get their own tuning. That accent replaces the Windows system accent on focus rings, toggles, buttons, checkboxes, progress bars, and the bookmark star and download "done" icons.
- **Classic and add-on themes:** use the theme's own focus color when it has one. Otherwise they fall back to the classic Firefox teal.
- **Classic themes' URL bar:** the dark `#161616` URL bar is replaced with the theme's top bar color.
- **Hover and selection:** URL bar, find bar and bookmark-editor fields get a soft tinted hover. Text selection is tinted with the accent.
- **Content area:** the area behind web pages matches the theme, so there's no white flash while a page loads.

### Tab strip
- **Size:** a compact 32px strip with 27px tabs, a 209px maximum tab width and 8px rounded tab backgrounds.
- **Pinned tabs:** widened to 42px with centered icons.
- **Close buttons:** only shown on the selected and hovered tabs, with a smaller click target. They stay usable on very narrow tabs.
- **Tab label:** the "Playing" / "Muted" text is hidden, and unselected tabs have slightly dimmer text.
- **Loading:** a spinning ring in the theme accent replaces the throbber, and the site icon pops in when the page finishes loading.
- **Tooltips:** tab tooltips and hover previews are hidden.

### Tab separators
Thin 1px separators drawn with pseudo-elements, built to behave like part of the tabs rather than fixed lines:
- **Hover and selection:** they disappear instantly around the selected and hovered tabs, and fade back in.
- **Closing:** they stay hidden while a neighboring tab closes, so they never flash.
- **Pinned divider:** the line between pinned and normal tabs has its own rules, including when the tab strip is overflowing.
- **Dragging:** separators slide along with the tabs in both directions, even when you change direction mid-drag. When you drop, they hand back to the normal separators on the exact same pixel, with no flicker.

### Tab animations
- **Open and close:** quicker than Firefox's default (100ms and 80ms) on a Material-style curve.
- **Drop:** a Chromium-style settle animation (200ms) when you drop a dragged tab.
- **Drag overshoot:** a 25px overshoot zone lets you drag a tab slightly past the last tab, over the + button.
- **Fixes:** corrects Firefox's 1px jump when dragging the last tab, and makes the + button keep up when tabs close.

### URL bar and dropdown
- **Selection:** a taller, Chrome-like text selection highlight, using a font with adjusted line metrics.
- **Placeholder:** the "Search with Google or enter address" text is hidden.
- **Hover:** a smooth Material-style hover transition.
- **Nova themes:** the dropdown reuses the toolbar's gradient with a subtle accent sweep, and opens with a short reveal animation.
- **Results:** tighter rows, no badge on history results, and a capped height with a hidden scrollbar.
- **Shape:** slightly tighter corner rounding than Nova's defaults.
- **Site identity:** the site identity box is hidden on normal secure pages, leaving room for the trust panel icon.

### Menus and panels
- **Backgrounds:** context menus and panels follow the theme.
  - **Default theme:** a gradient ring like the selected tab.
  - **Nova themes:** a shared dark surface made from the toolbar's gradient and the accent.
  - **Older themes:** the theme's frame color.
- **Shape and states:** soft shadows, 8px corners, consistent hover and active states, and disabled items that don't react to the mouse.
- **Context menus:** cleaned up. No Send to Device, Share, Strip on Share, send image or video, or "Add search engine".
- **Panel headers:** a soft gradient.
- **Trust panel (shield icon):** compacted, with an accent border, and a "No trackers blocked on this site" message when the list is empty.
- **Bookmark editor and downloads panel:** themed fields and tidier spacing.

### Find bar and status panel
- **Find bar:** a floating, rounded card in the top-right of the page. It's themed like the menus, opens and closes instantly, and only shows the controls you actually use.
- **Status panel:** the link and loading label in the bottom-left uses the same styling.

### Everything else
- **Icons:** custom back and forward icons (`Back.svg`, `Forward.svg`).
- **Toolbars:** rounded top corners on the navigation bar, and adjusted bookmarks toolbar spacing and separators.
- **Small details:** cleaner add-on badges (such as uBlock Origin's count) and a bolder zoom indicator.
- **Library window:** the Library window (history, bookmarks, downloads) gets matching menus and selection colors.

---

## userContent.css

- **Background:** the new tab, blank and private browsing pages use a dark violet background instead of white.
- **New tab page:** the **Personalize** button stays hidden until you hover the bottom-right corner, and the wallpaper credit is hidden.
- **Private browsing page:** reduced to a single centered logo.

---

## Known limitations

- **Firefox updates can break things.** The CSS relies on Firefox's internal markup (attributes like `movingtab`, `dragtarget` and inline styles set during drags), and any update can change that, as Nova did.
- **Context menu animation:** Firefox 161 added an open animation to panels like the main menu. Context menus can't get it from CSS alone. It would need a userChrome.js loader.
- **Dragging a tab into first place:** with pinned tabs, the divider appears instantly instead of sliding, because the sliding version gets clipped.
- **Overflowing tab strip:** the drag overshoot is disabled when tabs overflow.

## Credits

- [Betterfox](https://github.com/yokoffing/Betterfox) by yokoffing, the base of `user.js`.
- Firefox's own source code, used throughout to track down variable names, theme behavior and tab drag internals.
