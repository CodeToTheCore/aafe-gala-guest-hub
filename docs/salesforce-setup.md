# Salesforce: the gala file and notes for AAFE and Idlewild Partners

First written by Natalie Walker, product lead (Oct 20, 2026); updated Nov 6, 2026.

For AAFE and **Idlewild Partners**, the firm building and configuring AAFE's Salesforce. It describes what the stable
build gives Salesforce today, the route AAFE has chosen, and what is left to scope together. Code:
`prototype/staff-app/src/exports.ts` (the file) and `prototype/supabase/functions/_shared/salesforce.ts` (the direct
sync); technical detail: `prototype/docs/BACKEND.md` ("Demo with a stand-in Salesforce").

## Where things stand (Nov 4, 2026)

- **AAFE's Salesforce:** the gala tool works with the Nonprofit Success Pack (NPSP), decided Nov 4, 2026, since
  Give Lively's Salesforce connector runs on NPSP.
- **AAFE's route, for now:** import a CSV file from the gala tool into Salesforce, rather than connect the tool to
  Salesforce directly (AAFE's decision, Nov 2, 2026).
- **Next step:** shaping the file to AAFE's NPSP fields is to be scoped with AAFE and Idlewild after Demo Day
  (Nov 9). The column mapping below is a starting point for that conversation, not a finished import.
- **Online purchases:** Give Lively's own Salesforce app writes AAFE's online ticket and sponsorship purchases. The
  gala tool leaves those purchases out, so nothing is counted twice.

## What the stable build gives Salesforce today

| | What it is | Status |
| --- | --- | --- |
| **The Salesforce file** | Staff app, Integrations, **Download Salesforce file** (gala admins only). One row per guest who checked in at the gala, walk-ins included, with their organization and sponsor | Ready to download; the column mapping is to be confirmed against AAFE's NPSP setup |
| **The direct sync** (**Sync now**) | Sends sponsors, check, ACH and wire orders and payments, and guests with their gala-night status straight to Salesforce | Built against NPSP's field names. In the demo it sends only to a stand-in org; it isn't connected to a real Salesforce |

## Who sends what

| | Who sends it to Salesforce |
| --- | --- |
| Online tickets and sponsorships, the event's Campaign, ticket tiers, processing fees, each tier's tax-deductible amount | **Give Lively's own Salesforce app** |
| Guests who checked in at the gala, and walk-ins | **The gala tool's Salesforce file**, imported by AAFE |
| Sponsors who pay by check, ACH or wire, and those payments | Not in the Salesforce file today; a question for the scoping with AAFE and Idlewild |

The staff app's Reports page also has spreadsheet downloads (who came, sponsors and money, payments, the full guest
list). They are made for Excel or Google Sheets, not shaped for a Salesforce import.

## The Salesforce file

`salesforceCsv()` in `prototype/staff-app/src/exports.ts`, with rows from `staff_export_salesforce()` in
`prototype/database/backend.sql`. One row per person who checked in at the current gala, walk-ins included. Each
download is recorded in the change log. A demo gala's file says so on its first line and in its name; a real gala's
file starts with the column names.

A suggested mapping, for Idlewild to confirm against AAFE's NPSP setup:

| Column (as downloaded) | Salesforce field (suggested) | Notes |
| --- | --- | --- |
| First Name | Contact: First Name | Empty when the guest's name couldn't be split |
| Last Name | Contact: Last Name | Required by Salesforce; always filled (a walk-in's name as typed) |
| Email | Contact: Email | The match key for existing Contacts; empty for walk-ins |
| Title | Contact: Title | |
| Account Name | The Contact's Account, or a custom "Organization" field | The guest's organization. Where it goes is for Idlewild to decide with AAFE |
| Sponsor | Campaign Member: a custom "Sponsor" field, or leave out | Who invited the guest |
| Account ID | Leave out | Filled only after a direct sync, so empty while AAFE imports the file |
| Opportunity ID | Leave out | Filled only after a direct sync, so empty while AAFE imports the file |
| Guest Type | Campaign Member: a custom "Guest type" field | Sponsor guest, Ticket holder, Guest of AAFE, Government or VIP, AAFE staff, Event volunteer, Event staff, Walk-in |
| Checked In At | Campaign Member: Status = Attended, plus a custom "Arrived at" field | **UTC**, e.g. `2027-05-27T22:31:00+00:00` (6:31 PM in New York) |
| Check-in Method | Leave out, or a custom field | qr, search or exception |

No phones, dietary or accessibility details are in the file.

**Import:** the file was shaped for Salesforce's Data Import Wizard (Contacts and Campaign Members). Idlewild controls
how the import matches existing records.

**Two kinds of rows to leave out or match by hand:** walk-ins and guests without an email. With no email to match on,
an import would add them again each time.

## The direct sync in the code (not connected)

The stable build also has a direct sync: **Sync now** on the staff app's Integrations page (gala admins only), in
`prototype/supabase/functions/salesforce-push`. It is described here so a future technical partner knows what is in
the code; it is not part of AAFE's current route.

- **What it sends:** for the current gala, sponsors who pay by check, ACH or wire (Accounts), their orders
  (Opportunities in the gala's Campaign), the payments staff recorded, and every named guest (Contacts, and Campaign
  Members marked Registered, Attended or No Show). It leaves out Give Lively's online purchases, so nothing is counted
  twice, and it never sends phones, dietary or accessibility details.
- **Built for NPSP:** it uses NPSP's field names (`npe01__OppPayment__c` for payments, NPSP's Home, Work and Alternate
  email fields, `npo02__NumberOfClosedOpps__c`, `npe01__Do_Not_Automatically_Create_Payment__c`), which matches the
  NPSP route decided Nov 4, 2026. Idlewild would confirm them against AAFE's org before the sync is used.
- **Where it sends today:** in the demo, only to a stand-in org inside the prototype's own project. It has never sent
  to a real Salesforce. With no Salesforce keys set, **Sync now** sends nothing and says Salesforce isn't connected.
- **Duplicates:** it matches people by email, as Give Lively does, and skips guests without an email; it links a
  sponsor to an existing Account with exactly the same name and makes nothing when two match; and a run in which a
  lookup fails makes nothing.

The settings it reads (`SALESFORCE_BASE_URL`, `SALESFORCE_CLIENT_ID`, `SALESFORCE_CLIENT_SECRET`,
`SALESFORCE_SEND_PAYMENTS`, `SALESFORCE_SEND_GUESTS`) are described in `prototype/docs/BACKEND.md`. For AAFE today,
leave them unset.

## Sources

- [Give Lively: Salesforce integration logic and fields](https://www.givelively.org/resources/see-our-salesforce-integration-logic-and-fields-1-3)
- [Give Lively: campaign mapping](https://intercom.help/give-lively/en/articles/16239466-configure-campaign-mapping-in-give-lively-s-salesforce-integration)
