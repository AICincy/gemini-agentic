# gemini-agentic AutoModerator Configuration

This repository contains a production AutoModerator configuration for **r/amex** and supporting reference material used to validate syntax and behavior decisions.

Primary config file:

- `./amex_automod_remediated.yml`

Reference/source docs (snapshots):

- `./Reddit 1.yml`
- `./Reddit 2.txt`
- `./Reddit 3.txt`
- `./Reddit 4.txt`
- `./Reddit 5.txt`
- `./Reddit 6.txt`

---

## 1) What this configuration does

The AutoModerator file enforces subreddit policy with priority on:

1. **PII / doxxing prevention**
2. **Referral and fraud policy enforcement**
3. **Illegal/scam/external-link moderation controls**
4. **Submission quality controls**
5. **Community helper automation** (auto-replies for common topics)

The config is implemented as many YAML documents separated by `---`, where each document is one AutoModerator rule.

---

## 2) High-level architecture

The file is organized into ten sections (documented in header comments):

1. Shadowbans
2. PII / Doxxing Protection
3. Rule 1 (Referrals)
4. Rule 2 (Illegal/Links)
5. Account Quality
6. Rule 5 (Decorum)
7. Rule 3 (Quality)
8. Rule 4 (Moderator Interaction)
9. Report Handling
10. Community Helpers

### Rule execution model

AutoModerator evaluates checks and performs outputs/actions when checks match. This config uses:

- `action: remove|filter|report|approve` depending on risk/severity
- `priority` to influence ordering where needed
- `modmail_subject` + `modmail` for moderator escalation context
- `action_reason` (including `{{match}}`) for auditability in mod tooling
- `type: submission|comment|any` where scope control is needed
- `~check` negation for safe exclusions / false-positive control

---

## 3) Section-by-section behavior

## Section 1: Shadowbans

- Implements username-targeted removal for designated accounts.
- Uses placeholder `author: [INPUTNAMEHERE]` in template form and removal action.

## Section 2: PII / Doxxing Protection

Defensive rules for sensitive data leakage and evasion:

- Credit card detection:
  - issuer-aware grouped patterns (Visa/MC/Amex/Discover/Diners/JCB)
  - per-digit separated obfuscation detection
  - contiguous 13–19 digit detection with targeted exclusions
- SSN pattern detection with invalid-range exclusions
- US phone number detection with exclusion filters
- Email detection:
  - standard `user@domain`
  - bracketed obfuscation formats (`[at]`, `(dot)`, etc.)
  - bare `AT/DOT` obfuscation format
- Mailing address detection
- Edited-content re-check (`is_edited: true`) to catch edit-after-post bypasses
- Unicode evasion detection:
  - zero-width character detection
  - fullwidth/superscript/subscript/circled numeral detection

## Section 3: Rule 1 (Referral Protocol)

- Enforces referral posting constraints and ban-level policy cues.
- Removes and escalates violations with modmail guidance.

## Section 4: Rule 2 (Illegal Activity / Links)

- Blocks prohibited external-link and scam patterns.
- Escalates high-risk language and action-relevant contexts.

## Section 5: Account Quality

- Uses account-level indicators (e.g., account age/karma style checks where configured) to reduce low-quality abuse/spam ingress.

## Section 6: Rule 5 (User Decorum)

- Detects abusive language/slurs/threat-like content classes.
- Removes/filters and escalates to moderators for ban decisions where applicable.

## Section 7: Rule 3 (Submission Quality)

- Enforces posting quality standards (low-effort/FAQ-type patterns).
- Directs users to research resources and clearer post structure.

## Section 8: Rule 4 (Moderator Interaction)

- Supports rules around moderator authority/interaction expectations.

## Section 9: Report Handling

- Handles report-triggered workflows to route suspicious or high-signal items to moderators.

## Section 10: Community Helpers

- Non-enforcement helper automations that provide standard guidance on frequent topics (acronyms, pop-up jail, retention offers, transfer partners, financial review, etc.).

---

## 4) Why the configuration is implemented this way

- **Defense in depth:** Multiple pattern classes (grouped, contiguous, obfuscated, Unicode-evasive) close common bypass paths.
- **Operational triage:** `remove` for clear violations, `filter` for review-needed ambiguity, modmail for high-priority escalation.
- **False-positive reduction:** Exclusion clauses (`~body (regex)` and contextual guards) are used on broader detectors.
- **Auditability:** `action_reason` and placeholders preserve traceability for moderator review.
- **Policy traceability:** Rules map directly to subreddit policy categories in top comments.

---

## 5) YAML and AutoModerator syntax requirements (critical)

1. Rules are separated by `---` with no indentation.
2. Use consistent indentation spaces (no tabs).
3. Quote regex strings appropriately:
   - single quotes are usually safer for regex literals
   - double-quoted strings may be used intentionally for `\uXXXX` escaping
4. Avoid unsupported literal special characters that can break save behavior in AutoMod editors.
5. Keep keys unique within each YAML mapping (duplicate keys overwrite or invalidate intent).

---

## 6) Deployment procedure (production)

1. Open old Reddit config page:
   - `https://old.reddit.com/r/<subreddit>/wiki/config/automoderator`
2. Paste full contents of `amex_automod_remediated.yml`.
3. Save with a clear revision reason (ASCII-safe text recommended).
4. Monitor:
   - modqueue volume
   - modmail alerts
   - false-positive rate from filtered content

For staged rollout, deploy during active moderator coverage windows.

---

## 7) Validation workflow before every production change

Because this repository has no app/test framework, validate with Python checks:

```bash
cd /path/to/gemini-agentic
python - <<'PY'
import yaml, re
from pathlib import Path
text = Path('amex_automod_remediated.yml').read_text(encoding='utf-8')
docs = list(yaml.safe_load_all(text))
print('docs_total', len(docs))
print('rule_docs', sum(isinstance(d, dict) for d in docs))
print('empty_docs', sum(d is None for d in docs))
print('regex_keys', sum(1 for d in docs if isinstance(d, dict) for k in d if '(regex' in k))
print('has_literal_zero_width', any(c in text for c in ['\u200b','\u200c','\u200d','\u200e','\u200f','\u2060','\ufeff']))
print('has_literal_fullwidth_digits', any(c in text for c in '０１２３４５６７８９'))
for i, d in enumerate(docs, 1):
    if not isinstance(d, dict):
        continue
    for k, v in d.items():
        if '(regex' in k:
            pats = v if isinstance(v, list) else [v]
            for p in pats:
                if isinstance(p, str):
                    re.compile(p)
print('regex_compile_ok')
PY
```

Also run a duplicate-key YAML loader check when making structural edits.

---

## 8) Operational guidance for moderators

- Treat **remove** actions as policy-enforced and review `action_reason`.
- Treat **filter** actions as triage queue candidates; re-approve when false positives are confirmed.
- Use modmail alerts for incident response (PII/scam/high-risk signals).
- For repeat abuse patterns, promote detections from `filter` to `remove` in controlled increments.

---

## 9) Change-management recommendations

- Keep edits scoped to one section at a time.
- Validate after each edit set.
- Record reason for change and expected effect in comments near modified rules.
- Prefer additive safeguards over broad pattern relaxation.
- Avoid unrelated refactors during incident-driven fixes.

---

## 10) Repository intent and limitations

- This repository is configuration-centric (no runtime service or packaged application).
- Production correctness depends on:
  - YAML validity
  - AutoModerator-supported syntax
  - Moderator operational response

If unsure about syntax behavior, verify against the AutoModerator documentation snapshots in this repository before deploying.
