# 📢 Mod Announcement: r/amex AutoModerator has been significantly upgraded — here's what changed and why

**Flair: Mod Announcement**

---

Hey r/amex,

We've just deployed a major upgrade to the subreddit's AutoModerator configuration. If you've been here a while, you may have noticed the old bot was pretty minimal — it caught a few things but left a lot of gaps. The new version (v5.1) is a complete overhaul with smarter protections, a proper community helper system, and much cleaner enforcement aligned with our five subreddit rules.

This post walks you through what changed, what stayed, and — most importantly — **what it means for you as a member**.

---

## 🔒 What's New: Dramatically Improved Privacy Protection

This is the biggest upgrade. The old config had two basic PII rules:

- A simple email pattern (`word@word.word`) that missed obfuscated addresses
- A credit card pattern that only covered four card types and had no separator flexibility

**The new config has 11 layered PII rules** covering every common exposure vector:

| What's now detected | Old config | New config |
|---|---|---|
| Credit card numbers | 4 basic card types (Visa, MC, Amex, Discover only) | All 6 networks (+ Diners Club, JCB) with flexible separator handling |
| Cards written with dots/dashes between digits | ❌ | ✅ Caught |
| Cards written digit-by-digit (e.g., `4 1 2 3 ...`) | ❌ | ✅ Caught |
| Long digit dumps (e.g., raw 16-digit strings) | ❌ | ✅ Caught (filter, not remove) |
| Social Security Numbers | ❌ | ✅ Caught and removed |
| US phone numbers | ❌ | ✅ Caught (with common false-positive exclusions) |
| Email addresses | Basic only | ✅ Standard, spaced-@ bypass, and obfuscated (`user [at] domain [dot] com`) |
| Mailing addresses | ❌ | ✅ Caught (filter for mod review) |
| PII added via **post editing** | ❌ | ✅ Edited content is re-scanned |
| Unicode evasion (invisible chars, fullwidth digits) | ❌ | ✅ Caught |
| PII hidden inside blockquotes | ❌ | ✅ Secondary scan covers blockquoted content |

**Why does this matter?** Occasionally members post card numbers accidentally (e.g., when uploading screenshots of denial letters or billing disputes). In those cases, the bot will now **remove the post automatically and notify you why**, so you can re-share without the sensitive data. Moderators are also alerted for high-risk removals.

If your post is removed with a message about PII, simply remove the sensitive information and repost.

---

## 🔗 Rule 1 (Referrals): More Comprehensive Enforcement

The old rule was a single pattern: if your post contained `refer.amex`, it got removed. That's it.

**The new config adds three referral-related rules:**

1. **Direct referral links** — catches `refer.amex`, Amex MGM links, co-brand referrals (Hilton, Delta, Marriott, etc.), and explicit "referral link/code" language
2. **Veiled solicitation** — catches phrasing like *"DM me for a referral,"* *"check my profile for a referral,"* or *"happy to send a referral"*
3. **DM/PM solicitation** — catches *"slide into my DMs,"* *"inbox me,"* *"my DMs are open,"* and similar language (often a referral workaround)

All referral sharing belongs in the **designated pinned Referral Thread**. That's where members who are actively looking for referrals will find your links. Outside that thread, violations result in removal and — per Rule 1 — are subject to a permanent ban.

**The referral thread itself now gets a sticky welcome comment** when it goes live each month, reminding everyone of the 30-day rule and the one-comment-per-member limit.

---

## 🌐 Rule 2 (Links): Full Domain Allowlist

The old config had no external link enforcement.

**The new config blocks all external links** except a specific allowlist of trusted domains:

| Category | Approved domains |
|---|---|
| American Express | americanexpress.com, amex.com |
| Reddit | reddit.com, redd.it |
| Rewards | rakuten.com |
| Credit News | doctorofcredit.com, nerdwallet.com, thepointsguy.com, wallethub.com, bankrate.com |
| Credit Scores | myfico.com, fico.com, experian.com, equifax.com, transunion.com, annualcreditreport.com |
| Media | imgur.com |
| Co-Brand Partners | delta.com, hilton.com, marriott.com |

The bot also now detects **phishing attempts** (typosquatted Amex domains, deceptive markdown links) and **financial scam language** (wire transfers, gift card payments, crypto "opportunities"). Both are immediate removals and permanent ban offenses under Rule 2.

If you're linking to a site you think should be on the allowlist, [message the mods](https://www.reddit.com/message/compose?to=%2Fr%2Famex) and we'll consider adding it.

---

## 📊 Account Quality Gates: Now Uses Reddit's Native Scoring

The old config used simple thresholds: `combined_subreddit_karma < 500` to catch low-karma accounts posting links, and `combined_karma < 5 AND account_age < 30 days` for new accounts.

**The new config uses Reddit's native Contributor Quality Score (CQS)**, which is a platform-level signal that considers the full context of an account's history. Posts and comments from accounts below the "low" CQS threshold are sent to the modqueue for review (not auto-removed).

Additional quality gates:
- New accounts (combined karma < 10, age < 30 days) cannot submit posts — they're redirected to our pinned **Weekly Free Talk / Newbie Thread** instead
- Brand-new commenters (karma < 5, age < 7 days) are sent to the modqueue
- Accounts with combined karma below −25 are treated as troll accounts and removed
- Aged accounts (> 1 year old) with near-zero karma are flagged for review

---

## 🛡️ New: Rule 3 (Quality), Rule 4 (Mods), and Rule 5 (Decorum) Enforcement

The old config had no enforcement for these three rules. They were entirely manual.

**Rule 3 — Submission Quality** now auto-enforces:
- Posts with no flair are removed and prompted to add one before resubmitting
- Low-effort titles (single words like "Help," "Question," "Amex") are removed
- All-caps titles are removed
- Image posts with no context text are removed
- Common FAQ-pattern titles (e.g., "What card should I get?", "Is it worth getting the Platinum?") are filtered and directed to the Monthly FAQ

**Rule 4 — Moderator Interaction** is now lightly monitored. Comments that express frustration about moderation in heated terms are filtered for review (not auto-removed).

**Rule 5 — User Decorum** now has three AutoMod-backed rules:
- **Hate speech/slurs**: Auto-removed, no modmail (silent to reduce alert fatigue)
- **Personal attacks and insults**: Filtered for mod review, with a comment directing the user to use the block/report features
- **Threats**: Auto-removed with immediate modmail to moderators

---

## 📋 Report Handling: Changed from Auto-Remove to Human Review

The old config **auto-removed** anything that received 2+ reports.

**The new config sends items with 3+ reports to the modqueue** for human review and notifies mods via modmail. We made this change because auto-removal on reports is easily gamed — any coordinated group could get posts auto-removed simply by mass-reporting them. Moderators now review all reported content before action is taken.

The one exception: the **Referral Thread** uses a 2-report threshold for comments within that thread specifically, since duplicates are easier to verify.

---

## 🤖 Community Helper Bot: Now with 19 Commands

The old config had passive triggers — if you mentioned certain keywords ("jail," "not eligible," "return protection," etc.), the bot would reply automatically.

**The new system is primarily command-driven.** Type a `!` command in any post or comment to summon a reply:

| Command | What you get |
|---|---|
| `!help` | Full list of all commands |
| `!acronyms` | Common r/amex acronym definitions (AF, MR, SUB, FR, NLL, etc.) |
| `!popup` or `!jail` | Pop-up jail explanation + workaround |
| `!recon` | Amex reconsideration line tips |
| `!retention` | Annual fee retention call tips |
| `!fr` or `!financialreview` | Financial Review guidance |
| `!status` | Application status check |
| `!contact` or `!support` | Amex contact directory |
| `!transfer` | Membership Rewards transfer partners |
| `!offers` | Amex Offers overview |
| `!trifecta` | Platinum + Gold + BBP strategy |
| `!lounge` or `!centurion` | Airport lounge access by card |
| `!rules` | Subreddit rules summary |
| `!return` | Return protection policy link |
| `!purchase` | Purchase protection policy link |
| `!warranty` | Extended warranty coverage link |
| `!trip` | Trip cancellation/delay insurance link |
| `!cellphone` | Cell phone protection link |
| `!car` | Car rental insurance link |

We kept a few passive triggers for the most common scenarios (pop-up jail, reconsideration), but the command-based approach is cleaner — it reduces noise in threads where someone mentions "return protection" in passing without actually needing the bot to respond.

As a member, you can use any of these commands to help fellow members. Veteran members: feel free to drop `!popup` or `!acronyms` when you see someone who would benefit.

---

## ❓ What Should You Do Differently?

In practice, most members won't notice anything different. A few things to keep in mind:

- **Referral links go in the pinned thread only.** Veiled requests elsewhere will now be caught.
- **Don't post card numbers, SSNs, or phone numbers in your posts.** The bot will remove them and notify you.
- **External links should be to approved domains.** If you're quoting a source, make sure it's on the list.
- **Add a flair to your posts.** The bot now enforces this.
- **Use the `!help` command** to see what the community helper bot can do for you.

If a removal feels like an error, you can always [message the mods](https://www.reddit.com/message/compose?to=%2Fr%2Famex). We review the modqueue regularly and approve legitimate content that was over-filtered.

---

## 🛠️ For the Curious: Full Config is Public

The full AutoMod configuration is maintained openly at [github.com/AICincy/Amex-Automod-Wiki](https://github.com/AICincy/Amex-Automod-Wiki). If you're a mod on another subreddit or just interested in how it works, the repository includes a full reference guide and CI validation pipeline.

---

Thanks for being part of r/amex. As always, mod decisions are final, and Rule 5 applies to how you interact with us too. Questions? Drop them below — or for private matters, use the mod mail link.

*— The r/amex Mod Team*
