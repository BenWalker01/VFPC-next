# Controller callsign variants verification

16 November 2026. Follow-up to the v4.1.0 report for `EGSS__GND` and
`EGSS_M_GND`. No installed DLL was replaced by the build checks.

## Automated checks

- Debug and Release Win32 MSBuild builds passed with zero warnings and errors.
  vcpkg integration was explicitly imported and dependency installation disabled;
  outputs and intermediate files were redirected away from the installed plugin.
- Release DLL: `C:\Users\peter\.codex\visualizations\2026\11\16\01a1e4b9-9ca8-7000-8a06-c333274f9b40\verified-build\Release\VFPC.dll`.
  SHA-256: `BD27C5CAE72DA6F0168EBF0C58FE1D2B93DF90BCEEC74B97E334D82B6A601AEF`.
  Generated identity: commit `c4bd1166fafe32da0a41e5755291c26a5fd03f53`, dirty `true`.
  Debug/Release build logs are in the parent `verified-build` directory.
- 24 focused C++20 MSVC x86 assertions passed with `/W4 /WX`, using the lookup
  block extracted from production source. Cases cover normal, relief and local
  mentor resolution; sector-preserving APP/CTR relief; unsupported aliases;
  exact-record precedence; and empty, conflicting or multi-airport coverage.

## Pending user-operated EuroScope/Sweatbox checks

Load the verification Release DLL using EuroScope's plugin settings. Confirm
its path and startup build identity before testing; retain the existing DLL so
it can be restored afterwards. Use an IFR EGSS departure with suitable test
constraints and an assigned departure runway, plus an IFR departure from a
different airport to check exclusion. Run `.vfpc scope` to capture scope details.

| Callsign / condition | Expected result | Status |
|---|---|---|
| `EGSS_GND` | EGSS selected through exact UKCP mapping | Passed: user screenshot, 15:15:00 / 15:15:25 |
| `EGSS__GND` | EGSS selected; diagnostic identifies base `EGSS_GND` | Passed: user screenshot, 15:16:30 / 15:16:45 |
| `EGSS_M_GND` | EGSS selected; diagnostic identifies base `EGSS_GND` | Passed: user screenshot, 15:17:41 |
| `EGSS_M_GND` with eligible EGSS IFR departure | Validation runs; result depends on flight constraints | Passed: TESTSS, runway 04, displays SID; console reports DET2D absent from database |
| `EGSS__GND` with eligible EGSS IFR departure | Validation runs; result depends on flight constraints | Passed: TESTSS visibly displays SID, runway 04 |
| `EGSS_GND` with eligible EGSS IFR departure | Validation runs; result depends on flight constraints | Passed: TESTSS visibly displays SID, runway 04 |
| `EGSS_M_GND` with another airport's IFR departure | OFF | Passed: TESTGW, EGGW, runway 25, visibly displays OFF alongside TESTSS SID |
| Other-airport detailed OFF explanation | Show Checks identifies out-of-scope origin | Passed under `EGSS__GND`: TESTGW OFF / Not checked / Origin is outside the selected controller scope |
| `EGSS__GND` with another airport's IFR departure | OFF | Passed: TESTGW (EGGW, runway 25) displays OFF alongside TESTSS SID |
| `EGSS_GND` with another airport's IFR departure | OFF | Passed: TESTGW visibly displays OFF; console confirms origin outside selected scope |
| Local scope with EGSS not departure-active | EGSS retained with Runway Activity warning | Passed: normal and relief screenshots show warning and retained EGSS |
| `EGSS_TEST_GND` (absent from UKCP) | Scope paused; no inferred airport | Pending |

An exact mapping, including an empty or conflicting mapping, overrides aliases.
Do not claim these precedence cases were exercised in the live UKCP data; they
are covered by the focused assertions. A mentor callsign's base mapping must
still name exactly one airport matching its prefix.

Scope results above were reviewed from user-supplied screenshots in the chat.
The mentor screenshot has a notification obscuring the right-hand portion of
the console, but the wrapped 15:17:41 line visibly identifies `EGSS_GND` and
`Selected airports: EGSS`. Departure lists in those initial screenshots are
empty, so they did not establish per-flight validation or other-airport exclusion. A subsequent
two-aircraft scenario screenshot confirms TESTSS (EGSS, runway 04) reaches SID
validation under `EGSS_M_GND`; the console reports DET2D absent from the database.
This proves validation ran, not that the scenario route is current or valid.
The next screenshot shows TESTGW (EGGW, runway 25) in the departure list with
OFF alongside TESTSS with SID, under `EGSS_M_GND`. This confirms airport
exclusion at the tag level; the detailed Show Checks explanation is not shown.
The screenshots
alone do not verify the loaded DLL hash. The optional unknown-callsign runtime
check remains pending (unsupported aliases are covered by focused assertions).
Release remains pending.

A subsequent relief-session screenshot shows `EGSS__GND`, TESTSS SID and
TESTGW OFF. The console explicitly reports TESTGW as not checked because its
origin is outside the selected controller scope, and identifies resolution to
`EGSS_GND` with selected airport EGSS.

The final baseline screenshot shows `EGSS_GND`, TESTSS SID and TESTGW OFF,
with the console confirming that TESTGW's origin is outside the selected
controller scope. Scope selection, in-scope validation and other-airport
exclusion have therefore been observed for all three reported/test callsigns.
