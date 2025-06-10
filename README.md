# PostgreSQL Sample Database - DVD Rental

## Overview

This repository contains the **DVD Rental** sample database for PostgreSQL, which represents the business processes of a DVD rental store. This database is perfect for learning and practicing PostgreSQL features and SQL queries.

## Database Structure

The DVD Rental database contains:
- **15 tables**
- **1 trigger**
- **7 views**
- **8 functions**
- **1 domain**
- **13 sequences**

## ER Diagram

The complete Entity-Relationship diagram is available in the file `printable-postgresql-sample-database-diagram.pdf`. In the diagram, the asterisk (*) in front of a field indicates the primary key.

### Database Schema Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   country   │    │    city     │    │   address   │
│*country_id  │────│*city_id     │────│*address_id  │
│ country     │    │ city        │    │ address     │
│ last_update │    │ country_id  │    │ address2    │
└─────────────┘    │ last_update │    │ district    │
                   └─────────────┘    │ city_id     │
                                      │ postal_code │
                                      │ phone       │
                                      │ last_update │
                                      └─────────────┘
                                            │
                   ┌─────────────┐    ┌─────────────┐
                   │   customer  │    │    staff    │
                   │*customer_id │    │*staff_id    │
                   │ store_id    │    │ first_name  │
                   │ first_name  │    │ last_name   │
                   │ last_name   │    │ address_id  │
                   │ email       │    │ email       │
                   │ address_id  │────│ store_id    │
                   │ activebool  │    │ active      │
                   │ create_date │    │ username    │
                   │ last_update │    │ password    │
                   │ active      │    │ last_update │
                   └─────────────┘    │ picture     │
                         │            └─────────────┘
                         │                  │
                   ┌─────────────┐    ┌─────────────┐
                   │   rental    │    │    store    │
                   │*rental_id   │    │*store_id    │
                   │ rental_date │    │ manager_    │
                   │ inventory_id│    │ staff_id    │
                   │ customer_id │────│ address_id  │
                   │ return_date │    │ last_update │
                   │ staff_id    │────└─────────────┘
                   │ last_update │
                   └─────────────┘
                         │
                   ┌─────────────┐
                   │   payment   │
                   │*payment_id  │
                   │ customer_id │
                   │ staff_id    │
                   │ rental_id   │────
                   │ amount      │
                   │ payment_date│
                   └─────────────┘
```

## Database Tables

### Core Tables

| Table | Description |
|-------|-------------|
| **actor** | Stores actor data including first name and last name |
| **film** | Stores film data such as title, release year, length, rating, etc. |
| **film_actor** | Stores the relationships between films and actors |
| **category** | Stores film category data |
| **film_category** | Stores the relationships between films and categories |
| **store** | Contains store data including manager staff and address |
| **inventory** | Stores inventory data |
| **rental** | Stores rental transaction data |
| **payment** | Stores customer payment information |
| **staff** | Stores staff data |
| **customer** | Stores customer data |
| **address** | Stores address data for staff and customers |
| **city** | Stores city names |
| **country** | Stores country names |
| **language** | Stores language information for films |

### Key Relationships

- **Films** are connected to **actors** through the `film_actor` junction table
- **Films** are categorized using the `film_category` junction table
- **Inventory** tracks which films are available at which stores
- **Rentals** connect customers, inventory items, and staff
- **Payments** are linked to specific rental transactions
- **Geographic hierarchy**: Country → City → Address

## Loading the Database

### Prerequisites

Before loading the sample database, ensure you have:
- PostgreSQL server installed and running
- PostgreSQL client tools (`psql` and `pg_restore`)
- The `dvdrental.tar` file (included in this repository)

### Step-by-Step Loading Process

#### 1. Connect to PostgreSQL Server

```bash
psql -U postgres
```

You'll be prompted for the postgres user password.

#### 2. Create the Database

```sql
CREATE DATABASE dvdrental;
```

Expected output:
```
CREATE DATABASE
```

#### 3. Verify Database Creation

```sql
\l
```

You should see `dvdrental` in the list of databases.

#### 4. Exit psql

```sql
exit
```

#### 5. Restore the Database from TAR File

```bash
pg_restore -U postgres -d dvdrental dvdrental.tar
```

**Note**: Replace the path with the actual location of your `dvdrental.tar` file if it's in a different directory.

You'll be prompted for the postgres user password again.

#### 6. Verify the Database Load

Connect to the database:
```bash
psql -U postgres
```

Switch to the dvdrental database:
```sql
\c dvdrental
```

List all tables:
```sql
\dt
```

Expected output:
```
 Schema |     Name      | Type  |  Owner
--------+---------------+-------+----------
 public | actor         | table | postgres
 public | address       | table | postgres
 public | category      | table | postgres
 public | city          | table | postgres
 public | country       | table | postgres
 public | customer      | table | postgres
 public | film          | table | postgres
 public | film_actor    | table | postgres
 public | film_category | table | postgres
 public | inventory     | table | postgres
 public | language      | table | postgres
 public | payment       | table | postgres
 public | rental        | table | postgres
 public | staff         | table | postgres
 public | store         | table | postgres
(15 rows)
```

### Alternative Loading Methods

#### Using psql with custom format
If you prefer using psql directly:
```bash
psql -U postgres -d dvdrental -f dvdrental.sql
```

#### Using pgAdmin
1. Open pgAdmin
2. Create a new database named `dvdrental`
3. Right-click on the database → Restore
4. Select the `dvdrental.tar` file
5. Click Restore

## Sample Queries

Once loaded, you can try these sample queries:

### Basic Queries

```sql
-- Get all films with their ratings
SELECT title, rating, length FROM film LIMIT 10;

-- Count films by rating
SELECT rating, COUNT(*) FROM film GROUP BY rating;

-- Find all films with a specific actor
SELECT f.title, a.first_name, a.last_name
FROM film f
JOIN film_actor fa ON f.film_id = fa.film_id
JOIN actor a ON fa.actor_id = a.actor_id
WHERE a.first_name = 'PENELOPE' AND a.last_name = 'GUINESS';
```

### Advanced Queries

```sql
-- Top 10 customers by total payment amount
SELECT c.first_name, c.last_name, SUM(p.amount) as total_spent
FROM customer c
JOIN payment p ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC
LIMIT 10;

-- Films never rented
SELECT f.title
FROM film f
LEFT JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
WHERE r.rental_id IS NULL;
```

## Files in this Repository

- `dvdrental.tar` - PostgreSQL database backup file
- `printable-postgresql-sample-database-diagram.pdf` - Complete ER diagram
- `README.md` - This documentation file

## Troubleshooting

### Common Issues

1. **Connection refused**: Ensure PostgreSQL service is running
2. **Permission denied**: Check user privileges and database ownership
3. **File not found**: Verify the path to `dvdrental.tar` is correct
4. **Encoding issues**: Ensure your PostgreSQL installation supports UTF-8

### Getting Help

If you encounter issues:
1. Check PostgreSQL logs for detailed error messages
2. Verify PostgreSQL version compatibility
3. Ensure sufficient disk space for the database
4. Confirm user has necessary privileges

## License

This sample database is provided for educational purposes. Please refer to PostgreSQL's licensing terms for usage guidelines.

## Contributing

Feel free to submit issues or improvements to this documentation.
