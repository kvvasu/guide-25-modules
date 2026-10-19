# Phase 4 Audit — Modules 14–25

**File audited:** `guide/modules_14_to_25.md` (15,053 words).  
**Audited modules:** 20, 21, 22, 23, 24, 25.  
**Out of scope:** Modules 1–13 were never generated in this session. Modules 14–19 are present only as `MISSING FROM HISTORY` stubs and contain no prose, quotes, code blocks, or sources to audit.

**Pass mark:** under 5% of audited source quotes failing, and every code block marked.

## 1. Quote checks

| Metric | Count |
|---|---:|
| Quoted spans extracted from Modules 20–25 | 32 |
| Quoted spans that are code literals, config values, response-field names, or status labels | 21 |
| Source-bearing quotes checked against a fetched page | 11 |
| Source-bearing quotes matched | 11 |
| Source-bearing quotes mismatched | 0 |
| Non-source colloquial quotation marks cleaned up | 1 |
| Mismatch rate | 0.0% |

Quotes with ellipses were treated as separate fragments. The `RangeError: Maximum call stack size exceeded` example is explicitly labeled reconstructed in the guide and has no fetched-source citation, so it is not counted as a source quote.

### Mismatch table

| Module | Quote fragment | URL | Problem | Correction |
|---|---|---|---|---|
| 23 | “the index fixed it” | none; colloquial phrase, not a source quote | Quotation marks implied a citation that does not exist. | Removed the quotation marks and rephrased as ordinary prose. This is a style cleanup, not a failed source check. |

No source-quote mismatch was found in the checked set.

## 2. URL table

All ServiceNow Community URLs below returned HTTP 200 and a readable `<title>` when fetched. Dates were not consistently exposed in the fetched HTML and are recorded as `not extracted` rather than guessed.

| Complete URL | Exact title | Date | Status |
|---|---|---|---|
| https://www.servicenow.com/community/api-insights-forum/custom-data-stream-action-error-error-code-1-socket-closed-when/m-p/3434429 | Custom Data Stream Action Error: [ERROR CODE: -1] ... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/csm-forum/how-to-clear-the-field-value-based-on-another-fiel/m-p/3219367 | Solved: How to clear the field value based on another fiel... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/getting-400-message-when-sending-attachment-from-one-servicenow/m-p/3188394 | Solved: Getting 400 Message when sending attachment from o... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/rest-api-explorer-returns-403-forbidden-for-admin-role-on-table/m-p/3280304 | REST API Explorer returns 403 Forbidden for admin ... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/syslog-showing-many-invalid-query-detected/td-p/3145556 | Solved: Syslog showing many Invalid query detected . . . - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/transform-map-how-to-display-a-custom-error-when-a-coalesce/m-p/3425888 | Transform Map: How to display a custom error when ... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/unable-to-clone-git-repo-in-servicenow-ide/m-p/3290332 | Unable to clone git repo in Servicenow IDE - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/developer-forum/widget-cancelled-maximum-execution-time-exceeded-quot/m-p/3170494 | Widget cancelled - maximum execution time exceeded... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/itom-forum/how-to-resolve-this-quot-mid-server-failed-to-upgrade-quot-issue/m-p/3153088 | How to resolve this "MID Server failed to upgrade"... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/platform-analytics-forum/dashboard-is-showing-empty-in-test-environment-post-deployment/m-p/3403341 | Dashboard is showing empty in test environment pos... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/servicenow-studio-forum/cannot-use-quot-link-to-source-control-quot-need-alternative/m-p/3289514 | Solved: Cannot Use "Link to Source Control" – Need Alterna... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/sysadmin-forum/scheduling-a-clone-destination-does-not-appear-to-be-a-service/m-p/3340353 | Solved: Scheduling a clone: Destination does not appear to... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/sysadmin-forum/sharing-platform-analytics-dashboard-while-multiple-users-have/td-p/3415149 | Solved: Sharing Platform Analytics dashboard while multipl... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/upgrades-and-patching-forum/clone-from-production-to-development-fails-at-applying/m-p/3392564 | Clone from Production to Development Fails at “App... - ServiceNow Community | not extracted | 200 |
| https://www.servicenow.com/community/upgrades-and-patching-forum/how-to-make-successful-connection-oauth2-0-with-grant-type/m-p/3194565 | How to make successful connection OAUTH2.0 with g... - ServiceNow Community | not extracted | 200 |
| https://github.com/ServiceNow/sdk-examples | ServiceNow/sdk-examples: ServiceNow SDK examples | not extracted | 200 |

Placeholder URLs such as `https://<instance>.service-now.com` are templates, not sources, and are excluded from this table. Documentation URLs cited in prose but not written into the file are listed as unsourced below.

## 3. Code-block audit

All 29 fenced blocks have a label immediately above the fence, an unindented opening fence, and a marker identifying the block as written by the author or fetched from a repository.

| Module | Lines | Label present | Fence unindented | Marked |
|---|---|---:|---:|---:|
| 20 | 105–111 | yes | yes | yes |
| 20 | 121–157 | yes | yes | yes |
| 20 | 165–171 | yes | yes | yes |
| 20 | 179–219 | yes | yes | yes |
| 20 | 227–253 | yes | yes | yes |
| 20 | 261–275 | yes | yes | yes |
| 20 | 279–287 | yes | yes | yes |
| 21 | 374–376 | yes | yes | yes |
| 21 | 390–392 | yes | yes | yes |
| 22 | 476–486 | yes | yes | yes |
| 22 | 522–527 | yes | yes | yes |
| 22 | 546–556 | yes | yes | yes |
| 22 | 571–591 | yes | yes | yes |
| 22 | 599–612 | yes | yes | yes |
| 22 | 627–646 | yes | yes | yes |
| 22 | 654–662 | yes | yes | yes |
| 22 | 668–670 | yes | yes | yes |
| 22 | 688–690 | yes | yes | yes |
| 22 | 696–707 | yes | yes | yes |
| 22 | 711–722 | yes | yes | yes |
| 23 | 824–847 | yes | yes | yes |
| 23 | 855–857 | yes | yes | yes |
| 23 | 867–883 | yes | yes | yes |
| 23 | 901–905 | yes | yes | yes |
| 23 | 913–923 | yes | yes | yes |
| 24 | 1014–1022 | yes | yes | yes |
| 24 | 1038–1064 | yes | yes | yes |
| 24 | 1068–1074 | yes | yes | yes |
| 24 | 1082–1090 | yes | yes | yes |

**Code blocks marked:** 29 of 29.

## 4. Unsourced claims

The audit rule counts a claim as unsourced when the line names a page, doc, or guide but gives no inline URL. This is a strict rule; many named pages are real but their URLs were lost with the missing proof tables.

| Module | Unsourced lines |
|---|---:|
| 20 | 16 |
| 21 | 16 |
| 22 | 18 |
| 23 | 13 |
| 24 | 21 |
| 25 | 0 |
| **Total** | **84** |

Representative examples:

- Module 20, line 99: client-credentials property behavior is attributed to “the Client Credentials page” with no URL.
- Module 21, line 416: aggregation behavior is attributed to “the Aggregation in reporting page” with no URL.
- Module 22, line 496: Script Debugger limits are attributed to “the Script Debugger page” with no URL.
- Module 23, line 955: the 20-rows-per-page recommendation is attributed to a “ServiceNow Technical Support performance article” with no URL.
- Module 24, line 1010: namespace rules are attributed to “the Namespace identifier page” with no URL.

These are reported, not silently repaired. Adding guessed URLs would be worse than leaving the gap visible.

## 5. Pass/fail

| Criterion | Result |
|---|---|
| Under 5% of audited source quotes failing | **Pass** — 0 of 11 source-bearing quotes failing |
| Every code block marked | **Pass** — 29 of 29 |
| Whole-file audit including Modules 1–19 | **Not possible** — Modules 1–13 never existed in this session; Modules 14–19 are only `MISSING FROM HISTORY` stubs |
| Complete source coverage | **Fail** — 84 named-page claims have no inline URL because the proof tables were lost |

**Scoped result:** the audited Modules 20–25 meet the quote and code-block pass marks. The guide as a whole cannot be called fully sourced. This is a partial audit, not a clean bill of health for a complete 25-module file.

No commit was made during the audit.
