# Customizing an App Entry

**Status: draft.**

Two weights of editing, deliberately kept on separate gestures so they don't
compete: a lightweight everyday rename, and an occasional, deliberate change to how
the app launches.

---

## Story: I want to shorten or genericize an app's name

**As a** user who finds "Philips Hue" noisy
**I want** to call it "Lights"
**so that** the list reads as plain language instead of branding.

### Narrative

This is the most common customization and it gets the lightest gesture: tap
directly on the app's text in the group editor. Inline edit, type, done.

Once an entry has a custom label, the row renders the custom label in the normal
weight with the original app name underneath it, smaller and greyed out. That
serves an obvious purpose — knowing what "Lights" actually is six months later —
but it has a useful side effect too.

Rows with a custom label are taller than rows without one. We considered forcing
uniform row heights and decided against it. The varying height becomes information:
I can scan a group and immediately see which entries I've customized without
reading a word. Strict uniformity would have thrown that away for no real gain.

This is also the point of the feature in product terms. Renaming is how you strip
branding out of your own home screen, which is the thing RestWord exists to do.

### Acceptance criteria

- **Given** an app row, **when** I tap its text, **then** the label becomes
  editable in place.
- **Given** I set a custom label, **when** the row renders, **then** the custom
  label shows in the primary style and the original app name shows beneath it,
  smaller and de-emphasized.
- **Given** no custom label, **when** the row renders, **then** only the app's real
  name shows and the row is shorter.
- **Given** I clear the custom label, **then** the entry reverts to the app's real
  name and the secondary line disappears.
- **Given** a custom label, **when** the widget renders, **then** only the custom
  label is shown. The original name is an editor affordance, not widget content.

---

## Story: I want an app to launch straight into a task

**As a** user who opens my notes app only to write something
**I want** tapping it to open a new note directly
**so that** I skip the app's own landing screen entirely.

### Narrative

This one fits the product's whole argument better than almost anything else we
discussed. Hiding the icon removes the visual pull; launching straight into the
task removes the app's chance to show me a feed on the way.

Swipe the row **right** to edit. That opens a dedicated full page rather than a
sheet or a panel, because this is more involved than a rename and because the page
gives us somewhere to put future extensions without cramming.

The page is titled with the app's real name. Below that is an editable field for
the custom label, then the list of things this entry can launch. The label sits on
this screen too, on the reasoning that if I'm changing *what* an app launches, I
will very likely want to update what it's *called* at the same time. Forcing a
separate hop between those two edits would be silly.

The launch list always offers **Open app** as the zeroth, default option, followed
by whatever shortcuts the app itself publishes. Android caps apps at four shortcuts
total across static and dynamic, so this list is always small and bounded. That's a
real constraint working in our favour: the picker never needs scrolling, search, or
pagination.

Selecting a shortcut requires an explicit **Save**. Since I've already paid the
cost of a full page transition, one deliberate tap isn't meaningful friction, and
it guards against a stray tap silently rewiring what an app opens.

### Technical note

Shortcuts come from the `LauncherApps` system service via `getShortcuts()`, scoped
to a package, with query flags covering manifest (static), dynamic, and pinned
shortcuts. This requires RestWord to hold the relevant launcher permissions. The
four-shortcut cap is imposed by the OS, not by us.

### Acceptance criteria

- **Given** an app row, **when** I swipe it right, **then** the customization page
  opens for that entry.
- **Given** the customization page, **then** it shows the app's real name as the
  title and an editable custom-label field.
- **Given** the app publishes shortcuts, **then** they are listed below "Open app",
  fetched live rather than cached indefinitely.
- **Given** the app publishes no shortcuts, **then** only "Open app" is listed and
  the label field is still editable.
- **Given** I change a selection or the label, **when** I leave without tapping
  Save, **then** nothing is persisted.
- **Given** I tap Save, **then** both the label and the launch target are persisted
  together.
- **Given** a selected shortcut later disappears (app updated or uninstalled),
  **then** the entry falls back to "Open app" rather than failing silently. (Not
  fully designed — see 05.)

---

## Rejected alternatives

**One swipe direction revealing both actions in a panel.** Considered, and rejected
in favour of symmetrical directions: left for delete, right for edit. Each of the
two common per-item actions gets its own uncontested gesture instead of both
fighting for room in a single revealed strip.

**Using the swipe-right panel for renaming too.** Rejected once renaming moved to a
direct tap on the text. Renaming is the everyday action and deserves the cheaper
gesture; the full page is for the occasional deliberate one.

**A read-only display of the custom label on the shortcut page.** This was the
first shape and it was wrong. Changing the shortcut and changing the label are
correlated actions, so making the label read-only would force a pointless trip back
to the editor.

**Uniform row heights regardless of customization.** Rejected. The height
difference is free information about which entries have been customized.

**Arbitrary custom URIs and deep links — deferred, not rejected.** There's real
appetite for this, but it's a different kind of feature: it needs the user to know
or discover a URI, which is a much less friendly interaction than picking from a
list the app itself published. The dedicated page was chosen partly to leave room
for it later. Explicitly out of scope for this version.

**Intercepting or skipping an app's own splash screen.** Not available on stock
Android. Once you have a package name you get to launch it; what it does next is
its own business. Worth stating plainly so nobody tries to design around it.
