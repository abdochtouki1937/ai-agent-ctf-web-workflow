# Strong Workflow: Solving CTF Web Machines with AI Agents (Claude + DeepSeek)

**Date:** 2026-11-20
**Built from:** the collection in `COLLECTION.md` (234 items collected, 68 with fully verified URLs in `collection.csv`).
**Goal:** CTF only — solve web machines and anything web-related in CTF (web exploitation, web recon, web crypto, web forensics). Authorised targets only: CTF platforms, HackTheBox/TryHackMe machines, and your own lab. Never point an agent at a live site you do not own or have written permission to test.

## The workflow in one paragraph

Start with a plain coding agent (Claude Code) on a Kali VM, load the `ctf-skills` pack, and give the agent a short written brief (`CLAUDE.md`). Do deterministic recon first (scripts and standard tools), let the agent reason only where a human would have to think, validate every finding yourself, and keep a notes file per challenge. Use cheap models (DeepSeek, Gemini Flash) for bulk work and Claude Opus for the hard steps. When the agent fails, do not re-prompt harder — write the missing technique into a small skill file and run again. That failure→skill loop is the whole method: it took Transilience from 89.4% to 100% (104/104) on web CTFs.

## 0. Before you start — rules and safety (5 minutes, never skip)

1. **Check the event's AI rules.** LakeCTF bans generative AI entirely. UMassCTF splits into no-AI and LLM-allowed divisions. RITSEC 2026 disqualified ~100/800 teams (hallucinated flags, near-instant solves, hollow writeups). HTB events default to AI-disabled unless the admin enables it. If the rules ban AI, do not use it — no workflow is worth a disqualification.
2. **Isolate the agent.** Run it in a Docker container or a VM (Kali), never on your main machine with your real credentials. Agents run shell commands: PentAGI workers run without approval, dsh-superagent workers run as your OS user, HexStrike AI has no auth at all (unauthenticated server on 127.0.0.1:8888). Give the agent its own user, its own API keys, no Docker socket, no access to your files.
3. **Never pull random Docker images.** ARTEX images were backdoored (~7,946 pulls, repo pulled after real bank breaches). Use only official images (Kali, Ubuntu) or images you built yourself.
4. **Never let an agent touch other people's repos.** The hackerbot-claw incident (Feb–Mar 2026): an agent opened 12+ pull requests against 7 big open-source repos, got RCE in 4, destroyed one repo, and published a malicious VSCode extension. No agent opens PRs, no agent pushes code anywhere except your own lab.
5. **Budget.** Set a hard spend cap per challenge (e.g. $1–3). Easy HTB box ≈ $3 with PentestGPT; a directed web CTF challenge is usually <$1; a full live CTF run can reach ~$95. Free tiers die mid-event (ChatGPT free credits ran out during Huntress CTF 2026) — have a fallback model ready.

## 1. Set up the lab (once, ~30 minutes)

- **Machine:** Kali Linux VM (or Docker container). Kali alone added +9.5 points on the NYU CTF Bench vs plain Ubuntu — the tools matter.
- **Tools pre-installed:** nmap, ffuf, gobuster, sqlmap, nuclei, Burp Suite (Community), python3 + requests/httpx/pwntools, gdb, Ghidra, Radare2, git, curl, jq.
- **Agent:** Claude Code (the baseline — plain Claude Code beats most specialised frameworks). Install the `ctf-skills` pack: `npx skills add ljagiello/ctf-skills`. Optional second pack: `Claude-BugHunter` as a Claude Code plugin (`/plugin marketplace add elementalsouls/Claude-BugHunter`) for web technique patterns from 681 disclosed reports.
- **DeepSeek path:** install Pi agent (`@earendil-works/pi-coding-agent`) or DeepSeek Harness (`npx @deepseek-ai/dsh web`) with a DeepSeek API key. Pi was the best harness for DeepSeek V4 Flash in community testing (66.7% pass, $0.028/success — cheapest). DeepSeek Harness is still a developer preview — expect breaking changes.
- **Free/cheap fallback:** OpenRouter `:free` models, Groq free tier, Gemini API free tier (Flash only, 250 req/day). Local: Qwen2.5-Coder-32B or Devstral 24B via Ollama for bulk triage only — they are far weaker than frontier models.
- **Folders per challenge:** `challenge-name/` with `notes.md`, `solve.py`, `writeup.md`, `evidence/` (raw tool output). The agent writes everything here; nothing else.

## 2. Write the brief — `CLAUDE.md` / `AGENTS.md` (the most important file)

Copy the pattern that solved FLARE-On 13 (all 9 in ~6h) and Insomni'hack. Put this in the challenge folder before the agent starts:

```
You are solving a CTF web challenge. The target is <URL/IP> and no other.
Rules:
- Never run a challenge binary directly; use Ghidra/Radare2 for reversing.
- Classify first, then triage, then try the cheapest hypothesis first.
- Keep notes, scripts and artifacts in this folder; one solve.py + writeup per challenge.
- A finding is only real if it appears verbatim in tool output (byte-level proof).
- Never exfiltrate, never touch anything outside the target. Stop and ask if unsure.
- Work in this order: recon -> map the app -> hypothesise the vuln class -> test one payload at a time -> prove it -> extract the flag -> write it up.
```

Keep it short. A short brief the model actually follows beats a long one it ignores.

## 3. Step-by-step: solving one web machine

### Step 1 — Recon, deterministic first (you + scripts, ~10 min)

Do not let the agent guess yet. Run the standard tools and save raw output to `evidence/`:

1. `nmap` the target (ports, services, versions).
2. `ffuf` / `gobuster` for directories and files; `katana` or a small crawler for links, forms, JS files.
3. Fetch `robots.txt`, `sitemap.xml`, `/.git/HEAD`, JS bundles; grep JS for API endpoints, keys, comments.
4. Identify the stack (headers, cookies, error pages, Wappalyzer-style fingerprinting).
5. Write the map into `notes.md`: URLs, parameters, forms, auth flows, interesting headers/cookies.

**Model:** none, or Haiku / DeepSeek Flash for summarising tool output. This step is mechanical — do not spend Opus tokens on it.

### Step 2 — Classify (agent, cheap model)

Give the agent `notes.md` + the raw recon and ask: "Classify this target. What vuln classes are plausible, ranked by likelihood? Cheapest test first." Typical web classes: SQLi, XSS (reflected/stored/DOM), SSTI, SSRF, IDOR/BOLA, auth bypass, JWT flaws, file upload, LFI/RFI, prototype pollution, XXE, open redirect, CSRF, business-logic flaws.

**Model:** Sonnet for recon/classification, or DeepSeek V4 Flash for bulk (Rogdham solved two real web challenges with DeepSeek 4.1 Flash + Pi for ~$0.03 each). Load the matching `ctf-web` skill file for the top class.

### Step 3 — Hypothesise and test one payload at a time

For each hypothesis, in order:

1. The agent writes a small test script (Python `requests`/`httpx`) — never hand-crafted curl one-liners you cannot re-run.
2. Run it, save output to `evidence/`.
3. Judge: did the response change in a way that proves the vuln (error, timing, reflected value, different object returned)? If not, record the negative result in `notes.md` and move on — negatives are knowledge, not waste.
4. If the test needs a bypass (WAF, filter), use the `ctf-web` skill files (keyword fragmentation, double encoding, case variation, alternate encodings) — one technique at a time.

**Model routing:** Sonnet/DeepSeek for routine tests; escalate to Opus only when two cheap attempts fail on the same hypothesis. Opus is 59% on NYU CTF Bench vs DeepSeek-V3's 6.5% — the gap is real, but so is the price ($2.12/solve vs $0.01–0.02).

### Step 4 — Prove it, then exploit

- **Byte-level proof rule (from VulnClaw):** the flag or the proof must appear verbatim in real tool output. If the agent claims a vulnerability, make it show the request and the response that prove it. No proof, no claim.
- Write the exploit as `solve.py` (one file, re-runnable, prints the flag). This is also your writeup artifact.
- **Web crypto:** if the challenge mixes web + crypto (JWT, custom signing, session tokens), switch to the `ctf-crypto` skill and treat the token as a crypto problem: decode first (base64/JWT header), check `alg=none`, weak secrets, key confusion — do not brute-force blindly.
- **Web forensics:** if there is a pcap, log, or image, export it to text first — agents cannot read raw pcapng/binaries well until exported (Copilot failed on 404CTF until the pcap was converted to text). Use `tshark`, `strings`, `exiftool`, then let the agent read the text.

### Step 5 — If stuck: the failure→skill loop (the core of this workflow)

This is the Transilience method, and it is the difference between 89% and 100%:

1. Stop. Write down exactly what failed and what technique was missing (e.g. "missed indirect IDOR via archive endpoint", "didn't test X-UserId header override").
2. Write that technique into a small general skill file (a few lines: when to try it, the exact payload pattern, an example). Keep it general, not tied to this challenge.
3. Run again. Repeat ~15 cycles and the skill pack becomes the reason you win.

Do NOT just re-prompt harder, raise the temperature, or switch to a bigger model first — the missing piece is usually knowledge, not reasoning.

### Step 6 — Validate and write up (human, always)

- Re-run `solve.py` from a clean state. Does it still get the flag? If not, it was luck or state — fix it.
- Write `writeup.md`: target, recon summary, vuln class, the exact request/response proof, the exploit, the flag, and what you learned (which skill file helped / which one you added).
- Check the flag format and submit it yourself. Never let the agent submit flags or interact with a scoreboard.

## 4. Which model for which step

| Step | First choice | Cheap alternative | When to escalate |
|---|---|---|---|
| Recon / summarise tool output | Haiku / Sonnet | DeepSeek V4 Flash, Gemini Flash | — |
| Classify + plan | Sonnet | DeepSeek V4 Flash | Opus if the app is unusual |
| Routine payload tests | Sonnet | DeepSeek V4 Flash / Pi | Opus after 2 cheap failures |
| Hard reasoning (auth logic, crypto, filter bypass) | Opus 4.5/4.6 | Gemini 3 Pro ($0.43/solve) | — |
| Bulk fuzzing / many attempts | DeepSeek V4 Flash, Gemini Flash | Local Qwen2.5-Coder / Devstral | Never use Opus for bulk |
| Reversing a binary (web-adjacent) | Opus + Ghidra MCP | — | — |

**Guardrail warning:** Claude Opus 5.5 blocks or silently downgrades cyber work. If Claude refuses or gives oddly weak answers, that is the guardrail, not the challenge. The official fix is Anthropic's Cyber Verification Program; the practical fallback is DeepSeek / Kimi / GLM / Qwen, which do not block.

## 5. What works and what fails (from the collection)

**Works:** plain agent + skill files; deterministic recon before agent reasoning; one payload at a time with saved evidence; the failure→skill loop; cheap-first model routing; byte-level proof; a written brief; re-running `solve.py` from clean state.

**Fails:** chat-only prompting with no tools; fancy multi-agent frameworks on easy challenges (they cost more and solve less than plain Claude Code); letting the agent run free without a brief or a budget; trusting a claimed flag without tool output; re-prompting the same failed idea; using Opus for bulk fuzzing; trusting self-reported benchmark scores (LuaN1aoAgent "90.4% XBOW", DeepSeek-V4-Fable "63.8% web") without reproduction; practising on OWASP Juice Shop and believing the score (every solution is public).

## 6. Practice plan (first week, free or nearly free)

1. **Day 1:** set up the Kali VM + Claude Code + `ctf-skills` (Section 1). Run the agent on one OverTheWire Natas level (web, free) with the brief from Section 2.
2. **Day 2–3:** five more Natas levels, one skill file added per failure (Section 3, Step 5). Keep the notes.
3. **Day 4:** one retired easy HTB machine (Claude Code + custom CLAUDE.md, like the Cicada run). Budget $3.
4. **Day 5:** one web challenge with DeepSeek only (Pi + DeepSeek V4 Flash, like Rogdham's NdH2k17 run) — learn the cheap path.
5. **Day 6–7:** one picoCTF web challenge set end-to-end with the full workflow; write the writeups.

## 7. Quick reference — the files and links to keep

- Collection: `COLLECTION.md` — CSV: `collection.csv` (same folder).
- Skill pack: https://github.com/ljagiello/ctf-skills
- Second pack: https://github.com/elementalsouls/Claude-BugHunter
- Baseline agent framework: https://github.com/GreyDGL/PentestGPT
- Best blueprint paper: https://raw.githubusercontent.com/transilienceai/communitytools/95fdc128af4ca1ae16b3226f9430f1bad97b0656/papers/practice-makes-perfect.pdf
- Best worked example: https://lbhoe.dev/posts/flare-on-13/
- Cheapest proven DeepSeek recipe: https://www.rogdham.net/2026/11/11/revisiting-my-ndh2k17-challenges.en
- Model numbers: https://arxiv.org/pdf/2604.17159
- Rules check before every event: https://refusalrate.com/ai-for-ctf-learning
