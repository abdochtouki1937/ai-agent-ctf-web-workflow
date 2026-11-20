# AI-Agent CTF Web Workflow — Collection

**Date:** 2026-11-20
**Goal:** CTF only (not bug bounty) — solve CTF web machines and anything web-related in CTF: web exploitation, web recon, web crypto, web forensics.
**Models in focus:** Anthropic Claude, DeepSeek, plus other models people actually use.
**Method:** 12 research batches searched GitHub, X, Reddit, Hacker News, blogs, Medium, Dev.to, YouTube, CTF writeup sites, HackTheBox / TryHackMe writeups, benchmarks and papers. Every URL in the main sections below was returned by a search or fetch on 2026-11-20, or by one of the 12 batch agents and re-verified in a fresh search pass the same day. No URL was invented.
**Counts:** 234 items collected across the 12 batches; ~215 unique after dedup (known overlaps: PentestGPT, Transilience, Rogdham, FLARE-On, Huntress, EnIGMA, HTB MCP, HTB AI-vs-Human, NYU CTF Bench, AIRTBench, DeepSeek V4.1 Flash / Enclave, AI-rules survey, and others appeared in 2–4 batches each). Plus 49 web-security-routine items (§I, topic `web-security-routine` in collection.csv) and 8 saved public skill files + 10 more verified skill collections (§J). Grand total in collection.csv: 135 data rows. This file lists every entry whose URL could be verified in this pass, plus an appendix of named resources whose exact URL was lost when the batch reports were condensed — those are listed WITHOUT a URL rather than with a guessed one.

**Guides built from this collection:** `WORKFLOW.md` — the step-by-step CTF web-machine workflow (solve web machines with Claude + DeepSeek); `ROUTINE.md` — the full step-by-step web security routine for every web target (scope → recon → map → login/sessions → inputs/APIs → prove → report), built from §I + the saved skills; `skills/` — 8 public skill files copied verbatim, with install steps in `skills/README.md`.

## Synthesis — what works, what to copy, what to avoid

### What works (said by many independent sources)

1. **A simple harness + a strong model beats a fancy framework.** Plain Claude Code / a plain coding agent beats most specialised pentest frameworks. Palisade got 95% on InterCode-CTF with "plain agents" (ReAct + planning, multiple attempts). Transilience started at 89.4% with a bare Claude Opus 4.6 and reached 100% (104/104) only by adding skill files. Start with Claude Code as the baseline, not with a framework.
2. **Skill files win.** The single biggest lever after the model is a pack of short, specific markdown skill files. Transilience: 230 skill files took Opus 4.6 from 89.4% to 100% on 104 web challenges. The ready-made pack is `ctf-skills` (web skill alone has 23 files: SQLi, XSS, SSTI, SSRF, XXE, JWT, OAuth, prototype pollution) and `Claude-BugHunter` (83 skills distilled from 681 disclosed HackerOne reports). Copy this pattern: one small file per technique, loaded on demand.
3. **Model routing by task.** Opus = hard reasoning, Sonnet = recon, Haiku = bulk/cheap work (YesWeHack labs pattern). DeepSeek = cheap bulk runs; Claude = the hard steps. Measured gap is large: Claude 4.5 Opus 59% on NYU CTF Bench vs DeepSeek-V3 6.5% and DeepSeek-R1 4.0% (arXiv 2604.17159). DeepSeek is the cheapest per solve ($0.01–0.02) but solves almost nothing hard; Gemini 3 Flash is the best cost-efficiency ($0.05/solve).
4. **Automation first, agents second.** The Huntress CTF winners' pattern: deterministic solvers (scripts, standard tools) as the fast path, agents only where reasoning is needed. "Find fast, validate manually" — a human must check every finding; platforms punish unvalidated AI output.
5. **Real tooling, not chat-only.** A Kali box, file I/O, a terminal, MCP servers (Ghidra, Radare2, Burp), a written `CLAUDE.md` / `AGENTS.md` brief, and per-category skills. Chat-only prompting stalls on hard boxes. The FLARE-On 13 solve used a written rule set (never run a challenge binary, Ghidra for reversing, one `solve.py` + writeup per challenge) and solved all 9 in ~6 hours.

### What to copy

- The `CLAUDE.md` / `AGENTS.md` brief pattern (classify → triage → cheapest hypothesis first → keep notes/scripts/artifacts).
- The Transilience failure loop: run the benchmark, find a failure, diagnose the missing technique, write it into a general skill file, run again (~15 cycles to 100%).
- The ctf-skills install: `npx skills add ljagiello/ctf-skills`, then `/solve-challenge` routes to the right category skill.
- The "byte-level proof" rule from VulnClaw: a claimed flag/vulnerability must appear verbatim in real tool output — no victory without receipts.
- Cost discipline: easy HTB box ≈ $3 with PentestGPT, <$1 per directed web CTF challenge; run cheap models first, escalate to Opus only when stuck.

### What to avoid

- **Claude's cyber guardrails are the main practical blocker.** Opus 5.5 blocks or silently downgrades cyber work; the official fix is Anthropic's Cyber Verification Program. DeepSeek / Kimi / GLM / Qwen have no such block — this is a real reason to keep a DeepSeek path.
- **Never run agents against targets without authorization.** Isolate in Docker/VM. HexStrike AI has no auth and runs shell as the operator (its server listens unauthenticated on 127.0.0.1:8888); PentAGI workers run commands without approval; dsh-superagent workers run as the operator's OS user.
- **ARTEX Docker images are backdoored** (~7,946 pulls, repo pulled after real bank breaches). Never pull random ARTEX images.
- **The hackerbot-claw incident (Feb–Mar 2026):** an agent calling itself "autonomous security research agent powered by claude-opus-4-5" opened 12+ PRs against 7 big open-source repos exploiting GitHub Actions workflows; 4 repos got RCE. Do not point agents at other people's repos, and never let an agent open PRs.
- **Check each CTF's AI rules first.** LakeCTF bans generative AI entirely; UMassCTF 2026 split into no-AI vs LLM-allowed divisions; RITSEC 2026 disqualified ~100/800 teams (hallucinated flags, near-instant solves). HTB events default to AI-disabled unless the admin enables it.
- **Anecdote ≠ proof.** Single-video claims (Grok Code "rooted Ghostlink in 18.5 min", "FLARE-On in 2 hours") are unverified hype until reproduced. Trust benchmarks (NYU CTF Bench, AIRTBench, Cybench) over videos.
- **Self-reported scores.** LuaN1aoAgent claims 90.4% on XBOW at $0.09/exploit via DeepSeek; DeepSeek-V4-Fable claims 63.8% on web — both self-reported, not independently reproduced. Treat as leads, not facts.
- **CAI is ARCHIVED — do not build on it.** Use PentestGPT, PentAGI, HackSynth or Shannon instead.

## Model & cost cheat sheet (measured, not marketing)

| Model | NYU CTF Bench (200 challs) | Cost per solve | Notes |
|---|---|---|---|
| Claude 4.5 Opus | 59% (118/200) | $2.12 | Best overall; cyber guardrails are the blocker |
| Gemini 3 Pro | 52% | $0.43 | Strong second |
| Gemini 3 Flash | 27% | $0.05 | Best cost-efficiency |
| GPT-5.2-Codex | 18% | — | — |
| DeepSeek-V3 | 6.5% | $0.01–0.02 | Cheapest, solves almost nothing hard |
| DeepSeek-R1 | 4.0% / 41% on AIRTBench (29/70) | very cheap | Mid-tier on CTFs; good for bulk |
| Llama 3.3 70B | 2.5% | — | Weak |
| Claude 3.7 Sonnet | 61% on AIRTBench (43/70) | — | Best on AI-security CTFs |

Free tiers that exist (all can die without warning — GitHub Models retired July 2026, Gemini CLI 1,000 req/day ended June 2026): OpenRouter `:free` models (~50 req/day), Groq free tier (~1,000 req/day), Gemini API key (250 req/day, Flash only). Anthropic has NO free API tier. Cheapest paid step: DeepSeek API (V4-Flash ~$0.14/M input tokens). Local free options: Qwen2.5-Coder-32B, Devstral 24B, mythos-sec / NotMythos on Ollama — much weaker than frontier (best open model only 10% on AIRTBench), use for bulk/triage only.

## A. Start here — the 10 most useful resources

1. **ctf-skills** — ljagiello, GitHub, 2026-09. The ready-made skill pack: 12 skills, web skill has 23 files (SQLi, XSS, SSTI, SSRF, XXE, JWT, OAuth, prototype pollution). Free (MIT), install with `npx skills add ljagiello/ctf-skills`. Works with Claude Code, Cursor, Codex, Gemini CLI, Copilot. https://github.com/ljagiello/ctf-skills
2. **Transilience — "Practice Makes Perfect"** — Transilience AI, Mar 2026. Claude Opus 4.6 + 230 skill files = 104/104 web CTF flags (100%); Sonnet 4.6 = 96.2%, Haiku 4.5 = 62.5%. The paper documents the exact failure→skill loop. This is the single best blueprint for the workflow. https://raw.githubusercontent.com/transilienceai/communitytools/95fdc128af4ca1ae16b3226f9430f1bad97b0656/papers/practice-makes-perfect.pdf
3. **PentestGPT** — GreyDGL, GitHub (USENIX Security 2024 paper). The most-used open agent framework; v1.0 is agentic with a Claude Code / Codex backend; legacy mode supports OpenAI, Anthropic, Gemini, DeepSeek, xAI, Qwen, Moonshot. Easy HTB boxes in 30–50 min at ~$3. Free (MIT), bring your own API key; also works with local Ollama models free. https://github.com/GreyDGL/PentestGPT
4. **Solving FLARE-On 13 with Claude Code** — LB, lbhoe.dev, Oct 2026. All 9 challenges in ~6 hours with Claude Code + Opus 5.5 on a Kali VM, under a written `CLAUDE.md` rule set. Full solve scripts published. Copy the rule-set pattern. https://lbhoe.dev/posts/flare-on-13/
5. **NYU CTF Bench benchmark (arXiv 2604.17159)** — Tyler H. Merves et al., Apr 2026. 200 challenges, 10 frontier models: the numbers behind the model cheat sheet above (Opus 59%, DeepSeek-V3 6.5%). Read before choosing a model. https://arxiv.org/pdf/2604.17159
6. **Rogdham — Revisiting my NdH2k17 challenges** — Rogdham, rogdham.net, 2026-11-11. DeepSeek 4.1 Flash + Pi agent solved his two 2017 web challenges from "find the flag" only; easy in 4m54s for $0.031, hard in ~3 min. The cheapest proven DeepSeek web-CTF recipe. https://www.rogdham.net/2026/11/11/revisiting-my-ndh2k17-challenges.en
7. **Claude-BugHunter** — elementalsouls (Sachin Sharma), GitHub, 2026-05. 83 Claude Code skills + 15 slash commands distilled from 681 disclosed HackerOne reports across 24 vuln classes; Burp MCP integration; scope enforced in code. Free (MIT/CC BY 4.0). https://github.com/elementalsouls/Claude-BugHunter
8. **Palisade — Hacking CTFs with Plain Agents** — Palisade Research, Dec 2024. 95% on InterCode-CTF with plain ReAct+Plan prompting — proof that simple beats fancy. Paper + forked harness. https://arxiv.org/html/2412.02776v1 — code: https://github.com/palisaderesearch/intercode
9. **PentAGI** — vxcontrol, GitHub. Fully autonomous multi-agent pentest platform, ~33k stars, 20+ real tools (nmap, sqlmap, Metasploit) in Docker sandboxes, 12+ LLM providers incl. DeepSeek and local Ollama. Free (MIT). Use Assistant (human-in-the-loop) mode first. https://github.com/vxcontrol/pentagi
10. **Anthropic — "Claude does cyber competitions"** — Anthropic research, Aug 2025. Claude placed 30th/161 teams in HTB AI vs Human (19/20 solved) and 297th/10,460 in picoCTF 2025 (32/41); web challenges were a particular strength. https://www.anthropic.com/research/cyber-competitions

## B. Agent frameworks & repos

### Claude-first / Claude-compatible

- **Shannon** — KeygraphHQ, GitHub. Autonomous AI pentester for web apps and APIs; reads source code, plans attack paths, executes real exploits (white-box only). Built on Anthropic's Claude + Claude Agent SDK. Free (AGPL-3.0), you pay only model costs. Warning: executes real exploits — only against apps you own or are authorised to test. https://github.com/KeygraphHQ/shannon
- **Claude-CTFd-Solver** — GitHub. CTFd solver agent (named in batch 9; see appendix for URL status). Model: Claude. Type: repo.
- **Strix** — usestrix, GitHub. AI pentest agent (named in batch 2; repo path confirmed in field-guide text). Model: Claude / multi. Type: repo. https://github.com/usestrix/strix
- **CAI (Cybersecurity AI)** — aliasrobotics, GitHub. **ARCHIVED — do not build on it.** Model: multi. Type: repo. https://github.com/aliasrobotics/cai

### DeepSeek-first

- **DeepSeek Harness (dsh)** — deepseek-ai, GitHub, Aug 2026. DeepSeek's official open-source agent runtime (MIT): agent loop, tools, web UI, MCP, subagents, sandboxing; everything is a plugin. Developer preview — breaking changes expected. `npx @deepseek-ai/dsh web`. Free software, but model calls are paid. https://github.com/deepseek-ai/deepseek-harness
- **dsh-superagent-prototype** — deepseek-ai, GitHub, 2026-10-31. Proactive coding agent for one GitHub repo (Router/Worker split, Watchdog), MIT. README: "Research prototype for reference only, not an official DeepSeek product" — workers run commands without approval as the operator's OS user, so isolate it. Useful as a design reference for multi-agent orchestration, not as a CTF tool. https://github.com/deepseek-ai/dsh-superagent-prototype
- **DeepSeek-V4-Fable** — Chunjiang Intelligence, Jun 2026. Security-specialised agent fine-tuned from DeepSeek-V4-Flash (284B MoE); trained on SecDojo-80K (80,000 verified CTF trajectories). Self-reported: 63.8% web, 58.7% overall on 300 held-out challenges. Free weights (via Hugging Face / report), needs GPU. Warning: self-reported, not independently reproduced. Report: https://hg.pleisto.app/Chunjiang-Intelligence/DeepSeek-v4-Fable/resolve/main/report.pdf — overview: https://hackernoon.com/deepseek-v4-fable-a-security-focused-ai-agent-for-ctfs
- **ctfSolver** — passer-W / xjtuHunter team, GitHub. 8-agent web-CTF solver (Explorer, Scanner, Solutioner, Executor, Actioner, Master, Vulner, Saver); DeepSeek for Explorer/Actioner, Tencent Hunyuan for Solutioner/Master. 2nd place at Tencent Cloud AI Penetration Hackathon. Needs DeepSeek + Tencent API keys (paid). Docs: https://deepwiki.com/passer-W/ctfSolver
- **dsh-vulnclaw-pentest** — R1145141919810, GitHub. Community plugin that adds VulnClaw's pentest tools as native DeepSeek Harness tools (37 tools). Needs DeepSeek Harness. Warning: runs tools locally with full permissions. https://github.com/R1145141919810/dsh-vulnclaw-pentest

### Multi-model / model-agnostic

- **HackSynth** — aielte-research (Muzsai et al.), GitHub + arXiv 2412.01778. Planner + Summarizer dual-module agent; benchmark of 200 PicoCTF + OverTheWire challenges (incl. Natas web). Best with GPT-4o; supports Llama, Mixtral, Phi, Qwen. Free, bring your own key. Repo: https://github.com/aielte-research/HackSynth — paper: https://arxiv.org/abs/2412.01778v1
- **hackingBuddyGPT** — ipa-lab (TU Wien), GitHub. Academic LLM agent loop (shell → read output → next step); started on Linux privesc, expanded to web pentesting and web API testing. Free (MIT), bring your own API key (OpenAI, Anthropic, or local endpoint). Good to study, honest published numbers. https://github.com/ipa-lab/hackingBuddyGPT
- **Pentest-Swarm-AI** — Armur-Ai, GitHub. Swarm-based pentester: dozens of agents coordinate via a shared blackboard (stigmergy), not a fixed pipeline; Claude by default, also Ollama/local and any OpenAI-compatible model (incl. DeepSeek). Free (AGPL-3.0); runs fully local on Ollama with no API key. Warning: sends real attack traffic — authorised targets only, scope enforced fail-closed. https://github.com/Armur-Ai/Pentest-Swarm-AI
- **HexStrike AI** — 0x4m4, GitHub. MCP server exposing 150+ security tools (nmap, sqlmap, nuclei, hashcat, gobuster, ffuf…) to Claude/GPT/Copilot; 12+ "agents". Free (MIT). **Warning: no auth — its server listens on unauthenticated 127.0.0.1:8888 and runs shell=True; isolated VM only.** https://github.com/0x4m4/hexstrike-ai
- **VulnClaw** — UncleC, GitHub. AI pentest agent: LLM tool-calling + MCP + 21 security skills; models pentesting as state-space search with evidence-based validation (a claimed flag must appear verbatim in tool output). Supports 13 LLM providers incl. DeepSeek. Warning: Python execution is not sandboxed. Overview: https://wire.fourthweb.ai/openais-o1-model-just-automated-the-entire-hacking-process/
- **EnIGMA** — Princeton / NYU / Tel Aviv (Abramovich et al.), ICML 2025. LM agent for CTFs with Interactive Agent Tools (gdb debugger, pwntools remote connection); SOTA on NYU CTF, InterCode-CTF and Cybench at the time. Built on SWE-agent v0.7. Free (research code). Paper: https://arxiv.org/pdf/2409.16165 — project: https://enigma-agent.github.io/assets/paper.pdf — code base: https://github.com/SWE-agent/SWE-agent/tree/v0.7
- **D-CIPHER** — NYU (Udeshi et al.), arXiv 2502.10931. Multi-agent CTF framework: Auto-prompter + Planner + heterogeneous Executors; 22.0% NYU CTF Bench, 22.5% Cybench, 44.0% HackTheBox. Free (research code). Paper: https://arxiv.org/pdf/2502.10931 — code: https://github.com/NYU-LLM-CTF/nyuctf-agents
- **CTF-Dojo** — Amazon Science (Zhuo et al.), ICML 2026. 658 Dockerised CTF challenges for training agents with verifiable feedback; CTF-Forge pipeline builds environments from public artifacts in minutes. Training on 486 trajectories gave up to +11.6 points; best 32B open model 31.9% Pass@1. Free (CC BY-NC 4.0); challenge files come from pwn.college's CTF Archive. Repo: https://github.com/amazon-science/CTF-Dojo — paper: https://arxiv.org/html/2508.18370v3
- **verialabs ctf-agent** — verialabs, GitHub. Two-tier agent (Coordinator + Solver swarms in Docker sandboxes); won 1st place at BSidesSF 2026 CTF, 52/52 challenges. Free. Docs: https://deepwiki.com/verialabs/ctf-agent

## C. Skill packs (the highest-leverage category)

- **ctf-skills** — ljagiello, GitHub, 2026-09-02. 12 skills (web, pwn, crypto, rev, forensics, OSINT, AI/ML, malware, writeup, solve-challenge…); `ctf-web` alone has 23 files; `/solve-challenge` does first-pass triage and routes to the right skill. Install: `npx skills add ljagiello/ctf-skills`. Free (MIT), ~3.7k stars, 109k installs. Works with Claude Code, Cursor, Codex, Gemini CLI, Copilot. https://github.com/ljagiello/ctf-skills
- **Claude-BugHunter** — elementalsouls (Sachin Sharma), GitHub, 2026-05. 83 skills + 15 slash commands from 681 disclosed HackerOne reports, 24 vuln classes, Burp MCP integration, scope enforced in code, terminal `cbh` runner. Install as Claude Code plugin: `/plugin marketplace add elementalsouls/Claude-BugHunter`. Free (MIT / CC BY 4.0). https://github.com/elementalsouls/Claude-BugHunter
- **Transilience skill files** — Transilience AI, Mar 2026. 230 markdown skill files that took Claude Opus 4.6 to 104/104 on web CTFs; the paper documents each failure→skill cycle. The files live in the repo under `projects/pentest/.claude/skills/`. Copy the method, not just the files. Paper: https://raw.githubusercontent.com/transilienceai/communitytools/95fdc128af4ca1ae16b3226f9430f1bad97b0656/papers/practice-makes-perfect.pdf

## D. Benchmarks, papers & model comparisons

- **NYU CTF Bench — 10 frontier models (arXiv 2604.17159)** — Tyler H. Merves et al., Apr 2026. 200 challenges, Kali + generic prompts. Claude 4.5 Opus 59% ($2.12/solve), Gemini 3 Pro 52% ($0.43), Gemini 3 Flash 27% ($0.05, best value), GPT-5.2-Codex 18%, DeepSeek-V3 6.5%, Llama 3.3 70B 2.5%. Kali env alone +9.5 points vs Ubuntu; category tips often hurt. https://arxiv.org/pdf/2604.17159
- **AIRTBench (arXiv 2506.14682)** — Dreadnode, Jun 2025. 70 black-box AI/ML CTFs: Claude 3.7 Sonnet 43/70 (61%), Gemini 2.5 Pro 39, GPT-4.5 34, DeepSeek R1 29 (41%); best open model 7/70 (10%). Frontier models excel at prompt injection (~49%), struggle at system exploitation/model inversion (<26%). https://arxiv.org/pdf/2506.14682
- **Palisade — Hacking CTFs with Plain Agents (arXiv 2412.02776)** — Dec 2024. 95% on InterCode-CTF with plain ReAct+Plan prompting, multiple attempts. Saturates General Skills, Web, Binary Exploitation, Reverse Engineering. https://arxiv.org/html/2412.02776v1
- **Wiz / Irregular web-security CTF study** — SC Media, ~Jan 2026. Claude Sonnet 4.5, GPT-5, Gemini 2.5 Pro each solved the same 9/10 directed web challenges (auth bypass, exposed secrets, stored XSS, SSRF, IMDS); hard ones needed 4–5 runs; cost mostly <$1 per success. All three failed the broad-scope (whole attack surface, little direction) version — direction matters more than the model. https://www.scworld.com/news/ai-agents-solve-9-of-10-web-security-ctf-challenges-in-recent-study
- **Warwick — The Disruptive Impact of LLMs on CTFs (arXiv 2607.25425)** — Michael Macaulay et al., Jul 2026. Easy/intermediate crypto, web and pwn are now "reliably automated"; proposes tiered divisions, LLM-resistant design, telemetry, code of conduct. https://arxiv.org/abs/2607.25425
- **Irregular — From Scripts to Strategy: Claude 4's Approach to Offensive Security** — Irregular / Anthropic, 2026. Claude Opus 4: 65% overall on 48 challenges, 80% on web exploitation; Sonnet 4: 25/48. Shows the jump from scripted to adaptive behaviour. https://www.irregular.com/research/from-scripts-to-strategy-claude-4s-advanced-approach-to-offensive-security
- **D-CIPHER paper** — see Section B. https://arxiv.org/pdf/2502.10931
- **EnIGMA paper** — see Section B. https://arxiv.org/pdf/2409.16165
- **CTF-Dojo paper** — see Section B. https://arxiv.org/html/2508.18370v3
- **HackSynth paper** — see Section B. https://arxiv.org/abs/2412.01778v1
- **PentestGPT paper (USENIX Security 2024)** — Deng et al. Paper: https://ar5iv.labs.arxiv.org/html/2308.06782 — repo: https://github.com/GreyDGL/PentestGPT
- **GPT-4 vs Claude 3 Opus vs Gemini Pro 1.5 on CTFs** — thesecuritywind.com, ~May 2024. Head-to-head on solved web/forensics/misc challenges: GPT-4 won 7, Claude Opus 5, Gemini 4. Claude solved 4/4 web challenges but refused some prompts and couldn't run code. Prototype Pollution stumped all four. Old models, small set — direction only. https://www.thesecuritywind.com/post/large-language-models-for-ctfs
- **DeepSeek V4.1 Flash on offensive tasks — Enclave benchmark** — Enclave, ~Oct 2026. V4.1 Flash verified 4/11 runs, solved 3/4 tasks (Grafana plugin import, Jenkins CLI response-file, Jenkins upload race, Nextcloud share cache); $3.00 for the 15-run frame, cheapest on their board. Warning: benchmark tasks, not a live CTF; the viral "73%" figure is second-hand and unaudited. https://enclave.ai/blog/deepseeks-new-v41-flash-snapshot-ties-the-old-one-at-4-of-11-and-solves-one-more
- **Bug-bounty agent benchmark (MAPTA etc.)** — Joshua Martinelle (jomar.fr), 2026. MAPTA reported 76.9% on the 104-challenge XBOW benchmark for $21.38; warns that OWASP Juice Shop scores are meaningless (every solution is public) and that XBOW's confidentiality broke once it went on GitHub. https://www.jomar.fr/blog/2026/bugbounty-agent-benchmark/
- **How Are Cybersecurity RL Environments and CTF Benchmarks Built?** — Medium (@sarathc225), 2026. Clear explainer of Cybench, CyberGym, CTF-Dojo, Cyber-Zero and how benchmarks become RL environments; useful before trusting any benchmark number. https://medium.com/@sarathc225/how-are-cybersecurity-rl-environments-and-ctf-benchmarks-built-752246e69431

## E. Writeups & live runs (HTB / TryHackMe / CTFs)

- **Solving FLARE-On 13 with Claude Code** — LB, lbhoe.dev, Oct 2026. All 9 challenges, ~6h, Opus 5.5 on Kali, written CLAUDE.md rules. https://lbhoe.dev/posts/flare-on-13/
- **"It finally hit me" — Claude Opus 5.5 solves FLARE-On in 2 hours** — u/lxyCRO, Reddit r/ClaudeAI, 2026-09-28 (via secondary coverage). A top European reverser who normally takes 10–12 days finished FLARE-On in ~2h with Claude Pro. Warning: anecdote only, unverified; a comment claims the scoreboard showed nobody past 9 tasks. Secondary: https://ainews.leverageai.com.au/case/flareon-ai-reverse-engineering-speedup
- **Anthropic — Claude does cyber competitions** — Anthropic, Aug 2025. HTB AI vs Human: 30th/161 teams, 19/20; picoCTF 2025: 297th/10,460, 32/41. Web was a strength. https://www.anthropic.com/research/cyber-competitions
- **HTB — AI vs Human: CTF results** — Hack The Box / Palisade Research, Mar 2025. 48h, 20 challenges, $7,500 pool: 5 of 8 AI teams solved 19/20 (95%); best AI team 20th/403. Only ~12% of human teams solved all 20. https://www.hackthebox.com/blog/ai-vs-human-ctf-hack-the-box-results
- **Claude Code vs HTB "Cicada" (easy AD)** — Chris Alupului, YouTube, ~Oct 2026. Claude Code from scratch with a custom CLAUDE.md methodology + custom `@htb-pentester` agent; user + root flags in ~6 min (guest SMB → default password → SeBackupPrivilege → NTDS dump). Warning: easy box; public writeups exist for older boxes. https://www.youtube.com/watch?v=ChW98DrTtjQ
- **Grok Code solved hard HTB "Ghostlink" in ~18.5 minutes** — Harrison McCall (@theharrisonmccall), Instagram, 2026-08-06. Claims ~3 prompts to root a hard AD machine (MQTT relay → Gogs RCE → ESC11). Warning: single screen recording, unverified, hype risk. https://www.instagram.com/reel/Dbr0GAKtDl4/
- **PentestGPT vs HackTheBox lab** — ironhackers.es, ~May 2026. PentestGPT v2 solves retired easy HTB machines in 30–50 min at ~$3 API cost; fails on medium machines needing multi-host pivoting; stuck on hard without human guidance. Companion: https://aipentestservices.com/guides/pentestgpt/ (GPT-4 PentestGPT: 6/7 easy, 2/4 medium, 0 hard; 4/10 active HTB machines for $131.50). Main: https://ironhackers.es/en/agentic-red-team-pentestgpt-xbow-2023-2026/
- **r/Pentesting consensus: "Purely automated AI testing is just fancy vulnerability scanning"** — r/Pentesting, quoted by paperclipped.de, ~Mar 2026. The community's verdict: valuable in the right context, not the same as penetration testing. https://www.paperclipped.de/en/blog/ai-pentesting-agents-autonomous-red-team/
- **ChatGPT guided a beginner through TryHackMe "Basic Pentesting"** — Mehedi Hasan Remon, redsentry.com, ~Nov 2025. nmap → Dirb → Enum4linux → Hydra → LinPEAS; free ChatGPT tier. Beginner room; AI suggested standard tools rather than finding novel bugs. https://redsentry.com/resources/blog/turning-chatgpt-into-a-hacker-how-i-used-it-to-solve-a-tryhackme-room
- **TryHackMe "Capture!" — Gemini failed, Claude wrote the working script** — Dan Schwarzentraub, Medium, ~May 2026. Captcha-solving brute-force script; small concrete Claude-vs-Gemini comparison on a real THM web room. Free tiers used. https://daniel-schwarzentraub.medium.com/tryhackme-free-boot-to-root-room-capture-easy-9ad2bcd1cd2b
- **GPT-4 at BSides SF 2023 — "GPT-4 straight up solved some challenges for me"** — Vuink, Apr 2023. Early account: GPT-4 solved flags directly and wrote tedious scripts; refused overtly malicious phrasing; couldn't take a 30 MB APK or a full codebase as context. Outdated now, but shows the pattern early. https://vuink.com/post/zvpnusyrr-d-dpbz/2023/04/capturing-the-flag-with-gpt-4
- **ChatGPT as "third teammate" — 11th/200 at DEF CON 31 Cloud Village CTF** — Charlton, blog.ctis.me, Oct 2023. Two-person team, 3 hours, ChatGPT for fast script prototyping and S3 enumeration; human insight drove the key deductions. https://blog.ctis.me/2023/10/defcon-31-cloud-ctf-writeup/
- **GPT-4 as an autonomous agent on HTB — solved Blue and Legacy, 2 of 7** — T. Chauvin (SERI MATS), GreaterWrong, Apr 2024. GPT-4 wired to a Kali VM via terminal; conclusion "not there yet" — scaffolding matters more than the base model; every command was human-vetted. Outdated model, honest about failure modes. https://www.greaterwrong.com/posts/jqXZzwvDWJZ3yAvYY/end-to-end-hacking-with-language-models
- **Huntress CTF 2026 — player diary** — Michael Meade, michaelmeade.org, 2026-11-01. AI-required event: ChatGPT free credits ran out mid-event; event-provided AI decent but couldn't solve a few; z.ai "worked great" with no limits. Early web challenges still solved manually because no agents were set up. Warning: z.ai is run by a Chinese company — be careful what you paste. https://michaelmeade.org/huntress-ctf-2026/
- **Agent & skills setup for Insomni'hack CTF** — malware.news, ~Apr 2026. Played "full AI": OpenCode + Radare2 MCP + CTF skills, `AGENTS.md` (classify → triage → cheapest path first → keep notes/scripts/artifacts), `/triage`, `/solve`, `/notes` commands. Honest about the learning trade-off and the pay-to-win question. https://malware.news/t/agent-skills-setup-for-insomni-hack-ctf/105958
- **RedPick — HackTheAgent 5/5** — Simone Bovi, redpick.ai, Jun 2026. Full live AI-agent CTF (5/5 levels, black-box, server-confirmed): each level maps to a production LLM-agent risk (policy exception, refusal leaking the value, privileged action from user text, chained secret, SSRF redirect). Good practice target for agent-security thinking. https://www.redpick.ai/blog/redpick-hacktheagent
- **AgentCTF x AgentXploit — Berkeley RDI** — Berkeley RDI Center, 2026. Competition to build AI agents that find and exploit real-world vulnerabilities; public + hidden test cases. https://ctf.secure-agent.com/
- **SANS Find Evil! winners** — SANS Institute, Aug 2026. Largest practitioner evaluation of autonomous AI incident-response agents (4,413 entrants); all five winning harnesses are open source on GitHub (Mulder, TRUDI, Camel, FindEvil, Protocol SIFT++). DFIR-focused, useful for the forensics side of CTF. https://www.sans.org/press/announcements/sans-names-the-five-winners-of-find-evil-2026

## F. Community, rules & safety

- **AI for CTF: How to Use It to Learn, Not Cheat (2026)** — refusalrate.com, ~Nov 2026. Best rules survey: LakeCTF bans AI entirely; UMassCTF 2026 split divisions; RITSEC 2026 DQ'd ~100/800 teams. Copilot strong on 404CTF web but couldn't read raw pcapng/binaries until exported to text. "AI mostly raises the floor, not the ceiling." https://refusalrate.com/ai-for-ctf-learning
- **NorthSec CTF and AI Agents** — NorthSec CTF volunteer, nsec.io, 2026. By late 2025 agents could solve many challenges semi- to fully-autonomously, incl. DEF CON finals-level ones; agent swarms used at Concordia's hackCTF. Debates ban vs two scoreboards vs dynamic scoring vs nothing. https://nsec.io/blog/2026-ctf-agents
- **The Use of LLM Agents in CTFs: A Discussion on Legitimacy** — Cyber Hub (summarising a Reddit thread), ~Dec 2025. Reddit debate: using available tools is part of security skill, but agents automate tasks meant to be done manually. Warning: author admits he could not access the original Reddit thread — weak source. https://www.cyberhub.blog/article/16684-the-use-of-llm-agents-in-ctfs-a-discussion-on-legitimacy
- **hackerbot-claw incident — AI agent attacks on GitHub repos** — Pillar Security, Mar 2026. First publicly documented campaign where an AI agent (self-described "powered by claude-opus-4-5") attacked production open-source infrastructure: 12+ PRs, 7 repos, RCE in 4, one repo destroyed, malicious VSCode extension published. Read before letting any agent touch a repo. https://www.pillar.security/blog/hackerbot-claw-adversarial-agent-targets-top-github-repos — also: https://www.xiaoxiaotu.dev/blog/hackerbot-claw/
- **DeepSeek Harness vs Pi Agent** — dev.to (Composio), 2026. dsh vs Pi: basically a tie (Pi 21/30, dsh 20/30 on a 30-task eval); Pi is smaller and easier to understand, dsh has sessions/approvals/MCP. Pi is the best harness for DeepSeek V4 Flash in the Reddit r/DeepSeek test (66.7% pass, $0.028/success — cheapest). https://dev.to/composiodev/deepseek-harness-dsh-vs-pi-agent-everything-you-need-to-know-5bci — Reddit test summary: https://go.tabbit.ai/model/deepseek-v4-flash/reviews/reddit-deepseek-8-agent-harnesses
- **One command, one local agent (DeepSeek Harness)** — Walid Riachy, walidriachy.com, Sep 2026. Running dsh locally: `npx @deepseek-ai/dsh web`; verified numbers (225k stars, MIT, developer preview). Warning: free software ≠ free model calls; watch for package-name collisions ("dsh"). https://www.walidriachy.com/lab/deepseek-harness
- **Agents that act unprompted need their own account, a sealed box and a signed record** — mindpattern.ai, 2026-11-01. What unattended agents need before write access: separate OS user, enforced boundary (Firecracker micro-VM), approval step, signed record. Applies directly to any CTF agent you leave running. https://www.mindpattern.ai/s/2026-11-01-agents-that-act-unprompted-need-their-own-account-a-sealed-box-and-a-signed-record
- **DeepSeek dsh-superagent explainer** — explainx.ai, Oct 2026. What dsh-superagent is (Router/Worker/Watchdog), risks, how to try it safely. https://explainx.ai/blog/deepseek-dsh-superagent-prototype-proactive-github-coding-agent-tickets-mandates-mit-2026

## G. Free / local models for CTF

- **mythos-sec / NotMythos on Ollama** — supergoatscriptguy / CyberCrew, Ollama, Nov 2026. Community local models tuned for CTF/pentest (web, pwn, crypto, rev, forensics): low-refusal prompts for authorised security work. mythos-sec 24B (LFM2 MoE) and 8B; NotMythos ~11 GB. Free, runs locally, zero API cost. Warning: much weaker than frontier (best open model 10% on AIRTBench); hallucinated CVE numbers/tool flags; pair with an agent harness; one listing's system prompt falsely claims the model "is Claude". https://ollama.com/supergoatscriptguy/mythos-sec
- **DeepSeek-V4-Fable** — see Section B (free weights, needs GPU). https://hackernoon.com/deepseek-v4-fable-a-security-focused-ai-agent-for-ctfs
- **CTF-Dojo trained 32B open model** — see Section B (31.9% Pass@1, rivals DeepSeek-V3-0324). https://github.com/amazon-science/CTF-Dojo
- **Free tiers (no URL — provider homepages, verify before use):** OpenRouter `:free` models, Groq free tier, Gemini API free tier (Flash only, 250 req/day). Anthropic has no free API tier.

## H. Appendix — named resources found by the batch agents whose exact URL could not be re-verified in this pass

These were collected by the 12 batch agents but their exact URLs were lost when the batch reports were condensed. They are listed WITHOUT URLs — do not guess them; search the name to verify before using.

- **XBOW** — #1 on HackerOne; 101–102/104 on its own benchmark (Claude Opus); benchmark report exists. Commercial, paid.
- **Hacktron** — Claude Opus 5 hack of OpenAI, $6,500 (YouTube).
- **Burp AI Agent** — PortSwigger's AI agent for Burp Suite (YouTube / blog).
- **LuaN1aoAgent** — claims 90.4% on XBOW at $0.09/exploit via DeepSeek (self-reported, GitHub).
- **awesome-ai-pentest** and **awesome-autonomous-pentest** — both by Acorzo1983 (GitHub awesome lists).
- **AutoPT, VulnBot, Cochise, NeuroSploit, Redamon, ReaperAI, Cyber-Zero, RapidPen (paper only, no public repo), Google MANTIS (demo only), Darkmoon** — agent frameworks/repos from batch 9 (GitHub / papers).
- **Anthropic 0-days research** — Opus 4.6, 500+ vulns found (blog).
- **PortSwigger HTTP Terminator** — James Kettle, Black Hat 2026 (blog).
- **YesWeHack Claude Code labs** — model routing: Opus=reasoning, Sonnet=recon, Haiku=bulk (blog).
- **NorForge / Cairn campaign** — ~$50/hack with cheap models (blog).
- **XHack Claude Code CTF guide** — ctf-skills + CLAUDE.md + flag-check script (blog).
- **Copilot prompting guide** — 404CTF 165th/2,800 (blog).
- **Tri-Modal Knowledge Engine** — GTFOBins / HackTricks retrieval for agents (blog).
- **CISPA free local ReAct agent tutorial** — Ollama gpt-oss:20b, $0 (blog).
- **Insomni'hack OpenCode setup** — see also Section E entry (malware.news) for the same event.
- **PentestGPT tutorials** — several (blogs).
- **HTB MCP, HTB AI Range benchmarks, HTB Cache / Gofer / TwoMillion Claude-assisted writeups** — HTB blog / writeups. Warning: Club JK item concerns ACTIVE HTB machines — workflow only, no solve details.
- **Neurogrid AI-only CTF** — BoxPwnr 38/45 flags (84.4%).
- **DeepSeek on TryHackMe Hard CTF; THM AI rooms** — writeups.
- **Huntress CTF win** — Maximus Fisher, 1st of 1,602 teams, 561/561, 3-layer system (deterministic solvers → Sonnet proposer → lane agents).
- **Enclave audit** — only 6 of 11 "wins" used the intended vuln; $4.65 total spend.
- **AI-rules survey** — LakeCTF bans AI, UMassCTF two divisions, RITSEC disqualified ~100/800 teams (see also Section F).
- **Grayxploit, Faraday HTB Cache, Memento, msinfo Blogspot** — writeups seen in multiple batches.
- **GLM 5.3** — praised on HN for hacking tasks (model, no free tier confirmed).
- **DeepSeek superagents (dsh-superagent-prototype)** — see Section B for the verified repo.
- **ARTEX warning** — popular Docker images backdoored (~7,946 pulls), repo pulled after real bank breaches; never pull random ARTEX images.
- **Claude cyber safeguards / Cyber Verification Program** — Anthropic's official fix for Opus 5.5 cyber refusals; DeepSeek/Kimi/GLM/Qwen have no such block.

## I. Web security routine — the day-to-day workflow for testing a website

This section is the routine half of the collection: the standard workflow for testing a website step by step (recon, mapping, auth/session, input testing, API testing, business logic, reporting), the skills that cover each step, and the public checklists/playbooks people follow every time. All 49 items below are public, free unless noted, and were verified on 2026-11-20. Model = n/a for all of them (none are AI-model resources). In `collection.csv` these rows carry topic `web-security-routine`.

### I.A Core standards & guides (the routine itself)

- **OWASP Web Security Testing Guide (WSTG) — Latest** — OWASP community, OWASP, living guide. The standard step-by-step web testing guide: 12 test categories (info gathering, config, auth, session, authz, input validation, business logic, client-side…), each with objectives and how-to-test. Free. Use the "Latest" version; older v4.2 copies float around. https://wstg.owasp.org/
- **OWASP Application Security Verification Standard (ASVS) 5.0.0** — OWASP, 2025-05-30. ~350 testable security requirements in 17 chapters (auth, session, access control, validation, business logic, API, files, crypto) at 3 verification levels — the pass/fail checklist behind a thorough test. Free. https://asvs.owasp.org/
- **OWASP Top 10:2025** — OWASP, 2025. Ranked top-10 web risks (A01 Broken Access Control incl. SSRF, A02 Misconfiguration, A03 Supply Chain, A05 Injection, A07 Auth Failures, new A10 Mishandling of Exceptional Conditions) — the priority list every routine is built around. Free. Warning: official site is top10.owasp.org; many 2021-based articles are now outdated. https://devguide.owasp.org/en/02-foundations/05-top-ten/
- **OWASP API Security Top 10 — 2023** — OWASP API Security Project, 2023. The 10 API risks (BOLA/API1, Broken Auth, BOPLA, resource consumption, BFLA, sensitive business flows, SSRF, misconfig, inventory, unsafe consumption) — the checklist for the API-testing phase. Free. https://owasp.org/projects/api-security-project
- **OWASP Cheat Sheet Series — REST Assessment Cheat Sheet** — OWASP, living. How to pentest a REST API step by step: find the attack surface from docs/proxy traffic, identify non-standard parameters, fuzz them; includes OpenAPI/Swagger discovery. Free. https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html
- **OWASP Cheat Sheet Series — Authorization Regression Testing** — OWASP, living. Build an automated authorization test matrix (actor–resource–action) to catch IDOR/BOLA, vertical escalation and tenant-isolation failures on every release. Free. https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Regression_Testing_Cheat_Sheet.html
- **OWASP Web Application Penetration Checklist v1.1 (PDF)** — OWASP, v1.1 (archived). The original OWASP one-line-per-test checklist (info gathering, config, auth, session, authz, data validation, DoS, business logic, crypto, file upload). Free. Warning: OLD/archived — use WSTG Latest as the current version. https://owasp.org/www-project-web-security-testing-guide/assets/archive/OWASP_Web_Application_Penetration_Checklist_v1_1.pdf
- **Penetration Testing Execution Standard (PTES)** — PTES community, v1.1 (standard ~2009–2012). The 7-phase pentest standard: pre-engagement, intelligence gathering, threat modeling, vulnerability analysis, exploitation, post-exploitation, reporting. Free. Warning: dated (mentions BackTrack); official site pentest-standard.org was unreachable/insecure in Nov 2026 — use this readthedocs mirror. https://pentest-standard.readthedocs.io/en/master/

### I.B The big methodology wikis & payload repos

- **HackTricks (book + repo)** — Carlos Polop (@carlospolopm), GitHub / book.hacktricks.xyz, ongoing (repo created 2020-07-15). Community wiki; the "Pentesting Web Methodology" page is a complete ordered routine (fingerprint → scanners → robots/sitemap → spider → dir brute-force → backups → hidden params → per-input vuln tests), plus a page per vuln class. Free. Warning: repo moved from carlospolop/hacktricks to the HackTricks-wiki org (old URL redirects); PDF version is sponsor-only/paid. https://github.com/HackTricks-wiki/hacktricks
- **PayloadsAllTheThings** — swisskyrepo, GitHub, ongoing (created 2016-10-18; ~82,800 stars Nov 2026). Payloads and bypasses organised by vuln class — SQLi, XSS, SSRF, SSTI, XXE, IDOR, file upload, JWT, GraphQL, business logic, race conditions — each with explanation plus Burp Intruder lists. Free (MIT). Web mirror: swisskyrepo.github.io/PayloadsAllTheThings/. https://github.com/swisskyrepo/PayloadsAllTheThings
- **SecLists** — Daniel Miessler, Jason Haddix, Ignacio Portal, g0tmi1k, GitHub, ongoing (created 2012-02-19; ~74,900 stars Nov 2026). The standard wordlist collection: Discovery/Web-Content (dirs, APIs, CMS), Discovery/DNS (subdomains), Fuzzing (SQLi/XSS/LFI/SSTI), Passwords, Usernames, Web-Shells. Free (MIT). Warning: large download; may trigger antivirus false positives. https://github.com/danielmiessler/SecLists
- **PortSwigger Web Security Academy** — PortSwigger research team, portswigger.net, ongoing. Free structured learning paths + hands-on labs for every vuln class in the routine (SQLi, XSS, auth, access control/IDOR, SSRF, file upload, business logic, API/GraphQL, JWT, OAuth, race conditions) — the practical companion to the WSTG. Free. Warning: Burp Suite Pro ($499) and the optional BSCP exam ($99) are paid; Community Edition is enough for the labs. https://portswigger.net/web-security

### I.C Checklists & playbooks (follow-every-time)

- **Web and API Pentest Checklist (CSbyGB / pentips)** — CSbyGB, GitHub, living. Tick-box checklist built directly from the OWASP Testing Guide + API Security Top 10 2023, from information gathering through business logic to the 10 API risks. Free. https://raw.githubusercontent.com/CSbyGB/pentips/main/checklists/web-and-api-pentest-checklist.md
- **API Penetration Testing Checklist (web-cipher-007)** — web-cipher-007, GitHub, living. 22-section API checklist: pre-engagement/recon, auth, BOLA, BFLA, mass assignment, all injection types, rate limiting, GraphQL, WebSockets, JWT, SSRF, CORS, reporting. Free. https://github.com/web-cipher-007/penetration-testing/blob/HEAD/API%20Key%20Leaks/api_pentest_checklist.md
- **Web Application Penetration Testing Checklist — Tushar Verma (PDF, 200+ test cases)** — Tushar Verma, GitHub (0xmaximus/HowToHunt), undated. 200+ custom test cases: recon (subdomains, JS analysis, Wayback), registration, session management, auth, access control, input validation — a dense print-and-follow list. Free. https://github.com/0xmaximus/HowToHunt/raw/84b7f812672da49586d865555ea9036a373e4813/CheckList/Web_Application_Penetration_Testing_Checklist_by_Tushar_Verma.pdf
- **Pentest Book — Pentesting Web checklist** — community compilation, GitHub (Kalyan-Deva/Hacking-Bug-Bounty-Books), undated. Recon by scope size (large/medium/small), then user management, registration, auth, session and access-control test lists with tool commands (amass, ffuf, nuclei, dalfox…). Free. Warning: compilation of unknown provenance; verify items against WSTG. https://github.com/Kalyan-Deva/Hacking-Bug-Bounty-Books/raw/refs/heads/main/Check%20list.pdf
- **OWASP Testing Checklist (text mirror)** — OWASP (mirror by Prashant-Surya), GitHub, undated (old OWASP checklist text). Plain-text version of the classic OWASP checklist: info gathering, config, secure transmission, auth, session, authz, data validation, DoS, business logic, crypto. Free. Warning: old version; prefer WSTG Latest. https://raw.githubusercontent.com/Prashant-Surya/Cheatsheets/master/Cheatsheet_OWASPCheckList.txt
- **OWASP Web Application Security Testing Checklist (WP STAGING)** — WP STAGING team (OWASP content), wp-staging.com, undated (2021-era copy). Readable web rendering of the classic OWASP checklist, same sections as above, used as a dev-team hardening list. Free. Warning: old checklist copy. https://wp-staging.com/owasp-security-testing-checklist/

### I.D Platform methodology (Bugcrowd / HackerOne)

- **Bugcrowd — Pen Test Methodology Checklist and Uploading Logs (docs)** — Bugcrowd, living. How Bugcrowd methodology-based pen tests run: step-by-step checklist per engagement step, Burp log upload, executive summary and submission flow. Free. Warning: the per-step checklists themselves are only visible inside a live pen-test engagement (login-gated); this doc explains the process. https://docs.bugcrowd.com/researchers/participating-in-program/pen-test-programs/
- **Bugcrowd — API Penetration Testing datasheet (PDF)** — Bugcrowd, Nov 2023. API pen-test methodology in 4 cyclical phases (Reconnaissance, Enumeration, Exploitation, Documentation), based on OWASP Testing Guide, SANS Top 25, CREST, WASC and PTES. Free. https://www.bugcrowd.com:443/wp-content/uploads/2023/11/API-Pen-Test-Datasheet.pdf
- **Bugcrowd — Viewing Pen Test Methodology (docs)** — Bugcrowd, living. Companion doc: how to view methodology steps, reported vulns, researcher notes and uploaded logs for an engagement. Free. https://docs.bugcrowd.com/customers/program-management/viewing-pen-test-methodology/
- **API Bug Bounty Methodology (skill file)** — ml-ke, GitHub, 2026-08-06 (backup snapshot). Full API workflow: program intel (scope/rules on HackerOne), asset recon, tech fingerprinting, endpoint mapping, two-account IDOR testing, reporting — with per-program notes. Free. Warning: a personal backup repo, not an official source; method itself is standard. https://github.com/ml-ke/ml-ke/blob/HEAD/_atlas_backups/2026-08-06/skills/bug-bounty/api-bug-bounty-methodology/SKILL.md

### I.E Blog / Medium / Dev.to routine walkthroughs

- **How a Web App Pentest Actually Flows: Recon, Scanning, and Attack Chains** — @dreaith, Medium, Nov 2026. Beginner walkthrough of the real flow — recon → enumeration/mapping → vuln discovery → exploitation → privesc → chaining → impact/reporting — based on the WEB-RTA certification. Free. Warning: Medium may limit free articles per month. https://medium.com/@dreaith/how-a-web-app-pentest-actually-flows-recon-scanning-and-attack-chains-fundamental-lessons-from-72627ca4cc04
- **Web Application Penetration Testing: Methodology, Tools & Guide** — James Cantor, Medium, ~May 2026. End-to-end methodology: scope validation → recon → application mapping → vulnerability testing → exploitation/validation → reporting, with tools per phase. Free. Warning: Medium may limit free articles per month. https://medium.com/@jamescantor38/web-application-penetration-testing-methodology-tools-guide-4e44f78dd36f
- **Our Web Application Penetration Testing Methodology** — Triaxiom Security, triaxiomsecurity.com, updated 2026-08-06. Professional methodology in 3 stages (planning, execution, post-execution) built on OWASP Testing Guide, NIST 800-115 and PTES, with tool list (Burp Pro, Gobuster, Nikto…). Free. https://www.triaxiomsecurity.com/blog/our-web-application-penetration-testing-methodology/
- **What is the Penetration Testing Execution Standard?** — Triaxiom Security, triaxiomsecurity.com, 2021-03-10. Plain-English explainer of the PTES 7 stages with what happens in each — good first read before the full standard. Free. Warning: dated 2021, but PTES itself hasn't changed. https://www.triaxiomsecurity.com/blog/what-is-the-penetration-testing-execution-standard/
- **Pentest Methodology — a 10-day plan** — niebardzo, niebardzo.github.io, 2020-05-05. Day-by-day web pentest plan based on the Web Application Hacker's Handbook: Day 1 mapping, Day 2 client-side/auth/session, Days 3–4 access control, Days 5–6 input vulns, Days 7–8 logic flaws, Day 10 reporting. Free. Warning: from 2020; some tool references are dated. https://niebardzo.github.io/2020-05-05-pentest-methodology/
- **Bug Bounty Hunting: How to Start Earning from Hacking** — Vishal Tripathi, vslprl.com, Apr 2026. Practical bug-bounty routine: recon → scope review → attack-surface mapping → systematic vuln testing (IDOR/access control first) → report writing; points to PortSwigger Academy and Bug Bounty Bootcamp. Free. https://www.vslprl.com/blog/bug-bounty-guide/
- **Bug Bounty Hunting for Beginners 2026: From Recon to Your First Valid Report** — tutorials.technology, ~Jun 2026. Step-by-step beginner routine: legal/scope first, program choice, recon (subfinder/amass/httpx), XSS/IDOR hunting, and writing a report that gets triaged. Free. https://tutorials.technology/tutorials/bug-bounty-hunting-beginners-2026.html
- **How a Junior Bug Hunter Bagged 4 HackerOne Bounties** — Undercode Testing, undercodetesting.com, ~Apr 2026. Recon → tech fingerprinting → manual/automated testing → reporting workflow that produced 4 HackerOne bounties, incl. parameter-pollution IDOR tests. Free. https://undercodetesting.com/how-a-junior-bug-hunter-bagged-4-hackerone-bounties-and-how-you-can-too-video/
- **Top 7 Payload Lists Every Bug Bounty Hunter Needs To Know** — verylazytech, Dev.to, ~Mar 2025. Quick comparison of the 7 payload/wordlist collections to use during input testing: PayloadsAllTheThings, PayloadBox, SecLists, XSS Payloads, FuzzDB and more. Free. https://dev.to/verylazytech/top-7-payloads-lists-every-bug-bounty-hunter-need-to-know-pf7
- **Wordlists for Pentesting: SecLists, RockYou, CeWL and Crunch** — Hackita, en.hackita.it, ~Aug 2026. Which wordlist for which routine step (directory brute-force, DNS, passwords, API discovery, fuzzing) and how to build target-specific lists with CeWL/crunch. Free. https://en.hackita.it/articoli/wordlist/

### I.F Business-logic testing (the step scanners miss)

- **Business Logic Flaw: What It Is and How to Find It in Web Pentests** — Hackita, en.hackita.it, ~Mar 2026. Hands-on method with curl examples on a local shop app: map the flow, write the rules, break one rule at a time (skip/repeat/reverse steps, 0/negative/huge numbers), always check state not just HTTP status. Free. https://en.hackita.it/articoli/business-logic-flaw/
- **Business Logic Bugs Are Broken Invariants** — Colorful White, 1392081456.github.io, 2026-06-27. Reframes business-logic testing as invariant testing (price from catalog, quantity positive, coupon once, no confirmation without payment) across all 12 PortSwigger business-logic labs, live-verified 12/12. Free. https://1392081456.github.io/2026/06/27/business-logic-bugs-are-broken-invariants/
- **Bug-bounty-methodology: Business Logic, Files, and Misc Vulnerabilities** — deathbringer-gd, GitHub, living. Methodology file for business logic, file upload, race conditions, path traversal, deserialization, GraphQL/API vulns — boundary values, step-skipping, parameter trust, coupon replay. Free. https://github.com/deathbringer-gd/bug-bounty-methodology/blob/HEAD/archived/06-Business-Logic-Files-and-Misc-Vulnerabilities.md

### I.G API testing (dedicated)

- **API Pentesting Notes** — Azka, azka.pro, 2024-09-11 (updated 2026-05-18). Compact full API methodology: discovery/enumeration, BOLA/IDOR two-account test, BFLA, mass assignment, GraphQL introspection, auth testing, plus a final checklist. Free. https://www.azka.pro/2024/09/11/web/access-control/api-pentesting-notes/
- **API Penetration Testing: A Practical Guide and Checklist** — SecureNetworks, securenetworks.cloud, ~Oct 2026. Repeatable API routine with identity matrix (anon → admin → service account), endpoint inventory, auth/token tests, authz swaps, injection, mass assignment, business logic, hand validation and reporting. Free. https://www.securenetworks.cloud/api-penetration-testing-guide/
- **API VAPT Checklist: REST and GraphQL Penetration Testing Guide** — PentestHint, blog.pentesthint.com, ~May 2026. 10-phase API VAPT checklist (info gathering → auth → authz → input validation → business logic → rate limiting → data exposure → session → GraphQL → reporting) with tool table. Free. https://blog.pentesthint.com/api-security-vapt-checklist/
- **API Penetration Testing: Step-by-Step Guide (2026)** — Pentest Testing Corp, pentesttesting.com, ~Aug 2026. Six-phase methodology (recon/endpoint discovery → auth → authz/BOLA/BFLA → injection → business logic → reporting) with scoping inputs and per-phase tests. Free. https://www.pentesttesting.com/api-penetration-testing-2026/
- **API Security Checklist: 30 Checks From the OWASP Top 10** — brco.dev, ~Nov 2026. 30 concrete API checks in 5 areas (authorization, authentication, limits/abuse, config/inventory, outbound calls), each mapped to the OWASP API Top 10 2023. Free. https://Brco.dev/insights/api-security-checklist/

### I.H Recon methodology & reporting

- **AMA with @nahamsec** — Ben Sadeghipour (NahamSec), bugbountyforum.com, undated (AMA). NahamSec's routine in his own words: use the app as a normal user first, map functionality (uploads, external fetches), then recon subdomains/dev hosts — and why a mental checklist beats testing vuln classes at random. Free. https://bugbountyforum.com/blog/ama/nahamsec/
- **NahamSec Teaches Me Bug Bounty Basics — video breakdown** — emptyarray, emptyarray.com, 2026-03-28. Step-by-step notes from the John Hammond × NahamSec masterclass: recon pipeline (Whois → subfinder → httpx → Chaos/Censys/Shodan), attack-surface expansion, Red Bull finding walkthrough, automation with cron. Free. https://www.emptyarray.com/articles/nahamsec-bug-bounty-basics-john-hammond.html
- **Penetration Testing Execution Standard (PTES) 2026 Framework** — QualySec, qualysec.com, ~Nov 2026. Modern rewrite of the PTES 7 phases with comparisons to OWASP/NIST and how to apply it in 2026 engagements. Free. https://qualysec.com/penetration-testing-execution-standard/
- **Penetration Testing Standards: PTES and OWASP** — KindaTechnical, kindatechnical.com, Feb 2026. Side-by-side PTES (7 phases) vs OWASP WSTG (12 categories) vs OSSTMM vs NIST SP 800-115, with practical recon commands — good for choosing which framework drives the routine. Free. https://kindatechnical.com/hacking/penetration-testing-standards-ptes-owasp.html
- **Lab 17 — Penetration Test Reporting** — Plaintext Security, plaintextsecurity.org, living. Hands-on reporting lab: copy a report template, fill findings with severity + CVSS + CWE + repro + impact + remediation, write an executive summary, validate with a script (11 checks). Free. https://www.plaintextsecurity.org/01-offensive/modules/17-reporting/lab/
- **OSCP Report Template** — TheGetch, github-wiki-see.page, undated (2020-era template). Classic pentest report skeleton: intro/objective, high-level summary + recommendations, methodology walkthrough, per-finding proof, appendices — reusable for any web test report. Free. Warning: OSCP-flavoured (exam-oriented), adapt the structure. https://github-wiki-see.page/m/TheGetch/Penetration-Testing-Methodology/wiki/OSCP-Report-Template

### I.I Video walkthroughs (full routine)

- **How Hackers Take Full Control of a Website (Full Walkthrough)** — The Hacker Academy, YouTube, ~Nov 2026. Full beginner walkthrough on TryHackMe "Pickle Rick": recon (nmap) → web enumeration (robots.txt, gobuster) → command injection/filter bypass → reverse shell → privesc → fixes for each weakness. Free. https://www.youtube.com/watch?v=PpynfsItfk8
- **The Penetration Testing Masterclass | Complete Ethical Hacking Methodology | Bug Bounty Day 54** — Nexiform Technologies, YouTube, ~Nov 2026. Complete methodology video: scope/rules of engagement → recon → vuln assessment → web app testing (SQLi, XSS, SSRF, auth/access control) → controlled exploitation → reporting/remediation/retesting. Free. https://www.youtube.com/watch?v=5XlF9hGtDFQ
- **PortSwigger Web Security Academy: Beginner's Guide (2026)** — HackerDNA, hackerdna.com, ~Oct 2026. How to actually use the Academy: what's free, which topics first, Burp Community setup, first-lab walkthrough; notes the only costs are Burp Pro and the $99 BSCP exam. Free. https://hackerdna.com/blog/portswigger-web-security-academy

### Skipped (routine) — not public, not included

HackerOne's official researcher methodology docs (login-gated, no public URL found); Bugcrowd's in-engagement step checklists (only visible inside a live pen-test engagement); HackTricks PDF (paid, sponsor-only); the books "Web Application Hacker's Handbook", "Bug Bounty Bootcamp" and "Hacking APIs" (paid books, referenced inside other items but not listed as free sources); O'Reilly PTES excerpts (paywalled previews).

## J. Public AI-agent skills (saved in `skills/`)

Public skill files and skill collections for web/CTF work. The 8 best files are
copied verbatim into `skills/` (each with a header noting source URL, repo,
author and license). Everything else is repo-only — clone it yourself. All
free to read/clone; running a skill costs model usage (Claude Code needs
Pro/Team/Max or an API key; OpenCode + a free model is the free route).
See `skills/README.md` for install steps and the step-by-step mapping.

### J.1 Saved skill files (8)

| File in `skills/` | Skill | Repo | License | What it covers |
|---|---|---|---|---|
| `ctf-skills-ctf-web-SKILL.md` | `ctf-web` | [ljagiello/ctf-skills](https://github.com/ljagiello/ctf-skills) | MIT | CTF web: SQLi, XSS, SSTI, SSRF, XXE, JWT, OAuth, prototype pollution; routes to 23 sub-skill files |
| `ctf-skills-solve-challenge-SKILL.md` | `solve-challenge` | [ljagiello/ctf-skills](https://github.com/ljagiello/ctf-skills) | MIT | Orchestrator: classify any CTF challenge, then route to the category skill |
| `claude-bughunter-bb-methodology-SKILL.md` | `bb-methodology` | [elementalsouls/Claude-BugHunter](https://github.com/elementalsouls/Claude-BugHunter) | MIT | Master orchestrator: 5-phase hunt (recon → map → find → prove → report), mindset, false-positive discipline |
| `claude-bughunter-hunt-xss-SKILL.md` | `hunt-xss` | [elementalsouls/Claude-BugHunter](https://github.com/elementalsouls/Claude-BugHunter) | MIT | XSS from 174 disclosed reports: canary/reflection discipline, sanitizer + CSP + WAF bypasses, 6 chains to ATO |
| `claude-bughunter-hunt-xxe-SKILL.md` | `hunt-xxe` | [elementalsouls/Claude-BugHunter](https://github.com/elementalsouls/Claude-BugHunter) | MIT | XXE from 10 disclosed reports: SVG/Office/SOAP/SAML, blind OOB, parser-ecosystem matrix, CosmicSting |
| `claude-red-offensive-sqli-SKILL.md` | `offensive-sqli` | [SnailSploit/Claude-Red](https://github.com/SnailSploit/Claude-Red) | not stated | SQLi: error/UNION/blind/OOB, MySQL/MSSQL/Postgres/Oracle, NoSQL/GraphQL/WebSocket, WAF bypass, ORM CVEs |
| `claude-red-offensive-jwt-SKILL.md` | `offensive-jwt` | [SnailSploit/Claude-Red](https://github.com/SnailSploit/Claude-Red) | not stated | JWT: alg confusion, weak secrets, kid/jku/x5u injection, JWKS poisoning, timing attacks, mobile storage |
| `offensive-claude-browser-exploitation-SKILL.md` | `browser-exploitation` | [hypnguyen1209/offensive-claude](https://github.com/hypnguyen1209/offensive-claude) | MIT | Browser/client-side: V8 JIT type confusion, heap-sandbox escape, Mojo/GPU escape, Electron RCE, 1-click chains. Index file only — its `references/` and `scripts/` live in the repo, not here |

### J.2 Other verified public skill collections (repo only, not copied)

- **yaklang/hack-skills** — Yaklang, ~2,674 stars, 102 skills. Broad offensive skill pack. Free. https://github.com/yaklang/hack-skills
- **trailofbits/skills** — Trail of Bits, CC-BY-SA-4.0. Defensive audit skills (code review, audit checklists) — use for the "validate your finding" half of the routine, not for attacking. Free. https://github.com/trailofbits/skills
- **anthropics/skills** — Anthropic (official), Apache-2.0. The official skill spec reference — not security-specific, but the format every other repo follows. Free. https://github.com/anthropics/skills
- **ComposioHQ/awesome-claude-skills** — ~79k stars. Huge awesome-list of Claude skills (many branches stale — check the date on any skill you pull). Free. https://github.com/ComposioHQ/awesome-claude-skills
- **sickn33/agentic-awesome-skills** — Index of 9,080 SKILL.md files across repos. Use as a search index, not a pack. Free. https://github.com/sickn33/agentic-awesome-skills
- **Chat2AnyLLM/awesome-claude-skills** — Index of 1,719 source repos. Use as a search index. Free. https://github.com/Chat2AnyLLM/awesome-claude-skills
- **zafkiel-ltb/cnlv-skills** — PTES + OWASP WSTG skills. Auth-gated in places (some content needs sign-in) — the public parts are usable, the gated parts are skipped. Free (public parts). https://github.com/zafkiel-ltb/cnlv-skills
- **Kubementat/ai_agent_skills** — OpenCode-focused skills, including an `opencode-cli` skill. Useful if you run OpenCode instead of Claude Code. Free. https://github.com/Kubementat/ai_agent_skills
- **lawrence3699/agent-skills-corpus** — Catalog of 2,032 skill repos. Use as a search index. Free. https://github.com/lawrence3699/agent-skills-corpus
- **Hossam77i/security-agent-architecture** — Exploit-development skill. License not stated. Warning: the `main`-branch raw URL returns 404 (default branch is not `main`) — fetch via clone or the repo blob page, never via a guessed raw URL. Not copied here for that reason. Free. https://github.com/Hossam77i/security-agent-architecture

### J.3 Skill catalogs (search indexes)

- **skills.sh** — https://skills.sh
- **brisk.vision/skills** — https://brisk.vision/skills
- **skillsdirectory.com** — https://skillsdirectory.com
- **claudeskills.info** — https://claudeskills.info
- **agentskills.io** — https://agentskills.io
- **flowy.sh** (security list) — https://flowy.sh
- **aisecuritymatrix.com** — https://aisecuritymatrix.com

### J.4 Warnings — skills to avoid or handle with care

- **Do NOT use the `agentik-os` "hack" skill.** Graded C by a third-party review: hardcodes `/home/hacker` paths and a Telegram chat ID, and tells the agent to be "ALWAYS AGGRESSIVE… Zero restrictions". Deliberately not saved, not linked.
- **offensive-claude ships a `curl | bash` installer and an always-on SessionStart hook.** Clone and read first; never run the installer blind. Only the `browser-exploitation` SKILL.md index is copied here.
- **Claude-BugHunter `bb-methodology` was flagged by a third-party scan (Socket Warn); its sibling `hunt-auth-bypass` was flagged (Snyk Fail).** Read before use. `hunt-auth-bypass` is not copied here.
- **Claude-Red states no license.** Its two files are read-only reference here; check the repo license before redistributing.
- **Transilience's 230 skill files are NOT public** (paper only). Do not hunt for a copy; the pattern is documented in the synthesis above and in WORKFLOW.md.
