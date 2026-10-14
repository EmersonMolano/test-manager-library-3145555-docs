# Constitution — Manager Library (Laravel)

These are the project's non-negotiable principles. They are not a matter of style or preference: every article defines how it is verified.
Any change that violates an article is rejected regardless of the rest of the implementation.

---

## Article I — The domain depends on nothing

`app/Domain/` must not import another project module or a third-party framework. It may use the PHP standard library, but not Laravel,
Eloquent, HTTP concerns, validation packages or infrastructure adapters.

**Why:** this makes it possible to test invariants without a database, server or container, and to change the ORM without changing business rules.
**Verification:** architecture tests and static analysis prevent `Domain` from depending on `Application`, `Adapters` or `Providers`.

## Article II — The core declares its ports; the outside world implements them

The application layer knows only the domain. Everything it needs from the outside (repositories, clock, password hashing, cover storage and
token generation) is expressed as a **port defined by the application layer**. No Laravel, Eloquent or framework type may appear in a port signature.

**Verification:** architecture tests prevent `Application` from importing `Adapters`. Application tests run with fakes and without Laravel infrastructure.

## Article III — One composition point

`app/Providers/LibraryServiceProvider.php` is the only place where a port is bound to its adapter. Infrastructure must not be instantiated outside
that composition point.

**Verification:** architecture tests detect adapter instantiation outside `LibraryServiceProvider`.

## Article IV — SOLID is not decorative

| Principle | Concrete application | Verification |
|---|---|---|
| **SRP** | One application service = one use case. `LendBooksService` lends books and does nothing else | A service with two reasons to change is split |
| **OCP** | Changing cover storage (disk → S3) does not touch the domain or application | A new adapter only requires a binding change |
| **LSP** | Test fakes replace real adapters without changing expectations | The same test passes against the fake and the real adapter |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage` | No implementation has unused methods |
| **DIP** | The consumer owns the port | Ports live in the application layer |

## Article V — Migrations own the schema

The `infra` repository starts an **empty** database engine. All DDL (tables, indexes, constraints, extensions and category seed data) lives in
Laravel migrations under `database/migrations/`. See [ADR-001](adr/adr-001-propiedad-del-esquema.md).

**Verification:** `infra` contains no `CREATE TABLE` statements; the migration output matches the queries in `data-model.md` §10.

## Article VI — The database enforces what it can enforce

A rule that exists only in code protects the application, not the data. Every invariant expressible by the engine (`CHECK`, `UNIQUE`, FK) is declared
there. Rules that are not applied at the database level are marked **pending** in [`data-model.md`](data-model.md); they are not promised as database guarantees.

**Verification:** the queries in `data-model.md` §10 match the constraints declared by the migrations.

## Article VII — Derived values are calculated, not stored

Loan status (`open`, `overdue`, `closed`) and late days are calculated. There is no `status` or `days_late` column.

**Verification:** `data-model.md` does not list a `status` or `days_late` column.

## Article VIII — Frozen values do not follow the catalog

A loan line stores the title, ISBN and category name as they were at lending time. Reports never read those values from the live catalog.

**Verification:** a test renames a book after lending and verifies that the report retains the old title ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)).

## Article IX — No secret enters the repository or a log

Passwords, precomputed hashes and token keys never enter the repository. All secrets come from the environment. The real `.env` is ignored;
only `.env.example` is versioned.

**Verification:** `gitleaks` or equivalent CI scanning; no HTTP response includes `password` or `password_hash`.

## Article X — The engine wins

If the data document and the database disagree, the document is broken and must be corrected.

## Article XI — Language

Code, columns and routes remain in English. Documentation is in English. User-facing API messages remain in Spanish because that is part of the
business contract.

## Article XII — Nothing is done without evidence

A task in [`tasks.md`](tasks.md) is complete only when its **evidence** exists and can be executed, not when it is merely planned.
