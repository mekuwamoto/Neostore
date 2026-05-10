# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

Monorepo root with two git submodules and local dev infrastructure:

- `Neostore-front/` — Angular 21 SPA (submodule → github.com/mekuwamoto/Neostore-front)
- `Neostore-back/` — ASP.NET Core 10 Web API (submodule → github.com/mekuwamoto/Neostore-back)
- `docker-compose.yml` — local dev services (MariaDB + ministack)

Each submodule has its own `CLAUDE.md` with deeper architecture notes.

## Local Dev Services

```bash
docker compose up -d        # start MariaDB (port 3306) and ministack/LocalStack (port 4566)
docker compose down
```

| Service | Image | Port | Credentials |
|---------|-------|------|-------------|
| mariadb | mariadb:10.6 | 3306 | root/example, neo/neopass, db=neostore |
| ministack | ministackorg/ministack | 4566 | S3, DynamoDB, Lambda, RDS (local AWS mock) |

## Backend (Neostore-back)

Stack: ASP.NET Core 10, EF Core 9 + Pomelo (MySQL), MediatR (CQRS), FluentValidation, AutoMapper, Serilog, Scalar (OpenAPI UI).

Solution: `src/Neostore.slnx`

```bash
cd Neostore-back
dotnet restore src/Neostore.slnx
dotnet build src/Neostore.slnx -c Release
dotnet test src/Neostore.slnx
dotnet test src/Neostore.slnx --filter "FullyQualifiedName~<TestName>"
```

**Layer dependencies (outer → inner, no reverse):**

```
Api → Application → Domain
Api → Infrastructure → Persistence → Domain
```

**Adding a feature** follows CQRS pattern:
1. Domain entity in `Neostore.Domain/Entities/`
2. Command/Query + Handler in `Neostore.Application/Handlers/`
3. FluentValidation validator in `Neostore.Application/Validators/`
4. AutoMapper profile in `Neostore.Application/Mappings/MappingProfile.cs`
5. EF config + repository in `Neostore.Persistence/`
6. Controller endpoint in `Neostore.Api/`

Tests: xUnit + Moq. Use explicit types, not `var`.

## Frontend (Neostore-front)

Stack: Angular 21, TypeScript 5.9, Tailwind CSS 4, Vitest.

```bash
cd Neostore-front
npm install
npm start           # dev server → localhost:4200
npm run build       # production build → dist/
npm test            # Vitest
```

## Branch Strategy

```
feature/* → (CI: build + test) → auto-PR to develop
develop   → auto-PR to main
```

CI workflows live inside each submodule under `.github/workflows/`.
