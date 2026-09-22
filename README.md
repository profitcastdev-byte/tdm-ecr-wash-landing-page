# The Detailing Mafia ECR | Car Wash Landing Page

Static landing page for the ECR car wash campaign. No build step: open `index.html`,
or serve the folder with any static host (Netlify, Vercel, S3, plain nginx).

```
index.html
assets/css/style.css
assets/js/main.js
assets/img/     logo, favicon, hero banner, intro photo, gallery (work-01 to work-12)
deploy/         KVM deploy script and nginx vhost (see DEPLOYMENT.md)
deploy-kvm.cmd  runs the deploy from PowerShell or cmd
```

## Deploying

Hosted on the Profitcast KVM, set up like the PPF page:

- Review link: <https://tdmecr-wash-preview.187.127.149.216.nip.io>
- Live address: <https://wash.lp.thedetailingmafiaecr.com> (once the `*.lp` DNS record exists)

Update it with `.\deploy-kvm.cmd` from the project root. Everything about the
server, going live and the rules for that box is in `DEPLOYMENT.md`. The
ready-to-upload zip is kept in the repo for hosts without SSH.

Built to match the PPF landing page at https://tdmecr-ppf.netlify.app/ so the two
campaigns read as one brand. Content is from `ECR LP Content (2).pdf`.

## Design

- **Type:** Manrope (400/500/600/700/800), loaded from Google Fonts.
- **Palette:** sampled from `Logo - ECR.png`, which is only three colours:
  `#FF0000`, `#000000`, `#FFFFFF`. On light bands the red drops to `#C40000`
  so body-size text clears 4.5:1; pure `#FF0000` only passes on black.
- **Surfaces:** the dark surface is the default. Any section given
  `class="surface-light"` re-declares the same tokens and everything inside it
  recolours itself. Never hardcode a text or border colour, use a token.

## Images

All supplied by the client (`ECR.zip`, `TDM - Wash - ECR - image.zip`) and
converted for the web:

| Slot | File | Size | Notes |
|---|---|---|---|
| Hero banner | `hero-banner.webp`, `hero-banner-1280.webp` | 1672 x 941, 1280 x 720 | The client's replacement shot (`Hero Bannr.jpg`). Served with `srcset`, so phones get the smaller file. Preloaded in `<head>`. On desktop it is sized to the hero's height and anchored right; on phones it is cropped to 4:3, aimed 87% across where the car sits. |
| Link preview | `hero-banner.jpg` | 1200 x 630 | The `og:image`. JPEG because WhatsApp and Facebook previews do not reliably render WebP. |
| Intro photo | `intro-wash.webp` | 1400 x 1050 (4:3) | In the Car Wash & Detailing section. Cropped to 16/10 once the section stacks, so it does not tower over the copy. |
| Gallery (9) | `work-01` to `work-08`, `work-12` | 900 x 675 | No captions on the cards, by request. Marques alternate so no two of the same sit side by side in the 3-up view. |

A 4:5 video of a foam wash held the intro slot until 22 September 2026, when the
client sent this photo for it instead. The video, its poster and the code that
played them are in the git history if they are ever wanted back.

Gallery photos are numbered from `work-01`, not `work-1`, and the nine in use
skip from `work-08` to `work-12`. Both follow one rule: the server caches photos
for 30 days, so a name that has been out there never gets different content.
New pictures get new names, and dropped ones leave their numbers unused rather
than shuffling the rest along.

## Changing phone, WhatsApp or address

Edit the `CLIENT` block at the top of `assets/js/main.js` only. Every phone,
WhatsApp and map link on the page (header, hero, cards, footer, floating buttons,
mobile bar) is rewritten from it at runtime, so nothing can drift out of sync.

The `href`s in `index.html` are the no-JS fallback and should stay correct too,
but the `CLIENT` block is what actually ships to visitors.

## Tracking

`index.html` loads one gtag base tag for GA4 + Google Ads. Conversion labels live
in the `ADS` block in `main.js`, not in the HTML.

Do not paste Google's per-conversion snippets into the head. Each one defines the
same `gtag_report_conversion` function, so the second silently overwrites the
first and every conversion reports as whichever loaded last. That is why the
labels are kept as data and fired through one function.

Installed, TDM ECR's own tracking:

| What | ID / label | Fires on |
|---|---|---|
| GA4 | `G-4Q3YJM42GR` | every page view |
| Google Ads account | `AW-10990978713` | loaded with the page |
| PC - LP - Phone Call Click | `AW-10990978713/T8MzCIy3obEcEJmN9Pgo`, 1.0 INR | a click on any phone link (8 on the page) |
| PC - LP - WhatsApp Click | `AW-10990978713/Sf9YCMPEobEcEJmN9Pgo`, 1.0 INR | a click on any WhatsApp link (5 on the page) |

Links are matched by `href` (`tel:` and `wa.me/`), so any phone or WhatsApp
button added later is tracked automatically.

## Responsive behaviour

| Width | Behaviour |
|---|---|
| Desktop | Logo left, Call Now in the header, hero CTA pair, floating Call (bottom left) and WhatsApp (bottom right), 3 gallery slides per view |
| <= 1080px | 2 gallery slides per view |
| <= 980px | Hero copy goes full width over the banner |
| <= 900px | Intro, why-us cards and location stack to one column; the intro photo switches to a 16/10 crop so the section stays in balance |
| <= 640px | Logo centred and alone in the header, hero CTA pair removed (the intro and final banner keep theirs, full width), sticky Call + WhatsApp bar pinned to the bottom, floating buttons lift above it, 1 slide per view |

The floating buttons appear once the visitor has scrolled past 300px. The sticky
bar does not wait, because the phone hero has no CTA of its own, and it stays
pinned all the way down, footer included.

## Notes on the copy

Content is taken from the brief. Three deliberate departures, all flagged:

1. **The hero mini-form was dropped** on request, so the FAQ answer "Fill out the
   form above, call us, or WhatsApp us directly" would have pointed at something
   that is not there. It now reads "Call us or WhatsApp us directly, and we'll
   confirm your slot."
2. **Long dashes were removed** on request. Where the brief used one as a
   connector it became a comma or a full stop, and the H1 splits across two lines
   instead. No wording was cut.
3. **Section 3 (gallery) carries no copy in the brief**, so the heading and lede
   were written to match the voice of the rest of the page. The cards carry no
   captions, by request.
