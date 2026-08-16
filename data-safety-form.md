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
| **App activity → App interactions** | Yes. Which app is in the foreground, read on-device to enforce blocking, **and sent to the household server**: one live signal per foreground change, and a batch about once a minute of closed sessions (app identifier, start time, end time, child profile where one is known). Also, when a child uses "ask a parent", the identifier of that one app. | No (same reading as the profile-name row — the household server is the developer's own, not another company) | App functionality | Required — core purpose |
| **App activity → Installed apps** | Yes — the launchable apps on the device, each with name, version, first-install and last-update times, install source, Android's own category, and whether it is a system app — plus the app's own launcher icon, sent only for the apps the server asks for. Sent on connect and on every install/update/removal. See the judgment call below. | No (same reading) | App functionality — it is the picker a parent chooses rules from | Required; there is no way to choose an app to block without a list of apps |
| **App activity → In-app search history** | No | — | — | — |
| **App info and performance → Crash logs** | Yes (via Firebase Crashlytics) | Yes — with Google (Firebase, as the analytics/crash provider) | Analytics | Not user-facing/optional — standard crash reporting |
| **App info and performance → Diagnostics** | Yes, from two sources: Firebase Crashlytics attaches device/OS info to crash reports, and the device's own status message carries its model, Android version, language and time zone to the household server. See the judgment call below. | Yes — with Google (Firebase) for the crash half. The status half goes only to the household server | Analytics (Firebase) **and** App functionality (naming a device in the panel, and keeping schedules on the household's clock) | Not optional |
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
   the household server only, and only after a parent confirms the pairing in the web panel; an
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

## Four taxonomy calls this release forces — decide them, do not inherit them

Disclosure revision 2 (installed-app detail, icons, the device description, usage as time) puts
four questions to Play's data-type picker that the picker does not answer cleanly. Each is
written below as a recommendation with its reasoning and its runner-up, because a wrong row here
is a policy violation and not a wording preference. **None of them is settled by this document.**

**1. Where the installed-app list goes.** Play's *App activity* group is documented as including an
**Installed apps** type — "information about the apps installed on a user's device" — and that is
the exact fit; the table above answers on that basis. If the live Console picker does not offer it
under whatever it is currently named, the runner-up inside the same group is *App activity → Other
actions* with a plain description; *App info and performance* is the wrong group, because that
group is about how **this** app behaves, not about what else is on the device. **Read the Console's
own picker before submitting** — this document cannot see it, and the type list changes.

**2. Where the device description goes** — model, Android version, language, time zone.
Recommendation: *App info and performance → Diagnostics*, which is where device and OS
characteristics already sit for the Crashlytics half, with **App functionality** added as a
purpose alongside Analytics because the household panel uses it to name a television and to keep a
schedule on the household's clock. Rejected: *Device or other IDs* — none of these four identifies
anything, and folding them into that row would blur an answer that currently has one careful,
`ANDROID_ID`-shaped explanation. Rejected also: leaving it undeclared because it feels like
plumbing. It leaves the device, so it is declared.

**3. Whether an icon is a data type at all.** Recommendation: **no separate row.** An app icon is a
resource shipped inside an already-declared installed app, produced by its developer, containing
nothing about the user; it is disclosed in the *Installed apps* description above, on the policy
page and in the in-app screen. Rejected: *Photos and videos*, which is about a user's own media and
would misdescribe this badly. The conservative alternative, if preferred, is to keep the row absent
but say "including app icons" in the Installed apps free-text description — which the table above
already does.

**4. Whether App activity is "linked to the user" — and, underneath it, whether the monitored child
is a *user* at all.** This is the sharpest of the four and the one with the most consequence, since
Play asks per data type whether the collection is linked to a user's identity, and answering wrong
is a misdeclaration rather than a wording preference.

A `UsageEvent` carries `profile_id` whenever the device knows which child was watching — the
active-allowance path fills it from the profile whose gate opened the app. A profile has a name the
parent typed. So the household server can say "this child watched this app from 19:04 to 19:41",
and on the plain reading **App activity is linked to a user** and must be declared as such. The
same then applies to the installed-app list, which is linked to a device that is linked to an
account.

The prior question is whether the child is a *user* for Play's purposes. The child creates no
account, supplies nothing directly, and everything about them is entered by the parent, who is the
one contracting; on that reading the only user is the parent, the profile is the parent's own
label for a person in their household, and App activity is linked to **the parent's** account
rather than to the child's identity — which still lands on "linked", just by a different route.
The reading that would produce "not linked" requires treating the profile as an anonymous bucket,
and it is not one: it has a name, and a parent can read a timeline off it.

**Recommendation: declare App activity as linked to the user, and do not spend the argument about
who the user is on this row.** Both defensible readings arrive at "linked"; only the strained one
arrives elsewhere, and Play's reviewers see the account, the profile name and the timeline in the
same panel. The unresolved half — a child's rights being reachable only through the parent's
account, and no separate handling for teenagers — is stated on the policy page's §10 rather than
argued away here, and it is legal work, not a form answer.

## What the current app version does NOT transmit

The list is short now, and that is the point of keeping it: the app version that carries
`CURRENT_DATA_DISCLOSURE_VERSION = 2` sends the installed-app list, per-app usage and app icons,
all of which earlier versions did not. What it still does not send is everything §3 of the policy
page calls never-collected — window titles, keystrokes, screenshots, clipboard contents, visited
URLs, file names, location, contacts, microphone and camera — and, specifically, no free-text
field of any kind travels with a usage record: an app identifier and two timestamps is the whole
of it.

**What the server does with each message is not what the form asks about.** As this release
ships, `p-control-server` ingests the installed-app snapshot (`InstalledAppsSubscriber` →
`InstalledAppsService`) and the status message; there is no usage ingest and no icon handler yet,
so those two are published to the household broker and dropped for want of a subscriber. The rows
above still answer **Yes** for both, because Play's question is what the app collects and
transmits, not what survives at the other end — and a subscriber appearing later must not be the
event that changes this form.

**It does send an online/offline status, and that belongs on the form rather than in this list.**
`DeviceStatusReporter` publishes ONLINE on connect, retained, and registers the OFFLINE payload as
the connection's Last Will so the broker sends it when the device stops answering. The same
message carries the app's version, what this build is able to do, whether the permissions
enforcement needs are still granted, and the disclosure revision the household accepted. It says
nothing about a person — it is what lets a parent's panel show a television as connected instead
of leaving them to infer it from silence. Understating collection is the worse direction of the
two errors this section exists to prevent.

## Prominent disclosure and consent — satisfied by a screen, not by this form

**Play requires an *in-app* disclosure** wherever collection could exceed a user's expectations,
and names background collection explicitly — which is what `MonitorService` does. It must appear
during normal use, describe the data **and how it is used and shared**, and be followed by an
affirmative tap. Google states it cannot be satisfied by the privacy policy or the terms of
service, so nothing in this document or on the published page closes it.

**It is built, and it is at revision 2.** `p-control-android` shows `DataDisclosureScreen` as the
first screen of first run (`disclosure_leaves_device`, `disclosure_sharing`, `disclosure_accept`),
stores the accepted revision, and `MainActivity.startMonitorServiceIfPermitted` gates the
foreground service on it — so enforcement cannot begin before the parent has been told. Revision 2
is the one that names each app's version, dates, origin and icon, the device's own description, and
usage as time that leaves the device; a household that accepted revision 1 is asked again before
any of it is sent. The accepted revision is what `DeviceStatus.disclosure_version` carries, and the
server refuses an installed-app snapshot from a device still reporting 1. Its rules live in that
repo's
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

**What this row does NOT cover.** The account is the parent's own data on the server; it is
unrelated to what a television sends, and adding an account changed nothing about the device's
collection. Installed apps and per-app usage are covered by the App activity rows above, which the
current binary does send — a form that overstates collection is wrong in exactly the way one that
understates it is, and the check that keeps both honest is reading the rows against the code
rather than against the design.

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

## Two further judgment calls flagged for your decision, not assumed

These are older than the three taxonomy calls above and are unrelated to them.

1. **Is the household's own self-hosted server a "third party" for Play's purposes?** Google's
   definition centers on *other companies*; a service the developer personally operates for
   their own users arguably isn't one — I've marked profile-name sharing "No" on that reading.
   If you'd rather declare it transparently as shared regardless, flip that row to "Yes,
   shared with: service provider" — it's a defensible, more conservative choice.
2. **Firebase Analytics/Crashlytics rows above assume default SDK behavior** (no consent-gated
   collection, no ad personalization use). If either SDK's config in this app has been
   customized beyond what `FirebaseAnalyticsRepository`/`FirebaseCrashReportingRepository`
   show, re-verify against Firebase's own current data-safety mapping guide before submitting.
