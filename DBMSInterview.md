# UBS DBMS Interview — 40 Complete Answers

**Prep target: July 17 technical interview**
Same format as your OOP and Concurrency sets — talk-track length, banking-anchored, with SQL code where you'll likely need to write it live.

---

## Category 1: Core Relational Foundations & Storage Models

### 1. DBMS vs RDBMS — fundamental architectural difference
A **DBMS** is general software for storing and retrieving data, but doesn't enforce a fixed structure — data can be stored as flat files, hierarchies, or navigational structures, with no mandatory relationships between data sets (e.g., a simple file-based system). An **RDBMS** specifically organizes data into **tables (relations)** with rows and columns, enforces **schema constraints** (primary keys, foreign keys, data types), and guarantees **ACID-compliant transactions**. The defining architectural difference: RDBMS mandates relationships between tables via keys and enforces integrity constraints at the engine level, whereas a plain DBMS leaves that structure and consistency entirely up to the application. MySQL, PostgreSQL, and Oracle are RDBMS; a basic file system or early DBMS products would just be "DBMS."

### 2. SQL vs NoSQL — when to choose a NoSQL document store in banking
- **SQL (Relational)**: fixed schema, strong ACID guarantees, relationships enforced via foreign keys, optimized for complex joins and structured, consistent data — the default choice for core ledger/transaction data where correctness is non-negotiable.
- **NoSQL (Non-relational)**: flexible/schema-less structure, horizontally scalable, often trades strict consistency for availability and partition tolerance (per the CAP theorem).
In banking, you'd choose a **NoSQL document store** for scenarios like: storing **audit logs or event streams** with varying, evolving shapes; **customer session/preference data** where schema flexibility matters more than joins; **KYC document metadata** with nested, semi-structured fields; or **high-velocity market data caching** where you need horizontal write scale more than strict relational integrity. Core account balances and transaction ledgers, however, stay in an RDBMS — you cannot compromise ACID guarantees on money movement.

### 3. Types of database keys
- **Super Key**: any column (or combination of columns) that can uniquely identify a row — may contain extra, unnecessary attributes.
- **Candidate Key**: a *minimal* super key — no redundant columns; a table can have multiple candidate keys.
- **Primary Key**: the candidate key chosen to be the table's official unique identifier — cannot contain `NULL`, and there's exactly one per table.
- **Alternate Key**: any candidate key **not** chosen as the primary key (e.g., if `accountNumber` is the primary key, `email` might be an alternate key on a `Customer` table).
- **Foreign Key**: a column in one table that references the primary key of another table, enforcing a relationship between the two (e.g., `Transaction.accountId` referencing `Account.id`).

### 4. Why one Primary Key but multiple Unique Keys? Can a Primary Key be `NULL`?
A table can have only **one Primary Key** because the primary key is the table's single, canonical way to uniquely identify a row — used by the engine for the clustered index and as the default target for foreign key relationships; having more than one would be structurally ambiguous. **Unique Keys**, however, can be applied to multiple columns independently — each just enforces "no duplicate values in this column," without claiming to be *the* row identifier (e.g., both `accountNumber` and `email` could be separately marked `UNIQUE` on a `Customer` table). **A Primary Key column cannot contain `NULL`** — by definition, a primary key must uniquely and completely identify every row, and `NULL` represents "unknown/absent," which is incompatible with guaranteed uniqueness and identification. (Unique key columns, by contrast, typically *can* hold one `NULL` in most RDBMS implementations, since `NULL` isn't considered equal to another `NULL` for uniqueness checks.)

### 5. Referential integrity and delete behavior (`CASCADE`, `SET NULL`, `RESTRICT`)
Referential integrity ensures a foreign key value in a child table always points to an existing, valid row in the parent table — you can't have a `Transaction` referencing a non-existent `Account`. When deleting a parent record referenced by a child, the engine's behavior depends on the foreign key's defined action:
- **`CASCADE`**: automatically deletes all child rows referencing the deleted parent (e.g., deleting a `Customer` also deletes all their `Account` rows) — powerful but dangerous if used carelessly on financial data.
- **`SET NULL`**: sets the child's foreign key column to `NULL` instead of deleting the child row (requires the FK column to be nullable) — e.g., deleting an `Employee` who approved a `Transaction` sets `approvedBy` to `NULL` rather than deleting the transaction record.
- **`RESTRICT`** (or `NO ACTION`): **blocks** the delete entirely if any child rows still reference the parent — the safest default for financial systems, since it forces explicit handling rather than silently cascading destructive deletes through a ledger.

### 6. View vs Materialized View
A **View** is a **virtual table** defined by a stored SQL query — it has no independent storage; every time you query it, the underlying query re-executes against the live base tables, so it always reflects real-time data. A **Materialized View** actually **stores the query's result set physically on disk** at creation time — querying it is fast (just reading pre-computed data) but the data becomes **stale** until the materialized view is manually or periodically **refreshed**. Trade-off: standard Views cost nothing extra in storage but pay the query cost every time; Materialized Views cost storage and go stale, but offer fast reads — ideal for expensive aggregate reports (e.g., daily transaction summaries) that don't need up-to-the-second accuracy.

### 7. DDL vs DML vs DCL vs TCL
- **DDL (Data Definition Language)**: defines/modifies schema structure. E.g., `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`. Auto-commits in most RDBMS.
- **DML (Data Manipulation Language)**: manipulates actual data. E.g., `INSERT`, `UPDATE`, `DELETE`, `SELECT` (sometimes classified separately as DQL). Participates in transactions.
- **DCL (Data Control Language)**: manages permissions/access. E.g., `GRANT`, `REVOKE`.
- **TCL (Transaction Control Language)**: manages transaction boundaries. E.g., `COMMIT`, `ROLLBACK`, `SAVEPOINT`.
In a banking context: DDL sets up the `Account`/`Transaction` table structure once; DML processes the actual deposits/withdrawals; DCL restricts which service accounts can write to the ledger table; TCL ensures a multi-step transfer either fully commits or fully rolls back.

### 8. Implicit vs Explicit Cursor
A **Cursor** is a pointer used to iterate through the rows of a result set one at a time, typically inside stored procedures for row-by-row processing.
- **Implicit Cursor**: automatically created and managed by the database engine for every DML statement (`INSERT`/`UPDATE`/`DELETE`/single-row `SELECT`) — you never declare it explicitly, and it's used internally (e.g., you can check `SQL%ROWCOUNT` in PL/SQL after a statement runs).
- **Explicit Cursor**: manually declared by the developer for a multi-row `SELECT` query when you need fine-grained row-by-row control — you `DECLARE`, `OPEN`, `FETCH` (in a loop), and `CLOSE` it yourself. Used for scenarios like processing a batch of pending transactions individually, applying custom logic per row that can't be expressed in a single set-based SQL statement. Cursors are generally a last resort in performance-sensitive systems — set-based operations are almost always faster than row-by-row cursor processing.

### 9. Database Schema vs Database Instance
The **Schema** is the **structural blueprint** of the database — table definitions, column types, constraints, relationships — defined once and rarely changed. The **Instance** is the **actual data** present in the database at a specific point in time — the live snapshot of rows and values, which changes constantly as transactions occur. Analogy: the schema is like a class definition in Java (`Account` with fields `id`, `balance`), while an instance is like the actual set of objects (all current `Account` rows) existing right now in memory/storage.

### 10. Triggers — `AFTER INSERT` example for banking audit logs
A **Trigger** is a stored procedure that automatically executes in response to a specified DML event (`INSERT`, `UPDATE`, `DELETE`) on a table, without being explicitly called by application code. An `AFTER INSERT` trigger on a `Transaction` table is a classic banking use case: every time a new row is inserted into `Transaction`, the trigger automatically fires and inserts a corresponding row into an immutable `TransactionAuditLog` table, capturing who performed it, when, and the transaction details — providing a tamper-resistant audit trail that exists independent of application logic (so even a buggy or malicious application-layer bypass can't skip the audit log, since it's enforced at the database engine level).

```sql
CREATE TRIGGER trg_audit_transaction
AFTER INSERT ON Transaction
FOR EACH ROW
BEGIN
    INSERT INTO TransactionAuditLog (transaction_id, amount, created_at, action)
    VALUES (NEW.id, NEW.amount, NOW(), 'INSERT');
END;
```

---

## Category 2: ACID Properties, Transactions & Concurrency

### 11. ACID properties — bank account transfer example
Consider transferring ₹1000 from Account A to Account B (debit A, credit B — two operations that must succeed or fail together):
- **Atomicity**: both the debit and credit happen, or **neither** does. If the system crashes after debiting A but before crediting B, the entire transaction is rolled back — money can never vanish mid-transfer.
- **Consistency**: the database moves from one valid state to another — e.g., a rule like "balance can never go negative" is never violated at the end of the transaction, and the total money across A + B remains unchanged before and after.
- **Isolation**: if another transaction concurrently checks Account A's balance while the transfer is in progress, it doesn't see a "half-done" intermediate state (e.g., debited but not yet credited) — isolation level determines exactly how strictly this is enforced (see Q13).
- **Durability**: once the transfer transaction **commits**, the result is permanently saved — even if the server crashes one millisecond later, the completed transfer survives and is recoverable from disk (via the transaction log).

### 12. How does the engine guarantee Atomicity across a crash? (Write-Ahead Logging)
The database uses **Write-Ahead Logging (WAL)**: before any actual data page on disk is modified, the engine first writes a record of the intended change to a sequential, append-only **transaction log** (on stable storage) — recording enough information to either **redo** the change (if the transaction had committed) or **undo** it (if it hadn't). If the system crashes mid-transaction, on restart the engine replays the WAL: any transaction that had **not** reached a `COMMIT` record in the log is **rolled back** using the undo information, restoring affected data to its pre-transaction state; any transaction that *had* committed but whose data changes weren't yet flushed to disk is **redone** using the log. This guarantees that a transaction is either fully applied or fully undone, regardless of exactly when a crash occurs — the log is always written first, so it's the authoritative record of "what was actually promised to be durable."

### 13. Four transaction isolation levels
- **Read Uncommitted**: no isolation — a transaction can read another transaction's **uncommitted** changes (dirty reads possible). Fastest, least safe.
- **Read Committed**: a transaction only ever sees data that has been **committed** by other transactions — but re-reading the same row twice within the same transaction can return different values if another transaction commits a change in between (non-repeatable read possible). Default in PostgreSQL, Oracle, SQL Server.
- **Repeatable Read**: guarantees that if you read a row once in a transaction, re-reading it later in the **same transaction** returns the **same value**, even if another transaction commits a change to it — achieved by locking/snapshotting rows already read. Phantom reads (new rows matching a `WHERE` clause appearing on re-query) are still possible in the strict standard definition (though MySQL's InnoDB implementation actually prevents phantoms too, via gap locking — a common interview nuance). Default in MySQL InnoDB.
- **Serializable**: the strictest level — transactions execute as if run one at a time, sequentially, with zero concurrency-induced anomalies (no dirty reads, non-repeatable reads, or phantom reads). Achieved via full range locking or serializable snapshot isolation; safest but slowest, since concurrency is maximally restricted.

### 14. Dirty Read — what it is, and why unacceptable in ledgers
A **Dirty Read** occurs when a transaction reads data that another transaction has **modified but not yet committed** — if that other transaction later **rolls back**, the first transaction acted on data that never actually existed in a valid, permanent state. It's possible only at the **Read Uncommitted** isolation level. In a financial ledger, this is catastrophic: imagine Transaction 1 tentatively debits ₹50,000 from an account (not yet committed) and Transaction 2 reads that in-flight balance to approve a large withdrawal based on "sufficient funds" — if Transaction 1 then rolls back (e.g., due to a validation failure), Transaction 2's decision was based on a balance state that never truly existed, potentially approving an overdraft or double-spend. This is precisely why banking systems universally require at least **Read Committed**, and typically **Repeatable Read or Serializable**, for core ledger operations.

### 15. Non-Repeatable Read vs Phantom Read — which isolation level prevents Phantoms?
- **Non-Repeatable Read**: within the same transaction, you read a specific row twice, and get **different values** the second time — because another transaction **updated** (or deleted) that exact row and committed in between your two reads.
- **Phantom Read**: within the same transaction, you re-run the **same range query** (`WHERE` clause) twice, and the **set of rows returned changes** — because another transaction **inserted or deleted rows** matching that condition and committed in between. The difference: non-repeatable read is about an existing row's *value* changing; phantom read is about the *set of rows* matching a condition changing.
**Serializable** is the isolation level guaranteed by the SQL standard to prevent Phantom Reads (via full range/predicate locking). Note: as mentioned in Q13, some engines like MySQL InnoDB prevent phantoms even at Repeatable Read via gap locks — a detail worth mentioning if asked, since it shows you understand standard-vs-implementation nuance.

### 16. MVCC (Multi-Version Concurrency Control) — reads without blocking writes
MVCC allows readers and writers to operate **without blocking each other** by maintaining **multiple versions of a row** simultaneously. Instead of a writer taking an exclusive lock that blocks all readers, when a row is updated, the engine creates a **new version** of that row (with a new transaction/timestamp marker) while keeping the **old version** intact for any transactions that had already started and expect to see the "old" consistent snapshot. Each transaction, based on its start time/snapshot, reads the version of each row that was **committed as of when its own transaction/statement began** — so a long-running read transaction sees a consistent snapshot throughout, completely unaffected by concurrent writes happening after it started, and without ever needing to acquire a read lock that would block those writers. PostgreSQL implements this directly at the row level (each row has hidden `xmin`/`xmax` transaction-ID columns marking its visibility range); InnoDB achieves similar behavior using **undo logs** to reconstruct older row versions on demand.

### 17. Optimistic vs Pessimistic Locking — which for concurrent withdrawal on the same account?
- **Pessimistic Locking**: assumes conflicts are likely, so it locks the row **upfront** (`SELECT ... FOR UPDATE`) before reading/modifying it — any other transaction attempting to touch that row must wait until the lock is released. Safe but reduces concurrency, since competing transactions queue up.
- **Optimistic Locking**: assumes conflicts are rare, so it **doesn't lock upfront** — it reads the row (often with a `version` column), performs the calculation, and on write, checks whether the `version` still matches what was read (`UPDATE ... WHERE id = ? AND version = ?`); if another transaction modified the row in between, the version won't match, the update affects zero rows, and the application detects this and retries. Higher concurrency, but requires explicit conflict-handling/retry logic.
**For two users simultaneously withdrawing from the same account**, I'd use **Pessimistic Locking** — withdrawals are a high-conflict-risk, correctness-critical operation (an incorrect optimistic retry storm or a missed conflict could allow an overdraft), and the brief lock-wait cost is an acceptable trade-off for the guarantee that no two withdrawals can ever race against the same balance check. Optimistic locking is better suited to low-conflict scenarios like updating a user's profile/preferences, where retry-on-conflict is cheap and rare.

### 18. Database Deadlock — causes and resolution
A deadlock occurs when two (or more) transactions each hold a lock the other needs, and each is waiting for the other to release its lock — neither can proceed. Classic example: Transaction 1 locks Account A then tries to lock Account B; Transaction 2 has already locked Account B and tries to lock Account A — both wait forever. This satisfies the same four Coffman conditions covered in the concurrency set (mutual exclusion, hold-and-wait, no preemption, circular wait). The DBMS's **deadlock detector** periodically builds a **wait-for graph** (which transaction is waiting on which) and checks for **cycles** — if found, it picks a **victim** transaction (usually the one that's done the least work, or based on a configurable priority) and forcibly **rolls it back**, releasing its locks so the remaining transaction(s) can proceed. The rolled-back transaction's application code typically needs to **catch the deadlock error and retry** the operation.

### 19. Shared Lock vs Exclusive Lock — can multiple transactions share-lock the same row?
- **Shared Lock (Read Lock, `S`)**: allows a transaction to **read** a row; multiple transactions **can** hold a shared lock on the same row **simultaneously**, since concurrent reads don't conflict with each other.
- **Exclusive Lock (Write Lock, `X`)**: allows a transaction to **modify** a row; it is fully exclusive — no other transaction can hold *any* lock (shared or exclusive) on that row while an exclusive lock is held, and an exclusive lock cannot be granted while any shared locks are held.
**Yes**, multiple transactions can simultaneously hold a Shared Lock on the same row (e.g., many transactions all reading the same account balance concurrently is fine) — but the moment one transaction needs an Exclusive Lock to write to that row, it must wait for **all** existing shared locks to be released first, and no new shared locks can be granted while it waits (to prevent writer starvation).

### 20. Two-Phase Commit (2PC) — distributed transactions across shards
2PC coordinates a transaction that spans **multiple independent database instances/shards**, ensuring all of them either commit or all roll back together, even though each is a technically separate transactional system. It works in two phases, managed by a **coordinator**:
1. **Prepare phase**: the coordinator asks every participating database ("participant") to **prepare** to commit — each participant does all the work, writes it to its own log, and replies "yes, I can commit" (locking resources) or "no" (if it hit an error), **without actually committing yet**.
2. **Commit phase**: if **all** participants voted "yes," the coordinator sends a **commit** command to everyone, and each participant finalizes and releases locks. If **any** participant voted "no" (or timed out), the coordinator sends **rollback** to everyone instead.
This is required for distributed transactions (e.g., a transfer where the sender's account lives on Shard 1 and the receiver's account lives on Shard 2) because a single shard's local ACID guarantees **only cover that one shard** — without 2PC, you could end up with the debit committed on Shard 1 but the credit failing on Shard 2, silently destroying money. Trade-off: 2PC is a **blocking protocol** (if the coordinator crashes after prepare but before sending commit/rollback, participants can be left holding locks indefinitely), which is why many modern distributed systems favor eventual consistency + compensating transactions (Saga pattern) over 2PC for high-scale systems, using 2PC only where strict atomicity across shards is unavoidable.

---

## Category 3: Normalization, Indexing & Engine Optimization

### 21. Normalization — 1NF, 2NF, 3NF, BCNF
Normalization is the process of structuring tables to eliminate redundancy and anomalies, by progressively enforcing stricter rules about functional dependencies:
- **1NF**: every column holds **atomic (indivisible)** values — no comma-separated lists or repeating groups within a single cell (e.g., split a `phoneNumbers` column holding "9876543210,9123456789" into separate rows or a related table).
- **2NF**: must already be in 1NF, **and** every non-key attribute must depend on the **entire** primary key, not just part of it — relevant only for tables with a **composite** primary key (e.g., in an `OrderItem(orderId, productId, productName)` table, `productName` depends only on `productId`, not the full composite key, so it violates 2NF and should move to a separate `Product` table).
- **3NF**: must already be in 2NF, **and** no **transitive dependencies** — non-key attributes must depend only on the primary key, not on other non-key attributes (e.g., in `Employee(empId, deptId, deptLocation)`, `deptLocation` depends on `deptId`, not directly on `empId` — transitive — so it should move to a separate `Department` table).
- **BCNF**: a stricter version of 3NF — for **every** functional dependency `X → Y`, `X` must be a **super key**. This catches edge cases 3NF misses, typically involving overlapping candidate keys.

### 22. Insertion, Deletion, and Updation anomalies
These are the actual problems normalization solves, occurring in an unnormalized (denormalized) table where redundant data is mixed together:
- **Insertion Anomaly**: you can't insert a fact about one entity without also having data for another — e.g., in a single `StudentCourse(studentId, courseName, instructorName)` table, you can't add a new course that has no students enrolled yet, because there's no `studentId` to attach it to.
- **Deletion Anomaly**: deleting one fact accidentally destroys unrelated information — e.g., if the last student drops a course, deleting that row also erases the record of who the `instructorName` for that course was.
- **Updation (Update) Anomaly**: because the same fact is duplicated across many rows, updating it requires touching **every** duplicate row — e.g., if `instructorName` for a course is repeated on every student's row, correcting a typo in the instructor's name requires updating potentially hundreds of rows, and missing even one leaves the data **inconsistent**.

### 23. Denormalization — when it's appropriate
Denormalization intentionally **reintroduces redundancy** into a normalized schema, trading some write-time complexity/storage for significantly faster reads by avoiding expensive joins. It's architecturally appropriate in **read-heavy OLAP (analytical) pipelines** — e.g., a nightly ETL job that flattens normalized transactional tables into a wide, denormalized **reporting/data-warehouse table** (star schema fact tables with denormalized dimension attributes) specifically for fast dashboard/reporting queries that would otherwise require joining 8+ normalized tables on every read. It's generally **not** appropriate for the live, write-heavy OLTP ledger itself, where normalization's consistency guarantees matter far more than raw read speed — you denormalize downstream, in reporting/analytics layers, not in the source-of-truth transactional schema.

### 24. Internal structure of a database Index — why B+ Trees over BSTs/Hash Tables
A database index is a separate, ordered data structure that maps column values to the physical location of the corresponding rows, avoiding a full table scan. **B+ Trees** are the near-universal choice because:
- They stay **balanced** and **shallow** (typically 3–4 levels even for millions of rows), unlike a plain Binary Search Tree, which can degrade to a linked-list-like depth (O(n)) if data is inserted in sorted order without self-balancing.
- **All actual data/row-pointers live only in the leaf nodes**, which are additionally **linked together sequentially** — this makes **range queries** (`WHERE balance BETWEEN 1000 AND 5000`, or `ORDER BY`) extremely efficient: find the starting leaf, then just walk the linked leaves, rather than repeatedly traversing the tree.
- **Hash Tables** offer O(1) average lookup for exact-match queries (`WHERE id = 5`) but are **useless for range queries** — a hash of `1000` gives no information about where `1001` lives, so `BETWEEN`/`>`/`<`/`ORDER BY` queries can't leverage a hash index at all, forcing a full scan.
- B+ Trees are also optimized for **disk I/O**: each node is sized to match a disk page, minimizing the number of disk reads needed to traverse from root to leaf — critical since disk (or even far-cache) access is orders of magnitude slower than in-memory comparisons.

### 25. Clustered Index vs Non-Clustered Index — why only one clustered index per table
A **Clustered Index** determines the **physical storage order of the actual table rows** on disk — the table data itself is organized according to the index key (e.g., rows physically stored in order of `id`). Because rows can only be physically arranged in **one** order, a table can have **only one clustered index** (in most engines, it's the primary key by default). A **Non-Clustered Index** is a **separate structure** that stores the indexed column's values alongside a **pointer/reference** back to the actual row's location (in the clustered index, or a row ID in engines without clustering) — the table data itself isn't reordered, so you can have **many** non-clustered indexes on different columns. Lookups via a non-clustered index typically require an extra step (index → pointer → actual row), sometimes called a "bookmark lookup," unless it's a covering index (Q26).

### 26. Covering Index — avoiding the extra data lookup
A **Covering Index** is a non-clustered index that includes **all the columns a specific query needs** — not just the columns in the `WHERE`/`JOIN` clause, but also every column in the `SELECT` list. Because every needed value is already present directly in the index's own leaf nodes, the query planner can satisfy the entire query **by reading only the index**, without ever touching the actual full table row (no "bookmark lookup" back to the clustered index / heap). This is the difference between an **Index Scan** (fast — reads only the compact index) and needing an **Index Seek + Table/Key Lookup** (slower — extra round-trip to fetch remaining columns) or a full **Table Scan** (slowest — no usable index at all). E.g., an index on `(accountId, transactionDate, amount)` fully "covers" a query like `SELECT amount FROM Transaction WHERE accountId = ? AND transactionDate > ?`, letting the engine answer it purely from the index.

### 27. When does an index degrade performance? Write-time overhead
Every index must be **updated** every time the underlying data changes — so on every `INSERT`, `UPDATE` (to an indexed column), or `DELETE`, the engine must also insert/update/remove the corresponding entries in **every index** defined on that table, not just modify the base table row. A table with many indexes therefore pays a **write amplification cost**: a single logical `INSERT` might translate into 1 base-table write + N index writes. In heavy `INSERT`/`UPDATE`/`DELETE` workloads (e.g., a high-throughput transaction ledger processing thousands of writes/second), over-indexing can noticeably slow down write throughput and increase lock contention on the index structures themselves — which is why index design is a genuine trade-off: index the columns that are actually queried frequently (especially in `WHERE`/`JOIN`/`ORDER BY`), and avoid blindly indexing every column "just in case," particularly on write-heavy tables.

### 28. Troubleshooting a slow query — reading `EXPLAIN`/`EXPLAIN ANALYZE`
`EXPLAIN` shows the **query planner's chosen execution plan** — which indexes (if any) it intends to use, the join order and join algorithm (nested loop, hash join, merge join), and the **estimated** cost/row counts for each step — without actually running the query. `EXPLAIN ANALYZE` actually **executes** the query and shows the **real** elapsed time and actual row counts alongside the plan, letting you compare estimated vs. actual to spot where the planner's assumptions are wrong (a common cause of bad plans being chosen). Troubleshooting workflow: look for **Sequential/Table Scans** on large tables where you'd expect an index to be used (suggests a missing or unused index, or an unSARGable predicate like wrapping a column in a function); check whether **estimated vs actual row counts diverge wildly** (suggests stale table statistics — run `ANALYZE`/`UPDATE STATISTICS`); look for expensive **nested loop joins** on large row sets (often better served by a hash join, which the planner might avoid due to bad stats or missing indexes); and check total execution time contribution per node to find the actual bottleneck step, rather than guessing.

### 29. Partitioning vs Sharding; Horizontal vs Vertical Partitioning
- **Partitioning**: splitting a large table into smaller pieces **within the same database instance** — the engine still manages it as logically one table, but physically stores/queries subsets more efficiently (e.g., partition a `Transaction` table by month, so queries filtering on a date range only scan the relevant partition).
- **Sharding**: splitting data across **multiple separate database instances/servers** (often on different physical machines) — each shard is essentially an independent database holding a subset of the data (e.g., customers with IDs 1–1M on Shard 1, 1M–2M on Shard 2), requiring application-level or middleware-level routing logic to know which shard holds which data.
- **Horizontal Partitioning**: splits a table by **rows** — e.g., older transactions (by date range) go into one partition, recent ones into another; each partition has the same columns, just a different subset of rows.
- **Vertical Partitioning**: splits a table by **columns** — e.g., splitting a wide `Customer` table into a frequently-accessed `CustomerCore(id, name, accountStatus)` table and a rarely-accessed `CustomerExtended(id, fullAddressHistory, documentScans)` table, improving cache efficiency for the hot columns.

### 30. Connection Pooling — why per-request fresh connections are an anti-pattern
Establishing a physical database connection is **expensive** — it involves a TCP handshake, authentication, session/memory setup on the database server side, and potentially TLS negotiation — typically tens to hundreds of milliseconds of pure overhead, completely separate from the actual query execution time. **Connection Pooling** maintains a pool of **already-established, reusable connections**; when application code needs a connection, it **borrows** one from the pool, uses it, and **returns** it (instead of closing it) for the next request to reuse. Creating a brand-new physical connection for every single HTTP request is an anti-pattern because: it adds significant per-request latency purely from connection setup; it risks **exhausting the database's max-connections limit** under load (each physical connection consumes server-side memory/resources, and databases cap total concurrent connections — a spike in traffic creating unbounded fresh connections can crash the database); and it wastes the connection-teardown cost too, on every single request. Pooling amortizes that setup/teardown cost across many logical requests, which is critical for a banking backend under high transaction volume.

---

## Category 4: Advanced SQL Syntax & Query Execution

### 31. `DELETE` vs `TRUNCATE` vs `DROP`
- **`DELETE`**: a DML operation, removes rows (optionally filtered by `WHERE`) one at a time, logged row-by-row in the transaction log — **can be rolled back**, and triggers fire per row. Slowest of the three for bulk removal, but most flexible/safe.
- **`TRUNCATE`**: a DDL operation (in most engines), removes **all** rows instantly by deallocating the data pages rather than logging individual row deletions — much faster than `DELETE` for clearing an entire table, but **cannot use a `WHERE` clause**, and in many engines it's **auto-committed** and can't be rolled back (though some engines like PostgreSQL allow it within a transaction). The table **structure remains intact**.
- **`DROP`**: a DDL operation that **removes the entire table** — structure, data, indexes, constraints, everything — permanently freeing all associated storage. Cannot be rolled back in most systems. Fastest of the three since there's nothing left to preserve.
Ordered by "how much is destroyed and how fast": `DELETE` (slowest, most surgical, rollback-safe) < `TRUNCATE` (fast, clears data, keeps structure) < `DROP` (fastest, destroys everything).

### 32. SQL query execution order
Even though you *write* SQL in this order — `SELECT ... FROM ... JOIN ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT` — the engine actually **executes** it in a different logical order:
1. **`FROM`** — identify the base table(s).
2. **`JOIN`** — combine rows from multiple tables.
3. **`WHERE`** — filter individual rows *before* any grouping.
4. **`GROUP BY`** — group the filtered rows.
5. **`HAVING`** — filter the *groups* (post-aggregation).
6. **`SELECT`** — compute the actual output columns/expressions.
7. **`ORDER BY`** — sort the final result set.
8. **`LIMIT`** — restrict the number of returned rows.
This ordering explains several common SQL rules: you can't reference a `SELECT`-aliased column in `WHERE` (because `WHERE` runs before `SELECT` computes that alias) but you often *can* in `ORDER BY` (since it runs after `SELECT`); and it's exactly why aggregate functions belong in `HAVING`, not `WHERE` (see Q33).

### 33. Why aggregate functions can't go in `WHERE`; why `HAVING` exists
Per the execution order above, **`WHERE` runs before `GROUP BY`** — it filters individual raw rows, before any aggregation/grouping has even happened, so at that point there's no "group" for `COUNT()`/`SUM()`/`AVG()` to aggregate *over* yet — the aggregate simply doesn't have its input ready. **`HAVING` runs after `GROUP BY`**, specifically to filter the **already-computed groups** based on aggregate results — e.g., `HAVING COUNT(*) > 5` filters out groups with 5 or fewer rows, which is only meaningful once grouping has actually happened. Rule of thumb: `WHERE` filters rows before grouping; `HAVING` filters groups after grouping.

### 34. `INNER JOIN` vs `LEFT/RIGHT/FULL OUTER JOIN`; nullable columns
- **`INNER JOIN`**: returns only rows where the join condition **matches in both tables** — non-matching rows from either side are excluded entirely.
- **`LEFT OUTER JOIN`**: returns **all rows from the left table**, plus matching rows from the right table; if there's no match, right-table columns come back as `NULL`.
- **`RIGHT OUTER JOIN`**: mirror of `LEFT` — all rows from the right table, `NULL` for unmatched left-table columns.
- **`FULL OUTER JOIN`**: returns **all rows from both tables** — matched where possible, with `NULL` filled in on whichever side lacks a match.
**Joining on nullable columns**: SQL's three-valued logic means `NULL = NULL` evaluates to **`UNKNOWN`**, not `TRUE` — so rows where the join column is `NULL` on either side will **never match** in the join condition, even if both sides happen to be `NULL`. This is a common source of "missing rows" bugs — if you actually need `NULL`s to be treated as matching, you'd need an explicit `IS NULL` check or a `COALESCE`-based workaround in the join condition, since standard equality won't do it.

### 35. `UNION` vs `UNION ALL`
Both combine the result sets of two or more `SELECT` queries (with matching column counts/types) into a single result set. **`UNION`** additionally removes **duplicate rows** from the combined result — which requires the engine to perform an internal **sort or hash-based deduplication pass** over the entire combined set. **`UNION ALL`** simply **concatenates** all rows from every query as-is, keeping duplicates, with **no deduplication step**. `UNION ALL` is significantly faster because it skips that expensive sort/dedup pass entirely — if you know in advance the result sets can't overlap (or duplicates are acceptable/expected), always prefer `UNION ALL` for performance.

### 36. Correlated Subquery vs non-correlated — time complexity difference
A **non-correlated (standalone) subquery** is fully independent of the outer query — it can be executed **once**, and its result reused for every row of the outer query (e.g., `WHERE salary > (SELECT AVG(salary) FROM Employees)` — the average is computed once). A **correlated subquery** references a column from the **outer query** inside its own `WHERE` clause, meaning it **cannot be computed once in isolation** — conceptually, it must be **re-evaluated once per row** of the outer query (e.g., `WHERE salary > (SELECT AVG(salary) FROM Employees e2 WHERE e2.deptId = e1.deptId)` — a different average per department, recomputed per outer row). This makes a naive correlated subquery's complexity roughly **O(N × M)** (N outer rows × M inner-query cost each) versus a non-correlated subquery's **O(N + M)** (M once, then N cheap comparisons) — though modern query optimizers often **rewrite** correlated subqueries into equivalent joins internally to avoid the literal per-row re-execution, so real-world performance depends heavily on how smart the specific engine's planner is.

### 37. `COALESCE()` vs `NULLIF()`
- **`COALESCE(a, b, c, ...)`**: returns the **first non-`NULL`** value among its arguments, scanning left to right — commonly used to supply a default value when a column might be `NULL` (e.g., `COALESCE(middleName, '')` to avoid `NULL` breaking a string concatenation).
- **`NULLIF(a, b)`**: returns `NULL` if `a` equals `b`, otherwise returns `a` — the inverse use case, used to **convert a specific "sentinel" value into `NULL`** (e.g., `NULLIF(discountRate, 0)` turns a `0` discount into `NULL` to distinguish "explicitly zero" scenarios from downstream logic, or to avoid a divide-by-zero: `amount / NULLIF(quantity, 0)` returns `NULL` instead of erroring when `quantity` is 0).

### 38. Window Functions — `ROW_NUMBER()` vs `RANK()` vs `DENSE_RANK()`
A **Window Function** performs a calculation across a set of rows related to the current row (a "window," defined via `OVER (PARTITION BY ... ORDER BY ...)`) **without collapsing the rows** into groups the way `GROUP BY` does — every input row still appears in the output, just annotated with the window calculation's result.
```sql
SELECT employeeId, department, salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn,
       RANK()       OVER (PARTITION BY department ORDER BY salary DESC) AS rnk,
       DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS drnk
FROM Employees;
```
The structural difference when encountering **duplicate values** (e.g., two employees tied at the same salary):
- **`ROW_NUMBER()`**: assigns a **strictly unique, sequential** number to every row regardless of ties — tied rows get arbitrarily different numbers (e.g., 1, 2, 3, 4 — no repeats, no gaps).
- **`RANK()`**: gives **tied rows the same rank**, but then **skips** the next rank number(s) by the count of ties — e.g., two rows tied for 2nd get rank `2, 2`, and the next distinct row gets rank `4` (3 is skipped).
- **`DENSE_RANK()`**: also gives tied rows the same rank, but does **not** skip subsequent numbers — e.g., `2, 2`, then the next distinct row gets rank `3` (no gap).

### 39. CTE — 3rd highest salary without `LIMIT`
```sql
WITH RankedSalaries AS (
    SELECT employeeId, salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM Employees
)
SELECT employeeId, salary
FROM RankedSalaries
WHERE salary_rank = 3;
```
Using `DENSE_RANK()` here (rather than `ROW_NUMBER()`) is deliberate — if two employees are tied for the highest salary, you want the "3rd highest **distinct** salary," not just the row in the 3rd physical position, which `ROW_NUMBER()` would give incorrectly if ties exist at the top.

### 40. Identifying and deleting duplicate rows, keeping one copy
Using `ROW_NUMBER()` over a window partitioned by the columns that define a "duplicate," then deleting everything except the first occurrence in each partition:
```sql
WITH Ranked AS (
    SELECT ctid,  -- or a primary key column, e.g. id
           ROW_NUMBER() OVER (
               PARTITION BY accountId, transactionDate, amount
               ORDER BY id
           ) AS rn
    FROM Transaction
)
DELETE FROM Transaction
WHERE ctid IN (SELECT ctid FROM Ranked WHERE rn > 1);
```
Logic: within each group of rows that are duplicates of each other (same `accountId`, `transactionDate`, `amount`), `ROW_NUMBER()` labels them `1, 2, 3, ...` in a defined order (here, by `id`) — keep `rn = 1` (the "original"), and delete everything with `rn > 1`. A self-join achieves the same result differently: join the table to itself on the duplicate-defining columns where one side's primary key is greater than the other's, then delete the "greater id" side — but the window-function approach is generally cleaner and more efficient, especially when more than two duplicates of the same row exist.

---

## Your 4-Day DBMS Execution Focus (unchanged from your notes, reinforced here)

- **OA (MCQs/SQL)**: drill **Category 4** — especially **Q32 execution order** and **Q38 window functions** — plus **Category 1 Q3 (keys)** and **Q5 (referential integrity)**.
- **Technical interviews**: go deep on **Category 2** — **Q11 ACID**, **Q13 isolation levels**, **Q14–15 dirty/phantom reads**, and **Q16 MVCC**. Be ready to explain, unprompted, exactly how each of these prevents a race condition on a shared account balance — that's the through-line UBS interviewers are testing for across this entire category.
- Additionally worth a cold read-through: **Q17 (optimistic vs pessimistic locking)** and **Q20 (2PC)** — both come up naturally as follow-ups once you've nailed ACID/isolation, since they're the practical mechanisms banks actually use to enforce those guarantees at scale.
