# Ticketing Tool

A simple, dependency-free support ticket management system.

**Stack:** PHP 8+, MySQL, HTML5, CSS3, vanilla JavaScript (Fetch API/AJAX).
No frameworks (no Laravel, React, Bootstrap, jQuery, Node.js).

---

## Folder structure

```
ticketing-tool/
├── index.php                 # Main page: submission form + ticket queue
├── config.php                # PDO database connection
├── database.sql              # Schema + sample data
├── api/
│   ├── helpers.php           # Shared JSON response + validation helpers
│   ├── create_ticket.php     # POST — create a ticket
│   ├── get_tickets.php       # GET  — list/search/filter tickets
│   ├── get_ticket.php        # GET  — fetch one ticket
│   ├── update_ticket.php     # POST — update a ticket (full or partial)
│   └── delete_ticket.php     # POST — delete a ticket
├── assets/
│   ├── css/style.css
│   └── js/app.js
└── README.md
```

---

## Setup with XAMPP

1. **Copy the project into `htdocs`**
   Copy the whole `ticketing-tool` folder into your XAMPP `htdocs` directory, e.g.:
   - Windows: `C:\xampp\htdocs\ticketing-tool`
   - macOS: `/Applications/XAMPP/htdocs/ticketing-tool`
   - Linux: `/opt/lampp/htdocs/ticketing-tool`

2. **Start Apache and MySQL**
   Open the XAMPP Control Panel and start both **Apache** and **MySQL**.

3. **Create the database**
   - Open **phpMyAdmin** at `http://localhost/phpmyadmin`.
   - Click **Import**, choose `database.sql`, and click **Go**.
   - This creates the `ticketing_tool` database, the `tickets` table, and 6 sample tickets.

   *(Alternative, via command line):*
   ```bash
   mysql -u root -p < database.sql
   ```

4. **Check `config.php`**
   The defaults match a stock XAMPP install (`host=localhost`, `user=root`, `password=""`).
   If your MySQL uses a different user/password, edit the values at the top of `config.php`.

5. **Open the app**
   Visit: `http://localhost/ticketing-tool/`

That's it — no `composer install`, no build step, no Node.js required.

---

## How it works

- `index.php` renders the page shell only (form, filters, empty table, hidden modal).
- `assets/js/app.js` calls the `api/*.php` endpoints with `fetch()` to load, create,
  update, and delete tickets, and re-renders the table without a page reload.
- Every endpoint in `api/` returns JSON in the same shape:
  ```json
  { "success": true, "message": "...", "data": { ... } }
  ```
- All SQL uses PDO **prepared statements** (no string-concatenated queries).
- All server input is validated in PHP (required fields, email format, enum values)
  *in addition to* the JavaScript validation, since client-side checks can be bypassed.
- All dynamic text inserted into the page (`assets/js/app.js`) goes through an
  `escapeHtml()` helper before being placed in `innerHTML`, to prevent XSS.

---

## Testing checklist

Use this to verify every requirement works end-to-end.

### Create
- [ ] Submit the "New Ticket" form with all fields filled in → success message appears, no page reload, new ticket shows up at the top of the queue.
- [ ] Submit the form with an empty **Title** → inline error appears under the field, nothing is sent to the server (open browser dev tools → Network tab to confirm no request, or confirm PHP still rejects it if you bypass JS).
- [ ] Submit with an invalid email (e.g. `not-an-email`) → validation error shown, no ticket created.
- [ ] Leave **Assigned To** blank → ticket is created with `Unassigned`.

### Read
- [ ] On page load, the queue shows the 6 sample tickets from `database.sql`.
- [ ] Click **View** on any ticket → modal opens pre-filled with that ticket's full details, including created/updated timestamps.

### Update
- [ ] Click **Edit**, change the **Status** dropdown (e.g. Open → In Progress), click **Save Changes** → modal shows a success message, closes, and the table row updates its status badge without a page reload.
- [ ] Change **Assigned To** and **Priority** together and save → both changes are reflected in the table.
- [ ] Try saving with the Title field cleared → inline validation error, save is blocked.

### Delete
- [ ] Click **Delete** on a table row → a confirmation dialog appears.
- [ ] Cancel the dialog → ticket remains in the queue.
- [ ] Confirm the dialog → ticket disappears from the queue immediately, toast confirms deletion.
- [ ] Delete a ticket from inside the **View/Edit modal** as well → same behavior, modal closes.

### Persistence after refresh
- [ ] Create a new ticket, then reload the browser page (`F5`) → the new ticket is still there.
- [ ] Edit a ticket's status, reload the page → the updated status is still shown (confirms data is stored in MySQL, not just in memory/localStorage).
- [ ] Delete a ticket, reload the page → it stays deleted.

### Search
- [ ] Type part of a ticket **title** into the Search box → the table filters live (debounced) to matching tickets only.
- [ ] Search by part of a **requester's email** → matching ticket(s) appear.
- [ ] Search by part of the **description** text → matching ticket(s) appear.
- [ ] Clear the search box → full list returns.

### Filters
- [ ] Select **Status = Resolved** → only resolved tickets show.
- [ ] Select **Priority = High** → only high-priority tickets show.
- [ ] Select **Assignee = Mike Chen** → only his tickets show.
- [ ] Combine Search + Status + Priority + Assignee at once → results satisfy all conditions simultaneously (AND logic).
- [ ] Click **Clear Filters** → all filters reset and the full ticket list reloads.

---

## Notes / possible extensions

- The "Assigned To" field is currently a free-text input with a `<datalist>` suggestion list (Mike Chen, Priya Patel, Unassigned) rather than a hardcoded `<select>`, so you can assign to anyone. Swap it for a fixed `<select>` if you want a closed list of staff.
- Pagination isn't included since it wasn't requested — for a large number of tickets you'd add `LIMIT`/`OFFSET` to `get_tickets.php` and paging controls in `app.js`.
- Authentication/login isn't included — every visitor can create/edit/delete tickets, appropriate for a learning project but not for production use as-is.
