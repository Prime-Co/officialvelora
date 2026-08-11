# Velora — Repaired Local Build

This is your saved "Velora" website, repaired so it runs correctly from a local folder.

## How to run it

Because the site uses `fetch`-style relative CSS/font loading and (on some pages) `localStorage`,
it works best served over a tiny local web server rather than double-clicking the HTML file
(`file://` links to font/CSS files with no extension can be blocked by the browser's MIME checks).

**Easiest option (Python, already on most machines):**
```bash
cd velora-fixed
python3 -m http.server 8080
```
Then open `http://localhost:8080/index.html` in your browser.

**Alternative:** the VS Code "Live Server" extension, or `npx serve`, both work the same way.

Opening `index.html` directly by double-click will also mostly work now (all paths are relative
and use standard extensions), but the server method above is recommended for full compatibility.

## What was wrong

Your ZIP was a browser "Save Page As" export of four separate pages (`index.html`,
`privacy-policy.html`, `dashboard.html`, `payment.html`), each saved at a different time into
its own `..._files` folder. That created several problems:

1. **Renamed script files** — Chrome saves `.js` files with a `.download` extension
   (`bootstrap.bundle.min.js.download`, `jquery-3.7.1.min.js.download`, `aos.js.download`).
   Browsers refuse to execute these, so Bootstrap's JS (navbar toggle, dropdowns, modals, carousels),
   jQuery, and the AOS scroll-animation library were all silently failing to load.
   **Fix:** renamed all three back to proper `.js` files.

2. **Google Fonts stylesheet saved without an extension** — saved as a file literally named `css2`.
   When opened directly this mostly works, but if served through a basic web server it can be
   returned with the wrong `Content-Type` and get blocked as a stylesheet.
   **Fix:** renamed to `fonts-index.css` / `fonts-privacy.css` / `fonts-dashboard.css` and updated
   the `<link>` tags. The actual font files still load from Google's CDN (`fonts.gstatic.com`),
   which is normal and requires an internet connection, same as the live site.

3. **Missing Font Awesome icon files** — `all.min.css` (Font Awesome 6.5.2) references icon font
   files at `../webfonts/*.woff2`/`.ttf`, but the `webfonts` folder was never included in the
   export, so every icon on the site (nav icons, social icons, feature icons, FAQ chevrons, etc.)
   was rendering as a blank box.
   **Fix:** downloaded the matching Font Awesome Free 6.5.2 webfont files and added them at
   `assets/webfonts/`.

4. **Duplicated, page-specific asset folders** — `velora-files/`, `Privacy Policy - Velora_files/`,
   and `Velora AI Academy - Dashboard_files/` each held their own (partially overlapping, partially
   incomplete) copies of the shared CSS/JS/images. This is fragile and easy to break further when
   editing.
   **Fix:** consolidated everything into one shared `assets/` folder (`assets/css`, `assets/js`,
   `assets/images`, `assets/webfonts`) and repointed every page's `<link>`/`<script>`/`<img>` tags
   at it.

5. **Broken in-page anchor links on `index.html`** — nav links were saved as
   `href="index.html/#about-section"` (an invalid path, since `#about-section` was turned into a
   sub-path instead of a hash fragment). This made the "About / How It Works / Earn / Packages /
   FAQ" nav links not scroll to their sections.
   **Fix:** restored them to plain `href="#about-section"` etc.

6. **`privacy-policy.html` pointed back at the live production site** — its nav/footer links used
   absolute URLs like `https://www.ai-velora.site/index.html#about-section`, so clicking them in
   your local copy would leave your local site and open the real website instead.
   **Fix:** rewritten as local relative links (`index.html#about-section`, `privacy-policy.html#section-1`,
   `register.html`).

7. **`dashboard.html`** — this page's CSS is fully inline (no external stylesheet dependency
   besides the Google Fonts link), so it needed only the font-file path corrected. Its JS is also
   self-contained.

`payment.html` was already fully self-contained (inline CSS, no external references) and needed
no changes.

## Things that still won't work locally (by design — not bugs I could fix)

These aren't things the "save page as" process broke — they're server/data dependencies that a
static, offline copy of a website can't have:

- **`register.html` is not included in this export.** Both `index.html` and `privacy-policy.html`
  link to it ("Join Now" / "Get Started" / "Register"), and `dashboard.html`'s JavaScript expects
  a signed-in session (`localStorage.veloraSession`, normally created by the registration/login
  flow) — without it, `dashboard.html` will redirect to `register.html` and 404. If you have that
  file separately, drop it in this same folder and it will work with the existing links.
- **`assets/welcome-audio.mpeg` is missing.** `dashboard.html` references
  `assets/welcome-audio.mpeg` for a welcome sound, but this audio file wasn't part of the saved
  ZIP. The page still works; that one `<audio>` element just has nothing to play. Add the file at
  that path to restore it.
- **Hero video and dashboard avatar/background images are loaded from external CDNs**
  (`res.cloudinary.com` for the homepage hero video, `images.unsplash.com` for some dashboard
  decorative images). These need an internet connection to display, exactly as they did on the
  live site — they were never downloaded locally by the browser's "Save Page As" and there's no
  local copy to restore.
- **Google Fonts** (`fonts.gstatic.com`) likewise load over the internet, same as on the live site.

If you want a version that's 100% usable with zero internet connection, let me know and I can
also download and self-host the Google Fonts files and swap the hero video for a placeholder.
