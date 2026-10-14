# ServiceNow administration scripts

Small, copy/paste-oriented server-side JavaScript utilities for ServiceNow background scripts, scheduled jobs, transform maps, and troubleshooting. The scripts use ServiceNow globals such as `GlideRecord`, `GlideAggregate`, `GlideDateTime`, `GlideRecordSecure`, `gs`, and (for the date helper) the server-side Java timezone bridge.

## Prerequisites

- A ServiceNow instance and an account allowed to run the relevant Background Script or Scheduled Script Execution.
- Sufficient ACL and role access to the tables queried by the selected script. Several utilities intentionally read or update administrative tables.
- A release that supports the classic server-side JavaScript APIs used here. These scripts predate scoped-app restrictions and should be reviewed before use in scoped applications.
- Copy the complete script, plus any helper functions it calls, into the same execution context. Files in `lib/` are standalone snippets rather than an npm package.

## Usage

For example, to find duplicate values in a table, open **System Definition → Scripts - Background**, paste `lib/getDuplicates.js`, and run:

```javascript
gs.print(getDuplicates("sys_user", "email", "active=true").join("\n"));
```

Expected result: the script prints one Markdown link for each duplicate group, for example `[2 duplicate found. Group by email 1](https://example.service-now.com/sys_user_list.do?sysparm_query=sys_idIN...)`. The URL uses the instance URL from `glide.servlet.uri`; no records are changed.

Other common entry points include `searchnow(...)` for a bounded cross-table search, `findInvalidEmailAddresses(false)` for a report-only email check, `validateaccess(table, sys_id, user_name)` for an impersonated ACL check, and `setUserNotificationbyEmail(emailArray, "disabled", 100)` for a bounded update.

## Script catalogue

| Script | Purpose |
| --- | --- |
| `findInvalidEmailAddresses.js` | Reports active users whose email does not match the built-in validation pattern; optionally disables notifications. |
| `findmatch.js` | Finds a target record using transform-map-style coalesce field pairs. |
| `findNotificationfromEmail.js` | Traces notification records related to an email. |
| `findInboundActionTraces.js` | Traces inbound email actions and related logs. |
| `findSourceRecordFromTarget.js` | Finds import/source records associated with a target record. |
| `getDuplicates.js` | Groups records by one or more fields and links duplicate groups. |
| `retrieveBouncedBackEmail.js` | Extracts unique addresses from bounded bounced-email searches. |
| `returnMatchesonRecord.js` | Reports fields on one record matching a regular expression. |
| `returnPrintableVariables.js` | Prints non-empty catalog variables for a record. |
| `searchnow.js` | Searches configured dictionary-backed fields across bounded tables. |
| `setDisplayTime.js`, `DateTimeUTCconverter.js` | Converts display dates using a named timezone. |
| `setUserNotificationbyEmail.js` | Enables or disables user notifications for an email list with a limit. |
| `validateaccess.js` | Impersonates a user and reports record ACL capabilities. |
| `printRecord.js`, `printLogs.js` | Formats records and filtered system logs as readable output. |
| `SSOInfo.js`, `generate_properties.js` | Reports SSO configuration and compares selected properties/scripts. |
| `ParseGlideElement.js` | Casts a GlideElement using its dictionary internal type. |

## Version and compatibility notes

- These are classic global-background-script utilities, not tested as scoped Script Includes. `Packages.java.util.TimeZone` is retained for compatibility with the original global scripts; instances that restrict Java package access must replace that bridge with a supported timezone API before use.
- The scripts use legacy Rhino-era idioms (`escape`, `Class.create`, implicit globals in older snippets, and encoded-query JavaScript dates). They are intentionally not rewritten wholesale because those changes can alter query or display behavior across family releases.
- Validate table names, encoded queries, ACLs, query limits, and update flags in a non-production instance first. Run report-only modes before enabling updates.
- `sso_lite.js` is an obfuscated/generated legacy artifact and is intentionally excluded from documentation and maintenance edits.

## Validation

There is no Node.js test suite because the code depends on ServiceNow runtime globals. Local validation checks JavaScript parsing where possible, scans project-owned functions for documentation, and excludes the obfuscated `sso_lite.js` artifact.
