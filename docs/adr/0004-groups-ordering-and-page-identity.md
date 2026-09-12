# ADR 0004: Groups, ordering and page identity

**Status:** Draft. Group titles render on the home screen — the dusk palette document draws
them as uppercase Medium headings above each block, which settles what was an open question
here. Per-widget settings moved to ADR 0005.

## Context

A page today is a flat list of apps whose order is not expressible at all. `SelectAppActivity`
saves the alphabetically sorted picker list filtered by the checkboxes, so order is a side
effect of sorting by system label. Custom names break even that: once "Files by Google" displays
as "Files", it still sorts under F-for-Files-by-Google, and the order on screen looks arbitrary.
So explicit ordering is forced by ADR 0003 whether or not it had been asked for.

The request on top of that is named groups *inside* a page, with the groups reorderable and the
apps reorderable within them. That makes the model three levels: page, group, app.

Pages are currently referenced by position. A widget stores a page index, and deleting a page
shifts every later page down, so every widget after the deleted one silently starts showing
different apps and a widget pointing at the last page renders empty. Reordering groups or pages
would do the same thing, more often. Stable identifiers are a prerequisite, not a nicety.

### The widget is the real constraint

This is a text-only widget. Its entire aesthetic is one kind of text — a bare column of app
names — and a group header is a second kind. Getting that wrong costs more than the feature is
worth.

There is also a hard technical limit. The current code renders a page as
`data[num].chunked(10).map { Column { ... } }`, and the commit that introduced it is called
"Fix widget col limit". Glance builds `RemoteViews`, and its containers accept a bounded number
of children — ten. Chunking into nested columns is a workaround for that ceiling, not a layout
choice. Any grouped layout has to keep working within it, and a group header makes each block
one child larger.

I am inferring the exact limit from that code and commit message rather than from a
measurement. It needs confirming on a device before the rendering code is written.

## Decision

### Model

One document, with names and labels keyed by package as decided in ADR 0003, and ordering
carried by array order everywhere:

```json
{
  "version": 1,
  "names":  { "com.google.android.apps.nbu.files": "Files" },
  "labels": { "com.google.android.apps.nbu.files": "Files by Google" },
  "pages": [
    {
      "id": "p-8f3a2c",
      "title": "Home",
      "description": "",
      "groups": [
        { "id": "g-1c20b7", "title": "Work", "apps": ["com.slack", "com.mail"] },
        { "id": "g-44e901", "title": "",     "apps": ["com.android.chrome"] }
      ]
    }
  ]
}
```

A page always has groups; there is no second shape for an ungrouped page. A group with an empty
title renders no header, so the plain case — one untitled group holding everything — looks and
behaves exactly like today's flat list, and there is one code path rather than two.

Groups do not nest. Three levels is the whole model.

### Identity

Pages and groups carry generated stable ids. Widgets store a page id, never an index. Deleting
or reordering anything leaves every other widget pointing where it did before.

A widget whose page has been deleted says so, rather than rendering an empty tile. Silent
emptiness is how the current bug presents and it is indistinguishable from a widget that is
simply broken.

### Widget rendering

Flatten a page to a sequence of rows — a header row wherever a group has a title, then its app
rows — and chunk that flat sequence into nested columns exactly as the current code does. One
chunking strategy, unaffected by how the groups happen to be sized.

Groups are always separated by vertical spacing. A title is drawn only when set. Someone who
wants the grouping but not the labels leaves the title empty and gets it, without a setting to
explain.

### Configuration and editing

The widget configuration screen picks a page by title from a list, not by typing a number. That
retires "App Display Number" and the entire class of bug around it. It does the minimum at drop
time — choose a page, or create one — and everything richer lives in the app, where there is
room for it and where it is reachable again afterwards. See ADR 0005.

Tapping a widget's page title opens that widget's settings in the app. This is the main route
back to a placed widget, and it works on every platform version. The launcher's own reconfigure
affordance is a courtesy path on top of it, not the mechanism.

Reordering uses drag handles. This reverses an earlier draft of this ADR, which chose move-up
and move-down controls on cost grounds: Compose Foundation has no reorderable lazy list, so drag
means a third-party dependency or hand-rolled long-press gesture handling. The design settles it
the other way, and correctly — a pair of arrow buttons on every row is exactly the visual noise
an app arguing for calm cannot afford, and the editing screen is already carrying both naming and
ordering. The cost is real and we are choosing to pay it.

Pages carry an optional description, shown on the page list in place of the app-name preview
when one is set.

## Alternatives considered

**Groups as a renaming of pages, one level instead of two.** Smaller, and it would have
delivered reordering and titles without touching the widget's visual language. Rejected because
it is not what was asked for: the point is blocks *within* one widget, not more widgets.

**Optional groups: a page holds either apps or groups.** Avoids the empty-title convention.
Rejected because two shapes means two rendering paths, two editing paths and two migrations
later, to save one field.

**No headers on the home screen — grouping expressed only as spacing.** Very much in the spirit
of a text-only widget, and it keeps the surface to one kind of text. Not chosen outright
because it forecloses something that was explicitly asked for, but the empty-title behaviour
above means it is available per group at no cost.

**Move-up and move-down controls instead of drag handles.** Free accessibility, no dependency,
no gesture code. Rejected on appearance: two controls per row, permanently visible, on the one
screen where restraint matters most.

**Keep positional page references and renumber widget preferences on delete or reorder.** No
schema change. Rejected as the same bug with more bookkeeping — every operation on pages would
have to remember to fix up every widget, and forgetting once is silent.

## Consequences

The page-index bug disappears rather than being fixed, along with the numeric configuration
field that exposed it.

Group headers give the widget a second kind of text, which is the thing most likely to look
wrong on a real home screen. Their styling should be decided against a screenshot, not in this
document, and the empty-title escape hatch exists precisely because that decision might go
against headers entirely.

The ten-child ceiling now applies to headers as well as apps, so a page of many small groups
hits it sooner than a page of few large ones. Flattening before chunking keeps this correct
without special cases, but it means the limit is on total rows, which is worth stating in the
editor if anyone ever hits it.

Drag handles mean either a dependency or hand-rolled long-press gesture handling, and they need
an accessible alternative — a reorder action exposed to TalkBack — which arrow buttons would have
given for free. That is the price of the quieter screen.

## Open questions

**What happens to the page-number label?** The widget currently renders "App List N" with a
toggle to hide it. With titled pages that becomes "Home", which is more useful, but it is a
second piece of non-app text competing with group headers for the same attention.

**Ten children, or some other number?** Confirm against a device before the rendering code
depends on it.
