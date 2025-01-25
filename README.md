# Projector HSA Home work #9: Transactions. Isolations. Locks.

## To run the project:

```bash
$ docker-compose up
```

# Results

## Percona:

| Isolation Level  | Lost Update | Dirty Read | Non-repeatable Read | Phantom Read |
| ---------------- | ----------- | ---------- | ------------------- | ------------ |
| Read Uncommitted | ✅          | ✅         | ⛔️                 | ⛔️          |
| Read Committed   | ✅          | ✅         | ⛔️                 | ⛔️          |
| Repeatable Read  | ⛔️         | ✅         | ⛔️                 | ⛔️          |
| Serializable     | ⛔️         | ✅         | ⛔️                 | ⛔️          |

## PostgreSQL:

| Isolation Level                  | Lost Update | Dirty Read | Non-repeatable Read | Phantom Read |
| -------------------------------- | ----------- | ---------- | ------------------- | ------------ |
| Read Uncommitted (Not supported) | –           | –          | –                   | –            |
| Read Committed                   | ✅          | ⛔️        | ✅                  | ✅           |
| Repeatable Read                  | ⛔️         | ⛔️        | ⛔️                 | ⛔️          |
| Serializable                     | ⛔️         | ⛔️        | ⛔️                 | ⛔️          |

# Details:

# Percona

### DB Schema:

```sql
CREATE TABLE users
(
    id   bigint auto_increment primary key,
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    dateOfBirth DATE NOT NULL
);
```

### Insert data:

```sql
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES
('John', 'Doe', '1990-01-01'),
('Jane', 'Smith', '1985-05-20');
```

## Read Uncommitted

Run:

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

### Try to reproduce **Lost Update**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is reproduced. First transaction will override second transaction even if second one is already committed.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
START TRANSACTION;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is reproduced. Second transaction will read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction will read the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.

---

## Read Committed

Run:

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### Try to reproduce **Lost Update**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is reproduced. First transaction will override second transaction even if second one is already committed.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
START TRANSACTION;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is reproduced. Second transaction will read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction will read the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.

---

## Repeatable Read

Run:

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```

### Try to reproduce **Lost Update**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is not reproduced. Second transaction will wait for the first transaction to commit.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
START TRANSACTION;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is reproduced. Second transaction will read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction will read the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.

---

## Serializable

Run:

```sql
SET GLOBAL TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Try to reproduce **Lost Update**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is not reproduced. Second transaction will wait for the first transaction to commit.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
START TRANSACTION;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is reproduced. Second transaction will read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE id = 1;

SELECT SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction will read the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
START TRANSACTION;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
START TRANSACTION;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.

---

# Postgres

### DB Schema:

```sql
CREATE TABLE users
(
  id SERIAL PRIMARY KEY,
  firstName VARCHAR(50) NOT NULL,
  lastName VARCHAR(50) NOT NULL,
  dateOfBirth DATE NOT NULL
);
```

## Read Uncommitted

_Is not supported_

## Read Committed

### Try to reproduce **Lost Update**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is reproduced. First transaction will override second transaction even if second one is already committed.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT PG_SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is not reproduced. Second transaction not able to read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is reproduced. First transaction reads different values.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is reproduced. First transaction will read different number of rows.

---

## Repeatable Read

### Try to reproduce **Lost Update**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is not reproduced. First transaction can't override second transaction.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT PG_SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is not reproduced. Second transaction not able to read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction reads the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.

---

## Serializable

### Try to reproduce **Lost Update**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

UPDATE users SET firstName = 'Bob' WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE id = 1;
UPDATE users SET firstName = 'Alice' WHERE id = 1;
COMMIT;
```

Result:

Lost updated is not reproduced. First transaction can't override second transaction.

---

### Try to reproduce **Dirty Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
UPDATE users SET firstName = 'Temporary' WHERE id = 1;

SELECT PG_SLEEP(10);

ROLLBACK;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE id = 1;
COMMIT;
```

Result:

Dirty read is not reproduced. Second transaction not able to read uncommitted data.

---

### Try to reproduce **Non-repeatable Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE id = 1;

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE id = 1;
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
UPDATE users SET firstName = 'Changed' WHERE id = 1;
COMMIT;
```

Result:

Non repeatable read is not reproduced. First transaction reads the same value twice.

---

### Try to reproduce **Phantom Read**

```sql
-- First transaction
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM users WHERE dateOfBirth > '1980-01-01';

SELECT PG_SLEEP(10);

SELECT * FROM users WHERE dateOfBirth > '1980-01-01';
COMMIT;

-- Second transaction. Run during the sleep of the first transaction.
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
INSERT INTO users (firstName, lastName, dateOfBirth) VALUES ('New', 'User', '1995-01-01');
COMMIT;
```

Result:

Phantom read is not reproduced. First transaction will read the same number of rows.
