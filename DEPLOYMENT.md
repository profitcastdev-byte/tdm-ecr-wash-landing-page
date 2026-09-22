# Deploying to the Profitcast KVM

| | |
| --- | --- |
| Server | Profitcast KVM: Hostinger VPS `root@187.127.149.216` (srv1575430, Ubuntu 24.04, nginx 1.24) |
| Review link | <https://tdmecr-wash-preview.187.127.149.216.nip.io> (live now, HTTPS, kept out of Google with noindex) |
| Live address | <https://wash.lp.thedetailingmafiaecr.com> (waiting on one DNS record, see *Going live*) |
| Files | `/var/www/wash.lp.thedetailingmafiaecr.com`, previous release at `.prev` |
| nginx vhost | `/etc/nginx/sites-available/wash.lp.thedetailingmafiaecr.com`, source in `deploy/nginx/` |
| Certificate | certbot lineage `wash.lp.thedetailingmafiaecr.com`, renews automatically |

Set up the same way as the PPF page (`TDM ECR PPF - Landing Page`), whose
`DEPLOYMENT.md` this follows. SSH is key-based from this PC
(`~/.ssh/id_ed25519`). Deploying from another machine needs that machine's
public key added to `/root/.ssh/authorized_keys` on the server first.

---

## Updating the page

From the project root, in PowerShell or cmd:

```powershell
.\deploy-kvm.cmd              # upload, swap in, verify
.\deploy-kvm.cmd --check      # is the KVM running exactly this page? (changes nothing)
.\deploy-kvm.cmd --rollback   # put the previous release back (run again to undo)
```

From Git Bash, macOS or Linux: `bash deploy/deploy-kvm.sh [--check | --rollback]`.

A deploy:

1. Refuses to start if `canonical`, `og:url` or `og:image` in `index.html` do
   not point at `https://wash.lp.thedetailingmafiaecr.com/`.
2. Uploads only the 21 files the page serves: `index.html` and `assets/`, minus
   the unused logo master. README, the audit report, this file, `deploy/` and
   the zip never leave this machine.
3. Unpacks beside the live folder and swaps it in, keeping the previous release
   as `.prev`, so visitors never load a half-uploaded page.
4. Compares a checksum of every file on both ends, then prints the HTTP status
   of both addresses.

It only ever touches the site's files. It never edits nginx, reloads it or
changes certificates.

**Edited `style.css` or `main.js`?** Bump `?v=` on both links in `index.html`
(currently `?v=9`) before deploying. The server tells browsers to keep CSS and
JS for a day.

**Replaced a photo?** Give the new file a new name and update the reference.
Photos are cached for 30 days, so a file swapped under the same name stays old
for returning visitors.

After any change, rebuild the deploy zip and commit it with the change (see
`.gitignore`).

---

## Going live on wash.lp.thedetailingmafiaecr.com

The domain's DNS is managed at Hostinger. The main website stays exactly where
it is; only names under `.lp` point at the KVM.

1. **Add one DNS record**, if it is not there already. hPanel → Domains →
   thedetailingmafiaecr.com → DNS / Nameservers:

   | Type | Name | Points to | TTL |
   | --- | --- | --- | --- |
   | `A` | `*.lp` | `187.127.149.216` | default |

   This is the same wildcard the PPF page is waiting on: one record serves
   `ppf.lp`, `wash.lp` and any later landing page. On 18 September 2026 it did
   not exist yet (neither name resolved at 8.8.8.8).

2. **Wait until public DNS returns the KVM:**

   ```powershell
   Resolve-DnsName wash.lp.thedetailingmafiaecr.com -Server 8.8.8.8
   ```

3. **Add the live name to the certificate**, on the server:

   ```bash
   ssh root@187.127.149.216
   certbot --nginx --non-interactive --redirect --expand --cert-name wash.lp.thedetailingmafiaecr.com -d tdmecr-wash-preview.187.127.149.216.nip.io -d wash.lp.thedetailingmafiaecr.com
   ```

   This fails until step 2 passes. Until it runs, `http://wash.lp.thedetailingmafiaecr.com`
   answers 404. That is certbot's placeholder, not a fault.

4. **Check:** `.\deploy-kvm.cmd --check` should show 200 for both addresses.

5. **Point the Google Ads final URLs** at `https://wash.lp.thedetailingmafiaecr.com/`,
   then make one real Call click and one WhatsApp click from a phone and
   confirm both land in Google Ads → Goals → Conversions.

If an office PC shows **ERR_SSL_PROTOCOL_ERROR** right after the switch while a
phone on mobile data loads the page fine, the office router is serving a cached
DNS answer. The site is fine.

---

## How the server was set up (18 September 2026)

Recorded so it can be rebuilt. It does not need running again.

```bash
# 1. vhost: the HTTP-only source; certbot adds :443 and the redirect itself
ssh root@187.127.149.216 'set -o noclobber; cat > /etc/nginx/sites-available/wash.lp.thedetailingmafiaecr.com' < deploy/nginx/wash.lp.thedetailingmafiaecr.com.conf

# 2. on the server: enable, test, reload
ln -s /etc/nginx/sites-available/wash.lp.thedetailingmafiaecr.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

# 3. from this PC: the files
.\deploy-kvm.cmd

# 4. on the server: HTTPS for the review link
certbot --nginx --non-interactive --redirect --cert-name wash.lp.thedetailingmafiaecr.com -d tdmecr-wash-preview.187.127.149.216.nip.io
certbot renew --dry-run --no-random-sleep-on-renew --cert-name wash.lp.thedetailingmafiaecr.com
```

Verified after setup:

- `nginx -t`: no errors and no conflicting server names; no other vhost claims
  either name
- HTTP redirects to HTTPS; Let's Encrypt certificate valid to 17 December 2026,
  renewal dry run passes
- `nosniff` and `Referrer-Policy` on every response: page, CSS, JS, images, video
- gzip on HTML, CSS and JS
- Cache: page revalidated on every visit, CSS/JS 1 day, photos and video 30 days
- `X-Robots-Tag: noindex` on the review link; by config the live name sends none
  (checkable once its DNS resolves)
- The video streams: range requests answer 206 with `video/mp4`
- Dotfiles return 403; README, audit report, this file, `deploy/`, the zip and
  the logo master return 404
- All 17 assets the page references return 200; GA4 and both Ads conversion
  labels are in the served files
- Deploy, `--check` and `--rollback` (twice, back to the same release) all
  exercised; files owned by `www-data`, 644/755
- Four neighbouring sites (the PPF review link, ppf.tdmhyderabad.in,
  lp.meditarina.in, the bare IP) returned 200 before and after

The vhost is the PPF one with the names changed and one addition: a 30-day
cache rule for `/assets/video/`. The page carried a video until 22 September
2026 and does not now, so that rule currently matches nothing; it is left in
place for the next one. It caches with `expires` rather than `add_header` for the reason
given in the PPF `DEPLOYMENT.md`: a location that declares any `add_header`
silently drops the ones it would inherit from the server block.

---

## Rules for this server

It is shared production: about 55 live client sites.

- **`nginx -t` before every `systemctl reload nginx`.** Reloading a broken
  config takes every site down. If the test fails, remove what you just added
  before anything else.
- `nginx -t` prints three *could not build optimal server_names_hash*
  warnings. They predate this site and are harmless; the last line
  (*test is successful*) is what counts.
- **Never add `default_server`.** `rentla-preview` holds it, and moving it
  changes where every unmatched hostname on the box lands.
- **Scope certbot dry runs** with `--cert-name wash.lp.thedetailingmafiaecr.com`.
  Without it certbot simulates every certificate on the box, which takes many
  minutes and holds a lock. Add `--no-random-sleep-on-renew` as well: run over
  SSH without a terminal, certbot otherwise sits through a random delay of up
  to 8 minutes before it starts.
- `grep -r` over `sites-enabled/` silently finds nothing, because the entries
  are symlinks. Use `grep -H pattern /etc/nginx/sites-enabled/*`.

### Taking the site offline

```bash
rm /etc/nginx/sites-enabled/wash.lp.thedetailingmafiaecr.com
nginx -t && systemctl reload nginx
```

Files, vhost and certificate stay in place, so bringing it back is the `ln -s`
line from setup step 2, then `nginx -t` and reload.
