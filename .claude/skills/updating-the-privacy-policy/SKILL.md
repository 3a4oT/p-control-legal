---
name: updating-the-privacy-policy
description: Use when a change alters what p-control collects, transmits, stores, retains, or who it reaches — a new field on the wire, a new SDK, a new processor, a new permission, a new admin surface — or when preparing a Play submission. Covers what Google actually requires, what GDPR Art. 13 adds, and the in-app disclosure a policy page cannot substitute for.
---

# Updating the privacy policy and the Data safety answers

Two artifacts, one repo (`p-control-legal`), and they must agree with each other **and with the
binary**:

- `index.html` — the published page. Its URL is the Privacy Policy link on the Play listing.
- `data-safety-form.md` — a working document holding the draft answers to Play Console's
  structured form. Never published.

A published policy that no longer matches the app is a **Play policy violation**, not a
documentation gap. That is the whole reason this skill exists.

## Trigger: change the answer to any of these, and both files need editing

1. A new field on a `control/v1` message that leaves the device.
2. A new SDK, or a new company in the request path (a host, a CDN, an analytics vendor).
3. A new Android permission in the merged manifest.
4. A new identifier of any kind.
5. A new surface that handles household data (the web admin panel is the pending one).
6. Retention changing anywhere.

**And the inverse, which is the one people skip:** a claim in the policy that the app *stopped*
doing, or never started. Overstating collection is as wrong as understating it — the Data safety
form has to match observed behaviour, and Google can and does check.

## What Google actually requires of the page

Five elements, from Play Console Help's User Data policy — not a style preference:

1. Developer information and a privacy contact, or a mechanism to submit inquiries.
2. The types of personal and sensitive data accessed, collected, used and shared, **and the
   parties it is shared with**.
3. Secure data-handling procedures.
4. The data retention and deletion policy.
5. Labelled clearly as a privacy policy.

Note what is *not* there: no requirement to name the API a value came from. `PACKAGE_USAGE_STATS`
belongs on the page because a reader recognises the permission; `Settings.Secure.ANDROID_ID` does
not — "a device identifier Android provides to this app" is the disclosure, and the exact symbol
lives in `data-safety-form.md`, where it keeps the form answerable against the code. A page that
names one symbol and misses another reads as a complete inventory when it is not.

## The requirement a privacy policy cannot satisfy

**Prominent disclosure and consent.** Where collection could exceed what a user expects —
explicitly including **background collection**, which is exactly what `MonitorService` does —
Play requires an *in-app* disclosure that:

- appears during normal use, not buried in a menu or a settings screen;
- describes the data being collected **and how it is used and shared**;
- is followed by an affirmative action (a tap, a checkbox) that is not navigation-away and not an
  auto-dismissing toast;
- **cannot rely on the privacy policy or the terms of service to carry it.**

So "we updated the policy" never closes this one. Check the onboarding copy, not the page.

## Identifiers: which restriction actually applies

Play restricts linking **persistent hardware identifiers** (IMEI, IMSI, SIM serial) to personal
data, permitting it in only two cases — telephony tied to a SIM identity, and enterprise device
management in device-owner mode. p-control is neither, and must never touch those.

`Settings.Secure.ANDROID_ID` is **not** in that category: since Android 8 it is scoped per
app-signing-key and per user profile, so it cannot correlate a device across apps. It is still a
**Device or other IDs** entry on the Data safety form. Scoping changes the privacy impact, never
the disclosure obligation.

## What GDPR Art. 13 adds on top of Google's five

The page needs these too, and Google's list does not mention them:

- who the controller is, by name;
- the **legal basis** per purpose;
- international transfers and the safeguard relied on;
- the right to object, restrict, port, and to complain to a supervisory authority.

For the legal basis, **performance of a contract** is the right ground for enforcement,
pairing, profiles and approvals — it is the service the parent installed the app to obtain, and
the same ground comparable parental-control products rely on (Qustodio states it explicitly).
Crash reporting and count-only analytics sit on legitimate interests, which means the page has to
offer a way to object.

## Children's data — say what is unresolved

The parent is the one contracting; a child never creates an account, never touches Telegram, and
supplies nothing directly. `p-control-server`'s `2026-08-08-web-ui-data-protection-design.md`
deliberately leaves two things unresolved, and the page states them rather than implying they are
handled: the child's own rights are reachable only through the parent's account, and there is no
"teen" case. Whether that is sufficient depends on jurisdiction (COPPA under 13, GDPR-K 13–16
depending on member state) and is legal work, not an architectural claim.

## How it is written, so the next feature does not need a lawyer

A policy written like a changelog of the code has to be reissued every sprint, and each reissue is
a chance to contradict the last one. A policy written like marketing says nothing and fails review.
The shape that survives both is **category and purpose, with the implementation left out** — and it
is a drafting style, not a way of hiding anything.

**Write one level above the code.** «A device identifier Android provides to this app» covers the
symbol, the day the symbol changes, and the day a second platform supplies its own. Adding a field
inside a category the page already describes is then an edit to `data-safety-form.md` alone. What
still forces an edit to the page: a new **category** of data, a new **purpose**, a new **recipient**,
a longer **retention**, or a change in **whether something is optional**.

**«May» is for a genuine option, never for something already done.** A regulator reads «we may
collect» as «we collect», and Play reads the Data safety form as the truth whatever the page says.
So «may» belongs where the answer really depends on the reader's own choice or on a setting of this
installation («if you link a Telegram chat», «where this installation requires it») — and the page
says which position is in force today. Anywhere else, write the present tense.

**Name the recipients you use, in a sentence that survives replacing one.** GDPR allows categories
of recipients; review — Google's and a regulator's — goes better when the ones in the path today are
named. «Hosting in Germany (Hetzner), or another provider of the same kind that replaces it» keeps
both: a reader learns who holds their data, and swapping a host is not a policy amendment.

**Keep the numbers in one table.** Retention is the one place vagueness reads as evasion, and it is
also the cheapest thing to keep exact: one table, one row per category. A number that changes is an
edit you have to make, and that is the deal the rest of the style buys.

**Say what is optional, in the reader's terms.** «Only if you link Telegram» is worth more than a
paragraph of qualifiers, and it is what makes an **Optional** answer on the Data safety form true.
The moment a setting makes that data a condition of using the product, the page says so and the form
flips to **Required** in the same change — the two answers are read side by side.

**Every version is dated and says what changed in one sentence.** That block is what proves the page
tracked the product rather than being rewritten under pressure, and a reader who saw the old text
can tell in ten seconds whether the change touches them.

**What the style never buys:** a missing legal basis, an unnamed purpose, a retention nobody stated,
or an Optional answer for data the product actually requires. Those are the four things transparency
enforcement looks for first, and vagueness there is what regulators call out
([EDPB's 2026 transparency focus](https://www.nixondigital.io/blog/edpb-transparency-enforcement-2026/)).
On the Play side the recurring rejection is **drift** — a form that still describes last year's
behaviour, an account-deletion link that is missing or broken, or a policy URL that does not resolve
in every locale
([2026 rejection reasons](https://qawerk.com/blog/google-play-rejection-reasons/)).

## Model it on a shipped product, not on instinct

The page is written; do not start it over from a template. When a new obligation appears, look at
how a comparable parental-control product words the same thing before inventing phrasing —
Qustodio's family policy is the reference used so far, and three of its patterns are already in
our page: performance-of-a-contract as the ground for the monitoring itself, concrete retention
per category rather than one blanket sentence, and an explicit undertaking by the parent that
they have told the monitored person the software is there. That last one is the pattern most
easily missed: monitoring software should not be a secret from the person monitored, and the
policy is where the parent accepts that duty.

## What is still genuinely undone

Two things, and neither may be described as handled:

- **No lawyer has reviewed the text.** It is accurate to the code and structurally complete
  against Play's and the GDPR's requirements. That is not the same as legal review, and the
  difference matters most for the children's-data framing, where COPPA and national GDPR
  implementations diverge.
- **There is no DPA with any processor** (Google, Telegram, Hetzner, Cloudflare).

The controller is named as an individual developer with an email contact and no postal address.
That is normal for an indie developer and is what the page says; if the publisher ever becomes a
company, §1 and §15 both change.

## Procedure

1. Diff the app's behaviour, not its docs: what does the binary now send, store or receive?
2. Update `index.html` in categories; update `data-safety-form.md` with the symbol-level detail
   and the Data safety row.
3. Re-read the page's own contradictions — the §3 table, §5's list of recipients, §7's retention.
   A table near the top asserting the old thing is the usual failure.
4. Check the in-app prominent disclosure separately. The page does not cover it.
5. State plainly, in the PR, which judgment calls remain the human's.

## Sources

- [Play Console Help — User Data](https://support.google.com/googleplay/android-developer/answer/10144311)
- [Qustodio family privacy policy](https://www.qustodio.com/en/family/privacy/) — the
  performance-of-contract precedent for a parental-control product.
