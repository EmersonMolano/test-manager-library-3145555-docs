# Specification — Manager Library

**What is built and why.** A system for a small library to manage its book catalog, members and loans, and to see which books are borrowed most often.

This specification is governed by [`constitution.md`](constitution.md). The data model lives only in [`data-model.md`](data-model.md), and the HTTP
contract lives only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | What they do | Authentication |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and members; creates librarians; views all reports | Username and password → token |
| **Librarian** (`librarian`) | Registers loans and returns; queries the catalog, members and overdue items | Username and password → token |
| **Member** | Person who borrows books. **Does not enter the system**: it is data, not a user | — |

The `admin` role is provisioned by deployment from environment variables. **It cannot be granted at runtime.**

## 2. Scope

**Included:** book catalog with optional cover, five fixed categories, members, multi-line loans, line-by-line returns, overdue books,
most-borrowed report and role-based login.

**Excluded and not reopened:** fines and payments, reservations, renewals, email or SMS, bulk import, inter-branch loans and book attributes beyond
§3 (no publisher, year or language).

## 3. User stories and acceptance criteria

### HU-01 · Log in

As a user, I want to authenticate so that I can operate according to my role.

- **CA-01.1** Valid credentials return a token valid for 8 hours.
- **CA-01.2** Invalid credentials return 401 with a message that does **not** reveal whether the username or password failed.
- **CA-01.3** Every endpoint except login and `/health` requires a token. Without a token: 401. Insufficient role: 403.

### HU-02 · Create librarians (admin)

- **CA-02.1** The admin creates a user with username, password and role `librarian`.
- **CA-02.2** The username is stored lowercase and trimmed; it is unique. Duplicate: 409.
- **CA-02.3** Requesting the `admin` role through the API returns 422. The password is never returned or logged.

### HU-03 · Manage books (admin)

- **CA-03.1** Create a book with title, author, ISBN, category and total copies. Available copies start equal to total copies.
- **CA-03.2** ISBN has 13 digits and is unique among active books. Duplicate: 409.
- **CA-03.3** Total copies must be ≥ 1. Changing the total adjusts available copies by the same difference and **never** leaves available copies < 0 (422).
- **CA-03.4** Upload a cover (JPG/PNG, maximum 2 MB) and store an **opaque key**; without a cover, the field is `null`, never an empty string.
- **CA-03.5** Deleting a book is a **logical deletion**: it disappears from the catalog but remains in past loans. A book with open loans cannot be deleted (409).

### HU-04 · Search books

- **CA-04.1** Search by partial text in title or author, filter by category, active books only, sort by title and paginate (`page`, `size`, maximum 50).
- **CA-04.2** The response includes the total number of results.
- **CA-04.3** Requesting a nonexistent or logically deleted book returns 404.

### HU-05 · Manage members

- **CA-05.1** The admin and librarian create members with name and document. The document is unique: duplicate 409.
- **CA-05.2** Members can be listed and searched by name or document, with pagination.
- **CA-05.3** Members are not deleted.

### HU-06 · Register a loan

- **CA-06.1** A loan belongs to **one** member, is registered by the authenticated user and contains 1 to 3 lines.
- **CA-06.2** The same book cannot be repeated in one loan (422).
- **CA-06.3** Each line withdraws **one** available copy in the same operation. If no copy is available: 409 and **nothing is recorded**.
- **CA-06.4** A member cannot have more than **3 unreturned books** in total (422 with the current count).
- **CA-06.5** A member with any **overdue** book cannot request another one (409).
- **CA-06.6** Due date is `loaned_at + 14 days`. The line freezes title, ISBN and category name **at that instant**.
- **CA-06.7** A registered loan cannot be edited or deleted.

### HU-07 · Register a return

- **CA-07.1** One **line** is returned at a time. It sets `returned_at` and adds one to available copies.
- **CA-07.2** Returning an already returned line returns 409. It **never** adds twice.
- **CA-07.3** Available copies never exceed total copies; the database guarantees this.
- **CA-07.4** When every line is returned, the loan is shown as **closed**.

### HU-08 · View and list loans

- **CA-08.1** A loan detail includes its lines and calculated status (`open`, `overdue`, `closed`).
- **CA-08.2** Loans can be listed by date range, status and member, sorted by descending date, with pagination.
- **CA-08.3** Status is **calculated**, not stored.

### HU-09 · Overdue books

- **CA-09.1** List unreturned lines whose `due_at` is before now: book, member and late days.
- **CA-09.2** Sort by most late days first. Only `admin` and `librarian` may access it.

### HU-10 · Most-borrowed book report

- **CA-10.1** For a date range, return one row for each **book and frozen category name**, with the number of times borrowed, sorted descending.
- **CA-10.2** End before start: 422. A range with no loans returns an empty list, not an error.
- **CA-10.3** The report is **not** broken down by librarian or member.
- **CA-10.4** A report for a closed period does **not** change if the book or its category is later renamed.
- **CA-10.5** Only `admin` may access it.

## 4. Business rules

| # | Rule |
|---|---|
| RN-1 | Available copies are always between 0 and total copies. |
| RN-2 | Loan period: 14 days. Maximum 3 unreturned books per member. |
| RN-3 | A member with overdue books cannot request more until returning them. |
| RN-4 | Frozen line values (title, ISBN, category) do not follow the catalog. |
| RN-5 | Loans and returns are never deleted. Books only have logical deletion. |
| RN-6 | Dates are UTC instants. User messages are in Spanish; code and columns are in English. |

## 5. Non-functional requirements

| # | Requirement | Measurement |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5,000 books | Load test using the `tool` seeder |
| RNF-2 | Two librarians lending the **last copy** at the same time: one wins, the other receives 409 | Concurrency test |
| RNF-3 | Passwords are never transmitted or stored in plaintext and never appear in logs | Log and response review |
| RNF-4 | Every HTTP error uses the same body (see contract) | Contract test |
| RNF-5 | The system starts with one command and the database starts empty | `infra` README |
