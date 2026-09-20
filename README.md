# Akademos — Student Result Management System

A complete Spring Boot 3 + Java 17 application for managing students, subjects, and exam results, with automatic grade/pass-fail calculation and report card generation. Ships with an H2 in-memory database (zero setup) and a built-in vanilla HTML/CSS/JS front end — no Node.js needed.

## ⚠️ Important — please build it once before relying on it

This project was written and carefully reviewed by hand, but **it has not been compiled or run** in the environment it was generated in (Maven Central wasn't reachable there). Run the build steps below as your first step. If you hit any compiler error, paste it back to me and I'll fix it immediately — that's much faster than me guessing blind.

## Requirements

- Java 17 or newer (`java -version`)
- Maven 3.6+ (`mvn -version`) — or use an IDE (IntelliJ / Eclipse / VS Code) which bundles its own
- Internet access on first build (Maven needs to download dependencies once; they're cached after that)

## Project structure

```
student-result-management/
├── pom.xml
├── src/main/java/com/srms/app/
│   ├── StudentResultManagementApplication.java   # main() entry point
│   ├── model/        # Student, Subject, Result entities + Grade enum
│   ├── repository/   # Spring Data JPA repositories
│   ├── service/       # business logic (grade calc, validation, report cards)
│   ├── controller/   # REST API endpoints (/api/students, /api/subjects, /api/results)
│   ├── dto/          # request/response shapes
│   ├── exception/    # custom exceptions + a global @RestControllerAdvice handler
│   └── config/       # CORS config + DataSeeder (sample data on first run)
├── src/main/resources/
│   ├── application.properties   # H2 config (MySQL config included, commented out)
│   └── static/                  # the front end: index.html, css/style.css, js/app.js
└── src/test/java/...            # a basic context-load smoke test
```

## How to run

### Option 1 — command line

```bash
cd student-result-management
mvn spring-boot:run
```

### Option 2 — build a jar and run it

```bash
mvn clean package
java -jar target/student-result-management.jar
```

### Option 3 — your IDE

Import as a Maven project, then run `StudentResultManagementApplication.java` directly.

Once it's up, open: **http://localhost:8080**

The H2 console (to inspect the database directly) is at **http://localhost:8080/h2-console**
JDBC URL: `jdbc:h2:mem:srmsdb`, username `sa`, no password.

On first run, `DataSeeder` populates 3 sample students, 4 subjects, and their results, so the dashboard isn't empty.

## Switching to MySQL

H2 is in-memory — your data resets every restart. To persist data with MySQL:

1. Create the database: `CREATE DATABASE srmsdb;`
2. In `src/main/resources/application.properties`, comment out the H2 block and uncomment the MySQL block, filling in your username/password.
3. Re-run. Hibernate will auto-create the tables (`ddl-auto=update`).

## REST API reference

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/students` | List all students (supports `?search=` and `?department=`) |
| GET | `/api/students/{id}` | Get one student |
| POST | `/api/students` | Create a student |
| PUT | `/api/students/{id}` | Update a student |
| DELETE | `/api/students/{id}` | Delete a student (cascades their results) |
| GET | `/api/subjects` | List all subjects |
| POST | `/api/subjects` | Create a subject |
| PUT | `/api/subjects/{id}` | Update a subject |
| DELETE | `/api/subjects/{id}` | Delete a subject |
| GET | `/api/results` | List all results |
| GET | `/api/results/student/{studentId}` | All results for one student |
| GET | `/api/results/student/{studentId}/report-card` | Full computed report card (grades, pass/fail, totals) |
| POST | `/api/results` | Record a result (marks are validated against the subject's max marks) |
| PUT | `/api/results/{id}` | Update a result |
| DELETE | `/api/results/{id}` | Delete a result |

All endpoints return JSON. Validation errors return `400` with a `fieldErrors` map; not-found returns `404`; duplicate roll numbers/subject codes return `409`.

## Grading logic

Defined in `Grade.java` / applied in `ResultService.java`:

| Percentage | Grade |
|---|---|
| ≥ 90% | O (Outstanding) |
| ≥ 80% | A+ |
| ≥ 70% | A |
| ≥ 60% | B+ |
| ≥ 50% | B |
| ≥ passing marks | C |
| below passing marks | F |

Each `Subject` defines its own `maxMarks` and `passingMarks`, so grading thresholds are per-subject.

## Notes on what's already handled

- Duplicate roll numbers / subject codes are rejected with a clear `409` message.
- Marks exceeding a subject's max marks are rejected with `400`.
- Deleting a student or subject cleans up their associated results first (no orphaned rows).
- CORS is open (`*`) for local development — tighten this in `WebConfig.java` before any real deployment.
- Lombok is used throughout (`@Data`, `@RequiredArgsConstructor`) — make sure your IDE has the Lombok plugin installed, or annotation processing enabled, or you'll see "method not found" errors that aren't real compile errors.
