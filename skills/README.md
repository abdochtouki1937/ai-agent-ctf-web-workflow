# Public AI-agent skills — saved files

Copied 2026-11-20 from public repos only. Each file has an HTML-comment header
with its exact source URL, repo, author and license. Do not edit the content
below those headers — if a skill updates upstream, re-copy it from the source
URL instead of patching the copy here.

## How to use them

Claude Code / Codex / OpenCode load `SKILL.md` files from a skills directory.
The usual layout is one folder per skill:

```
~/.claude/skills/<skill-name>/SKILL.md        # personal, all projects
<project>/.claude/skills/<skill-name>/SKILL.md # this project only
```

The files here are flat copies (one file per skill, renamed with a repo prefix)
so they fit in this collection. To install one, copy it into a folder named
after the skill, e.g.:

```bash
mkdir -p ~/.claude/skills/ctf-web
cp ctf-skills-ctf-web-SKILL.md ~/.claude/skills/ctf-web/SKILL.md
```

Or install a whole repo at once (see the repo links in COLLECTION.md §J):

```bash
npx skills add ljagiello/ctf-skills        # ctf-skills, whole pack
git clone https://github.com/elementalsouls/Claude-BugHunter.git
git clone https://github.com/SnailSploit/Claude-Red.git
git clone https://github.com/hypnguyen1209/offensive-claude.git
```

Running a skill costs model usage: Claude Code needs a Pro/Team/Max plan or an
API key. The free route is OpenCode with a free model (DeepSeek / Kimi / GLM /
Qwen via their free tiers or OpenRouter free models).

## Saved files (8)

| File | Skill | Source repo | License | Covers |
|---|---|---|---|---|
| `ctf-skills-ctf-web-SKILL.md` | `ctf-web` | ljagiello/ctf-skills | MIT | CTF web: SQLi, XSS, SSTI, SSRF, XXE, JWT, OAuth, prototype pollution, routing to the right sub-skill |
| `ctf-skills-solve-challenge-SKILL.md` | `solve-challenge` | ljagiello/ctf-skills | MIT | Orchestrator: classify any CTF challenge, then route to the category skill |
| `claude-bughunter-bb-methodology-SKILL.md` | `bb-methodology` | elementalsouls/Claude-BugHunter | MIT | Master orchestrator: 5-phase hunt (recon → map → find → prove → report), mindset, false-positive discipline |
| `claude-bughunter-hunt-xss-SKILL.md` | `hunt-xss` | elementalsouls/Claude-BugHunter | MIT | XSS hunting from 174 disclosed reports: reflection/canary discipline, bypasses, chains to ATO |
| `claude-bughunter-hunt-xxe-SKILL.md` | `hunt-xxe` | elementalsouls/Claude-BugHunter | MIT | XXE hunting from 10 disclosed reports: SVG/Office/SOAP/SAML, blind OOB, parser matrix |
| `claude-red-offensive-sqli-SKILL.md` | `offensive-sqli` | SnailSploit/Claude-Red | not stated | SQLi: error/UNION/blind/OOB, MySQL/MSSQL/Postgres/Oracle, WAF bypass, ORM CVEs |
| `claude-red-offensive-jwt-SKILL.md` | `offensive-jwt` | SnailSploit/Claude-Red | not stated | JWT: alg confusion, weak secrets, kid/jku/x5u injection, JWKS poisoning, mobile storage |
| `offensive-claude-browser-exploitation-SKILL.md` | `browser-exploitation` | hypnguyen1209/offensive-claude | MIT | Browser/client-side: V8 JIT, heap-sandbox escape, Mojo/GPU escape, Electron RCE, 1-click chains. Index file only — its `references/` and `scripts/` live in the repo, not here |

## Which skill for which routine step

| Routine step (see ROUTINE.md) | Skill file(s) |
|---|---|
| 0. Scope & rules, 1. Recon | `bb-methodology` (Phases 0–1), `ctf-web` |
| 2. Map the app | `bb-methodology` (Phase 2), `ctf-web` |
| 3. Login / sessions | `bb-methodology` (Phase 3), `offensive-jwt` |
| 4. Inputs / injection | `hunt-xss`, `hunt-xxe`, `offensive-sqli`, `ctf-web` |
| 5. APIs | `offensive-sqli` (GraphQL/REST), `offensive-jwt`, `ctf-web` |
| 6. Prove (impact chains) | `bb-methodology` (Phase 4), `hunt-xss` (Chains), `hunt-xxe` |
| 7. Report | `bb-methodology` (Phase 5) |
| Browser / client-side targets | `browser-exploitation` |
| Any CTF challenge, any category | `solve-challenge` (routes to the right skill) |

## Warnings — read before you use or install

- **Claude-BugHunter `bb-methodology` was flagged by a third-party scan
  (Socket Warn)** and its sibling `hunt-auth-bypass` was flagged (Snyk Fail).
  Read the files before use; they are included because the methodology is
  strong, not because every line is endorsed.
- **offensive-claude ships a `curl | bash` installer and an always-on
  SessionStart hook.** Clone the repo and read it first. Never run the
  installer blind. Only the `browser-exploitation` SKILL.md index is copied
  here, not the installer or hooks.
- **Claude-Red states no license in the repo.** Treat its two files as
  read-only reference here; check the repo license before redistributing.
- **Deliberately NOT saved:** the `agentik-os` "hack" skill (graded C by a
  third-party review; hardcodes `/home/hacker` paths and a Telegram chat ID,
  and tells the agent to be "ALWAYS AGGRESSIVE… Zero restrictions"). Do not
  use it.
- **Transilience's 230 skill files are NOT public** (paper only, no repo).
  Do not look for a copy; the pattern is documented in COLLECTION.md and
  WORKFLOW.md instead.
- **Hossam77i/security-agent-architecture** (exploit-development skill) is a
  verified public repo, but its `main`-branch raw URL returns 404 — the
  default branch is not `main`. Fetch it via clone or the repo's blob page,
  not via a guessed raw URL. Not copied here for that reason.

## Other verified public skill collections (not copied, repo only)

Listed in full in COLLECTION.md §J: yaklang/hack-skills (102 skills),
trailofbits/skills, anthropics/skills, ComposioHQ/awesome-claude-skills,
sickn33/agentic-awesome-skills, Chat2AnyLLM/awesome-claude-skills,
zafkiel-ltb/cnlv-skills, Kubementat/ai_agent_skills,
lawrence3699/agent-skills-corpus, plus the Hossam77i repo above and the
skill catalogs (skills.sh, brisk.vision/skills, skillsdirectory.com,
claudeskills.info, agentskills.io, flowy.sh, aisecuritymatrix.com).
