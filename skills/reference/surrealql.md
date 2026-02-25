# SurrealQL

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Applies to: SurrealDB v3.0.0
- Changelog:
  - v1.0: Initial skill

## Overview
SurrealQL is the query language for SurrealDB. It handles all data operations: CRUD,
graph traversal, vector search, full-text search, real-time subscriptions, and more.
It looks like SQL but has unique features for multi-model data (documents, graphs, vectors).

Every project in Scott's stack uses SurrealQL for all database operations, whether
through the embedded Rust SDK (desktop/Tauri) or the JavaScript SDK (web).

## Key Concepts

- **Tables:** Like spreadsheet sheets. Each table holds records of the same type.
- **Records:** Individual items in a table. Each has a unique ID like `user:abc123`.
- **Record IDs:** Format is `table:id`. SurrealDB auto-generates IDs unless you specify one.
- **Fields:** Properties on a record (like columns in a spreadsheet).
- **Graph edges:** Relationships between records. A record can "point to" another record.
- **Namespaces and databases:** Organization layers. Think folders → subfolders → tables.

## CRUD Operations

### CREATE (Insert a new record)
```sql
-- Auto-generated ID
CREATE user SET name = 'Scott', email = 'scott@example.com';
-- Returns: user:randomid123

-- Specific ID
CREATE user:scott SET name = 'Scott', email = 'scott@example.com';

-- With variable binding (from SDK)
CREATE user SET
    name = $name,
    email = $email,
    created_at = time::now();
```

### SELECT (Read records)
```sql
-- All records from a table
SELECT * FROM user;

-- Specific record by ID
SELECT * FROM user:scott;

-- With filtering
SELECT * FROM user WHERE age > 18;

-- Specific fields only
SELECT name, email FROM user;

-- With aliases (rename fields in output)
SELECT name, email, time::format(created_at, '%Y-%m-%d') AS joined FROM user;

-- Count records
SELECT count() FROM user GROUP ALL;
```

### UPDATE (Modify existing records)
```sql
-- Update a specific record (replaces all fields)
UPDATE user:scott SET name = 'Scott O', email = 'scott@new.com';

-- Merge (only update specified fields, keep the rest)
UPDATE user:scott MERGE { name: 'Scott O' };

-- Update multiple records with a condition
UPDATE user SET verified = true WHERE email CONTAINS '@advosy.com';

-- Increment a field
UPDATE user:scott SET login_count += 1;
```

### DELETE (Remove records)
```sql
-- Specific record
DELETE user:scott;

-- With condition
DELETE user WHERE last_login < time::now() - 1y;

-- Delete all records in a table (careful!)
DELETE user;
```

### UPSERT (Create or update)
```sql
-- Creates if doesn't exist, updates if it does
UPSERT user:scott SET
    name = 'Scott',
    email = 'scott@example.com',
    updated_at = time::now();
```

## Graph Traversal

SurrealDB supports graph relationships between records. This is one of its most
powerful features — it lets you model relationships naturally.

### Creating graph edges (RELATE)
```sql
-- Scott wrote a post
RELATE user:scott->wrote->post:first_post
    SET created_at = time::now();

-- Scott follows Jane
RELATE user:scott->follows->user:jane;

-- With data on the edge
RELATE user:scott->rated->movie:inception
    SET score = 9, review = 'Amazing film';
```

### Traversing graphs
```sql
-- Outgoing: What did Scott write? (->)
SELECT ->wrote->post FROM user:scott;

-- Incoming: Who wrote this post? (<-)
SELECT <-wrote<-user FROM post:first_post;

-- Bidirectional: All connections (<->)
SELECT <->follows<->user FROM user:scott;

-- Multi-hop: Friends of friends
SELECT ->follows->user->follows->user FROM user:scott;

-- With field access on the traversal
SELECT ->follows->user.name FROM user:scott;

-- Filtering on graph edges
SELECT ->rated->movie WHERE ->rated.score > 8 FROM user:scott;
```

### Direction reference
| Symbol | Direction | Meaning |
|--------|-----------|---------|
| `->` | Outgoing | "From this record to..." |
| `<-` | Incoming | "...points to this record" |
| `<->` | Both | "Connected in either direction" |

## Record References (v3)

New in v3: REFERENCE fields create automatic bidirectional links.

```sql
-- Define a reference field
DEFINE FIELD author ON post TYPE record<user> REFERENCE;

-- Query incoming references with <~
SELECT *, <~post AS posts FROM user;

-- Computed field using references
DEFINE FIELD posts ON user COMPUTED <~post;

-- Narrow by source field
DEFINE FIELD authored_posts ON user COMPUTED <~(post FIELD author);
```

## COMPUTED Fields (v3)

Replaced futures from v2. Calculated on every read, not stored on disk.

```sql
-- Simple computation
DEFINE FIELD full_name ON user COMPUTED string::concat(first_name, ' ', last_name);

-- Age calculation
DEFINE FIELD age ON user COMPUTED time::year(time::now()) - time::year(born);

-- Boolean computation
DEFINE FIELD is_adult ON user COMPUTED age >= 18;

-- Count related records
DEFINE FIELD post_count ON user COMPUTED count(->wrote->post);

-- Current timestamp (recalculates each read)
DEFINE FIELD accessed_at ON user COMPUTED time::now();
```

**Restrictions:**
- Top-level fields only (no nested fields)
- Cannot combine with VALUE, DEFAULT, or other modifiers
- Cannot create indexes on computed fields
- Cannot define on the `id` field

## Indexes and Query Planning

```sql
-- Basic index (speeds up WHERE queries on this field)
DEFINE INDEX idx_email ON user FIELDS email UNIQUE;

-- Composite index (for queries filtering on both fields)
DEFINE INDEX idx_status_date ON claim FIELDS status, created_at;

-- Full-text search index
DEFINE INDEX idx_search ON post FIELDS title, body SEARCH ANALYZER snowball BM25;

-- HNSW vector index (for AI/similarity search)
DEFINE INDEX idx_embedding ON document FIELDS embedding HNSW DIMENSION 1536;
```

### Full-text search
```sql
-- Search using the index
SELECT * FROM post WHERE title @@ 'SurrealDB tutorial';

-- With BM25 scoring
SELECT *, search::score(1) AS relevance
FROM post
WHERE title @1@ 'SurrealDB'
ORDER BY relevance DESC;
```

### Vector search
```sql
-- Find similar documents by vector
SELECT * FROM document
WHERE embedding <|10,COSINE|> $query_vector;
```

## Common Query Patterns

### Pagination
```sql
-- Offset-based pagination
SELECT * FROM post
ORDER BY created_at DESC
LIMIT 20
START 0;  -- Change START for next page (20, 40, 60...)

-- With total count (for showing "Page 1 of 5")
LET $total = (SELECT count() FROM post GROUP ALL)[0].count;
LET $items = SELECT * FROM post ORDER BY created_at DESC LIMIT $limit START $offset;
RETURN { total: $total, items: $items };
```

### Search with filters
```sql
SELECT * FROM claim
WHERE status = $status
    AND (name CONTAINS $search OR address CONTAINS $search)
ORDER BY created_at DESC
LIMIT 50;
```

### Aggregation
```sql
-- Count by status
SELECT status, count() AS total
FROM claim
GROUP BY status;

-- Sum and average
SELECT
    count() AS total_claims,
    math::sum(amount) AS total_amount,
    math::mean(amount) AS avg_amount
FROM claim
GROUP ALL;
```

### Subqueries
```sql
-- Get users with their post count
SELECT *, (SELECT count() FROM post WHERE author = $parent.id GROUP ALL)[0].count AS post_count
FROM user;

-- Note: In v3, use <string>$parent.id when referencing parent ID in subqueries
```

### Transactions
```sql
BEGIN TRANSACTION;
    LET $from = (UPDATE account:scott SET balance -= $amount);
    LET $to = (UPDATE account:jane SET balance += $amount);

    IF $from.balance < 0 {
        CANCEL TRANSACTION;
    };
COMMIT TRANSACTION;
```

## Schema Definition

### Define a table with schema
```sql
-- Schemaless (any fields allowed — good for prototyping)
DEFINE TABLE claim SCHEMALESS;

-- Schemafull (only defined fields allowed — good for production)
DEFINE TABLE claim SCHEMAFULL;

-- With permissions
DEFINE TABLE claim SCHEMALESS
    PERMISSIONS
        FOR select WHERE user = $auth.id
        FOR create, update WHERE user = $auth.id
        FOR delete NONE;
```

### Define fields
```sql
-- Required string
DEFINE FIELD name ON claim TYPE string;

-- Optional string
DEFINE FIELD notes ON claim TYPE option<string>;

-- String with default
DEFINE FIELD status ON claim TYPE string DEFAULT 'new';

-- Validated field
DEFINE FIELD email ON user TYPE string
    ASSERT string::is::email($value);

-- Enum-like validation
DEFINE FIELD status ON claim TYPE string
    ASSERT $value IN ['new', 'in_progress', 'complete'];

-- Record link
DEFINE FIELD author ON post TYPE record<user>;

-- Array of record links
DEFINE FIELD tags ON post TYPE array<record<tag>>;

-- Nested object
DEFINE FIELD address ON user TYPE object;
DEFINE FIELD address.street ON user TYPE string;
DEFINE FIELD address.city ON user TYPE string;
DEFINE FIELD address.zip ON user TYPE string;

-- Auto-timestamps
DEFINE FIELD created_at ON claim TYPE datetime DEFAULT time::now();
DEFINE FIELD updated_at ON claim TYPE datetime VALUE time::now();
```

## LIVE SELECT (Real-Time)

```sql
-- Subscribe to all changes on a table
LIVE SELECT * FROM claim;

-- With filter
LIVE SELECT * FROM claim WHERE status = 'new';

-- DIFF mode (returns patches instead of full records)
LIVE SELECT DIFF FROM claim;

-- With FETCH (include related records)
LIVE SELECT * FROM claim FETCH assigned_to;
```

**Requirements:** WebSocket connection only. Single-node deployment only.

## Common Gotchas

1. **v3: Non-existing tables throw errors** — In v2, querying a table that doesn't exist
   returned `[]`. In v3, it throws an error. Always DEFINE TABLE before querying.

2. **v3: LET is required for parameters** — `$val = 10` no longer works. Use `LET $val = 10;`

3. **v3: type::thing → type::record** — The function was renamed. Old name won't work.

4. **v3: Optional chaining** — `$val?` is now `$val.?`

5. **RecordId objects from SDKs** — The JavaScript SDK returns RecordId objects, not strings.
   Always use `String()` before comparing or doing string operations.

6. **JS null ≠ SurrealQL NONE** — When sending data from JavaScript, omit optional fields
   entirely instead of passing `null`.

7. **SCHEMAFULL tables reject extra fields** — If you define a table as SCHEMAFULL,
   only DEFINE'd fields are accepted. Unexpected fields cause errors.

8. **Subquery parent ID** — Use `<string>$parent.id` when referencing a parent record's ID
   in a subquery to avoid RecordId type issues.

## Resources
- Migration guide: https://surrealdb.com/docs/surrealdb/installation/upgrading/migrating-data-to-3x
- SurrealQL reference: https://surrealdb.com/docs/surrealql
- Function reference: https://surrealdb.com/docs/surrealql/functions
