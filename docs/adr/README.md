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

Numbering starts at 0003 because 0001 and 0002 are about a different codebase. RestWord began as
an attempt to fix bugs in [Simple](https://github.com/MichaelZhao21/Simple), which turned out to
have no licence and no active maintainer — the reasoning is in ADR 0001. Those first two records
analyse Simple's source and the Lawnchair launcher bug that the investigation turned up, so they
stayed with that work rather than moving here. They are on the
`claude/simple-widget-lawnchair-issues-1lo0n2` branch of the Simple fork.

The visual documents in [`../visual`](../visual) are the other half of ADR 0002. Where the two
disagree, the visual documents are usually right and this directory needs an edit.
