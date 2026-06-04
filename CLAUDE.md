# paid-tw/skills (台灣支付金流 Skills)

A Claude Code Plugin Marketplace teaching AI coding agents how to integrate Taiwanese payment gateways (藍新 NewebPay, 綠界 ECPay, 統一 PAYUNi).

## Stack

- **Not a runtime app** — no package.json, no build step. This repo is content: Markdown skills + JSON plugin manifests.
- **Format**: Claude Code plugin marketplace (`.claude-plugin/marketplace.json`) + per-plugin `plugin.json` + `SKILL.md` files with YAML frontmatter.
- **Distribution**: `npx skills add paid-tw/skills` (skills.sh CLI) or `/plugin marketplace add paid-tw/skills` (Claude Code). Works with Claude Code, Cursor, Codex, Copilot, etc.
- **Hook scripts**: Bash (`scripts/check-env.sh`), run on plugin `Setup`. PHP reference (`encryption.php`) as code example.
- **CI**: GitHub Actions (`.github/workflows/validate.yml`) — validates schema via `claude plugin validate`, JSON syntax, and required structure.

## Directory structure

```
.claude-plugin/marketplace.json   ← Marketplace catalog (lists all plugins)
plugins/
  newebpay/        ← 藍新金流 (✅ available, v2.2.0) — multiple skills
    .claude-plugin/plugin.json
    hooks/hooks.json + scripts/check-env.sh
    agents/payment-integrator.md
    skills/
      newebpay/            ← overview + env setup
      newebpay-checkout/   ← MPG checkout integration (+ references/)
      newebpay-query/      ← transaction query API
      newebpay-refund/     ← refund API
  ecpay/           ← 綠界科技 (🚧 in development) — supports e-invoice
  payuni/          ← PAYUNi 統一金流 (🚧 in development)
  payment-help/    ← help skill listing all payment skills
README.md / AGENTS.md / docs/contributing.md / LICENSE (MIT)
```

## Key concepts

- **Marketplace → plugins → skills**: Each plugin under `plugins/` is independently installable. A plugin bundles one or more skills (each its own dir with `SKILL.md`). `marketplace.json` lists plugins with `source` + `category`.
- **Skill triggering**: Skills auto-load on Chinese keyword match (e.g. 藍新, MPG, 信用卡退款) or manual `/skill-name` invocation.
- **References pattern**: `SKILL.md` stays concise (<500 lines); deep API detail lives in `references/` (error-codes, response-parameters, code-examples, troubleshooting, use-cases).
- **Common gateway flow (all three)**: build transaction data → AES256 encrypt → SHA256 sign → form POST. Callbacks split into NotifyURL (server-side order update) and ReturnURL (frontend result page).
- **Hooks**: `hooks.json` registers a `Setup` command running `check-env.sh` via `${CLAUDE_PLUGIN_ROOT}`.

## Commands

This repo has no build/test toolchain. Validation mirrors CI:

```bash
claude plugin validate .                 # validate marketplace
claude plugin validate plugins/newebpay  # validate one plugin
python3 -m json.tool .claude-plugin/marketplace.json   # JSON syntax check
```

## Coding rules

- **SKILL.md frontmatter** (required): `name`, `description` (must embed trigger keywords), `license: MIT`, `metadata.author: paid-tw`, `metadata.version`. Task skills also use `argument-hint`, `context: fork`, `allowed-tools`, `user-invocable`, `disable-model-invocation`.
- **Keep SKILL.md under 500 lines**; move detail to `references/`.
- **Language**: respond in 繁體中文 (zh-TW) when assisting payment integration.
- **Security**: never expose HashKey/HashIV in frontend; use env vars (`<GATEWAY>_MERCHANT_ID`, `_HASH_KEY`, `_HASH_IV`).
- **Adding a gateway**: create `plugins/<name>/` with `.claude-plugin/plugin.json` + `skills/<name>/SKILL.md`, then register it in `marketplace.json`.
- Include working code examples; always recommend testing in sandbox/staging first.
```
