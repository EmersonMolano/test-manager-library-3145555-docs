# ADR-003 · Books are logically deleted

- **Status:** accepted
- **Context:** a loan line points to a book. If the book is physically deleted, history and reports break.
- **Decision:** the book has `deleted_at`. Logical deletion fills it; the catalog filters `deleted_at IS NULL` in one place (a global filter). A book with open loans **cannot** be logically deleted (409 `BOOK_HAS_OPEN_LOANS`). ISBN uniqueness is a **partial** index on active books, so a logically deleted ISBN can be registered again.
- **Rejected alternatives:** physical deletion with cascade (destroys history); an `active = false` flag (does not say when the deletion happened).
- **Consequences:** `loan_item.book_id` has an FK `RESTRICT` as a **last-resort barrier**: a manual `DELETE` or future code change fails loudly instead of corrupting history. The cover binary is the only data that is physically deleted.
- **Verification:** requesting a logically deleted book returns 404; a `psql` delete of a book referenced by loan lines fails because of the FK.
