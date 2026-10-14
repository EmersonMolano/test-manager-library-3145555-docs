# Tasks — Manager Library (Laravel)

Each task is complete only when its **evidence** exists and can be executed (Article XII). Initial state for every task: **pending**.

| # | Task | Depends on | Evidence |
|---|---|---|---|
| T-01 | Project skeleton and `GET /health` | — | `GET /health` returns `{"status":"ok"}`; the container starts with one command |
| T-02 | Initial migration: six tables, FKs, unique indexes and five seeded categories | T-01 | The queries in `data-model.md` §10 return the documented result; no column has a `DEFAULT` |
| T-03 | Domain: `Isbn`, `Quantity`, `DateRange` (value objects) | T-01 | Unit tests: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain: `Book` (`Withdraw`, `Restock`, logical deletion) | T-03 | Tests: never below 0, never above total |
| T-05 | Domain: `Loan` (`AddItem`, `ReturnItem`, calculated status, 14 days, maximum 3) | T-04 | Tests with a fake `Clock`: `open`, `overdue`, `closed` |
| T-06 | Domain: `Member` and `User` (username normalization, valid role) | T-03 | `"Ana "` becomes `ana`; the `admin` role is rejected by the API |
| T-07 | Authentication ports and use cases (`LoginService`, `CreateLibrarianService`) with fakes | T-06 | HU-01 and HU-02 pass against fakes |
| T-08 | Book use cases (create, search, edit, logical deletion, cover) with fakes | T-04 | HU-03 and HU-04 pass against fakes |
| T-09 | Lend use case (`LendBooksService`) with fakes | T-05, T-08 | HU-06 is complete: limits, overdue members and unavailable copies |
| T-10 | Return and overdue use cases | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | Most-borrowed report use case and read port | T-09 | HU-10; the old title remains after a rename (ADR-004) |
| T-12 | Persistence adapters: book, member and user repositories | T-02, T-08 | Integration tests against real PostgreSQL |
| T-13 | Persistence adapter: loan repository and version witness | T-12, T-09 | Two loans for the last copy: one 201, one 409 (RNF-2) |
| T-14 | Security adapter: password hashing and JWT token | T-07 | The password never appears in logs or responses (RNF-3) |
| T-15 | Disk cover adapter | T-08 | 2 MB PNG succeeds; 3 MB → 422; no cover → `cover_key` is null |
| T-16 | HTTP adapter: routes from [`api-contract.md`](api-contract.md), one error body and role authorization | T-10, T-11, T-14, T-15 | Contract tests cover every route and every status code in §2 |
| T-17 | Composition and startup (initial admin from environment) | T-16 | The system starts after migrations and the admin can log in |
| T-18 | Move **domain-only** invariants into the database: `ck_book_total_positive`, `ck_user_role` | T-12 | An `INSERT` through `psql` that violates them fails; `data-model.md` §4 is updated |
| T-19 | Architecture tests (Articles I, II and III) | T-17 | PHPUnit/Pest and static-analysis checks fail when a forbidden dependency is introduced |
| T-20 | `infra`: `docker compose up` starts the empty engine, API and frontend | T-17 | One command; the database has no tables until migrations run |
| T-21 | `tool`: demonstration seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-17 | After running it, `/loans/overdue` returns 3 lines and the report has rows |
| T-22 | `app`: login, catalog, members, lend, return, overdue and report | T-16 | Manual end-to-end flow at the presentation |
| T-23 | `page`: public static presentation site | — | Opens without the API; correct on mobile and desktop |

## Definition of done for the test

Reaching T-17 with evidence means the **core** is complete. T-18 through T-23 distinguish a good submission from an outstanding one.
Evaluation focuses on SDD performance, not task count: one evidenced task is worth more than three unfinished tasks.
