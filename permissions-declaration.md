# Play Console — Permissions declaration

Play Console → App content → Permissions declaration form. Two permissions in this app need a
declared justification: `PACKAGE_USAGE_STATS` and `SYSTEM_ALERT_WINDOW`.

## PACKAGE_USAGE_STATS

Google's declaration form asks you to pick a category, then gives a free-text box.

**Category:** Parental control

**Justification (paste as-is or adapt):**

> p-control is a parental-control app for Android TV. It uses PACKAGE_USAGE_STATS to detect
> which app is currently running in the foreground so it can enforce a parent-configured block
> list and daily screen-time limit in real time. This is the app's core function — without it,
> the app cannot determine what to block. Usage data is processed on-device to decide
> block/allow; only aggregated results (which rule fired, when) are sent to the household's own
> server for the parent to view — raw per-second usage never leaves the device.

**Demo video / screenshots:** Google may ask for a short screen recording showing the
permission's actual use (the block screen appearing when a restricted app is opened). Not
scripted here — record ~15-20s: open a blocked app on the TV → block screen appears →
`PACKAGE_USAGE_STATS`'s role is visually obvious.

## SYSTEM_ALERT_WINDOW

**Category:** Parental control / accessibility-adjacent overlay

**Justification:**

> p-control needs to display a full-screen "blocked" overlay on top of a restricted app the
> instant it's detected, without waiting for the user to background it first — that immediacy
> is the point of a real-time parental block. SYSTEM_ALERT_WINDOW is the only Android mechanism
> that draws over another app's foreground activity. The overlay only appears when a
> parent-configured rule is triggered; it is never used for ads, prompts, or any purpose
> unrelated to the block list.

## Notes

- Both declarations are per-app, resubmitted whenever Play Console flags a policy update — check
  back if a future Play Console review email mentions either permission.
- If Google's review rejects either justification, the actual behavior (real-time on-device
  block enforcement) is unlikely to be deniable — a rejection more often means the justification
  text itself needs to be more specific/concrete, not that the permission use needs to change.
