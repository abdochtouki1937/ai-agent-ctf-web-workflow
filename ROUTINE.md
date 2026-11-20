# The Web Security Routine — the full repeatable routine for every web target

**Date:** 2026-11-20
**Built from:** the 49 routine workflows in `COLLECTION.md` §I (OWASP WSTG, ASVS 5.0.0,
OWASP Top 10:2025, OWASP API Top 10 2023, PTES, HackTricks, PayloadsAllTheThings,
SecLists, PortSwigger Academy, Bugcrowd methodology docs, NahamSec's routine, API
checklists, business-logic guides) + the 8 saved skill files in `skills/`
(`COLLECTION.md` §J).
**Companion guide:** `WORKFLOW.md` is the same method tuned for CTF web machines
(find the flag fast). This file is the same routine tuned for real web targets:
bug bounty programs, pentests, and your own lab apps. The steps are the same;
the proof bar is higher (a finding counts only when you can show real impact).

**Rule 0, before anything else: only test what you are allowed to test.**
Your own lab, a CTF target, a bug bounty program's in-scope assets, or a pentest
with a signed scope. Never point this routine at a live site you have no
permission for. If the scope is unclear, ask first — testing out of scope is
how reports get rejected and accounts get banned.

---

## The routine in one paragraph

Confirm the scope and the rules. Recon the target (subdomains, URLs, tech,
JavaScript, secrets) with deterministic tools first. Map the app like its
developer: endpoints, auth model, roles, business flows. Test login and
sessions. Test inputs one vuln class at a time, cheapest hypothesis first.
Test the API the same way with two accounts. Prove every candidate with a
second tool and a second account, then chain it to real impact. Report it
short, with exact reproduction steps. Log what you tried, rotate when stuck,
and stop a dead end after 45 minutes.

**Time budget per target (first pass):** ~30 min scope + recon, ~30 min mapping,
~2–3 h testing (one vuln class per 20-minute block), ~30 min proving, ~30 min
reporting. A full first pass fits in half a day. Depth comes on the second
pass, not the first.

---

## Step 0 — Scope & rules (10 minutes, never skip)

**Skill:** `skills/claude-bughunter-bb-methodology-SKILL.md` (Part 0: Mode Confirmation)

1. **Write down the engagement type as the first line of your notes.**
   Bug bounty, pentest, red team, or CTF/lab — the same target produces a
   different report in each mode. Bug bounty counts impact-demonstrated bugs
   only; hygiene alone (old software, permissive CSP, stack traces) gets
   rejected. A pentest counts hygiene too. Getting this wrong is the single
   biggest waste of time.
2. **Read the scope, out of scope, and safe harbor.** Copy the in-scope
   domains into your notes. Note rate limits, forbidden test types (usually:
   no DoS, no spam, no social engineering), and whether automated scanning is
   allowed.
3. **Define today's goal in one sentence:** "Today I target [feature/domain]
   to achieve [confidentiality / integrity / availability / account takeover /
   RCE]." Pick **1–2 vuln classes** for the session. No wandering.
4. **Set up two accounts** (A = attacker, B = victim) wherever the target
   allows it. Most access-control bugs (IDOR, privilege escalation) need two
   accounts to prove. If you only have one account, note it — some tests
   cannot be proven.
5. **Start your notes file.** Every endpoint, every payload, every result.
   The notes are what turn a lucky hit into a filed report.

**Free vs paid:** nothing here costs money. Scope is text; notes are a file.

---

## Step 1 — Recon: maximise the attack surface (30 minutes)

**Skills:** `bb-methodology` (Phase 1), `ctf-web`
**Sources:** OWASP WSTG (information gathering), PTES Phase 1 (intelligence
gathering), NahamSec's routine, HackTricks recon sections.

Do this with **deterministic tools first, agents second**. An agent is slow
and expensive at listing subdomains; `subfinder` is fast and free.

1. **Subdomains:** `subfinder -d target.com` → resolve with `dnsx` → probe
   live hosts with `httpx`. Note the tech stack (`httpx -td` / Wappalyzer /
   `whatweb`). Rule: nothing interesting after 5 minutes on a host → skip it,
   try the next one.
2. **URLs:** `gau` + `waybackurls` for archived URLs (forgotten endpoints
   live here) → `katana` to crawl the live app → `uro` to deduplicate.
   Archived URLs are where old, weakly-protected endpoints hide.
3. **JavaScript:** download every JS bundle. Grep for API routes, hidden
   endpoints, and secrets (`jsluice`, `trufflehog --only-verified` — only
   verified keys count, the rest is noise). JS files are the best map of an
   app's real attack surface.
4. **Ports & known software:** `naabu` for a fast top-1000 port sweep.
   If the target runs known software (WordPress, Jira, GitLab), check its
   version against known CVEs immediately — a 1-day bug is the fastest win.
5. **Cloud & third parties:** look for S3 buckets, Firebase, and third-party
   integrations (Stripe, Auth0, Intercom). Test their permissions, and check
   whether webhooks verify their signature.

**What to write down:** live subdomains + tech stack, all URLs, JS-derived
endpoints, known-software versions, cloud resources. If recon finds nothing
after the sweep, the target is either small or well-hidden — move to mapping
with what you have instead of reconning forever.

**Free vs paid:** subfinder, httpx, dnsx, katana, gau, waybackurls, naabu,
nuclei, trufflehog — all free and open source. Shodan/Censys have free tiers
with limits; paid plans are not needed for this step.

---

## Step 2 — Map the app: understand it like its developer (30 minutes)

**Skills:** `bb-methodology` (Phase 2), `ctf-web`
**Sources:** OWASP WSTG (configuration & deployment management, identity
management testing), PortSwigger Academy, Bugcrowd methodology docs.

1. **Use the app as a normal user first.** (NahamSec's routine, and the
   single most skipped step.) Click everything. Upload a file. Change your
   password. Buy something, or go through checkout. You cannot break business
   logic you have never seen work correctly.
2. **Map all endpoints.** Burp/Caido sitemap + the JS analysis from Step 1.
   Note every endpoint, its method, and its parameters — including hidden
   parameters (`arjun`, ParamMiner).
3. **Identify the auth model.** Cookie, JWT, OAuth, SAML, API key? Note how
   sessions work, how long tokens live, and what happens on logout.
4. **Identify roles and permissions.** User, admin, API keys, service
   accounts. Draw the matrix: which role can do what.
5. **Find the business-critical flows.** Payment, registration, password
   reset, data export, coupon/discount, file sharing. Complex flows
   (coupon + points + refund) are where edge-case bugs live.
6. **Note anomalies.** A parameter named `user_id` everywhere but `userId`
   in one place. A 403 with a different JSON shape. A dev/staging host with
   debug headers. Anomalies are the scent of a bug — log them, then test them.

**What to write down:** endpoint inventory, auth model, role matrix,
business flows, anomaly list. If a new API endpoint appears later while
testing, come back here and map it before attacking it.

**Free vs paid:** Burp Community is free (no active scanner, but the proxy
and Repeater are enough for this routine). Caido has a free tier. Burp Pro
($449/year) adds the scanner and Collaborator — useful, not required;
`interactsh` is the free OOB alternative.

---

## Step 3 — Login & sessions: authentication and session testing

**Skills:** `offensive-jwt`, `bb-methodology` (Phase 3)
**Sources:** OWASP WSTG (authentication testing, session management testing),
ASVS 5.0.0 (V2 authentication, V3 session management), PortSwigger Academy
(authentication labs).

Test in this order — cheapest first:

1. **Login basics.** Username enumeration (does "user not found" differ from
   "wrong password"?), rate limiting / lockout, default or weak credentials,
   verbose error messages. Time-based user enumeration needs n ≥ 10
   interleaved trials and a ≥ 2σ difference — a single slow response is
   network jitter, not a finding.
2. **Password reset.** Request a reset for account B with account A's
   session. Is the token guessable, reusable, or leaked in the response? Does
   the reset link work twice? Does changing the password kill old sessions?
3. **MFA / 2FA.** Can you skip it by navigating straight to `/dashboard`?
   Is the OTP brute-forceable, reusable, or the same for every user? Does a
   backup code work forever?
4. **Session handling.** Does logout actually invalidate the token? Does the
   session survive a password change? Is the cookie missing `HttpOnly`,
   `Secure`, or `SameSite` — and does that actually matter here (it only
   matters if you can already run script or force a request)?
5. **JWTs (if the app uses them).** Decode the token. Then run the
   `offensive-jwt` checklist in order: `alg:none` (try `none`, `None`,
   `NONE`, `nOnE`), RS256→HS256 confusion (re-sign with the public key as
   the HMAC secret), weak HMAC secret (`jwt_tool.py <token> -C -d
   wordlist.txt`), `kid` injection (`../../../../dev/null`,
   `' OR 1=1 --`), `jku`/`x5u`/`jwk` header injection, missing `exp`/`aud`/
   `iss` validation. Never brute-force a secret on a live target without
   permission to do so — in a bounty, the token is yours to test; the
   rate limit is the program's.
6. **OAuth / SSO.** Test the redirect URI, the `state` parameter (missing
   `state` = CSRF on login), and token confusion (ID token sent where an
   access token is expected, and vice versa).

**Prove it:** a session bug is only real if account A can act as account B,
or a logged-out token still works. "The cookie lacks a flag" alone is not a
finding in a bug bounty.

---

## Step 4 — Inputs: test one vuln class at a time

**Skills:** `hunt-xss`, `hunt-xxe`, `offensive-sqli`, `ctf-web`,
`bb-methodology` (Phase 3)
**Sources:** OWASP Top 10:2025, OWASP WSTG (input validation testing),
PortSwigger Academy, HackTricks, PayloadsAllTheThings, SecLists.

**Method for every class:** error-based first, then blind. Send `'`, `"`,
`{{7*7}}`, `${7*7}` and watch for 500 errors and stack traces. No error?
Try time-based (`SLEEP(10)`). No time difference? Try out-of-band
(`interactsh`). Still nothing? Try boolean (`AND 1=1` vs `AND 1=0`).
**Rotate every 20 minutes** — no progress on one endpoint means move to the
next endpoint, the next subdomain, or the next vuln class. **Cap: 45 minutes
per parameter.** Longer than that is a rabbit hole.

### 4a. Cross-site scripting (XSS)

**Skill:** `hunt-xss` (built from 174 disclosed reports).

- **Canary first:** send a unique random string (8+ chars, e.g.
  `cpmark987abc`) and check it appears in the response. Then check the
  baseline (no-marker) response — if your marker appears naturally in the
  page, it is a false positive and you need a different marker. This one
  check kills 80% of false-positive reflection claims.
- **Encoding is everything:** `<script>` in the response is XSS;
  `&lt;script&gt;` is safe encoding, not a bug.
- **Contexts in order:** inline script injection, attribute event injection
  (`" onmouseover="`), URL/href context (`javascript:`).
- **Stored XSS** pays more than reflected: profile fields, comments, ticket
  titles, wiki pages — anything another user (especially an admin) views.
- **Blind/stored XSS needs an OOB callback** (Collaborator/interactsh). No
  callback, no claim. Plant beacons early (error messages, `User-Agent`,
  `Referer`, usernames) and keep the listener open — admin views can fire
  hours later.
- **Validate in a real browser** before reporting. Burp echo ≠ browser
  execution (CSP, framework escaping, parsing differences).
- **Chain it:** reflected XSS + cache poisoning → stored at CDN scale;
  self-XSS + CSRF → account takeover; DOM XSS on `/oauth/callback` →
  token theft. Standalone `alert(1)` is Low–Medium; a chain to ATO is High.

### 4b. SQL injection (SQLi)

**Skill:** `offensive-sqli`.

- **Detect:** basic probes (`'`, `"`, `OR 1=1`), error-based (syntax errors
  leak DB type), boolean-based (true vs false response difference),
  time-based (`SLEEP(5)` / `pg_sleep(5)` / `WAITFOR DELAY`).
- **Identify the DB, then enumerate:** column count via `UNION SELECT NULL`,
  string columns, `@@version` / `version()`, `information_schema.tables`,
  `information_schema.columns`.
- **Modern surfaces:** JSON operators, GraphQL filters, WebSocket messages,
  REST filter objects (`$regex`, `$ne`) — injection lives wherever input
  reaches a query, not just in URL parameters.
- **Automate, then verify by hand:** `sqlmap -r req.txt --dbs` or `ghauri`
  for blind. Never report a sqlmap hit you have not reproduced yourself.
- **WAF bypass:** case variation, comments (`UN/**/ION`), encoding,
  string concatenation. Combine tamper scripts for layered WAFs.

### 4c. XML external entities (XXE)

**Skill:** `hunt-xxe` (built from 10 disclosed reports).

- **Fingerprint the parser first.** Most modern parsers are safe by
  default (Python `ElementTree`, Ruby Nokogiri, .NET 4.5.2+). Java SAX/DOM
  without hardening and PHP `DOMDocument` with `LIBXML_NOENT` are the
  usual suspects. Do not waste time on a hardened parser.
- **Pre-severity gate:** run the inline-entity probe (`<!ENTITY hello
  "world!">` then `&hello;`). If `hello!` does not echo back, entity
  expansion is disabled and `SYSTEM file://` will not work either.
- **Entry points:** file uploads (SVG, DOCX, XLSX, PPTX — all XML under
  the hood), SOAP/WSDL, SAML endpoints, any API that silently accepts
  `Content-Type: application/xml` (try swapping a JSON request to XML).
- **Blind XXE needs an OOB callback** with actual file content exfiltrated.
  A DNS ping alone is Low/Medium — escalate to file read or internal
  HTTP (metadata endpoint) for Critical.

### 4d. Access control (IDOR / BOLA / BFLA) — the highest-yield class

**Skills:** `bb-methodology` (Phase 3), `ctf-web`.

- **The test is always the same:** take an object ID from account A, replay
  the request with account B's token. If B can read, change, or delete A's
  object, it is IDOR. Sequential IDs (`/users/101`, `/orders/102`) are the
  first place to look; UUIDs need a leak source first.
- **Vertical:** can a normal user call an admin endpoint
  (`/admin/deleteUser`)? Try it directly — frontend hiding is not access
  control.
- **BFLA:** can a normal user call a function meant for admins (export,
  approve, refund) even if the button is hidden?
- **Mass assignment:** add fields the UI never sends (`"role":"admin"`,
  `"price":1`, `"is_admin":true`) to profile-update and checkout requests.
- **Always test with two accounts and compare bodies, not just status
  codes.** A 200 with a byte-identical body to the baseline is not a
  bypass — diff the body (`diff <(curl … baseline) <(curl … bypass)`).

### 4e. Server-side request forgery (SSRF), template injection (SSTI), file upload

- **SSRF:** URL inputs, webhooks, PDF generators, image fetchers. Point at
  `http://169.254.169.254/latest/meta-data/` and internal hosts. DNS-only
  is not a finding — reach the metadata or an internal service.
- **SSTI:** `{{7*7}}` → `49` means template evaluation. Escalate per
  engine (Jinja2, Twig, Freemarker) — see `ctf-web` and HackTricks.
- **File upload:** SVG with `<script>` (stored XSS), HTML files served
  without `Content-Disposition: attachment`, path traversal in filenames,
  extension confusion (`.php` disguised as `.jpg`).
- **Business logic:** the most under-tested class. Write the rules of the
  flow first (price comes from the catalog, quantity is positive, a coupon
  applies once, no confirmation without payment), then break one rule at a
  time: skip a step, repeat a step, reverse steps, use 0 / negative / huge
  numbers, replay a coupon. Check the resulting *state*, not just the HTTP
  status. (Source: Hackita business-logic method; Colorful White invariant
  method — both in `COLLECTION.md` §I.F.)

### 4f. Everything else, in rotation

Race conditions (send the same request 10× at once — coupon, checkout,
vote), open redirect (`?next=javascript:…`, OAuth flows), CSRF (state-changing
request with no token), cache poisoning (unkeyed header reflected in a
cacheable response), CORS misconfiguration (reflected `Origin` +
`Access-Control-Allow-Credentials: true`), HTTP request smuggling (only if
the scope allows it — it can break the target for other users).

---

## Step 5 — APIs: same tests, two accounts, machine-readable

**Skills:** `offensive-sqli` (GraphQL/REST), `offensive-jwt`, `ctf-web`
**Sources:** OWASP API Top 10 2023, API checklists in `COLLECTION.md` §I.G.

1. **Inventory the API.** Endpoints, versions (`/api/v1` vs `/api/v2` —
   the old version often has weaker auth), parameters, GraphQL schema
   (introspection enabled?).
2. **Authorisation matrix.** For every endpoint × every role (anonymous,
   user A, user B, admin, service account): who can call it, and what
   happens. BOLA (one user's object) and BFLA (one role's function) are
   the top two API bugs in the OWASP API Top 10.
3. **Auth & tokens.** Expired tokens, tokens from another app, missing
   `aud`/`iss` checks, API keys in URLs (they end up in logs).
4. **Input & mass assignment.** Same as Step 4, but JSON-shaped: extra
   fields, type confusion (`"id":"1"` vs `"id":1`), array injection.
5. **Rate limiting & abuse.** Login, OTP, password reset, coupon, voting —
   try 10 rapid requests and watch for a 429. No limit is a finding only
   when it enables something (account takeover via OTP brute force, not
   "I can send many requests").
6. **GraphQL specifics.** Introspection, deeply nested queries (DoS),
   field-level authorisation (a field that returns another user's data).

---

## Step 6 — Prove & escalate: turn a candidate into a finding

**Skills:** `bb-methodology` (Phase 4), `hunt-xss` (Chains),
`hunt-xxe` (Related Skills)
**Sources:** the discipline rules in `bb-methodology` Part 4.

A candidate is not a finding until it passes these gates:

1. **Reproduce with a second tool.** curl + Python `requests`, or Burp +
   Python — different HTTP stacks. A curl-only timing difference that
   disappears under `requests` is an artefact, not a bug. Paste-into-shell
   reproduction commands go in the report.
2. **Reproduce with a second account.** The victim account sees the impact
   (their data read, their email changed, their session stolen).
3. **Marker discipline.** Unique random markers (8+ chars, no English
   words). Check the baseline first. `test`, `marker`, `evil`, `payload`,
   `javascript`, `script` are forbidden as markers — they appear naturally
   in pages.
4. **Body-diff rule.** A bypass claim needs a response *body* difference,
   not just a status-code change. Byte-identical body = no bypass.
5. **Statistical-sample rule.** Any timing claim needs n ≥ 10 interleaved
   trials per group and a ≥ 2σ difference. One slow response is jitter.
6. **Escalate to real impact.** XSS → steal a cookie, force an email
   change, or take the account. IDOR → read PII at scale, or change a
   password. SSRF → reach metadata, extract keys. If the impact is only
   "this looks wrong", find the chain or drop it — in a bug bounty it will
   come back N/A.
7. **If stuck 4+ hours on one candidate that will not reproduce on a
   second account, it is dead.** Document the retraction in your notes
   and move on. Do not sink another 4 hours into it.

**If you find a Critical in the first 30 minutes, stop reconning.**
Validate it fully, report it, then return to recon.

---

## Step 7 — Report: short, exact, impact-first

**Skills:** `bb-methodology` (Phase 5)
**Sources:** reporting labs and templates in `COLLECTION.md` §I.H.

1. **Run the 7-question gate first** (from `bb-methodology` Phase 5 /
   `triage-validation`): is the impact real, is the reproduction exact,
   is it in scope, is it a duplicate, is the severity honest? Any fail →
   kill the finding. Do not waste the triager's time.
2. **Title:** `[Bug Class] in [Endpoint] allows [role] to [impact]`.
3. **Sentence 1 = what the attacker CAN do.** Not what the bug is — what
   it lets an attacker do.
4. **Steps to reproduce:** exact HTTP requests, paste-ready. Under 600
   words for the whole report. CVSS 3.1 score that matches the actual
   impact — not the maximum the class could theoretically have.
5. **After submission:** while waiting for triage, try to escalate further
   (one bug usually has siblings — same developer, same mistake, 20 more
   minutes). If a fix is deployed, re-test for a bypass — an incomplete
   patch is a new bug.
6. **Log it.** Record the finding, the failed attempts, and the "weird but
   not yet exploitable" behaviours. Failed attempts stop you re-testing
   the same dead end next week.

---

## Daily / weekly rhythm (how to run this routine every day)

| When | What |
|---|---|
| Start of session (10 min) | Step 0: scope, goal sentence, 1–2 vuln classes, notes file |
| First pass (1 h) | Steps 1–2: recon + map; write the endpoint inventory and anomaly list |
| Testing blocks (20 min each) | Step 4/5: one vuln class per block; rotate on no progress |
| End of session (15 min) | Step 6: prove the day's best candidate, or mark it dead; update notes |
| When a finding is proven | Step 7: report it the same day — do not let proven findings sit |
| Weekly | Re-check the target's changelog / new release; a new feature is a new attack surface (same rule as CTF: always work on the latest version) |

**Rotation rules that keep the routine honest:** 20-minute rotation clock
(no progress → rotate endpoint → subdomain → vuln class → target); 45-minute
rabbit-hole cap per parameter; 5-minute rule per host in recon; 2 weeks /
30 hours on one program with no findings → consider switching programs.

---

## Skill map — which saved skill covers each step

| Routine step | Skill file | Load it when |
|---|---|---|
| Step 0 — scope & rules | `claude-bughunter-bb-methodology-SKILL.md` | Start of every session |
| Step 1 — recon | `claude-bughunter-bb-methodology-SKILL.md` (Phase 1), `ctf-skills-ctf-web-SKILL.md` | Listing subdomains, URLs, JS |
| Step 2 — map the app | `claude-bughunter-bb-methodology-SKILL.md` (Phase 2) | Before testing anything |
| Step 3 — login & sessions | `claude-red-offensive-jwt-SKILL.md`, `ctf-skills-ctf-web-SKILL.md` | Any JWT, OAuth, or session test |
| Step 4a — XSS | `claude-bughunter-hunt-xss-SKILL.md` | Any reflected/stored/DOM XSS test |
| Step 4b — SQLi | `claude-red-offensive-sqli-SKILL.md` | Any injection test on a DB-backed input |
| Step 4c — XXE | `claude-bughunter-hunt-xxe-SKILL.md` | Any XML input or file upload |
| Step 4d — access control | `claude-bughunter-bb-methodology-SKILL.md` (Phase 3), `ctf-skills-ctf-web-SKILL.md` | Any two-account test |
| Step 4e — SSRF / SSTI / upload / logic | `ctf-skills-ctf-web-SKILL.md` | URL inputs, templates, uploads, business flows |
| Step 5 — APIs | `claude-red-offensive-sqli-SKILL.md`, `claude-red-offensive-jwt-SKILL.md` | REST / GraphQL testing |
| Step 6 — prove & escalate | `claude-bughunter-bb-methodology-SKILL.md` (Part 4) | Before writing any report |
| Step 7 — report | `claude-bughunter-bb-methodology-SKILL.md` (Phase 5) | Writing the report |
| Browser / Electron targets | `offensive-claude-browser-exploitation-SKILL.md` | Client-side targets only |
| Any CTF challenge | `ctf-skills-solve-challenge-SKILL.md` | CTF mode — see `WORKFLOW.md` |

---

## Free vs paid — what this routine costs

| Need | Free | Paid (only if you can afford it) |
|---|---|---|
| Agent / model | OpenCode + DeepSeek / Kimi / GLM / Qwen free tiers; Claude free tier (limited) | Claude Pro/Team/Max, or Anthropic API key |
| Proxy & repeater | Burp Community, Caido free tier | Burp Pro ($449/yr: active scanner + Collaborator) |
| Recon | subfinder, httpx, dnsx, katana, gau, naabu, nuclei, ffuf, arjun — all free | Shodan/Censys paid tiers (not needed) |
| OOB callbacks | interactsh (self-hosted, free) | Burp Collaborator (needs Pro) |
| Wordlists & payloads | SecLists, PayloadsAllTheThings, HackTricks — free | — |
| Learning labs | PortSwigger Academy (free), HackTheBox free tier, TryHackMe free rooms | Burp Pro, BSCP exam ($99) |
| Reporting | Markdown + the template in `COLLECTION.md` §I.H | — |

**If you have $0, you can run this whole routine.** The only thing money
buys is speed (a scanner, a stronger model). The findings come from the
method, not the subscription.

---

## What to copy / what to avoid (from the 49 routine sources)

**Copy:**

- NahamSec's order: use the app as a normal user first, map functionality,
  then recon. A mental checklist beats testing vuln classes at random.
- The `bb-methodology` 5-phase loop with the discipline gates — it is the
  difference between 11 impact-demonstrated bugs and 5 hygiene findings
  that get rejected.
- The business-logic invariant method: write the rules, break one rule at
  a time, check state not status.
- The API identity matrix: anonymous → user A → user B → admin → service
  account, for every endpoint.
- The two-account, two-tool proof bar before any report.

**Avoid:**

- Testing vuln classes at random with no goal sentence for the session.
- Reporting hygiene as a finding in a bug bounty (EoL software alone,
  permissive CSP alone, stack traces, info disclosure with no concrete
  impact).
- Status-code-only bypass claims — always diff the body.
- Single-shot timing claims — n ≥ 10 interleaved trials or it is jitter.
- Non-unique markers (`test`, `payload`, `javascript`) — they cause the
  false positives that burn your reputation with triagers.
- Staying on one dead parameter for hours. Rotate at 20 minutes, drop it
  at 45.
- Pointing any of this at a target you have no permission to test.

---

## Sources

- Full routine source list (49 items, all verified, all free):
  `COLLECTION.md` §I — "Web security routine" (OWASP WSTG Latest, ASVS 5.0.0,
  OWASP Top 10:2025, OWASP API Top 10 2023, OWASP cheat sheets, PTES,
  HackTricks, PayloadsAllTheThings, SecLists, PortSwigger Academy,
  checklists, Bugcrowd methodology docs, blog walkthroughs, business-logic
  guides, API guides, recon/reporting, video walkthroughs).
- Saved skill files (8, verbatim, with source + license headers):
  `skills/` — see `skills/README.md` and `COLLECTION.md` §J.
- CTF web-machine version of this routine: `WORKFLOW.md`.
- Full index of everything collected (135 rows): `collection.csv`.
