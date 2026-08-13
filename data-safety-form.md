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
app-signing-key and per user profile, so it cannot correlate this device across other apps. It
is sent for one purpose: recognising that a reinstall of p-control is the same television
returning, instead of stranding the old device record offline forever. **It is still a device
identifier and must be declared as one** — the scoping changes the privacy impact, not the
disclosure obligation.

## What the current app version does NOT transmit

Worth stating because the privacy policy's §3 table describes the household service's full
design, part of which the app does not implement yet. As of this app version, the TV sends **no**
usage history, **no** screen-time totals, **no** installed-app list, and **no** online/offline
status. If a later release starts sending any of them, this form and privacy-policy §3 both need
updating before that release ships.

## The requirement this form and the policy both fail to satisfy

**Prominent disclosure and consent.** Play requires an *in-app* disclosure wherever collection
could exceed a user's expectations, and names background collection explicitly — which is what
`MonitorService` does. It must appear during normal use, describe the data **and how it is used
and shared**, and be followed by an affirmative tap. Google states it cannot be satisfied by the
privacy policy or the terms of service.

The current onboarding explains three permissions (`permissions_overlay_rationale_v2`,
`permissions_usage_rationale_v2`, `permissions_notifications_rationale`) and says
"Connect Telegram — you'll manage limits from there". **None of it tells the parent that anything
leaves the television.** Neither this form nor the privacy policy closes that; a screen does.

## Security practices section

- **Is all user data encrypted in transit?** Yes — MQTT connection to the household server uses
  TLS; Firebase SDK traffic is HTTPS by default.
- **Do you provide a way for users to request data deletion?** Yes — unpairing a device from
  the Telegram bot clears its stored credentials; a parent can also contact the developer
  directly for full account/household deletion (see privacy policy §8).
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
