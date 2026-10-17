# Review brief: favicon, social cards, titles and descriptions

Staging repo for watfordhandyman.uk (Ashton Studios). A health check of the live site on 16 Oct 2026 found four problems. This change fixes them on both HTML pages in the repo: `index.html` (homepage) and `thanks/index.html` (enquiry confirmation). There are no other pages.

Please review the diff. Do not treat this as approved for staging or live.

## How the site is served

GitHub Pages for this repo is a project site, not a user site:

- Staging: `https://ashton1617.github.io/watfordhandyman-staging/` (source branch `main`, path `/`, `.nojekyll` present, no custom domain)
- Live canonical host: `https://watfordhandyman.uk/`

Icon and manifest links are relative, with no leading slash, so the same files resolve on both hosts. The homepage uses `favicon.ico` and `assets/...`. The thanks page is one directory down, so it uses `../favicon.ico` and `../assets/...`. Manifest icon paths are relative to `site.webmanifest`, which sits at the site root in both deployments.

Checked locally: every icon, the manifest, the homepage and `/thanks/` returned HTTP 200 both at `/` and under a `/watfordhandyman-staging/` prefix.

## 1. Favicon

### Problem the health check found

`/favicon.ico` returned 404, and no page had a `<link rel="icon">`.

### Why it matters

A missing favicon shows a broken-image mark in the browser tab, bookmarks and some mobile home screens. That reads as unfinished and weakens brand trust, even when the rest of the page is fine.

### What changed

Built a favicon set from the existing logo in `assets/logo.png`. The logo is a wide lockup: a cyan check (sampled `#00ADEF`), an orange wordmark (about `#F6941F`) and a cyan tagline, on a transparent background. The check alone is about 802×298 pixels. Fitted into 16×16 it collapses into a dash, so the tab icon is a simplified tick in that same cyan, on the site ink colour `#161410` (`--ink` in the page CSS). The full logo is used for the social image (section 2), not squeezed into the favicon.

| File | What it is |
| --- | --- |
| `favicon.ico` | Multi-size icon at the repo root: 16×16, 32×32 and 48×48 PNG frames. Root placement is what makes `https://watfordhandyman.uk/favicon.ico` resolve. |
| `assets/favicon-16x16.png` | 16×16 PNG |
| `assets/favicon-32x32.png` | 32×32 PNG |
| `assets/apple-touch-icon.png` | 180×180 PNG, opaque background (no alpha) |
| `site.webmanifest` | Name "Watford Handyman", short name "Handyman", those three PNGs, `theme_color` and `background_color` `#161410`, `display` `browser`, `start_url` `.` |

Linked from `index.html` and `thanks/index.html`:

- `rel="icon"` → `favicon.ico` (sizes 16, 32 and 48)
- `rel="icon"` → 32×32 PNG and 16×16 PNG
- `rel="apple-touch-icon"` → 180×180 PNG
- `rel="manifest"` → `site.webmanifest`

Homepage hrefs are site-relative (`favicon.ico`, `assets/...`, `site.webmanifest`). Thanks page hrefs are `../favicon.ico`, `../assets/...`, `../site.webmanifest`.

### How to verify

```bash
file favicon.ico assets/favicon-16x16.png assets/favicon-32x32.png assets/apple-touch-icon.png
python3 -c "from PIL import Image; print(Image.open('favicon.ico').info['sizes']);
print(Image.open('assets/favicon-16x16.png').size, Image.open('assets/favicon-32x32.png').size, Image.open('assets/apple-touch-icon.png').size)"
# expect {(16, 16), (32, 32), (48, 48)} and (16, 16) (32, 32) (180, 180)

grep -n 'rel="icon"\|apple-touch-icon\|rel="manifest"' index.html thanks/index.html
```

After this is on a host (not before; this PR is not merged):

- Live: `https://watfordhandyman.uk/favicon.ico` should be 200, not 404.
- Staging tab icon: `https://ashton1617.github.io/watfordhandyman-staging/favicon.ico`
- Thanks page: view source on `/thanks/` and confirm the icon hrefs start with `../`.

A request to `https://ashton1617.github.io/favicon.ico` (the GitHub user-site root, outside this project) will still 404. Browsers that honour `<link rel="icon">` will use the project path instead.

## 2. Open Graph and Twitter Cards

### Problem the health check found

No Open Graph or Twitter Card tags, so a shared link has no title, description or image preview.

### Why it matters

Facebook, LinkedIn, iMessage, Slack and Twitter/X use these tags for the card. Without them the share is a bare URL, which lowers click-through and looks unlike a finished local business.

### What changed

Both pages now have:

- `og:title`, `og:description`, `og:type` (`website`), `og:url`, `og:image`
- `og:image:width` (`1200`), `og:image:height` (`630`), `og:image:alt`
- `twitter:card` (`summary_large_image`), `twitter:title`, `twitter:description`, `twitter:image`, `twitter:image:alt`

`og:title` / `twitter:title` match that page's `<title>`. `og:description` / `twitter:description` match that page's meta description. Values are in sections 3 and 4.

`og:url` uses the live domain, not the staging host:

- Homepage: `https://watfordhandyman.uk/`
- Thanks: `https://watfordhandyman.uk/thanks/`

`og:image` and `twitter:image` on both pages:

`https://watfordhandyman.uk/assets/og-image.png`

That file is `assets/og-image.png`, 1200×630. It is the existing `assets/logo.png` centred on `#161410`. Alt text: "Watford Handyman — the professional alternative to DIY, established 2005".

### How to verify

```bash
file assets/og-image.png
python3 -c "from PIL import Image; im=Image.open('assets/og-image.png'); print(im.size, im.mode)"
# expect (1200, 630) RGB

grep -n 'og:\|twitter:' index.html thanks/index.html
```

After deploy to the live domain, paste `https://watfordhandyman.uk/` into Facebook's Sharing Debugger or Twitter's Card Validator. Until then, `https://watfordhandyman.uk/assets/og-image.png` 404s because this file is not on the live site yet. See "Left open" below.

## 3. Title length

### Problem the health check found

The homepage `<title>` was about 95 characters. Aim is 50 to 60. Counts below are the decoded title (what a browser and Google use). The old homepage title stored the ampersand as `&amp;` in the HTML; that is one character, `&`.

### Why it matters

Google cuts long titles in the search result, often mid-phrase. A title in the 50–60 character range is more likely to show in full, which is the line people click. Each page needs its own title so the confirmation page is not indexed or shared as a second homepage.

### What changed

`index.html` and `thanks/index.html` only. No other pages exist.

| Page | | Text | Characters |
| --- | --- | --- | --- |
| Homepage | Before | Watford Handyman — professional handyman, decorating & flat-pack assembly in Watford since 2005 | 95 |
| Homepage | After | Watford Handyman \| Decorating Services, Hertfordshire | 53 |
| Thanks | Before | Got your list — Watford Handyman | 32 |
| Thanks | After | Enquiry received \| Watford Handyman, Hertfordshire | 50 |

The thanks title was short, not long. It is now inside the same 50–60 range and no longer repeats the homepage title. Keywords kept: handyman, Watford, Hertfordshire, and services (services is in the homepage title; on the thanks page it is in the description).

### How to verify

```bash
python3 - << 'PY'
from html.parser import HTMLParser
from pathlib import Path
import html
class P(HTMLParser):
    def __init__(self):
        super().__init__(); self.in_title=False; self.title=''
    def handle_starttag(self, tag, attrs):
        if tag=='title': self.in_title=True
    def handle_endtag(self, tag):
        if tag=='title': self.in_title=False
    def handle_data(self, data):
        if self.in_title: self.title+=data
for path in ['index.html','thanks/index.html']:
    p=P(); p.feed(Path(path).read_text()); t=html.unescape(p.title).strip()
    print(f'{len(t):3d}  {path}  {t}')
PY
```

Expect 53 and 50, both between 50 and 60 inclusive.

## 4. Meta description length

### Problem the health check found

The homepage meta description was about 196 characters. Aim is 150 to 160. The thanks page had no meta description.

### Why it matters

The meta description is the snippet under the title in search results. Past about 160 characters it is truncated, so the price and the call to action were at risk of being cut. A unique description per page tells searchers which URL they are opening.

### What changed

| Page | | Text | Characters |
| --- | --- | --- | --- |
| Homepage | Before | Watford Handyman — professional handyman, decorating & flat-pack assembly in Watford since 2005. Family-run, award-winning, no job too small. First hour £50, full day £240. Book a handyman online. | 196 |
| Homepage | After | Professional handyman services in Watford, Hertfordshire. Decorating, flat-pack assembly, repairs and odd jobs since 2005. First hour £50 — book online. | 152 |
| Thanks | Before | (no meta description) | 0 |
| Thanks | After | Your Watford Handyman enquiry is with us. We will email a time and a price for decorating, repairs, flat-pack and other Hertfordshire services shortly. | 151 |

Both new descriptions include handyman, Watford, Hertfordshire and services, and they are different from each other.

### How to verify

```bash
python3 - << 'PY'
from html.parser import HTMLParser
from pathlib import Path
import html
class P(HTMLParser):
    def __init__(self):
        super().__init__(); self.desc=None
    def handle_starttag(self, tag, attrs):
        a=dict(attrs)
        if tag=='meta' and a.get('name')=='description':
            self.desc=html.unescape(a.get('content',''))
for path in ['index.html','thanks/index.html']:
    p=P(); p.feed(Path(path).read_text())
    print(f'{len(p.desc):3d}  {path}  {p.desc}')
PY
```

Expect 152 and 151, both between 150 and 160 inclusive.

## Robots and noindex — left exactly as they were

Not part of the four fixes. Nothing in `robots.txt` changed. It is still:

```
User-agent: *
Disallow: /
```

Homepage still has both of these, in this order:

```html
<meta name="robots" content="noindex,nofollow">
<meta name="robots" content="index, follow">
```

The thanks page still has both of these:

```html
<meta name="robots" content="noindex,nofollow">
<meta name="robots" content="noindex, nofollow">
```

The homepage canonical is still `https://watfordhandyman.uk/`.

```bash
git diff -- robots.txt
grep -n 'name="robots"' index.html thanks/index.html
```

`git diff -- robots.txt` should be empty.

## Reviewer checklist

- [ ] `favicon.ico` is at the repo root and contains 16, 32 and 48.
- [ ] `assets/favicon-16x16.png` is 16×16, `assets/favicon-32x32.png` is 32×32, `assets/apple-touch-icon.png` is 180×180 and opaque.
- [ ] `site.webmanifest` is valid JSON and its icon `src` values have no leading slash.
- [ ] Homepage icon and manifest hrefs have no leading slash. Thanks page hrefs use `../`.
- [ ] Both pages have `og:title`, `og:description`, `og:type`, `og:url`, `og:image`, and `twitter:card` = `summary_large_image` plus matching twitter title, description and image.
- [ ] `og:url` is `https://watfordhandyman.uk/` on the homepage and `https://watfordhandyman.uk/thanks/` on the thanks page.
- [ ] `assets/og-image.png` is 1200×630.
- [ ] Homepage title is 53 characters and description is 152. Thanks title is 50 and description is 151.
- [ ] The two titles differ, and the two descriptions differ.
- [ ] `robots.txt` is unchanged. Both robots meta tags on each page are unchanged, including the homepage pair `noindex,nofollow` and `index, follow`.
- [ ] No pages other than `index.html` and `thanks/index.html` were missed (`find . -name '*.html'`).

## Left open / unsure

- The tab icon is a simplified tick in the logo's cyan (`#00ADEF`) on `#161410`, not a scaled copy of `assets/logo.png`. The logo check is too wide to read at 16px. The social image does use the real logo file. If the owner wants the wordmark in the favicon as well, it will not be legible at tab size.
- Logo file colours are not the CSS tokens. CSS orange is `#FFA21C` and navy is `#1a1580`. The logo pixels are cyan `#00ADEF` and orange about `#F6941F`. The favicon follows the logo file. The background follows `--ink`.
- `favicon.ico` also includes 48×48, which was not in the health check. It is inside the `.ico`, not a separate file, so Windows has a sharper icon.
- `og:image` points at `https://watfordhandyman.uk/assets/og-image.png`. That URL will 404 until this file is published on the live domain. I did not point it at the GitHub Pages host, because the card should match the canonical site. Previews of staging links will miss the image until the live site has the file.
- Browsers that ignore `<link rel="icon">` and request the host root will still 404 on `https://ashton1617.github.io/favicon.ico`. This repo cannot place a file on the user-site root. The live root domain is covered by `favicon.ico` at the repo root.
- The homepage has two conflicting robots tags (`noindex,nofollow` and `index, follow`), plus `robots.txt` `Disallow: /`. I left that conflict as I found it. I do not know which one is meant to survive when a later change goes live. Google generally applies the stricter rule, so this staging copy should stay out of the index.
- The thanks page still has no `<link rel="canonical">`. It never had one. `og:url` is set. I did not add a canonical.
- The homepage JSON-LD `description` is still the longer original sentence. The health check was about the meta description, so the structured-data text was not rewritten.
