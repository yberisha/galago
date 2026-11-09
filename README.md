# Galago (Chrome Extension, Manifest V3)

Galago reads ticket and engineer information from 3CIS and status information from CMS. It adds a **Live CMS Update** column to the 3CIS Task Table and locally displays CMS values for matching ticket IDs. For tickets that pass the configured CMS checks, it can submit the 3CIS pre-check form to move the ticket from **Pending Pre-check** to **Ready For Upgrade**. Its head-only, big-eyed martial-arts galago represents attentive monitoring; the **3CIS + CMS Sync** label explains the job.

## Install locally

1. In Chrome, open `chrome://extensions` and enable **Developer mode**.
2. Select **Load unpacked** and choose this folder.
3. Sign in to 3CIS and open the Task Table.
4. Sign in to CMS and open the Code Upgrade Dashboard. Leave both pages open while you want live updates.

## Behavior

- Reads the `readyForUpgrade` task data embedded in the 3CIS Task Table response and maps engineer IDs to names.
- Captures only the CMS Code Upgrade Dashboard `FetchTicketsInformation` response through fetch or XHR and repeats that read request every 30 seconds while the dashboard is open.
- Stores CMS ticket status, change status, current stage, and step locally, joined by ticket number.
- Adds those fields to the 3CIS Task Table under **Live CMS Update**, with green, yellow, red, and neutral status badges.
- Compares CMS scheduled start timestamps with the 3CIS scheduled date and start time in the browser's local timezone, and flags differences.
- Stores each distinct CMS status change in `cmsTicketStatusHistory`, up to the latest 250 changes per ticket; identical 30-second polls are deduplicated, and the oldest history entries are pruned to keep the history under a 2 MB local limit.
- Retains 3CIS ticket-to-engineer mappings for up to 24 hours after a ticket disappears from the source response.
- Shows current and retained ticket mapping counts separately in the 3CIS sync banner.
- Applies the existing 3CIS engineer-name display to CMS `Created By` cells locally.
- Automatically submits eligible 3CIS device pre-check forms with up to four concurrent background workers. A ticket must still be **Pending Pre-check**, have a CMS change status exactly **Pre-work Completed**, have CMS data no older than 90 seconds, and have no CMS failure/cancellation or prefilled issue description. The extension refreshes the Task Table once through its DataTables AJAX endpoint when the batch finishes, without reloading the page. A ticket attempt is not repeated for that record for 24 hours, including when the response is ambiguous.
- Uses the current server-side DataTable page to override older embedded ticket status data, so recently promoted records are skipped even when the table is paginated.

## Notes

- The added column and name replacement are browser-only display changes. CMS remains read-only. The guarded pre-check submission is the only server write and updates 3CIS only after the checks above pass.
- Keep both authenticated pages open for cross-site live updates. The CMS poll uses the request made by the signed-in page.
- If the CMS response shape or either site's table headers change, update the parsing logic in `cms-api-bridge.js`, `source-datatable-bridge.js`, or `config.js`.
# galago
