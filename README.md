# biblioteca

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Java 17](https://img.shields.io/badge/Java-17-orange)
![Spring Boot 4](https://img.shields.io/badge/Spring%20Boot-4-brightgreen)
![React 19](https://img.shields.io/badge/React-19-61DAFB)
![Oracle XE 21c](https://img.shields.io/badge/Oracle-XE%2021c-red)

A small library system built end to end: a **REST API** with its rules enforced by the database, a **shared API contract**, and a **React panel** for the librarian. This repository is the front door of the project: it starts everything with one command and explains how the pieces fit.

> All data in this project is fictional. It is a portfolio project and is not related to any real organization.

![Architecture](docs/architecture.png)

| | |
|---|---|
| ![Catalog](docs/screenshots/catalogo.png) | ![Member with a fine](docs/screenshots/socio.png) |
| ![An overdue loan](docs/screenshots/prestamo.png) | ![Pending fines](docs/screenshots/multas.png) |

## Run it

You need [Docker](https://www.docker.com/) (with Compose). Nothing else: Java, Node and Oracle all run in containers.

```bash
git clone https://github.com/caronteisback/biblioteca.git
cd biblioteca
cp .env.example .env
docker compose up --build
```

Open <http://localhost:8081>.

The first start downloads the Oracle image and builds the API and the panel, so it takes a few minutes; later starts take about a minute. The API loads a made-up library on its first start, so every screen has something to show. To stop it, press `Ctrl+C`; to remove the containers and the database, run `docker compose down -v`.

The example values in `.env.example` are fine for a demo on your own machine: the database has no published port, so only the API can reach it. Change them for anything else.

### What is in the demo

20 books with their copies and 8 members, with loans in every state: due soon, renewed, overdue, returned late with a **pending fine** (that member cannot borrow), returned late and paid, a deactivated member and a retired copy. Try lending a copy, renewing the overdue loan (it is refused, and the panel says why) or paying the fine.

## The pieces

| Repository | What it is | Version |
|------------|------------|---------|
| [biblioteca-java](https://github.com/caronteisback/biblioteca-java) | The API: Java 17, Spring Boot 4, JPA and Oracle XE 21c, with 312 tests on a real database | v0.2.0 |
| [biblioteca-web](https://github.com/caronteisback/biblioteca-web) | The panel: React, TypeScript and Vite, with 69 tests including a whole-journey test and an accessibility check of every screen | v0.1.1 |
| [biblioteca-contrato](https://github.com/caronteisback/biblioteca-contrato) | The contract: one OpenAPI 3.1 file that the API is tested against and the panel's types are generated from | v0.1.1 |

`docker-compose.yml` builds the API and the panel from their repositories at those tags, so what runs is what each tag released. To build from folders on your machine instead, set `API_SOURCE` and `WEB_SOURCE` in `.env` (see `.env.example`).

## Ideas worth a look

- **Contract first.** The API and the panel never describe each other by hand: both follow `openapi.yaml`. The panel's types are generated from it, so a call the contract does not describe does not compile, and a test calls the real API and checks every status, body and error code against the contract.
- **Rules where they cannot be skipped.** A copy cannot have two active loans because the database says so (a unique index), and the services say it first with a clear error. Concurrent requests are serialized with row locks in a fixed order, and tests with real threads prove it.
- **Calculated, never stored.** The member's score, the alerts and the availability of a copy are computed from the history when asked, so they cannot go stale.
- **One error shape.** Every failure is `{ code, message }` with a stable code; the panel has a Spanish message for each code, and the build fails if the contract adds a code without one.
- **Safe by default.** No secrets in the repositories, the database is not reachable from outside, the panel is served by nginx without root with a strict Content-Security-Policy, and the API never leaks SQL or stack traces.

## Limits

- **There is no login yet.** The API has no authentication, so the panel is bound to `localhost` and must not be exposed to a network. Authentication with JWT and roles is the next step.
- The panel's screens are in Spanish; the code, the API contract and the documentation are in English.

## License

[MIT](LICENSE).
