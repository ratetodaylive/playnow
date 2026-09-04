# Episode Player

A single-file, zero-dependency episode index + player wrapper for GitHub Pages.
Lists every episode in a range (1–900 by default), loads the matching player URL in
an embedded frame, and gives you Prev / Next buttons under the player.

## What it does

- Episode grid, split into 100-episode blocks so 900 buttons stay usable
- **Prev / Next** buttons directly under the player, plus `←` / `→` keyboard shortcuts
- "Go to ep #" box in the header for jumping straight to any number
- Deep links: every episode has its own URL — `.../index.html#ep742` — so you can
  bookmark or share a specific episode, and refresh keeps your place
- Watched marks + "resume where you left off", stored in the browser (`localStorage`)
- Settings panel so the player URL can be changed in the browser, no re-deploy
- Pop-up blocking via iframe `sandbox` (toggleable, in case a host needs it off)

## Setup — the only thing you must change

Open `index.html` and edit the `CONFIG` block near the top of the `<script>`:

```js
var CONFIG = {
  seriesName   : "My Series",
  firstEpisode : 1,
  lastEpisode  : 900,
  urlTemplate  : "https://host.com/embed/anime?ep={n}",
  sandbox      : true,
  overrides    : {},
  titles       : {}
};
```

`urlTemplate` is your player URL with the episode number replaced by a placeholder:

| Placeholder | Episode 7 becomes |
|---|---|
| `{n}`    | `7`    |
| `{nn}`   | `07`   |
| `{nnn}`  | `007`  |
| `{nnnn}` | `0007` |

Examples:

```
https://host.com/embed?ep={n}
https://host.com/watch/my-series-episode-{n}
https://host.com/v/{nnn}/index.html
https://host.com/e/{n}?sub=1&autoplay=1
```

You can also set it at runtime: open the page, click **Settings**, paste the
template, Save. That value is stored in your browser and survives reloads —
handy for testing before you commit.

### Episodes that break the pattern

```js
overrides: {
  12  : "https://host.com/embed/special-12",
  405 : "https://other-host.com/e/405"
}
```

### Episode titles (optional)

```js
titles: {
  1: "Romance Dawn",
  2: "The Great Swordsman"
}
```

Titles show in the "now playing" bar and as a tooltip on each grid button.

## Deploy to GitHub Pages (free)

1. Create a new **public** repo on GitHub, e.g. `episode-player`.
2. Push this folder:

```bash
git init
git add .
git commit -m "Episode player"
git branch -M main
git remote add origin https://github.com/<your-username>/episode-player.git
git push -u origin main
```

3. On GitHub: **Settings → Pages → Source: Deploy from a branch → `main` / `root` → Save**.
4. Wait ~1 minute. Your site is live at
   `https://<your-username>.github.io/episode-player/`

Every later change is just `git push` — Pages redeploys itself.

## If the video does not appear

Many video hosts send `X-Frame-Options: DENY` or a `Content-Security-Policy:
frame-ancestors` header, which blocks *any* site from embedding them. That is
enforced by the browser and cannot be worked around from a static page.

Two things to try, in order:

1. **Settings → uncheck "Block pop-ups (sandbox)"** — some players need
   permissions the sandbox withholds.
2. Use the **Open in new tab** button. The episode list, prev/next and progress
   tracking all still work; only the inline embed is unavailable.

## Notes

- No build step, no dependencies, no server code. One HTML file.
- Nothing is uploaded anywhere — watched history lives in your browser only.
- Works offline apart from the player itself.
