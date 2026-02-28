# USER.md — Owner Profile & Notification Preferences

*ZeroClaw reads this file every session to understand you.*

**Location:** `/Users/agent/.zeroclaw/workspace/AMII.md`
**Read by:** Steph (before every writing task), Marcus (before every deployment notification), Axel (before every escalation), all agents (when owner notification is required).
**Maintained by:** Steph updates the Voice Notes section after every edit delta.
Axel updates the Notification Preferences section if the owner changes her preferences.
**Last Updated:** February 2026

---

## Who Amii Is

Amii is the founder and owner of Projex. She is semi-tech-literate — she understands product decisions and user needs deeply, but does not need to understand implementation details unless they affect her decisions. All agent communications to Amii should be in plain English with no unexplained jargon.

She is the only person who can:
- Approve Phase gate transitions (Phase 1 → Phase 2, etc.)
- Authorise exceptions to blocked tasks
- Approve new agent SKILL.toml files
- Confirm owner-level memory nodes (confidence = 0.85)
- Approve max iterations escalations

---

## Contact & Notification

**Primary contact:** requested2019@gmail.com
**Authorised send:** Steph may send directly to this address using the send_email tool. All other recipients require Axel review before sending.

**Notification format preference:** Plain English. Lead with the decision she needs to make or the thing she needs to know. Do not bury the action item.

**Response window expectation:**
- Urgent (blocked task, production incident): same day
- Standard (status update, flag): within 24 hours
- Low priority (copy feedback, style notes): within 48 hours

**Do not send:** Multiple messages about the same issue within 4 hours unless it's a production incident. Consolidate if possible.

---

## Deployment Notifications

Marcus reads this section before every deployment notification.

**Before deployment:**
> Deploying [feature name] now. Expected downtime: [none / X minutes].
> I'll confirm once it's live.

**After successful deployment:**
> [Feature name] is live. Everything looks stable. No issues.

**After failed deployment / rollback:**
> [Feature name] deployment didn't go as planned — I've rolled back to the previous version.
> Users are unaffected. I'm investigating. Will update you within [timeframe].

Keep it short. Amii does not need technical details in deployment notifications unless something went wrong and she needs to make a decision.

---

## Escalation Notifications

Axel reads this section before every escalation or flag.

Use the FLAG format from axel-SKILL.toml. Always include:
- What the problem is (plain English)
- What she needs to decide (one clear question or set of options)
- What happens if she doesn't respond (so she understands urgency)
- Response needed by: [specific datetime]

Do not send an escalation without a clear decision request. Amii should never have to ask "so what do you need from me?" — that should already be answered.

---

## Voice Profile — Living Document

Steph maintains this section. Update after every edit delta received from Amii.
Add new confirmed patterns below. Never remove a pattern without 3+ contradicting edits.

### Confirmed patterns (confidence ≥ 0.75)
*These have been reinforced or approved. Apply with confidence.*

- Uses em dashes for asides — not parentheses, not commas.
- Short sentences are preferred. Vary for rhythm: short-short-medium.
- Active voice always. Passive voice only when the subject genuinely doesn't matter.
- Direct sign-offs. No "Kind regards." Context-appropriate — match the tone of the email.
- Numbers: uses numerals for anything 10+, words for 1–9 (standard style).

### Provisional patterns (confidence 0.60–0.74)
*These are emerging. Apply, but note if they feel off.*

*(Add new patterns here as you observe them. Move to Confirmed after 3+ reinforcements.)*

### Hard exclusions (never use)
*Patterns Amii has edited out 3+ times, or explicitly excluded.*

- "I wanted to reach out" — too soft, too vague
- "As discussed" — sounds passive-aggressive in email
- "Please find attached" — stiff and dated
- "Do not hesitate to contact me" — too formal
- "I trust this email finds you well" — never
- "Going forward" — corporate filler
- "Leverage" / "Utilise" / "Synergy" / "Circle back" — jargon, always
- Exclamation marks in professional emails — too eager

---

## Edit Delta Log

Steph logs edit deltas here when Amii returns a revised document or provides explicit feedback.
Format: [date] — [what changed] — [pattern extracted]

*(Steph appends to this section after every edit delta received.)*

---

## Authorised Calendars

- Home
- Work

Do not create events in any other calendar without Amii's explicit instruction.
Never delete or modify events Amii created herself.

---

*This file is maintained by Steph (Voice Profile, Edit Delta Log) and Axel (Notification Preferences).
Changes to contact details must be confirmed by the owner before taking effect.*

---
*Update this anytime. The more ZeroClaw knows, the better it helps.*
