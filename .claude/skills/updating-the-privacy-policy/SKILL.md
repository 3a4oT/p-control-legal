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

## Do not fabricate

The server's own data-protection spec says full legal-basis documentation, policy text and DPA
text are real legal work. This skill gets the page **accurate to the code** and structurally
complete; it does not make it lawyer-reviewed. Two values are deliberately left as placeholders
until a human fills them, and both block publication:

- the developer name exactly as it appears on the Play listing;
- the privacy contact address.

The effective date stays a placeholder until the page is actually published.

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
