# ADR-002 · Optimistic concurrency for copies

- **Status:** accepted
- **Context:** two librarians can lend the **last copy** of the same book at the same time (RNF-2). If both read `available_copies = 1` and both write, the system lends a book that does not exist.
- **Decision:** use **optimistic concurrency**. The persistence adapter uses a book version witness; when saving, if the witness changed, the operation fails and the server returns **409 `BOOK_NOT_AVAILABLE`** (or retries once if the copy is still available). The **last barrier** is the `CHECK` `ck_book_available_range` (`0 <= available_copies <= total_copies`).
- **Rejected alternatives:** pessimistic row locking (`SELECT … FOR UPDATE`) on every loan: it serializes the whole library for a rare case; an in-memory counter does not survive multiple processes.
- **Consequences:** if the `CHECK` fails, **something wrote outside the adapter**: that is a defect, not a business case. Returning the same line twice does not add twice because `returned_at` is already filled (CA-07.2).
- **Verification:** a concurrency test sends two simultaneous requests for a book with one copy: exactly one receives 201.
