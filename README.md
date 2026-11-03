# engplat: English Mastery Platform

A self-hosted, single-user web app for moving from B2 to C1/C2 English:

- **Refactorer**: paste text and an LLM returns a C1/C2 rewrite plus discrete suggestions.
- **Cards**: turn suggestions (or your own notes) into active-production flashcards.
- **Review room**: cards are scheduled with FSRS (Again / Hard / Good / Easy).
- **Dashboard**: due counts, reviews today, 30-day retention.

Spring Boot (Java) serves both the REST API and the built Angular app from a single jar,
backed by PostgreSQL.

> **Security warning: there is no login.** The app binds to `127.0.0.1` by default and must
> stay that way unless something else controls access. To use it from other devices, put it
> behind [Tailscale](https://tailscale.com) or a reverse proxy with basic auth.
> **Never expose the port directly to the internet**: anyone who reaches it can read your data
> and `/api/refactor` spends your Anthropic API credit.

## Prerequisites

- Java 21+ (25 LTS preferred)
- Node.js LTS + npm
- Docker with Docker Compose

## Setup

```bash
cp .env.example .env          # then edit DB_PASSWORD (and ANTHROPIC_API_KEY if you want the Refactorer)
docker compose up -d          # start Postgres on 127.0.0.1:5432
set -a && . ./.env && set +a  # export the variables into your shell for the backend
```

`ANTHROPIC_API_KEY` is optional. Without it the app still runs; only the Refactorer is
disabled.

## Commands

```bash
docker compose up -d                  # start Postgres
cd backend && ./mvnw spring-boot:run  # API on 127.0.0.1:8080
cd backend && ./mvnw verify           # unit + integration tests (needs Docker)
cd frontend && npm start              # UI on :4200 with /api proxy
cd frontend && npm test
cd backend && ./mvnw spotless:apply   # format Java
cd frontend && npm run format         # Prettier write
cd frontend && npm run check          # lint + format check + tests
./scripts/build.sh                    # production jar with UI bundled
java -jar backend/target/*.jar        # run self-hosted
```

In development, open <http://localhost:4200>; the Angular dev server proxies `/api` to the
backend. In production (`java -jar`), everything is served from <http://127.0.0.1:8080>.

## Layout

```
docker-compose.yml   Postgres only, bound to 127.0.0.1
.env.example         environment variables (copy to .env)
scripts/build.sh     builds the UI, bundles it into the backend jar
backend/             Spring Boot API (Maven)
frontend/            Angular app
```
