# Technical plan — Manager Library (Laravel)

This document explains how the requirements in [`spec.md`](spec.md) and [`constitution.md`](constitution.md) are built. The schema is defined only
in [`data-model.md`](data-model.md); the system shape is defined in [`architecture.md`](architecture.md).

## 1. Technical decisions

| # | Decision | Laravel choice | Why |
|---|---|---|---|
| D-01 | Language and runtime | PHP 8.x / Laravel | Runtime and framework required by the challenge |
| D-02 | HTTP framework | Laravel routing, middleware and controllers | It implements the contract in [`api-contract.md`](api-contract.md) |
| D-03 | Persistence | Eloquent ORM + Laravel migrations on PostgreSQL 16 | The migrations own the schema (ADR-001) |
| D-04 | Concurrency | Book version witness + range `CHECK` ([ADR-002](adr/adr-002-concurrencia-optimista.md)) | Protects the last available copy (RNF-2) |
| D-05 | Dates | UTC instants (`timestamptz`); the clock is a `Clock` port | Expiration can be tested without waiting 14 days |
| D-06 | Report | One aggregate query through a read port ([ADR-004](adr/adr-004-reporte-agregado-y-congelado.md)) | Stable and inexpensive |
| D-07 | Value objects | `Isbn`, `Quantity`, `DocumentNumber`, `DateRange`: embedded in the owner's row, without a table | They have no identity |
| D-08 | Covers | Binary in external storage (local disk, S3 in production); the database stores only `cover_key` | Opaque key, never a path |
| D-09 | Passwords | Argon2id behind the `PasswordHasher` port; the domain never sees plaintext | Article IX |
| D-10 | Startup | The application creates the initial admin from `ADMIN_USERNAME` / `ADMIN_PASSWORD` | The database cannot create the hash |
| D-11 | Authentication | JWT HS256, 8 hours, key in `JWT_SECRET` | Simple and stateless |
| D-12 | Tests | PHPUnit or Pest, with fakes for ports and a real database only for integration tests | Test pyramid |

The persistence adapter uses the database's version witness for the book update. The witness is not exposed by the API and is not a business field.
The lending and return operations must update the book and loan data inside one database transaction.

## 2. Environment configuration

| Variable | Required | Example |
|---|---|---|
| `DATABASE_URL` | yes | `postgresql://library:library@db:5432/library` |
| `JWT_SECRET` | yes | long random string |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | defined in `.env`, never in the repository |
| `COVERS_DIR` | no | `./var/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

## 3. Test strategy

| Level | What it tests | Against |
|---|---|---|
| Domain | Invariants: maximum 3, 14-day due date, copies within range | Nothing: pure objects |
| Application | Use cases: lend, return, report | Port fakes |
| Integration | Repositories and report query | Real PostgreSQL |
| Contract | One error body and status codes | Running API |
| Architecture | Articles I, II and III | PHPUnit/Pest architecture tests and static analysis |
| Concurrency | Two loans for the last copy | Real PostgreSQL |

## 4. Build order

Skeleton and health check → initial migration with seed → domain → use cases with fakes → adapters → HTTP API → contract and concurrency tests.
The evidence for each step is in [`tasks.md`](tasks.md).
