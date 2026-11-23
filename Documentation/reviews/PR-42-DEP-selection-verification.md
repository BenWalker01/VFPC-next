# PR #42 DEP selection verification

- Pull request: [VFPC-next #42](https://github.com/VFPC/VFPC-next/pull/42)
- Related decision: ADR-005
- Test dates: 22–23 November 2026
- Environment: EuroScope with local Sweatbox
- Controller callsigns: `SCO_CTR` for DEP toggling and `EGPH_TWR` for the baseline comparison
- Tested commit: `3c8a27c9a595b575afa1d16a61014d408469bd21`
- Build: VFPC DLL 4.1.1.0, Release, clean working tree at build time (`dirty=false`)
- Build result: Successful
- Evidence: `vfpc-20261122144839.log`. All log times below are UTC.

## Result

**Pass for the EGPH/EGPF DEP-selection cases.** VFPC followed changes to
the airport-level DEP selection. Each deselected departure went to
`OFF / Not Checked`. Each reselected departure was picked up and checked
again. The departure from the airport left selected continued to be
checked.

The scenario used `RYR86GY` from EGPH and `RYR46XP` from EGPF. Other
airports were selected during the toggling sequence.

| Case | Expected behaviour | Observed evidence (UTC) |
| --- | --- | --- |
| Start with no airports in scope | Neither flight is checked. | At 14:48:59 VFPC reported no airports selected. Both flights were `OFF / Not Checked` when first seen at 14:49:34. |
| Both test airports selected | Both flights enter scope and airport data loads. | Both entered scope at 14:49:38. At 14:49:46 the data was available, EGPF returned `RTE` and EGPH was `PDG / Waiting for Runway`. |
| Clear both test airports | Both flights leave scope while other selections remain. | At 14:54:00 both airport snapshots were released and both flights returned to `OFF / Not Checked`. Other airports remained selected. |
| Select EGPH, then add EGPF | EGPH enters scope first. Both flights are checked once both airports are selected and the required data is available. | EGPH entered scope at 14:54:19. It returned `RTE` at 14:54:42 after runway 24 and SID `GOSAM1C` appeared in the flight plan. EGPF entered scope at 14:55:03 and returned `RTE` at 14:55:04. The log shows validation running for both. |
| Clear and reselect EGPH | EGPH goes out of scope, then is checked again. EGPF stays in scope. | EGPH went to `OFF` at 14:55:20 and returned to `RTE` at 14:56:44 after reselection and a successful data load. EGPF continued to be checked. No intervening EGPH flight-plan change was logged. |
| Clear and reselect EGPF | EGPF goes out of scope, then is checked again. EGPH stays in scope. | EGPF went to `OFF` at 14:57:15 and returned to `RTE` at 14:57:45 after reselection and a successful data load. EGPH continued to be checked. No intervening EGPF flight-plan change was logged. |
| Clear EGPH again | EGPH goes out of scope and EGPF stays in scope. | At 14:58:01 EGPH returned to `OFF` and EGPF continued to be checked. |

`RTE` confirms that VFPC ran the route check and rejected the filed route.
The pass criteria here concern whether each flight was checked when its
departure airport was selected.

## Baseline comparison at EGPH_TWR

**Pass.** With EGPF DEP selected in local Sweatbox, PR #42 validated
`RYR46XP` from EGPF under `EGPH_TWR`. The pre-PR DLL had excluded EGPF
under the same controller callsign and EGPF DEP setting.

| Build | VFPC console scope | EGPF `RYR46XP` | EGPH `RYR86GY` |
| --- | --- | --- | --- |
| Pre-PR DLL | UKCP airport-specific scope, EGPH only | `OFF` | `RTE` |
| PR #42 DLL | Departure Runway Activity scope, EGPF and EGPH | `RTE` | `RTE` |

Evidence is the baseline screenshot (`image(20261122-160459).png`),
the PR screenshot (`image(20261122-160948).png`), and the tester's
confirmation that EGPF DEP was selected on the baseline. The screenshots
show the scopes and Departure List results. The startup logs identify both
DLLs as version 4.1.1, DLL version 4.1.1.0, Release.

| Run | Startup (UTC) | Build commit | Working tree |
| --- | --- | --- | --- |
| Pre-PR DLL | 16:00:06 | `f80c47a365836de59d7b7a818aff6b3dd6c477da` | `dirty=false` |
| PR #42 DLL | 16:08:18 | `3c8a27c9a595b575afa1d16a61014d408469bd21` | `dirty=true` |

The tester confirmed that the only local change at build time was the
untracked `Documentation/reviews/PR-42-DEP-selection-verification.md`
file. The `EGPH_TWR` comparison therefore passed with a documentation-only
worktree change. The earlier `SCO_CTR` DEP toggling run logged
`dirty=false` at the same PR commit.

## VFR regression at EGPH_TWR

**Pass (23 November 2026).** In Sweatbox with the PR #42 DLL loaded,
VFPC reported both EGPH and EGPF in scope. The tester changed flight
rules for EGPF departure `RYR46XP` from IFR to VFR. Its Departure List
row showed `V` in the EuroScope **R** column and `VFR` in the VFPC
**FPL** column. The EGPH control departure `RYR86GY` still showed
`I` and `RTE`, respectively.

Evidence: `image(20261123-102606).png`. The screenshot shows VFPC
v4.1.1. The commit ID for this session was not captured in the screenshot.

The planned manual verification for PR #42 is complete.
