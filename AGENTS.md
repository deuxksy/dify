# AGENTS.md

## Project Overview

Dify is an open-source platform for developing LLM applications with an intuitive interface combining agentic AI workflows, RAG pipelines, agent capabilities, and model management.

The codebase is split into:

- **Backend API** (`/api`): Python Flask application organized with Domain-Driven Design
- **Frontend Web** (`/web`): Next.js application using TypeScript and React
- **CLI** (`/cli`): TypeScript CLI (`difyctl`) for managing Dify instances
- **Docker deployment** (`/docker`): Containerized deployment configurations
- **Dify Agent Backend** (`/dify-agent`): Backend services for managing and executing agent
- **E2E Tests** (`/e2e`): Cucumber + Playwright end-to-end test suite
- **Shared Packages** (`/packages`): `dify-ui` (design tokens + primitives), `iconify-collections`, `contracts`, `dev-proxy`, `tsconfig`
- **SDKs** (`/sdks`): Node.js client SDK

## Prerequisites

- **Node.js**: `^22.22.1` (see `.nvmrc`)
- **pnpm**: enabled via Corepack (`corepack enable`)
- **uv**: Python package manager (replaces poetry since v1.3.0)
- **Docker & Docker Compose**: required for middleware and E2E tests

## Backend Workflow

- Read `api/AGENTS.md` for details
- Run backend CLI commands through `uv run --project api <command>`.
- Integration tests are CI-only and are not expected to run in the local environment.

## Quick Reference Commands

| Task | Command |
| :--- | :--- |
| Initial dev setup | `make dev-setup` |
| Format backend | `make format` |
| Lint backend | `make lint` |
| Type-check backend | `make type-check` |
| Run backend tests | `make test` |
| Run all backend tests (Docker) | `make test-all` |
| Frontend dev server | `pnpm --filter dify-web dev` |
| Frontend lint | `pnpm --filter dify-web lint:fix` |
| Frontend type-check | `pnpm --filter dify-web type-check` |
| E2E (authenticated) | `pnpm -C e2e e2e` |
| E2E (full reset) | `pnpm -C e2e e2e:full` |
| CLI dev | `pnpm --filter difyctl dev <command>` |

## Frontend Workflow

- Read `web/AGENTS.md` for details
- Read `cli/AGENTS.md` for CLI development details
- Read `e2e/AGENTS.md` for E2E testing details

## Testing & Quality Practices

- Follow TDD: red → green → refactor.
- Use `pytest` for backend tests with Arrange-Act-Assert structure.
- Enforce strong typing; avoid `Any` and prefer explicit type annotations.
- Write self-documenting code; only add comments that explain intent.

## Language Style

- **Python**: Keep type hints on functions and attributes, and implement relevant special methods (e.g., `__repr__`, `__str__`). Prefer `TypedDict` over `dict` or `Mapping` for type safety and better code documentation.
- **TypeScript**: Use the strict config, rely on ESLint (`pnpm lint:fix` preferred) plus `pnpm type-check`, and avoid `any` types.

## General Practices

- Prefer editing existing files; add new documentation only when requested.
- Inject dependencies through constructors and preserve clean architecture boundaries.
- Handle errors with domain-specific exceptions at the correct layer.

## Project Conventions

- Backend architecture adheres to DDD and Clean Architecture principles.
- Async work runs through Celery with Redis as the broker.
- Frontend user-facing strings must use `web/i18n/en-US/`; avoid hardcoded text.
- This is a pnpm monorepo. See `pnpm-workspace.yaml` for workspace package membership.
