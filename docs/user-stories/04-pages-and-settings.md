# Pages and Settings

**Status: draft.**

The in-app surfaces. Worth being honest about their role: the widget config screen
is the primary entry point for essentially everything, so these screens are a
browse-and-overview fallback rather than the main way people manage their setup.
They should be good, but they shouldn't be where the design effort concentrates.

---

## Story: I open RestWord directly and want to find a page

**As a** user who wants to edit something specific
**I want** to see all my pages at once
**so that** I can jump straight into the one I mean.

### Narrative

Opening the app lands on the page list, front and centre. That's the thing people
actually manage, so it gets primary billing rather than competing with settings.

Each row shows the page's **name** prominently. Underneath is a secondary line,
resolved in this order:

1. The user's own description, if they've written one.
2. Otherwise, an auto-generated preview built from the first few app names on that
   page.

Name is the primary identity; description is optional texture that most people will
skip. The fallback matters because it means even a page someone never described
still shows something meaningful instead of a blank line.

Tapping a row opens that page's group editor — the same screen the widget's deep
link lands on. There's one editor, reached two ways.

App-wide settings sit behind a gear icon in the top bar, one tap away and
deliberately not competing for space.

### Acceptance criteria

- **Given** the app opens with no deep link, **then** the page list is the landing
  screen.
- **Given** a page with a description, **when** its row renders, **then** the
  description is the secondary line.
- **Given** a page with no description, **then** the secondary line is generated
  from its app names.
- **Given** a page with no description and no apps, **then** the secondary line
  handles the empty case gracefully. (See 05.)
- **Given** I tap a page row, **then** the group editor opens for that page.
- **Given** the page list, **then** settings are reachable via a gear icon and are
  not the primary element.

---

## Story: I want to give a page a description

**As a** user with several similar pages
**I want** to write a note about what a page is for
**so that** the list is readable at a glance.

### Narrative

Optional, and genuinely overkill for most people. It exists because a page already
has a name and a description gives somewhere to attach detail without committing to
a whole new content type inside the list itself. It's a small surface for notes, not
a feature anyone has to engage with.

### Acceptance criteria

- **Given** a page, **then** its description is optional and empty by default.
- **Given** I set a description, **then** it replaces the auto-generated preview in
  the page list.
- **Given** I clear it, **then** the auto-generated preview returns.

---

## Story: I want the app to match my phone's theme

**As a** user
**I want** to control theme and font
**so that** the widget sits comfortably on my home screen.

### Narrative

Settings is short on purpose. Two things live here, both genuinely app-wide rather
than per-page:

- **Theme** — light, dark, or follow system.
- **Font** — the typeface used by the widget and the app.

Font *size* is deliberately not here. Size is per widget instance, set on the
config screen, because two widgets on different home screens can reasonably want
different sizes. Typeface is global because mixing typefaces across widgets would
look like a mistake.

### Acceptance criteria

- **Given** settings, **then** theme offers light, dark, and follow-system, with
  follow-system as the default.
- **Given** I change the theme, **then** both the app and all widget instances
  update.
- **Given** settings, **then** font size is **not** offered, since it is per widget
  instance.

---

## Rejected alternatives

**Settings as the app's landing screen.** Rejected. Pages are what people manage;
settings are set once and forgotten. Giving settings primary billing would invert
the actual frequency of use.

**Making the description the primary identifier.** Rejected in favour of name as
identity with description as optional detail. A required description is a chore, and
a page called "Morning" needs no further explanation.

**Letting the user choose between description and auto-preview.** Rejected as a
pointless setting. Falling back automatically gives the same outcome with nothing
to configure.

**Creating pages from the page list as the primary path.** The page list may well
have a create affordance, but it is explicitly *not* the path we design or document
for. Page creation belongs to the widget config screen so that one learned flow
covers day one and day one hundred. Anything here is a convenience, not the route.

**Global font size in settings.** Rejected because size is legitimately per widget
instance — a compact widget sharing space with something else wants smaller text
than a full-page one.
