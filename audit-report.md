# Pre-launch QA: TDM ECR Car Wash Landing Page

**Date:** 18 September 2026
**Page:** `index.html` + `assets/` (static site, no build step)
**Traffic:** Google Ads, mostly phones

## Verdict: DEPLOY-READY ✅ YES

No blocking issues. One loading problem was found and fixed during this QA. Five
checks remain that can only be done on the live URL, listed at the end.

## What was checked

| Area | Result | What that means |
|---|---|---|
| Conversion tracking | ✅ Pass | GA4 (`G-4Q3YJM42GR`) and Google Ads (`AW-10990978713`) load on every page view. Every one of the 13 call and WhatsApp buttons fires exactly one conversion with the right label: 8 phone buttons fire **PC - LP - Phone Call Click**, 5 WhatsApp buttons fire **PC - LP - WhatsApp Click**, each 1.0 INR. Tested by clicking every button with sending intercepted, so no test conversions reached the Ads account. |
| Lead capture | ✅ Pass | No form, by the client's request. Leads come in as calls and WhatsApp messages, and both are tracked as conversions. |
| Mobile | ✅ Pass | 11 layout breakpoints. Rendered at 375, 768, 1024, 1280, 1440 and 1920 px wide: no sideways scrolling at any size. On phones the logo is centred and the Call / WhatsApp bar stays pinned to the bottom of the screen. |
| Loading speed | ✅ Pass | Measured on a clean load with an empty cache: **about 320 KB on a phone** and about 590 KB on desktop, fonts included. The hero image is preloaded and sized per device (88 KB on phones, 127 KB on desktop), downloaded once. The 3.7 MB video and the 9 gallery images load only when needed. |
| Search and sharing | ✅ Pass | Title 58 characters, description 159 (both inside search-result limits), one H1, canonical link, Open Graph tags for WhatsApp / Facebook previews, favicon. |
| Accessibility | ✅ Pass | Every image has alt text, the page language is set, there is a skip-to-content link, every button has a name, headings are in order, and the red headline keeps at least 3:1 contrast over the photo at every screen size. |
| Code health | ✅ Pass | No broken HTML, no script errors, all 18 files the page uses are present, no insecure `http://` links, links that open a new tab are safe. |

## Fixed during this QA

1. **The video no longer downloads during the first load.** It used to start
   streaming as soon as it came within 200px of the screen, and on a typical
   900px-tall screen it already is at load. So a 3.7 MB stream competed with
   the hero image before anyone had scrolled, and visitors who bounced paid
   for it in mobile data. It now starts only once it is actually on screen.
   The poster is the video's first frame, so the switch is invisible. Verified
   in a clean browser: nothing is downloaded at the top of the page, the video
   plays when scrolled to, and pauses when scrolled past.

## Notes on the automated scan

- The scanner first reported two false warnings (no media queries, no
  responsive-image rule): it could not open the stylesheet through the page's
  cache-busting `?v=` link. Re-run on a copy with those tags removed, both
  cleared.
- Two warnings are left on purpose. The logo and hero image load immediately
  rather than lazily, because they are the first thing on screen and
  lazy-loading them would slow the page. `hero-banner.jpg` stays JPEG because
  it is the WhatsApp / Facebook preview image and those apps do not reliably
  show WebP.

## To do before and after going live

These cannot be checked from the files.

1. **Add one DNS record at Hostinger before launch.** The page's address is
   `https://wash.lp.thedetailingmafiaecr.com/`, following the Profitcast KVM
   convention (the PPF page is `ppf.lp.`), and its canonical and link-preview
   URLs point there. That name only resolves once the wildcard record
   `A *.lp -> 187.127.149.216` exists; the same record serves the PPF page.
   Until then the page is reachable on its preview address only.
2. **Test the conversions on the live URL.** Open it with Google Tag Assistant,
   tap Call and WhatsApp once each, and confirm both conversions register. In
   Google Ads (Goals > Conversions) both actions should show "Recording
   conversions" within a day.
3. **Run PageSpeed Insights on the live URL.** Lighthouse is not installed on
   this machine, so the page weight was measured directly instead.
4. **Server settings.** Turn on gzip or brotli compression for HTML, CSS and JS,
   and long cache headers for images and the video.
5. **Share the live URL once on WhatsApp** to confirm the preview shows the Urus
   photo.

## Known, not blocking

- **Grey section labels.** The small labels in the footer and above the "Car
  Wash & Detailing" heading show grey and oversized instead of small red
  capitals, because of a style override inherited from the PPF page. Cosmetic
  only; readable at 7:1 contrast.
- **Mirrored hero photo.** The "Lamborghini" script on the car's tail reads
  backwards. Mostly under the dark shading on desktop, fully visible on phones.
- **Consent.** Analytics and Ads tags load on arrival. That is fine for Indian
  traffic; EU or UK traffic would need a consent banner first.
