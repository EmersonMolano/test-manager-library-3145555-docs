# ADR-004 · The report is aggregated by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a range. If it read the title or category from the live catalog, renaming a book or category would change a report for a closed period.
- **Decision:** (1) the report is **not persisted**: it is calculated by one aggregate query through a read port. (2) It groups by `book_id, book_title, category_name` **as frozen in the line**. (3) `loan_item.category_name` intentionally has **no FK**. (4) If a book changes category within the range, the report shows **two rows**; it does not choose a winner.
- **Rejected alternatives:** joining the live catalog (the report stops being stable); taking “the most recent value in the range” (a new loan would change what was already read); materializing a report table (second source of truth).
- **Consequences:** CA-10.1 says “one row per book and frozen category name”, not “one row per book”. The report contains no members or librarians (privacy).
- **Verification:** lend a book, rename it and request a report for the same range: the report title is the old title.
