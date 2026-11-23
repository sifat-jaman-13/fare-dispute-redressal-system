<div align="center">

# 🚌 Public Transport Fare Dispute Redressal System

### A relational database for fare transparency, passenger complaints, and transport analytics.

A CSE311L database project that models approved route fares, passenger overcharge complaints, administrator reviews, and audit-friendly reporting.

[![MySQL](https://img.shields.io/badge/MySQL-8.0.16%2B-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)
[![MariaDB](https://img.shields.io/badge/MariaDB-10.4%2B-003545?logo=mariadb&logoColor=white)](https://mariadb.org/)
[![SQL](https://img.shields.io/badge/Language-SQL-336791?logo=databricks&logoColor=white)](https://en.wikipedia.org/wiki/SQL)
[![Course](https://img.shields.io/badge/Course-CSE311L-red)](https://ece.northsouth.edu/courses/cse-311/)
[![License](https://img.shields.io/badge/License-Proprietary%20Non--Commercial-orange)](LICENSE)

[**ERD & Schema PDF**](docs/CSE311L_Team06_ERD_Schema_Design.pdf) · [SQL Script](fare_dispute_redressal.sql) · [Repository](https://github.com/sifat-jaman-13/fare-dispute-redressal-system)

</div>

---

## ✦ Project overview

The **Public Transport Fare Dispute Redressal System** is a relational database designed to support approved public-transport fare lookup, passenger overcharge complaints, administrator review, and transport data analytics.

It models:

- Passengers and administrators
- Bus operators and individual buses
- Routes and their ordered stops
- Approved fares with effective date ranges
- Passenger complaints and optional evidence-file references
- Administrator decisions and complaint review history
- Views for fare lookup, complaint details, and transport analytics

> **Scope:** This repository contains the SQL database implementation, sample data, verification queries, and ERD/schema documentation. It does **not** include a finished web frontend, mobile app, REST API, authentication service, or photo-upload implementation.

## ✦ What the database can do

| Capability | Implementation |
|---|---|
| User roles | Passenger and Admin roles stored in `USER` |
| Route modelling | `ROUTE`, `STOP`, and ordered `ROUTE_STOP` relationships |
| Fare management | Approved fares with effective start and end dates |
| Complaint filing | Passenger complaints linked to a bus and fare |
| Tracking | Automatically generated `FD-YYMMDD-XXXXXX` tracking number |
| Review history | Separate records for administrator decisions |
| Status updates | Complaint status is synchronized when a review is inserted |
| Reporting | Four views for complaint details, current fares, operator statistics, and route statistics |
| Integrity | Foreign keys, unique constraints, CHECK constraints, and triggers |

### Example workflow

```text
Admin configures routes, stops, buses and approved fares
                         |
                         v
              Passenger checks a fare
                         |
                         v
              Passenger files a complaint
                         |
                         v
              Database generates tracking ID
                         |
                         v
              Admin records a review decision
                         |
                         v
              Complaint status is updated
                         |
                         v
             Views provide reports and analytics
```

## ✦ Database design

The SQL script creates **10 tables, 4 views, 6 explicitly declared secondary indexes, and 7 triggers**.

### Entity relationship diagram

The project's approved ERD and relational schema documentation is available here:

📄 **[Open ERD & Schema Design (PDF)](docs/CSE311L_Team06_ERD_Schema_Design.pdf)**

### Tables

| Table | Purpose |
|---|---|
| `USER` | Passenger and administrator accounts/role records |
| `BUS_OPERATOR` | Bus operators and licence details |
| `BUS` | Individual buses assigned to operators |
| `ROUTE` | Public transport routes |
| `STOP` | Reusable bus stops |
| `ROUTE_STOP` | Many-to-many route/stop mapping and stop sequence |
| `FARE` | Approved fare between two stops on a route |
| `COMPLAINT` | Passenger fare dispute reports |
| `COMPLAINT_PHOTO` | References to optional evidence file paths |
| `COMPLAINT_REVIEW` | Administrator decisions and review history |

### Views

| View | Purpose |
|---|---|
| `v_complaint_details` | Complaint information with calculated fare difference |
| `v_operator_complaints` | Complaint totals and positive overcharge totals by operator |
| `v_route_disputes` | Complaint totals and average positive overcharge by route |
| `v_current_fares` | Fare records active on the current database date |

### Triggers and business rules

The database uses triggers to enforce rules that go beyond basic foreign keys:

- Only an Admin user can create or update fare records.
- Only a Passenger user can create or update a complaint.
- Only an Admin user can create or update a complaint review.
- Boarding and destination stops must be different.
- Both fare stops must belong to the selected route.
- A complaint's journey date must fall within the linked fare's effective period.
- A tracking number is generated when a complaint is inserted without one.
- A new review record updates the linked complaint's current status.

**Review-history detail:** Every review insertion is retained as a separate row. The status-sync trigger runs after review insertion; editing an existing review row does not run that status-sync trigger.

### Fare calculations

The complaint-details view calculates:

`fare_difference = amount_paid - approved_fare`

The operator and route analytics views use `GREATEST(amount_paid - approved_fare, 0)` so a complaint with no overcharge does not reduce aggregate positive-overcharge figures. The detailed complaint view can still show a negative fare difference.

### Important database limitations

- The schema does not prevent overlapping effective date ranges for otherwise matching fare records.
- The current-fare view filters by today's database date; it does not choose between overlapping active records.
- `COMPLAINT_PHOTO.file_path` stores a path/reference only. The repository does not upload, store, or serve image files.
- The sample `password_hash` values are dummy placeholders. There is no login or password-verification implementation in this repository.
- Role checks are database triggers, not a substitute for application-level authentication and authorization.

## ✦ Requirements

Choose **one** supported database environment:

| Requirement | Version / notes |
|---|---|
| MySQL Server | **8.0.16 or newer** |
| MariaDB Server | **10.4 or newer** |
| phpMyAdmin | Optional graphical interface for importing and querying SQL |
| MySQL/MariaDB command-line client | Optional; needed for terminal-based import and queries |
| XAMPP | Optional package that provides a local database environment and phpMyAdmin |

The project has no Node.js, Python, Java, PHP, Composer, or npm package dependencies. You do not need to install a programming-language runtime to run the SQL database script.

**Why MySQL 8.0.16?** The script relies on enforced CHECK constraints, which are enforced by MySQL starting with 8.0.16.

## ✦ Installation

### Option A — XAMPP (Windows)

This is the easiest route if you want a local database server and a graphical SQL interface.

#### 1. Install XAMPP

1. Download XAMPP from [Apache Friends](https://www.apachefriends.org/download.html).
2. Run the installer.
3. Install the MySQL/MariaDB database component and phpMyAdmin.
4. Finish the installation. The default location is usually `C:\xampp`.

XAMPP bundles MariaDB rather than Oracle MySQL. Check the bundled MariaDB version and make sure it is **10.4 or newer**.

#### 2. Start the database

1. Open **XAMPP Control Panel**.
2. Start **MySQL**.
3. Open [http://localhost/phpmyadmin](http://localhost/phpmyadmin) in your browser.

Apache is only needed to serve phpMyAdmin through the XAMPP web interface. It is not a dependency of the SQL database itself.

### Option B — Install MySQL or MariaDB directly

If you do not want XAMPP, install one database server:

- [MySQL Community Server](https://dev.mysql.com/downloads/mysql/)
- [MariaDB Server](https://mariadb.org/download/)

During installation, configure a database administrator account and remember its password. Install the command-line client if the installer offers it as a separate component.

Verify that the server is available:

```bash
mysql --version
```

If your system uses the MariaDB client, the command may also be available as:

```bash
mariadb --version
```

> You only need one database engine. Do not install MySQL and MariaDB side by side unless you know how to configure separate ports and services.

## ✦ Get the project

Clone the repository:

```bash
git clone https://github.com/sifat-jaman-13/fare-dispute-redressal-system.git
cd fare-dispute-redressal-system
```

Or download the repository as a ZIP from GitHub and extract it.

The SQL file you need is:

```text
fare_dispute_redressal.sql
```

## ✦ Import the database

<badge color="warning">Destructive import</badge>

**Back up any existing database named `fare_dispute_db` before importing.** The SQL script starts with `DROP DATABASE IF EXISTS fare_dispute_db`; importing it deletes and recreates that database, including its existing data.

### Method 1 — phpMyAdmin

1. Start the database service.
2. Open [http://localhost/phpmyadmin](http://localhost/phpmyadmin), or open phpMyAdmin through your own hosting environment.
3. Select the **Import** tab.
4. Choose `fare_dispute_redressal.sql`.
5. Keep the format set to **SQL**.
6. Click **Import** or **Go**.
7. Wait for the success message.
8. Select `fare_dispute_db` in the left sidebar.

The script creates the database, tables, constraints, indexes, triggers, views, sample records, and commented demo queries.

### Method 2 — MySQL / MariaDB command line

Run the command from the directory containing `fare_dispute_redressal.sql`.

**MySQL or MariaDB client:**

```bash
mysql -u root -p < fare_dispute_redressal.sql
```

Or:

```bash
mariadb -u root -p < fare_dispute_redressal.sql
```

Enter the database administrator password when prompted.

**Windows with the default XAMPP installation:**

Open Command Prompt in the project directory.

Default port `3306`:

```cmd
"C:\xampp\mysql\bin\mysql.exe" -u root -p < fare_dispute_redressal.sql
```

If you configured MySQL/MariaDB to use port `3307`:

```cmd
"C:\xampp\mysql\bin\mysql.exe" -P 3307 -u root -p < fare_dispute_redressal.sql
```

If your local XAMPP root account has no password, press **Enter** when prompted. Do not leave a production database administrator account without a password.

## ✦ Verify the installation

Open the SQL tab in phpMyAdmin or connect with your database client, then run:

```sql
USE fare_dispute_db;

SHOW TABLES;

SHOW FULL TABLES
WHERE Table_type = 'VIEW';

SELECT COUNT(*) AS total_complaints
FROM COMPLAINT;

SELECT *
FROM v_current_fares;

SELECT tracking_number,
       passenger_name,
       route_name,
       bus_number,
       approved_fare,
       amount_paid,
       fare_difference,
       status
FROM v_complaint_details;
```

The sample dataset should contain:

| Object / record | Expected count |
|---|---:|
| Base tables | 10 |
| Views | 4 |
| Explicit secondary indexes | 6 |
| Triggers | 7 |
| Users | 5 |
| Bus operators | 3 |
| Buses | 4 |
| Routes | 2 |
| Stops | 7 |
| Route-stop rows | 10 |
| Fare records | 5 |
| Complaints | 4 |
| Complaint photo references | 3 |
| Complaint reviews | 3 |

## ✦ Demo queries

Try these queries after importing the sample data.

### 1. Complaint tracking and fare difference

```sql
SELECT tracking_number,
       passenger_name,
       route_name,
       bus_number,
       approved_fare,
       amount_paid,
       fare_difference,
       status
FROM v_complaint_details;
```

### 2. Operator complaint analytics

```sql
SELECT operator_name,
       operator_status,
       total_complaints,
       total_overcharge AS total_overcharge_bdt
FROM v_operator_complaints
ORDER BY total_complaints DESC, total_overcharge DESC;
```

### 3. Route dispute analytics

```sql
SELECT route_name,
       total_complaints,
       ROUND(avg_overcharge, 2) AS avg_overcharge_bdt
FROM v_route_disputes
ORDER BY total_complaints DESC, avg_overcharge DESC;
```

### 4. Current fare lookup

```sql
SELECT *
FROM v_current_fares
WHERE route_name = 'Mirpur-10 to Motijheel';
```

### 5. Complaint review audit trail

```sql
SELECT c.tracking_number,
       u.full_name AS admin_name,
       cr.decision,
       cr.remarks,
       cr.reviewed_at
FROM COMPLAINT_REVIEW cr
JOIN COMPLAINT c ON c.complaint_id = cr.complaint_id
JOIN `USER` u ON u.user_id = cr.admin_id
ORDER BY cr.reviewed_at ASC;
```

## ✦ Test the business constraints

The following statements are expected to fail because they violate trigger-enforced rules. Run them **one at a time** after importing the database.

### Passenger attempting to review a complaint

```sql
INSERT INTO COMPLAINT_REVIEW
    (complaint_id, admin_id, decision, remarks)
VALUES
    (1, 3, 'Resolved', 'Unauthorized passenger review');
```

Expected: the database rejects the insert because user 3 is a Passenger, not an Admin.

### Boarding stop and destination stop are identical

```sql
INSERT INTO FARE
    (route_id, from_stop_id, to_stop_id, approved_fare, effective_from, managed_by)
VALUES
    (1, 1, 1, 15.00, '2026-01-01', 1);
```

Expected: the database rejects the fare because both stops are identical.

### Journey date is outside the fare validity period

```sql
INSERT INTO COMPLAINT
    (passenger_id, bus_id, fare_id, journey_date, amount_paid, description)
VALUES
    (3, 1, 1, '2025-06-15', 50.00, 'Backdated journey complaint');
```

Expected: the database rejects the complaint because the journey date is earlier than the linked fare's effective date.

> These statements are negative tests. They should not create records. Do not re-import the full SQL script just to run a test; the import script drops and recreates the database.

## ✦ Repository structure

```text
fare-dispute-redressal-system/
├── docs/
│   └── CSE311L_Team06_ERD_Schema_Design.pdf
├── fare_dispute_redressal.sql
├── LICENSE
└── README.md
```

## ✦ Troubleshooting

| Problem | What to do |
|---|---|
| MySQL/MariaDB service will not start | Check whether another database service is already using the configured port. Stop the conflicting service or configure a different port consistently. |
| Port 3306 is already in use | Check the active MySQL/MariaDB service before changing XAMPP's port. If you change it to 3307, use that port for both the server and client connection. |
| `mysql` is not recognized | Add the MySQL/MariaDB `bin` directory to PATH, reopen the terminal, or use the full executable path shown in the XAMPP instructions. |
| Access denied for user | Check the username, password, host, and port. Use the credentials configured when installing the database server. |
| phpMyAdmin cannot connect | Confirm the database service is running and phpMyAdmin is configured for the same host and port. |
| CHECK constraints are not enforced | Confirm you are using MySQL 8.0.16+ or MariaDB 10.4+. Older MySQL versions may parse CHECK syntax without enforcing it. |
| Import stops at a trigger or delimiter statement | Use the full SQL script in phpMyAdmin's Import tab or the MySQL/MariaDB command-line client. Do not execute the script as isolated statements in an interface that does not support delimiter changes. |
| Database already contains old data | Back it up first. The script drops and recreates `fare_dispute_db`. |

## ✦ Academic information

| Detail | Information |
|---|---|
| Institution | North South University (NSU) |
| Department | Electrical & Computer Engineering (ECE) |
| Course | CSE311L — Database Systems Lab |
| Section | 03 |
| Team | Team 06 |
| Faculty | Ms. Tanzilah Noor Shabnam (`TnS1`) |
| Lab Instructor | Md. Amanullah Shah |

### Team members

| Student | Project responsibility |
|---|---|
| **Md Sifat Jaman** | Project Coordinator & Database Designer — ERD, logical schema, documentation, and coordination |
| **Sabbir Ahmed** | Database Designer & Frontend Developer — schema modelling, DDL implementation, and interface planning |
| **Progna Thakur** | Database Engineer & Backend Developer — referential integrity, triggers, views, and backend connection |
| **Tausif Taher** | Database Administrator & System Tester — test data, edge-case validation, and integrity testing |

Student ID digits are masked in this public repository for privacy.

## ✦ Learning and academic integrity

This project was developed as a university database lab project. Students are encouraged to study its ERD, relational schema, constraints, triggers, and reporting queries as a learning reference.

Use the ideas as inspiration to design a similar or improved system with your own requirements, schema decisions, implementation, and documentation. Experiment with additional features, improve the business rules, and explain your design choices. Do not copy the project wholesale and submit it as your own coursework.

## ✦ Licence

This project is distributed under a [proprietary non-commercial use licence](LICENSE).

- **Permitted:** Individual personal and educational use under the licence terms, with attribution.
- **Not permitted without prior written permission:** Company, organisational, business, or other commercial use.
- **Team project:** Commercial licensing or other commercial dealings require prior written approval from every co-author and copyright holder whose rights are affected.
- **Third-party materials:** Remain subject to their own licences and notices.

The licence does not grant permission to publicly redistribute the project. See [LICENSE](LICENSE) for the complete terms.

---

<div align="center">

**Designed for transparent fares. Built for accountable transport systems.**

[Repository](https://github.com/sifat-jaman-13/fare-dispute-redressal-system) · [ERD & Schema PDF](docs/CSE311L_Team06_ERD_Schema_Design.pdf)

</div>
