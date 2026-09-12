# Building a Page

**Status: draft.**

The group editor is where the actual work happens. It's reached by deep link from
widget config, or by tapping a page in the in-app page list. Both arrive at the
same screen.

The editor is a list editor. That framing drove most of the decisions here: a page
*is* a list, the widget renders a list, so editing should feel like editing a list
rather than filling in a form.

---

## Story: I have an empty page and want apps on it

**As a** user who just created a page
**I want** to pick the apps that belong on it
**so that** the widget has something to show.

### Narrative

An empty page is mostly whitespace and a prominent **+** button. There's nothing
else to look at and nothing else to learn.

Tapping + opens the app picker: every installed app, as a multi-select list, with a
search field at the top. Search is fuzzy filtering over an already-known list, not
a query against anything remote. Selected rows are highlighted with the accent
colour rather than carrying a checkbox — the row itself is the control.

Multi-select matters because the common case is batch-adding eight or ten apps at
once, not adding one and coming back. I tick through what I want and tap Done.
Everything I picked lands in the page's first group, in selection order.

The + button doesn't go away once the page has content. It stays as the way to add
more.

### Acceptance criteria

- **Given** an empty page, **when** the editor opens, **then** a prominent + is the
  primary visible affordance.
- **Given** the app picker is open, **when** I type, **then** the list filters
  fuzzily over installed app names.
- **Given** I select several apps, **when** I tap Done, **then** all of them are
  appended to the first group in the order I selected them.
- **Given** a page with content, **when** the editor renders, **then** + is still
  available.
- **Given** an app is already on the page, **when** the picker opens, **then** its
  row shows as selected.

---

## Story: I want to remove an app from a page

**As a** user tidying a page
**I want** to drop an app I don't use
**so that** the list stays short.

### Narrative

Swipe the row left. A delete action is revealed; swipe far enough and it just goes.
Standard, well-worn, requires no explanation.

This gesture was what resolved an earlier problem: a + button tells you that you
can *add* things, but says nothing about modifying what's already there. Swipe
supplies the missing half without adding another button.

### Acceptance criteria

- **Given** an app row, **when** I swipe it left, **then** a delete action is
  revealed.
- **Given** I swipe far enough left, **then** the entry is removed without needing
  a second tap.
- **Given** I delete the last app in a group, **then** the empty group is removed
  too, unless it has a heading. (See 05 — not fully settled.)

---

## Story: I want to reorder the groups on a page

**As a** user with several groups
**I want** to change the order they appear in
**so that** the most-used group sits at the top.

### Narrative

This is the nicest bit of the design and it works by reusing one gesture.

I touch and hold a group's drag affordance. Every group on the page immediately
collapses down to just its header row, so I'm looking at group-level structure with
none of the app rows in the way. The group I'm holding detaches and follows my
finger; the others compress around it in the usual reorderable-list way.

When I drop it, groups expand again — *except* that a group I'd deliberately
collapsed before I started stays collapsed. The interface simplifies itself exactly
when I need it to and restores itself when I don't, and I never had to learn a
separate collapse control.

The one deliberate exception: after reordering, groups do not force themselves back
open if I'd been using the collapsed state as an overview. Snapping everything back
open would punish the exact moment I was trying to see the whole page at once.

### Acceptance criteria

- **Given** a page with multiple groups, **when** I begin dragging a group header,
  **then** all groups collapse to headers only.
- **Given** I am dragging a group, **when** I move over another group's position,
  **then** the others shift to show where it will land.
- **Given** I drop the group, **when** the drag ends, **then** groups re-expand,
  except those the user had already collapsed manually.
- **Given** I cancel the drag, **then** the previous order and expansion state are
  restored.

---

## Story: I want to reorder apps, or move one between groups

**As a** user refining a page
**I want** to move an app up, down, or into a different group
**so that** related things sit together.

### Narrative

Same gesture, different target. Touch and hold an app row and it detaches and
follows my finger. Dragging within its group reorders it. Dragging into a different
group moves it there. No mode, no separate "move to group" command.

### Acceptance criteria

- **Given** an expanded group, **when** I drag an app row within it, **then** it
  reorders.
- **Given** multiple groups, **when** I drag an app into another group, **then** it
  is removed from the source group and inserted at the drop position.
- **Given** I drag the last app out of a group, **then** the empty group's handling
  follows the delete rule above.

---

## Story: I want to split a group in two

**As a** user whose single list has grown
**I want** to break some apps out into their own group
**so that** there's visual separation between contexts.

### Narrative

There is no "create group" command. Creating a group falls out of the drag gesture
that already exists.

I drag an app toward the gap between two existing groups. An insertion indicator
appears in the gap, the two groups slide apart, and an empty group box forms
between them. I drop, and that's a new group containing one app.

The new group has no heading. That's not a prompt I have to dismiss — headings are
always optional, and the default visual treatment of a group boundary is just a
thin divider. If I want to name it later, I can. Most of the time the separation
itself is the point, and a label would be noise.

### Acceptance criteria

- **Given** I am dragging an app, **when** I hover over the gap between two groups,
  **then** an insertion indicator appears and the groups animate apart to show a
  forming group.
- **Given** I drop in that gap, **then** a new group is created at that position
  containing the dragged app.
- **Given** a group is created this way, **then** it has no heading and the user is
  **not** prompted for one.
- **Given** a group with no heading, **when** the page renders, **then** its
  boundary is drawn as a thin divider only.

---

## Rejected alternatives

**A plain unfiltered checkbox list of every installed app.** This is what the app
that inspired RestWord does, and it's the specific thing we didn't want to copy. It
has no search, no ordering, and no way to modify what you've already built. Fuzzy
filtering on top of the same multi-select shape fixes it without abandoning batch
selection.

**Checkboxes in the app picker.** Dropped in favour of accent-colour row
highlighting. A checkbox is a second control sitting next to the thing it controls;
tinting the row makes the row itself the target and reads as calmer, which is
consistent with the rest of the product.

**A separate collapse/expand toggle control.** Rejected because the drag gesture
already triggers collapse exactly when it's useful. Adding a toggle would be a
control to learn for a state you otherwise get for free. (Though a deliberate
"collapse all" for overview purposes is still an open question — see 05.)

**Forcing groups to re-expand after a reorder.** Rejected explicitly. It would undo
the overview the user had just built for themselves.

**Prompting for a heading when a group is created.** Rejected. Headings are
optional by design, and a prompt makes the optional feel mandatory. It also adds a
modal interruption in the middle of a fluid drag gesture, which is the worst
possible moment for one.

**Separate + controls for "add app" and "add group".** An earlier idea put an
add-group control in the top bar alongside the settings gear. Once group creation
fell out of the drag gesture, this became mostly redundant. Flagged in 05 rather
than fully removed, because there may still be a case for creating an empty named
group up front.
