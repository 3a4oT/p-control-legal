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
| **App activity → App interactions** | Yes. Which app is in the foreground, read on-device to enforce blocking, **and sent to the household server** on two channels: a live signal saying what is on screen now and whose it is, and a batch of hourly totals (app identifier, which local hour, how many of its seconds the app held the screen, when a session began inside it, and whose time it is — a child profile, an adult who answered, or nobody). Also, when a child uses "ask a parent", the identifier of that one app and which child is asking where the device knows. | No (same reading as the profile-name row — the household server is the developer's own, not another company) | App functionality | Required — core purpose |
| **App activity → Installed apps** | Yes — the apps on the device that can be opened, plus the home screen itself, each with name, version, first-install and last-update times, install source, Android's own category, and whether it is a system app — plus the app's own launcher icon, sent only for the apps the server asks for. Sent on connect and on every install/update/removal. See the judgment call below. | No (same reading) | App functionality — it is the picker a parent chooses rules from | Required; there is no way to choose an app to block without a list of apps |
| **App activity → In-app search history** | No | — | — | — |
| **App info and performance → Crash logs** | Yes (via Firebase Crashlytics) | Yes — with Google (Firebase, as the analytics/crash provider) | Analytics | Not user-facing/optional — standard crash reporting |
| **App info and performance → Diagnostics** | Yes, from two sources: Firebase Crashlytics attaches device/OS info to crash reports, and the device's own status message carries its model, Android version, language and time zone to the household server. See the judgment call below. | Yes — with Google (Firebase) for the crash half. The status half goes only to the household server | Analytics (Firebase) **and** App functionality (naming a device in the panel, and keeping schedules on the household's clock) | Not optional |
| **Device or other IDs → Device or other IDs** | Yes — **two separate sources**, see the note below | Yes — with Google (Firebase) for the app-instance ID. The pairing identifiers go only to the household server, and reach it through **Cloudflare**, which terminates TLS for the app's first request (same judgment call as the profile-name row for the server itself; Cloudflare is a processor either way) | Analytics (Firebase) **and** App functionality (pairing) | Not optional — the app cannot pair without sending them, and they are sent before any pairing is confirmed |
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
   per-installation UUID, the device model string, `Settings.Secure.ANDROID_ID` and the device
   kind. They go to the household server only, and they are sent **before** a parent confirms
   anything: `PairingBootstrapRequest` carries all four in the app's first contact, the
   `POST /api/v1/pairing/hello` that asks for a pairing code. Do not answer this row as though an
   unpaired app transmits nothing — it transmits these four to the household server, and only these
   four **there**; Firebase Analytics is its own path and logs `pairing_code_displayed` from the
   pairing screen while the app is still unpaired, with the app-instance ID, unless diagnostics are
   switched off. And it repeats
   the request: `rotateSessionsUntilPaired` re-mints a session at every 10-minute code expiry and
   `askForASession` retries with backoff, against a server budget of 20 hellos an hour per machine
   and 60 per address.

   **That first contact is HTTPS to `control.rovenskyi.com`, which is behind Cloudflare**, so
   Cloudflare terminates TLS for it and handles those four identifiers and the connecting IP
   address. It is not the only HTTPS the app makes to that address: a **paired** device fetches
   profile pictures from `<origin>/media/avatars/<sha>.webp`, so Cloudflare sees a paired device's
   address and its picture requests too. Everything else a device sends travels on its own MQTT
   link to the broker, which is DNS-only and does not pass through Cloudflare. "Everything else"
   means everything else it sends **to the household service** — Firebase traffic is its own path
   to Google and is answered in the rows above.

   **How to answer the "shared" column for this.** Cloudflare processes it on our instructions and
   for no purpose of its own, so under Play's definition it is **not** a third-party share — the
   same reading as judgment call 1 below about the household server itself. Keep the row's "shared"
   answer as it is; what changed is a transfer and a processor, which belongs in the security
   answers and on the policy page (§7), not in a new recipient here. Nothing new is collected: the
   same four fields previously travelled over MQTT under a shared pairing credential compiled into
   the package, which no build has any more.

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

## Four taxonomy calls this release forces — decided 2026-09-12

Disclosure revision 2 (installed-app detail, icons, the device description, usage as time) puts
four questions to Play's data-type picker that the picker does not answer cleanly. Each is
written below with its reasoning and its runner-up, because a wrong row here is a policy
violation and not a wording preference. **All four are settled: the recommendation stands in
every one of them.** The runners-up are kept because the Console's picker can force a fallback —
call 1 is the only one where that can happen, and it says what to do then.

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
| **Personal info → Phone number** | Yes (the parent's, when they link a Telegram chat) | No | App functionality (account management) | **Optional**, and optional in the sense Play means: a parent who never links a chat never provides one, and every feature except Telegram delivery works without it. It arrives from Telegram's own `request_contact` button — the parent taps to share their own number, so Telegram has verified it against the device holding it — and `p-control-server` stores it on the account. Nothing else reads it, and it is shown to no one but its owner. The Android app never touches it: linking happens between the parent's Telegram client and the server |

### Signing in with Google — LIVE since 7 September 2026

A parent may sign in at `control.rovenskyi.com` with a Google account instead of a password, when
the operator has switched that door on. What the server is given is a signed ID token carrying
Google's account identifier (`sub`) and the address Google says it has verified; it is stored as a
row beside the account, and the identifier rather than the address is what signs the person in
afterwards.

| Data type | Collected | Shared | Purpose | Notes |
|---|---|---|---|---|
| **Personal info → Email address** | Already declared above; Google supplies a verified one for a parent who signs in this way | No | App functionality (authentication) | The same row as the address a parent types — the source differs, the collection does not |
| **Personal info → User IDs** | Yes — Google's account identifier for that parent, when they use this way in | No | App functionality (authentication) | Absent for a parent who signs in with a password, and removed with the linked account from the account page. It is an identifier for **the parent's own account**, never for a child or a device |

**What is shared with Google, and by whom.** Not by us: the browser loads Google's own script from
`accounts.google.com` to draw the button, so Google learns that a browser opened the panel's
sign-in page while the door is open — before anybody presses anything. The panel loads nothing from
Google when the door is shut. This is a browser-side disclosure of the kind Play's form does not
have a row for, and it is stated on the policy page's §7 instead, which is where a person can act
on it.

**The in-app disclosure revision does not move for this.** `CURRENT_DATA_DISCLOSURE_VERSION`
tracks what leaves the **device**, and nothing new leaves it: this happens in a browser, and the
television's agent is not in the path. The same reading as the phone number below.

**Why a phone number is declared at all when the app never sees it.** The same reason the email
row above is: Play asks what the *app's* product collects, and a parent using this app has one
account across the television, the panel and the bot. Declaring only what crosses the app's own
process would understate collection, which is the worse of the two errors this document exists to
prevent. It is the same judgment already made for the account itself on 14 August 2026.

**The in-app disclosure revision does not move for this.** `CURRENT_DATA_DISCLOSURE_VERSION` tracks
what leaves the **device**, and nothing new leaves it: the number travels from the parent's Telegram
client to the server, and the television's agent is not in that path. Re-prompting every household
to accept a revision that describes no change to their device would train them to tap through the
prompt that does matter.

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

## The panel's session records, and why this form does not grow a row for them

**Added 2026-08-19.** Signing in to the web panel now records, per session: the browser and
operating system matched to a short list of known names, and the IP address the session was last
seen from. A person can list their sessions and end one.

**No row in the table above changes, and `Location` stays "No".** Play's Data safety form declares
what **this app** collects and shares. The app does not sign in to the panel and does not see a
browser, so a session's stored address is a browser's and not the app's.

**Since 1.5 the app does supply an address of its own, and the conclusion survives while the old
reasoning does not.** `POST /api/v1/pairing/hello` arrives from the device, and the server counts it
against that connecting address — so "the app never handles an IP address of its own" is no longer
the argument. The argument is that an IP address is not `Location` on this form: Play's `Location`
row is approximate or precise location derived from the device (GPS, network location, and the
like), nothing in this product derives one, and no address here is ever resolved to a place. It is
also not *stored* as data about a user — see the counting subsection under the security answers
below. `Location: No` therefore stands on its own facts rather than on the app having no address. The distinction is worth writing down because "we now store IP addresses"
sounds like it must belong here, and answering `Location: Yes` because of it would be **wrong** in
the direction that matters: it would declare a data type the app does not collect, against a
binary Google can and does check.

Two things follow, and neither is optional:

- **The privacy policy carries it**, because that page covers the service and not only the app —
  §5 names what a session record holds, §3 says plainly that an address is never turned into a
  place, §6 gives its ground (legitimate interests, account security) and §9 says it dies with the
  session it belongs to. That is where a reader looks, and it is the artifact Play's User Data
  policy requires to be complete.
- **If a future app build ever authenticates to the panel itself** — a phone client, a companion
  app — this call reverses and the row is real. Whoever writes that build reads this paragraph
  first.

## The developer's own support access, and why this form does not grow a data-type row

**Added 2026-09-04.** A super admin — today, the developer alone — can now open a household's own
data directly, from a console separate from the household's own panel, to answer a support
question or diagnose a fault: the same rules, devices, profiles and history that household's own
panel already shows it, nothing more.

**No row in the table above changes.** Play's Data safety form declares what the app collects,
transmits and shares with parties outside the service; a support session is the developer's own
service looking at data the developer's own service already holds, not a new data type, a new
recipient or a new company in the path — the same reasoning the panel's session records section
above gives for staying out of this table.

**What changed belongs on the privacy policy page, and now does.** `p-control-server`'s
`2026-08-14-super-admin-console-design.md` "Support access" section owns the mechanism: a
**support session** is a distinct, time-limited access token — up to one hour — derived from the
super admin's own sign-in rather than a silent widening of an existing one, and every household it
enters and every write it makes is recorded in an append-only log
(`super_admin_audit`) that only a super admin can read. The household is not notified of an
individual visit, which is a deliberate product decision the policy states rather than glosses
over. §6, §7, §9 and §11 of the published page carry the household-facing half of it: the legal
basis, who this is, how long the log itself is kept, and how a household asks about it.

**No new data type, no new recipient, no new permission.**

## The self-expiring records are swept too, and one of them held an address

**Changed 2026-09-03.** A retention answer rather than a new data type: nothing new is collected,
and something that was kept **forever** now has a bound.

Four tables carry their own `expires_at`, checked when the row is read and enforced by nothing
that removed it: `pending_pairings`, `telegram_link_requests`, `password_resets` and
`refresh_tokens`. `RetentionSweep` now deletes each one a week after its own expiry.

**`refresh_tokens` is the row that matters for this form.** It carries `client_ip`, which this
document already lists as collected personal data, and until now an expired session nobody had
signed out of kept that address indefinitely. The page said a session's address goes "when the
session goes"; for a session that simply stopped being used, nothing went. The page now states the
expiry path explicitly rather than leaving the reader to read the sign-out path as the only one.

The week is not for the rows — a spent pairing token is useless the moment it expires. It is so
that "why did my session end on Tuesday" is answerable on Wednesday. `RetentionSweep.expiredGraceDays`
is the constant.

**`invites` is deliberately left standing** and is not an omission: an expired invite is shown to
the household that minted it, as expired, and a consumed one records how a member came to be one.
Removing either changes what a household is told, which is a product decision rather than
retention.

**No new data type, no new recipient, no new permission.**

## Retention is now enforced by a job, not only stated on the page

**Changed 2026-08-28**, and it moves two rows from "kept while the household is in use" to a
bounded window, which is a Data safety **retention** answer rather than a new data type.

`p-control-server`'s `RetentionSweep` now deletes `approval_requests`, `device_events` and the
per-parent copies in `notifications` at the household's plan window (`plans.usage_history_days`,
90 days on the seeded free plan), on the same schedule that already swept usage and presence. A
plan naming no window keeps them, which is what an unlimited-history tier means.

Three details this form has to carry, because the page states them only in plain words:

- **A copy never outlives its record, and a record that survives keeps its copy.** The inbox rows
  are per-person copies of an approval or a device event; they are deleted in the same statement
  as the row they copy, so no reader can observe one without the other.
- **A read-and-expired inbox copy goes at 90 days regardless of the plan**, and an unread one is
  never swept early — an unread tamper alert is the point of the inbox. It still leaves at the
  plan window with everything else.
- **No row is kept past the window for any reason.** A request that granted bonus time used to be
  kept indefinitely so the grant could point at it; the foreign key is now `ON DELETE SET NULL`,
  the grant keeps its own record — profile, day, minutes, who granted it — and retention is
  uniform. Keeping personal data indefinitely to preserve a pointer to that same data was
  circular.

**No new data type, no new recipient, no new permission.** `control/v1` is unchanged: nothing new
leaves the device, and `device_events` rows are derived on the server from what a device already
reports.

**Judgment call left to a human:** the Data safety form asks whether data "can be deleted by the
user" and whether it is deleted automatically. Both are now true for these categories, and the
answer should say so — but which of the form's fixed phrasings fits a *plan-dependent* window is
a choice about the form, not about the code.

## Security practices section

- **Is all user data encrypted in transit?** Yes, on every path. Two are HTTPS to
  `control.rovenskyi.com`, terminated by Cloudflare and re-encrypted to the origin: the app's first
  contact (`POST /api/v1/pairing/hello`, repeated while a pairing screen waits) and a paired
  device's profile-picture fetches (`/media/avatars/<sha>.webp`) — a paired device's path, so
  "after pairing" does not mean "off Cloudflare". What a paired device *reports* to the household
  service — status, activity, usage, the app list, approvals — is MQTT over TLS straight to the
  broker, which is DNS-only and not proxied. Firebase SDK traffic is its own path to Google and is
  HTTPS by default.
- **What the server does with the device's connecting address, since 1.5.** Named here because
  the privacy policy discloses it (§4, §6, §9) and a reviewer re-checking rows against the code
  would otherwise find the page disclosing something this document denies.
  `PairingMintRateLimiter.check` writes two counters into the shared cache for every hello — one
  keyed on the connecting address (`pairing:mint:address:<address>`) and one on `machine_anchor`
  (`pairing:mint:anchor:<anchor>`), or on the address again when a hello names no anchor — each with
  a fixed 3600-second window, 60 hellos an hour per address and 20 per anchor. Nothing else is
  written under those keys, they are joined to no household or account, no location is inferred from
  an address, and both expire on their own. **This adds no row to the table above**: it is a
  server-side rate limit on an unauthenticated endpoint rather than data the app collects about a
  user, and `Location` stays "No" for the reasons in the section on IP addresses.
- **Do you provide a way for users to request data deletion?** Yes, at four levels, and the last
  of them is self-service. Unpairing a device from the web panel clears the credentials it holds.
  Signing out everywhere ends every browser session at once, and a single session can be ended on
  its own — which also deletes the browser, system and address recorded with it, since those are
  columns of the session row and nothing keeps a history of them.

  **A parent closes the whole account themselves**, from Account → Close this account. It takes
  the email address, the phone, the password hash, every session and the Telegram link with it,
  and any home the person is the only member of closes with them — every screen in it disconnected
  and every child's history erased. The panel names those homes before the act and asks for the
  password again, because a session left open on a shared machine may not end somebody's account.

  Two accounts cannot do it and the panel says which applies. One that is the last person able to
  administer a home other people are still in is refused, because erasing it would leave a home
  nobody can run; handing that over first clears it. And an account that has acted inside other
  people's homes as a server administrator is refused, because that record is not the account
  holder's to erase — the accountability carve-out GDPR Article 17(3) makes. Both are answered by a
  person at the address in privacy policy §11.
- **Data collection is required or can users opt out?** Two different answers, and Play's form
  takes them per data type rather than once.
  - **Core enforcement data (app activity) is required** — it is the product's function, and a
    household that switches it off has switched off the product.
  - **Analytics and crash reporting are optional as of 2026-08-30.** *Settings → Privacy → Send
    diagnostics* on the television switches off Firebase Analytics and Crashlytics together, at
    the SDK level (`setAnalyticsCollectionEnabled` / `isCrashlyticsCollectionEnabled`), so the
    automatic events — sessions, first-open, screen views — stop as well, not just the events this
    app logs itself. Mark every Firebase-sourced row **Optional**. Debug builds never report at
    all, whatever the setting says, which is a build-time rule and not a user-facing one.

  Do not mark the pairing identifiers optional: they go to the household's own server and the app
  cannot pair without them.

## Two further judgment calls — the first is decided, the second is a re-check before every submission

These are older than the four taxonomy calls above and are unrelated to them.

1. **Is the household's own self-hosted server a "third party" for Play's purposes? No — decided
   2026-09-12.** Google's definition centers on *other companies*, and a service the developer
   personally operates for their own users is not one. Every row that sends data only to
   `control.rovenskyi.com` answers "Shared: No", which is what the table above already says; the
   rows that name Google (Firebase) or Cloudflare keep their "Yes", because those are other
   companies whatever the household server is. The conservative reading — declaring the
   household server as a shared "service provider" — was considered and rejected: it would put
   "shared" on the public card for a destination the household itself owns.
2. **Firebase Analytics/Crashlytics are consent-gated as of 2026-08-30**, so the rows above are
   no longer "default SDK behavior". `AnalyticsConsentRepository` stores the household's answer and
   `ApplyAnalyticsConsentUseCase` holds both SDKs to it for the life of the process; the default is
   on, and a parent switching it off is obeyed without a restart. What has *not* changed is ad
   personalization: it was never used and is now unreachable — see the advertising-ID note below.
   Re-verify against Firebase's own current data-safety mapping guide before submitting anyway.
3. **The advertising ID is no longer collected, and the row should say so.** Until 2026-08-30 the
   Firebase Analytics library merged `com.google.android.gms.permission.AD_ID` and two Privacy
   Sandbox ad-services permissions into the built app, even though nothing in this codebase reads
   an advertising ID. All three are removed with `tools:node="remove"`. **Verify before every
   submission rather than trusting this note** — the source manifest never showed them, which is
   how they went unnoticed for three weeks:

   ```bash
   grep -o 'uses-permission[^>]*' app/build/intermediates/merged_manifests/release/*/AndroidManifest.xml | sort -u
   ```

   Confirmed on the release variant again on 2026-09-12, for the `0.4.0` build that is going to
   closed testing: the same eleven permissions, none of them advertising.
   `com.google.android.finsky.permission.BIND_GET_INSTALL_REFERRER_SERVICE` remains and is
   deliberate — install attribution is not the advertising ID and Play does not treat it as one.
