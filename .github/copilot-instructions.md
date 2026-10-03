# Copilot Instructions for DEVPILOT

## Repository layout

- This repo is a two-part app: `backend/` is a Spring Boot service, and `client/my-app/` is a Next.js app. The extra `my-app/` segment is real; there is no top-level `client/` app.
- The root `docker-compose.yml` provisions only PostgreSQL (`pgvector/pgvector:pg16`) on `localhost:5432`; the API and web app both run on the host, not in Docker containers.
- The Java package root is `devPilot` (capital `P`), not `devpilot`. Keep package names and group IDs consistent with that casing.
- The repo already contains agent-specific guidance in `AGENTS.md` and `client/my-app/AGENTS.md`. Follow those constraints when working in either runtime.

## Build, test, and lint commands

```bash
# Required once before backend or tests:
docker compose up -d

# Backend (Spring Boot 4.1.1 / Java 17)
cd backend
./mvnw spring-boot:run           # or mvnw.cmd spring-boot:run on Windows
./mvnw test                      # full backend test suite
./mvnw -Dtest=BackendApplicationTests test

# Frontend (Next.js 16.3.8)
cd client/my-app
npm install
npm run dev                      # app runs at http://localhost:3000
npm run build
npm run lint
```

Notes:
- There is no frontend test runner in this project; `npm run lint` is the only static check available for the client.
- The backend has one current test class: `BackendApplicationTests`, so the single-test pattern is `./mvnw -Dtest=BackendApplicationTests test`.
- `docker compose up -d` is required before running the backend tests because the application expects PostgreSQL to be available.

## High-level architecture

- `backend/` is a Spring Boot application using Spring Data JPA, Spring Security, OAuth2 client support, PostgreSQL, Flyway, and Spring AI OpenAI integration. The main app entry point is `backend/src/main/java/devPilot/backend/BackendApplication.java`.
- The persistence model is centered on `User` in `backend/src/main/java/devPilot/backend/entity/User.java`; it models GitHub-authenticated users and stores OAuth metadata (`githubId`, `githubUsername`, `accessToken`, etc.).
- Application configuration lives in `backend/src/main/resources/application.properties` and uses environment-variable-based defaults, such as `DB_URL`, `DB_USERNAME`, `DB_PASSWORD`, and `OPENAI_API_KEY`. The app is designed to boot with sensible defaults, but the DB must still be running.
- `client/my-app/` is a Next.js App Router project built with Tailwind v4, the `app/` directory, and a shadcn-style component set under `components/` and `components/ui/`. The UI is separate from the backend runtime and only communicates through HTTP endpoints when they exist.
- The repo does not currently provide a full database migration setup or a complete security configuration; work in this codebase should assume basic bootstrapping is still in progress and avoid assuming the schema or auth layer is fully finished.

## Key conventions and gotchas

- Follow the repo's explicit Java and Next.js guidance in `AGENTS.md` and `client/my-app/AGENTS.md`.
- `backend` uses Spring Boot 4.1.1 and Java 17. Do not use older Spring Boot starter names such as `spring-boot-starter-web`; the codebase expects `spring-boot-starter-webmvc`.
- Lombok is wired through the Maven compiler annotation processor configuration. New Lombok usage does not need extra POM changes.
- Flyway is enabled, but there are no migrations present in `backend/src/main/resources/db/migration`. Do not assume Hibernate is creating the full schema.
- The `User` entity is intended for GitHub OAuth data, but the schema is still incomplete; when schema work is needed, add migration SQL and ensure IDs are valid before relying on the table.
- The repo root intentionally has no `.gitignore` at the top level; keep the project-level rules in `backend/.gitignore` and `client/my-app/.gitignore`.
- In the client app, `LayoutProps<"/">` is a generated Next 16 type and should not be manually imported.
- `client/my-app/lib/utils.ts` re-exports `cn` from the standalone `cn` package rather than from `clsx` + `tailwind-merge`; the Tailwind theme tokens live in `client/my-app/app/globals.css`.
- Keep secrets and credentials in environment variables instead of hardcoding them into app code or config files.
- The generated `client/my-app/AGENTS.md` block is not a random comment block; it is part of the repo’s Next.js guidance and should remain intact.

## Working style expectations for this repo

- Prefer exact repo conventions over generic framework defaults when they differ.
- Validate changes with the smallest existing command that checks the modified behavior.
- Since the project is split across backend and frontend, run the nearest relevant validation command from the correct runtime rather than assuming a repo-wide script exists.
- Keep package names, folder names, and active configuration aligned with the repo’s `devPilot` naming and the `backend` / `client/my-app` layout.
