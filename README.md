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

## Password lock

The library is not stored in the page. `config.enc.js` holds it encrypted with
**AES-256-GCM**, under a key derived from your password with **PBKDF2-SHA256,
310,000 iterations**. Without the password the file is an opaque blob — "View
Source" on the deployed site reveals no player URL, no episode ids, nothing.

### First-time setup

1. Open `encrypt.html` from this folder (double-click it, or serve it locally).
2. Enter a password twice, fill in the player URL template and episode range.
3. Click **Generate config.enc.js**, then **Download** (or **Copy**).
4. Save the result as `config.enc.js` in this folder, replacing the placeholder.
5. Open `index.html`, unlock with your password to confirm it works.
6. Commit and push. The site redeploys in about a minute.

To change the URL or episode count later, re-run `encrypt.html` and replace the
file again. `encrypt.html` contains no secrets of its own — it is safe to commit.

### Remembering devices

Ticking **Remember this device for 90 days** stores the derived key in that
browser's `localStorage`, so the device unlocks silently on later visits. It is
per-browser: unlock once on your phone and once on your PC.

To revoke a device you still hold, open it and press **Settings → Forget this
device**. To revoke a device you *don't* hold, re-run `encrypt.html` with a new
password and push — every stored key stops working.

### What this does and does not protect

**Does:** someone who finds the URL sees a password box and nothing else. The
contents are genuinely encrypted, not merely hidden.

**Does not:** there is no server, so an attacker who downloads `config.enc.js`
can guess passwords offline as fast as their hardware allows, with no rate limit
and no lockout. PBKDF2 at 310k iterations makes each guess cost real time, but a
short or common password will still fall. **Use a long passphrase** — four or
five unrelated words is far stronger than a short string of symbols.

There is also no per-user access and no audit trail; anyone you give the
password to has it permanently, until you re-encrypt with a new one.

If you want real access control — per-person logins, revocation, server-side
rate limiting — put the site behind Cloudflare Access instead. The two can be
combined.

## Configuration

Everything below is set in `encrypt.html` and lives inside the encrypted
`config.enc.js`. Nothing sensitive is kept in `index.html` or in this README.

### The id offset — read this if the wrong episode plays

The player has its own counter that usually does **not** start at 1. The grid
shows 1–900; the URL uses `{id}`, computed as:

```
id = idBase + (episode - firstEpisode)
```

So an `idBase` of 2142 maps episode 1 → 2142, episode 2 → 2143, and episode
900 → 3041.

If clicking "Episode 1" plays a different episode, fix it with one number —
**Settings → "Player id of first episode"**. If your starting id is really
episode *N*, subtract `N - 1` from it. The current id is shown next to the Next
button, so you can watch the mapping as you adjust it. To make the change
permanent, re-run `encrypt.html` with the corrected value.

If a few episodes break the run (specials or movies inserted into the
sequence), pin them individually in the `overrides` object inside the encrypted
payload — episode number to full URL.

### Placeholders

| Placeholder | Episode 7 becomes |
|---|---|
| `{id}`   | `idBase + 6` |
| `{n}`    | `7`    |
| `{nn}`   | `07`   |
| `{nnn}`  | `007`  |
| `{nnnn}` | `0007` |

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
