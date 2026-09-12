# ADR 0005: Widget settings live in the app

**Status:** Draft

## Context

Two settings belong to a placed widget rather than to a page: which page it shows, and its font
size. Everything else — pages, groups, names, order, descriptions — belongs to the data and is
edited in the app.

The conventional home for per-widget settings is the configuration activity, launched by the
host when a widget is dropped. Reaching it again afterwards depends on
`WIDGET_FEATURE_RECONFIGURABLE`, which is API 31, and we target 26 (ADR 0002). On a pre-31
device those two settings would be chosen once, in a modal thrown up mid-drag, and then frozen
for the life of the widget.

There is a second reason not to depend on that path, and it is not theoretical. The launcher
side of widget reconfiguration is the least reliable code in this ecosystem. Tracing the bug
that started this project turned up six or more Lawnchair issues closed by working around
`startConfigActivity` failing with "Bad widget id", a reallocate-and-retry workaround, and then
a second commit capping that retry loop at three attempts because it could spin. Building the
main interaction surface on top of an arbitrary third-party launcher re-invoking our activity
correctly is building on the one part of the platform we have direct evidence is unreliable.

### What the platform will and will not tell us

The host owns placement and exposes none of it. There is no API for which home screen a widget
is on, where, or how large in grid cells. `getAppWidgetOptions` gives approximate dp bounds and
the host category, and that is all. We cannot label an instance in the launcher either — the
launcher shows the provider's label, identical for every instance. Long-press inside a widget is
not available; RemoteViews supports click only, and long-press belongs to the host.

What we can do is re-render any widget instantly and make any part of it tappable. That is
enough.

## Decision

**Per-widget settings are edited in the app.** A widget settings surface, reached from the gear,
lists every placed widget with the page it shows and its font size, and opens the same picker
used at drop time.

**The way in is the widget itself.** Tapping a widget's page title opens that widget's settings
directly. The person was already looking at the widget they wanted to change, so there is
nothing to identify and nothing to name. This works on every supported version.

**The drop-time configuration activity does the minimum**: choose a page, or create one. It is a
modal the system throws up in the middle of a drag, with no room and no second chance, and it is
no longer the only way back.

**The launcher's reconfigure affordance stays declared** — `widgetFeatures="reconfigurable"`,
per ADR 0002 — so people on API 31 and above who reach for the pencil get what they expect. It
is a courtesy on top of the real mechanism, not the mechanism.

**In the list, rows are labelled with what is already meaningful**: page title and font size,
ordered by widget id, which is the order they were added. Where two widgets show the same page,
the approximate size from `getAppWidgetOptions` disambiguates them weakly.

**Tapping a row flashes that widget.** The row's widget re-renders in `accentWarm` for a couple
of seconds so the person can look at their home screen and see which one answered, then reverts.
This is the reserved use of Dusk Rose from ADR 0002. It is the same idea as "identify this
display" in monitor settings.

## Alternatives considered

**A user-editable name per widget.** The obvious answer, and it puts the person in control.
Rejected because it asks someone to name a thing at the worst possible moment — seconds after
dropping it, before they have any reason to believe they will ever own two — and because it is a
third naming concept beside page titles and group titles in an app whose entire argument is
restraint. Most people would type nothing, or "widget".

**A generated pronounceable code per widget.** No effort required and guaranteed unique.
Rejected because a code means nothing until the widget displays it, which puts "zibro"
permanently on the home screen of an app whose pitch is no icons, no branding, just the list.
Show it only during an identify mode and the code was never needed — flashing the widget does
the same job without the artefact.

**Require API 31 and use the launcher's reconfigure flow.** No second surface to build, one
entry point, the design's "used identically on day one and day one hundred" intact. Rejected on
reach (ADR 0002) and on reliability, above.

**Put per-widget settings on the page editor** — show which widgets use this page and edit their
font size there. Keeps the surface count down. Rejected because it inverts the relationship: a
widget points at a page, not the other way round, and a page used by no widgets would have a
confusing empty section.

## Consequences

There are now two entry points to widget settings rather than one, which costs the design its
"single entry point" story. In exchange the surface that matters is reachable forever, on every
version, without the launcher's cooperation.

The identify flash needs the app process alive, which it is by definition — the person is in the
app looking at the list. It also needs a revert that survives the app being backgrounded
mid-flash, so the widget must not be left permanently rose.

The list degrades quietly past three or four widgets on the same page, where neither the page
title nor the approximate size separates them and only the flash does. Acceptable; worth
watching if anyone ever reports it.

Tapping the page title to configure means the title has to be shown. A widget with its page
title hidden — which ADR 0004 allows — has no tap affordance and can only be reached from the
app. That is a defensible consequence of choosing minimalism, but it should be a stated
consequence rather than a surprise.

## Implementation notes

`PendingIntent` equality ignores extras. Two widgets whose tap intents differ only by an
`appWidgetId` extra can collapse onto one `PendingIntent`, and the second widget will silently
deliver the first one's id — so tapping widget B opens widget A's settings. Give each intent a
distinct `data` Uri, `restword://widget/<appWidgetId>`, since data *is* part of the comparison.
This is a classic widget bug and it presents as flakiness rather than as a mistake.

Enumerate placed widgets through `GlanceAppWidgetManager.getGlanceIds(...)`, which returns only
currently-bound ids, so removed widgets fall out of the list without bookkeeping.

## Open questions

**Where the widget list lives.** Behind the gear is the assumption here. Hanging it off the page
list is the alternative and would surface it more, at the cost of putting infrastructure on the
screen that is meant to be about content.

**Whether the flash should be colour or motion.** Colour is all Glance gives us cheaply; a
fade or a brief inset would read as calmer if it is achievable at all.
