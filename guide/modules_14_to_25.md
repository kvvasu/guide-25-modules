# ServiceNow CAD Mastery Guide — Modules 14–25

## Scope note

This file covers Modules 14–25 only. Modules 1–13 were never generated in this session and are not included. They must not be inferred from this file.

Modules 14–19 are marked **MISSING FROM HISTORY** because their full module bodies and proof tables are not available in the visible conversation history. Modules 20–25 are assembled from the text generated in this session.

## Proof table — Modules 20–22

**MISSING FROM HISTORY**

The complete proof table for Modules 20–22 is not present in the visible conversation history. The partial rows that were visible do not cover every module in this group, so no reconstructed rows are included.

## Module 14. Advanced scripting

**MISSING FROM HISTORY**

The full Module 14 body, its code blocks, its error table, its proof table, and its design note are not present in the visible conversation history.

## Module 15. Forms and catalog depth

**MISSING FROM HISTORY**

The full Module 15 body, its code blocks, its error table, its proof table, and its design note are not present in the visible conversation history.

## Module 16. Service Portal widget

**MISSING FROM HISTORY**

Only a closing excerpt of Module 16 is visible. The full module body, its code blocks, its error table, its proof table, and its design note are not present.

## Module 17. ITSM processes

**MISSING FROM HISTORY**

Only the beginning of the Module 17 proof table and the verification text are visible. The full module body is not present.

## Module 18. Flow Designer depth

**MISSING FROM HISTORY**

Only the beginning of the Module 18 proof table and the verification text are visible. The full module body is not present.

## Module 19. Security depth

**MISSING FROM HISTORY**

Only the beginning of the Module 19 proof table and the verification text are visible. The full module body is not present.

## Module 20. Integration depth

**Level:** senior  
**Time:** 5–7 hours on a PDI, plus waiting for any external API or MID Server you involve.

**What you build:** a scoped integration lab that authenticates with OAuth, pages a Table API result set, retries a transient failure, accepts an idempotent inbound request, moves an attachment, runs a Transform Map with `onBefore`/`onAfter`, and documents where a MID Server is mandatory.

### PDI build

**Scope:** `x_r3_integration`  
**Application name:** R3 Integration Lab

#### Tables and fields

Create these in the scoped app.

`u_r3_sync_run`

| Field | Type | Notes |
|---|---|---|
| `u_number` | String (40) | Run label, for example `RUN-0001` |
| `u_source` | String (40) | Source system name |
| `u_status` | Choice | `new`, `running`, `done`, `failed` |
| `u_cursor` | String (255) | Last processed watermark or offset |
| `u_pages` | Integer | Pages fetched |
| `u_records` | Integer | Records processed |
| `u_last_error` | String (1000) | Last exception text |
| `u_idempotency_key` | String (100) | Unique index |

`u_r3_sync_item`

| Field | Type | Notes |
|---|---|---|
| `u_run` | Reference `u_r3_sync_run` | Parent run |
| `u_external_id` | String (100) | Unique index |
| `u_idempotency_key` | String (100) | Unique index; backstop for duplicate POSTs |
| `u_payload` | String (4000) | Raw request or row summary |
| `u_status` | Choice | `new`, `done`, `failed` |
| `u_target` | Reference `incident` | Created or matched incident |

Do not make `u_external_id` unique unless the source really guarantees uniqueness. If it does not, enforce uniqueness in the Scripted REST resource and record the conflict.

### Ordered steps

1. Create the scoped application and both tables. Add the fields above. Create a unique index on `u_r3_sync_item.u_idempotency_key`; this is the concurrency backstop, not the primary duplicate check.

2. Decide which OAuth flow you are teaching. For a user-context integration, use the authorization code flow: the user authenticates with the OAuth server, the client receives a code at its redirect URL, and exchanges that code for an access token and a refresh token. For server-to-server inbound calls, use client credentials. The two flows are not interchangeable, and client credentials normally has no interactive user consent step.

3. Enable inbound client credentials on the PDI. Create the system property `glide.oauth.inbound.client.credential.grant_type.enabled` and set it to `true`. The Client Credentials page states the property is false by default and that the OAuth Application User field must be added to the OAuth Entity form. If the field is not on your form, add it through the form layout; do not assume the field exists on every release.

4. Create an OAuth application registry record for the client. Use a redirect URL that you control even if you are testing client credentials, because the same record is often reused for authorization code later. Record the client ID and client secret in a password manager or your local untracked notes, not in the update set.

Token request (curl — written by me, not fetched; test it):

```
curl -s -X POST "https://<instance>.service-now.com/oauth_token.do" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>"
```

The response is the authority on whether a refresh token was issued. Do not build a refresh path just because the authorization code flow returns one. The token management page states that access tokens issued by the instance default to a 30-minute lifespan and refresh tokens to 100 days when the instance is the OAuth provider; both values are configurable.

5. Test the token before writing any integration code. A successful token response is not proof that the token can read the target table. Call the Table API with the token and check the HTTP status and the returned record count.

6. Build pagination against the Table API. Use `sysparm_limit` and `sysparm_offset`, and always add a stable order such as `ORDERBYsys_id`. Without an order, offset paging can skip or repeat records. The Table API page notes that `sysparm_limit` is applied before ACL evaluation, which is why a page can contain fewer rows than the limit even when more rows exist that you cannot read.

Background script (pagination — written by me, not fetched; test it):

```
var base = gs.getProperty('glide.servlet.uri').replace(/\/$/, '') + '/api/now/table/incident';
var limit = 200;
var offset = 0;
var total = 0;

while (true) {
    var rm = new sn_ws.RESTMessageV2();
    rm.setHttpMethod('GET');
    rm.setEndpoint(base
        + '?sysparm_limit=' + limit
        + '&sysparm_offset=' + offset
        + '&sysparm_query=' + encodeURIComponent('ORDERBYsys_id')
        + '&sysparm_fields=' + encodeURIComponent('sys_id,number,sys_updated_on'));
    rm.setRequestHeader('Accept', 'application/json');
    rm.setRequestHeader('Authorization', 'Bearer <access_token>');

    var resp = rm.execute();
    var status = resp.getStatusCode();
    if (status !== 200) {
        gs.error('Page failed: ' + status + ' ' + resp.getBody());
        break;
    }

    var body = JSON.parse(resp.getBody());
    var rows = body.result || [];
    if (rows.length === 0) break;

    total += rows.length;
    gs.info('Page offset=' + offset + ' rows=' + rows.length);

    if (rows.length < limit) break;
    offset += limit;
}

gs.info('Fetched ' + total + ' incident rows');
```

If the source changes while you page, replace offset paging with a watermark on `sys_updated_on` or keyset paging on `sys_id`. Offset paging is easy to read and easy to get wrong.

7. Build the retry path in Flow Designer, not in a tight server-side loop. Create a connection and credential alias, add a REST step, and attach a retry policy. The Retry policy page states that retry policies can be created for JDBC, REST, and SOAP steps and can use exponential backoff, fixed interval, or the `Retry-After` header.

Flow Designer action input (written by me, not fetched; test it):

```
Connection: R3 OAuth
Method: GET
Resource path: /api/now/table/incident
Query: sysparm_limit=200&sysparm_offset=0&sysparm_query=ORDERBYsys_id
Retry policy: R3 5xx and 429, exponential backoff, interval 5s, count 4
```

Do not retry a non-idempotent POST blindly. Retry reads, idempotent upserts, and requests that carry an idempotency key. A 429 is a retry signal, not a reason to increase concurrency; the rate-limit article notes that a full semaphore queue can return HTTP 429.

8. Build the idempotent inbound endpoint. Create a Scripted REST API named `R3 Inbound` and a resource `/incidents` with method POST. Read the `X-Idempotency-Key` header, look it up in `u_r3_sync_item`, return the original target on a duplicate, and create only when the key is new. The unique index catches the race where two requests pass the lookup at the same time.

Scripted REST API resource script (written by me, not fetched; test it):

```
(function process(request, response) {
    var key = request.getHeader('X-Idempotency-Key');
    if (!key) {
        response.setStatus(400);
        response.setBody({ error: 'X-Idempotency-Key required' });
        return;
    }

    var existing = new GlideRecord('u_r3_sync_item');
    existing.addQuery('u_idempotency_key', key);
    existing.setLimit(1);
    existing.query();
    if (existing.next()) {
        response.setStatus(200);
        response.setBody({
            sys_id: existing.u_target.toString(),
            duplicate: true
        });
        return;
    }

    var inc = new GlideRecord('incident');
    inc.initialize();
    inc.short_description = request.body.short_description || 'R3 inbound';
    inc.caller_id = request.body.caller_id || '';
    var targetId = inc.insert();

    var row = new GlideRecord('u_r3_sync_item');
    row.initialize();
    row.u_idempotency_key = key;
    row.u_external_id = request.body.external_id || key;
    row.u_payload = JSON.stringify(request.body).substring(0, 4000);
    row.u_status = 'done';
    row.u_target = targetId;
    row.insert();

    response.setStatus(201);
    response.setBody({ sys_id: targetId, duplicate: false });
})(request, response);
```

The lookup-then-insert sequence is not atomic. The unique index is what makes it safe. Catch the insert failure caused by a duplicate key and return the existing record; do not swallow the exception silently.

9. Move an attachment outbound. The Attachment API’s multipart upload requires the table name and record sys_id in the form body, and the file content must be in the final part. For a raw-body upload, use the `/api/now/attachment/file` endpoint with `table_name`, `table_sys_id`, and `file_name` as query parameters. The accepted Community solution for the 400 error switched to exactly that endpoint.

Background script (send attachment — written by me, not fetched; test it):

```
var sourceIncident = '<source_incident_sys_id>';
var targetIncident = '<target_incident_sys_id>';
var targetInstance = 'https://<target>.service-now.com';

var att = new GlideRecord('sys_attachment');
att.addQuery('table_name', 'incident');
att.addQuery('table_sys_id', sourceIncident);
att.query();

while (att.next()) {
    var rm = new sn_ws.RESTMessageV2();
    rm.setHttpMethod('POST');
    rm.setEndpoint(targetInstance
        + '/api/now/attachment/file?table_name=incident'
        + '&table_sys_id=' + encodeURIComponent(targetIncident)
        + '&file_name=' + encodeURIComponent(att.getValue('file_name')));
    rm.setRequestHeader('Accept', 'application/json');
    rm.setRequestHeader('Content-Type', att.getValue('content_type'));
    rm.setRequestBodyFromAttachment(att.getUniqueValue());

    var resp = rm.execute();
    gs.info('Attachment ' + att.getValue('file_name')
        + ' status=' + resp.getStatusCode()
        + ' body=' + resp.getBody());
}
```

Receiving an attachment is a read: query `sys_attachment` by `table_sys_id`, then download with `GET /api/now/attachment/{sys_id}/file`. Do not assume the attachment is present on the target just because the parent record was created.

10. Build the Transform Map. Load a CSV with `external_id`, `short_description`, and `caller_email` into an import set, then map it to `u_r3_sync_item` or `incident`. Use `onBefore` for validation and normalization, and `onAfter` for side effects that should happen only after the target exists.

Transform map `onBefore` script (written by me, not fetched; test it):

```
(function runTransformScript(source, map, log, target) {
    var externalId = source.u_external_id.toString().trim();
    if (!externalId) {
        ignore = true;
        log.warn('Rejected row ' + source.sys_id + ': missing external_id');
        return;
    }
    source.u_external_id = externalId;

    if (source.u_short_description.nil()) {
        source.u_short_description = 'Imported without description';
    }
})(source, map, log, target);
```

Transform map `onAfter` script (written by me, not fetched; test it):

```
(function runTransformScript(source, map, log, target) {
    var run = new GlideRecord('u_r3_sync_run');
    if (run.get(source.u_run)) {
        run.u_records = (parseInt(run.u_records, 10) || 0) + 1;
        run.update();
    }
})(source, map, log, target);
```

The transformation event page lists `onStart`, `onBefore`, `onAfter`, and `onComplete`, and warns that the available objects differ by event. In `onBefore`, `target` may not yet be the final persisted record; do not treat it as one.

11. Build the MID Server conceptual lab. Create a MID Server record named `R3-MID` and a JDBC data source pointing at a database you are allowed to query. Do not expect the data source to run on a stock PDI with no MID Server. The MID Server is a Java application that opens an outbound connection on port 443 and requires no inbound connection. It subscribes to AMB messages and polls the ECC queue, which is where the work waits until a MID Server is available.

This subtask is intentionally instance-state dependent. If you have no MID Server host, stop after creating the records and documenting the queue behavior; do not claim the JDBC or PowerShell probe ran. A PowerShell probe needs a MID Server with the appropriate PowerShell capability and an execution policy that allows the script. A JDBC data source needs a MID Server that can reach the database host and a credential with the right grants.

12. Record the run outcome in `u_r3_sync_run`. Set `u_status` to `done` only after the last page, the last attachment, and the last transform row have succeeded. On failure, store the HTTP status and a redacted error string in `u_last_error`; never store the bearer token or client secret.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `{"error":{"message":"Invalid file type: application/json","detail":null},"status":"failure"}` | Attachment POST returned 400 | https://www.servicenow.com/community/developer-forum/getting-400-message-when-sending-attachment-from-one-servicenow/m-p/3188394 | Yes. The accepted reply changed the endpoint to `/api/now/attachment/file?table_name=...&table_sys_id=...`; the header fix alone did not solve it. |
| `[ERROR CODE: -1] Socket closed` | Data Stream action failed around 13,000–14,000 records | https://www.servicenow.com/community/api-insights-forum/custom-data-stream-action-error-error-code-1-socket-closed-when/m-p/3434429 | No accepted solution. The reply attributes it to an HTTP transaction timeout, not a record-count limit. |
| `Unable to resolve the target record, coalesce field values are not present...` | Transform Map with an empty coalesce field | https://www.servicenow.com/community/developer-forum/transform-map-how-to-display-a-custom-error-when-a-coalesce/m-p/3425888 | No accepted solution. The thread reports the engine throws this before `onBefore`; validate in the staging table or accept the platform message. |
| `invalid_client`, `invalid_grant`, `invalid_scope` | OAuth token endpoint | https://www.servicenow.com/community/upgrades-and-patching-forum/how-to-make-successful-connection-oauth2-0-with-grant-type/m-p/3194565 | No accepted solution. The replies are troubleshooting suggestions, not a verified fix. |
| `org.apache.commons.httpclient.ConnectTimeoutException: The host did not accept the connection within timeout of 10000 ms` | MID Server agent log / ECC connectivity | https://www.servicenow.com/community/itom-forum/how-to-resolve-this-quot-mid-server-failed-to-upgrade-quot-issue/m-p/3153088 | No accepted solution. The thread shows the timeout but does not mark a fix. |

`invalid_client` usually means the client ID or secret is wrong; `invalid_grant` means the grant or credentials were rejected; `invalid_scope` means the requested scope is not allowed. Do not treat those three as interchangeable. Confirm the exact response body from your token endpoint before changing configuration.

## Sourced best practices

- Use the authorization code flow when a user must consent and revoke access; use client credentials for server-to-server inbound integrations. The authorization code flow page explicitly contrasts it with username/password and describes the code-for-token exchange.
- Treat access and refresh tokens as credentials. The token management page stores them in `oauth_credential`, defaults instance-issued access tokens to 30 minutes and refresh tokens to 100 days, and provides Revoke Access. Do not log token values.
- Page with a stable order and a limit. The Table API page documents `sysparm_limit` and `sysparm_offset`, and notes the limit is applied before ACL evaluation.
- Use Data Stream actions for responses larger than 10 MB or genuinely paginated APIs, and remember they require an Integration Hub subscription. The Data Stream page also states Data Stream retries have a 30-second maximum elapsed time and support only fixed intervals.
- Retry transient failures only. The Retry policy page supports exponential backoff, fixed interval, and `Retry-After`; it limits retry policies to JDBC, REST, and SOAP steps.
- Make writes idempotent with a unique key and a recorded outcome. The Community idempotency article recommends an `X-Idempotency-Key` header, a log table with a unique key, and returning the original response for a duplicate.
- Put file content last in a multipart upload. The Attachment API page states the file content must be in the final part and earlier parts should contain only metadata.
- Validate in `onBefore`, do side effects in `onAfter`. The transformation event page documents the event order and the different objects available at each stage.
- Keep MID Server traffic outbound. The connection prerequisites page states the MID Server communicates on port 443 and requires no inbound connections; the ECC queue page explains AMB notification and polling.
- Expect 429 under load. The rate-limit article describes per-user rules, semaphores, queuing, and HTTP 429 when the queue is full; back off rather than adding parallel calls.

## How to know it works

1. A token request returns an access token, and a Table API call with that token returns HTTP 200 and a known record.
2. The pagination script retrieves every incident exactly once. Compare the count with a Table API count or a `GlideAggregate` count on the same query. Then run it again and confirm no duplicates.
3. The idempotency endpoint returns `201` on the first POST and `200` with `duplicate: true` on the second POST with the same key. The incident count increases by one, not two.
4. The retry flow succeeds after you temporarily point it at an endpoint that returns 503 once and then 200. The flow log shows the retry attempt and the final success.
5. The attachment appears on the target incident with the same file name and a non-zero size. Download it and compare the bytes.
6. The Transform Map rejects a row with a blank external ID and increments the run count for a valid row. The import log shows the warning.
7. The MID Server record exists, but the JDBC data source remains queued or unavailable until a MID Server is actually connected. That is the expected result on a PDI without one.

## Refactor after

Replace offset pagination with watermark pagination on `sys_updated_on`, then replay the same dataset and prove there are no gaps or duplicates. Add a second idempotency test that fires two requests concurrently and verifies the unique index prevents a double insert. Move the attachment loop out of the business rule and into a subflow with a retry policy, then compare the failure behavior when the target returns 500.

## Design note

**What I built:** a scoped integration lab with OAuth, paging, retry, idempotent inbound writes, attachment transfer, transform scripts, and a MID Server boundary.

**One alternative:** use a single Data Stream action for both pagination and parsing, instead of a hand-written Table API loop. That is better for very large responses, but it hides the offset semantics and requires Integration Hub.

**Why I chose this one:** the hand-written loop makes the paging contract visible, and the Scripted REST endpoint makes the idempotency contract explicit. The MID Server section stays conceptual because a PDI without a MID Server cannot honestly demonstrate JDBC or PowerShell execution.

## Module 21. Reports and dashboards

**Level:** mid  
**Time:** 3–4 hours, plus one data-collection cycle for the indicator.

**What you build:** a Core UI report with conditions, grouping, and aggregation; a Platform Analytics dashboard shared to a group; and one automated indicator with a breakdown and a scorecard, with the reporting-versus-PA distinction made explicit.

### PDI build

No new tables are required. Use these existing tables and fields.

| Table | Fields used | Purpose |
|---|---|---|
| `incident` | `active`, `assignment_group`, `priority`, `opened_at`, `resolved_at`, `caller_id` | Report, dashboard, and indicator source |
| `task` | `sys_created_on`, `closed_at`, `assignment_group` | Cross-task reporting and PA source if you extend the exercise |
| `pa_indicator_sources` | `name`, `conditions`, `table` | Indicator source record |
| `pa_indicators` | `name`, `direction`, `aggregate`, `source` | Automated indicator |
| `pa_breakdowns` | `name`, `field`, `table` | Breakdown definition |
| `pa_scores` | `indicator`, `date`, `value`, `breakdown` | Collected scores |

The exact physical table names can vary by release and by whether the instance has migrated to Platform Analytics. Use the form labels and related lists as the source of truth; do not hardcode `pa_*` table names into scripts without checking the dictionary.

### Ordered steps

1. Decide which reporting surface you are on. The Create a report page notes that on net new Zurich instances and instances migrated to the Platform Analytics experience, Reporting functionality is replaced with Data Visualizations. If your PDI shows `All > Reports > Create New`, you are in the legacy Report Designer. If it shows Data Visualizations, use `All > Platform Analytics > Library > Data Visualizations > New` and translate the steps below. Do not mix the two surfaces in one screenshot set.

2. Create the report. Use the Incident table. Set conditions: `Active is true` and `Assignment group is not empty`. Group by `Assignment group`. Set the aggregation to `Count`. Choose a bar visualization. The Aggregation in reporting page lists count, count distinct, average, sum, minimum, maximum, and standard deviation, and states aggregation is not supported on histogram, pareto, single score, box, calendar, and list reports.

Report condition (encoded query — written by me, not fetched; test it):

```
active=true^assignment_groupISNOTEMPTY^ORDERBYassignment_group
```

3. Run the report and record the total. Then remove the group-by and compare the ungrouped count with a list view count under the same conditions. If the numbers differ, check ACLs, the report’s filter, and whether the report is using a database view rather than the base table.

4. Save the report as `R3 Open Incidents by Assignment Group`. Use the Sharing option to make it visible to `Everyone` or to a specific group. The Create a report page documents the Sharing options: Me, Everyone, and Groups and Users.

5. Create the dashboard. Go to `All > Platform Analytics > Library > Dashboards` and create `R3 Service Ops`. Add the report or its data visualization. Add a filter for `Assignment group` if you want the dashboard to be interactive. Keep the first version to one tab and fewer than ten elements; a dashboard that takes more than a few seconds to render is a debugging problem, not an executive view.

6. Share the dashboard. Open the dashboard, use `More actions > Share`, and add a group as viewer. The Share a dashboard page states that users with `admin` or `dashboard_admin` can share all dashboards, that data visualizations based on table data are automatically shared with the dashboard recipients, and that edit rights on the dashboard do not automatically grant edit rights on the underlying visualizations. It also states that users without rights to raw list data see a message in place of the visualization.

7. Create the Performance Analytics indicator source. Use the Incident table with conditions `Active is true` and `Priority is 1 - Critical`. Name it `R3 Open Critical Incidents Source`. The Create an automated indicator page requires a suitable indicator source and a Performance Analytics subscription; do not assume PA is available on every PDI.

Indicator source condition (written by me, not fetched; test it):

```
active=true^priority=1
```

8. Create the automated indicator. Use `Count` as the aggregate, set `Direction` to `Minimize`, and attach the indicator source. The Indicators page explains that automated indicators are based on indicator sources, formula indicators are calculated from other indicators, and manual indicators are primarily for testing. Set the direction deliberately: a cost or open-incident count should normally be minimized.

9. Add a breakdown by `Assignment group`. If you are using the newer Platform Analytics experience, use the breakdown definition and the indicator’s breakdown settings. The Scorecard breakdown widget page is explicitly a Core UI feature in maintenance mode; it shows trends by element for one breakdown applied to one indicator. Do not present it as the default modern experience.

10. Run the data collection job. Open the indicator and use the related links to run collection, or wait for the scheduled job. Then open KPI Details and confirm there is a score for today. If there are no scores, check the indicator source conditions, the collection job, and the indicator’s time zone. Do not “fix” missing scores by creating a manual score and calling it automated.

11. Optional: enable data snapshots only if you have RaptorDB Professional and a non-domain-separated instance. The Data snapshots page states that multiple breakdowns beyond the previous two-level limit require the Data Snapshots plugin (`com.snc.pa.mlb`), a Performance Analytics subscription, and RaptorDB Professional, and that data snapshots are not supported on domain-separated instances. Scripted indicators do not support multiple breakdowns. If those conditions are not met, stay with two breakdown levels and say so.

12. Record the difference between the report and the indicator. A report is a query run at view time. An indicator is a score collected on a schedule and stored over time. A report can show the current count; an indicator can show whether the count is improving. That distinction belongs in the dashboard’s description, not only in your head.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Widget cancelled - maximum execution time exceeded"` | A report on the `task` table rendered in a dashboard widget | https://www.servicenow.com/community/developer-forum/widget-cancelled-maximum-execution-time-exceeded-quot/m-p/3170494 | No accepted solution. The replies suggest splitting the report by task type and checking slow queries/explain plans. |
| No error string. Dashboard renders blank after deployment. | Platform Analytics dashboard moved to test | https://www.servicenow.com/community/platform-analytics-forum/dashboard-is-showing-empty-in-test-environment-post-deployment/m-p/3403341 | No accepted solution. The suggestions are to include all components in the update set and clear `par_dashboard_cache`. |
| No error string. Share dialog returns only 20 name matches. | Sharing a PA dashboard when many users have the same name | https://www.servicenow.com/community/sysadmin-forum/sharing-platform-analytics-dashboard-while-multiple-users-have/td-p/3415149 | Yes. The accepted reply points to `pa_dashboards_permissions` and having an admin set the user. |

The blank-dashboard thread is a deployment problem, not a rendering bug. Before changing the dashboard, verify that the dashboard record, tabs, elements, data visualizations, indicators, and breakdowns all traveled in the update set. Then clear the cache and re-login. Do not clear the cache as the first move; it hides a missing-component problem for one deploy and it comes back.

## Sourced best practices

- Use aggregation for the question you actually have. The Aggregation in reporting page lists count, count distinct, average, sum, min, max, and standard deviation, and explicitly excludes several visualization types from aggregation. If you need a single score, use a single score visualization or PA, not a list report with a sum column bolted on.
- Share the dashboard, not just the report. The Share a dashboard page separates dashboard rights from visualization rights and warns that edit rights on the dashboard do not grant edit rights on its visualizations.
- Share to groups or roles, not named individuals, when the audience is a team. The Share a dashboard page supports users, groups, and roles, and notes that sharing to `dashboard_user` makes the dashboard visible to authenticated users who can use dashboards.
- Measure direction. The Create an automated indicator page tells you to set `Maximize`, `Minimize`, or `None`; without a direction, the visualization cannot tell a good trend from a bad one.
- Use an indicator source, not a copied report condition. The Indicators page distinguishes automated, formula, and manual indicators; a manual indicator is for testing, not for production reporting.
- Do not promise unlimited breakdowns without the prerequisites. The Data snapshots page requires the plugin, the subscription, RaptorDB Professional, and a non-domain-separated instance, and excludes scripted indicators.
- Treat the Core UI scorecard widget as maintenance mode. The Scorecard breakdown widget page says the feature is in maintenance mode and is not supported on Platform Analytics dashboards or UI Builder pages except in limited compatibility mode.
- For new work, prefer Platform Analytics data visualizations over legacy Report Designer. The Create a report page itself notes the replacement on new and migrated Zurich instances.

## How to know it works

1. The report returns the same total as the list view under the same conditions, and the grouped counts sum to that total.
2. The dashboard renders for a user in the shared group and does not render row-level data for a user without table read access. That second result is correct, not a bug.
3. The indicator has a score for today, the score matches a fresh count from the indicator source conditions, and the direction is `Minimize`.
4. The breakdown shows at least two assignment groups with non-zero counts, or you can explain why the data has only one group.
5. A second user in the shared group can open the dashboard but cannot edit it. If they can edit, the share was granted as editor, not viewer.
6. If you enabled data snapshots, the indicator shows the expected number of breakdown levels and the collection job has no errors. If you did not enable them, the indicator stays within two breakdown levels.

## Refactor after

Replace the report’s hardcoded `priority=1` with a dashboard filter that lets the viewer choose priority. Then create a second indicator for `Priority 2` and a formula indicator that shows the ratio of critical to high incidents. Verify that the formula indicator does not collect its own scores but evaluates from the contributing indicators. Finally, move the dashboard’s slowest element to a scheduled report or a PA scorecard and measure the dashboard load time before and after.

## Design note

**What I built:** a grouped, aggregated incident report; a shared Platform Analytics dashboard; and one automated indicator with a breakdown and a scorecard, with the legacy-versus-modern surfaces called out.

**One alternative:** build everything in Platform Analytics data visualizations and skip the legacy Report Designer entirely. That is the better choice on a new Zurich instance, but it would hide the report-versus-indicator distinction that many administrators still need when they inherit a legacy instance.

**Why I chose this one:** the report shows the query-time behavior, the indicator shows the scheduled-score behavior, and the dashboard shows the sharing boundary. The three artifacts fail differently, so keeping all three makes the lesson testable.

## Module 22. Debugging and performance as a skill

**Level:** senior  
**Time:** 6–8 hours, including the five drills and one Instance Scan pass.

**What you build:** a bug lab with five seeded defects, a diagnostic pass using the Script Debugger, Session Log, transaction logs, slow queries, node logs, indexes, Debug Business Rule, and Instance Scan, and a before/after measurement for each fix.

### PDI build

**Scope:** `x_r3_debug`  
**Application name:** R3 Debug Lab

#### Tables and fields

`u_r3_bug_lab`

| Field | Type | Notes |
|---|---|---|
| `u_name` | String (100) | Record label |
| `u_stage` | Choice | `new`, `processing`, `processed` |
| `u_amount` | Decimal | Used by the N+1 and recursion drills |
| `u_owner` | Reference `sys_user` | Used by the N+1 drill |
| `u_external_id` | String (100) | Used by the hardcoded-sys_id and index drills |
| `u_group` | Reference `sys_user_group` | Used by the GlideAjax drill |
| `u_notes` | String (1000) | Diagnostic notes |

Create at least 2,000 records so the N+1 drill has a measurable cost. Use a background script to seed them if you do not want to import a CSV.

Background script (seed rows — written by me, not fetched; test it):

```
for (var i = 0; i < 2000; i++) {
    var gr = new GlideRecord('u_r3_bug_lab');
    gr.initialize();
    gr.u_name = 'R3 row ' + i;
    gr.u_stage = 'new';
    gr.u_amount = i;
    gr.u_external_id = 'EXT-' + i;
    gr.insert();
}
```

### Ordered steps

1. Create the table and fields. Add a list view that shows `u_name`, `u_stage`, `u_amount`, `u_owner`, `u_external_id`, and `u_group`. This is your baseline.

2. Seed the five defects below, one at a time. After each seed, reproduce the symptom before you fix it. Do not seed all five and then debug them together; you will not know which change caused which symptom.

3. For each drill, record a baseline: transaction response time, SQL count, business rule count, and the exact error text if there is one. Use `System Logs > Transactions` and open the transaction record. The Transaction logs page documents the response time, SQL count, business rule count, and business rule time fields.

4. Use the Script Debugger for server-side logic. Open `System Diagnostics > Script Debugger`, set a breakpoint in the Script Include or Business Rule, and trigger the code from an interactive form. The Script Debugger page states that it only pauses scripts in interactive transactions where `GlideSystem.isInteractive()` returns true, and that scripts run in interpreted mode while debugging; strict-mode scripts may fail under the debugger.

5. Use the Session Log for code that you cannot breakpoint, especially client-callable Script Includes and UI interactions. The Script Debugger page describes the Session Log for users with `log_debugger`, with default limits of 100 transactions and 10,000 messages, configurable through `glide.debugger.log.transaction.count` and `glide.debugger.logmessageslimit`.

6. Use transaction logs to find the expensive transaction before you read the code. Sort by response time, then inspect SQL count and business rule count. A high SQL count points to a query-in-a-loop or a missing index; a high business rule count points to recursion or a rule that runs on every update.

7. Use slow queries for database cost. Go to `System Diagnostics > Stats > Slow Queries`, open a query, and use `Explain Plan` and `Index Suggestion`. The Use a slow query log page states that the log aggregates similar queries and reports data where total execution time exceeds 5 seconds. The Slow queries and SQL debugging page lists common causes: too many OR conditions, a non-domain-path query method, and a missing index.

8. Use node logs when the syslog message is not enough. Go to `System Logs > Utilities > Node Log File Browser`, filter by session ID or message, and inspect the thread. The Log file browser page states that logs are available only for the node you are currently logged into, that archives are created every two days, and that archives are purged after 21 days.

9. Use indexes deliberately. Create an index on `u_r3_bug_lab.u_external_id` through `System Definition > Tables > [table] > Database Indexes > New`. The Create a table index page warns that field order matters, that a large table is generally around 1 TB, and that index creation on a large table should be done off-peak. Do not index every field; an index has a write cost.

10. Use Debug Business Rule (Details) to see conditions and values. Go to `System Diagnostics > Session Debug > Debug Business Rule (Details)`, reproduce the update, and read the output. The Debugging business rules page also recommends the System Dictionary, System Log, and alert messages as complementary tools.

11. Run Instance Scan. Open Instance Scan from the filter navigator and run a full scan, then a point scan on the `x_r3_debug` application. The Explore page states that a full scan runs all active checks, a point scan targets a specific record, update set, or application, and a test scan tests one check. The `scanuser` role is required to create checks, execute scans, schedule scans, and analyze results. Instance Scan has limited domain separation support; findings are domain-separated based on the source record.

12. Fix each drill and re-measure. Do not close a drill because the error disappeared; close it when the baseline metric improved or the correct behavior is proven.

## The five seeded bug drills

### Drill 1 — Recursive Business Rule

**Symptom:** updating a `u_r3_bug_lab` record never completes, the transaction log shows an unusually high business rule count, and the browser or system log reports a stack overflow. The exact server-side wrapper text varies by release; `RangeError: Maximum call stack size exceeded` is the expected JavaScript error, but it is reconstructed here, not fetched from a Community thread. Confirm it by running the rule with Debug Business Rule (Details) and reading the transaction error.

**Business Rule (seeded bug 1 — written by me, not fetched; test it):**

```
(function executeRule(current, previous /*null when async*/) {
    current.u_stage = 'processed';
    current.update(); // seeded bug: re-enters the same before-update rule
})(current, previous);
```

**How to diagnose it:**

1. Disable the rule and confirm the update completes.
2. Re-enable it with `System Diagnostics > Session Debug > Debug Business Rule (Details)` and watch the same rule appear repeatedly.
3. Check the transaction record’s business rule count; it should be far above the normal count for one update.
4. Search the script for `current.update()`, `current.insert()`, `current.deleteRecord()`, and `GlideRecord.update()` calls inside `before` rules.

**Cause:** a `before` Business Rule calls `current.update()`, which starts another update and re-enters the same rule. The rule’s condition does not stop it because `u_stage` is set before the recursive update.

**Fix:** remove `current.update()` from a `before` rule. Set the field directly and let the current transaction save it. If the side effect must run after the record exists, use an `after` rule with a condition that checks `current.u_stage.changes()` and a guard that prevents re-entry.

### Drill 2 — Query in a loop (N+1 problem)

**Symptom:** the list or script is slow, the transaction log shows a high SQL count, and the time grows roughly linearly with the number of rows. There may be no error until the transaction quota is exceeded. `Transaction cancelled: maximum execution time exceeded` is an expected outcome for a large enough dataset, but it is reconstructed here; the fetched 2025 thread shows the same class of report failure as `Widget cancelled - maximum execution time exceeded"`.

**Business Rule (seeded bug 2 — written by me, not fetched; test it):**

```
(function executeRule(current, previous /*null when async*/) {
    var lab = new GlideRecord('u_r3_bug_lab');
    lab.query();
    while (lab.next()) {
        var owner = new GlideRecord('sys_user');
        owner.get(lab.u_owner); // seeded bug: one query per row
        gs.info(lab.u_name + ' owner=' + owner.name);
    }
})(current, previous);
```

**How to diagnose it:**

1. Open the transaction in `System Logs > Transactions` and compare SQL count with the number of rows processed.
2. Enable `Debug SQL` or `Debug SQL (Detailed)` under `System Diagnostics > Session Debug` and look for the same query shape repeated with different values.
3. Use `System Diagnostics > Stats > Slow Queries` to see whether the query pattern is aggregated and how much total time it consumes.
4. Use the Script Debugger to pause inside the loop and inspect how many times the inner `get()` runs.

**Cause:** the code executes one `sys_user` query per `u_r3_bug_lab` row. The database work is N+1: one query for the list plus one per row.

**Fix:** load the users once into a map, or use a single query with an `IN` clause. A `GlideAggregate` is appropriate when you only need counts, not display names.

Script Include (fixed pattern — written by me, not fetched; test it):

```
var R3OwnerMap = Class.create();
R3OwnerMap.prototype = {
    initialize: function() {},

    getNames: function(ids) {
        var names = {};
        if (!ids || ids.length === 0) return names;

        var gr = new GlideRecord('sys_user');
        gr.addQuery('sys_id', 'IN', ids.join(','));
        gr.query();
        while (gr.next()) {
            names[gr.getUniqueValue()] = gr.getValue('name');
        }
        return names;
    },

    type: 'R3OwnerMap'
};
```

### Drill 3 — Hardcoded sys_id

**Symptom:** the assignment group or reference field is correct in development and empty in test or production. There is usually no error; the field simply resolves to nothing. On a clone, the sys_id from the source instance does not exist on the target.

**Script Include (seeded bug 3 — written by me, not fetched; test it):**

```
var R3Group = Class.create();
R3Group.prototype = {
    initialize: function() {},

    getDefaultGroup: function() {
        var gr = new GlideRecord('sys_user_group');
        gr.get('d9062c2f3b1f1e1066e3342a85e45a1f'); // seeded bug: dev sys_id
        return gr.getUniqueValue();
    },

    type: 'R3Group'
};
```

**How to diagnose it:**

1. Compare the hardcoded value with `sys_user_group.sys_id` on the target instance.
2. Search the application for 32-character hex strings and for `sys_id` literals.
3. Reproduce with the same user and record on both instances; if the behavior differs only by instance, suspect a hardcoded identifier.
4. Check the update set XML for the literal value; a hardcoded sys_id travels with the update set and remains wrong.

**Cause:** a sys_id is instance-specific. It is not a stable business key.

**Fix:** query by a business key such as `name`, or store the target in a system property or a custom configuration table that is part of the application. If you must use a sys_id for a one-time migration, resolve it at run time and fail loudly when it is missing.

Script Include (fixed pattern — written by me, not fetched; test it):

```
var R3Group = Class.create();
R3Group.prototype = {
    initialize: function() {},

    getDefaultGroup: function() {
        var gr = new GlideRecord('sys_user_group');
        gr.addQuery('name', 'Service Desk');
        gr.setLimit(1);
        gr.query();
        if (!gr.next()) {
            gs.error('R3Group: Service Desk group not found');
            return '';
        }
        return gr.getUniqueValue();
    },

    type: 'R3Group'
};
```

### Drill 4 — ACL blocking an integration user

**Symptom:** the integration user gets HTTP 403 on the Table API, while an admin gets 200. The response body is:

**Fetched error body (quoted from the June 2025 thread; not written by me):**

```
{
  "error": {
    "message": "User Not Authorized",
    "detail": "Failed API level ACL Validation"
  },
  "status": "failure"
}
```

This exact body is from the June 2025 thread. The thread has no accepted solution, so treat the reply as a diagnostic lead, not a proven fix.

**ACL script (seeded bug 4 — written by me, not fetched; test it):**

```
answer = gs.hasRole('sn_r3_user'); // seeded bug: integration user lacks this role
```

**How to diagnose it:**

1. Impersonate the integration user and repeat the Table API call. If it still fails, the issue is the user’s roles or the ACL, not the caller’s code.
2. Open `System Diagnostics > Session Debug > Debug Security` and repeat the request. The Debugging sessions page lists Debug Security as the option that displays access-control messages.
3. Use the Access Analyzer on the table and field. Check the read ACL, the role requirement, the condition, and the `Admin Overrides` setting.
4. Confirm that the table is accessible via web services. The June 2025 thread’s later reply points to the `sys_db_object` application access setting “Allow access to this table via web services”.
5. Check whether a field-level ACL is denying a field that the query selects or filters on. A table-level allow does not override a field-level deny.

**Cause:** the ACL requires a role the integration user does not have, or the table is not exposed to web services. Disabling all ACLs to “test” is not a diagnosis; it removes the evidence.

**Fix:** grant the least privilege that satisfies the integration. If the integration only reads incidents, give read access to the required fields and records, not `admin`. If the ACL is correct and the user is wrong, fix the user’s role assignment.

### Drill 5 — Wrong GlideAjax name

**Symptom:** the client script throws a `ReferenceError`, or the callback never receives a useful answer. The exact fetched error from the March 2025 thread is:

```
onChange script error: ReferenceError: getAssignmentGroupsByRegion is not defined function () { [native code] }
```

That thread’s accepted solution shows the correct pattern: construct `GlideAjax` with the Script Include name, set `sysparm_name` to the method name, pass parameters with `addParam`, and use `getXMLAnswer`.

**Client Script (seeded bug 5 — written by me, not fetched; test it):**

```
function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || !newValue) return;

    var ga = new GlideAjax('R3Utils');
    ga.addParam('sysparm_name', 'getGroups');
    ga.addParam('sysparm_region', newValue);
    ga.getXMLAnswer(function(answer) {
        g_form.setValue('u_group', answer);
    });
}
```

**Script Include (seeded bug 5 — written by me, not fetched; test it):**

```
var R3UtilsSN = Class.create();
R3UtilsSN.prototype = {
    initialize: function() {},

    getAssignmentGroups: function() {
        return 'sys_idIN' + '';
    },

    type: 'R3UtilsSN'
};
```

**How to diagnose it:**

1. Open the browser developer console and read the first error, not the last one.
2. Check the Script Include’s exact `Name` field. `R3Utils` and `R3UtilsSN` are different classes.
3. Check the method name in `sysparm_name`. It must match the method on the prototype, not the display label.
4. Check `Client callable` on the Script Include. A server-only Script Include cannot be called from `GlideAjax`.
5. Add a `gs.info` at the top of the server method and check the Session Log. If the method is never entered, the name is wrong; if it is entered, the parameter or return value is wrong.

**Cause:** the client asks for class `R3Utils` and method `getGroups`, but the Script Include is `R3UtilsSN` and the method is `getAssignmentGroups`.

**Fix:** make the names match, make the Script Include client callable, and return a value the client can use. Do not return a `GlideRecord`; return a string, number, or JSON string.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Invalid query detected, please check logs for details [Unknown field 1b615096db43f7001e317722ba96190a in table sc_item_option]` | Syslog warning; field does not exist on the table | https://www.servicenow.com/community/developer-forum/syslog-showing-many-invalid-query-detected/td-p/3145556 | Yes. The accepted reply says to search the node log for `Invalid query detected` and check `glide.invalid_query.returns_no_rows`. |
| `Widget cancelled - maximum execution time exceeded"` | Report widget on the `task` table | https://www.servicenow.com/community/developer-forum/widget-cancelled-maximum-execution-time-exceeded-quot/m-p/3170494 | No accepted solution. The replies suggest splitting the report and checking slow queries/explain plans. |
| `onChange script error: ReferenceError: getAssignmentGroupsByRegion is not defined function () { [native code] }` | Client script calls a method that is not defined on the client | https://www.servicenow.com/community/csm-forum/how-to-clear-the-field-value-based-on-another-fiel/m-p/3219367 | Yes. The accepted reply replaces the direct function call with `GlideAjax`, `sysparm_name`, and `getXMLAnswer`. |
| `"message": "User Not Authorized", "detail": "Failed API level ACL Validation"` | Table API 403 | https://www.servicenow.com/community/developer-forum/rest-api-explorer-returns-403-forbidden-for-admin-role-on-table/m-p/3280304 | No accepted solution. A later reply points to the `sys_db_object` web-service access setting. |
| `RangeError: Maximum call stack size exceeded` | Recursive Business Rule (drill 1) | No fetched thread | Expected/reconstructed. Confirm with Debug Business Rule (Details) and the transaction error; the exact wrapper varies by release. |
| `Transaction cancelled: maximum execution time exceeded` | N+1 or long-running transaction (drill 2) | No fetched 2025 thread | Expected/reconstructed. Confirm by measuring the transaction’s SQL count and response time before the quota is reached. |

The invalid-query thread’s accepted reply is useful because it does not pretend the syslog message is the whole story. The detail is in the node log. The message includes an “unknown field” that is often a field that was deleted, renamed, or referenced with the wrong table prefix.

## Sourced best practices

- Use the debugger that matches the failure. The Script Debugger page covers breakpoints, step-through, expression evaluation, and call-stack inspection for server-side interactive transactions; the Session Log covers logs for business rules, Script Includes, and custom UIs.
- Do not expect Script Debugger to breakpoint a `Scripts - Background` script directly. The Script Debugger page states it does not directly support breakpoints in the Background module, but it does support scripts called by them.
- Read the transaction before reading the code. The Transaction logs page exposes SQL count, business rule count, business rule time, response time, and URL. That is enough to separate a database problem from a rule problem.
- Treat slow-query data as aggregated evidence, not a single event. The Use a slow query log page says similar queries are aggregated and only queries whose total execution time exceeds 5 seconds are reported.
- Use Explain Plan before adding an index. The Slow queries and SQL debugging page recommends Explain Plan to see what the database is doing; the Create a table index page warns that index field order matters and that large-table index creation should run off-peak.
- Check the node log when the syslog line is truncated. The Log file browser page documents the Node Log File Browser, current-node limitation, two-day archives, and 21-day retention.
- Use Debug Business Rule (Details), not just `gs.info`. The Debugging business rules page lists condition and value inspection as the reason to use the details module.
- Run Instance Scan as a pre- and post-upgrade habit. The Instance Scan landing page describes it as part of development operations, release management, and pre/post-upgrade work; the Explore page lists full, point, and test scans and the `scanuser` role.
- Do not over-index. The Create a table index page explicitly says constructing an effective index requires database architecture knowledge and recommends consulting a DBA if you do not have it.

## How to know it works

1. Drill 1: the update completes once, the business rule count is normal, and `u_stage` is `processed`. There is no recursive second update.
2. Drill 2: the SQL count for the same 2,000-row workload drops substantially, and the output still contains the correct owner names. Record the before and after counts; do not claim a speedup without them.
3. Drill 3: the same Script Include returns the correct group on two different instances, or returns an explicit error when the group is missing. No 32-character literal remains in the application.
4. Drill 4: the integration user gets 403 before the fix and 200 after the least-privilege change. The admin still gets 200. Debug Security shows the ACL that allowed or denied the request.
5. Drill 5: the client script receives the expected group sys_id, the Session Log shows the server method was entered, and the browser console has no `ReferenceError`.
6. Instance Scan completes and you can explain every high-severity finding you chose not to fix. “Accepted risk” is a decision, not a silent dismissal.
7. The slow-query record for the N+1 pattern disappears or its total time drops after the fix. If it does not, the fix addressed the wrong query.

## Refactor after

Turn each drill into an ATF test or a repeatable check: a test that updates the lab record and asserts it completes once; a test that asserts the owner map returns the right count; a test that asserts the group resolves on a cloned instance; a test that calls the Table API as the integration user and asserts 200; and a test that calls the GlideAjax method and asserts a non-empty answer. Then add a transaction budget: fail the build if the lab’s list transaction exceeds a SQL count or response-time threshold. A bug lab without a regression test becomes a demo, not a skill.

## Design note

**What I built:** one bug-lab table, five seeded defects, and a diagnostic path that uses the same tools an administrator would use on a real incident.

**One alternative:** build five separate scoped applications, one per defect. That isolates the update sets and makes each drill independently deployable, but it multiplies the setup and hides the fact that real instances combine these failures.

**Why I chose this one:** a single table keeps the baseline comparable. The same transaction log, the same slow-query view, and the same Instance Scan can be used before and after each fix. The trade-off is that the defects can interact if you seed them all at once, so the instructions require one at a time.

## Module 23. Scale test

**Level:** senior  
**Time:** 4–6 hours, including seeding and measurement. The seed runs may take longer than the analysis.

**What you build:** a 50,000–100,000-row scoped table, a baseline measurement of list, report, and query performance, two indexes, rewritten queries, and a measurement sheet that proves or disproves the fix.

### PDI build

**Scope:** `x_r3_scale`  
**Application name:** R3 Scale Lab

#### Tables and fields

`x_r3_scale_case`

| Field | Type | Notes |
|---|---|---|
| `u_number` | String (40) | `SC-000001` … |
| `u_short_description` | String (200) | Text |
| `u_state` | Choice | `new`, `in_progress`, `closed` |
| `u_priority` | Choice | `1`, `2`, `3`, `4` |
| `u_assignment_group` | Reference `sys_user_group` | Grouped report field |
| `u_caller` | Reference `sys_user` | Dot-walk target |
| `u_external_id` | String (100) | Equality filter and index candidate |
| `u_category` | String (80) | Prefix filter candidate |
| `u_amount` | Decimal | Aggregation field |
| `u_created_on` | Date/Time | Range filter candidate |

Create the table before seeding. Add a list view with the fields above. Do not add a unique index yet; you want the baseline to be honestly slow.

### Ordered steps

1. Create the scoped application and table. Add the fields. Add the list view. Record the empty-table response time only as a sanity check, not as a baseline.

2. Seed 50,000 rows first. The Scripts - Background module can run arbitrary server-side JavaScript, but it is still a transaction and it is not a production tool. The page warns that running free-form JavaScript can cause system disruption or data loss and says not to run it from production. Use a sub-production instance.

3. Seed in batches, not one 100,000-iteration loop. A single background script may hit the transaction quota or the page’s own cancellation limit. The page shows a `Cancel after 4 hours` checkbox, but that is not a promise that a 100,000-row insert loop will complete; it is a safety valve. Seed 2,000–5,000 rows per run and track the next index in a system property.

Background script (batched seed — written by me, not fetched; test it):

```
var TOTAL = 50000; // change to 100000 for the larger run
var BATCH = 2000;
var next = parseInt(gs.getProperty('x_r3_scale.next'), 10);
if (isNaN(next)) next = 0;

var end = Math.min(next + BATCH, TOTAL);
for (var i = next; i < end; i++) {
    var gr = new GlideRecord('x_r3_scale_case');
    gr.initialize();
    gr.u_number = 'SC-' + ('000000' + i).slice(-6);
    gr.u_short_description = 'Scale row ' + i;
    gr.u_state = (i % 5 === 0) ? 'closed' : (i % 3 === 0 ? 'in_progress' : 'new');
    gr.u_priority = String((i % 4) + 1);
    gr.u_external_id = 'EXT-' + ('000000' + i).slice(-6);
    gr.u_category = (i % 4 === 0) ? 'Network' : (i % 4 === 1 ? 'Database' : (i % 4 === 2 ? 'Application' : 'Security'));
    gr.u_amount = i % 1000;
    gr.u_created_on = new GlideDateTime();
    gr.insert();
}

gs.setProperty('x_r3_scale.next', String(end));
gs.info('Seeded rows ' + next + ' through ' + (end - 1) + ' of ' + TOTAL);
```

Run the script repeatedly until `x_r3_scale.next` reaches `TOTAL`. Create the property first if `gs.setProperty` does not create it in your release. Do not run 25 copies concurrently; that is a self-inflicted scale test.

4. Build the baseline list. Open the list with an equality filter on the unindexed field:

**List URL (written by me, not fetched; test it):**

```
x_r3_scale_case_list.do?sysparm_query=u_external_id%3DEXT-049999
```

Record the response time and SQL count from `System Logs > Transactions`. The Transaction logs page documents response time, SQL count, business rule count, and business rule time. Use the same user, the same filter, and the same rowcount for every run.

5. Build the baseline report. Create a report on `x_r3_scale_case` with conditions `u_state is new` and `u_priority is 1`, grouped by `u_assignment_group`, aggregation `Count`, bar chart. Run it and record the response time. If the report is a legacy Report Designer report, use `All > Reports > Create New`; if your instance has migrated to Platform Analytics, use the equivalent Data Visualization and record that surface instead. Do not compare a report time to a list time and call it a regression.

6. Build the baseline query benchmark. Run the same `GlideRecord` query five times and keep the median. The script below measures wall time, not SQL count. Do not claim a SQL-count improvement from this script alone; read the transaction log or Debug SQL for that.

Background script (query benchmark — written by me, not fetched; test it):

```
var times = [];
for (var run = 0; run < 5; run++) {
    var t0 = new Date().getTime();
    var gr = new GlideRecord('x_r3_scale_case');
    gr.addQuery('u_category', 'STARTSWITH', 'Network');
    gr.addQuery('u_state', 'new');
    gr.setLimit(200);
    gr.query();
    var count = 0;
    while (gr.next()) count++;
    times.push(new Date().getTime() - t0);
    gs.info('run=' + run + ' ms=' + times[run] + ' rows=' + count);
}
times.sort(function(a, b) { return a - b; });
gs.info('median_ms=' + times[2]);
```

Record the result count as well as the time. A faster query that returns the wrong rows is not a fix.

7. Identify what is slow. Use these three lenses:

| Surface | What to inspect | Typical cause |
|---|---|---|
| List view | Response time, SQL count, rowcount preference, grouped-list behavior | High rowcount, unindexed filter, dot-walk display fields, ACL evaluation per row |
| Report | Response time and aggregation type | Group by high-cardinality field, unindexed filter, large table scan, too many groups |
| Query | Wall time, Debug SQL, Explain Plan | `CONTAINS`, leading wildcard, `OR`, `!=`, `ORDERBY` on unindexed field, query in a loop |

The Zurich grouped-list article is relevant here: grouped lists are capped at 20 groups per page, and the `Show` setting controls rows within a group, not the number of groups. Raising `glide.db.max.aggregate.size` changes the cap globally and can hurt performance. Treat that property as a last resort, not a tuning default.

8. Add indexes. Start with the equality filter and the common report filter. The Create a table index page warns that field order matters and that large-table index creation should run off-peak. A composite index on `(u_state, u_priority)` is useful when both are equality predicates; an index on `(u_priority, u_state)` is not the same index.

Index plan (written by me, not fetched; test it):

```
Index 1: u_external_id
Index 2: u_state, u_priority
Index 3: u_category
```

Do not add all three at once if you want to attribute the improvement. Add index 1, measure. Add index 2, measure. Add index 3 only if the `STARTSWITH` query is still slow. If you add all three, you can prove the final state but not which index mattered.

9. Rewrite the query. Replace a leading-wildcard `CONTAINS` with `STARTSWITH` when the business question allows it. Add `setLimit`. Select only the fields you need. Avoid `get()` inside a loop. Use `GlideAggregate` when you need a count rather than rows.

Background script (aggregate instead of row loop — written by me, not fetched; test it):

```
var ga = new GlideAggregate('x_r3_scale_case');
ga.addQuery('u_state', 'new');
ga.addQuery('u_priority', '1');
ga.addAggregate('COUNT');
ga.groupBy('u_assignment_group');
ga.query();
while (ga.next()) {
    gs.info(ga.u_assignment_group.getDisplayValue() + ' = ' + ga.getAggregate('COUNT'));
}
```

10. Re-measure the list, report, and query. Use the same user, same filter, same rowcount, and same time window. Discard the first run after a code or index change if the cache is cold; report the median of five runs. Open the transaction record and compare response time and SQL count. Then open `System Diagnostics > Stats > Slow Queries`, find the query pattern, and use `Explain Plan` and `Index Suggestion`. The Use a slow query log page says the log reports similar queries whose total execution time exceeds 5 seconds.

11. Fill in the measurement sheet. If you did not run it, leave it blank. This guide did not run it.

| Artifact | Baseline | After | Evidence |
|---|---|---|---|
| List: `u_external_id=EXT-049999` | not measured | not measured | Transaction response time and SQL count |
| Report: state=new, priority=1, group by assignment group | not measured | not measured | Report run time and row count |
| Query: category STARTSWITH Network, state=new, limit 200 | not measured | not measured | Median wall time and result count |
| Explain Plan | not measured | not measured | Index used vs full scan |
| Slow Queries record | not measured | not measured | Total execution time and query pattern |

12. If the list is still slow after the index, check the display fields. A list that dot-walks `caller_id.name` and `assignment_group.name` for every row does extra work that an index on the filter field cannot fix. Reduce the displayed columns, reduce the rowcount, or accept the cost. Do not index a display-only field and expect the join to disappear.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Widget cancelled - maximum execution time exceeded"` | A report on the `task` table rendered in a dashboard widget | https://www.servicenow.com/community/developer-forum/widget-cancelled-maximum-execution-time-exceeded-quot/m-p/3170494 | No accepted solution. The replies suggest splitting the report and checking slow queries/explain plans. |
| `Invalid query detected, please check logs for details [Unknown field 1b615096db43f7001e317722ba96190a in table sc_item_option]` | A query references a field that does not exist on the table | https://www.servicenow.com/community/developer-forum/syslog-showing-many-invalid-query-detected/td-p/3145556 | Yes. The accepted reply says to search the node log for the full detail and check `glide.invalid_query.returns_no_rows`. |
| `Transaction cancelled: maximum execution time exceeded` | A long-running background/load transaction | No fetched 2025 thread | Expected/reconstructed. Confirm on the PDI by reading the transaction record and the quota message; the exact wrapper varies by release. |

The `Invalid query detected` message is not a performance error by itself. It becomes one when the query planner is asked to resolve a field that does not exist; the node log carries the detail that the syslog line truncates.

## Sourced best practices

- Do not run free-form background scripts on production. The Scripts - Background page states the risk and offers a rollback context, a sandbox option, and a cancellation limit. Use a sub-production instance for the seed.
- Measure the transaction, not the stopwatch in your head. The Transaction logs page exposes response time, SQL count, business rule count, and business rule time. Compare like with like.
- Use slow-query data before adding indexes. The Use a slow query log page aggregates similar queries and reports those whose total execution time exceeds 5 seconds; the Slow queries and SQL debugging page recommends `Explain Plan`.
- Respect index order and maintenance cost. The Create a table index page says field order affects how the index works, that a large table is generally around 1 TB, and that index creation on a large table should run off-peak.
- Keep rowcount low. The ServiceNow Technical Support performance article recommends 20 rows per page and warns that a high rowcount multiplies security and rendering work per row, especially with group-by.
- Know the Zurich grouped-list cap. The Community article states that grouped lists render 20 groups per page in Zurich and that `glide.db.max.aggregate.size` can raise the cap globally, with a performance cost.
- Avoid leading-wildcard `CONTAINS` when the question allows `STARTSWITH`. A B-tree index cannot efficiently serve a leading wildcard; rewriting the query is often more effective than adding another index.
- Do not index everything. Every index adds write cost and storage. Add the index that matches the filter and the sort order, then prove it with Explain Plan.

## How to know it works

1. The list returns the same row for `EXT-049999` before and after the change.
2. The median response time for five runs is lower after the index, and the transaction’s SQL count is not higher.
3. `Explain Plan` shows the index is used for the filter, not a full table scan.
4. The report returns the same grouped counts before and after; the runtime is lower or the report is moved to a scheduled/PA surface with a documented reason.
5. The query benchmark returns the same row count and a lower median wall time.
6. The slow-query record for the pattern is gone or its total time is lower. If it is unchanged, the index did not address the actual predicate.
7. You can state the measured numbers, the user, the filter, the rowcount, and the time window. If you cannot, you have an anecdote, not a result.

## Refactor after

Archive closed rows into a separate table or add a filtered index strategy, then re-run the same three measurements. Replace the report with a `GlideAggregate`-backed scheduled snapshot if the business only needs a daily count. Add a second composite index for the report’s actual filter order and prove with Explain Plan that the planner chooses it. Finally, test the same list with rowcount 20 and rowcount 100 and record the difference; that is often larger than the index gain.

## Design note

**What I built:** a 50,000–100,000-row lab, three baseline surfaces, three candidate indexes, and a measurement protocol that separates list, report, and query behavior.

**One alternative:** use an Import Set to load the rows instead of a background script. That is faster and more realistic for bulk data, but it hides the transaction-quota behavior that the exercise is meant to teach.

**Why I chose this one:** the batched background script makes the quota and rollback constraints visible, and the three-surface measurement prevents the common mistake of assuming the index fixed everything when only one of the three surfaces improved.

## Module 24. Dev lifecycle

**Level:** senior  
**Time:** 5–7 hours, plus time for a clone and an app-repository round trip.

**What you build:** a scoped app under Git source control, a second copy installed from the application repository, an update set for a base-system change, a skipped-change review after an upgrade preview, a clone with preservers/exclusions, and a naming/upgrade-safety checklist.

### PDI build

**Scope:** `x_r3_lifecycle`  
**Application name:** R3 Lifecycle Lab

#### Tables and fields

`x_r3_lifecycle_item` extends `task`

| Field | Type | Notes |
|---|---|---|
| `u_external_id` | String (100) | Business key, not a sys_id |
| `u_stage` | Choice | `draft`, `active`, `retired` |
| `u_owner_group` | Reference `sys_user_group` | Assignment |
| `u_release` | String (40) | Release label |
| `u_notes` | String (1000) | Change notes |

Use a table that extends `task` so you inherit `number`, `state`, `assigned_to`, and the task behaviors. If your app design does not need task semantics, use a standalone table and say why; do not extend `task` just to get a number.

### Ordered steps

1. Write the naming convention before creating anything. The Namespace identifier page states that scoped applications start with `x_`, that the instance customer prefix is two to five characters, and that the application ID can be up to 40 characters but is truncated for the namespace if it is longer than 18. The SDK CLI page also states that the scope name must begin with `x_<prefix>` and be 18 characters or fewer.

Naming convention (written by me, not fetched; test it):

```
Scope:            x_r3_lifecycle
Table:            x_r3_lifecycle_item
Fields:           u_external_id, u_stage, u_owner_group
Script Include:   R3LifecycleUtils
Update set:       R3 Lifecycle - <ticket>
Branch:           feature/r3-lifecycle-item
Commit:           R3-123 add lifecycle item table
```

The scope cannot be renamed later without recreating artifacts. Treat the scope name as an architectural decision, not a label.

2. Create the scoped application in ServiceNow Studio. The ServiceNow Studio page describes it as the unified environment for app files, metadata, and packaging. Create the table and fields. Add a module and a list view. Do not customize an out-of-the-box table for the lab; keep the app self-contained.

3. Create a dedicated private Git repository, for example `r3-lifecycle`. Generate a personal access token with read/write access to that repository only. The Link an app to source control page recommends a dedicated repository, MFA, a scoped token, network access from the non-production instance, and an email address on each developer’s `sys_user` record. Do not reuse a personal token across applications.

4. Link the app to source control. In ServiceNow Studio, open the app, go to `App details > Source control > Link to source control`, and enter the HTTPS URL, branch `main`, credential, and default email. The Link an app to source control page states that the integration does not support linking an application on a production instance; install on production from the application repository, an update set, or Studio. If you are on a PDI and the option is missing, the June 2025 Community thread’s accepted workaround is to publish the app to an update set from the `sys_app` record; that is a workaround, not source control.

5. Clone the repository into the ServiceNow IDE or use the SDK locally. The SDK page states that the SDK develops scoped applications in Visual Studio Code Desktop and uploads them to an instance, and that it does not support Global scope or global applications on Zurich. The Fluent page states that Fluent is a TypeScript-based DSL for `sys_metadata`, with two-way synchronization and a Record API fallback for metadata types that do not have a dedicated API.

6. Explore the fetched `sdk-examples` repository. The README lists these exact folders. Quote them as listed; do not rename them in your notes.

sdk-examples folders (fetched from README):

```
acl-sample
applicationmenu-sample
businessrule-sample
clientscript-sample
dependencies-sample
flow-sample
hello-world-sample
list-sample
record-sample
restapi-sample
service-catalog-sample
service-portal-sample
script-include-sample
scriptaction-sample
sys_module-sample
table-sample
test-atf-sample
uiaction-sample
uipage-sample
react-ui-page-ts-sample
react-ui-page-vite-sample
react-components-vite-sample
solidjs-ui-page-sample
svelte-ui-page-sample
vue-ui-page-sample
```

SDK sample commands (fetched from the repository README):

```
git clone https://github.com/ServiceNow/sdk-examples
cd sdk-examples
pnpm install
code hello-world-sample
pnpm run build
```

The repository requires Node v20+ and pnpm v9+. The last commit shown by the GitHub API was September 14, 2026. Do not present the sample folders as a substitute for the SDK docs; they are examples, and some samples exercise APIs that are not part of the CAD core.

7. Create a feature branch and make one small change through source control.

Git commands (written by me, not fetched; test it):

```
git clone https://github.com/<org>/r3-lifecycle.git
cd r3-lifecycle
git checkout -b feature/r3-lifecycle-item
git status
git add .
git commit -m "R3-123 add lifecycle item table"
git push -u origin feature/r3-lifecycle-item
```

The Using source control page lists checkout, clone, commit, branch, discard, fetch, pull, push, stage, stash, merge-conflict resolution, and commit history. It also states that only one branch per repository can be checked out on an instance or developer sandbox at a time. Do not plan a workflow that assumes two branches are checked out simultaneously on one instance.

8. Create an update set for a base-system change that is not part of the scoped app. For example, add a field to a form or change a label on an OOTB table. The Get started with update sets page says update sets capture and move customizations, are used for patches/changes to installed applications, and are not designed for installing applications; use the application repository or the ServiceNow Store for installation. The same page recommends keeping sets small, using a consistent migration path, previewing before commit, not deleting sets, and not backing out the Default set.

9. Compare update sets and applications. Use this table in your design note, not as a substitute for the docs:

| Need | Update set | Application repository |
|---|---|---|
| Change a base-system record or an installed app | Yes | Not the primary tool |
| Install a scoped app on another instance | Not designed for it | Yes |
| Version and distribute an app with entitlements | No | Yes |
| Small, reviewable change with a ticket | Yes | Possible as a release, but heavier |
| Source-level collaboration and branching | No | Git is the source system; app repo is distribution |

10. Publish the app to the application repository. The App repository page states that it is a central repository for scoped applications published by customers, that an instance must be entitled to install, and that the repository is not available for self-hosted customers. Publish a version, install it on a second instance, and record the version number. Do not call a Git push a deployment; Git stores source, the app repository distributes a packaged app.

11. Simulate a skipped change. Customize an OOTB object that the upgrade will change, then run an upgrade preview or review the upgrade history. The Resolve a skipped update page says the upgrade skips changes to objects that have a current version in the Customer Updates table, and the resolution options are retain, merge, revert, or review with no action. The Review skipped records page lists the related lists and the dispositions, including `Skipped`, `Skipped Error`, and `Skipped Manual Merge`.

12. Resolve one skipped record each way: retain one, merge one, and revert one. Set the Resolution Status and confirm the record moves from `Skipped Changes to Review` to `Skipped Changes Reviewed`. If you review a skipped record while in the wrong application scope, you can capture items from different scopes in your update set; the Resolve page warns about this.

13. Request a clone. The Request a clone page requires `clone_admin` on the source and `clone_admin` plus `soap` on the target, a target instance, and a clone profile. The Explore page states that cloning uses the most recent daily backup, at most 36 hours old, and that the target adopts the source instance’s version when cloning across versions. Configure exclusions and preservers deliberately; do not clone production over a target that has unique configuration you need without preserving it.

14. After the clone, verify the target. Check that users, groups, properties, and the source-control link are present; re-link if the clone did not preserve the credential. Run a smoke test on the app’s list, form, and one flow. A clone is not finished when the status says complete; it is finished when the app works.

15. Refactor one upgrade-hostile customization. The December 2025 Community thread lists upgrade-safe patterns (UI Policies, Flow Designer, clear Business Rule conditions, scoped Script Includes, supported extension points) and upgrade-hostile patterns (editing OOTB scripts directly, hardcoded sys_ids, DOM manipulation, untracked changes). Pick one direct OOTB edit in your lab, replace it with a scoped Script Include or a UI Policy, and document the before/after.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Scope 88fe7645c3fd6e505c545b2ed40117c8 is not in the instance` / `Unable to create scope 88fe7645c3fd6e505c545b2ed40117c8` / `Cloning https://github.com/test-sn-2/fluent1.git failed.` / `Scope "x_140000_abc1" is invalid or already exists` | Cloning a Fluent Git repo into a second instance | https://www.servicenow.com/community/developer-forum/unable-to-clone-git-repo-in-servicenow-ide/m-p/3290332 | No accepted solution. The thread has no replies. |
| `Destination does not appear to be a Service-now.com instance` | Scheduling a clone while authenticating | https://www.servicenow.com/community/sysadmin-forum/scheduling-a-clone-destination-does-not-appear-to-be-a-service/m-p/3340353 | Yes. The accepted reply points to clone ID roles and target `admin`; the original poster confirms roles fixed it. |
| `Clone stage changed from Applying Preservers to Intervened` | Production-to-development clone at about 66% | https://www.servicenow.com/community/upgrades-and-patching-forum/clone-from-production-to-development-fails-at-applying/m-p/3392564 | No accepted solution. The thread reports the stage and `sys_clone_log` detail but no fix. |
| No error string; “Link to Source Control” is absent | Scoped app on a PDI | https://www.servicenow.com/community/servicenow-studio-forum/cannot-use-quot-link-to-source-control-quot-need-alternative/m-p/3289514 | Yes. The accepted workaround is `Publish to Update Set` from the `sys_app` record; the app repository is described as for connected customer instances. |

The Git clone errors are not one error. `Scope ... is not in the instance` is a missing scope, `Unable to create scope` is a creation failure, and `Scope ... is invalid or already exists` is a collision. Read the first error in the IDE console, not the last line of the failed clone.

## Sourced best practices

- Use a dedicated repository per app, MFA, and a scoped PAT. The Link an app to source control page recommends all three, plus a `sys_user` email for commits.
- Do not link source control on production. The same page states the integration does not support linking an application or customization on a production instance; deploy to production from the app repository, an update set, or Studio.
- Use source control for source, the app repository for distribution. The App repository page describes entitlements and installation; Git is not an entitlement system.
- Do not use update sets to install applications. The Get started with update sets page says to use the application repository or the ServiceNow Store instead.
- Keep update sets small and preview them. The Get started page recommends small/medium sets, a consistent migration path, preview before commit, and not deleting sets.
- Review skipped changes while the upgrade is fresh. The Resolve page says customized objects are skipped to preserve customizations, and the resolution statuses are explicit; the Review page lists `Skipped Error` as a disposition that needs attention.
- Clone from a recent backup with deliberate exclusions and preservers. The Explore page describes the workflow and the 36-hour backup window; the Request page lists the required roles and profile.
- Treat the scope name as permanent. The Namespace identifier page says the identifier cannot be changed or removed, and the SDK CLI page caps the scope at 18 characters.
- Prefer supported extension points over OOTB edits. The December 2025 thread’s upgrade-safe list is practical: UI Policies, Flow Designer, clear Business Rule conditions, scoped Script Includes, and documented extension points.

## How to know it works

1. The remote Git repository shows the feature branch and the commit, and the diff contains only the intended app files.
2. A second instance installs the app from the application repository and shows the correct version.
3. The update set preview shows the expected records and no unrelated scopes; the set commits cleanly to test.
4. The skipped-change review has one retained, one merged, and one reverted record, each with the correct resolution status.
5. The clone completes, the target has the preserved users/properties, and the app’s list, form, and one flow pass a smoke test.
6. The app contains no hardcoded 32-character sys_ids and no direct edits to OOTB scripts; the refactored customization works.
7. The naming check passes: scope starts with `x_`, is 18 characters or fewer, and every custom field is prefixed `u_`.

## Refactor after

Replace the update-set deployment for the scoped app with an app-repository release, then keep update sets only for base-system changes. Add branch protection and a CI check that fails on a hardcoded sys_id or a modified OOTB script. Convert one more OOTB edit to a supported extension point, then run Instance Scan on the app and record the findings before and after.

## Design note

**What I built:** a scoped app with a Git-backed source workflow, an app-repository release, an update set for a base change, a skipped-change review, and a clone with preservers.

**One alternative:** keep everything in update sets and skip Git entirely. That is simpler for a one-person PDI, but it loses branch review, diffs, and the ability to reproduce a build from source.

**Why I chose this one:** the lab separates the three things teams routinely conflate — source control, packaging/distribution, and instance cloning. Each has a different failure mode, and the skipped-change review is the only place where upgrade risk becomes visible before production.

## Module 25. CAD exam mapping and skills checklist

**Level:** senior  
**Time:** 2–3 hours to study the map, plus a self-assessment pass.

**Source of the domains and weights:** the exact blueprint list provided in Round 4. This replaces the provisional six-domain table. No external URL is used for the domain list or weights.

### Domain-to-module map

| Learning domain | Weight | Modules that cover it | Coverage |
|---|---:|---|---|
| 1. Designing and Creating an Application | 15% | 14, 15, 16, 19, 20, 22, 23, 24 | Partial. Data model, scope, scripting, scale, and lifecycle are covered; mobile versions and formal app-fit assessment are not. |
| 2. Application User Interface | 20% | 14, 15, 16, 19, 22 | Partial. Forms, fields, client/server scripts, Record Producer, catalog UI, and Service Portal are covered; mobile client-side behavior is not. |
| 3. Security and Restricting Access | 20% | 19, 20, 22, 24 | Strong. Field/scripted ACLs, GlideSystem checks, cross-scope privileges, app access, client-callable Script Includes, XSS, impersonation, and OAuth. |
| 4. Application Automation | 20% | 14, 17, 18, 21, 22, 23 | Partial. Workflow, events, Script Includes, flows, email actions, reports, and debugging are covered; Document Feeds, scheduled jobs, app properties, and outbound notifications are partial or absent. |
| 5. Working with External Data | 10% | 20, 23, 24 | Strong for CSV/import sets, Transform Maps, REST, pagination, retries, attachments, and MID Server concepts; SOAP depth is absent. |
| 6. Managing Applications | 15% | 22, 24 | Strong for Git, app repository, update sets, skipped changes, cloning, naming, and upgrade-safe customization; Team Development as a product is partial. |

### Sub-topic mapping

| Domain | Sub-topic | Modules | Status |
|---|---|---|---|
| 1A | Determine if an application is a good fit with ServiceNow | 24, 25 | Partial — lifecycle decisions are covered; no dedicated fit-assessment framework |
| 1B | Design and implement a data model | 22, 23, 24 | Mapped |
| 1C | Create modules | 24 | Partial — a module is created in the lifecycle lab, but module design is not taught deeply |
| 1D | Create mobile versions of applications | — | Unmapped |
| 1E | Use Application Scope | 19, 20, 24 | Mapped |
| 2A | Create, design, and customize forms | 15, 16 | Mapped |
| 2B | Add/remove fields from forms and tables | 15, 24 | Mapped |
| 2C | Write, test, and debug client-side scripts for desktop and mobile | 15, 16, 22 | Partial — desktop and portal client scripts are covered; mobile is not |
| 2D | Write, test, and debug server-side scripts | 14, 19, 20, 22 | Mapped |
| 2E | Use a Record Producer as an application's UI | 15 | Mapped |
| 3A | Restrict access to applications and application modules | 19, 24 | Mapped |
| 3B | Automatically create application Access Controls | 19 | Partial — ACL creation is taught manually; automatic generation is not demonstrated |
| 3C | Manually create, test, and debug Access Controls | 19, 22 | Mapped |
| 3D | Use GlideSystem methods to script security | 19, 22 | Mapped |
| 3E | Use Application Scope to protect application artifacts | 19, 24 | Mapped |
| 4A | Write, test, and debug Workflow | 18 | Mapped |
| 4B | Implement and use Document Feeds | — | Unmapped |
| 4C | Create and use Application Properties | 20, 23 | Partial — properties are used in integration and scale labs, but app-property design is not a lesson |
| 4D | Create Events, Scheduled Script Executions, and Utils Script Includes | 14, 18, 22 | Partial — events and Script Includes are covered; scheduled jobs are not |
| 4E | Send and receive email | 17 | Partial — inbound email actions are covered; outbound notifications are not |
| 4F | Design and create Homepages and reports | 21 | Partial — reports and dashboards are covered; classic homepages are not |
| 5A | Import data in CSV or Excel format | 20, 23 | Mapped |
| 5B | Integrate with an external data source using SOAP | 20 | Partial — REST is taught; SOAP is only contrasted |
| 5C | Integrate with an external data source using REST | 20 | Mapped |
| 6A | Download and install applications | 24 | Mapped |
| 6B | Use Team Development to manage source code versions and code review | 24 | Partial — Git branches, commits, and review are covered; the Team Development product is not |
| 6C | Use the ServiceNow Git integration to manage source code | 24 | Mapped |

### Topics no module covers

These are fully unmapped in Modules 14–25:

| Topic | Domain | Why it is unmapped |
|---|---|---|
| Mobile app versions and mobile UI | 1D | No module builds or tests a mobile version. |
| Client-side scripts for mobile | 2C | Module 15/16 cover desktop and Service Portal, not mobile client scripts. |
| Document Feeds | 4B | No module creates or tests a document feed. |
| SOAP integration depth | 5B | Module 20 is REST-first; SOAP is mentioned only as a contrast. |

These are partial, not complete:

| Topic | Domain | What is missing |
|---|---|---|
| App-fit assessment | 1A | No repeatable framework or decision record. |
| Module creation depth | 1C | One module is created; module design and navigation are not taught deeply. |
| Automatic ACL creation | 3B | Manual ACLs are taught; automatic generation is not demonstrated. |
| Application Properties | 4C | Properties are used, but app-property lifecycle and scoping are not taught. |
| Scheduled Script Executions | 4D | Background scripts are covered; scheduled job design, clustering, and retries are not. |
| Outbound email | 4E | Inbound email actions are covered; notifications and outbound email are not. |
| Classic homepages | 4F | Reports and dashboards are covered; homepage design is not. |
| Team Development product | 6B | Git workflow is covered; Team Development records and code review are not. |

### 4-to-5-year skills checklist

Each item is mapped to a module. “Partial” means the module teaches the mechanism but not the full production depth. “Unmapped” means no module in the visible set covers it.

#### Domain 1 — Designing and Creating an Application (15%)

| # | Skill | Module | Status |
|---|---|---|---|
| 1 | Decide whether a requirement fits ServiceNow or belongs in another system | 24, 25 | Partial |
| 2 | Design a table with typed fields, reference fields, and a business key | 24 | Mapped |
| 3 | Decide whether to extend `task` or use a standalone table | 24 | Mapped |
| 4 | Design a data model that survives a clone and an upgrade | 22, 23, 24 | Mapped |
| 5 | Create an application module and a list view | 24 | Mapped |
| 6 | Create and test a mobile version of an application | — | Unmapped |
| 7 | Choose and defend an application scope, and explain why it cannot be renamed later | 19, 24 | Mapped |
| 8 | Apply a naming convention to scopes, tables, fields, and artifacts | 24 | Mapped |

#### Domain 2 — Application User Interface (20%)

| # | Skill | Module | Status |
|---|---|---|---|
| 9 | Create and customize a form with the right fields and sections | 15 | Mapped |
| 10 | Add or remove a field from a form and table without breaking existing data | 15, 24 | Mapped |
| 11 | Write and debug a desktop client script | 15, 22 | Mapped |
| 12 | Write and debug a mobile client script | — | Unmapped |
| 13 | Write and debug a server-side script | 14, 19, 20, 22 | Mapped |
| 14 | Build a Record Producer as the application’s UI | 15 | Mapped |
| 15 | Choose Data Policy vs UI Policy and explain server-side enforcement | 15 | Mapped |
| 16 | Write a reference qualifier and debug it in the right UI context | 15 | Mapped |
| 17 | Write a catalog client script without assuming Core UI form APIs | 15 | Mapped |
| 18 | Build a Service Portal widget with a client controller and server script | 16 | Mapped |

#### Domain 3 — Security and Restricting Access (20%)

| # | Skill | Module | Status |
|---|---|---|---|
| 19 | Restrict access to an application and its modules | 19 | Mapped |
| 20 | Explain how application Access Controls are generated | 19 | Partial |
| 21 | Create a manual table ACL and test it with a non-admin user | 19 | Mapped |
| 22 | Create a field-level ACL and explain its interaction with a table ACL | 19 | Mapped |
| 23 | Write a scripted ACL condition and debug it with Debug Security | 19, 22 | Mapped |
| 24 | Use `gs.hasRole()` and related GlideSystem methods in a security decision | 19, 22 | Mapped |
| 25 | Use application scope and cross-scope privileges to protect artifacts | 19, 24 | Mapped |
| 26 | Secure a client-callable Script Include | 19, 22 | Mapped |
| 27 | Test access by impersonating a user | 19 | Mapped |
| 28 | Prevent XSS in a widget or HTML field | 19 | Mapped |

#### Domain 4 — Application Automation (20%)

| # | Skill | Module | Status |
|---|---|---|---|
| 29 | Read, test, and debug a legacy Workflow | 18 | Mapped |
| 30 | Create and use a Document Feed | — | Unmapped |
| 31 | Create and use an application property | 20, 23 | Partial |
| 32 | Create and test an event and a script action | 14 | Mapped |
| 33 | Create a Scheduled Script Execution with batching and failure handling | — | Unmapped |
| 34 | Write a reusable Utils Script Include | 14, 22 | Mapped |
| 35 | Build a flow with a subflow, custom action, and error handler | 18 | Mapped |
| 36 | Use a decision table from a flow | 18 | Mapped |
| 37 | Configure an inbound email action | 17 | Mapped |
| 38 | Send an outbound email notification with a controlled recipient set | — | Unmapped |
| 39 | Design a report with conditions, grouping, and aggregation | 21 | Mapped |
| 40 | Share a dashboard and explain dashboard vs visualization rights | 21 | Mapped |
| 41 | Design a classic homepage | 21 | Partial |

#### Domain 5 — Working with External Data (10%)

| # | Skill | Module | Status |
|---|---|---|---|
| 42 | Import CSV or Excel data into an import set | 20 | Mapped |
| 43 | Write Transform Map `onBefore` and `onAfter` scripts | 20 | Mapped |
| 44 | Build a REST integration with pagination, retry, and idempotency | 20 | Mapped |
| 45 | Send and receive attachments through REST | 20 | Mapped |
| 46 | Build and debug a SOAP integration | 20 | Partial |
| 47 | Explain when a MID Server is required for JDBC or PowerShell | 20 | Mapped |
| 48 | Load 50,000–100,000 rows and measure the import path | 23 | Mapped |

#### Domain 6 — Managing Applications (15%)

| # | Skill | Module | Status |
|---|---|---|---|
| 49 | Download and install an application from the application repository | 24 | Mapped |
| 50 | Link an app to Git and perform branch, commit, push, and pull | 24 | Mapped |
| 51 | Resolve a Git merge conflict and review a diff | 24 | Mapped |
| 52 | Use Team Development records for source versions and code review | 24 | Partial |
| 53 | Use update sets for base-system changes and preview before commit | 24 | Mapped |
| 54 | Review skipped changes and retain, merge, or revert correctly | 24 | Mapped |
| 55 | Request a clone with preservers and exclusions, then validate the target | 24 | Mapped |
| 56 | Refactor an upgrade-hostile customization into a supported extension point | 24 | Mapped |
| 57 | Run Instance Scan and interpret its findings | 22 | Mapped |
