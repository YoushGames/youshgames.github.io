# Yoush Games site — how to add a game page

This repo is the Yoush Games website on GitHub Pages: `https://youshgames.github.io/`.
It has a studio landing page that lists our games, plus one **privacy policy page per game**.
Follow this guide every time a new game is added. Everything below was learned building
`defensetime/`, which is the **reference implementation**. Copy its structure, not its look.

---

## 0. Ground rules

- **Never commit or push without the user explicitly saying so.** Pushing to `main` publishes the
  site immediately. Build and verify locally, show the user, then ask.
- **The policy must describe what the game actually does.** Never copy a template or another game's
  policy text. Audit the game's code first (step 1). Every sentence must be true for *this* build.
- **No third-party requests from the site.** No Google Fonts CDN, no analytics, no external scripts
  or images. A privacy page that leaks visitors' IPs is a bad look (and a GDPR issue in the EU).
  Everything is self-hosted in this repo.
- **Plain static HTML + inline CSS.** No build step, no frameworks, no JavaScript needed.

---

## 1. Site structure

```
/
├── index.html                 Studio landing page (dark "ember" design, lists all games)
├── favicon.ico                Studio icon (Yoush wings, dark square) — do not replace
├── apple-touch-icon.png       Studio icon, 180×180 — do not replace
├── _config.yml                Keeps .md docs out of the published site
├── GAME_PAGES.md              This guide (not published)
├── CLAUDE.md                  Points Claude at this guide (not published)
├── assets/
│   ├── yoush-logo.png         Studio logo (wings + YOUSH), 720 px, transparent
│   ├── fonts/                 Self-hosted OFL fonts + one OFL_<Font>.txt licence each
│   └── games/<slug>.webp      144×144 game icon used on the landing page card
└── <slug>/                    One folder per game, lowercase, no dashes (e.g. defensetime)
    ├── index.html             The game's privacy policy page
    ├── favicon.ico            Game icon, 16/32/48 px
    ├── apple-touch-icon.png   Game icon, 180×180, opaque (no transparency)
    └── img/
        ├── bg.jpg             Game background, 1600×900, ~30 KB
        ├── logo.webp          Game title logo, 900 px wide
        └── icon.webp          Game icon (196–512 px) shown on the page
```

Live URL of a game page: `https://youshgames.github.io/<slug>/`. This is the URL that goes into
the Google Play Console / App Store Connect "Privacy policy" field.

Existing games:

| Slug | Game | Engine / source | Data collected | Page style |
|---|---|---|---|---|
| `paintpool/` | Paint Pool | Android (Kotlin), `F:\Projects\PaintPool` | Nothing (offline, no SDKs); progress in Android backup | Light: gouache paper, tinted paint blobs, the game's own fonts ← **reference for "no data" games** |
| `defensetime/` | Defense Time | Unity, `D:\Project\DefenseTime` | Ads, analytics, leaderboard | Dark: teal HUD + hexagons ← **reference for games with SDKs** |

Some game repos keep their own copy of the policy text (Paint Pool: `PRIVACY_POLICY.md`). When the
page text changes, update that copy too, so both say the same thing. (`F:\Projects\PaintPool\web\` is
an old snapshot of this site. It is not the live source; edit this repo instead.)

---

## 2. Workflow for a new game (checklist)

1. **Audit the game's data collection** (section 3) → list of facts.
2. **Write the policy text** from those facts (section 4).
3. **Collect the game's art** and pick the page's look (section 5).
4. **Prepare assets** with the scripts in section 6 (background, logo, icons, favicons).
5. **Build `<slug>/index.html`** by copying `defensetime/index.html` and re-skinning it (section 7).
6. **Add the game card** to the landing page `index.html` (section 8).
7. **Verify** locally with screenshots on desktop and phone width, and check links (section 9).
8. **Report to the user.** List the files, the screenshots, and the compliance gaps you found in the game.
   Then **ask** before committing or pushing.
9. After the user pushes: remind them to set the privacy URL in the store console and make the
   **Data safety** (Google) / **App Privacy** (Apple) answers match the page.

---

## 3. Audit the game (do this before writing a word)

Find out exactly what leaves the device. For a **Unity** project check:

- `Packages/manifest.json` — look for `com.unity.services.*` (analytics, leaderboards, cloud save,
  authentication), `com.unity.purchasing` (IAP), `com.unity.services.levelplay` / ironSource / AdMob,
  Firebase, Facebook, GameAnalytics, etc.
- `Assets/Plugins/Android/AndroidManifest.xml` and any `*Dependencies.xml` (ad network adapters).
  Note permissions: `AD_ID`, `INTERNET`, location, camera, mic, contacts.
- Grep the scripts for what is actually *used*. An installed package is not the same as a used one.
  Defense Time had `com.unity.purchasing` installed but no IAP code, so the policy does not mention purchases.
  Useful patterns: `LevelPlay|IronSource|Analytics|RecordEvent|Leaderboards|AuthenticationService|
  Purchasing|Firebase|Input.location|WebCamTexture|Microphone|deviceUniqueIdentifier|OpenURL|PlayerPrefs`.
- Consent / privacy UI. Is there a toggle? A "delete my data" button? Which scene is it in? Is it on by default?
  Unity scenes may be binary: `grep -a -c <fieldName> Assets/Scenes/*.unity` still finds serialized
  field names.
- Leaderboards: what is shown to other players (scores, generated player names)?
- Ads: rewarded only (opt-in), or also interstitial/banner? Find where `ShowAd` is called.
- Product name, publisher, package ID: `ProjectSettings/ProjectSettings.asset`
  (`grep -a productName|companyName|applicationIdentifier`).

For a **native Android** project, check `build.gradle(.kts)` dependencies, `AndroidManifest.xml`
permissions, and any network code. Also list **every** key the game saves locally (e.g. grep
`getSharedPreferences|put(Int|String|Boolean|StringSet)`), because new features add new saved data
(Paint Pool had gained coins and owned skins since its first policy).

For **any** Android game, check **`android:allowBackup`** and `res/xml/backup_rules.xml` /
`data_extraction_rules.xml`. If saves are included, the policy must say that Android may back up the
progress to the player's Google account and copy it to a new phone (handled by Google; we never
see it). Don't claim the data "never leaves the device". Unity sets `allowBackup` in its generated
manifest, so check the built manifest or Player Settings.

Also tell the user about **compliance gaps in the game itself**. Don't paper over them in the policy.
Examples found in Defense Time: analytics on by default (should be opt-in for EU), and no consent
signal passed to the ad SDK.

---

## 4. Policy content

### Section order (game with SDKs: copy from `defensetime/index.html`)

1. Header meta: **Effective date** (the day you write it, e.g. `22 October 2026`) and **Publisher:
   Yoush Games, Denmark**.
2. Intro: game icon + one-sentence lead ("X is a free game supported by ads…").
3. **Summary** — 3–5 plain bullets: what we don't collect, account or no account, ads, analytics.
4. **Data collected** — bold label + description: device/technical, identifiers, gameplay,
   leaderboard (say it is *visible to other players*), ad data.
5. **How the data is used** — bullets, then "We do not sell your data…".
6. **Service partners** — one `.partner` card per SDK (name + what it receives). Link to each
   vendor's privacy policy. Mention transfer outside the EU.
   Unity links: `https://unity.com/legal/game-player-and-app-user-privacy-policy` (covers Unity Ads,
   LevelPlay/ironSource, Analytics) and `https://unity.com/legal/privacy-policy`.
7. **Your choices** — the in-game toggle/delete button (name the menu where it is), device ad
   settings (Settings → Privacy → Ads / Settings → Google → Ads), email us for leaderboard/other data,
   local data removed on uninstall.
8. **Retention**, 9. **Children** (not directed at under-13s, unless the game targets kids; check
   the Play Console target audience), 10. **Security**, 11. **Changes**, 12. **Contact**.

### Section order (game that collects nothing — see `paintpool/index.html`)

Data we collect: **None** → Data stored on your device (list every saved item) → Android backup
(if enabled) → Network (offline) → Children → Changes → Contact.

Don't claim legal compliance ("complies with COPPA"). State the facts ("collects no personal data
from anyone, including children").

### Never include

Template filler that does not apply: location (unless actually used), browser/"pages visited",
accounts/account settings, email marketing, purchases (unless IAP is used), affiliates.

Contact email for all pages: **youshgamestudio@gmail.com**.

---

## 5. Design: each game page looks like its game

The **landing page** has the studio look (see 5.1). Each **game page** takes its look from that
game's own art, so a player recognises it at once.

### 5.1 Studio landing page (fixed, don't restyle per game)

- Background `#060303` with radial ember glow `#3d1009` → `#1a0705` → bg, centred near the top.
- Studio logo `assets/yoush-logo.png` at `min(340px, 72vw)` with a slow red `drop-shadow` pulse.
- Tagline "Handcrafted mobile games", uppercase, letter-spacing `.32em`, colour `#a8958c`.
- Game cards: dark glass `rgba(255,255,255,.035)`, border `rgba(236,90,18,.18)`, radius 16 px;
  hover lifts 2 px with a red-orange glow. Real game icon (52 px, radius 13 px), name, "Privacy
  policy", arrow →.
- Brand colours: red `#e5161a`, orange `#ec5a12`.

### 5.2 Finding a game's art

Look in the game project for:

| Need | Where it was for Defense Time | What to look for |
|---|---|---|
| Background | `Assets/Graphics/Textures/HUD/MainMenuBG.png` | main menu background, 1920×1080 |
| Title logo | `Assets/Graphics/Textures/HUD/AirDefenseLogo.png` | logo with transparency |
| Icon | `Assets/Graphics/Textures/GameIcon.png` (196 px only) | store icon, `*512*`, `ic_launcher` |
| UI motifs | `HUD/New02/HudBorder.png`, `HUD/Hexagon.png` | frames, corner shapes, bullet shapes |
| Fonts | `Assets/Fonts/` (Agency FB, Gunplay) | game fonts (to find a look-alike) |

For native Android: `store/play_icon_512.png`, `app/src/main/res/mipmap-*/ic_launcher*.png`, `art/`.

**Always preview transparent PNGs on a dark background** before using them. White elements are
invisible on a white preview. `AirDefenseLogo.png` hid white HUD lines that only showed on the dark page.

### 5.3 Turning the art into a page (pattern from Defense Time)

- **Palette:** sample 1 accent colour from the art (DT: cyan `#4fe3d6`), a near-black tinted
  background (DT: `#020a0b`), a panel colour with ~86% opacity, white for strong text, and a
  soft tint for body text. Define all of them as CSS custom properties on `:root`.
- **Background:** game background as a `body::before` fixed layer (works on iOS, unlike
  `background-attachment: fixed`). Put a dark vertical gradient over it so text stays readable.
- **Header:** small "← Yoush Games" back link, the game's logo image (`min(500px, 90vw)`, gentle glow
  pulse), "PRIVACY POLICY" subtitle, and two meta chips (effective date, publisher).
- **Panel:** echo the game's UI frames. DT uses chamfered corners (`clip-path` polygon with a
  `--cut` variable). The 1 px glowing edge comes from an outer `.frame` with a gradient background and
  `padding:1px`, with the inner `.card` clipped the same way. Hexagon nodes sit on two corners as
  `::before/::after` of the `.hud` wrapper.
- **Headings:** a display font in the spirit of the game font, uppercase and letter-spaced, in the
  accent colour, with a game-shaped marker (DT: hexagon) and a fading rule after.
- **Bullets / partner cards:** reuse the motif (hexagon bullets, cards with an accent left border and
  one cut corner).
- **Body text:** system font stack, 16 px / 1.7. Keep long legal text in a readable font; the
  display font is only for labels and headings.
- **Footer:** small studio logo + "A Yoush Games title", linking to `../`.

### 5.4 Second pattern: Paint Pool (light, painterly)

- **Palette:** paper `#efe7d6`, cream slab `#fbf6ea`, ink `#35302a`, and the game's six paint colours
  (red `#e04a35`, orange `#f0a23a`, green `#6aaa4a`, teal `#2f9e9b`, blue `#2f62c4`, purple `#9a5bb8`).
- **Background:** the game's paper texture (`bg_warm.jpg`, centre square, 900 px) as the fixed layer.
- **Header:** the game's impasto title image with a soft drop shadow. Five paint blobs float around
  it (a gentle bob animation), resized and moved on phones so they don't cover the title.
- **Tinted sprites:** the game ships **white** paint sprites that it tints in code. The page does the
  same in CSS. The `.blob` class uses the sprite as both `mask` (shape) and a background image
  multiplied with `--c` (`background-blend-mode: multiply`), which keeps the painted relief. One image
  gives every colour. Heading markers cycle through the six colours with `section:nth-of-type`.
- **Card:** a cream "paint slab" with a large radius and inset shadows, like the game's buttons. Saved
  data is shown as raised pill chips, and "None." sits on a teal paint swatch.
- **Fonts:** the game's own Lilita One (display) and Varela Round (body).

Another genre would give other choices. A cosy puzzle game might use rounded cards, warm paper
and a soft serif. A racing game might use slanted cards and speed lines. Keep the **structure** and
the **accessibility rules** the same (section 7). Change the skin.

### 5.5 Fonts

- Only self-host fonts under the **SIL Open Font License** (Google Fonts are OFL or Apache). Put the
  `.woff2` files (latin subset, only the weights used) in `assets/fonts/` named `<font>-<weight>.woff2`,
  with the licence next to them as `OFL_<FontName>.txt`. Already there: Rajdhani (DT),
  Lilita One + Varela Round (Paint Pool).
- If the game itself ships OFL fonts (Paint Pool does, in `app/src/main/assets/fonts/`), use
  **the game's real fonts**, with the licence files copied from the game.
- **Never put the game's own font files on the web** unless their licence allows web embedding.
  Agency FB and similar commercial fonts don't.
- Look-alikes: Agency FB / military HUD → **Rajdhani** (already in repo); techy → Oxanium, Chakra
  Petch; rounded/cute → Baloo 2, Fredoka; bold arcade → Russo One, Bungee.
- To download a font's latin woff2 files, fetch the Google Fonts CSS with a Chrome user agent and
  grab the `/* latin */` URLs (that is how Rajdhani was fetched). Use `font-display: swap` and
  `preload` the main weight.

---

## 6. Asset recipes (Python + Pillow)

Run with `python -I`. Write source paths explicitly. Targets: background ≈ 30 KB, logo < 80 KB,
icons < 20 KB.

```python
from PIL import Image, ImageChops, ImageDraw, ImageFilter
R   = r'F:\Projects\youshgames.github.io'
G   = R + r'\<slug>'                     # game folder
SRC = r'<path to game art folder>'

# Background: 1600x900 progressive JPEG
bg = Image.open(SRC + r'\MainMenuBG.png').convert('RGB').resize((1600, 900), Image.LANCZOS)
bg.save(G + r'\img\bg.jpg', quality=80, optimize=True, progressive=True)

# Title logo: crop to the title + its glow, feather the edges so the glow
# doesn't end in a hard rectangle, 900 px wide WebP
lg = Image.open(SRC + r'\Logo.png').convert('RGBA')
c  = lg.crop((x0, y0, x1, y1))           # pick by rendering on dark with a ruler first
m  = Image.new('L', c.size, 0)
ImageDraw.Draw(m).ellipse((30, 10, c.width - 30, c.height - 10), fill=255)
m  = m.filter(ImageFilter.GaussianBlur(40))
c.putalpha(ImageChops.multiply(c.split()[3], m))
c  = c.resize((900, round(c.height * 900 / c.width)), Image.LANCZOS)
c.save(G + r'\img\logo.webp', quality=88, method=6)
# -> put the real width/height on the <img> tag

# Icons: page icon, landing-card icon, favicon, apple-touch icon
ic = Image.open(SRC + r'\Icon.png').convert('RGBA')         # largest available
ic.save(G + r'\img\icon.webp', quality=88, method=6)
ic.resize((144, 144), Image.LANCZOS).save(R + r'\assets\games\<slug>.webp', quality=90, method=6)
ic.save(G + r'\favicon.ico', format='ICO', sizes=[(16, 16), (32, 32), (48, 48)])
inset = round(ic.width * 0.06)   # only if the icon has transparent rounded corners (iOS adds its own)
ic.crop((inset, inset, ic.width - inset, ic.height - inset)).resize((180, 180), Image.LANCZOS) \
  .convert('RGB').quantize(256, method=Image.MEDIANCUT, dither=Image.FLOYDSTEINBERG) \
  .save(G + r'\apple-touch-icon.png', optimize=True)
```

**Removing baked-in elements** (e.g. HUD lines through a logo): render the source on a dark
background with a coordinate ruler, find the rows/columns, and fill them by interpolating between
the pixel rows just above and below. **Stay a few pixels clear of letter edges**, because overlapping
them leaves nubs. Then re-render on dark and zoom in to check.

---

## 7. Page template rules (`<slug>/index.html`)

Start from a copy of `defensetime/index.html`. Keep these rules:

- `<html lang="en">`, charset, viewport, `<title>Privacy Policy — <Game></title>`, a meta description,
  and `theme-color` set to the page background.
- Icons are **relative**: `<link rel="icon" href="favicon.ico" sizes="any">` and
  `<link rel="apple-touch-icon" href="apple-touch-icon.png">`. Don't use `/favicon.ico`, which is the studio icon.
- `color-scheme: dark` (or light) matching the design. Colours only as `:root` tokens.
- Content width 680–720 px, side padding 16 px; must work at **390 px** with no horizontal scroll.
  Add a `@media (max-width:480px)` block (smaller panel padding, stacked intro).
- All `<img>` have `width`/`height` attributes. Decorative images get `alt=""`, and the logo gets
  `alt="<Game name>"`.
- Animations are subtle (rise-in, glow pulse) and fully disabled under
  `@media (prefers-reduced-motion: reduce)`.
- Links: accent colour + 1 px underline border. Text contrast must stay readable over the background.
- Back link `../` at top, studio footer link `../` at bottom.

---

## 8. Add the game to the landing page

In `index.html`, append inside `<ul class="games">` (newest last, or ask the user for the order):

```html
<li>
  <a class="game" href="<slug>/">
    <img class="g-icon" src="assets/games/<slug>.webp" width="144" height="144" alt="">
    <span class="g-text">
      <span class="g-name"><Game Name></span>
      <span class="g-link">Privacy policy</span>
    </span>
    <span class="arrow" aria-hidden="true">&rarr;</span>
  </a>
</li>
```

---

## 9. Verify before showing the user

1. Serve locally, because fonts don't load reliably over `file://`:
   `python -I -m http.server 8765 --bind 127.0.0.1 --directory "F:/Projects/youshgames.github.io"`
   (run it in the background and **stop it afterwards**).
2. Screenshot with headless Chrome at desktop size:
   `chrome --headless=new --hide-scrollbars --force-prefers-reduced-motion --window-size=1280,900
   --virtual-time-budget=3000 --screenshot=out.png http://127.0.0.1:8765/<slug>/`
3. **Phone width:** headless Chrome won't go narrower than ~500 px, so a 390 px window gives a
   misleading cropped shot. Wrap the page in a scratch HTML file with
   `<iframe src="http://127.0.0.1:8765/<slug>/" style="width:390px;height:844px;border:0">` and
   screenshot that.
4. Look at the screenshots: logo artifacts, glow edges, cut-off text, contrast, the font actually loaded.
5. Build a favicon sheet (the `.ico` at 16/32 px on light and dark strips, plus the 180 px touch icon)
   and look at it. Make sure each page shows **its own** icon.
6. `curl -s -o /dev/null -w '%{http_code}' <url>` every external link in the policy (expect 200).
7. `git status` — confirm only the intended files changed, and nothing outside the repo.

Put screenshots in the session scratchpad, never in the repo.

---

## 10. Pitfalls we already hit

- The freeprivacypolicy.com template described location tracking and website browsing that the game
  never did. **Audit, don't template.**
- An installed SDK package isn't proof it's used (Unity IAP in Defense Time). Check the code.
- "Never leaves your device" was wrong for Paint Pool: `allowBackup` sends saves to Google backup.
  Check backup rules on every Android game.
- Policies go stale as games grow (new saved data, new SDKs). On every game update, re-run the
  audit and bump the effective date if anything changed.
- White-on-transparent art hides things. Preview on dark.
- Cropping a glowing logo tightly leaves a visible rectangle. Crop wide and feather.
- The site root `favicon.ico` / `apple-touch-icon.png` are the **studio** icon. Game pages must point
  to their own relative icons.
- Headless Chrome minimum width makes phone screenshots look broken. Use the iframe trick.
- GitHub Pages (Jekyll) publishes `.md` files as pages unless excluded. Add any new doc files to
  `exclude:` in `_config.yml`.
- Keep the policy's "Effective date" current whenever the text changes, and the meta chip with it.
