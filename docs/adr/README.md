# Architecture decision records

Drafts. They get updated in place rather than superseded, and the reasoning behind an edit
belongs in the section it changed.

| ADR | Decision |
| --- | --- |
| [0001](0001-rebuild-rather-than-fork.md) | Rebuild rather than fork |
| [0002](0002-platform-floor-and-visual-system.md) | Platform floor and visual system |
| [0003](0003-custom-display-names.md) | Custom display names for apps |
| [0004](0004-groups-ordering-and-page-identity.md) | Groups, ordering and page identity |
| [0005](0005-widget-settings-live-in-the-app.md) | Widget settings live in the app |

RestWord began as an attempt to fix bugs in
[Simple](https://github.com/MichaelZhao21/Simple), which turned out to have no licence and no
active maintainer — the reasoning is in ADR 0001. Two earlier records from that investigation
are not here: one on Simple's theming and the black-on-black rendering nobody has explained, one
on the widget-configuration crash and the Lawnchair launcher bug it exposed. Both analyse code
this project does not contain, so they stayed with that work, on the
`claude/simple-widget-lawnchair-issues-1lo0n2` branch of the Simple fork. Where the records here
refer to them, they are named rather than numbered, because their numbers mean something else in
this repository.

The visual documents in [`../visual`](../visual) are the other half of ADR 0002. Where the two
disagree, the visual documents are usually right and this directory needs an edit.
