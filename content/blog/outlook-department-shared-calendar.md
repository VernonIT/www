---
title: "Setting Up a Department Shared Calendar in Outlook / Exchange"
date: 2026-03-17
draft: true
tags: ["microsoft-365", "outlook", "exchange", "calendar", "how-to"]
categories: ["productivity"]
description: "A complete setup and administration guide for a shared Outlook calendar that tracks PTO and business travel across your department."
showToc: true
cover:
  image: "https://images.unsplash.com/photo-1506784983877-45594efa4cbe?w=1200&q=80"
  alt: "Open calendar on a desk"
---

A shared department calendar gives everyone a single, visible source of truth for approved time off and business travel. This guide walks your management team through creating and administering one in Microsoft Outlook / Exchange — from initial mailbox setup through the day-to-day request and approval workflow.

**Who this is for:** A department admin or manager standing up a shared PTO and travel calendar for the first time, or anyone inheriting an existing one and trying to understand how it works.

---

## Overview

The design behind this setup:

- **One shared mailbox** (e.g., `dept-calendar@company.com`) owns the calendar — one place to subscribe to, one place to manage.
- The **Location field** on each event identifies which manager approved it, enabling easy filtering by team.
- Employees submit requests via standard Outlook meeting requests; managers approve and post the final event to the shared calendar.
- Each manager holds **Editor** permissions and manages only their own team's events.
- All department staff have **Reviewer** (read-only) access so anyone can check availability at a glance.

---

## Prerequisites

Before you start, confirm:

- Your organization runs Microsoft Exchange Server 2016+ or Microsoft 365 (Exchange Online).
- An **Exchange Administrator** (IT admin) is available for the initial mailbox creation steps.
- All managers and employees have active Exchange / Microsoft 365 accounts.
- One person has been designated as **Calendar Owner** — a department admin in a stable, long-term role is ideal. The Owner holds full control of the shared mailbox calendar and can grant or revoke permissions for all other users.

---

## Calendar Architecture

### Shared Mailbox (Recommended)

A shared mailbox calendar:

- Keeps all events in one place.
- Supports granular per-user permissions (Reviewer, Editor, Owner).
- Works with Outlook desktop, Outlook Web App (OWA), and Outlook mobile.
- Does **not** consume a paid Microsoft 365 license when configured as a shared mailbox.

### Permission Levels

| Role | Permission Level | What They Can Do |
|---|---|---|
| Calendar Owner | Owner / Full Access | Create, edit, delete all events; manage permissions for others |
| Manager | Editor | Create, edit, and delete events; view all events |
| All Department Staff | Reviewer | View all approved events; no create or edit rights |
| Co-Manager (optional) | Editor | Same as Manager; used when covering another team temporarily |

---

## Step-by-Step Setup

### Step 1 — Create the Shared Mailbox *(IT Admin)*

1. Sign in to `admin.microsoft.com` with admin credentials.
2. Navigate to **Teams & groups → Shared mailboxes**.
3. Click **+ Add a shared mailbox**.
4. Enter a display name (e.g., "Department Calendar") and email address (e.g., `dept-calendar@company.com`).
5. Click **Save changes**.
6. Open the new mailbox record, click **Edit** under Members, and add the Calendar Owner with Full Access.

> The shared mailbox appears automatically in Outlook for users with Full Access. Others will need to open it manually or be granted Folder permissions (covered in Steps 3 and 4).

### Step 2 — Configure the Calendar Owner *(IT Admin + Owner)*

1. In Exchange Admin Center (EAC), go to **Recipients → Shared** and open the new mailbox.
2. Under **Mailbox delegation**, assign the Calendar Owner both **Full Access** and **Send As** permissions.
3. The Calendar Owner opens Outlook desktop and confirms the shared mailbox appears in the left folder pane. Allow up to 60 minutes for propagation.
4. The Owner right-clicks the calendar inside the shared mailbox and renames it — e.g., "Dept PTO & Travel."

### Step 3 — Grant Manager (Editor) Permissions *(Calendar Owner)*

Perform these steps in Outlook desktop for each manager:

1. Switch to **Calendar view**.
2. In the left pane, right-click the shared calendar and select **Properties → Permissions tab**.
3. Click **Add**, search for the manager's name, and click Add → OK.
4. Set the **Permission Level** dropdown to **Editor**.
5. Click OK. Repeat for each manager.

> **Important:** Outlook's Editor permission is calendar-wide — a manager can technically create, edit, and delete *any* event, not just their own team's. The Location field convention (Section 5) provides organizational clarity but doesn't enforce boundaries. Set a clear norm at your team kickoff: each manager only creates or edits events for their own direct reports.

### Step 4 — Grant Staff (Reviewer) Permissions *(Calendar Owner)*

Repeat the same steps from Step 3 for all department employees, but select **Reviewer** as the Permission Level.

- For large teams, ask IT to create an Exchange Distribution Group for the department, then add the group to calendar permissions as Reviewer in a single step.
- Verify by having a staff member open the shared calendar and confirm they can view but not edit events.

### Step 5 — Subscribe to the Shared Calendar *(All Users)*

Each user must open the shared calendar in their own Outlook.

**Outlook Desktop:**

1. Go to **File → Open & Export → Other User's Folder**.
2. Search for the shared mailbox name and select **Calendar**.
3. The calendar will appear in your left pane under **Other Calendars**.

**Outlook Web App (OWA):**

1. Click the Calendar icon, then **Add calendar → Add from directory**.
2. Search for the shared mailbox and click **Add**.

---

## Event Naming & Location Field Conventions

Consistent naming is critical to making the calendar useful at a glance.

### Location Field = Manager Identifier

Set the Location field of every event to the approving manager's last name, or a short team code. This allows anyone to filter or search the calendar by manager or team.

**Example:** Manager Sarah Johnson approves PTO for Mark Davis. She creates the event and sets **Location = "Johnson"**. Any colleague can filter the calendar by "Johnson" to see all events Sarah has approved.

### Event Title Format

| Event Type | Title Format | Example |
|---|---|---|
| PTO / Full Day | [Last Name] – PTO | Davis – PTO |
| PTO / Partial Day | [Last Name] – PTO (AM) or (PM) | Davis – PTO (PM) |
| Business Travel | [Last Name] – Travel: [Destination] | Davis – Travel: Chicago |
| Work From Home (optional) | [Last Name] – WFH | Davis – WFH |

### Event Description Field

Use the Description field to capture approval metadata:

```
Approved by: [Manager Name] | Employee: [Full Name] | Type: PTO / Travel | Submitted: [Date] | Notes: [Optional]
```

---

## Employee Request & Approval Workflow

### Employee Submits a Request

1. Employee opens a new **Meeting Request** in Outlook.
2. Adds their manager as the required attendee, and optionally a coverage contact as optional.
3. Sets the date(s) and a descriptive subject line, e.g., "PTO Request – June 12–14" or "Travel Approval Request – NYC, June 18."
4. In the body, includes: type of absence (PTO or travel), destination if applicable, dates, and any coverage notes.
5. Sends the meeting request to their manager.

### Manager Reviews & Approves

1. Manager receives the request and reviews it against team coverage and scheduling needs.
2. **If approved:** Manager accepts the meeting request to confirm receipt, then creates the formal calendar event on the shared calendar following the naming conventions above.
3. **If denied:** Manager declines the request and includes a brief explanation in the response body.
4. Manager notifies the employee directly if the accept/decline message alone isn't sufficient.

> The employee's meeting request is the **routing mechanism only**. The dedicated event on the shared calendar is the **official approval record**. Don't treat a meeting acceptance as the final confirmation.

### Verification

- Employee confirms their approved time appears on the shared calendar with the correct title and location.
- Other managers and staff can see the event and plan coverage accordingly.
- The Calendar Owner periodically reviews for formatting consistency and permission issues.

---

## Ongoing Maintenance

### When a New Employee Joins

- Calendar Owner adds them as Reviewer (Step 4).
- Their manager walks them through the request submission workflow.

### When a Manager Changes

- Calendar Owner updates the departing manager's permission to Reviewer or removes them entirely.
- New manager is added as Editor (Step 3).
- Existing events with the old manager's name in the Location field don't need to be updated unless your team prefers it.

### Periodic Audits

- **Quarterly:** Calendar Owner reviews permissions to ensure no unauthorized Editor access has been granted.
- **Annually:** Review naming convention adherence and update your documentation if processes have changed.

---

## Troubleshooting

| Issue | Resolution |
|---|---|
| Shared calendar not appearing in Outlook | Allow up to 60 minutes after permissions are granted. If still missing, use File → Open & Export → Other User's Folder to manually open the shared calendar. |
| Manager cannot create events | Confirm they have Editor (not Reviewer) permission. Check the Permissions tab and correct if needed. |
| Events appearing on wrong calendar | Ensure the manager is creating events on the shared calendar, not their personal calendar. Both may be visible side-by-side in Outlook. |
| Permission changes not taking effect | Outlook can cache permissions for up to an hour. Have the user close and reopen Outlook, or wait and retry. |
| Staff member can edit events they shouldn't | Verify the staff member is set to Reviewer, not Editor. Check and correct in the calendar properties. |
| Distribution group not inheriting permissions | Some Exchange configurations require permissions on individual mailboxes. Contact IT to verify whether group-based calendar permissions are supported in your environment. |

---

## Quick Reference

| Task | Responsible | Frequency |
|---|---|---|
| Create shared mailbox | IT Admin | One-time setup |
| Assign Calendar Owner permissions | IT Admin | One-time setup |
| Grant Manager (Editor) access | Calendar Owner | Setup + each new manager |
| Grant Staff (Reviewer) access | Calendar Owner | Setup + each new hire |
| Subscribe to calendar in Outlook | All users | Setup + new hires |
| Submit PTO or travel request | Employee | As needed |
| Approve request & post to shared calendar | Manager | As needed |
| Audit calendar permissions | Calendar Owner | Quarterly |
