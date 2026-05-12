# IntraConnect — Backend API

NestJS 11 · Prisma 5 · PostgreSQL · TypeScript 5

---

## Stack

| Layer | Technology |
|---|---|
| Framework | NestJS 11 |
| ORM | Prisma 5 |
| Database | PostgreSQL 14+ |
| Auth | JWT (httpOnly cookie) |
| File Storage | S3-compatible (MinIO for local dev) |
| Runtime | Node.js 23 |

---

## Prerequisites

- Node.js ≥ 23
- PostgreSQL 14+ running locally
- (Optional) MinIO for local file storage

---

## 1 — Clone and install

```bash
git clone <repo-url>
cd intranet-be
npm install
```

---

## 2 — Environment variables

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

### Required variables

| Variable | Description | Default |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | — |
| `JWT_SECRET` | Secret key for signing JWT tokens | — |
| `PORT` | HTTP port the API listens on | `3001` |
| `NODE_ENV` | `development` / `production` | `development` |
| `FRONTEND_URL` | Allowed CORS origin | `http://localhost:3000` |

### Password policy (all optional)

| Variable | Description | Default |
|---|---|---|
| `PWD_MIN_LENGTH` | Minimum password length | `8` |
| `PWD_REQUIRE_UPPER` | Require uppercase letter | `true` |
| `PWD_REQUIRE_LOWER` | Require lowercase letter | `true` |
| `PWD_REQUIRE_DIGIT` | Require digit | `true` |
| `PWD_REQUIRE_SYMBOL` | Require special character | `false` |

### Account lockout (optional)

| Variable | Description | Default |
|---|---|---|
| `LOCKOUT_MAX_ATTEMPTS` | Failed logins before lockout | `5` |
| `LOCKOUT_MINUTES` | Lock duration in minutes | `15` |

### SMTP / Email (optional — disables OTP emails if absent)

| Variable | Description |
|---|---|
| `SMTP_HOST` | SMTP server hostname |
| `SMTP_PORT` | SMTP port (usually `587` or `465`) |
| `SMTP_SECURE` | `true` for TLS (port 465), `false` for STARTTLS |
| `SMTP_USER` | SMTP username |
| `SMTP_PASS` | SMTP password |
| `SMTP_FROM` | Sender address, e.g. `"IntraConnect <no-reply@example.com>"` |

### S3 / MinIO file storage (optional)

| Variable | Description | Default |
|---|---|---|
| `S3_ENDPOINT` | S3 endpoint URL | `http://localhost:9000` |
| `S3_REGION` | AWS/MinIO region | `us-east-1` |
| `S3_ACCESS_KEY` | Access key ID | `minioadmin` |
| `S3_SECRET_KEY` | Secret access key | `minioadmin` |
| `S3_BUCKET` | Bucket name | `intraconnect` |
| `S3_FORCE_PATH_STYLE` | Required for MinIO | `true` |
| `S3_SIGNED_URL_TTL` | Pre-signed URL TTL in seconds | `300` |

**Generate a secure `JWT_SECRET`:**

```bash
node -e "require('crypto').randomBytes(48, (e,b) => console.log(b.toString('hex')))"
```

**Run MinIO locally (Docker):**

```bash
docker run -p 9000:9000 -p 9001:9001 \
  minio/minio server /data --console-address ":9001"
```

---

## 3 — Database setup

### Create the database

```bash
psql -U postgres -c "CREATE DATABASE intraconnect;"
```

### Run migrations

Applies all schema changes and keeps the database in sync with `prisma/schema.prisma`:

```bash
npx prisma migrate dev
```

For CI / production (no prompts, no schema drift check):

```bash
npx prisma migrate deploy
```

### Seed the database

Populates roles, permissions, departments, job titles, tools, ticket categories, and the four default user accounts:

```bash
npm run prisma:seed
```

---

## Seeded accounts

| Role | Email | Password |
|---|---|---|
| Admin | `admin@virtide.com` | `Admin123!` |
| HR | `hr@virtide.com` | `Hr123!` |
| Manager | `manager@virtide.com` | `Manager123!` |
| Employee | `employee@virtide.com` | `Employee123!` |

> Change all passwords immediately in any non-local environment.

What the seed creates:
- **Permissions** — 40 granular action/module pairs
- **Roles** — `admin`, `hr`, `manager`, `employee` (each with scoped permissions)
- **Departments** — IT, Human Resources, Operations, Finance
- **Job Titles** — CEO, CTO, HR Director, IT Manager, Senior Developer, Developer, HR Specialist
- **Tools** — 32 entries (Teams, Slack, Jira, GitLab, Figma, etc.)
- **Ticket Categories** — Hardware, Software, Network, Access & Security, Plumbing, Electrical, HVAC, Furniture, Janitorial, Other
- **Sample data** — one pending leave request and one open ticket for the employee account

---

## 4 — Run the server

```bash
# Development (watch mode)
npm run start:dev

# Debug mode
npm run start:debug

# Production build
npm run build
npm run start:prod
```

The API is available at `http://localhost:3001/api/v1`.

---

## Swagger / API docs

Browse to `http://localhost:3001/api/v1` while the server is running.
Click **Authorize** and paste a JWT token obtained from `POST /api/v1/auth/login`.

---

## Prisma commands

```bash
# Open the visual DB browser
npx prisma studio

# Generate the Prisma client after schema changes
npx prisma generate

# Create a new migration after editing schema.prisma
npx prisma migrate dev --name <migration-name>

# Reset the database and re-seed (DELETES ALL DATA)
npx prisma migrate reset
```

---

## Auth

Authentication uses **httpOnly cookie-based JWT** — the token is never exposed to client-side JavaScript.

**Frontend configuration required:**

- `axios`: set `withCredentials: true`
- `fetch`: set `credentials: 'include'`

**Flow:**
1. `POST /api/v1/auth/login` → sets `access_token` cookie
2. All subsequent requests send the cookie automatically
3. `POST /api/v1/auth/logout` → clears the cookie

---

## Testing

```bash
# Unit tests
npm run test

# Watch mode
npm run test:watch

# Coverage report
npm run test:cov

# End-to-end tests
npm run test:e2e
```

---

## Scripts reference

| Script | Description |
|---|---|
| `npm run start:dev` | Start in watch mode |
| `npm run start:debug` | Start with debugger attached |
| `npm run build` | Compile TypeScript to `dist/` |
| `npm run start:prod` | Run compiled production build |
| `npm run prisma:seed` | Seed the database |
| `npm run lint` | Lint and auto-fix |
| `npm run format` | Prettier format |
| `npm run test` | Run unit tests |
| `npm run test:cov` | Run tests with coverage |
