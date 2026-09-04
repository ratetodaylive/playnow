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

## Current configuration

The `CONFIG` block near the top of the `<script>` in `index.html`:

```js
var CONFIG = {
  seriesName   : "One Piece",
  firstEpisode : 1,
  lastEpisode  : 900,
  urlTemplate  : "https://gogoanime.me.uk/newplayer.php?id=one-piece-100?ep={id}&type=hd-1&category=sub",
  idBase       : 2142,
  sandbox      : true,
  overrides    : {},
  titles       : {}
};
```

### The id offset — read this if the wrong episode plays

The player has its own counter that does **not** start at 1. The grid shows
1–900; the URL uses `{id}`, computed as:

```
id = idBase + (episode - firstEpisode)
```

So with `idBase: 2142`:

| Grid button | Player URL id |
|---|---|
| Episode 1   | 2142 |
| Episode 2   | 2143 |
| Episode 100 | 2241 |
| Episode 900 | 3041 |

**This assumes id 2142 is episode 1.** If clicking "Episode 1" plays a
different episode, fix it with one number — open **Settings → "Player id of
first episode"** and adjust. If 2142 is really episode *N*, set the field to
`2142 - (N - 1)`. The current id is always shown next to the Next button, so
you can see the mapping while you check.

If a few episodes break the run (specials or movies inserted into the
sequence), pin those individually with full URLs:

```js
overrides: {
  405 : "https://gogoanime.me.uk/newplayer.php?id=one-piece-100?ep=2999&type=hd-1&category=sub"
}
```

### Other placeholders

| Placeholder | Episode 7 becomes |
|---|---|
| `{id}`   | `2148` (idBase + offset) |
| `{n}`    | `7`    |
| `{nn}`   | `07`   |
| `{nnn}`  | `007`  |
| `{nnnn}` | `0007` |

### Dub instead of sub

Change `category=sub` to `category=dub` in the template.

### Settings are stored per-browser

Anything you save in the **Settings** panel is kept in that browser's
`localStorage` and **overrides the values in the file**. If you edit `CONFIG`
in `index.html` and the page ignores your change, click **Settings → Reset to
file defaults**.

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

GitHub Pages is only free on **public** repos. Two ways to get one:

**A. New repo (easiest)** — at <https://github.com/new>, enter the name,
select the **Public** radio button, leave "Add a README" unchecked, Create.

**B. Repo already exists and is private** — open the repo → **Settings** (top
tab) → **General** → scroll to the bottom, **Danger Zone** → **Change
repository visibility** → **Change to public** → tick the warnings, type the
repo name to confirm.

Then:

1. Create the public repo, e.g. `episode-player`.
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

## The sandbox setting

`sandbox` is an iframe attribute that strips the embedded page's privileges
unless they are granted back individually. When on, this page grants scripts,
same-origin, forms and fullscreen but withholds `allow-popups` and
`allow-top-navigation` — which blocks the pop-under ads and tab-hijack
redirects these hosts run.

**It is off by default**, because gogoanime detects the sandbox and refuses to
play ("our player is not allowed"). That is deliberate on their side: the ads
are how the host is paid, so blocking them blocks playback. There is no
setting that keeps both.

With the sandbox off, use a browser content blocker (uBlock Origin) instead —
it filters the ads without the player being able to tell it is being sandboxed.

## If the video still does not appear

1. Some hosts send `X-Frame-Options: DENY` or a `Content-Security-Policy:
   frame-ancestors` header, which blocks *any* site from embedding them. That
   is enforced by the browser and cannot be worked around from a static page.
2. If the player reports the *domain* is not allowed, it is checking the
   `Referer` header. Find `f.referrerPolicy = 'no-referrer';` in `index.html`
   and delete that line so the browser sends your Pages origin instead.
3. Failing both, use the **Open in new tab** button. The episode list,
   prev/next and progress tracking all still work; only the inline embed is
   unavailable.

## Notes

- No build step, no dependencies, no server code. One HTML file.
- Nothing is uploaded anywhere — watched history lives in your browser only.
- Works offline apart from the player itself.
