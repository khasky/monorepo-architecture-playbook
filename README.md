# Monorepo Architecture Playbook

Practical guide for designing and operating JavaScript / TypeScript monorepos with clear package boundaries, fast local workflows, reproducible CI, and sane release discipline.

> *If I were setting up a serious Node.js / React / TypeScript codebase for multiple apps and shared packages today — what monorepo rules would I want from day one?*

This playbook is opinionated on purpose. It favors:

- **clear ownership over clever folder trees**;
- **package boundaries over "shared" dumping grounds**;
- **fast local workflows and reproducible CI**;
- **incremental releases instead of repo-wide chaos**.

---

## Table of Contents

- [Monorepo Architecture Playbook](#monorepo-architecture-playbook)
  - [Table of Contents](#table-of-contents)
  - [Companion playbooks](#companion-playbooks)
  - [Philosophy](#philosophy)
  - [When I would choose a monorepo](#when-i-would-choose-a-monorepo)
  - [The defaults I'd reach for first](#the-defaults-id-reach-for-first)
  - [A practical workspace shape](#a-practical-workspace-shape)
  - [Package taxonomy that scales](#package-taxonomy-that-scales)
    - [Good long-term package categories](#good-long-term-package-categories)
    - [Smells](#smells)
  - [Dependency and versioning rules](#dependency-and-versioning-rules)
    - [My preference](#my-preference)
  - [Task pipelines and CI](#task-pipelines-and-ci)
    - [I want CI to answer four questions fast](#i-want-ci-to-answer-four-questions-fast)
    - [Baseline task model](#baseline-task-model)
    - [Practical rules](#practical-rules)
  - [Release flows](#release-flows)
    - [Release styles I actually use](#release-styles-i-actually-use)
  - [Things that break monorepos](#things-that-break-monorepos)
  - [Worth reading](#worth-reading)
  - [License](#license)

---

## Companion playbooks

These repositories form one playbook suite:

- [Auth & Identity Playbook](https://github.com/khasky/auth-identity-playbook) — sessions, tokens, OAuth, and identity boundaries across the stack
- [Backend Architecture Playbook](https://github.com/khasky/backend-architecture-playbook) — APIs, boundaries, OpenAPI, persistence, and errors
- [Best of JavaScript](https://github.com/khasky/best-of-javascript) — curated JS/TS tooling and stack defaults
- [Caching Playbook](https://github.com/khasky/caching-playbook) — HTTP, CDN, and application caches; consistency and invalidation
- [Code Review Playbook](https://github.com/khasky/code-review-playbook) — PR quality, ownership, and review culture
- [DevOps Delivery Playbook](https://github.com/khasky/devops-delivery-playbook) — CI/CD, environments, rollout safety, and observability
- [Engineering Lead Playbook](https://github.com/khasky/engineering-lead-playbook) — standards, RFCs, and technical leadership habits
- [Frontend Architecture Playbook](https://github.com/khasky/frontend-architecture-playbook) — React structure, performance, and consuming API contracts
- [Marketing and SEO Playbook](https://github.com/khasky/marketing-and-seo-playbook) — growth, SEO, experimentation, and marketing surfaces
- **Monorepo Architecture Playbook** — workspaces, package boundaries, and shared code at scale
- [Node.js Runtime & Performance Playbook](https://github.com/khasky/nodejs-runtime-performance-playbook) — event loop, streams, memory, and production Node performance
- [Testing Strategy Playbook](https://github.com/khasky/testing-strategy-playbook) — unit, integration, contract, E2E, and CI-friendly test layers

---

## Philosophy

A good monorepo is not "one repo because it feels modern".

A good monorepo gives you:
- **atomic changes** across app + API + shared packages;
- **consistent tooling** and fewer one-off setups;
- **faster reuse** of UI, types, validation, and infrastructure code;
- **better visibility** into what depends on what.

A bad monorepo gives you:
- one giant `packages/shared` graveyard;
- one broken install that blocks everyone;
- slow CI because every task runs for every change;
- accidental tight coupling hidden behind import aliases.

---

## When I would choose a monorepo

I usually choose a monorepo when at least **two** of these are true:

- multiple apps share real code, not just design preferences;
- frontend and backend should evolve atomically;
- internal packages deserve first-class treatment;
- one team is feeling friction from duplicated tooling;
- release coordination matters more than repo autonomy.

I would **not** force a monorepo just because:
- the team is tiny and has one app;
- the projects barely share anything;
- every service has a different lifecycle and ownership model;
- platform maturity is too low to maintain workspace discipline.

---

## The defaults I'd reach for first

If I were starting a JS/TS monorepo today, I would usually begin close to this:

- **Package manager:** `pnpm`
- **Orchestration:** `Turborepo` for frontend-heavy repos, `Nx` when I need deeper workspace governance
- **Language:** `TypeScript`
- **Build linking:** TypeScript project references where they actually help
- **Releases:** `Changesets`
- **CI:** GitHub Actions with affected / cached tasks
- **Validation:** runtime schemas at package boundaries
- **Docs:** one root README + package READMEs for anything public or reusable

---

## A practical workspace shape

```text
.
├─ apps/
│  ├─ web/
│  ├─ admin/
│  ├─ api/
│  └─ docs/
├─ packages/
│  ├─ ui/
│  ├─ config-eslint/
│  ├─ config-ts/
│  ├─ auth/
│  ├─ api-contract/
│  ├─ validation/
│  ├─ observability/
│  └─ shared-utils/
├─ tooling/
│  ├─ scripts/
│  └─ generators/
├─ .changeset/
├─ .github/
│  ├─ workflows/
│  └─ pull_request_template.md
└─ turbo.json
```

My rule of thumb:

- `apps/` ship products;
- `packages/` export reusable capabilities;
- `tooling/` holds repo-level scripts and generators;
- root config should be thin and boring.

---

## Package taxonomy that scales

A monorepo gets healthier when packages have **roles**, not just names.

### Good long-term package categories

- **app** packages — deployable applications
- **ui** packages — reusable presentational components
- **feature** packages — cohesive business capabilities
- **data-access / client** packages — API and persistence access
- **contract** packages — shared schemas, types, API clients
- **config** packages — ESLint, TSConfig, Vitest, Prettier, Storybook presets
- **tooling** packages — CLIs, codemods, repo scripts

### Smells

- `shared`, `common`, `utils`, `core` with no clear boundary
- app packages importing from each other directly
- feature packages depending on everything
- runtime code importing dev-only config packages

---

## Dependency and versioning rules

These rules save real repos:

1. **Install dependencies where they are used.**
2. **Do not hide runtime dependencies in the root `package.json`.**
3. **Prefer internal package names with a namespace** like `@acme/ui`.
4. **Keep dependency direction intentional.**
5. **Document public surface areas.**
6. **Use a single release strategy on purpose, not by accident.**

### My preference

- one workspace-wide Node version policy;
- explicit package ownership;
- shared lint/TS/test config via internal packages;
- package contracts that are import-stable and documented.

---

## Task pipelines and CI

A monorepo should make CI **smarter**, not simply larger.

### I want CI to answer four questions fast

- Did formatting/linting fail?
- Did a contract change?
- Which projects were actually affected?
- Is this safe to merge and release?

### Baseline task model

- `lint`
- `typecheck`
- `test`
- `build`
- `storybook` or `docs`
- `release` or `publish`

### Practical rules

- cache deterministic tasks;
- run affected tasks on pull requests;
- run the full release graph only on merge to main;
- make local commands mirror CI names;
- keep "special" pipeline exceptions extremely rare.

---

## Release flows

For package-based repos, I strongly prefer:
- changes recorded in a small, reviewable release note file;
- version bumps generated automatically;
- changelogs produced by tooling, then edited when nuance matters;
- publishing only from trusted CI, never from random laptops.

### Release styles I actually use

- **locked / coordinated** releases for tightly coupled packages
- **independent** releases for library-heavy repos
- **internal-only packages** that never publish externally but still version meaningfully

---

## Things that break monorepos

The fastest ways to ruin one:

- creating a repo-wide `shared` folder before package boundaries are real;
- letting app code import deep internals from another package;
- centralizing every dependency in the root;
- mixing generated code, source, and build output carelessly;
- making CI all-or-nothing;
- not assigning ownership for packages;
- using a monorepo while still thinking like disconnected repos.

---

## Worth reading

- [Turborepo docs](https://turbo.build/repo/docs)
- [Nx concepts](https://nx.dev/docs/concepts)
- [Nx monorepo benefits](https://nx.dev/docs/concepts/decisions/why-monorepos)
- [TypeScript project references](https://www.typescriptlang.org/docs/handbook/project-references.html)
- [Changesets](https://github.com/changesets/changesets)
- [Bulletproof React](https://github.com/alan2207/bulletproof-react)

---

## License

MIT is a sensible default for a repository like this, but choose the license that fits how you want others to reuse the material.
