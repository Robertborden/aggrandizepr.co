# Aggrandize PR — DNS Records (Phase 1)

Paste these into your DNS provider (Cloudflare, Namecheap, GoDaddy, Google Domains, etc.).
Set them AFTER you've created the Google Workspace account and inboxes.

Replace `aggrandizepr.co` with your domain if different. TTL = 3600 (or "Auto").

---

## 1. MX Records (mail routing — Google Workspace)

| Type | Host / Name | Value / Points to                         | Priority | TTL |
|------|-------------|-------------------------------------------|----------|-----|
| MX   | @           | `aspmx.l.google.com`                      | 1        | 3600 |
| MX   | @           | `alt1.aspmx.l.google.com`                 | 5        | 3600 |
| MX   | @           | `alt2.aspmx.l.google.com`                 | 5        | 3600 |
| MX   | @           | `alt3.aspmx.l.google.com`                 | 10       | 3600 |
| MX   | @           | `alt4.aspmx.l.google.com`                 | 10       | 3600 |

> Google may also provide an `aspmx.l.google.com` + backup `alt1-4` set in the admin console —
> use exactly what the Google Workspace setup wizard shows. Remove any old MX records from the prior registrar.

---

## 2. SPF (TXT) — authorizes Google to send mail

Type: `TXT`
Host / Name: `@`
Value:
```
v=spf1 include:_spf.google.com ~all
```
> `~all` = softfail (recommended while learning). Move to `-all` once you're 100% sure only Google sends for this domain.

---

## 3. DKIM (TXT) — generated INSIDE Google Workspace

DKIM is NOT something you invent. After you create the Workspace account:
1. Admin Console → Apps → Google Workspace → Gmail → Authenticate email (SPF, DKIM, DMARC)
2. Select your domain → Generate a DKIM key (1024 or 2048-bit)
3. Google shows you a TXT record. It looks like:

Type: `TXT`
Host / Name: `google._domainkey`
Value (example — Google gives you the real one):
```
v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A... (long string from Google) ...
```
4. Paste it into DNS, then click **Start authentication** back in the admin console.

> You cannot finalize DKIM until Google gives you the key. Leave this until after Workspace setup.

---

## 4. DMARC (TXT) — start at p=none (monitor only)

Type: `TXT`
Host / Name: `_dmarc`
Value (PHASE 1 — monitoring only):
```
v=DMARC1; p=none; rua=mailto:dmarc@aggrandizepr.co; ruf=mailto:dmarc@aggrandizepr.co; sp=none; adkim=s; aspf=s; rf=afrf; pct=100; ri=86400
```
> Create a `dmarc@aggrandizepr.co` inbox to receive daily reports.
> After 2–4 weeks of cleanDMARC reports, tighten:  `p=quarantine` → later `p=reject`.

Recommended progression:
- Weeks 1–4 (warm-up): `p=none`
- Month 2: `p=quarantine; pct=25` (then raise pct to 100)
- Month 3+: `p=reject` (only when confident all legit mail is authenticated)

---

## 5. Optional but recommended

**BIMI (brand logo in inbox)** — once DMARC = `p=quarantine`/`reject` and you have a logo:
Type: `TXT`, Host: `default._bimi`, Value: `v=BIMI1; l=https://aggrandizepr.co/assets/logo.svg;`

**Google Workspace verification** — Google gives you a TXT record like `google-site-verification=XXXX`.
Add it exactly as shown during setup.

---

## Verify after 24–48h

Run these (free):
- https://mxtoolbox.com → check MX, SPF, DMARC
- https://www.dmarcanalyzer.com/dmarc/dmarc-record-check
- Send a test mail to `check-auth@verifier.port25.com` and read the report it emails back.

Expected: SPF = pass, DKIM = pass, DMARC = pass.
