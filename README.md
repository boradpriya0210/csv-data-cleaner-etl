# CSV Data Cleaner - Mini ETL Tool

A Spring Boot application that runs a small **ETL (Extract, Transform, Load)** pipeline:
upload a CSV file, clean it, store it in a SQL database, and view KPI-style reports.

| Step | What the app does |
|---|---|
| **Extract** | Reads rows from an uploaded CSV file (Apache Commons CSV) |
| **Transform** | Trims spaces, lower-cases emails, removes missing values, invalid emails, invalid marks (not a number / outside 0-100), duplicates inside the file, and duplicates already in the database |
| **Load** | Saves clean rows into the `students` table using Spring Data JPA |
| **Report** | `GROUP BY`, `HAVING`, `AVG/MIN/MAX/COUNT` queries exposed as a REST API and a simple web page |

## Tech stack
Java 21, Spring Boot 3, Spring Data JPA / Hibernate, H2 (default) or MySQL, Apache Commons CSV, JUnit 5 + Mockito, Maven.

## How to run
Requirements: JDK 21+ and Maven.

```bash
mvn spring-boot:run
```
Open **http://localhost:8080** and upload `sample-data/students.csv`.

Run tests: `mvn test`

### Use MySQL instead of H2
1. Edit username/password in `src/main/resources/application-mysql.properties`
2. Run: `mvn spring-boot:run -Dspring-boot.run.profiles=mysql`

(The database `csv_cleaner` is created automatically.) H2 console for the default DB: http://localhost:8080/h2-console
(JDBC URL `jdbc:h2:file:./data/cleanerdb`, user `sa`, empty password).

## REST API
| Method | URL | Purpose |
|---|---|---|
| POST | `/api/upload` | Upload CSV (form field `file`), returns rows read / loaded / rejected with reasons |
| GET | `/api/students` | All clean records |
| GET | `/api/stats?minStudents=2` | Total, avg/min/max marks, department report, top 3 students |
| DELETE | `/api/students` | Delete all records |

Quick test with curl:
```bash
curl -F "file=@sample-data/students.csv" http://localhost:8080/api/upload
curl http://localhost:8080/api/stats
```

## Expected result for sample-data/students.csv
11 rows read -> **6 loaded, 5 rejected** (missing marks, duplicate email, invalid email, marks not a number, marks out of range).

## SQL concepts used
- Primary key (`id`) and unique key (`email`)
- `GROUP BY`, `HAVING`, `COUNT`, `AVG`, `MIN`, `MAX`, `ORDER BY ... LIMIT 3`

```sql
SELECT department, COUNT(*) AS total, AVG(marks) AS avg_marks
FROM students
GROUP BY department
HAVING COUNT(*) >= 2
ORDER BY AVG(marks) DESC;
```

## Project structure
```
src/main/java/com/digitalsweep/csvcleaner
  controller/   REST endpoints
  service/      ETL logic (CsvEtlService)
  repository/   JPA repository + custom queries
  model/        Student entity
  dto/          Response objects
src/main/resources/static/index.html   simple web UI
src/test/...                           unit tests
sample-data/students.csv               test file
```

## Future improvements
Support more file types (Excel/JSON), configurable column mapping, batch inserts for very large files, scheduled ETL jobs, and a Dockerfile.
