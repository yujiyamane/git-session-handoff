# Worked Examples

## Example 1: Feature Commit with Partial Work and Failures

A realistic mid-development commit where one integration is complete, another is in progress,
and one dead end is documented.

```
feat: add weather API integration to daily digest email

✅ Completed:
- Open-Meteo API integration (temperature, wind speed, conditions)
- Location coordinates stored in APP_CONFIG
- HTML section renders weather data with conditional emoji
  (⛅ sunny, 🌧️ rainy, 🌬️ high-wind)
- 3 tests PASS: testWeatherFetch, testWeatherHtml, testNoDataFallback

🚧 In Progress:
- Humidity data API call written but not wired
  to HTML template

❌ Failed:
- WeatherAPI.com requires paid key — abandoned after 20 min research
- Open-Meteo wind_direction returns degrees not compass;
  conversion function degreesToCompass_() drafted but 360/0
  boundary untested

➡️ Next:
- Wire humidity data to template (function exists: formatHumiditySection_)
- Fix 360/0 boundary in degreesToCompass_() and add
  testCompassEdgeCases to tests/unit.test.js
- Deploy + manual trigger to verify layout
- STILL OPEN from 2026-06-03: score rounding bug in dashboard (low priority)
```

## Example 2: Task Management Promotion

A session-end commit where two ➡️ items qualify for promotion to external task manager.
Note: promoted items are removed from ➡️ Next to prevent duplicate task creation.

```
feat: data ingestion — third-party health CSV parser

✅ Completed:
- parseCsv_() reads exported CSV from third-party data source
- 5 tests PASS: testParseCsv, testDataHtml,
  testDataNoData, testIconGreen, testIconRed

❌ Failed:
- REST API approach — OAuth scope not supported natively.
  Switched to CSV export approach instead.

➡️ Next:
- Implement formatDuration_() in src/formatter.js
- Wire data section into APP_CONFIG section ordering
- Update build-spec.md with data section spec

📤 Promoted to Task Manager:
✓ task: "Share CSV export setup steps with team member"
  (created: Tags=Documentation, Due=2026-06-08)
✓ task: "3-day monitoring check after production deploy"
  (created: Tags=DevOps, Due=2026-06-08)
```

Note: "Share CSV export setup steps" and "3-day monitoring check" were removed from
➡️ Next when promoted. They no longer consume ➡️ slots and will not be re-evaluated.

## Example 3: Session Start Commit

The mandatory empty commit that opens every session, recording triage decisions.

```
chore: session start — wire humidity section and fix compass boundary

✅ Completed:
- Read git log -5: identified 4 open items from 2 previous sessions
- Decision: fix compass 360/0 first (unblocks layout test), then wire
  humidity section, then deploy

➡️ Next:
- Fix 360/0 boundary in degreesToCompass_() in src/utils.js
```

## Example 4: Exploration / Read-Only Session

A session with no code output — findings and decisions preserved in an empty commit.

```
chore: checkpoint — exploration session

✅ Completed:
- Investigated pagination approach for API responses: read docs,
  traced existing fetchData_() call chain in src/api.js
- Key finding: current implementation silently drops page 2+ results;
  cursor-based pagination param documented but never wired
- Decision: use cursor-based approach (not offset); matches API rate
  limit structure; fewer edge cases than offset on live data

❌ Failed:
- Offset pagination: tested locally, returned duplicate rows on
  concurrent writes — not viable for live data

➡️ Next:
- Implement cursor-based pagination in fetchData_() in src/api.js
- Add testPaginationCursor and testPaginationDrops to tests/api.test.js
```

## Example 5: Age-Out Promotion

A STILL OPEN item that has been dormant for 7+ days is promoted unconditionally.

```
fix: correct null handling in report generator

✅ Completed:
- formatReport_() now guards against null input — returns empty string
- 2 tests PASS: testReportNull, testReportEmpty

➡️ Next:
- Add CSV export option to formatReport_() in src/report.js

📤 Promoted to Task Manager:
✓ task: "Investigate intermittent test failure in CI pipeline"
  (created: Tags=DevOps, Due=2026-06-12)
```

The CI investigation item had been STILL OPEN since 2026-06-04 (8 days) — age-out gate
triggered regardless of whether the standard gate criteria were met.
