# DNS Pulse & Dig

A single-file DNS lookup tool that runs entirely in the browser. It queries Google Public DNS and Cloudflare 1.1.1.1 over DNS-over-HTTPS (JSON API), so there is no backend to run.

## What it does (Phase 1)

- Resolves **A, AAAA, CNAME, MX, TXT** (plus NS and SOA) via DoH
- Parses A/AAAA, MX (sorted by priority), CNAME chains, and TXT with **SPF / DKIM / DMARC** highlighting
- **DNSSEC indicator** from the AD bit (with honest wording: AD means "the resolver validated it", not "the zone is signed")
- **Simulated trace**: real NS data for each zone from root down, through your chosen resolver
- **Compare resolvers**: Google vs Cloudflare side by side, with an agree/differ flag
- **Propagation matrix**: 10 regions, *simulated* (seeded per domain, clearly labeled, not real probes)
- **Copy buttons** for an equivalent `dig` command and a BIND-style zone snippet
- **Shareable URLs**

## Run locally

```bash
# any static server works
python -m http.server 8080      # then open http://localhost:8080
# or
npx serve .
```

Opening `index.html` directly from disk also works for lookups.

## URL parameters

| Param | Meaning | Example |
|---|---|---|
| `d` | domain | `example.com` |
| `t` | type: A AAAA CNAME MX TXT NS SOA | `MX` |
| `r` | resolver: `google` or `cloudflare` | `cloudflare` |
| `m` | mode: `standard` `trace` `compare` | `trace` |
| `o` | comma list: `do`, `cd`, `short` | `do,short` |
| `ecs` | client subnet (Google only) | `203.0.113.0/24` |
| `q` | compact form: base64url of `{"d":"example.com","t":"MX","o":["do"]}`; wins over the others | `eyJkIjoi...` |

Example: `?d=gmail.com&t=MX&r=google&o=do,short`

The page deep-links: if `d` or `q` is present it resolves immediately.

## Deploy to Netlify

There is nothing to build and no server-side service to start. "Starting the service" on Netlify just means publishing the static site.

### Option A: Git-connected (recommended)

1. Push this repo to GitHub (see below).
2. Log in at <https://app.netlify.com> and choose **Add new site → Import an existing project**.
3. Pick **GitHub**, authorize Netlify, and select the `DNS-Checker` repo.
4. Build settings: leave **Build command** empty and set **Publish directory** to `.` (these already come from `netlify.toml`, so the form usually pre-fills).
5. Click **Deploy**. The first deploy takes under a minute and gives you a `*.netlify.app` URL.
6. Every push to `main` now redeploys automatically, and pull requests get deploy previews.

Optional: **Site configuration → Domain management** to rename the subdomain or add a custom domain (HTTPS is automatic).

### Option B: Netlify CLI

```bash
npm install -g netlify-cli
netlify login
netlify init            # link or create a site (pick manual or GitHub-connected)
netlify dev             # local dev server at http://localhost:8888, same headers as production
netlify deploy          # draft deploy to a preview URL
netlify deploy --prod   # publish to production
```

### Option C: Drag and drop

Go to <https://app.netlify.com/drop> and drop the project folder. Quickest, but no auto-deploys.

### After deploying

- Open the site and run `example.com A`. You should get an answer in well under a second.
- Open DevTools → Console. If there are no Content-Security-Policy warnings, open `netlify.toml` and rename `Content-Security-Policy-Report-Only` to `Content-Security-Policy` to enforce it, then push.

## Push this code to your repo

```bash
git clone https://github.com/arulmk/DNS-Checker.git
cd DNS-Checker
# copy index.html, netlify.toml, README.md, .gitignore into this folder
git add .
git commit -m "Add DNS Pulse & Dig Phase 1 MVP"
git push origin main
```

## Troubleshooting

| Symptom | Likely cause |
|---|---|
| "Could not reach Google/Cloudflare" | Ad-blocker, corporate proxy or firewall blocking `dns.google` / `cloudflare-dns.com` |
| Page is unstyled | Tailwind CDN blocked, or an enforced CSP missing `https://cdn.tailwindcss.com` |
| ECS box is greyed out | Cloudflare's JSON API has no client-subnet support; switch to Google |
| DNSSEC shows "not requested" | Turn on the DO bit |
| Propagation matrix looks "too perfect" | It is simulated by design. Real regional probing is optional Phase 3 |

## Roadmap

**Phase 2**: binary `application/dns-message` parsing (EDNS, NSID, additional section), rich SRV/PTR/DS/DNSKEY/CAA/SOA parsers, trace against real authoritative servers, batch mode with tabbed comparison, accessibility and cross-browser pass, and a compiled Tailwind stylesheet in place of the CDN.

## Notes

- Tailwind's CDN build is for prototyping. It is fine for this MVP; compile CSS for production-grade use.
- DNS answers are untrusted input. The UI never uses `innerHTML` for them.
- The `dig` command is the classic-DNS equivalent, so results can differ slightly from DoH.
