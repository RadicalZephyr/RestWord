# RestWord — User Stories

**Status: draft.** These document the interaction flows we talked through in design
sessions. They describe intent and behaviour, not implementation. Nothing here is
settled beyond the point of revision, but the decisions recorded as decisions have
been argued through rather than assumed.

## Documents

| Doc | Covers |
| --- | --- |
| [01-placing-a-widget.md](01-placing-a-widget.md) | Widget placement, the Android config screen, creating a page |
| [02-building-a-page.md](02-building-a-page.md) | The group editor: adding apps, ordering, groups |
| [03-customizing-an-app-entry.md](03-customizing-an-app-entry.md) | Renaming an entry, choosing which shortcut it launches |
| [04-pages-and-settings.md](04-pages-and-settings.md) | The in-app page list and app-wide settings |
| [05-open-questions.md](05-open-questions.md) | Gaps we've identified but not closed |

## Vocabulary

These three words mean specific things throughout. Worth pinning down because the
data model leans on them being consistent.

**Page** — the top-level content unit. A page is an ordered list of groups. Pages
have a name (primary identity) and an optional description. A page exists
independently of any widget; nothing stops two widgets pointing at the same page.

**Group** — an ordered list of apps inside a page, with an *optional* heading. A
page with no visible structure at all is still exactly one group, an anonymous
headerless one. There is no separate "ungrouped" concept and no second code path
for the simple case.

**App entry** — a reference to an installed app inside a group, with an optional
custom label and an optional shortcut it launches instead of the app's main
activity. The entry is not the app; renaming an entry doesn't touch anything
outside RestWord.

**Widget instance** — a thin viewport onto one page, plus a font size. That's the
whole of a widget's own state. Each instance needs a stable identity it can pass
in a deep link so the app knows which page to open into.

## Guiding principles

These came out of the design discussion and are worth stating because most of the
specific decisions below fall out of them.

**The widget is the product; the app is scaffolding.** The widget is what someone
looks at fifty times a day. The app exists to configure it. That inverts the usual
priority: the app can be plain, the widget can't.

**One path learned once.** Creating a page on day one and creating a page on day
one hundred use the identical flow, because there is no separate onboarding. The
empty state *is* the tutorial, rendered in the same UI a power user works in.

**Structure without forced labels.** Groups don't require headings. Apps don't
require custom names. Every organizational feature is additive on top of something
that already works plain. This mirrors the point of the whole product: don't make
people do work to get calm.

**Don't fight RemoteViews.** The widget renders through a restricted system with
no text input, no drag-to-reorder, and inconsistent behaviour across OEM launchers.
Anything interactive lives in the app behind a deep link. The widget displays and
launches, nothing more.
