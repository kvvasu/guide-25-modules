# Notes and Limitations

## Missing modules

| Module | Status | Detail |
|---|---|---|
| 1–13 | Never generated | These modules do not exist in this session’s history. They are not included and must not be inferred. |
| 14 | MISSING FROM HISTORY | Full body, proof table, code blocks, error table, and design note unavailable. |
| 15 | MISSING FROM HISTORY | Full body, proof table, code blocks, error table, and design note unavailable. |
| 16 | MISSING FROM HISTORY | Only a closing excerpt was visible. Full body and supporting tables unavailable. |
| 17 | MISSING FROM HISTORY | Only the beginning of the proof table and verification text were visible. Full body unavailable. Claim text was not recovered from chat history. |
| 18 | MISSING FROM HISTORY | Only the beginning of the proof table and verification text were visible. Full body unavailable. Claim text was not recovered from chat history. |
| 19 | MISSING FROM HISTORY | Only the beginning of the proof table and verification text were visible. Full body unavailable. Claim text was not recovered from chat history. |
| 20–25 | Present | Assembled from the text generated in this session. |

## Missing history

- The proof table for Modules 20–22 is missing. The assembled file states this explicitly instead of reconstructing rows.
- Complete proof tables for Modules 14–19 are missing.
- The original Module 14–19 text was examined from the conversation history. Only the module headings and the beginning of each proof table (stating that only the beginning is visible) are present; the full module bodies, proof tables, code blocks, error tables, design notes, and all supporting text could not be recovered.
- Some documentation URLs cited in prose were visible during research but were not written into the module text; the audit counts 84 named-page claims with no inline URL.

## Reconstructed or flagged material

- `RangeError: Maximum call stack size exceeded` in Module 22 is labeled as an expected/reconstructed JavaScript error, not as text fetched from a Community thread.
- `Transaction cancelled: maximum execution time exceeded` is labeled as an expected/reconstructed outcome for the N+1 drill. The fetched 2025 thread reports the report-level variant `Widget cancelled - maximum execution time exceeded`.
- Module 23 reports **no measured performance numbers**. The measurement sheet is blank on purpose.
- Module 25 uses the exact six-domain CAD list and weights supplied in Round 4. It is not derived from the blocked Now Learning page.
- The `sdk-examples` folder list is quoted from the repository README. The GitHub contents API was rate-limited.

## Not verified

- No PDI was available. No lab step, seed script, index, report, dashboard, clone, OAuth flow, REST call, MID Server probe, Git command, or SDK command was executed.
- No measured timing, SQL-count, or response-time result is claimed.
- Community thread dates were not consistently available and are marked `not extracted`.
- The official Now Learning CAD blueprint page returned no readable text; the domain list and weights come from the user-supplied list, not from that page.
- The guide has not been tested against a live Zurich instance.

## Audit status

- Scope: Modules 20–25.
- Source-bearing quotes checked: 11; matched after repair: 11; mismatch rate: 0.0%.
- Code blocks: 29 of 29 labeled and unindented.
- Unsourced named-page claims: 84.
- The scoped pass mark is met; the whole-file pass mark is not assessed because Modules 1–19 are missing or incomplete.
- Repository should remain **private** until the missing modules are supplied and a full audit is run.

## File provenance

- `guide/modules_14_to_25.md` — assembled guide, 15,053 words.
- `audit/phase4_audit_modules_14_to_25.md` — scoped audit report.
- `README.md` — repository introduction and scope warning.
- `NOTES.md` — this file.
