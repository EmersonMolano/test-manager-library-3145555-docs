# test-manager-library-3145555-docs

> **Technical test · ADSO form 3145555**
> Schedule: **Wednesday, October 14, 2026, from 6:50 a.m. to 11:00 a.m.** (Colombia time). Submission deadline: **11:00 a.m.**

This repository contains the **Manager Library specification**. The other five repositories start empty on purpose.

## Instructions

Each learner must **fork the six project repositories** and solve the project using the specification.

1. Fork each repository in the table at the end to your GitHub account.
2. Read the specification in [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs).
   It includes three reference variants: `spec-python/`, `spec-.net/` and `spec-larabel/`.
3. Develop in the forks.

## The challenge uses React and PHP (Laravel)

The Python and .NET folders are reference implementations. The actual challenge uses:

| Layer | Challenge technology |
|---|---|
| Frontend | React |
| Backend | PHP with Laravel |

Business requirements — user stories, acceptance criteria, rules, API contract and data model — must remain unchanged. The technical decisions are
translated to React and Laravel. The `spec-larabel/` folder is the implementation-oriented reference for the PHP backend.

## The test is not only about writing code

The main purpose is to evaluate **Spec-Driven Development (SDD)**: reading, interpreting and applying a specification to a different stack. Code is
the means, not the end.

## Submission

Only what is pushed to the forks before **11:00 a.m.** is counted. There is no extension: push partial progress with small commits during the test.

## Repository contents

| Folder | Content |
|---|---|
| [`spec-python/`](spec-python/) | Manager Library specification for a Python backend |
| [`spec-.net/`](spec-.net/) | Manager Library specification for a .NET backend |
| [`spec-larabel/`](spec-larabel/) | Manager Library specification adapted to PHP/Laravel |

All variants describe the **same product**: the same stories, business rules and endpoints. They differ only in technical decisions.

## How to read a specification

In the implementation-oriented variant, read in this order:

1. `constitution.md` — non-negotiable principles.
2. `spec.md` — what the system must do: actors, stories, acceptance criteria and business rules.
3. `plan.md`, `architecture.md` and `arquitectura-panoramica.md` — how it is built.
4. `data-model.md` and `api-contract.md` — data model and API contract.
5. `tasks.md` — what must be done and what evidence proves it.
6. `adr/` — architecture decisions, alternatives and consequences.

## The six repositories

| Repository | Purpose |
|---|---|
| [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs) | The specification |
| [`test-manager-library-3145555-api`](https://github.com/code-sena/test-manager-library-3145555-api) | PHP backend (Laravel) |
| [`test-manager-library-3145555-app`](https://github.com/code-sena/test-manager-library-3145555-app) | React frontend |
| [`test-manager-library-3145555-page`](https://github.com/code-sena/test-manager-library-3145555-page) | Public static presentation site |
| [`test-manager-library-3145555-infra`](https://github.com/code-sena/test-manager-library-3145555-infra) | Containers, network, volumes and empty database engine |
| [`test-manager-library-3145555-tool`](https://github.com/code-sena/test-manager-library-3145555-tool) | Demonstration data seeder |
