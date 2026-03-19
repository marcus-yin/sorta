# Notifications Table Setup (Send Message)

The **Send message** feature stores in-app messages in a **notifications** table and optional file attachments in the **documents** table, linked to each notification. After creating the table(s) in OpenGov, add the table and field keys to `API_CONFIG` in the relevant HTML files.

---

## 1. Notifications table

Create a table in OpenGov (e.g. named **Notifications** or **Messages**) with at least these fields:

| Field name (logical) | Purpose |
|----------------------|--------|
| `subject` | Message subject (text) |
| `message` | Message body (long text) |
| `related_application` | Relationship/lookup to the Application table (links message to one application) |
| `sender` | Who sent it (text or lookup to User – e.g. user Rid or email) |
| `direction` | `to_applicant` or `to_evaluator` (so you can trigger email to the right party) |

Optional but useful:

- `created_at` / timestamp (if your API auto-fills it)
- `read_at` or `status` if you add an in-app inbox later

After creation, set in `API_CONFIG`:

- `notificationsTableKey` – table key
- `notificationsReportId` – report ID used to query notifications
- `notificationsSubjectFieldKey`
- `notificationsMessageFieldKey`
- `notificationsRelatedApplicationFieldKey`
- `notificationsSenderFieldKey`
- `notificationsDirectionFieldKey`

Replace the placeholders `YOUR_NOTIFICATIONS_*` in:

- **Measure Q:** `opengov_project_overview.html`
- **MTIF:** `mtif_application_evaluator.html`, `mtif_application.html`

---

## 2. Documents table – link to notifications

To support **attachments** on messages, the existing **documents** table must have a field that links each document row to a notification record:

| Field name (logical) | Purpose |
|----------------------|--------|
| `related_notification` | Relationship/lookup to the Notifications table |

After adding this field, set in `API_CONFIG`:

- `documentsRelatedNotificationFieldKey` – field key for `related_notification`

**MTIF** pages also use (if you use a separate documents table for MTIF):

- `documentsTableKey`
- `documentsFileFieldKey`
- `documentsNameFieldKey`
- `documentsTypeFieldKey`

When saving a message with attachments, the app:

1. Creates one row in the **notifications** table (subject, message, related_application, sender, direction).
2. For each file, creates one row in the **documents** table with the file binary and `related_notification` = the new notification record ID.

If `documentsRelatedNotificationFieldKey` is not set (or left as the placeholder), the message is still saved but attachments are skipped.

---

## 3. Where the button and direction are set

- **Evaluator → Applicant:**  
  **Send message** is on the evaluator application view. Direction is set to `to_applicant`.  
  Files: `opengov_project_overview.html` (evaluator context), `mtif_application_evaluator.html`.

- **Applicant → Evaluator:**  
  **Send message** is on the applicant application view. Direction is set to `to_evaluator`.  
  Files: `opengov_project_overview.html` (applicant context), `mtif_application.html`.

Measure Q uses a single page for both roles. You can set `window.MQ_MESSAGE_DIRECTION` before opening the modal (e.g. from URL or role) to override the default `to_applicant`.

---

## 4. Email trigger

This repo only **writes** to the notifications table. To send email when a new notification is created, configure your backend or OpenGov workflow to:

- Watch for new rows in the **notifications** table (or use a report).
- Use `direction` to choose recipient (e.g. applicant email from the related application vs evaluator/SORTA).
- Send the email and optionally set a `read_at` or `email_sent_at` field.
