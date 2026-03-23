# Amex-Automod-Wiki

Production-ready [AutoModerator](https://www.reddit.com/wiki/automoderator/full-documentation) ruleset for the **r/amex** subreddit — covering PII protection, policy enforcement, spam control, and community automation.

> **Version 5.0** · 62 active rules across 11 sections · Referral thread management · 19 user `!` commands · Expanded domain allowlist · CI validation · MIT License

---

## Table of Contents

- [Overview](#overview)
- [Repository Contents](#repository-contents)
- [Architecture](#architecture)
  - [Rule Sections](#rule-sections)
  - [Execution Model](#execution-model)
- [Feature Details](#feature-details)
  - [PII & Doxxing Protection](#pii--doxxing-protection)
  - [Policy Enforcement](#policy-enforcement)
  - [Quality & Spam Controls](#quality--spam-controls)
  - [Community Helpers](#community-helpers)
- [Design Principles](#design-principles)
- [Deployment](#deployment)
  - [Prerequisites](#prerequisites)
  - [Steps](#steps)
  - [Validation](#validation)
- [Moderator Guide](#moderator-guide)
  - [Action Types](#action-types)
  - [Operational Workflow](#operational-workflow)
  - [Making Changes](#making-changes)
- [YAML Syntax Reference](#yaml-syntax-reference)
- [Reference Materials](#reference-materials)
- [License](#license)

---

## Overview

The **Amex-Automod-Wiki** repository maintains the AutoModerator configuration deployed on **r/amex**. The ruleset enforces five subreddit rules:

| Rule | Policy | Enforcement |
|------|--------|-------------|
| **Rule 1** | Referral Protocol | Designated threads only; permanent ban for violations |
| **Rule 2** | Illegal Activity | All non-`americanexpress.com`/`amex.com` links blocked |
| **Rule 3** | Submission Quality | Research required; low-effort posts redirected to Monthly FAQ |
| **Rule 4** | Moderator Interaction | Mod decisions are final |
| **Rule 5** | User Decorum | Respectful conduct required |

Beyond rule enforcement, the config provides **PII/doxxing protection** (credit cards, SSNs, phone numbers, emails, addresses) and **community helper bots** that auto-reply with guidance on common topics.

---

## Repository Contents

| File | Purpose |
|------|---------|
| `amex_automod_remediated.yml` | **Production AutoMod config** — deploy this file |
| `.github/workflows/validate.yml` | **CI validation** — auto-tests config on every push/PR |
| `automod-reference.md` | Consolidated AutoModerator documentation reference |
| `community-manager.md` | Community manager skill for r/amex post creation |

---

## Architecture

### Rule Sections

The config is organized into **10 sections**, each separated by `---` document dividers and labeled with header comments:

| # | Section | Purpose |
|---|---------|---------|
| 1 | **Shadowbans** | Username-targeted silent removal |
| 2 | **PII / Doxxing Protection** | Sensitive data detection and removal |
| 3 | **Rule 1 — Referral Protocol** | Referral posting enforcement |
| 3a | **Designated Referral Thread** | Sticky rules + 30-day limit enforcement via community reports |
| 4 | **Rule 2 — Illegal Activity & Links** | External link and scam blocking |
| 5 | **Account Quality Gates** | Account age/karma filtering |
| 6 | **Rule 5 — User Decorum** | Abusive language and threat detection |
| 7 | **Rule 3 — Submission Quality** | Low-effort post filtering |
| 8 | **Rule 4 — Moderator Interaction** | Mod authority enforcement |
| 9 | **Report Handling** | Report-triggered moderation workflows |
| 10 | **Community Helpers & Bot Commands** | Auto-replies for common topics; user-summoned `!` commands |

### Execution Model

Each rule is a YAML document that declares checks and actions. The config uses:

| Key | Purpose |
|-----|---------|
| `action`: `remove`, `filter`, `report` | Severity-appropriate response |
| `priority` | Influences rule evaluation order |
| `modmail_subject` + `modmail` | Escalation context for moderators |
| `action_reason` with `{{match}}` | Audit trail in mod tooling |
| `type`: `submission`, `comment`, `any` | Scope control |
| `~check` negation | False-positive exclusions |
| `is_edited: true` | Catches edit-after-post bypass attempts |

---

## Feature Details

### PII & Doxxing Protection

The most comprehensive section, providing layered defense against sensitive data exposure:

**Credit card detection** — covers all six major card networks:
- Issuer-aware grouped patterns (Visa, Mastercard, Amex, Discover, Diners Club, JCB)
- Per-digit separation defense (e.g., `4-1-2-3-...`)
- Contiguous 13–19 digit detection with targeted exclusions
- Obfuscation evasion via flexible separators (dots, dashes, pipes, markdown formatting)

**Other PII patterns:**
- **SSN** — `XXX-XX-XXXX` format with invalid-range exclusions (000, 666, 9xx)
- **Phone numbers** — Multiple US formats; excludes 555 numbers and crisis hotlines
- **Email addresses** — Standard, bracketed obfuscation (`[at]`/`[dot]`), and ALL-CAPS `AT`/`DOT` formats
- **Mailing addresses** — Street address pattern detection

**Anti-evasion measures:**
- Edited-content re-scan (`is_edited: true`) to catch post-then-edit bypass
- Zero-width Unicode character detection (`\u200B`, `\u200C`, `\u200D`, `\uFEFF`, etc.)
- Fullwidth, superscript, subscript, and circled numeral detection

### Policy Enforcement

- **Referral Protocol (Rule 1):** Detects referral links/language outside designated threads. Removes violations and sends modmail with ban-level escalation guidance.
- **Illegal Activity & Links (Rule 2):** Blocks all external links except `americanexpress.com` and `amex.com`. Detects scam patterns and escalates high-risk language.
- **User Decorum (Rule 5):** Detects slurs, abusive language, and threats. Removes or filters content and escalates to moderators for ban decisions.
- **Moderator Interaction (Rule 4):** Enforces respectful mod interaction expectations.

### Quality & Spam Controls

- **Account Quality Gates:** Filters based on account age and karma to reduce low-quality abuse and spam.
- **Submission Quality (Rule 3):** Detects low-effort and FAQ-pattern posts. Directs users to research resources.
- **Report Handling:** Routes user-reported content to moderators for review.

### Community Helpers & Bot Commands

Non-enforcement automations that auto-reply with standardized guidance. All commands work on both **posts and comments** using `type: any`.

#### User-Summoned `!` Commands

Any member or moderator can type a `!` command in any post or comment to summon an AutoMod reply:

| Command | Aliases | Topic |
|---------|---------|-------|
| `!acronyms` | | Common r/amex acronym definitions |
| `!popup` | `!jail` | Pop-up jail explanation and workaround |
| `!recon` | | Amex reconsideration line tips |
| `!lounge` | `!centurion` | Airport lounge access by card |
| `!retention` | | Annual fee retention offer tips |
| `!offers` | | Amex Offers overview |
| `!transfer` | | Membership Rewards transfer partners |
| `!fr` | `!financialreview` | Financial Review (FR) guidance |
| `!status` | | Application status links and tips |
| `!contact` | `!support` | Amex contact numbers and options |
| `!rules` | | Subreddit rules summary |
| `!trifecta` | | Platinum + Gold + BBP strategy |
| `!return` | | Return protection policy link |
| `!purchase` | | Purchase protection policy link |
| `!warranty` | | Extended warranty protection link |
| `!trip` | | Trip cancellation/delay insurance link |
| `!cellphone` | | Cell phone protection link |
| `!car` | | Car rental insurance link |
| `!help` | | List all available commands |

Commands also trigger organically when users discuss the topic naturally (e.g., mentioning "pop-up jail" triggers the same reply as `!popup`).

#### Designated Referral Thread (Section 3a)

The only permitted location for referral links is the designated scheduled thread:
`https://www.reddit.com/r/amex/submit/?scheduled_post_id=437126`

AutoMod enforces this thread with:
- **Sticky welcome comment** with rules when the thread is posted
- **Community-report pipeline**: 1 report → filter + modmail to mods for 30-day compliance review
- **Exemption**: the referral link and veiled solicitation rules are silenced inside this thread

> **Note:** The 30-day per-user limit cannot be enforced natively by AutoMod (per its documented limitations). Community reports + mod review are the enforcement mechanism. For automated enforcement, a Reddit [Devvit](https://developers.reddit.com/docs/devvit) app is recommended.

#### Approved External Domain Allowlist (Rule 2)

Rule 2 blocks all external links except the following trusted domains:

| Category | Domains |
|----------|---------|
| American Express | `americanexpress.com`, `amex.com` |
| Reddit | `reddit.com`, `redd.it` |
| Rewards / Cashback | `rakuten.com` |
| Credit News | `doctorofcredit.com`, `nerdwallet.com`, `thepointsguy.com`, `wallethub.com`, `bankrate.com` |
| Credit Scores | `myfico.com`, `fico.com`, `experian.com`, `equifax.com`, `transunion.com`, `annualcreditreport.com` |
| Media / Images | `imgur.com` |
| Co-Brand Partners | `delta.com`, `hilton.com`, `marriott.com` |

---

## Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Defense in depth** | Multiple pattern classes (grouped, contiguous, obfuscated, Unicode) close common bypass paths |
| **Operational triage** | `remove` for clear violations, `filter` for ambiguous cases, `modmail` for high-priority escalation |
| **False-positive control** | Exclusion clauses (`~body (regex)` and contextual guards) on broader detectors |
| **Auditability** | `action_reason` with `{{match}}` preserves traceability for every moderator action |
| **Policy traceability** | Rules map directly to subreddit rule numbers via section header comments |
| **Educational tone** | Removal comments guide users to the correct channel rather than threatening bans |
| **Community empowerment** | `!` commands let veteran members summon AutoMod help for new users |

---

## Deployment

### Prerequisites

- Moderator access to **r/amex**
- Access to the [old Reddit](https://old.reddit.com) interface
- Python 3.x with `pyyaml` (for validation only)

### Steps

1. **Validate** the config (see [Validation](#validation) below).
2. Navigate to [`old.reddit.com/r/amex/wiki/config/automoderator`](https://old.reddit.com/r/amex/wiki/config/automoderator).
3. Paste the full contents of `amex_automod_remediated.yml`.
4. Save with a clear, ASCII-safe revision reason.
5. **Monitor** after deployment:
   - Modqueue volume
   - Modmail alerts
   - False-positive rate from filtered content

> **Tip:** Deploy during active moderator coverage windows for staged rollout.

### Validation

Run this before every production change to verify YAML syntax, regex compilation, and structural integrity:

```bash
python - <<'PY'
import yaml, re
from pathlib import Path

text = Path('amex_automod_remediated.yml').read_text(encoding='utf-8')
docs = list(yaml.safe_load_all(text))

print(f"Total documents:  {len(docs)}")
print(f"Active rules:     {sum(isinstance(d, dict) for d in docs)}")
print(f"Empty documents:  {sum(d is None for d in docs)}")

# Check for problematic literal Unicode characters
zw_chars = ['\u200b', '\u200c', '\u200d', '\u200e', '\u200f', '\u2060', '\ufeff']
fw_digits = [chr(cp) for cp in range(0xFF10, 0xFF1A)]
print(f"Literal zero-width chars: {any(c in text for c in zw_chars)}")
print(f"Literal fullwidth digits: {any(c in text for c in fw_digits)}")

# Validate all regex patterns compile
errors = 0
for i, d in enumerate(docs, 1):
    if not isinstance(d, dict):
        continue
    for k, v in d.items():
        if '(regex' in k:
            for p in (v if isinstance(v, list) else [v]):
                if isinstance(p, str):
                    try:
                        re.compile(p)
                    except re.error as e:
                        print(f"  REGEX ERROR in doc {i}, key '{k}': {e}")
                        errors += 1
print(f"Regex validation: {'PASS' if errors == 0 else f'FAIL ({errors} errors)'}")
PY
```

This same script runs automatically via **GitHub Actions** (`.github/workflows/validate.yml`) on every push or pull request that touches `amex_automod_remediated.yml`.

---

## Moderator Guide

### Action Types

| Action | Meaning | What to Do |
|--------|---------|------------|
| `remove` | Clear policy violation — content removed automatically | Review `action_reason` in mod log for context |
| `filter` | Ambiguous — sent to modqueue for review | Re-approve false positives; escalate true violations |
| `report` | Flagged for attention | Review and take manual action as needed |
| modmail alert | High-priority signal (PII, scam, etc.) | Treat as incident response |

### Operational Workflow

1. **Monitor modqueue** for `filter`-action items requiring human review.
2. **Check modmail** for escalation alerts from PII/scam/high-risk detections.
3. **Re-approve** content that was incorrectly filtered (false positives).
4. **Promote** repeat abuse patterns from `filter` to `remove` in controlled increments.

### Making Changes

- Scope edits to **one section at a time**.
- **Validate** after every edit (see [Validation](#validation)).
- Document the reason and expected effect in comments near modified rules.
- Prefer **additive safeguards** over broad pattern relaxation.
- Avoid unrelated refactors during incident-driven fixes.

---

## YAML Syntax Reference

Key syntax rules for AutoModerator YAML:

1. Rules are separated by `---` on its own line with no indentation.
2. Use **spaces** for indentation (never tabs).
3. **Single-quote** regex strings for literal patterns.
4. Use **double-quoted** strings with `\uXXXX` when Unicode escaping is needed.
5. Avoid literal special characters that can break Reddit's AutoMod editor.
6. Keep keys **unique** within each YAML document (duplicates silently overwrite).

For full syntax documentation, see the [AutoModerator full documentation](https://www.reddit.com/wiki/automoderator/full-documentation) and the reference files in this repository.

---

## Reference Materials

Amex-Automod-Wiki includes a consolidated reference of Reddit's AutoModerator documentation for offline use:

| File | Contents |
|------|----------|
| `automod-reference.md` | All-in-one reference covering basic rules, inputs/modifiers/outputs, limitations, common mistakes, YAML syntax, error messages, and a library of common rule templates |

**Official documentation:**
- [AutoModerator Full Documentation](https://www.reddit.com/wiki/automoderator/full-documentation)
- [Reddit Help: AutoModerator](https://support.reddithelp.com/hc/en-us/articles/15484574206484)
- [Contributor Quality Score (CQS)](https://support.reddithelp.com/hc/en-us/articles/19023371170196)

---

## License

This project is licensed under the [MIT License](LICENSE).

Copyright © 2026 [AICincy](https://github.com/AICincy/Amex-Automod-Wiki)
