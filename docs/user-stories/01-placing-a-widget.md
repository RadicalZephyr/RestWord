# Placing a Widget

**Status: draft.**

This is the flow that matters most, because it's the entry point to essentially
everything. Creating a page happens here. Editing an existing page starts here.
The in-app page list is a fallback surface, never a required one.

---

## Story: I install RestWord and want something on my home screen

**As a** new user who just installed RestWord
**I want** to get a working text list onto my home screen
**so that** I can stop looking at app icons.

### Narrative

After install, there's nothing pushing me into the app. If I open it, I get the
page list, which is empty. That's fine, but it isn't the path we're designing for.

The path we're designing for is the Android-native one: long-press the home
screen, find RestWord in the widget picker, drag an instance onto a page. Android
then shows the widget's configuration screen, which is ours.

With zero pages existing, that screen has exactly one action: **Create new page**.
There's no page picker to show and no font-size control yet, because there's
nothing to size. Tapping it deep-links me into the app's page editor, which is the
same editor I'd use for my fifteenth page. I add apps, optionally organize them,
and back out.

I land back on the config screen, now with my page selected. The font-size slider
has appeared, with an approximate live preview of my actual apps underneath it. I
set a size and tap **Done**. The widget is placed. Nothing else is asked of me.

### Acceptance criteria

- **Given** no pages exist, **when** the config screen opens, **then** "Create new
  page" is the only offered action and no font-size control is shown.
- **Given** I tap "Create new page", **when** the app opens, **then** it opens
  directly into the page editor for a newly created page, not the page list.
- **Given** I back out of the page editor, **when** I return to the config screen,
  **then** the page I just created is selected.
- **Given** a page is selected, **when** the config screen renders, **then** the
  font-size slider appears with a live preview built from that page's real content.
- **Given** I tap Done, **when** the config screen closes, **then** the widget is
  placed immediately with no further steps.
- **Given** I abandon the flow before tapping Done, **then** no widget is placed.
  (Whether the orphaned page survives is an open question — see 05.)

---

## Story: I already use RestWord and want another page

**As an** existing user with pages already set up
**I want** to add a new page for a new home screen
**so that** I can organize a different context separately.

### Narrative

This is deliberately the *same* flow as above, not a different one. I drag another
widget instance onto a new area of my home screen. The config screen appears, and
this time it shows a radio list of my existing pages — Morning, Evening, Focus —
with **Create new page** sitting right there underneath them.

That "Create new page" affordance is present **every time**, not only when the list
is empty. This is the correction that makes the whole model coherent: there is one
way to make a page and I learned it in my first five minutes.

If I pick an existing page instead, I get a second widget showing the same content,
possibly at a different font size. That's a legitimate thing to want, not an error.

### Acceptance criteria

- **Given** one or more pages exist, **when** the config screen opens, **then** it
  shows a single-select list of pages *and* a "Create new page" action.
- **Given** I select an existing page, **when** the font-size slider appears,
  **then** the preview reflects that specific page's content.
- **Given** two widget instances point at the same page, **when** I edit that page,
  **then** both widgets update.
- **Given** two widget instances point at the same page, **when** I change one
  widget's font size, **then** the other is unaffected.

---

## Story: I want to change a widget I already placed

**As a** user with a widget already on my home screen
**I want** to get into that page's editor
**so that** I can add, remove, or reorder its apps.

### Narrative

The widget carries its own identity, so the deep link into the app is already
scoped to the right page. I don't navigate; I arrive.

Reconfiguring a placed widget goes through Android's own widget-reconfiguration
path, which reopens the same config screen. Changing the page a widget points at,
and changing its font size, both happen there.

### Acceptance criteria

- **Given** a placed widget, **when** I reconfigure it, **then** the config screen
  opens with its current page pre-selected and its current font size set.
- **Given** I change the selected page, **when** I tap Done, **then** the widget
  re-renders against the new page without being re-placed.

---

## Rejected alternatives

**Editing inside the widget itself.** Tempting, because zero-hop editing is
appealing — you tap the thing you're looking at. Rejected because RemoteViews gives
no free-form text input, no drag-to-reorder, and unpredictable rendering across OEM
launchers. Building group editing there would fight the platform constantly and
still end up worse than a real screen. The widget stays dumb and reliable.

**Forcing an app launch on first widget placement.** Rejected as a bad context
switch. Dragging a widget onto the home screen is a home-screen action; yanking
someone into a full app immediately breaks the gesture they were in the middle of.
The deep link is triggered by an explicit tap instead.

**Auto-creating a default starter page.** This would have made the config screen
uniform by always giving it something to point at. Rejected because it produces a
meaningless empty page that most users then have to notice and delete.

**A dedicated onboarding or tutorial flow.** Including a special "text item"
content type purely to hold tutorial copy. Rejected as a new data-model concept
justified by exactly one use, which is the kind of special case that rots. The
empty state in the real editor teaches the same thing with nothing extra to
maintain or keep in sync.

**"Create new page" only when the list is empty.** This was the original shape and
it was wrong. It would have meant first-time page creation and subsequent page
creation used different paths, so the thing you learned on day one didn't transfer.
Now it's always present.

**A pixel-perfect font-size preview.** Rejected as not worth the engineering cost.
Matching RemoteViews rendering exactly across OEM launchers is a deep hole, and an
approximate preview answers the only question the user actually has, which is
roughly how big the text will be.

**Native RemoteViews scrolling (StackView / ListView) for long pages.** Technically
available, but it feels dated and janky and constrains layout. Multiple widget
instances across multiple home screens is the compromise we took instead.
