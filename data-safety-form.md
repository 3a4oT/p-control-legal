# Play Console — Data safety form (draft answers)

Play Console → App content → Data safety. This is a structured questionnaire, not free text —
below maps each relevant data type to the answer this app's actual behavior supports. **You are
the one accountable for this form's accuracy to Google — read each row against the code before
submitting, don't paste blind.**

## Does your app collect or share any of the required user data types?

**Yes.**

## Data types

| Data type | Collected? | Shared? | Purpose | Optional? |
|---|---|---|---|---|
| **Personal info → Name** | Yes (child profile name, entered by the parent) | No (not shared with third parties — only stored on the household's own self-hosted server, which the developer, not a third-party company, operates) | App functionality | Required for the "who's watching" feature |
| **App activity → App interactions** | Yes (which app was foregrounded, to enforce blocking — read on-device only). Additionally, when a child uses "ask a parent", the identifier of that one app is sent to the household server so the parent knows what they are approving. | No | App functionality | Required — core purpose |
| **App activity → In-app search history** | No | — | — | — |
| **App info and performance → Crash logs** | Yes (via Firebase Crashlytics) | Yes — with Google (Firebase, as the analytics/crash provider) | Analytics | Not user-facing/optional — standard crash reporting |
| **App info and performance → Diagnostics** | Yes (via Firebase Crashlytics — device/OS info attached to crash reports) | Yes — with Google (Firebase) | Analytics | Same as above |
| **Device or other IDs → Device or other IDs** | Yes — **two separate sources**, see the note below | Yes — with Google (Firebase) for the app-instance ID. The pairing identifiers go only to the household server (same judgment call as the profile-name row) | Analytics (Firebase) **and** App functionality (pairing) | Not optional — the app cannot pair without sending them |
| **Location** | No | — | — | — |
| **Financial info** | No | — | — | — |
| **Health and fitness** | No | — | — | — |
| **Messages** | No | — | — | — |
| **Photos and videos** | No | — | — | — |
| **Audio files** | No | — | — | — |
| **Files and docs** | No | — | — | — |
| **Calendar** | No | — | — | — |
| **Contacts** | No | — | — | — |
| **Web browsing history** | No | — | — | — |

### Note on "Device or other IDs" — read this before answering that row

Two different identifiers, with different destinations. Google's form has one row for both, so
answer it for the union and be ready to explain the split:

1. **Firebase Analytics' app-instance ID** — SDK default, goes to Google, not the advertising
   ID, not used for cross-app tracking.
2. **Pairing identifiers, added when the app gained a real pairing flow** — a random
   per-installation UUID, the device model string, and `Settings.Secure.ANDROID_ID`. These go to
   the household server only, and only after a parent confirms the pairing in Telegram; an
   unpaired app transmits nothing.

The API name belongs here and **not** on the public policy page, which says "a device
identifier Android provides to this app" instead. Neither Play's User Data policy nor GDPR
Art. 13 asks a policy to name the API it read — they ask for the *category*, the purpose and the
recipients. Keeping the exact symbol in this working document is what makes the form above
answerable and re-checkable against the code; putting it on a published page would mean every
refactor is a legal-document edit, and a page that names one symbol and misses another reads as
a complete inventory when it is not.

`ANDROID_ID` is the one that needs care in the answer. On Android 8 and later it is scoped per
app-signing-key and per user profile, so it cannot correlate this device across other apps. **It
is still a device identifier and must be declared as one** — the scoping changes the privacy
impact, not the disclosure obligation.

It is sent by the device for one reason and used by the server for two, and the second is the one
to keep an eye on, because a *purpose* is what Art. 13 asks about and a new purpose for an
existing field is a real change even when nothing new leaves the device:

1. **Recognising a reinstall**, so the same television returning does not strand the old device
   record offline forever. Scoped to the confirming person's own households.
2. **Recognising that a device registered to one household has been claimed by another**, so the
   household that loses it is told at the moment of the claim rather than inferring it from
   silence three days later. This one is matched **globally**, across households — which is why
   what it may disclose is deliberately asymmetric: the losing household is told about **its own**
   device and the time, and learns nothing about who claimed it; the claimant is told only that
   the machine is registered somewhere else, never where or to whom.

Neither adds a field to the wire, a recipient, or a retention period — `control/v1` is unchanged
and the anchor never leaves the household service. `p-control-server`'s
`2026-08-14-device-reinstall-design.md` owns the design and records the rule that the anchor is
never shown to a human and never accepted as input, so it cannot become a search index across
families.

**Judgment call left to a human:** whether purpose 2 needs its own line in the Data safety form's
purpose selection, or is covered by the existing "App functionality" answer. It is app
functionality on any reading, and no new data type or recipient appears — so this document treats
the existing answer as still correct and flags it rather than deciding it.

## What the current app version does NOT transmit

Worth stating because the privacy policy's §3 table describes the household service's full
design, part of which the app does not implement yet. As of this app version, the TV sends **no**
usage history, **no** screen-time totals, **no** installed-app list, and **no** online/offline
status. If a later release starts sending any of them, this form and privacy-policy §3 both need
updating before that release ships.

## Prominent disclosure and consent — satisfied by a screen, not by this form

**Play requires an *in-app* disclosure** wherever collection could exceed a user's expectations,
and names background collection explicitly — which is what `MonitorService` does. It must appear
during normal use, describe the data **and how it is used and shared**, and be followed by an
affirmative tap. Google states it cannot be satisfied by the privacy policy or the terms of
service, so nothing in this document or on the published page closes it.

**It is built.** `p-control-android` shows `DataDisclosureScreen` as the first screen of first run
(`disclosure_leaves_device`, `disclosure_sharing`, `disclosure_accept`), stores the accepted
revision, and `MainActivity.startMonitorServiceIfPermitted` gates the foreground service on it —
so enforcement cannot begin before the parent has been told. The accepted revision is what
`DeviceStatus.disclosure_version` carries. Its rules live in that repo's
`docs/superpowers/specs/2026-08-13-in-app-data-disclosure-design.md`; **the revision moves only
when what leaves the device widens**, which is the check to run against every row of this form.

## The account, and the email address it collects

**LIVE since 14 August 2026.** A parent signs in at `control.rovenskyi.com` with an **email
address and a password**, tenancy keys on that account, and a Telegram chat is an optional linked
channel rather than the identity. Play's *App access* declaration is one of the reasons a reviewer
needs unchanging, reusable, location-independent credentials, which a Telegram account cannot be.

The row this adds to the table above:

| Data type | Collected | Shared | Purpose | Notes |
|---|---|---|---|---|
| **Personal info → Email address** | Yes (the parent's, at sign-up) | No | App functionality (account management, authentication) | Required. It identifies the account that owns a household and is how the developer reaches that parent about it. The password is not a Data safety data type — it is a credential, stored only as a scrypt hash — but it belongs in the security section below |

**The first account is not created through an open sign-up form.** It comes from first-run setup,
gated on a token the server prints once to its own log; every account after it is created by
redeeming an invite. There is no public registration endpoint, and the form should not be answered
as though there were one.

**What this row does NOT cover, deliberately.** Installed apps and per-app usage totals are still
not sent by the shipped binary — the server can receive and store them and no released client
publishes them. The rows above say so, and they must keep saying so until that code ships. A form
that overstates collection is wrong in exactly the way one that understates it is.

## Security practices section

- **Is all user data encrypted in transit?** Yes — MQTT connection to the household server uses
  TLS; Firebase SDK traffic is HTTPS by default.
- **Do you provide a way for users to request data deletion?** Yes, at three levels. Unpairing a
  device from the web panel clears the credentials it holds. Signing out everywhere ends every
  browser session at once. For the account and the whole household — the email address, the
  password hash, the profiles, the rules and the history — a parent contacts the developer
  directly, and that request is answered by a person (privacy policy §11). **There is no
  self-service account deletion in the panel yet**; say so rather than implying a button exists.
- **Data collection is required or can users opt out?** Core enforcement data (app activity) is
  required — it's the product's function. Analytics/crash reporting (Firebase) currently has no
  in-app opt-out toggle; note this honestly rather than claim one that doesn't exist. If an
  opt-out is added later, update this section then.

## Two judgment calls flagged for your decision, not assumed

1. **Is the household's own self-hosted server a "third party" for Play's purposes?** Google's
   definition centers on *other companies*; a service the developer personally operates for
   their own users arguably isn't one — I've marked profile-name sharing "No" on that reading.
   If you'd rather declare it transparently as shared regardless, flip that row to "Yes,
   shared with: service provider" — it's a defensible, more conservative choice.
2. **Firebase Analytics/Crashlytics rows above assume default SDK behavior** (no consent-gated
   collection, no ad personalization use). If either SDK's config in this app has been
   customized beyond what `FirebaseAnalyticsRepository`/`FirebaseCrashReportingRepository`
   show, re-verify against Firebase's own current data-safety mapping guide before submitting.
