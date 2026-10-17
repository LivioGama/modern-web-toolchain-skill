<div align="center">

# ⚡ modern-web-toolchain

**Migrate a JS/TS repo to the fast native toolchain — Bun, TypeScript 7, Vite+ — measured, minimal, no repo-wide reformat.**

[![Agent Skill](https://img.shields.io/badge/agent--skill-SKILL.md-blueviolet?style=for-the-badge)](SKILL.md)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-DA7756?style=for-the-badge&logo=anthropic&logoColor=white)](#install)
[![Codex](https://img.shields.io/badge/Codex-ready-412991?style=for-the-badge&logo=openai&logoColor=white)](#install)
[![Cursor](https://img.shields.io/badge/Cursor-ready-000000?style=for-the-badge&logo=cursor&logoColor=white)](#install)
[![Gemini](https://img.shields.io/badge/Gemini-ready-1BA1E3?style=for-the-badge&logo=googlegemini&logoColor=white)](#install)
[![Devin](https://img.shields.io/badge/Devin-ready-0A0A0A?style=for-the-badge)](#install)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

</div>

---

## What it does

Assesses a repository, then migrates four tools in one minimal PR:

| From | To | Measured gain |
|------|----|---------------|
| pnpm / npm / yarn | **Bun** | ~9× cold install |
| TypeScript 5 `tsc` | **TypeScript 7** (native) | ~7–10× typecheck |
| Vite / Vitest | **Vite+** (`vp test`) | test-run dependent |
| ESLint + Prettier | **oxlint + oxfmt** (`vp lint`, `vp fmt`) | orders of magnitude |

The migrations are simple and low-risk; the skill's real value is the discipline around them — assess before touching, keep `lint` in CI but `fmt` configured-not-enforced, derive `ignorePatterns` from the repo's own content-hashed paths, and publish before/after numbers in the PR body.

Reference migrations with real stats: [fdussert/fremkit#42](https://github.com/fdussert/fremkit/pull/42) · [fdussert/fremkit-sietch#13](https://github.com/fdussert/fremkit-sietch/pull/13)

### Bonus assessment

On Expo / React Native repos it also evaluates [OneStack](https://onestack.dev/) as a Metro replacement (~5.4×) — **suggestion only**, never installed or committed without explicit approval.

## Install

**One-click (agent-config deeplink):**

[![Install modern-web-toolchain skill](https://raw.githubusercontent.com/LivioGama/agent-config/main/assets/install-badge-small.jpg)](agent-config://https://raw.githubusercontent.com/LivioGama/modern-web-toolchain-skill/main/SKILL.md)

*Requires the [agent-config](https://github.com/LivioGama/agent-config) deeplink handler.*

**Or clone the whole folder into whichever agent you use:**

```bash
# Claude Code
git clone https://github.com/LivioGama/modern-web-toolchain-skill ~/.claude/skills/modern-web-toolchain

# Codex / Cursor / Gemini / Devin: same shape
git clone https://github.com/LivioGama/modern-web-toolchain-skill ~/.codex/skills/modern-web-toolchain
git clone https://github.com/LivioGama/modern-web-toolchain-skill ~/.cursor/skills/modern-web-toolchain
git clone https://github.com/LivioGama/modern-web-toolchain-skill ~/.gemini/skills/modern-web-toolchain
git clone https://github.com/LivioGama/modern-web-toolchain-skill ~/.devin/skills/modern-web-toolchain
```

## Usage

Ask your agent to speed up or modernize the toolchain ("migrate this repo to bun and Vite+", "why is my typecheck slow"), or invoke `/modern-web-toolchain`.

The skill inventories the repo first — package manager, TS version, test runner, lint/format setup, spawned binaries, CI steps, workspaces — reports what it found, then migrates and verifies end-to-end locally before CI confirms.

## License

MIT, see [LICENSE](LICENSE).
