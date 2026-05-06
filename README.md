# Demo DAO JDBC

A didactic Java project that demonstrates the **DAO (Data Access Object)** design pattern using plain JDBC against a MySQL database. It models a simple HR-like domain (`Seller` and `Department`) and exposes the full CRUD lifecycle through clean DAO abstractions.

---

## Table of Contents

1. [Overview](#overview)
2. [Tech Stack](#tech-stack)
3. [Project Structure](#project-structure)
4. [Domain Model](#domain-model)
5. [Architecture](#architecture)
6. [Getting Started](#getting-started)
7. [Database Setup](#database-setup)
8. [Running the Application](#running-the-application)
9. [Available Operations](#available-operations)
10. [Design Patterns Used](#design-patterns-used)
11. [Roadmap](#roadmap)
12. [License](#license)

---

## Overview

This project illustrates how to:

- Decouple persistence logic from business logic using the **DAO pattern**.
- Wire up implementations through a **Factory**, isolating clients from concrete classes.
- Manage JDBC resources (`Connection`, `Statement`, `ResultSet`) safely.
- Map related tables (`seller` ↔ `department`) into rich object graphs while avoiding duplicate instantiations through an in-memory identity map.

It is intentionally minimal: no Spring, no Hibernate, no build framework — just the JDK, the MySQL JDBC driver, and a clear separation of concerns.

---

## Tech Stack

| Layer        | Technology               |
|--------------|--------------------------|
| Language     | Java 8+                  |
| Persistence  | JDBC                     |
| Database     | MySQL 5.7+ / 8.x         |
| IDE          | Eclipse (project files included) |
| Build        | Manual / Eclipse classpath |

---

## Project Structure

```
demo-dao-jdbc/
├── db.properties              # Database connection settings (DO NOT COMMIT REAL CREDENTIALS)
├── .gitignore
├── src/
│   ├── application/
│   │   ├── Program.java       # Manual test driver for SellerDao
│   │   └── Program2.java      # Manual test driver for DepartmentDao
│   ├── db/
│   │   ├── DB.java                    # Connection / resource utilities
│   │   ├── DbException.java           # Generic persistence error
│   │   └── DbIntegrityException.java  # Referential-integrity error
│   └── model/
│       ├── entities/
│       │   ├── Seller.java
│       │   └── Department.java
│       └── dao/
│           ├── DaoFactory.java        # Static factory for DAO instances
│           ├── SellerDao.java         # Seller persistence contract
│           ├── DepartmentDao.java     # Department persistence contract
│           └── impl/
│               ├── SellerDaoJDBC.java
│               └── DepartmentDaoJDBC.java
```

---

## Domain Model

```
+----------------+         +-------------------+
|   Department   | 1 ---- N|       Seller      |
+----------------+         +-------------------+
| id: Integer    |         | id: Integer       |
| name: String   |         | name: String      |
+----------------+         | email: String     |
                           | birthDate: Date   |
                           | baseSalary: Double|
                           | department: Dep.  |
                           +-------------------+
```

A `Seller` belongs to exactly one `Department`. A `Department` may have zero or many `Sellers`.

---

## Architecture

The project follows a **layered architecture** with strict directional dependencies:

```
application  ─►  model.dao  ─►  model.dao.impl  ─►  db
                     ▲                  │
                     └── model.entities ┘
```

| Layer            | Responsibility                                   |
|------------------|--------------------------------------------------|
| `application`    | Entry points / manual test drivers               |
| `model.entities` | Plain domain objects (POJOs)                     |
| `model.dao`      | DAO **contracts** (interfaces) + Factory         |
| `model.dao.impl` | JDBC-backed DAO implementations                  |
| `db`             | Connection management and persistence exceptions |

Clients depend on **interfaces** (`SellerDao`, `DepartmentDao`), never on concrete `*JDBC` classes — making it easy to swap implementations (e.g., a future `SellerDaoJPA`).

---

## Getting Started

### Prerequisites

- **JDK 8** or higher
- **MySQL Server** 5.7 or 8.x running locally
- **MySQL JDBC Driver** (`mysql-connector-j-x.x.x.jar`) on the classpath
- **Eclipse IDE** (optional, project files are included)

### Clone the Repository

```bash
git clone <your-repo-url>
cd demo-dao-jdbc
```

---

## Database Setup

Create the schema and seed data in MySQL:

```sql
CREATE DATABASE coursejdbc;
USE coursejdbc;

CREATE TABLE department (
    Id   INT AUTO_INCREMENT PRIMARY KEY,
    Name VARCHAR(60) NOT NULL
);

CREATE TABLE seller (
    Id            INT AUTO_INCREMENT PRIMARY KEY,
    Name          VARCHAR(60) NOT NULL,
    Email         VARCHAR(100) NOT NULL,
    BirthDate     DATETIME NOT NULL,
    BaseSalary    DOUBLE NOT NULL,
    DepartmentId  INT NOT NULL,
    FOREIGN KEY (DepartmentId) REFERENCES department(Id)
);

INSERT INTO department (Name) VALUES
    ('Computers'), ('Electronics'), ('Fashion'), ('Books');

INSERT INTO seller (Name, Email, BirthDate, BaseSalary, DepartmentId) VALUES
    ('Bob Brown',  'bob@gmail.com',   '1998-04-21 00:00:00', 1000.0, 1),
    ('Maria Green','maria@gmail.com', '1979-12-31 00:00:00', 3500.0, 2),
    ('Alex Grey',  'alex@gmail.com',  '1988-01-15 00:00:00', 2200.0, 1),
    ('Martha Red', 'martha@gmail.com','1993-11-30 00:00:00', 3000.0, 4),
    ('Donald Blue','donald@gmail.com','2000-01-09 00:00:00', 4000.0, 3),
    ('Bobby Green','bobby@gmail.com', '1983-04-06 00:00:00', 2500.0, 2);
```

### Configure Connection

Edit [db.properties](db.properties) at the project root:

```properties
user=YOUR_DB_USER
password=YOUR_DB_PASSWORD
dburl=jdbc:mysql://localhost:3306/coursejdbc
useSSL=false
```

> **Security note:** never commit real credentials. Add `db.properties` to `.gitignore` and provide a `db.properties.example` with placeholder values.

---

## Running the Application

### From Eclipse

1. Import the project: **File → Import → Existing Projects into Workspace**.
2. Add the **MySQL JDBC Driver** JAR to **User Libraries** and reference it in the build path.
3. Right-click `Program.java` or `Program2.java` → **Run As → Java Application**.

### From the Command Line

```bash
# Compile
javac -d bin -cp "lib/mysql-connector-j.jar" src/**/*.java

# Run the Seller demo
java -cp "bin;lib/mysql-connector-j.jar" application.Program

# Run the Department demo
java -cp "bin;lib/mysql-connector-j.jar" application.Program2
```

> On Linux/macOS replace `;` with `:` in the classpath separator.

---

## Available Operations

### `SellerDao`

| Method                                      | Description                                |
|---------------------------------------------|--------------------------------------------|
| `insert(Seller obj)`                        | Persists a new seller, populating its ID   |
| `update(Seller obj)`                        | Updates all fields of an existing seller   |
| `deleteById(Integer id)`                    | Removes a seller by primary key            |
| `findById(Integer id)`                      | Returns a seller (with department) or null |
| `findAll()`                                 | Returns every seller, ordered by name      |
| `findByDepartment(Department department)`   | Returns all sellers in a given department  |

### `DepartmentDao`

| Method                       | Description                              |
|------------------------------|------------------------------------------|
| `insert(Department obj)`     | Persists a new department                |
| `update(Department obj)`     | Updates an existing department           |
| `deleteById(Integer id)`     | Removes a department by primary key      |
| `findById(Integer id)`       | Returns a department or null             |
| `findAll()`                  | Returns every department, ordered by name|

---

## Design Patterns Used

- **DAO (Data Access Object)** — separates persistence from business logic.
- **Factory Method** — `DaoFactory` centralizes DAO instantiation.
- **Singleton (lightweight)** — `DB` keeps a single `Connection` per JVM.
- **Identity Map (in-memory)** — `findAll` / `findByDepartment` reuse `Department` instances loaded in the same query, avoiding redundant object creation for the same FK.

---

## Roadmap

Improvements that would bring the project closer to production-grade quality:

- [ ] Replace the static singleton `Connection` with a connection pool (HikariCP).
- [ ] Migrate build to **Maven** or **Gradle** with managed JDBC dependency.
- [ ] Add **JUnit 5** unit tests with an in-memory H2 database (target ≥ 75% coverage).
- [ ] Replace `java.util.Date` with `java.time.LocalDate`.
- [ ] Return `Optional<T>` from `findById` instead of `null`.
- [ ] Use `try-with-resources` for all `AutoCloseable` JDBC objects.
- [ ] Preserve original exception cause: `throw new DbException(msg, e)`.
- [ ] Introduce structured logging (SLF4J + Logback) and remove `System.out.println` calls.
- [ ] Move `db.properties` out of version control and ship `db.properties.example`.

---

## License

This project is provided as-is for educational purposes. Feel free to fork, adapt, and learn from it.
