---
name: testing-bombar-backend
description: Run BOM-bar backend REST and H2 persistence checks locally without Flutter.
---

# Backend runtime testing

## Devin Secrets Needed
None for the default local profile. Read current datasource credentials from
`src/main/resources/application-db.properties`; do not assume production uses
the local credentials.

## Setup
- Use Java 21 and the Maven wrapper. If Central rate-limits, set
  `MVNW_REPOURL=https://maven-central.storage-download.googleapis.com/maven2`.
- Build with `./mvnw -B -q package -DskipTests` and run the resulting jar.
- Default port is 8080; the default file database is `~/bombar_db.mv.db`.
  Preserve existing data unless a reset is explicitly authorized.
- H2 console may be disabled in configuration. For local inspection only,
  pass `--spring.h2.console.enabled=true --spring.h2.console.path=/h2`.
  Log in at `/h2` using JDBC URL
  `jdbc:h2:file:~/bombar_db;NON_KEYWORDS=KEY`.
- Flutter under `ui/` is separate; backend startup does not serve that UI.

## Focused flow
- Browse `/actuator/health` and `/projects?q=`.
- Create a project via POST `/projects` with `{"title":"e2e"}` (201);
  retain returned UUID. Upload `src/test/resources/valid.spdx` as multipart
  field `file` to `/projects/{uuid}/upload` (200).
- Browse project, dependencies and `/packages?q=a`; use Chrome Pretty-print
  and zoom for readable evidence.
- Current fixture yields one `Some package`, version `1.2.3`, Apache-2.0.
  Dependency purl includes `pkg:` and version; package reference intentionally
  strips both (`maven/some/package`). Supplier retains `Organization: `.
- Dependency booleans serialize as `is_root`, `is_development`, `is_delivered`.
- In H2, quote `"flyway_schema_history"` and its lowercase column names.
  Schema history may contain a creation marker in addition to migrations;
  count versioned rows, not all rows.
- Query a projects/dependencies/packages join to establish real persisted
  associations. Restart gracefully without deleting the DB, reload the
  original UUID, and check Flyway reports schema up to date.
- Inspect startup/request/restart logs for exceptions and warnings. A clean
  H2 2.x flow does not establish legacy H2 1.4 file upgrade compatibility.
