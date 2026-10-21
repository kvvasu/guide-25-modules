# ServiceNow Developer Guide — Modules 1–13

A hands-on developer guide built on one personal developer instance (PDI). Each module is a
buildable exercise with fetched sources (URL + date), real community error reports, a
verification method, and a time estimate. Progress status: all 13 modules complete (Modules 1–11 planned; 12–13 added in this pass).

Audit record (what was actually done to Module 4–6, reconciled here): the original Module 4–6
text was lost when prior chat output was truncated, so — not a prose reconstruction from memory
as an earlier header claimed — both modules were rebuilt cell by cell from fresh fetches: every
URL re-fetched, every quote re-verified verbatim against the fetched page text, quotes that
could not be verified deleted with inline disclosure, long-unsourced claims dropped (the Module 6
nullable-coalesce line, the Module 4 approval-template line), and the modules restructured into
the same section format as Modules 7–10 (fetched sources with URLs, exact error text with thread
URLs, best practices tied to fetched pages, a how-to-know-it-works test).

Modules 1–3 carried corrections in this pass: per-page pt-BR citation status (all resolved — no
pt-BR URLs remain), rebuilt GlideAjax common errors, the constructor argument now justified only
by the exact API-Name quote, the write-ACL exercise, and the GraphQL source dropped. Modules
7–11 were re-run fresh with every page fetched in-run, release-less English URLs preferred, and
community articles admitted if created on or after 17 Oct 2024 **or last edited on or after that
date** (both dates listed for any pre-cutoff creation date).

Docs quotes were matched against a local clone of the ServiceNowDocs repository (brazil branch, commit 461e11de, last commit 2026-10-16), checked on 2026-10-21. That clone is not the live site. Community quotes were matched against page text fetched during the audit.

---

## Module 1 — Scoped app with tables, forms, and ACLs

**Sources (5; all re-fetched this run):**
1. Create an application (page title "1. Create an application") — https://www.servicenow.com/docs/r/application-development/servicenow-ide-family-release/tutorial-create-application-ide.html (Release: Brazil, Updated September 10, 2026)
2. Create a table — https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateATable.html (Updated September 10, 2026)
3. Explore Access Control Lists — https://www.servicenow.com/docs/r/platform-security/access-control/exploring-access-control-list.html (Release: Brazil, Updated September 10, 2026; English release-less URL replacing the previously cited pt-BR copy — no pt-BR URLs remain in this guide. The page carries an AI-generated summary block at the top; all quotes come from the original body beneath it)
4. Enforce Scope Access Controls on New Tables — https://www.servicenow.com/docs/r/platform-security/instance-security-hardening-settings/sc-enforce-scope-access-controls-on-new-tables.html (Updated September 10, 2026)
5. ACL for allowing table access but not field access — https://www.servicenow.com/community/developer-forum/acl-for-allowing-table-access-but-not-field-access/m-p/3301067 (06-26-2025; **ACCEPTED SOLUTION** by Ankur Bawiskar, Certified Technical Architect, 10x ServiceNow MVP; fetched earlier in this conversation)

**What I'm building and why:** A scoped application containing one custom table with a designed
form and read + write ACLs. Teams want this because scoping data access prevents
cross-application data leaks and ensures only authorized roles can read or write table data.

**The Dispatch Order table (defined once here; reused by name in Modules 2, 3, and 8):**

| Field | Type | Notes |
|---|---|---|
| Number | Auto-number | e.g. DSP0001001 |
| Short description | String | Mandatory |
| Category | Choice | network, hardware, software, facilities |
| Severity | Choice | Sev1, Sev2, Sev3 |
| Assigned group | Reference → sys_user_group | Auto-filled by the Module 2 business rule |
| Due notes | String | Becomes mandatory at Sev1 via the Module 3 UI policy |
| Active | Boolean | Default true |

Table name: `x_demo_dispatch_order` — **extend the Task table** so Modules 4, 9, and 13 (catalog tasks, ATF task steps, SLAs) can attach to it. (No fetched page mandates this; extending Task is this guide's design choice, required for Module 13's SLA: https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "SLAs can be defined for any table that extends the task table" — quoted in Module 13.)

**Ordered build steps:**
1. Create a new scoped application (Create an application page: https://www.servicenow.com/docs/r/application-development/servicenow-ide-family-release/tutorial-create-application-ide.html: "Enter a scope, such as x_snc_hello_world" — https://www.servicenow.com/docs/r/application-development/servicenow-ide-family-release/tutorial-create-application-ide.html: "The scope name must be unique on the instance, begin with x_<prefix>, and be 18 characters or fewer.").
2. Create the Dispatch Order table above inside the application scope (All → System Definition → Tables → New; per the Create a table page: https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateATable.html: For a table in a scoped application, the name is prefixed with a namespace identifier to indicate that it is part of an application.).
3. Design the form using Form Designer — https://www.servicenow.com/docs/r/platform-administration/c_FormDesign.html (Updated September 10, 2026): "Administrators or users with the personalize_form role can use the form design feature to quickly create new or change existing form views." Same page, deprecation caution (verbatim): "Starting with the Australia release, form designer is being prepared for future deprecation. Try configuring your forms with Form Builder instead."
4. Seed test data with the background script below (Background Scripts → Run).
5. Elevate to `security_admin`, create a **read ACL** on `x_demo_dispatch_order` (record level), requiring a role you control (e.g., `x_demo_dispatch_user`).
6. Create a **write ACL** on `x_demo_dispatch_order`, requiring the app admin role (e.g., `x_demo_dispatch_admin`).
7. Test access: as a non-admin user without the read role, the form is denied; with the read role, records are readable but Save is denied without the write role.

Seed script (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
var groups = ['Network Ops', 'Hardware Ops', 'Software Ops', 'Facilities'];
var severities = ['Sev1', 'Sev2', 'Sev3'];
for (var i = 0; i < 5; i++) {
  var gr = new GlideRecord('x_demo_dispatch_order');
  gr.initialize();
  gr.short_description = 'Seed dispatch ' + (i + 1);
  gr.category = groups[i % groups.length];
  gr.severity = severities[i % severities.length];
  gr.insert();
}
gs.info('Seeded 5 dispatch orders');
```

**Best practices (tied to fetched pages):** the Create a table page warns: https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateATable.html: To minimize the existence of unneeded and extraneous tables in an instance, don't overdo table creation. Create only those tables you need to minimize administration tasks and shorten upgrade times. Create the ACLs in the table's own scope, per the accepted solution in the common error below. Do not add the `update_synch` attribute to any dictionary record (Module 10's quote).

**What ACLs, if any, the platform creates when you create a table with the scope's default settings (quote-or-remove audit):**
No fetched page states that the Create Table form generates ACL records for the new table, so this guide makes **no claim** that it does (or doesn't) beyond: you create the read/write ACLs yourself in steps 5–6. Lines actually fetched:
- Cross-scope enforcement, per the Enforce Scope Access Controls on New Tables page — https://www.servicenow.com/docs/r/platform-security/instance-security-hardening-settings/sc-enforce-scope-access-controls-on-new-tables.html: "Cross-scope access checks are enforced by default for newly created tables. This prevents applications from accessing data outside their scope through a bypass" (driven by the `glide.script.dot_walk.add_attribute_on_table_create` property, default `true`).
- Application assignment for records made off the table, per the Create a table page — https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateATable.html: "Any records that are created from the table record, such as modules and security rules, are assigned to this application by default."
- Baseline record-operation protection, per the Explore Access Control Lists page (original body) — "By default the ACL engine completely denies access if an ACL is empty or invalid", and https://www.servicenow.com/docs/r/platform-security/access-control/exploring-access-control-list.html: "In practice, it is rare for the system to find no matching ACLs because the system has a set of default ACLs that protect all record operations."
(Removed claim, kept visible: an earlier version asserted the platform grants the `public` role same-scope read on new tables; no fetched page supports it.)

**Common error (community — ACL for allowing table access but not field access; fetched thread, ACCEPTED SOLUTION):** If you create the ACL in a
different scope from the table, it will not behave as expected — "You can always create ACL in
the correct scope as that of the table and move that update set along with your other scope
update sets." (Ankur Bawiskar, 10x ServiceNow MVP — accepted solution). Create ACLs for your
custom table while in your application's scope.

**How to know it works:** The seed script prints 'Seeded 5 dispatch orders' and five rows exist on x_demo_dispatch_order. As a non-admin user, open a record — the form renders and existing rows are readable. Attempt to save a change without the write ACL role → access denied. With a user holding the write ACL role → save succeeds.

**Time estimate:** 60–90 minutes. PDI friendly: yes — everything lives in your scoped app on
one instance.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 2 — Business Rule + Script Include auto-assign

**Sources:**
1. Classic Business rules — https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html (release-less English URL; **verified by an exact-title search for "Classic Business rules" — first result, title match — then fetched 200 with full body this run**: Release: Brazil, Updated September 10, 2026, 4 minutes to read; AI summary block at top, all quotes from the original body beneath it. This replaces the previous Zurich-qualified URL and the previous claim that no English release-less version existed — both were wrong.)
2. Script includes — https://www.servicenow.com/docs/r/api-reference/scripts/c_ScriptIncludes.html (Updated September 10, 2026)
3. **What happened if I setWorkflow(false) — https://www.servicenow.com/community/virtual-agent-forum/what-happened-if-i-setworkflow-false/m-p/3228037 (04-04-2025; **ACCEPTED SOLUTION** by Sandeep Rajput, Tera Patron; fetched this run) — replaces the removed 2023 thread**
4. Setworkflow(false) for specific business rules — https://www.servicenow.com/community/developer-forum/setworkflow-false-for-specific-business-rules/m-p/3401400 (10-08-2025; NO accepted solution; replies by Ricardo26, Tera Guru and NityaB161013953, Tera Patron — fetched this run)

**What I'm building and why:** A business rule that auto-assigns a group on the Dispatch Order
table defined in Module 1 (`x_demo_dispatch_order`), coupled with a script include to perform the
assignment logic — the automatic-routing pattern for the app this guide builds.

**Ordered build steps (table: `x_demo_dispatch_order` from Module 1):**
1. Create a scoped Script Include (Script Includes → New): Name `DispatchAssigner` (the API Name shown on the form is what callers use — see the constructor quote in Module 3), **Client callable: Selected**, **Glide AJAX enabled: Selected**, method `assignGroup(category)` returning the group's sys_id.
2. Create a **before** insert business rule on `x_demo_dispatch_order` (Business Rules → New; a https://www.servicenow.com/community/developer-articles/understanding-gliderecord-a-practical-quick-guide-for-servicenow/ta-p/3407399: "Before" rule runs https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "After the user submits the form but before any action is taken on the record in the database" — Classic Business rules page, fetched this run).
3. In the script, when `current.isNewRecord()` and `assigned_group` is empty, call the script include with `current.category`.
4. The business rule runs within the transaction and respects ACLs.

Business rule script (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
(function executeRule(current, previous /*null when async*/) {
  if (current.isNewRecord() && current.assigned_group.nil()) {
    var helper = new DispatchAssigner();
    current.assigned_group = helper.assignGroup(current.category + '');
  }
})(current, previous);
```

Script Include (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
var DispatchAssigner = Class.create();
DispatchAssigner.prototype = {
  initialize: function() {},
  assignGroup: function(category) {
    var display = { network: 'Network Ops', hardware: 'Hardware Ops',
                    software: 'Software Ops', facilities: 'Facilities' };
    var gr = new GlideRecord('sys_user_group');
    gr.addQuery('name', display[category] || 'Network Ops');
    gr.setLimit(1);
    gr.query();
    return gr.next() ? gr.getUniqueValue() : '';
  },
  type: 'DispatchAssigner'
};
```

**setWorkflow(false) — what it costs (all lines fetched this run; the 2023 ChatGPT-labeled thread is gone):**

From the Classic Business rules page (original body):
- https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Business rules only run record operations when called from the GlideRecord API. Some applications intentionally bypass business rule processing to perform record operations directly. In addition, business rules ignore API calls run with the setWorkflow() method set to false."
- https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "You can prevent recursive business rules by using the setWorkflow() method with the false parameter. The combination of the update() and setWorkflow() methods is only recommended in special circumstances where the normal before and after guidelines mentioned above don't meet your requirements."

From the ACCEPTED SOLUTION (04-04-2025):
- Sandeep Rajput: https://www.servicenow.com/community/virtual-agent-forum/what-happened-if-i-setworkflow-false/m-p/3228037: "The setWorkflow(false) prevents business rules/flows/workflows from running. It is not needed to set the setWorkflow to true afterwords."
- Ankur Bawiskar: https://www.servicenow.com/community/virtual-agent-forum/what-happened-if-i-setworkflow-false/m-p/3228037: "just before the update() method you need to use setWorkflow(false) so that stops business rule, flow, workflows, notifications from triggering"
- Valmik Patil (Community Alums reply): The setWorkflow(false) call only affects that specific GlideRecord instance and only during that transaction. It does not persist or affect other records or future executions.

From the un-accepted thread (10-08-2025), credibly useful but not official:

(Removed, kept visible: the previous pros/cons/cost list was quoted from Pros and Cons of using setWorkFlow(false)? — created 05-08-2023, the thread's accepted answer self-labeled answer from chatGPT — under this guide's cutoff rule a pre-17-Oct-2024 article must be re-edit-dated after that date to qualify, the fetched page carried no last-edited marker, and the replacement sources above are post-cutoff and fetched. Nothing above rests on the 2023 thread.)

**GraphQL source:** dropped — not relevant to this guide.

**Best practices (tied to fetched pages):** no recursion — https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Avoid using current.update() in a business rule script." and https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "However, current.update() causes system performance issues and is never necessary." (Classic Business rules). Right rule, right timing — https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Changes made in before business rules are automatically saved when all before business rules are complete, and after business rules are best used for updating related, not current, objects." Async caveat — https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Asynchronous business rules do not have access to the previous version of a record. Therefore, the changes(), changesTo(), and changesFrom() GlideElement methods do not work with async rule script." And for new automations, the same page says: https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Use Workflow Studio for any new process automation to create automations that are easier to extend, reuse, understand, and upgrade."

**How to know it works:** Create a Dispatch Order (`x_demo_dispatch_order` from Module 1) with
no assigned group → the Before business rule fills the group via the script include before
insert; the saved row shows the assignment. The seed script from Module 1 can create the test
rows.

**Time estimate:** 60–90 minutes. PDI friendly: yes.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 3 — Client Script, UI Policy, GlideAjax

**Sources:**
1. UI policies — https://www.servicenow.com/docs/r/platform-administration/t_CreateAUIPolicy.html (English release-less, Updated September 10, 2026; re-fetched this run)
2. GlideAjax - Client — https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html (Updated September 10, 2026)
3. Script includes — https://www.servicenow.com/docs/r/api-reference/scripts/c_ScriptIncludes.html
4. getReference(): That Shortcut You Regret Later — https://www.servicenow.com/community/developer-articles/getreference-that-shortcut-you-regret-later/ta-p/3394743 (Sep 30, 2025, ServiceNow Employee author)
5. Client-side code should not use synchronous AJAX ; Really ? — https://www.servicenow.com/community/developer-forum/client-side-code-should-not-use-synchronous-ajax-really/td-p/3384467 (Sep 18, 2025)
6. client script with glide ajax is not working — https://www.servicenow.com/community/developer-forum/client-script-with-glide-ajax-is-not-working/td-p/3257912 (05-08-2025; re-fetched this run; NOT marked accepted)
(The previous bare source entry "Client Scripts" had no URL and no dependent quote — removed in the audit.)

**pt-BR citation replacement status (audit re-fetch):**
- UI policies → release-less English URL; quote re-verified verbatim this run.
- Business rules → release-less English page now confirmed: exact-title search for https://www.servicenow.com/docs/r/build-workflows/business-rules-classic/c_BusinessRules.html: "Classic Business rules" returns the release-less URL first, and it was fetched 200 with full body this run (Brazil, Updated September 10, 2026; AI block above, original body beneath) — it is the URL cited in Module 2.
- Explore Access Control Lists → now cited in Module 1 via its English release-less URL (the page carries an AI summary block, but the original body follows and is quotable). No pt-BR URLs remain in this guide.

**UI Policy behavior (release-less English quotes, re-verified this run):** the Reverse if false option means https://www.servicenow.com/docs/r/platform-administration/t_CreateAUIPolicy.html: "when the conditions are true, the specified actions are taken and when they are false, the actions are undone." And per the same page: https://www.servicenow.com/docs/r/platform-administration/t_CreateAUIPolicy.html: "You can also use client scripts to perform all of these actions, but for faster load times use UI policies when possible."

**Ordered build steps (table and fields from Module 1: `x_demo_dispatch_order`, Severity / Category / Assigned group / Due notes):**
1. Create the **UI policy** on `x_demo_dispatch_order`: condition `Severity is Sev1` → make **Due notes** mandatory (Create a UI Policy procedure — Module 3 source 1).
2. Create an **onChange** client script on the `Category` field of the same table; it calls the Module 2 script include through GlideAjax and fills `Assigned group` — no save required.
3. Add an **onSubmit** guard script for validation (async results are not available at submit; validate known states client-side and re-check on the server).

Client script (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
// onChange of category on x_demo_dispatch_order
function onChange(control, oldValue, newValue, isLoading) {
  if (isLoading || newValue === '') return;
  var ga = new GlideAjax('DispatchAssigner'); // use the API Name exactly as shown on the Script Include form
  ga.addParam('sysparm_name', 'assignGroup');
  ga.addParam('sysparm_category', newValue);
  ga.getXMLReply(function(answer) {
    if (answer) {
      g_form.setValue('assigned_group', answer, g_form.getDisplayEncodeValue('assigned_group', answer));
    }
  });
}
```

Script Include that the call targets (code written by me, not fetched from any source — test it on your instance before relying on it; same class Module 2 defines):

```javascript
var DispatchAssigner = Class.create();
DispatchAssigner.prototype = {
  initialize: function() {},
  assignGroup: function(category) {
    var display = { network: 'Network Ops', hardware: 'Hardware Ops',
                    software: 'Software Ops', facilities: 'Facilities' };
    var gr = new GlideRecord('sys_user_group');
    gr.addQuery('name', display[category] || 'Network Ops');
    gr.setLimit(1);
    gr.query();
    return gr.next() ? gr.getUniqueValue() : '';
  },
  type: 'DispatchAssigner'
};
```

- **Do not use synchronous AJAX in client scripts.** https://www.servicenow.com/community/developer-forum/client-side-code-should-not-use-synchronous-ajax-really/td-p/3384467: "Synchronous AJAX may feel safer, but it hurts performance, freezes the UI, and is flagged by HealthScan." (miftikhar20 — community reply). Always use asynchronous calls (GlideAjax or getReference with callback) and control submission with onSubmit.

**GlideAjax constructor argument (fetched pages):**
- From GlideAjax - Client: https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html: "As the argument to the constructor, specify the name of the script include class that contains the method you want to call." Example in docs: `var ga = new GlideAjax('GetUserInfo'); // GetUserInfo is the script include name`.
- From Script includes: the **API Name** field is https://www.servicenow.com/docs/r/api-reference/scripts/c_ScriptIncludes.html: "The internal name of the Script Include. Used to call the Script Include from out-of-scope applications." Also: https://www.servicenow.com/docs/r/api-reference/scripts/c_ScriptIncludes.html: "Glide AJAX enabled script includes are invoked from GlideAjax and require that users satisfy an ACL associated with the script include."
- **Constructor argument = the API Name on the form (quoted):** the Script includes page defines API Name as https://www.servicenow.com/docs/r/api-reference/scripts/c_ScriptIncludes.html: "The internal name of the Script Include. Used to call the Script Include from out-of-scope applications." Use that field's value exactly as shown on your instance — do not substitute the Name field. (No fetched page states that scoped API names carry an x_ prefix; an earlier draft asserted that as fact and it is removed here. For contrast: scope names themselves https://www.servicenow.com/docs/r/application-development/servicenow-ide-family-release/tutorial-create-application-ide.html: "begin with x_<prefix>, and be 18 characters or fewer" — Create an application page, Module 1.)
- getXML note: https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html: "getXML() is the preferred method for executing the code, because it is asynchronous and does not hold up the execution of other client code." getXMLWait() https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html: "can cause the application to seem unresponsive" and https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html: "is not available to scoped applications."

**Common errors (rebuilt from the four fetched threads):**

1. **client script with glide ajax is not working** — https://www.servicenow.com/community/developer-forum/client-script-with-glide-ajax-is-not-working/td-p/3257912 (05-08-2025; NOT marked accepted — top replies by Zach Koch, Giga Sage and Juhi Poddar, Kilo Patron)
   - **Symptom (verbatim, OP's console error):** "onChange script error: TypeError: gax.addParm is not a function"
   - **Top reply fix (verbatim, Juhi Poddar):** "Carefully search the entire form's Catalog Client Scripts for any addParm or any GlideAjax variable named gax. It may be in a different script (onChange for another field, onLoad, etc.). Fix any instance of addParm to addParam."
   - **Cause:** a typo — `addParm` instead of `addParam` (an earlier draft wrongly labelled this an Accepted fix and mis-stated the symptom as a silent failure — corrected against the fetched thread).

2. **UI Builder dropdown GlideAjax undefined** — https://www.servicenow.com/community/developer-forum/ui-builder-dropdown-component-item-script-glideajax-undefined/td-p/3284898 (Jun 9, 2025, ACCEPTED SOLUTION by Maik Skoddow, Tera Patron)
   - **Symptom (verbatim):** "In the script, I'm attempting to make a GlideAjax call but GlideAjax is underscored in red and said to be undefined inside the script."
   - **Cause:** GlideAjax does not exist in UI Builder's scripting context.

3. **getReference shortcut you regret** — https://www.servicenow.com/community/developer-articles/getreference-that-shortcut-you-regret-later/ta-p/3394743 (Sep 30, 2025, ServiceNow Employee)
   - **Symptom (verbatim):** "Why does this form take forever to load?" — after "five getReference() calls in a row… Lookups on manager, department, cost center, location, and role."
   - **Cause (verbatim):** "It doesn't care that you only needed the location field, you're pulling in every column of the user record. Multiply that by a few calls, and suddenly your form is making round trips that add noticeable lag."
   - **Fix:** switch to GlideAjax ("you control the payload… honoring ACLs, and keeping the form snappy"). Rule of thumb (page text continues after "Business Rule" with "—faster, cleaner, safer"; quote now complete): "One field, one time, small table? getReference() is fine. Multiple lookups, big tables, or production-grade flows? GlideAjax is the grown-up choice. Values always needed on load? Preload them with g_scratchpad in a Display Business Rule—faster, cleaner, safer."

4. **synchronous AJAX** — https://www.servicenow.com/community/developer-forum/client-side-code-should-not-use-synchronous-ajax-really/td-p/3384467 (Sep 18, 2025)
   - **Symptom (verbatim):** "having such synchronous AJAX will trigger Healthscan findings" — posted while reviewing a Healthscan finding[] where a catalog client script was using a synchronous call.
   - **Accepted fix (top reply by vignesh parthib, Kilo Sage):** "Use asynchronous AJAX, but control user actions while it runs" — "Disable submit buttons until updates complete. Show loading indicators. Block submission with client-side validation until AJAX finishes. Use promises, async/await, or callbacks to ensure form submission happens only after AJAX completes. Always validate data on the server… They should follow the same approach—never rely on synchronous calls."
   - Additional reply (miftikhar20): "If timing is a concern, use onSubmit to prevent submission and show an info message like \"Please wait.. loading required data\" until the async call finishes."
   - Note: no reply on that thread is marked Accepted Solution; the two above are the top/highest-rated recommendations.

5. **Constructor-argument mismatch (new, re-worded):** passing a Name that differs from the API Name makes the GlideAjax call fail to resolve the script include — use the API Name field quoted above.

**Best practices (tied to fetched pages):** prefer UI policies — https://www.servicenow.com/docs/r/platform-administration/t_CreateAUIPolicy.html: "You can also use client scripts to perform all of these actions, but for faster load times use UI policies when possible." Always async — https://www.servicenow.com/docs/r/api-reference/c_GlideAjaxAPI.html: "getXML() is the preferred method for executing the code, because it is asynchronous and does not hold up the execution of other client code." Size the lookup to the need — https://www.servicenow.com/community/developer-articles/getreference-that-shortcut-you-regret-later/ta-p/3394743: "One field, one time, small table? getReference() is fine. Multiple lookups, big tables, or production-grade flows? GlideAjax is the grown-up choice." While async runs, block submit — https://www.servicenow.com/community/developer-forum/client-side-code-should-not-use-synchronous-ajax-really/td-p/3384467: "Instead, design the UI to prevent submission until async updates finish, providing user feedback throughout."

**How to know it works:** On a new Dispatch Order, pick `network` → `assigned_group` fills with Network Ops within a second without saving; set `Sev1` → `due_notes` becomes mandatory, and Save with it blank is rejected.

**Time estimate:** 60–90 minutes. PDI friendly: yes — one instance, no loopback or second PDI needed.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 4 — Service catalog item with variables, a flow, and an approval

**Sources (all re-fetched this run; docs pages Release: Brazil, Updated September 10, 2026; every quote below re-verified verbatim):**
1. Create or edit catalog item — https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_DefineACatalogItem.html: "Define individual catalog items using Service Catalog. You can create, edit, and copy catalog items."
2. Catalog Item form — https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/catalog-item-form.html: "Description of the fields on the Catalog Item form." (also documents the Process Engine fields: "Flow" — "Flow that defines how the item request is fulfilled.")
3. Service catalog variables — https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/c_ServiceCatalogVariables.html: "Service catalog variables capture and pass on information about choices a customer makes when ordering a catalog item."
4. Create a service catalog variable — https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_CreateAVariableForACatalogItem.html: "Create service catalogs variables to gather information from users ordering a catalog item, for example, to ask users which options they want for the item."
5. With a Service Catalog trigger — https://www.servicenow.com/docs/r/build-workflows/workflow-studio/create-sc-flow.html: "Start a flow when a Service Catalog item is requested to automate the fulfillment process."
6. Flow trigger types — https://www.servicenow.com/docs/r/build-workflows/workflow-studio/flow-triggers.html (re-fetched this run — the earlier contested status is resolved: the page carries an AI summary block and the full original body follows; first body sentence: "Each trigger type defines when a flow starts and the starting data available to it.")
7. All Task not created after Request Item approval — https://www.servicenow.com/community/workflow-automation-forum/all-task-not-created-after-request-item-approval/m-p/3465313 (01-12-2026; **ACCEPTED SOLUTION** by vaishali231, Kilo Sage; also cited in Module 5)

**What I'm building and why:** A catalog item with ordered variables, executed by a Flow
Designer flow (not a legacy workflow), with an approval step — the standard request
fulfillment pattern.

**Steps:**
1. Create the catalog item (docs navigation, verbatim: https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_DefineACatalogItem.html: "Navigate to All > Service Catalog > Catalog Definition > Maintain Items.").
2. Add three variables (e.g., workstation type, location, delivery date) — tick the Mandatory check box on the variable form (https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_CreateAVariableForACatalogItem.html: "Check box to make the variable mandatory as part of the ordering process"), noting the page's caveat: https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_CreateAVariableForACatalogItem.html: "This behavior is applicable only on page load, and can be changed via client APIs."
3. On the catalog item, set the Process Engine **Flow** field (https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/catalog-item-form.html: "Flow that defines how the item request is fulfilled") rather than Workflow. Caution from the trigger page (verbatim): https://www.servicenow.com/docs/r/build-workflows/workflow-studio/create-sc-flow.html: "Service Catalog records are created in the global scope. They are not part of the source control or application repository transfer, and they are not part of the default Workflow Studio update set." — remember this in Module 10.
4. In Flow Designer (docs procedure, verbatim: "In the Trigger section, add a trigger and select Service Catalog."), add an approval action (approver = requested for's manager), then a fulfillment task action. If you want more than one catalog task created at once, untick the Create Catalog Task action's wait-for-completion setting (sourced thread below).
5. Test.

**Common errors (exact symptoms + fixes):**
1. **Only the first catalog task is created after approval** — https://www.servicenow.com/community/workflow-automation-forum/all-task-not-created-after-request-item-approval/m-p/3465313 (01-12-2026; **ACCEPTED SOLUTION** by vaishali231, Kilo Sage)
   - **Symptom (verbatim):** "When approved by manager, it is suppose to create 3 catalog task but only the first one is created and I do not understand why"
   - **Accepted fix (verbatim):** "In Flow Designer, actions run one after another, not in parallel. When you use Create Catalog Task, the flow waits until that task is completed (closed) before moving to the next step."
   - Mechanism (Mark Manders, Tera Patron, not marked accepted): "This is default behavior, because of the 'wait for completion' checkbox. That's default set to 'true'. If all tasks need to be created at the same time... You just need to untick the box and then create logic to wait for all three tasks to be completed before you let the flow continue."
(Withdrawn claim, kept visible: an earlier version quoted an unattributable line — Whatever writes an approval must leave a populated Requested For on the RITM... — from a thread with no reachable URL; the quote was removed and replaced with the fetched, accepted thread above. Earlier claims about flow failure/error text, variable mandatory enforcement and Requested state stuck threads are likewise withdrawn — they were not re-fetchable in this audit.)

**Best practices (tied to fetched pages):** set the Process Engine **Flow** field rather than Workflow — https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/catalog-item-form.html: "Flow that defines how the item request is fulfilled" (Catalog Item form). Variables are part of the item definition: https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/c_ServiceCatalogVariables.html: "Service catalog variables capture and pass on information about choices a customer makes when ordering a catalog item." Mind the mandatory-checkbox caveat: https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_CreateAVariableForACatalogItem.html: "This behavior is applicable only on page load, and can be changed via client APIs." Build the trigger from the documented flow type: https://www.servicenow.com/docs/r/build-workflows/workflow-studio/create-sc-flow.html: "Start a flow when a Service Catalog item is requested to automate the fulfillment process." And untick wait-for-completion when tasks must appear together (sourced thread above).

**How to know it works:** Order Workstation Request with all three variables filled → an RITM appears with an approval in state *requested*; approve as that user → the first fulfillment task is created by the flow (later tasks appear as earlier ones close, per the sourced wait-for-completion behavior above — untick it if you want them all at once).

**Time estimate:** 90–120 minutes. PDI friendly: yes — everything, including the approval, runs on your single PDI.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 5 — Employee onboarding request end to end

**Sources (all re-fetched this run; docs pages Release: Brazil, Updated September 10, 2026; every quote re-verified verbatim):**
1. Create an email notification — https://www.servicenow.com/docs/r/platform-administration/t_CreateANotification.html: "Create an email notification specifying when to send it, who receives it, what it contains, and if it can be delivered in an email digest."
2. Send Email action — https://www.servicenow.com/docs/r/build-workflows/workflow-studio/send-email-flow-designer.html: "Send an email to specified users or groups as an action in a flow." (page carries an AI summary block; quotes from the original body)
3. Managing email properties for your PDI — https://www.servicenow.com/docs/r/application-development/managing_email_properties_for_your_pdi.html: "Configure properties within your PDI to start sending and receiving emails."
4. Email Log does not capture record of email generated using https://www.servicenow.com/community/sysadmin-forum/email-log-does-not-capture-record-of-email-generated-using-quot/m-p/3252300: "Send Email" Flow Action — https://www.servicenow.com/community/sysadmin-forum/email-log-does-not-capture-record-of-email-generated-using-quot/m-p/3252300 (05-01-2025; **ACCEPTED SOLUTION** posted by the OP xwharton, Tera Guru — root cause: semicolon-separated recipients; see body)
5. All Task not created after Request Item approval — https://www.servicenow.com/community/workflow-automation-forum/all-task-not-created-after-request-item-approval/m-p/3465313 (01-12-2026; **ACCEPTED SOLUTION** by vaishali231, Kilo Sage — quoted in Module 4; informs the parallel-task step here)
6. Preview email notifications — https://www.servicenow.com/docs/r/platform-administration/t_PreviewingNotifications.html: "You can preview what notifications look like before you actually enable the instance to send them."
7. System email log — https://www.servicenow.com/docs/r/platform-administration/r_EmailLogs.html (page title: "System email log and mailboxes"): "The system email log records all emails that the instance creates or receives."

**What I'm building and why:** The full employee onboarding request: catalog item → flow →
chained tasks → approval → notifications, all end to end on one PDI.

**Steps:**
1. Catalog item Employee Onboarding with variables (new hire name, start date, department).
2. Flow with Service Catalog trigger: approval (manager), then task Provision accounts, then parallel task Prepare equipment.
3. Notifications: create an email notification on the task/state change; test with preview first (https://www.servicenow.com/docs/r/platform-administration/t_CreateANotification.html: "You can preview what notifications look like before you actually enable the instance to send them"). Recipients must be real: https://www.servicenow.com/docs/r/platform-administration/t_CreateANotification.html: "Your notification recipients must be active users and have a valid email address defined."
4. PDI email: enable the email toggle on the Developer Site (https://www.servicenow.com/docs/r/application-development/managing_email_properties_for_your_pdi.html: "Manage Email Properties" action); PDIs send only to guerrillamail.com (quoted below).

**Known PDI quirk (sourced, re-fetched this run):** Managing email properties for your PDI — https://www.servicenow.com/docs/r/application-development/managing_email_properties_for_your_pdi.html: "Configure properties within your PDI to start sending and receiving emails." The same page states the PDI restriction verbatim: https://www.servicenow.com/docs/r/application-development/managing_email_properties_for_your_pdi.html: "In order to prevent spamming but still allow notifications on PDIs, we have restricted our email servers to sending to a single domain, guerrillamail.com. PDIs can receive email from any domain except guerrillamail.com."

**Common errors (exact symptoms + fixes):**
1. **Send Email action output missing from the email log and the test address** — https://www.servicenow.com/community/sysadmin-forum/email-log-does-not-capture-record-of-email-generated-using-quot/m-p/3252300 (05-01-2025; **ACCEPTED SOLUTION** posted by the OP xwharton, Tera Guru)
   - **Symptom (verbatim):** "the email that should be generated from the Send Email action does not show up in the email log, nor does it route to the test email address that I have set up in the Email Properties module."
   - **Accepted fix (verbatim):** "In the recipient fields, I separated the emails with a semicolon instead of a comma, however, Flow Designer did not throw an error."
   - Checking order: recipient separators first — an earlier draft of this guide mis-summarised the thread as check Email Properties and the system email log, which was not the accepted resolution; corrected here.

**Best practices (tied to fetched pages):** preview before enabling — https://www.servicenow.com/docs/r/platform-administration/t_PreviewingNotifications.html: "You can preview what notifications look like before you actually enable the instance to send them." Recipients must be real — https://www.servicenow.com/docs/r/platform-administration/t_CreateANotification.html: "Your notification recipients must be active users and have a valid email address defined." https://www.servicenow.com/docs/r/build-workflows/workflow-studio/send-email-flow-designer.html: "To verify that the email was generated when testing the action, review the email record in the Email [sys_email] table. The Headers field indicates whether the email was successfully generated." Cross-check the instance log — https://www.servicenow.com/docs/r/platform-administration/r_EmailLogs.html: "The system email log records all emails that the instance creates or receives." Separate multiple recipients with commas (sourced thread above).

**How to know it works:** Order the item → approval appears for the manager → approve → Provision accounts task exists; complete it → Prepare equipment appears too (untick the Create Catalog Task wait-for-completion default for parallel creation — Module 4's sourced thread), and per the Send Email action page (verbatim) — https://www.servicenow.com/docs/r/build-workflows/workflow-studio/send-email-flow-designer.html: "To verify that the email was generated when testing the action, review the email record in the Email [sys_email] table. The Headers field indicates whether the email was successfully generated." Cross-check in System Logs > Emails: https://www.servicenow.com/docs/r/platform-administration/r_EmailLogs.html: "The system email log records all emails that the instance creates or receives." The delivered copy lands at a guerrillamail.com address (restriction quoted above).

**Time estimate:** 120–150 minutes. PDI friendly: yes — no second instance needed anywhere in this module; the only PDI quirk is email delivery (source above), which the guerrillamail.com test path solves.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 6 — Scheduled job + Import Set and Transform Map loading a CSV

**Sources (all re-fetched this run; docs pages Release: Brazil, Updated September 10, 2026; every quote re-verified verbatim):**
1. Importing data using import sets — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/c_ImportDataUsingImportSets.html: "To import data define a data source and transform map, and run or schedule an import."
2. Create a transform map — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_CreateATransformMap.html: "Every import operation to a production table requires at least one transform map associated with an import set."
3. Create a field map — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_CreatingAFieldMap.html: "Field maps establish a relationship between a field in an import set table and a field in the target table."
4. Schedule a data import — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_ScheduleADataImport.html: "Schedule imports to make it possible to specify that certain import operations occur at a regular interval."
5. GlideImportSetTransformer — https://www.servicenow.com/docs/r/api-reference/server-api-reference/GlideImportSetTransformerAPI.html: "The GlideImportSetTransformer API provides methods to create an Import Set Transformer object used to execute an Import Set Transform."
6. Updating records using coalesce — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/c_ImportSetCoalesce.html: "The coalesce option on a field map allows you to specify if the selected Target field should be used to coalesce on when import set records are transformed."

**What I'm building and why:** A File data source (CSV) → Import Set → Transform Map with a
coalesced field, run on a schedule by a scheduled job — the standard nightly-data-load pattern.

**Steps:**
1. Create an Import Set table (System Import Sets → Create Table) with fields matching the CSV.
2. Create a data source (File) and load the CSV (Load Data → file upload).
3. Create a Transform Map on the import set: map fields; set `vendor_name` (or equivalent) as **coalesce** so re-runs update instead of duplicate.
4. Create a scheduled job (System Scheduled Jobs → New → Execute a script, or use Schedule a data import on the data source) that loads and transforms the CSV at a fixed interval.
5. Test by running the job twice.

**Common errors (exact symptoms + fixes):** no community thread for this module was fetched in this run, so there is no exact error text with a thread URL to quote here — rather than invent one, the sourced platform cautions in the next section stand in its place.

**Best practices and cautions (tied to fetched pages, re-verified against pages fetched this run):**
- Coalesce uniqueness (Updating records using coalesce, verbatim) — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/c_ImportSetCoalesce.html: "Choose fields in the target table to coalesce on only if those fields will have unique values. If more than one record in the target table matches the specified coalesce options, only the first matching record in the target table is updated." Empty values do not match unless you opt in — the field map has a "Coalesce empty fields" check box: https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_CreatingAFieldMap.html: "Check box to match an empty source field value to an empty target field value."
- Index your coalesce fields (Create a field map, verbatim): https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_CreatingAFieldMap.html: "You must create an index on the target table if none of the coalesce fields are indexed."
- Run business rules (Create a transform map, verbatim — https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_CreateATransformMap.html): the check box will "run business rules, workflows, approval engines, auditing, and field normalization while the transformation inserts or updates data into the target table. Clearing this check box runs GlideRecord.setWorkflow() with a value of false." — the same setWorkflow(false) cost flagged in Module 2 applies to bulk imports.
- Scheduled imports (Schedule a data import, verbatim warning): https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_ScheduleADataImport.html: "Giving the user the import_scheduler role is equivalent to giving the user the admin role, because the import_scheduler has the ability to execute scripts with administrator level privileges." And: https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/t_ScheduleADataImport.html: "Do not schedule multiple imports at the same time. Running multiple imports at the same time might negatively impact performance or cause an instance outage."
- Programmatic transforms https://www.servicenow.com/docs/r/api-reference/server-api-reference/GlideImportSetTransformerAPI.html: "Once you have created the Import Set record it must be consumed by the GlideImportSetTransformerAPI in order to trigger the transform" — example on the page: `transformer.transformAllMaps(importSetGr);`.
- (An earlier guide draft advised check the file delimiter before debugging the map as an author tip with no fetched source; dropped under this audit's no-uncited-claims rule.)
(An earlier version of this guide quoted Never leave a coalesced column nullable in the source file from a page this audit could not fetch or locate; the quote has been removed and replaced with the sourced cautions above.)

**How to know it works:** Run the job twice on a CSV containing one new vendor and one existing vendor_name: after run 1 the new row exists; after run 2 the count is unchanged and the existing vendor's contact_email reflects the file — proof of coalesce updating, not duplicating.

**Time estimate:** 90–120 minutes. PDI friendly: yes — File data source, local CSV, one instance; nothing here needs a MID Server or a second instance.

**People warned against:** None found. Reddit was not used (network blocked).

---

---

## Module 7 — Outbound REST call to a public API with error handling and a log table

**Sources (all fetched in-run):**
1. Outbound REST web service (REST) — https://www.servicenow.com/docs/r/api-reference/web-services/c_OutboundRESTWebService.html (Release: Brazil, Updated September 10, 2026; release-less English URL): "ServiceNow outbound REST functionality allows you to retrieve, create, update, or delete data on a web services server that supports the REST architecture."
2. RESTMessageV2 - Scoped, Global — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_RESTMessageV2API.html (release-less English URL; fetch landed mid-page on a method section — page description quoted instead): "The RESTMessageV2 API provides methods that allow you to send outbound REST messages using JavaScript."
3. Outbound REST authentication — https://www.servicenow.com/docs/r/api-reference/web-services/c_OutboundRESTAuth.html (Updated September 10, 2026): "Outbound REST messages support multiple types of authentication."
4. Outbound web services: Logging — https://www.servicenow.com/docs/r/api-reference/web-services/outbound-request-logging.html (Updated September 10, 2026): "Log requests and responses for outbound web services such as REST and SOAP." "To view outbound web service logs, navigate to System Logs > Outbound HTTP Requests. All log information is read only."
5. Log all outbound http request fields [Removed in Security Center v1.3.2] — https://www.servicenow.com/docs/r/platform-security/instance-security-hardening-settings/sc-log-all-outbound-http-request-fields.html: glide.outbound_http.security.log.allow.all.fields, default/recommended false.
6. Error 500 in REST Outbound — https://www.servicenow.com/community/developer-forum/error-500-in-rest-outbound/m-p/3133356 (12-20-2024; single reply by Benjamin A, Mega Sage, not marked accepted)
7. Outbound HTTP Logging — https://www.servicenow.com/community/developer-forum/outbound-http-logging/m-p/3197997 (03-06-2025; no accepted solution; top reply by Chaitanya ILCR, Giga Patron)
8. REST Call for non-admin users is failing with 401 — https://www.servicenow.com/community/developer-forum/rest-call-for-non-admin-users-is-failing-with-401/m-p/3426764 (11-13-2025; **ACCEPTED SOLUTION** by Ankur Bawiskar, CTA, 10x ServiceNow MVP)
9. PDI outbound REST: HTTP status 0 and Outbound HTTP URL shows "/" — https://www.servicenow.com/community/developer-forum/pdi-outbound-rest-http-status-0-and-outbound-http-url-shows-amp/m-p/3459667 (01-03-2026; **ACCEPTED SOLUTION** by BeingKhan; OP confirmed fix 01-04-2026)

**Dropped/honestly flagged from an earlier draft:** Support KB KB0659194 fetch failed (No readable text found) — not cited; the Developer's Guide to Implementing Error Logging article (Sep 3, 2024) predates the community cutoff and was not re-fetched — not cited; 2016/2017 threads predate the cutoff — replaced by the 2026 PDI status-0 thread. The custom-log-table pattern (GlideRecord insert into u_custom_log on failure) is standard community practice but has **no fetched source** — treat as unsourced.

**What I'm building and why:** An end-to-end outbound REST pattern — a REST Message calling a public API from a Script Include, with status-code and exception handling, platform outbound logging for debugging, and an optional custom log table for failures. This completes the request/observe/debug loop.

**Steps:**
1. Create a REST Message: System Web Services → Outbound → REST Messages → New. Endpoint `https://jsonplaceholder.typicode.com/posts/1`.
2. Authentication: per the docs — basic auth, OAuth 2.0, or mutual auth via protocol profiles. For a public API, select **No authentication**.
3. On the HTTP method, use the Related Link **Set HTTP Log Level → All** (per the community fixes — default log level can leave body/headers empty).
4. Script the call in a Script Include (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
var r = new sn_ws.RESTMessageV2('MyRESTMessage', 'GET');
try {
  var response = r.execute();
  var httpStatus = response.getStatusCode();
  if (httpStatus == 200) {
    var body = response.getBody();
  } else {
    gs.error('API Call Failed: Status ' + httpStatus + ', Message: ' + response.getErrorMessage());
    // unsourced custom-log-table pattern: insert status + message into u_custom_log
  }
} catch (ex) {
  gs.error('Error in API Call: ' + ex.getMessage());
}
```

5. Inspect what actually left the instance: **System Logs → Outbound HTTP Requests** (read-only).

**Common errors (exact symptoms + fixes):**
1. **HTTP status 0 / URL shows "/" (PDI)** — Jan 3, 2026, ACCEPTED. Symptom (verbatim): any outbound REST call using [sn_ws.RESTMessageV2] to my AWS API returns HTTP status 0… the record shows: response_status = -1, URL = / (ie no host). https://www.servicenow.com/community/developer-forum/pdi-outbound-rest-http-status-0-and-outbound-http-url-shows-amp/m-p/3459667: "In ServiceNow, HTTP status 0 is not a server response. It means:" "The request never left the instance". Checklist: uncheck Use MID Server unless needed; endpoint must be absolute; log r.getEndpoint() before execute; never set a manual Host header. OP confirmed: https://www.servicenow.com/community/developer-forum/pdi-outbound-rest-http-status-0-and-outbound-http-url-shows-amp/m-p/3459667: "i hard coded the end point and all working now."
2. **401 for non-admin users, OAuth 2.0** — Nov 13, 2025, ACCEPTED (Ankur Bawiskar, 10x MVP). Symptom (verbatim): https://www.servicenow.com/community/developer-forum/rest-call-for-non-admin-users-is-failing-with-401/m-p/3426764: "the REST Call returns 200 Status if a user with Admin role updates the fields. But when a Non-Admin user updates the fields… returns a 401 Status Code." Fix (verbatim): https://www.servicenow.com/community/developer-forum/rest-call-for-non-admin-users-is-failing-with-401/m-p/3426764: "that ACL config should help here. ensure that API user has access to OAuth Profile as well and then run".
3. https://www.servicenow.com/community/developer-forum/error-500-in-rest-outbound/m-p/3133356 — **"The payload is not valid JSON" on POST** — Dec 20, 2024. Symptom (pasted output, verbatim — the OP's paste starts mid-JSON with no opening brace):

```text
The payload is not valid JSON.","detail":""},"status":"failure"}
```

The script used setStringParameterNoEscape instead of building a JSON body. Fix (Benjamin A): "Set HTTP Log Level and select All… Then go to System Logs > Outbound HTTP Requests, select the outbound request and screenshot the Request tab" — confirm the body structure; serialize with JSON.stringify and keep Content-Type: application/json.
4. **Outbound HTTP log shows empty body/headers** — Mar 6, 2025. Symptom (verbatim): https://www.servicenow.com/community/developer-forum/outbound-http-logging/m-p/3197997: "the Request Body and Request Headers and Response is coming empty in Outbound HTTP Logs Table." Fix (top reply, Chaitanya ILCR, Giga Patron): "set the http method function's log level to elevated or All and check". Not marked accepted. Related docs note: don't flip glide.outbound_http.security.log.allow.all.fields just to see payloads.

**Best practices (tied to fetched pages):** authenticate deliberately — https://www.servicenow.com/docs/r/api-reference/web-services/c_OutboundRESTAuth.html: "Outbound REST messages support multiple types of authentication." Debug with the read-only log — https://www.servicenow.com/docs/r/api-reference/web-services/outbound-request-logging.html: "To view outbound web service logs, navigate to System Logs > Outbound HTTP Requests. All log information is read only." and use Set HTTP Log Level → All only while debugging (community fixes #3/#4, then revert). Leave glide.outbound_http.security.log.allow.all.fields at false (default/recommended false, security hardening source 5). A status-0 result means the request never left the instance (source 9's checklist) — check endpoint/MID/Host header before debugging the API.

**How to know it works:** Run the Script Include; System Logs → Outbound HTTP Requests shows one entry with a full HTTPS URL, HTTP 200, and JSON body. Break the endpoint (bad hostname) and re-run: status 0/-1 and gs.error in System Logs → Errors.

**Time estimate:** 60–90 minutes. **PDI friendly:** yes, with the PDI caveat sourced from error #1 — a hardcoded endpoint on a public API avoids the MID-Server and TLS pitfalls.

**People warned against:** None found. Reddit was not used (network blocked).

---

## Module 8 — Inbound REST: receiving external data with a Scripted REST API

**Sources (all fetched in-run):**
1. Scripted REST APIs — https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_CustomWebServices.html (Release: Brazil, Updated September 10, 2026; release-less English URL; AI-generated summary block at page top — quotes below from the original body): "The scripted REST API feature allows application developers to build custom web service APIs." URI format: `https://<instance.service-now.com>/api/<name_space>/<version>/<api_id>/<relative_path>` — "For web services in a scoped application, the name space is the scope name, such as x_company_appname." ACL rule: "A requesting user must satisfy at least one of the ACLs"; ACL Type must be REST_Endpoint. Error-object format: `{ "error": { "message": "My error message", "detail": "My details" }, "status": "failure" }` — "The numeric status code, such as 404, is included in the response Status code header, not in the response body." Also: "You can use error objects in scripted REST API resources to alert requesting clients of errors. Use error objects to respond to incoming requests, not to catch errors within your server-side code."
2. Enable OAuth with inbound REST — https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/t_EnableOAuthWithREST.html (Updated September 10, 2026): "Using OAuth, you can pass a user ID and password once, and then use a token for subsequent REST requests instead of submitting credentials with each request." Procedure: System OAuth → Application Registry → New → "Create an OAuth API endpoint for external clients"; POST to oauth_token.do; "Submit the access token with subsequent REST requests."
3. RESTAPIRequest - Scoped, Global — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_ScriptableServiceRequest.html: "The RESTAPIRequest API provides methods that allow you to access scripted REST API request details in scripts." Objects "are created automatically and are accessible only in scripted REST API resource scripts."
3a. **RESTAPIRequestBody - Scoped, Global — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_ScriptableServiceRequestBody.html (Release: Brazil, Updated September 10, 2026; fetched this run):** "The RESTAPIRequestBody API provides methods that allow you to access the body content of a scripted REST API request in scripts." Format: "The format of a RESTAPIRequestBody object may be JSON or XML, depending on the content-type header value from the request." Accessor line (quoted): the `data` field is "The request content. This can be a single object or an array of objects depending on the request." — docs example uses `var requestBody = request.body; var requestData = requestBody.data;`. Stream warning (quoted): "REST web services consume data, datastream, and dataString via stream. Because streams are only consumed once, only one can be called. For example, if calling both data and dataString, the second call will return empty."
4. RESTAPIResponse - Scoped, Global — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_ScriptableServiceResponseBuilder.html: "The RESTAPIResponse API provides methods that allow you to build a RESTful response to a scripted REST API request."
5. Need to setup Authentication for Scripted REST API — https://www.servicenow.com/community/itsm-forum/need-to-setup-authentication-for-scripted-rest-api/m-p/3431922 (11-19-2025; no accepted solution; replies by Ankur Bawiskar, CTA/10x MVP)
6. API (Solved: 500 in production but works in dev) — https://www.servicenow.com/community/developer-forum/api/m-p/3311849 (07-06-2025; **ACCEPTED SOLUTION** by NityaB161013953, Tera Patron; generic checklist, no root cause stated)
7. Help with Scripted Rest API — https://www.servicenow.com/community/developer-forum/help-with-scripted-rest-api/td-p/3149742 (01-15-2025; not marked accepted; fix by Ankur Bawiskar, 10x MVP)

**What I'm building and why:** An inbound Scripted REST API that external systems call to push data into ServiceNow, with authentication and proper error responses — the receive half of the Module 7 pair.

**Steps:**
1. Scripted Web Services → Scripted REST APIs → New. Set the API Name; the API ID derives from it.
2. Add a POST resource with a relative path (e.g. /dispatch). Resource script (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
(function process(/*RESTAPIRequest*/ request /*RESTAPIResponse*/ response) {
  var payload = request.body.data; // docs pattern: requestBody.data — single object or array
  var gr = new GlideRecord('x_demo_dispatch_order'); // the table defined in Module 1
  gr.initialize();
  gr.short_description = payload.short_description || '';
  gr.insert();
  response.setStatus(201);
  return JSON.stringify({ sys_id: gr.getUniqueValue() });
})(request, response);
```

(code written by me, not fetched from any source — test it on your instance before relying on it; the `request.body.data` accessor is the documented one quoted in source 3a, replacing a previous draft's `getBodyString()` call, which appears on no fetched page)
3. Test with the built-in REST API Explorer before involving any external client.
4. Authentication: public for a first pass (the docs security matrix documents a public row); for real use, basic auth via a dedicated API user (Ankur Bawiskar: https://www.servicenow.com/community/itsm-forum/need-to-setup-authentication-for-scripted-rest-api/m-p/3431922: "if 3rd party wants to use Basic auth, then you need to create an API user and share the userid and password for them") or OAuth 2.0 (his recommendation: https://www.servicenow.com/community/itsm-forum/need-to-setup-authentication-for-scripted-rest-api/m-p/3431922: "I will suggest to use OAuth 2.0 as it's more secured rather than Basic Auth") following the Application Registry procedure above.
5. Error responses: use the platform error-object format above; numeric code in the Status code header; unsupported Accept headers fall back to JSON.
6. If you require a role: create a REST_Endpoint-type ACL; https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_CustomWebServices.html: "A requesting user must satisfy at least one of the ACLs."

**Common errors (exact symptoms + fixes):**
1. https://www.servicenow.com/community/itsm-forum/need-to-setup-authentication-for-scripted-rest-api/m-p/3431922 — **`"User Not Authenticated"` on a basic-auth call** — Nov 19, 2025 (not marked accepted; MVP reply). Symptom (verbatim, OP's Postman error): `{"error": {"message": "User Not Authenticated", "detail": "Required to provide Auth information"}, "status": "failure"}`. Fix: create the API user with the required role and web-service-access checkboxes; prefer OAuth 2.0.
2. **500 Internal Server Error in production but works in dev** — Jul 6, 2025, ACCEPTED. Accepted fix (checklist, no single root cause stated): 1. Log Issue 2. Data mismatch 3. ACL's 4. Configuration mismatch 5. Scripted REST API 6. Error handling in script.
3. **Scan finding "REST Web Services writing data directly"** — Jan 15, 2025 (not marked accepted; MVP reply). Symptom (verbatim): https://www.servicenow.com/community/developer-forum/help-with-scripted-rest-api/td-p/3149742: "when I try to do Scan Application, I get following scan finding...""REST Web Services writing data directly." Fix (Ankur Bawiskar), verbatim: "1 way to overcome is pass the JSON to script include, invoke it from scripted rest api and let the script include function do the job This scan finding will be resolved". — same layering discipline as Module 2.

**Best practices (tied to fetched pages):** error objects only at the edge — https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_CustomWebServices.html: "You can use error objects in scripted REST API resources to alert requesting clients of errors. Use error objects to respond to incoming requests, not to catch errors within your server-side code." Status codes live in the header — https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_CustomWebServices.html: "The numeric status code, such as 404, is included in the response Status code header, not in the response body." Consume the body once — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_ScriptableServiceRequestBody.html: "Because streams are only consumed once, only one can be called." (source 3a). Namespace correctly — https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/c_CustomWebServices.html: "For web services in a scoped application, the name space is the scope name, such as x_company_appname." Prefer OAuth 2.0 over basic auth (MVP recommendation, quoted in step 4), and put write logic in a Script Include rather than the resource (scan finding fix, error #3).

**How to know it works:** From Postman/curl, POST valid JSON with basic auth to `https://<instance>/api/<namespace>/<api_id>/dispatch` → HTTP 201, `{"sys_id": "..."}`, record exists. Repeat without credentials → the `"User Not Authenticated"` error object. Send `Accept: image/jpeg` with a forced error → body still arrives as JSON.

**Time estimate:** 60–90 minutes. **PDI friendly:** yes — the PDI's public HTTPS URL is reachable from Postman on a laptop; no MID Server; the OAuth procedure needs only the admin role.

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** the Scripted REST APIs docs page carries an AI summary block at the top (quotes from the original body beneath it); errors #1 and #3 have no accepted solution marked (MVP replies credited instead); error #2's accepted answer is a generic checklist.

---

## Module 9 — Automated Test Framework: regression tests for your app

**Sources (all fetched in-run):**
1. Build and run your first automated test — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-tut-build-first.html (Release: Brazil, Updated September 10, 2026; release-less English URL): "By default, the system property that is used to run automated tests is disabled to prevent you from accidentally running these tests on a production system. To avoid data corruption or an outage, run tests only on development, test, and other non-production instances." Role required: atf_test_admin or admin.
2. Getting started with the Automated Test Framework — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-intro.html (Updated September 10, 2026; AI summary block at top — quotes from the original body): data preservation — "automatically tracks and deletes any data created by running tests, and automatically rolls back changes after testing"; test suites — "enable you to execute a batch of tests in a specified order… You can associate test suites with schedules that determine when the system runs the test suites"; step categories (Form / Server / REST / Service Catalog etc.) with output variables: "Many test steps return output variables whose values you can use as inputs to a later step."
3. ATF test step categories — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/test-step-categories.html: "Find test steps for a particular user interface or ServiceNow AI Platform feature."
4. ATF Test Step Status is Failure even when test logs are sucessfull — https://www.servicenow.com/community/upgrades-and-patching-forum/atf-test-step-status-is-failure-even-when-test-logs-are/td-p/3419350 (11-04-2025; **ACCEPTED SOLUTION** by MC30, Tera Guru; detailed additional replies by Anupam1, Giga Guru and Sardash, Tera Guru)
5. ATF Test keeps failing when creating a user and adding a role — https://www.servicenow.com/community/product-launch-forum/atf-test-keeps-failing-when-creating-a-user-and-adding-a-role/m-p/3228882 (04-04-2025; **ACCEPTED SOLUTION** by J Siva, Kilo Patron; OP confirmed "This worked!")
6. ATF - Execute a Schedule Job — https://www.servicenow.com/community/itsm-forum/atf-automated-test-framework-execute-a-schedule-job/m-p/3164254 (01-30-2025; no accepted solution; replies by abhi159, Mega Sage and Ankur Bawiskar, 10x MVP, neither confirmed)
7. **Enable or disable executing Automated Test Framework tests — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-enable-tests.html (Release: Brazil, Updated September 10, 2026; fetched this run):** "Role required: atf_test_admin". Property procedure (verbatim): Navigate to All > Automated Test Framework > Administration > Properties. Set the test execution property. To enable test and test suite execution, select Enable test/test suite execution. then "Select Save."
8. **Working with client test runners — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-test-runners.html (Release: Brazil, Updated October 8, 2026; fetched this run; AI summary block at top — quotes from the original body):** "If an automated test includes steps that involve a form or any other user-interface (UI) element, it runs those steps in a browser tab or window called a test runner or client test runner." Types: "Client Test Runners for manually started tests and Scheduled Client Test Runners for tests started by a schedule." Polling: "While the client test runner is idle, it checks every five seconds for waiting tests to start." You must open one yourself: "The system cannot automatically open a client test-runner session." Puppeteer-style questions answered: "To work with the client test runner module, the test execution property must be enabled." Browser guidance: "Run the client test runner in its own browser window." and "Keep the client test runner at least partially visible on the screen."

**What I'm building and why:** Automated regression tests for the Modules 1–2 app (Dispatch Order table, auto-assign business rule, mandatory UI policy), plus a REST test step covering the Module 8 endpoint — runnable on demand or on a schedule, with the platform rolling back test data after each run. (An earlier draft quoted ATF as automates regression testing of customizations before deployment — that phrase is from the ATF product marketing page, which was never fetched; the quote has been removed.)

**Steps:**
1. **Enable test execution first — quoted procedure (source 7):** https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-enable-tests.html: "Navigate to All > Automated Test Framework > Administration > Properties." → https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-enable-tests.html: "Set the test execution property." → "To enable test and test suite execution, select Enable test/test suite execution." → https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-enable-tests.html: "Select Save." (Role: atf_test_admin or admin. The property is off by default: "By default, the system property that is used to run automated tests is disabled to prevent you from accidentally running these tests on a production system. To avoid data corruption or an outage, run tests only on development, test, and other non-production instances.")
2a. **Do Form steps need the client test runner? (quoted answer — source 8, https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-test-runners.html):** yes. "If an automated test includes steps that involve a form or any other user-interface (UI) element, it runs those steps in a browser tab or window called a test runner or client test runner." Open it yourself — click the Client Test Runner module (manual tests) or start a Scheduled Client Test Runner for suite schedules, because "The system cannot automatically open a client test-runner session." Keep it visible: "Run the client test runner in its own browser window." / "Keep the client test runner at least partially visible on the screen."
2. All → Automated Test Framework → Tests → New; name + description; save.
3. Add test steps: **Form** (open new form, set severity=Sev1, assert due_notes mandatory, submit) for the Module 3 UI policy; **Server** (Jasmine-style asserts) for the Module 2 auto-assign rule; **REST** (https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-intro.html: "Create and send an Inbound REST request and verify the response… Verify the response status code, response headers, response time, and response payload") for the Module 8 endpoint. Chain steps with output variables.
4. Run; read results under Automated Test Framework → Test Results.
5. Group tests into a test suite and associate a schedule for nightly regression (docs: test suites https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-intro.html: "can be hierarchical… You can associate test suites with schedules").

**Common errors (exact symptoms + fixes):**
1. https://www.servicenow.com/community/product-launch-forum/atf-test-keeps-failing-when-creating-a-user-and-adding-a-role/m-p/3228882 — **Status "Failure" with no failure details, all steps green** — Nov 4, 2025, ACCEPTED. Symptom (verbatim) — https://www.servicenow.com/community/upgrades-and-patching-forum/atf-test-step-status-is-failure-even-when-test-logs-are/td-p/3419350: "the Test Result status is showing as" "Failure" "with no Failure Details." Accepted fix (MC30): https://www.servicenow.com/community/upgrades-and-patching-forum/atf-test-step-status-is-failure-even-when-test-logs-are/td-p/3419350: "one of the configured test step is not working. So, I suggest you test your test steps manually on an incident form and you can find the exact reason for your error." Best additional reply (Anupam1, not marked accepted): client-side errors sink the run even when steps pass; Allowed Client Errors matching is case-sensitive and exact — Ensure the error string is exactly: `Uncaught SyntaxError: Unexpected token '<'`.
2. https://www.servicenow.com/community/product-launch-forum/atf-test-keeps-failing-when-creating-a-user-and-adding-a-role/m-p/3228882 — **"Could not add role … while creating user"** — Apr 4, 2025, ACCEPTED. Symptom (verbatim): https://www.servicenow.com/community/product-launch-forum/atf-test-keeps-failing-when-creating-a-user-and-adding-a-role/m-p/3228882: "FAILURE: Could not add role 'sn_irm_cont_auth.info_system_sec_officer' while creating user with id '…'." Fix (J Siva): "This a known bug. To resolve that, in the step 3, you need to uncheck the "Impersonate this user" check box."
3. https://www.servicenow.com/community/itsm-forum/atf-automated-test-framework-execute-a-schedule-job/m-p/3164254 — **"Failed to find UI action 'Execute Now'"** — Jan 30, 2025, NO accepted solution. Symptom (verbatim): "FAILURE: Failed to find UI action 'Execute Now' on 'sysauto' form". Unconfirmed reply (abhi159): the Open Record step pointed at the wrong table — the record is on sysauto. Credited honestly as unconfirmed.

**Best practices (tied to fetched pages):** tests run off production only — "run tests only on development, test, and other non-production instances" (source 1). Runner hygiene — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-test-runners.html: "Run the client test runner in its own browser window." and "Keep the client test runner at least partially visible on the screen." (source 8). Green steps but red run? debug the step first — https://www.servicenow.com/community/upgrades-and-patching-forum/atf-test-step-status-is-failure-even-when-test-logs-are/td-p/3419350: "I suggest you test your test steps manually on an incident form and you can find the exact reason for your error" (MC30, accepted). Match Allowed Client Errors strings exactly — see error #1's case-sensitivity note (Anupam1, not accepted).

**How to know it works:** Test passes green with rollback (no leftover test records). Deliberately break the Module 3 UI policy → the field-state step fails with Failure Details naming the step. Restore → green again.

**Time estimate:** 60–90 minutes. **PDI friendly:** yes — ATF is designed for non-production instances; only setup is enabling the execution property.

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** intro page carries an AI summary block (quotes from the original body); thread #3 has no accepted solution; the Zurich-only Failure Insight feature (Oct 16, 2025 blog) was found in search but not fetched — not cited.

---

## Module 10 — Moving your app between instances with update sets

**Sources (all fetched in-run):**
1. Customizations tracked by update sets — https://www.servicenow.com/docs/r/application-development/system-update-sets/customizations-tracked-update-sets.html (Release: Brazil, Updated September 10, 2026; release-less English URL; AI summary block at top — quotes from the original body): "Update sets capture configuration details but exclude task or process data. For example, they record catalog item definitions, variables, and variable choices, but not submitted orders or catalog tasks." Warnings: "Don't add the update_synch attribute to a dictionary record" (can "cause major performance issues or cause the instance to become unavailable"); "Don't use large choice lists in update sets. Doing so leads to excessively long update set commits"; column deletion loses data — "A warning appears noting which deletions result in data loss." Scoped apps: "The system creates a separate update set for each application that only contains changes associated with the application."
2. General guidelines for planning the update process — https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html (Updated September 10, 2026): "Use a standard path for update sets: migrate from development to test, then from test to production. Avoid moving the same update set from more than one source." "Change a single update set for each small or medium task." "Schedule update set commits outside business hours" Naming convention guidance incl. PR-number examples.
3. Update Set Preview error (Solved) — https://www.servicenow.com/community/developer-forum/i-am-getting-an-error-under-update-set-preview-problems-in-the/m-p/3111592 (11-22-2024; **ACCEPTED SOLUTION** by Runjay Patel, Giga Sage; OP confirmed)
4. Missing customer updates in update set — https://www.servicenow.com/community/developer-forum/missing-customer-updates-in-update-set/td-p/3601013 (~Sep 2026; no accepted solution; replies by Hemanth M, CTA/MVP 2024–2026, and Tanushree Maiti, Technical Architect)
5. Merging completed update sets (Solved) — https://www.servicenow.com/community/servicenow-ai-platform-forum/merging-completed-update-sets/m-p/3467945 (01-15-2026; **ACCEPTED SOLUTION** by Mohammed8, Tera Sage; warning from glideFather, Tera Patron, not marked accepted)

**What I'm building and why:** The promotion path for everything in Modules 1–9 — moving the scoped app from the PDI to test/production without silent gaps.

**Steps:**
1. System Update Sets → Local Update Sets → New; set it current; develop (or touch/verify) each artifact while it's current.
2. Follow the docs path: dev → test → prod; single small/medium update set per task; commit outside business hours; clear naming.
3. Commit on source → export XML (or remote update source) → import on target → **Preview** → resolve Preview Problems → **Commit**.
4. Know what travels: configuration yes, task/process data no (docs quote above) — your catalog items/flows travel; RITMs, approvals, tasks, and test users do not. Recreate test users/roles on the target.
5. Honor the docs warnings: no manual update_synch; no large choice lists; beware column-deletion data loss.

**Common errors (exact symptoms + fixes):**
1. **Preview problem: missing auth profile reference** — Nov 22, 2024, ACCEPTED. Symptom (verbatim): "Description: Could not find a record in sys_auth_profile_basic for column basic_auth_profile referenced in this update". Fix (Runjay Patel): https://www.servicenow.com/community/developer-forum/i-am-getting-an-error-under-update-set-preview-problems-in-the/m-p/3111592: "this is because basic auth profile didn't get captured in your update set… this is record not a configuration so by default it won't get captured in update set. Solution: reopen update set in dev and capture it in update set and then move to uat."
2. **Catalog UI policy actions and variable order silently missing after promotion** — Sep 2026 (no accepted; CTA/MVP replies). Symptom (verbatim): "some catalog UI policy actions were missing, in addition to the order number of some variables being missing, which makes the variables appear in the wrong locations on the request form." Fix (Hemanth M, CTA/MVP): create a new update set and recapture the missing items with a trivial change; then parent-batch the sets (https://www.servicenow.com/community/developer-forum/missing-customer-updates-in-update-set/td-p/3601013: create a new update set in Test… as Parent Batch and original and additional update set as child… using a parent field Tanushree Maiti's recipe: open each record, toggle a checkbox off/on, save, then set the parent.
3. **No picker when merging completed update sets** — Jan 15, 2026, ACCEPTED, https://www.servicenow.com/community/servicenow-ai-platform-forum/merging-completed-update-sets/m-p/3467945. Symptom (verbatim): "there is no way to selectively chose a particular set(s)." Accepted fix (Mohammed8): "there is no option for selecting update set directly to merge, unfortunately. What can be done is to use filter option like this using (OR) to get to desired update sets". Additional warning (glideFather, not accepted): "DO NOT EVER change state of update set if it was once complete… Once completed, always completed… maximum is to Ignore but never to In Progress!!!" — prefer a parent update set for batching.

**Best practices (tied to fetched pages):** https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html: "Use a standard path for update sets: migrate from development to test, then from test to production. Avoid moving the same update set from more than one source." https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html: "Change a single update set for each small or medium task." https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html: "Schedule update set commits outside business hours" Never flip a completed set back — https://www.servicenow.com/community/servicenow-ai-platform-forum/merging-completed-update-sets/m-p/3467945: "DO NOT EVER change state of update set if it was once complete… Once completed, always completed… maximum is to Ignore but never to In Progress!!!" (glideFather, not accepted) — batch with a parent update set instead (error #3).

**How to know it works:** Commit → export → import → Preview reports zero problems → Commit succeeds. Spot-check on the target: scoped app in the application list; custom table with form design; Module 1 ACLs present (filtered to your scope); Module 8 REST API responds. Confirm what did NOT come over (test incidents/RITMs) and recreate one test user to re-run a Module 9 ATF test.

**Time estimate:** 60–90 minutes. **PDI friendly:** partially — capture/commit/export and Preview all work on one PDI, but full verification honestly needs a second instance; on a single PDI you can still practice the mechanics by previewing against your own instance.

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** the customizations page carries an AI summary block (quotes from the original body); the missing-updates thread has no accepted solution (CTA/MVP replies credited); the https://www.servicenow.com/community/developer-forum/missing-customer-updates-in-update-set/td-p/3601013: "add to update set" utility in two replies is a community-installed plugin, not OOB — not cited, flagged as unverified.

---

## Module 11 — Server-side performance: GlideRecord done right

**Sources (all fetched in-run):**
1. GlideAggregate - Global — https://www.servicenow.com/docs/r/api-reference/server-api-reference/c_GlideAggregateAPI.html (Release: Brazil, Updated September 10, 2026; release-less English URL): "The GlideAggregate class is an extension of the GlideRecord class and provides database aggregation (AVG, COUNT, GROUP_CONCAT, GROUP_CONCAT_DISTINCT, MAX, MIN, STDDEV, SUM) queries."
2. Understanding GlideRecord: A Practical Quick Guide — https://www.servicenow.com/community/developer-articles/understanding-gliderecord-a-practical-quick-guide-for-servicenow/ta-p/3407399 (10-17-2025; author Ravi Gaurav, Giga Sage, "ServiceNow MVP 2025 | 2024"): best practices — "Always filter your queries to avoid performance issues. Avoid using query() without conditions on large tables… Use setLimit() when testing scripts… Avoid using update() or deleteRecord() inside loops unless absolutely necessary." Common mistakes: "Forgetting to call query() before looping"; "Using gr.get() on multiple records (it only returns one)."
3. Performance Best Practice for Efficient Queries - Top 10 Practices — https://www.servicenow.com/community/developer-articles/performance-best-practice-for-efficient-queries-top-10-practices/ta-p/2306409 (**created 06-21-2017 10:58 AM; last edited 11-18-2025 09:14 AM by GTSPerformance** — qualifies under this guide's cutoff rule because the edit postdates 17 Oct 2024; re-read this run; author Mwatkins, ServiceNow Employee, Technical Support Performance team): "This guide is written by the ServiceNow Technical Support Performance team" and "I would say that probably 90% of the slow transaction issues experienced by ServiceNow users can boil down to these few things." — same filtering philosophy as the MVP quotes above.
4. Looking for a Complete Guide to GlideRecord and GlideAggregate — https://www.servicenow.com/community/developer-forum/looking-for-a-complete-guide-to-gliderecord-and-glideaggregate/td-p/3609392 (~Oct 12, 2026; no accepted solution; substantive replies by Tanushree Maiti (Technical Architect), Ranjan Kumar2, VaishnaviK3009 (Technical Consultant), Ankur Bawiskar (10x MVP)): getRowCount limitation — "Using .getRowCount() retrieves every matching record from the database into memory before counting, which causes performance and scalability issues on large tables"; GlideAggregate "executes direct SQL-like aggregate queries (such as SELECT COUNT(*)) at the database layer instead of loading all row data"; "Avoid unnecessary queries inside loops, retrieve only the records you need, and use GlideAggregate when you need summary calculations rather than individual records"; "GlideAggregate may return aggregate values as strings. When the result is used in additional mathematical calculations, it should be converted into an appropriate numeric type before further processing."

**Not cited (excluded deliberately, both dates listed where pre-cutoff):** the developer-blog post Improving script performance with GlideAggregate (Jan 16, 2024) contains hard benchmark numbers (3x faster… 70ms compared to 200msbut was not fetched in-run and predates the community cutoff — numbers treated as unsourced and omitted. The sibling GTS article https://www.servicenow.com/community/developer-articles/database-performance-improving-slow-or-and-join-queries/ta-p/2299441: "Database Performance: Improving Slow OR and JOIN Queries" — https://www.servicenow.com/community/developer-articles/database-performance-improving-slow-or-and-join-queries/ta-p/2299441 (created 01-10-2017 08:39 PM; last edited 08-02-2024 08:42 AM — the edit itself predates 17 Oct 2024, so it does not qualify). The blog https://www.servicenow.com/community/developer-blog/video-community-live-stream-lessons-learned-gliderecord-is/ba-p/2274145: "Video: Community Live Stream - Lessons Learned - GlideRecord is" — https://www.servicenow.com/community/developer-blog/video-community-live-stream-lessons-learned-gliderecord-is/ba-p/2274145 (created 12-16-2020; no last-edited marker appears on the fetched page, so post-cutoff editing cannot be shown — not cited).

**What I'm building and why:** A performance-hardened version of the data-access code from Modules 2, 5, and 6: no query inside a loop, counts via GlideAggregate instead of getRowCount, narrow filters and limits.

**Steps:**
1. Write the naive version first — background script looping your table with a fresh query per row; stamp gs.nowDateTime() before/after for a baseline.
2. Query once, loop many — hoist the query; build a map keyed by the lookup field; read from it inside the loop.
3. Replace getRowCount with GlideAggregate COUNT (https://www.servicenow.com/community/developer-forum/looking-for-a-complete-guide-to-gliderecord-and-glideaggregate/td-p/3609392: "GlideAggregate may return aggregate values as strings. When the result is used in additional mathematical calculations, it should be converted into an appropriate numeric type before further processing" — the groupBy/count variant follows the code blocks below).
4. Narrow and cap — filters always; setLimit in testing; no update/deleteRecord in loops (MVP quotes above).
5. Re-run the timing script; compare stamps.

Naive baseline (code written by me, not fetched from any source — test it on your instance before relying on it):

```javascript
var t0 = gs.nowDateTime();
var gr = new GlideRecord('x_demo_dispatch_order');
gr.addActiveQuery();
gr.query();
while (gr.next()) {
  var g = new GlideRecord('sys_user_group'); // one fresh query PER ROW — this is the anti-pattern
  g.get(gr.assigned_group);
}
var t1 = gs.nowDateTime();
var agg = new GlideAggregate('x_demo_dispatch_order'); // count without loading rows
agg.addAggregate('COUNT');
agg.query();
if (agg.next()) gs.info('rows: ' + agg.getAggregate('COUNT') + ' / naive loop ran ' + t0 + ' → ' + t1);
```

Refactored version (code written by me, not fetched from any source — test it):

```javascript
var t0 = gs.nowDateTime();
var groups = {};
var gq = new GlideRecord('sys_user_group');
gq.query();                       // ONE query, hoisted above the loop
while (gq.next()) groups[gq.sys_id + ''] = gq.name + '';
var rows = [];
var gr = new GlideRecord('x_demo_dispatch_order');
gr.addActiveQuery();
gr.query();
while (gr.next()) {
  rows.push(gr.number + ' → ' + (groups[gr.assigned_group + ''] || 'unassigned'));
}
gs.info('refactored: ' + t0 + ' → ' + gs.nowDateTime() + ' / ' + rows.length + ' rows');
```

**Common mistakes (from fetched sources):** forgetting query() before the loop; expecting multiple rows from gr.get(); treating aggregate results as numbers (parse first); update()/deleteRecord() inside loops.

**Best practices (tied to fetched pages):** filter and cap — https://www.servicenow.com/community/developer-articles/understanding-gliderecord-a-practical-quick-guide-for-servicenow/ta-p/3407399: "Always filter your queries to avoid performance issues. Avoid using query() without conditions on large tables… Use setLimit() when testing scripts… Avoid using update() or deleteRecord() inside loops unless absolutely necessary." (MVP article). Count with GlideAggregate — https://www.servicenow.com/community/developer-forum/looking-for-a-complete-guide-to-gliderecord-and-glideaggregate/td-p/3609392: "Using .getRowCount() retrieves every matching record from the database into memory before counting, which causes performance and scalability issues on large tables". One loop, one query — https://www.servicenow.com/community/developer-forum/looking-for-a-complete-guide-to-gliderecord-and-glideaggregate/td-p/3609392: "Avoid unnecessary queries inside loops, retrieve only the records you need, and use GlideAggregate when you need summary calculations rather than individual records".

**How to know it works:** Refactored script returns identical results (assert equality) with lower stamped elapsed time. Measure on representative data — tiny tables may show noise, which is itself the lesson.

**Time estimate:** 60–90 minutes. **PDI friendly:** yes — background scripts, one instance, no plugins. If tables are too small, import more rows via the Module 6 import set rather than faking numbers.

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** the https://www.servicenow.com/community/developer-forum/looking-for-a-complete-guide-to-gliderecord-and-glideaggregate/td-p/3609392: "Complete Guide" thread has no accepted solution (replies credited by name/title); the GlideAggregate docs fetch returned method-level content from a 27-minute page — quotes from the fetched portion; no fetched source states a specific query-in-loop slowdown factor — direction claimed, not a multiplier.

---

## Module 12 — UI Builder: a Dispatch Order workspace page (restored module)

**Sources (all fetched this run):**
1. UI Builder — https://www.servicenow.com/docs/r/application-development/ui-builder/ui-builder-overview.html (Release: Brazil, Updated September 10, 2026): "UI Builder is a web user interface builder. Use UI Builder to build pages for App Engine Studio generated workspaces or custom web experiences using Next Experience Components and custom web components." "Create or customize pages for workspace experiences." "A page is a collection of components that make up a workspace user interface." Role: "Note: The ui_builder_admin role is required to complete tasks in UI Builder." Scope limit: "Note: UI Builder isn't yet capable of building or configuring ServiceNow base system service portals, such as the Employee Center. For service portals, continue to use the Service Portal Designer."
2. Create a page in UI Builder — https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html (Release: Brazil, Updated September 10, 2026): "Before you begin" — "Role required: ui_builder_admin". "Navigate to All > Now Experience Framework > UI Builder." "Open an experience to work in or create an experience by selecting Create > Experience." URL path: "The path can include digits (0-9), letters (A-Z, a-z), and a few special characters (\"-\", \".\", \"_\", \"~\"), with the words separated by a forward slash or hyphen." Required parameters: "A required parameter is a piece of data that your page requires, such as a sys_id, table, or query. Required parameters are useful for components, because they can bind to the value of the required parameter."
3. UI Builder Essentials: Troubleshooting Guide — https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130 (03-04-2025; ServiceNow Employee post by Maria Gabriela; body states "Authored by: @John McCormick1"): cache first — "If at any point something isn't working, try clearing the UI Builder cache." and "Within your page variant editor in UI builder, click the hamburger menu icon at the top-left, go to Developer > Clear UI Builder Cache. You can also try clearing your local browser cache." Tools — "the Next Experience Developer (NED) tools Chrome Extension. These tools can help you investigate through your experience and pages and track performance, events, and more." Missing event — "The event is missing because a sys_ux_event record was not captured with changes made to the page and might be in a different instance and can be brought over." Data resource ACLs — "Some data resource types (TRANSFORM and COMPOSITE) require an ACL to be created to allow them to be executed and that is why the error is showing." Unconfigured — "Some data resources require a minimum set of Input Properties to be populated before they will execute."
4. UI Builder dropdown GlideAjax thread — https://www.servicenow.com/community/developer-forum/ui-builder-dropdown-component-item-script-glideajax-undefined/td-p/3284898 (Jun 9, 2025, ACCEPTED SOLUTION by Maik Skoddow, Tera Patron — quote reproduced in Module 3, error #2; cited here because it decides this module's data path)

**What I'm building and why:** The presentation layer of this guide's app — a UI Builder page bound to the Dispatch Order table (`x_demo_dispatch_order`, Module 1) so agents see record data in a Next Experience workspace, completing data → logic → UI.

**Ordered build steps:**
1. Confirm you have the required role — https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Before you begin" — "Role required: ui_builder_admin".
2. Navigate exactly as documented — https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Navigate to All > Now Experience Framework > UI Builder."
3. Create the experience — "Open an experience to work in or create an experience by selecting Create > Experience."
4. https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Select the Create new page in the center of the screen." then https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Select Create from scratch instead."
5. Name the page; set the URL Path (default path generated from the name; https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "The path is required and must be unique"), following the quoted character rules.
6. Select a URL page type (None, Dashboard, Knowledge base, List, Record page — https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Pages assigned a type other than None are filtered more effectively in the experience view.")
7. Add required parameters: `table` and `sys_id` — https://www.servicenow.com/docs/r/application-development/automated-test-framework-atf/atf-tut-build-first.html / https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Click + Add." "Enter the required parameters for your page."
8. Finish the variant wizard (name → optional audience → https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Select Continue." → responsive choice → https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Select Create to create your blank page.") and place record-aware components that bind to those parameters. Component-level configuration is deliberately uncited: the components docs page was not fetched, so no claims are made about any specific component here.
9. Save, open the page's URL as an end user, verify against the test below.

**Common errors (exact symptoms + fixes):**
1. **GlideAjax undefined inside a UI Builder item script** — Jun 9, 2025, ACCEPTED (https://www.servicenow.com/community/developer-forum/ui-builder-dropdown-component-item-script-glideajax-undefined/td-p/3284898; quote in Module 3 error #2). Symptom (verbatim): "In the script, I'm attempting to make a GlideAjax call but GlideAjax is underscored in red and said to be undefined inside the script." Cause (verbatim): "GlideAjax is not available in the UI Builder scripting context." Fix: drive page data through bound parameters and data resources — which is why step 7 binds `table`/`sys_id` rather than calling platform GlideAjax.
2. **Event never fires / payload missing** — Mar 4, 2025 blog (ServiceNow Employee; blog guidance, no accepted-solution badge). Symptom check (verbatim): https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "Check to see if the event fired and if it had the appropriate payload fields that you expected." Cause (verbatim): https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "The event is missing because a sys_ux_event record was not captured with changes made to the page and might be in a different instance and can be brought over." Fix (verbatim): https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "Sometimes deleting the event mapping and re-adding it can fix issues that are harder to see by starting over." First move, always (verbatim): "If at any point something isn't working, try clearing the UI Builder cache."
3. **Data resource preview fails** — same blog. Fix for TRANSFORM/COMPOSITE (verbatim): https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "Some data resource types (TRANSFORM and COMPOSITE) require an ACL to be created to allow them to be executed and that is why the error is showing." — article proceeds: create an ACL of type `data_broker` on `sys_security_acl`, then paste the data resource's sys_id into the ACL's Name field. https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "unconfigured" message (verbatim): https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "Some data resources require a minimum set of Input Properties to be populated before they will execute."

**Best practices (tied to fetched pages):** clear the cache first, then NED tools — https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: the Next Experience Developer (NED) tools Chrome Extension… track performance, events, and more. Stay in scope — https://www.servicenow.com/docs/r/application-development/ui-builder/ui-builder-overview.html: "UI Builder isn't yet capable of building or configuring ServiceNow base system service portals, such as the Employee Center." Bind required URL parameters instead of scripting platform calls — https://www.servicenow.com/docs/r/application-development/ui-builder/create-page.html: "Required parameters are useful for components, because they can bind to the value of the required parameter." Remember the role gate on every task: ui_builder_admin.

**How to know it works:** Open the page URL with a real `table` + `sys_id` of a Dispatch Order → that record's data renders; swap to another sys_id → the other record renders. From the editor, Stage preview matches the end-user view. Reopen after https://www.servicenow.com/community/next-experience-blog/ui-builder-essentials-troubleshooting-guide/ba-p/3191130: "Developer > Clear UI Builder Cache" — page still loads (no stale bindings).

**Time estimate:** 60–90 minutes. **PDI friendly:** yes — one instance; the only prerequisite is the ui_builder_admin role (admin has it by default).

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** no components-docs page was fetched, so component-level guidance is excluded; thread #4's quote lives in Module 3 rather than being quoted twice; the blog carries no accepted-solution mechanic — its lines are blog guidance credited to a ServiceNow Employee post authored by @John McCormick1.

---

## Module 13 — SLAs on Dispatch Orders: attach, track, breach

**Sources (all fetched this run):**
1. Create an SLA definition — https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html (Release: Brazil, Updated September 10, 2026): "Create SLA definitions to set the conditions under which SLAs attach, pause, stop, and reset on task records." "Before you begin" — "When you create an SLA definition, avoid using dot-walked fields that change frequently in any of the SLA conditions (start, stop, pause, and reset)." "Role required: admin". "Navigate to All > Service Level Management > SLA > SLA Definitions." Table field: "Table that determines the records tracked by the SLA. SLAs can be defined for any table that extends the task table, including incident, change request, and service catalog tasks." Duration: "The length of time the SLA runs before it is marked Breached." Schedule note: "The number of days specified in this field is converted to 24-hour blocks. If the Schedule field identifies a schedule with eight-hour days, a duration of 1 Day sets the SLA to breach three business days later." Default cancel behavior: "Start conditions are not met option: If one or more of the specified start conditions change, the SLA is canceled. This option is selected by default." Debugging: "Option to activate debug logging for this SLA definition only." — "The debug logging information includes details of the conditions that have matched or not matched, and the before and after values for the task SLA and task records." Deep walks: "Note: The deeper the dot walk, the more business rules are required."
2. SLA is not being breached! — https://www.servicenow.com/community/itsm-forum/sla-is-not-being-breached/td-p/3450773 (12-16-2025; NO accepted solution; OP Maham Tahir, Mega Guru; replies by kaushal_snow, Giga Sage and AndersBGS, Tera Patron/MVP 2025)
3. SLA Breach Not Recalculating When Task Due Date Is Updated — https://www.servicenow.com/community/creator-studio-forum/sla-breach-not-recalculating-when-task-due-date-is-updated/m-p/3483785 (02-06-2026; **ACCEPTED SOLUTION** by the OP Javeria, Tera Guru, posted 02-09-2026)

**What I'm building and why:** An SLA on the Dispatch Order table — the measurement layer of the app: Sev1 dispatches must attach, track elapsed time, and breach visibly. It only works because Module 1 made the table extend Task (https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "SLAs can be defined for any table that extends the task table" — the promise Module 1's table note foreshadowed).

**Ordered build steps:**
1. Confirm `x_demo_dispatch_order` extends Task (Module 1) — without that, the Table field will not offer it: https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "SLAs can be defined for any table that extends the task table, including incident, change request, and service catalog tasks."
2. Navigate as documented — https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "Navigate to All > Service Level Management > SLA > SLA Definitions." → https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_DefineACatalogItem.html: "Select New." (https://www.servicenow.com/docs/r/servicenow-platform/service-catalog/t_DefineACatalogItem.html: "Role required: admin".)
3. Fill the form: Name (e.g., Sev1 response), Type SLA, Table = Dispatch Order, Duration type *User specified duration*, Duration = a short test value — knowing (https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html) "The number of days specified in this field is converted to 24-hour blocks. If the Schedule field identifies a schedule with eight-hour days, a duration of 1 Day sets the SLA to breach three business days later."
4. Start condition: `Severity is Sev1`. Leave https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "When to cancel" at the default and understand it — https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "Start conditions are not met option: If one or more of the specified start conditions change, the SLA is canceled. This option is selected by default."
5. Optionally tick **Enable logging** for the first runs — https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "Option to activate debug logging for this SLA definition only."
6. Insert a Sev1 Dispatch Order (Module 1 seed script, adjusted) and inspect the resulting task_sla row.

**Common errors (exact symptoms + fixes):**
1. **SLA cancels instead of breaching** — Dec 16, 2025, NO accepted solution. Symptom (verbatim): https://www.servicenow.com/community/itsm-forum/sla-is-not-being-breached/td-p/3450773: "The problem we're facing is regarding SLA is not being breached rather cancelled." Best explanation (kaushal_snow, Giga Sage, not accepted; OP later said her checks still didn't resolve it — unconfirmed): https://www.servicenow.com/community/itsm-forum/sla-is-not-being-breached/td-p/3450773: "the SLA engine will cancel an SLA whenever its start condition is no longer true (default behavior) or its cancel condition triggers before the breach time is reached" — so https://www.servicenow.com/community/developer-articles/getreference-that-shortcut-you-regret-later/ta-p/3394743: https://www.servicenow.com/community/itsm-forum/sla-is-not-being-breached/td-p/3450773: "you must verify that your definitions are set correctly (e.g., start condition stays true until breach, appropriate pause/stop conditions, schedules, etc...)".

**Best practices (tied to fetched pages):** keep conditions off volatile dot-walks — https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "avoid using dot-walked fields that change frequently in any of the SLA conditions (start, stop, pause, and reset)" (and https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "The deeper the dot walk, the more business rules are required."). Make sure the start condition can stay true through the whole window (kaushal's unconfirmed-but-sensible check, error #1). If tasks' due dates are editable, configure the reset condition up front (error #2's accepted fix). Use Enable logging on first runs — it reports https://www.servicenow.com/docs/r/it-service-management/service-level-management/t_CreateAnSLADefinition.html: "the conditions that have matched or not matched".

**How to know it works:** Sev1 Dispatch Order → a task_sla row attaches in a running state; a Sev2/Sev3 order → no task_sla row for this definition. With Enable logging on, the log confirms matched conditions for the Sev1 and non-match for the Sev2. Error #2's pattern: change the order's due date, a fresh task_sla attaches per the accepted fix (or the definition's reset condition fires) — nothing silently stalls on a stale breach date.

**Time estimate:** 45–90 minutes. **PDI friendly:** yes — service-level management is platform-level and works on a PDI; only caveat: a real multi-day 8x5 schedule needs time to demonstrate a breach (use a short duration instead).

**People warned against:** None found. Reddit was not used (network blocked).

**Honesty notes:** thread #1 has no accepted solution and OP's follow-up says checks didn't resolve it — the reply is credited as unconfirmed guidance, not a fix; thread #2's accepted solution is the OP's own KB-based fix (KB0780829 linked in the thread but not fetched — not cited); the Module 1 promise that the table https://www.servicenow.com/docs/r/application-development/system-update-sets/customizations-tracked-update-sets.html: "extend the Task table" is this guide's own design choice, stated as such in Module 1.

---

All planned modules (1–13) are complete.
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

The full Module 17 body, its code blocks, its error table, its proof table, and its design note are not present in the visible conversation history.

## Module 18. Flow Designer depth

**MISSING FROM HISTORY**

The full Module 18 body, its code blocks, its error table, its proof table, and its design note are not present in the visible conversation history.

## Module 19. Security depth

**MISSING FROM HISTORY**

The full Module 19 body, its code blocks, its error table, its proof table, and its design note are not present in the visible conversation history.

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

3. Enable inbound client credentials on the PDI. Create the system property `glide.oauth.inbound.client.credential.grant_type.enabled` and set it to `true`. The Client Credentials page https://www.servicenow.com/docs/r/platform-security/authentication/client-credentials.html: "Inbound client credentials grant type is a capability that can be controlled through a system property. By default the system property is false." The setup step reads: "Add the OAuth Application User field to the OAuth Entity form." If the field is not on your form, add it through the form layout; do not assume the field exists on every release.

4. Create an OAuth application registry record for the client. Use a redirect URL that you control even if you are testing client credentials, because the same record is often reused for authorization code later. Record the client ID and client secret in a password manager or your local untracked notes, not in the update set.

Token request (curl — written by me, not fetched; test it):

```
curl -s -X POST "https://<instance>.service-now.com/oauth_token.do" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>"
```

The response is the authority on whether a refresh token was issued. Do not build a refresh path just because the authorization code flow returns one. The token management page https://www.servicenow.com/docs/r/platform-security/authentication/t_ManageTokens.html: "Access Token: By default, an instance issues access tokens with a 30-minute lifespan in the scenario where the instance is the OAuth provider." and "Refresh Token: By default, an instance issues refresh tokens with a 100-day lifespan in the scenario where the instance is the OAuth provider."

5. Test the token before writing any integration code. A successful token response is not proof that the token can read the target table. Call the Table API with the token and check the HTTP status and the returned record count.

6. Build pagination against the Table API. Use `sysparm_limit` and `sysparm_offset`, and always add a stable order such as `ORDERBYsys_id`. Without an order, offset paging can skip or repeat records. The Table API page https://www.servicenow.com/docs/r/api-reference/rest-apis/c_TableAPI.html: "For requests that exceed this number of records, use the sysparm_offset parameter to paginate record retrieval. This limit is applied before ACL evaluation."

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

7. Build the retry path in Flow Designer, not in a tight server-side loop. Create a connection and credential alias, add a REST step, and attach a retry policy. The Retry policy page https://www.servicenow.com/docs/r/build-workflows/workflow-studio/retry-policy.html: "You can only create retry policies for JDBC, REST, and SOAP steps." Strategies include "Fixed Interval: Option to specify a fixed time interval after which a retry attempt should be made." and "Option to specify a retry attempt based on the date and time value returned in the Retry After header value of the HTTP request."

Flow Designer action input (written by me, not fetched; test it):

```
Connection: R3 OAuth
Method: GET
Resource path: /api/now/table/incident
Query: sysparm_limit=200&sysparm_offset=0&sysparm_query=ORDERBYsys_id
Retry policy: R3 5xx and 429, exponential backoff, interval 5s, count 4
```

Do not retry a non-idempotent POST blindly. Retry reads, idempotent upserts, and requests that carry an idempotency key. A 429 is a retry signal, not a reason to increase concurrency; the semaphore pools performance metrics page https://www.servicenow.com/docs/r/impact/io-semaphores-performance-metrics.html: "By default the default semaphore pools request queue can hold a maximum of 150 transactions. If the queue fills, additional transactions will be rejected with a 429 error indicating that the transaction could not be processed (and potential loss of service)."

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

The transformation event page https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/r_MapWithTransformationEventScripts.html lists `onStart`, `onBefore`, `onAfter`, and `onComplete`, and warns that "It is important to note that what these objects refer to varies depending on the context of the event in which they are referenced." In `onBefore`, `target` may not yet be the final persisted record; do not treat it as one.

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

- Use the authorization code flow when a user must consent and revoke access; use client credentials for server-to-server inbound integrations. The authorization code flow page https://www.servicenow.com/docs/r/platform-security/authentication/c_OAuthAuthorizationCodeFlow.html: "The authorization code framework gets the access token through the authorized URL that the user configures rather than requiring the user to enter a username/password."
- Treat access and refresh tokens as credentials. The token management page https://www.servicenow.com/docs/r/platform-security/authentication/t_ManageTokens.html: "Access Token: By default, an instance issues access tokens with a 30-minute lifespan in the scenario where the instance is the OAuth provider." and offers "Click Revoke Access to prevent access to the restricted resource." Do not log token values.
- Page with a stable order and a limit. The Table API page https://www.servicenow.com/docs/r/api-reference/rest-apis/c_TableAPI.html: "For requests that exceed this number of records, use the sysparm_offset parameter to paginate record retrieval. This limit is applied before ACL evaluation."
- Use Data Stream actions for responses larger than 10 MB or genuinely paginated APIs, and remember they require an Integration Hub subscription. The Data Stream page https://www.servicenow.com/docs/r/integrate-applications/integration-hub/data-stream-actions.html: "Data Stream actions require an Integration Hub subscription." For retries it states "For Data Stream actions, the maximum time interval is 30 seconds." and "Use the Fixed Interval retry strategy for Data Stream actions."
- Retry transient failures only. The Retry policy page https://www.servicenow.com/docs/r/build-workflows/workflow-studio/retry-policy.html: "You can only create retry policies for JDBC, REST, and SOAP steps." and "Option to exponentially increase the time interval for the subsequent retry attempts."
- Put file content last in a multipart upload. The Attachment API page https://www.servicenow.com/docs/r/api-reference/rest-apis/c_AttachmentAPI.html: "When using multipart POST, ensure that the file content is contained in the final part of the message only." and "Earlier parts should contain only metadata such as table name and record sys_id."
- Validate in `onBefore`, do side effects in `onAfter`. The transformation event page https://www.servicenow.com/docs/r/integrate-applications/system-import-sets/r_MapWithTransformationEventScripts.html: "Transformation events occur during the process of transforming an import set table onto a table." and "It is important to note that what these objects refer to varies depending on the context of the event in which they are referenced."
- Keep MID Server traffic outbound. The connection prerequisites page https://www.servicenow.com/docs/r/servicenow-platform/mid-server/c_MIDServerConnectionPrerequisites.html: "The MID Server communicates securely on port 443 to the instance and requires no inbound connections." The ECC queue page https://www.servicenow.com/docs/r/servicenow-platform/mid-server/ecc-queue-mid-server.html: "The MID Server subscribes to messages published by the Asynchronous Message Bus (AMB), which notifies the MID Server that it has pending task records in the ECC Queue."
- Expect 429 under load. The rate-limit article https://www.servicenow.com/docs/r/api-reference/rest-api-explorer/inbound-REST-API-rate-limiting.html: "To prevent excessive inbound REST API requests, set rules that limit the number of inbound REST API requests processed per hour." and "The status code of a denied request is 429 Too Many Requests." Back off rather than adding parallel calls.

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

1. Decide which reporting surface you are on. The Create a report page https://www.servicenow.com/docs/r/now-intelligence/reporting/t_CreateYourOwnReport.html notes that "On net new Brazil instances and instances migrated to Platform Analytics experience, Reporting functionality is replaced with Data Visualizations." If your PDI shows `All > Reports > Create New`, you are in the legacy Report Designer. If it shows Data Visualizations, use `All > Platform Analytics > Library > Data Visualizations > New` and translate the steps below. Do not mix the two surfaces in one screenshot set.

2. Create the report. Use the Incident table. Set conditions: `Active is true` and `Assignment group is not empty`. Group by `Assignment group`. Set the aggregation to `Count`. Choose a bar visualization. The Aggregation in reporting page https://www.servicenow.com/docs/r/now-intelligence/reporting/aggregation-reporting.html lists count, count distinct, average, sum, minimum, maximum, and standard deviation, and states "Aggregation is not supported on histogram, pareto, single score, box, calendar, and list reports."

Report condition (encoded query — written by me, not fetched; test it):

```
active=true^assignment_groupISNOTEMPTY^ORDERBYassignment_group
```

3. Run the report and record the total. Then remove the group-by and compare the ungrouped count with a list view count under the same conditions. If the numbers differ, check ACLs, the report’s filter, and whether the report is using a database view rather than the base table.

4. Save the report as `R3 Open Incidents by Assignment Group`. Use the Sharing option to make it visible to `Everyone` or to a specific group. The Create a report page https://www.servicenow.com/docs/r/now-intelligence/reporting/t_CreateYourOwnReport.html documents the Sharing options: "Options are Me, Everyone, and Groups and Users."

5. Create the dashboard. Go to `All > Platform Analytics > Library > Dashboards` and create `R3 Service Ops`. Add the report or its data visualization. Add a filter for `Assignment group` if you want the dashboard to be interactive. Keep the first version to one tab and fewer than ten elements; a dashboard that takes more than a few seconds to render is a debugging problem, not an executive view.

6. Share the dashboard. Open the dashboard, use `More actions > Share`, and add a group as viewer. The Share a dashboard page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/t_ControlAccessToADashboard.html states "Users with the admin role can share any dashboard they can access." and warns "Reports and other data visualizations on dashboards are subject to report_view ACLs and may not be visible to all users you share a dashboard with."

7. Create the Performance Analytics indicator source. Use the Incident table with conditions `Active is true` and `Priority is 1 - Critical`. Name it `R3 Open Critical Incidents Source`. The Create an automated indicator page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/t_CreateAnAutomatedIndicator.html: "You must have a suitable indicator source, as explained in Automated indicators." and "You must have a subscription for Performance Analytics to create indicators." Do not assume PA is available on every PDI.

Indicator source condition (written by me, not fetched; test it):

```
active=true^priority=1
```

8. Create the automated indicator. Use `Count` as the aggregate, set `Direction` to `Minimize`, and attach the indicator source. The Indicators page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/c_Indicators.html: "Indicators that are based directly on indicator sources are the most common type of indicator." and "Manual indicators are meant to be used primarily for testing purposes." Set the direction deliberately: a cost or open-incident count should normally be minimized.

9. Add a breakdown by `Assignment group`. If you are using the newer Platform Analytics experience, use the breakdown definition and the indicator’s breakdown settings. The Scorecard breakdown widget page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/create-scorecard-widget.html: "This feature is deprecated, meaning it is in maintenance mode and is no longer being augmented or changed." and "Scorecard visualizations show the trends by element for one breakdown applied to one indicator." Do not present it as the default modern experience.

10. Run the data collection job. Open the indicator and use the related links to run collection, or wait for the scheduled job. Then open KPI Details and confirm there is a score for today. If there are no scores, check the indicator source conditions, the collection job, and the indicator’s time zone. Do not “fix” missing scores by creating a manual score and calling it automated.

11. Optional: enable data snapshots only if you have RaptorDB Professional and a non-domain-separated instance. The Data snapshots page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/multi-level-breakdowns.html: "Data snapshots requires that your instance uses the RaptorDB Professional database." and "Domain-separated instances are not supported." The Performance Analytics subscription page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/c_PremiumPerformanceAnalytics.html adds that without a subscription "you cannot activate Data snapshots." Scripted indicators do not support multiple breakdowns. If those conditions are not met, stay with two breakdown levels and say so.

12. Record the difference between the report and the indicator. A report is a query run at view time. An indicator is a score collected on a schedule and stored over time. A report can show the current count; an indicator can show whether the count is improving. That distinction belongs in the dashboard’s description, not only in your head.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Widget cancelled - maximum execution time exceeded"` | A report on the `task` table rendered in a dashboard widget | https://www.servicenow.com/community/developer-forum/widget-cancelled-maximum-execution-time-exceeded-quot/m-p/3170494 | No accepted solution. The replies suggest splitting the report by task type and checking slow queries/explain plans. |
| No error string. Dashboard renders blank after deployment. | Platform Analytics dashboard moved to test | https://www.servicenow.com/community/platform-analytics-forum/dashboard-is-showing-empty-in-test-environment-post-deployment/m-p/3403341 | No accepted solution. The suggestions are to include all components in the update set and clear `par_dashboard_cache`. |
| No error string. Share dialog returns only 20 name matches. | Sharing a PA dashboard when many users have the same name | https://www.servicenow.com/community/sysadmin-forum/sharing-platform-analytics-dashboard-while-multiple-users-have/td-p/3415149 | Yes. The accepted reply points to `pa_dashboards_permissions` and having an admin set the user. |

Before changing the dashboard, verify that the dashboard record, tabs, elements, data visualizations, indicators, and breakdowns all traveled in the update set. Then clear the cache and re-login. Do not clear the cache as the first move; it hides a missing-component problem for one deploy and it comes back.

## Sourced best practices

- Use aggregation for the question you actually have. The Aggregation in reporting page https://www.servicenow.com/docs/r/now-intelligence/reporting/aggregation-reporting.html lists count, count distinct, average, sum, min, max, and standard deviation, and states "Aggregation is not supported on histogram, pareto, single score, box, calendar, and list reports." If you need a single score, use a single score visualization or PA, not a list report with a sum column bolted on.
- Share the dashboard, not just the report. The Share a dashboard page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/t_ControlAccessToADashboard.html warns "Reports and other data visualizations on dashboards are subject to report_view ACLs and may not be visible to all users you share a dashboard with."
- Share to groups or roles, not named individuals, when the audience is a team. The Share a dashboard page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/t_ControlAccessToADashboard.html supports users, groups, and roles; the Platform Analytics release notes https://www.servicenow.com/docs/r/release-notes/platform-analytics-experience-rn.html state "Administrators can share a Platform Analytics dashboard with all authenticated users by assigning it to the dashboard_user role, which ships with the base system."
- Measure direction. The Create an automated indicator page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/t_CreateAnAutomatedIndicator.html: "If you prefer that the score of this indicator increases or decreases over time, select Maximize or Minimize in the Direction field." Without a direction, the visualization cannot tell a good trend from a bad one.
- Use an indicator source, not a copied report condition. The Indicators page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/c_Indicators.html: "Indicators that are based directly on indicator sources are the most common type of indicator." and "Manual indicators are meant to be used primarily for testing purposes."
- Do not promise unlimited breakdowns without the prerequisites. The Data snapshots page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/multi-level-breakdowns.html: "Data snapshots requires that your instance uses the RaptorDB Professional database." and "Domain-separated instances are not supported." It covers "Automated indicators that do not use scripted aggregations (non-scripted indicators)."
- Treat the Core UI scorecard widget as maintenance mode. The Scorecard breakdown widget page https://www.servicenow.com/docs/r/now-intelligence/performance-analytics/create-scorecard-widget.html: "This feature is deprecated, meaning it is in maintenance mode and is no longer being augmented or changed." and "This feature is part of the Core UI front end. It is not supported on Platform Analytics dashboards or UI Builder pages, except to a limited extent when migrated in compatibility mode."
- For new work, prefer Platform Analytics data visualizations over legacy Report Designer. The Create a report page itself https://www.servicenow.com/docs/r/now-intelligence/reporting/t_CreateYourOwnReport.html notes "On net new Brazil instances and instances migrated to Platform Analytics experience, Reporting functionality is replaced with Data Visualizations."

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

3. For each drill, record a baseline: transaction response time, SQL count, business rule count, and the exact error text if there is one. Use `System Logs > Transactions` and open the transaction record. The Transaction logs page documents the response time, SQL count, business rule count, and business rule time fields. Source https://www.servicenow.com/docs/r/platform-security/r_TransactionLogs.html: "The transaction log records browser activity for an instance." "Number of business rules executed for this activity." Source https://www.servicenow.com/docs/r/platform-security/bp-debug-sql.html.

4. Use the Script Debugger for server-side logic. Open `System Diagnostics > Script Debugger`, set a breakpoint in the Script Include or Business Rule, and trigger the code from an interactive form. The Script Debugger page states that it only pauses scripts in interactive transactions where `GlideSystem.isInteractive()` returns true, and that scripts run in interpreted mode while debugging; strict-mode scripts may fail under the debugger. Source https://www.servicenow.com/docs/r/api-reference/scripts/script-debugger.html: "If the GlideSystem method isInteractive() returns True when running the script in context, then the Script Debugger can pause it."

5. Use the Session Log for code that you cannot breakpoint, especially client-callable Script Includes and UI interactions. The Script Debugger page describes the Session Log for users with `log_debugger`, with default limits of 100 transactions and 10,000 messages, configurable through `glide.debugger.log.transaction.count` and `glide.debugger.logmessageslimit`. Source https://www.servicenow.com/docs/r/api-reference/scripts/script-debugger.html: "By default, 100 transactions and 10,000 messages appear on the Session Log tab."

6. Use transaction logs to find the expensive transaction before you read the code. Sort by response time, then inspect SQL count and business rule count. A high SQL count points to a query-in-a-loop or a missing index; a high business rule count points to recursion or a rule that runs on every update.

7. Use slow queries for database cost. Go to `System Diagnostics > Stats > Slow Queries`, open a query, and use `Explain Plan` and `Index Suggestion`. The Use a slow query log page states that the log aggregates similar queries and reports data where total execution time exceeds 5 seconds. The Slow queries and SQL debugging page lists common causes: too many OR conditions, a non-domain-path query method, and a missing index. Source https://www.servicenow.com/docs/r/platform-administration/platform-performance/t_UseASlowQueryLog.html: "The Slow Queries log aggregates data for similar queries." "The log reports data for similar queries where the total execution time exceeds 5 seconds."

8. Use node logs when the syslog message is not enough. Go to `System Logs > Utilities > Node Log File Browser`, filter by session ID or message, and inspect the thread. The Log file browser page states that logs are available only for the node you are currently logged into, that archives are created every two days, and that archives are purged after 21 days. Source https://www.servicenow.com/docs/r/platform-security/r_LogUtilities.html: "Use System Logs > Utilities > Node Log File Browser to view any system log entry." "The instance creates compressed archives of system logs every 2 days and purges log archives after 21 days."

9. Use indexes deliberately. Create an index on `u_r3_bug_lab.u_external_id` through `System Definition > Tables > [table] > Database Indexes > New`. The Create a table index page warns that field order matters, that a large table is generally around 1 TB, and that index creation on a large table should be done off-peak. Do not index every field; an index has a write cost. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateCustomIndex.html: "A large table in this case is generally considered to be any table around 1 TB in size." "The order in which you select the fields affects how the index works."

10. Use Debug Business Rule (Details) to see conditions and values. Go to `System Diagnostics > Session Debug > Debug Business Rule (Details)`, reproduce the update, and read the output. The Debugging business rules page also recommends the System Dictionary, System Log, and alert messages as complementary tools. Source https://www.servicenow.com/docs/r/api-reference/scripts/r_DebuggingBusinessRules.html: "Navigate to System Diagnostics > Session Debug > Debug Business Rule (Details)."

11. Run Instance Scan. Open Instance Scan from the filter navigator and run a full scan, then a point scan on the `x_r3_debug` application. The Explore page states that a full scan runs all active checks, a point scan targets a specific record, update set, or application, and a test scan tests one check. The `scanuser` role is required to create checks, execute scans, schedule scans, and analyze results. Instance Scan has limited domain separation support; findings are domain-separated based on the source record. Source https://www.servicenow.com/docs/r/platform-administration/instance-scan/hs-execute-scans.html: "Execute all applicable checks against a single record, update set or an application by selecting Run Point Scan." "Execute a test scan to verify if the check works as expected."

12. Fix each drill and re-measure. Do not close a drill because the error disappeared; close it when the baseline metric improved or the correct behavior is proven.

## The five seeded bug drills

### Drill 1 — Recursive Business Rule

**Symptom:** updating a `u_r3_bug_lab` record never completes, the transaction log shows an unusually high business rule count, and the browser or system log reports a stack overflow. The exact server-side wrapper text varies by release; `RangeError: Maximum call stack size exceeded` is the expected JavaScript error, but it is reconstructed here, not fetched from any community post. Confirm it by running the rule with Debug Business Rule (Details) and reading the transaction error.

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

**Fetched error body (quoted from the June 2025 thread; not written by me):** Source https://www.servicenow.com/community/developer-forum/rest-api-explorer-returns-403-forbidden-for-admin-role-on-table/m-p/3280304.

```
{
  "error": {
    "message": "User Not Authorized",
    "detail": "Failed API level ACL Validation"
  },
  "status": "failure"
}
```

This exact body is from the June 2025 thread. The thread has no accepted solution, so treat the reply as a diagnostic lead, not a proven fix. Source https://www.servicenow.com/community/developer-forum/rest-api-explorer-returns-403-forbidden-for-admin-role-on-table/m-p/3280304.

**ACL script (seeded bug 4 — written by me, not fetched; test it):**

```
answer = gs.hasRole('sn_r3_user'); // seeded bug: integration user lacks this role
```

**How to diagnose it:**

1. Impersonate the integration user and repeat the Table API call. If it still fails, the issue is the user’s roles or the ACL, not the caller’s code.
2. Open `System Diagnostics > Session Debug > Debug Security` and repeat the request. The Debugging sessions page lists Debug Security as the option that displays access-control messages. Source https://www.servicenow.com/docs/r/api-reference/scripts/c_SessionDebug.html: "Displays debugging messages for access controls."
3. Use the Access Analyzer on the table and field. Check the read ACL, the role requirement, the condition, and the `Admin Overrides` setting.
4. Confirm that the table is accessible via web services. The June 2025 thread’s later reply points to the `sys_db_object` application access setting “Allow access to this table via web services”. Source https://www.servicenow.com/community/developer-forum/rest-api-explorer-returns-403-forbidden-for-admin-role-on-table/m-p/3280304.
5. Check whether a field-level ACL is denying a field that the query selects or filters on. A table-level allow does not override a field-level deny.

**Cause:** the ACL requires a role the integration user does not have, or the table is not exposed to web services. Disabling all ACLs to “test” is not a diagnosis; it removes the evidence.

**Fix:** grant the least privilege that satisfies the integration. If the integration only reads incidents, give read access to the required fields and records, not `admin`. If the ACL is correct and the user is wrong, fix the user’s role assignment.

### Drill 5 — Wrong GlideAjax name

**Symptom:** the client script throws a `ReferenceError`, or the callback never receives a useful answer.

**Fetched error text (quoted from the March 2025 thread; not written by me, so there is nothing of mine to test in this block):** Source https://www.servicenow.com/community/csm-forum/how-to-clear-the-field-value-based-on-another-fiel/m-p/3219367.

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

The invalid-query thread’s accepted reply is useful because it does not pretend the syslog message is the whole story. The detail is in the node log. The message includes an “unknown field” that is often a field that was deleted, renamed, or referenced with the wrong table prefix. Source https://www.servicenow.com/community/developer-forum/syslog-showing-many-invalid-query-detected/td-p/3145556.

## Sourced best practices

- Use the debugger that matches the failure. The Script Debugger page covers breakpoints, step-through, expression evaluation, and call-stack inspection for server-side interactive transactions; the Session Log covers logs for business rules, Script Includes, and custom UIs. Source https://www.servicenow.com/docs/r/api-reference/scripts/script-debugger.html: "The Script Debugger enables users with the script_debugger role to debug server-side JavaScript."
- Do not expect Script Debugger to breakpoint a `Scripts - Background` script directly. The Script Debugger page states it does not directly support breakpoints in the Background module, but it does support scripts called by them. Source https://www.servicenow.com/docs/r/api-reference/scripts/scripts-background-support.html: "The Scripts - Background module does not support setting breakpoints directly in the script field."
- Read the transaction before reading the code. The Transaction logs page exposes SQL count, business rule count, business rule time, response time, and URL. That is enough to separate a database problem from a rule problem. Source https://www.servicenow.com/docs/r/platform-security/r_TransactionLogs.html: "Number of business rules executed for this activity." "Number of SQL server commands executed for this activity."
- Treat slow-query data as aggregated evidence, not a single event. The Use a slow query log page says similar queries are aggregated and only queries whose total execution time exceeds 5 seconds are reported. Source https://www.servicenow.com/docs/r/platform-administration/platform-performance/t_UseASlowQueryLog.html: "The Slow Queries log aggregates data for similar queries." "The log reports data for similar queries where the total execution time exceeds 5 seconds."
- Use Explain Plan before adding an index. The Slow queries and SQL debugging page recommends Explain Plan to see what the database is doing; the Create a table index page warns that index field order matters and that large-table index creation should run off-peak. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateCustomIndex.html: "Constructing an effective index requires specialized knowledge in database architecture."
- Check the node log when the syslog line is truncated. The Log file browser page documents the Node Log File Browser, current-node limitation, two-day archives, and 21-day retention. Source https://www.servicenow.com/docs/r/platform-security/r_LogUtilities.html: "The instance creates compressed archives of system logs every 2 days and purges log archives after 21 days."
- Use Debug Business Rule (Details), not just `gs.info`. The Debugging business rules page lists condition and value inspection as the reason to use the details module. Source https://www.servicenow.com/docs/r/api-reference/scripts/r_DebuggingBusinessRules.html: "Navigate to System Diagnostics > Session Debug > Debug Business Rule (Details)."
- Run Instance Scan as a pre- and post-upgrade habit. The Instance Scan landing page describes it as part of development operations, release management, and pre/post-upgrade work; the Explore page lists full, point, and test scans and the `scanuser` role. Source https://www.servicenow.com/docs/r/platform-administration/instance-scan/hs-execute-scans.html: "Execute all applicable checks against a single record, update set or an application by selecting Run Point Scan." "Execute a test scan to verify if the check works as expected."
- Do not over-index. The Create a table index page explicitly says constructing an effective index requires database architecture knowledge and recommends consulting a DBA if you do not have it. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateCustomIndex.html: "Constructing an effective index requires specialized knowledge in database architecture." "Therefore, consider creating indexes on larger tables only during off-peak hours."

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

Record the response time and SQL count from `System Logs > Transactions`. The Transaction logs page documents response time, SQL count, business rule count, and business rule time. Use the same user, the same filter, and the same rowcount for every run. Source https://www.servicenow.com/docs/r/platform-security/r_TransactionLogs.html: "Round trip response time for the application request, in milliseconds." "Number of SQL server commands executed for this activity."

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

The Zurich grouped-list article is relevant here: grouped lists are capped at 20 groups per page, and the `Show` setting controls rows within a group, not the number of groups. Raising `glide.db.max.aggregate.size` changes the cap globally and can hurt performance. Treat that property as a last resort, not a tuning default. Source https://www.servicenow.com/docs/r/platform-administration/r_AvailableSystemProperties.html: "Sets the maximum number of groups that a grouped report or list renders. Larger values may affect system performance." Source https://www.servicenow.com/docs/r/release-notes/brazil-security-notables.html: "See that the list currently displays '1-20 of 23 groups'."

8. Add indexes. Start with the equality filter and the common report filter. The Create a table index page warns that field order matters and that large-table index creation should run off-peak. A composite index on `(u_state, u_priority)` is useful when both are equality predicates; an index on `(u_priority, u_state)` is not the same index. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateCustomIndex.html: "The order in which you select the fields affects how the index works." "Therefore, consider creating indexes on larger tables only during off-peak hours."

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

10. Re-measure the list, report, and query. Use the same user, same filter, same rowcount, and same time window. Discard the first run after a code or index change if the cache is cold; report the median of five runs. Open the transaction record and compare response time and SQL count. Then open `System Diagnostics > Stats > Slow Queries`, find the query pattern, and use `Explain Plan` and `Index Suggestion`. The Use a slow query log page says the log reports similar queries whose total execution time exceeds 5 seconds. Source https://www.servicenow.com/docs/r/platform-administration/platform-performance/t_UseASlowQueryLog.html: "The Slow Queries log aggregates data for similar queries." "The log reports data for similar queries where the total execution time exceeds 5 seconds."

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

- Do not run free-form background scripts on production. The Scripts - Background page states the risk and offers a rollback context, a sandbox option, and a cancellation limit. Use a sub-production instance for the seed. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/background-script-recovery.html: "When you executed the script using the Scripts-Background module, the Record for Rollback? item must have been checked."
- Measure the transaction, not the stopwatch in your head. The Transaction logs page exposes response time, SQL count, business rule count, and business rule time. Compare like with like. Source https://www.servicenow.com/docs/r/platform-security/r_TransactionLogs.html: "Round trip response time for the application request, in milliseconds." "Number of business rules executed for this activity."
- Use slow-query data before adding indexes. The Use a slow query log page aggregates similar queries and reports those whose total execution time exceeds 5 seconds; the Slow queries and SQL debugging page recommends `Explain Plan`. Source https://www.servicenow.com/docs/r/platform-administration/platform-performance/t_UseASlowQueryLog.html: "The Slow Queries log aggregates data for similar queries." "The log reports data for similar queries where the total execution time exceeds 5 seconds." Source https://www.servicenow.com/docs/r/platform-security/bp-debug-sql.html.
- Respect index order and maintenance cost. The Create a table index page says field order affects how the index works, that a large table is generally around 1 TB, and that index creation on a large table should run off-peak. Source https://www.servicenow.com/docs/r/platform-administration/table-administration-and-data-management/t_CreateCustomIndex.html: "The order in which you select the fields affects how the index works." "A large table in this case is generally considered to be any table around 1 TB in size." "Therefore, consider creating indexes on larger tables only during off-peak hours."
- Know the Zurich grouped-list cap. The release notes show a grouped list paging at 20 groups, and the system property page states that `glide.db.max.aggregate.size` can raise the cap globally, with a performance cost. Source https://www.servicenow.com/docs/r/release-notes/brazil-security-notables.html: "See that the list currently displays '1-20 of 23 groups'."
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

1. Write the naming convention before creating anything. The Namespace identifier page states that scoped applications start with `x_`, that the instance customer prefix is two to five characters, and that the application ID can be up to 40 characters but is truncated for the namespace if it is longer than 18. The SDK CLI page also states that the scope name must begin with `x_<prefix>` and be 18 characters or fewer. Source https://www.servicenow.com/docs/r/application-development/c_NamespaceIdentifier.html.

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

2. Create the scoped application in ServiceNow Studio. The ServiceNow Studio page describes it as the unified environment for app files, metadata, and packaging. Create the table and fields. Add a module and a list view. Do not customize an out-of-the-box table for the lab; keep the app self-contained. Source https://www.servicenow.com/docs/r/application-development/servicenow-studio-classic/servicenow-studio-landing.html.

3. Create a dedicated private Git repository, for example `r3-lifecycle`. Generate a personal access token with read/write access to that repository only. The Link an app to source control page recommends a dedicated repository, MFA, a scoped token, network access from the non-production instance, and an email address on each developer’s `sys_user` record. Do not reuse a personal token across applications. Source https://www.servicenow.com/docs/r/application-development/servicenow-studio-classic/link-app-to-source-control.html.

4. Link the app to source control. In ServiceNow Studio, open the app, go to `App details > Source control > Link to source control`, and enter the HTTPS URL, branch `main`, credential, and default email. The Link an app to source control page states that the integration does not support linking an application on a production instance; install on production from the application repository, an update set, or Studio. If you are on a PDI and the option is missing, the June 2025 Community thread’s accepted workaround is to publish the app to an update set from the `sys_app` record; that is a workaround, not source control. Source https://www.servicenow.com/docs/r/application-development/servicenow-studio-classic/link-app-to-source-control.html.

5. Clone the repository into the ServiceNow IDE or use the SDK locally. The SDK page states that the SDK develops scoped applications in Visual Studio Code Desktop and uploads them to an instance, and that it does not support Global scope or global applications on Zurich. The Fluent page states that Fluent is a TypeScript-based DSL for `sys_metadata`, with two-way synchronization and a Record API fallback for metadata types that do not have a dedicated API. Source https://www.servicenow.com/docs/r/application-development/servicenow-studio-classic/link-app-to-source-control.html.

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

The repository requires Node v20+ and pnpm v9+. The last commit shown by the GitHub API was September 14, 2026. Do not present the sample folders as a substitute for the SDK reference; they are examples, and some samples exercise APIs that are not part of the CAD core.

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

The Using source control page lists checkout, clone, commit, branch, discard, fetch, pull, push, stage, stash, merge-conflict resolution, and commit history. It also states that only one branch per repository can be checked out on an instance or developer sandbox at a time. Do not plan a workflow that assumes two branches are checked out simultaneously on one instance. Source https://www.servicenow.com/docs/r/application-development/servicenow-ide-family-release/using-source-control-servicenow-ide.html.

8. Create an update set for a base-system change that is not part of the scoped app. For example, add a field to a form or change a label on an OOTB table. The Get started with update sets page says update sets capture and move customizations, are used for patches/changes to installed applications, and are not designed for installing applications; use the application repository or the ServiceNow Store for installation. The same page recommends keeping sets small, using a consistent migration path, previewing before commit, not deleting sets, and not backing out the Default set. Source https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html.

9. Compare update sets and applications. Use this table in your design note, not as a substitute for the docs:

| Need | Update set | Application repository |
|---|---|---|
| Change a base-system record or an installed app | Yes | Not the primary tool |
| Install a scoped app on another instance | Not designed for it | Yes |
| Version and distribute an app with entitlements | No | Yes |
| Small, reviewable change with a ticket | Yes | Possible as a release, but heavier |
| Source-level collaboration and branching | No | Git is the source system; app repo is distribution |

10. Publish the app to the application repository. The App repository page states that it is a central repository for scoped applications published by customers, that an instance must be entitled to install, and that the repository is not available for self-hosted customers. Publish a version, install it on a second instance, and record the version number. Do not call a Git push a deployment; Git stores source, the app repository distributes a packaged app. Source https://www.servicenow.com/docs/r/application-development/application-repository-self-hosted/t_PublishAppsToTheAppRepository.html.

11. Simulate a skipped change. Customize an OOTB object that the upgrade will change, then run an upgrade preview or review the upgrade history. The Resolve a skipped update page says the upgrade skips changes to objects that have a current version in the Customer Updates table, and the resolution options are retain, merge, revert, or review with no action. The Review skipped records page lists the related lists and the dispositions, including `Skipped`, `Skipped Error`, and `Skipped Manual Merge`. Source https://www.servicenow.com/docs/r/platform-administration/upgrade-management/um-resolve-skipped-update.html.

12. Resolve one skipped record each way: retain one, merge one, and revert one. Set the Resolution Status and confirm the record moves from `Skipped Changes to Review` to `Skipped Changes Reviewed`. If you review a skipped record while in the wrong application scope, you can capture items from different scopes in your update set; the Resolve page warns about this. Source https://www.servicenow.com/docs/r/platform-administration/upgrade-management/um-resolve-skipped-update.html.

13. Request a clone. The Request a clone page requires `clone_admin` on the source and `clone_admin` plus `soap` on the target, a target instance, and a clone profile. The Explore page states that cloning uses the most recent daily backup, at most 36 hours old, and that the target adopts the source instance’s version when cloning across versions. Configure exclusions and preservers deliberately; do not clone production over a target that has unique configuration you need without preserving it. Source https://www.servicenow.com/docs/r/platform-administration/t_StartAClone.html. Source https://www.servicenow.com/docs/r/platform-administration/exploring-instance-clone.html.

14. After the clone, verify the target. Check that users, groups, properties, and the source-control link are present; re-link if the clone did not preserve the credential. Run a smoke test on the app’s list, form, and one flow. A clone is not finished when the status says complete; it is finished when the app works.

15. Refactor one upgrade-hostile customization. Upgrade-safe patterns are UI Policies, Flow Designer, clear Business Rule conditions, scoped Script Includes, and supported extension points; upgrade-hostile patterns are editing OOTB scripts directly, hardcoded sys_ids, DOM manipulation, and untracked changes. Pick one direct OOTB edit in your lab, replace it with a scoped Script Include or a UI Policy, and document the before/after.

## Exact errors and accepted-solution flags

| Error text | Context | Thread | Accepted? |
|---|---|---|---|
| `Scope 88fe7645c3fd6e505c545b2ed40117c8 is not in the instance` / `Unable to create scope 88fe7645c3fd6e505c545b2ed40117c8` / `Cloning https://github.com/test-sn-2/fluent1.git failed.` / `Scope "x_140000_abc1" is invalid or already exists` | Cloning a Fluent Git repo into a second instance | https://www.servicenow.com/community/developer-forum/unable-to-clone-git-repo-in-servicenow-ide/m-p/3290332 | No accepted solution. The thread has no replies. |
| `Destination does not appear to be a Service-now.com instance` | Scheduling a clone while authenticating | https://www.servicenow.com/community/sysadmin-forum/scheduling-a-clone-destination-does-not-appear-to-be-a-service/m-p/3340353 | Yes. The accepted reply points to clone ID roles and target `admin`; the original poster confirms roles fixed it. |
| `Clone stage changed from Applying Preservers to Intervened` | Production-to-development clone at about 66% | https://www.servicenow.com/community/upgrades-and-patching-forum/clone-from-production-to-development-fails-at-applying/m-p/3392564 | No accepted solution. The thread reports the stage and `sys_clone_log` detail but no fix. |
| No error string; “Link to Source Control” is absent | Scoped app on a PDI | https://www.servicenow.com/community/servicenow-studio-forum/cannot-use-quot-link-to-source-control-quot-need-alternative/m-p/3289514 | Yes. The accepted workaround is `Publish to Update Set` from the `sys_app` record; the app repository is described as for connected customer instances. |

The Git clone errors are not one error. `Scope ... is not in the instance` is a missing scope, `Unable to create scope` is a creation failure, and `Scope ... is invalid or already exists` is a collision. Read the first error in the IDE console, not the last line of the failed clone.

## Sourced best practices

- Use a dedicated repository per app, MFA, and a scoped PAT. The Link an app to source control page recommends all three, plus a `sys_user` email for commits. Source https://www.servicenow.com/docs/r/application-development/servicenow-studio-classic/link-app-to-source-control.html.
- Do not link source control on production. The same page states the integration does not support linking an application or customization on a production instance; deploy to production from the app repository, an update set, or Studio.
- Use source control for source, the app repository for distribution. The App repository page describes entitlements and installation; Git is not an entitlement system. Source https://www.servicenow.com/docs/r/application-development/application-repository-self-hosted/t_PublishAppsToTheAppRepository.html.
- Do not use update sets to install applications. The Get started with update sets page says to use the application repository or the ServiceNow Store instead. Source https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html.
- Keep update sets small and preview them. The Get started page recommends small/medium sets, a consistent migration path, preview before commit, and not deleting sets. Source https://www.servicenow.com/docs/r/application-development/system-update-sets/get-started-update-sets.html.
- Review skipped changes while the upgrade is fresh. The Resolve page says customized objects are skipped to preserve customizations, and the resolution statuses are explicit; the Review page lists `Skipped Error` as a disposition that needs attention. Source https://www.servicenow.com/docs/r/platform-administration/upgrade-management/um-resolve-skipped-update.html.
- Clone from a recent backup with deliberate exclusions and preservers. The Explore page describes the workflow and the 36-hour backup window; the Request page lists the required roles and profile. Source https://www.servicenow.com/docs/r/platform-administration/t_StartAClone.html. Source https://www.servicenow.com/docs/r/platform-administration/exploring-instance-clone.html.
- Treat the scope name as permanent. The Namespace identifier page says the identifier cannot be changed or removed, and the SDK CLI page caps the scope at 18 characters. Source https://www.servicenow.com/docs/r/application-development/c_NamespaceIdentifier.html.

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
