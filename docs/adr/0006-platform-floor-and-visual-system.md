# ADR 0006: Platform floor and visual system

**Status:** Draft

## Context

Two decisions that look unrelated turn out to be the same decision, because the thing that sets
our minimum Android version is typography.

### What the floor could be

Glance moved its default `minSdk` from 21 to 23 in 1.2.0-beta01, so 23 is the hard floor for the
stack; below it nothing resolves. The rest of androidx has moved the same way.

Reach, from apilevels.com: API 26 reaches about 96.1% of devices, API 31 about 78.8%. The
seventeen-point gap between them is the whole question, and it falls precisely on the population
we said we cared about — people on OEM devices that stopped receiving updates.

What API 31 would buy is narrow: `WIDGET_FEATURE_RECONFIGURABLE`, `targetCellWidth` and
`targetCellHeight` for grid-cell sizing, and `WIDGET_FEATURE_CONFIGURATION_OPTIONAL`. Dynamic
colour and the system widget corner radii are both moot here — see the palette below, and the
widget is transparent and text-only.

### Why 26 rather than 23

Glance cannot use a bundled font. Its API is `FontFamily(family: String)`, with constants for
`serif`, `sans-serif`, `monospace` and `cursive`: a family *name*, resolved in the host
launcher's process against system-installed fonts. There is no path that accepts a font
resource.

The escape hatch is `AndroidRemoteViews` — embed a hand-written layout that sets
`android:fontFamily="@font/…"`, which the launcher inflates from our package's resources, so a
bundled face does resolve. Font resources are API 26.

FiraGO on the widget is not decoration. The design puts the whole hierarchy on one family and
two weights, and the widget is the product; a widget rendering in the device's default sans
while the app renders in FiraGO would be two different products. So 26 buys the thing the design
depends on, and 23 forecloses it, for a reach difference that is small next to the one between
26 and 31.

This path needs confirming on a device before the design leans on it.

### The palette

`docs/visual/restword-dusk-palette.html` fixes a hand-built palette rather than deriving one
from the wallpaper. Dark leads; light is the companion for systems that ask for it.

## Decision

### Platform

`minSdk = 26`, `compileSdk` and `targetSdk` at 36. Declare the API 31 widget attributes anyway —
`widgetFeatures="reconfigurable"`, `targetCellWidth`, `targetCellHeight` — alongside the
pre-31 `minWidth` and `minHeight` in dp. Pre-31 frameworks do not parse the newer attributes, so
they cost nothing at runtime. Keep them in one file with `tools:targetApi="31"` on the root
element rather than maintaining a parallel `res/xml-v31/` copy of a ten-line file.

### Colour

A fixed palette, no Material You. Two schemes, one set of role names, one source of truth that
both the Compose theme and the Glance `ColorProvider`s read from — so nobody ever eyedroppers a
mockup.

| Role | Dark (primary) | Light (companion) |
| --- | --- | --- |
| `background` | `#0A0E16` Ink | `#F0F5F9` Sky Mist |
| `surface` | `#121826` Deep Slate | `#E2EBF3` Cloud Surface |
| `divider` | `#1F2637` Hairline | `#CFD9E2` Hairline |
| `textPrimary` | `#E6E9EF` Frost White | `#0A0E16` Ink |
| `textSecondary` | `#98A1B3` Muted Slate | `#576375` Slate |
| `accentCalm` | `#9CC2DE` Dusk Blue | `#3984C6` Dusk Blue |
| `accentActive` | `#79B0D8` Twilight Blue | `#2B6BAB` Twilight Blue |
| `accentWarm` | `#D19FAB` Dusk Rose | `#C05972` Dusk Rose |

`accentWarm` is reserved. It marks the widget identify flash described in ADR 0007 and nothing
else; if a second use appears, that is the signal to argue about it rather than to reach for it.

### Dark by default

Dark is not "the night variant", it is the app. The selection rule is *dark unless something
asks for light*, which inverts the usual `values/` plus `values-night/` arrangement — that
arrangement would hand every device below API 29 the light companion permanently, since system
dark mode does not exist there.

So: an in-app theme preference with three values — Dark, Light, Follow system — defaulting to
Dark. "Follow system" is only meaningful on 29 and above and is hidden below it. The gear in the
screen mockups is where it lives.

### Type

FiraGO, OFL, bundled as static instances rather than a variable font: Book (350) for app names
and body, Medium (500) for group headings. Two weights carry the hierarchy; size and colour do
the rest.

## Alternatives considered

**minSdk 23.** Another couple of points of reach, and the lowest the stack allows. Rejected
because it takes FiraGO off the widget, which is most of the design's identity on the only
surface that is always visible.

**minSdk 31.** Reconfiguration and grid-cell sizing for free, no `tools:targetApi`, no
in-app widget settings surface to build. Rejected on reach: seventeen points, concentrated
exactly among the people the project set out to be kind to. ADR 0007 gets us reconfiguration
without it, and better.

**Material You dynamic colour.** Free, familiar, and it makes the app feel native to each
device. Rejected because the design has a specific point of view that a wallpaper-derived
palette would overwrite, and because a fixed palette is one fewer variable — we never explained
the black-on-black rendering in Simple, and dynamic colour was among the candidates. Choosing a
fixed palette does not fix that bug, but it does mean it cannot be that.

**A system font on the widget, FiraGO in the app.** No `AndroidRemoteViews`, no font
plumbing, minSdk could be 23. Rejected: the widget and the app would not look like the same
product, and the widget is the one people actually see.

## Consequences

`AndroidRemoteViews` means part of the widget is a hand-written layout rather than Glance
composables, and click handling across that boundary needs care. The size of that compromise is
not yet known, which is why the font path gets verified before anything else is built on it.

Below API 29 there is no system dark mode, so "Follow system" is hidden and those devices sit on
the Dark default — which is the intended experience, not a degradation.

Adaptive icons need API 26, which we now have, but legacy mipmaps still ship for hosts that ask
for them. `android:forceDarkAllowed="false"` is API 29 and needs `tools:targetApi` to survive
`warningsAsErrors`; with a fixed palette it is belt and braces rather than load-bearing.

The palette table above becomes the source of truth. If a colour appears in code that is not in
it, that is a bug in this document or in the code, and one of them is wrong.

## Open questions

**Does the `AndroidRemoteViews` font path actually work on a device?** Everything above assumes
it does. Verify before building on it; if it fails, the choice is a system font on the widget or
a rethink of the type system.

**Is the ten-child Glance container ceiling real, and is it ten?** Inherited from ADR 0005 and
still unmeasured.

**What does the page title look like on the widget?** ADR 0005 has it replacing "App List N" as
a toggleable label, and it is now also the tap target for configuring that widget. It is
competing with group headings for the same attention, and that is a drawing problem rather than
a writing one.
