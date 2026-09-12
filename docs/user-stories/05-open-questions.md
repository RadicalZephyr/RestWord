# Open Questions

**Status: draft.**

Gaps identified while writing up the flows. Some of these are things we agreed to
in principle and never actually designed; others are edges that simply never came
up. Roughly ordered by how much they'd hurt if left unresolved.

---

## 1. Multi-select and bulk move — agreed in principle, never designed

We agreed this was common enough to design into the flow now rather than bolt on
later: long-press to enter a selection mode, tick several apps, move them together
into a new or existing group. Then we moved on to single-item dragging and never
came back to it.

What's undefined: how selection mode is entered and exited, how it coexists with
the drag gesture that *also* starts with a long press, where a multi-selection gets
dropped, and whether the group-creation-by-gap gesture works for a selection of
several apps or only one.

**This is the largest gap.** The long-press collision with drag-to-reorder is a
real design problem, not a detail.

## 2. The widget's own edit affordance

We established that the widget deep-links into the app scoped to its page, but
never designed *how you trigger that from the widget*. Every tap on a widget row
launches an app, which is the whole point, so there's no obvious spare gesture.

Candidates: a small persistent edit glyph (costs visual quiet, which is expensive
here); long-press on the widget (collides with Android's own widget long-press for
move/remove); or accepting that editing always goes through Android's
reconfiguration path or the in-app page list. The third is cheapest and may be
right, but it weakens the "widget is the entry point for everything" claim.

## 3. Naming or renaming a group

Headings are optional and groups can be created without one. We never designed how
you *add* a heading to a headerless group. A headerless group renders as a thin
divider, and a divider is a poor tap target.

## 4. Empty group lifecycle

If I delete or drag out the last app in a group, does the group disappear? Probably
yes if it's headerless, since it'd render as a stray divider. Probably no if it has
a heading, since the user deliberately named it and may be about to refill it. Not
decided.

Related: can a page have zero groups, or is the anonymous first group guaranteed to
exist? The data model says a page is a list of groups; the simple case says a page
is always at least one group. Worth pinning down.

## 5. Deleting a page, and orphaned widgets

Never discussed. If a page is deleted while widgets point at it, those widgets need
to render *something*. Options: a placeholder prompting reconfiguration, refusing
deletion while referenced, or warning and proceeding.

Also: if I tap "Create new page" in widget config and then abandon the flow without
tapping Done, does the half-built page persist in the page list, or get cleaned up?

## 6. Shortcut disappearance

An entry points at a shortcut. The app updates and drops it, or is uninstalled
entirely. The entry needs a defined fallback — silently reverting to "Open app" is
probably right, but whether the user is told is undecided. Same question for an
entry whose app is uninstalled entirely: does the row vanish, or persist greyed out
so the page structure survives a reinstall?

## 7. "Collapse all" as a deliberate control

Noted in passing as possibly worth having, since the collapsed-groups rendering has
to exist anyway for the drag interaction. Cheap to add, and it gives a whole-page
overview without starting a drag. Never decided either way.

## 8. Is the top-bar "add group" control still needed?

An earlier design put an add-group control in the top bar next to the settings
gear. Once group creation fell out of the drag-into-gap gesture, it became largely
redundant. The remaining case is creating an empty named group before you have apps
to put in it, which may not be a real workflow. Leaning toward dropping it, but
worth a decision rather than drift.

## 9. Page-list empty states

Two of them, neither designed. A brand-new user who opens the app before placing
any widget sees an empty page list — and given page creation deliberately lives in
widget config, the right message is probably instructional ("drag a RestWord widget
onto your home screen to begin") rather than an action button, which would
contradict the single-path principle.

The other: a page with a name, no description, and no apps yet. Its page-list row
has nothing to put on the secondary line.

## 10. FiraGO in RemoteViews — still unverified

Carried over from the typography work and still outstanding. Whether a bundled
FiraGO renders correctly in RemoteViews widget layouts across OEM launchers has not
been tested on a device. This is empirically answerable and blocks nothing until
it's answered wrong, at which point it blocks the entire visual identity.

**Suggested experiment:** a minimal widget declaring a bundled font on a TextView,
installed against stock Android plus at least Samsung One UI and a Pixel launcher,
checking that the custom face actually applies rather than silently falling back to
the system font.
