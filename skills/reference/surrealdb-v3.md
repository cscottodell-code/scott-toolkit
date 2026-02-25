# SurrealDB v3

## Metadata
- Last updated: 2026-02-25
- Version: 1.0
- Applies to: SurrealDB v3.0.0
- Changelog:
  - v1.0: Initial skill

## Overview

SurrealDB v3 is a major release with several breaking changes from v2. It introduces COMPUTED fields (replacing futures), record references, a custom API endpoint system (DEFINE API), and stable GraphQL support. Scott's stack currently runs v2.6.0 stable, so this skill is a reference for when the team upgrades. The core role of SurrealDB in the stack stays the same: it is the single database layer that handles queries, authentication, real-time subscriptions, and permissions -- all from one place.

## Key Concepts

- **COMPUTED fields** -- Fields whose value is calculated on-the-fly every time you read the record, rather than being stored on disk. They replace the old "futures" syntax from v2. Think of them like a formula column in a spreadsheet: you define the formula once, and it recalculates automatically whenever you view the row.

- **Record References (REFERENCE clause)** -- A way to create relationships between tables where SurrealDB automatically tracks which records point to which. Similar to a lookup column in a spreadsheet. When a referenced record is deleted, you can control what happens (delete the parent too, clear the field, block the delete, etc.).

- **Reverse references (<~ syntax)** -- Lets you query "which records point TO this record?" without storing the relationship on both sides. Think of it like a reverse VLOOKUP: the comic book doesn't need to store its owners, because SurrealDB can follow the reference backwards from the person table.

- **DEFINE API** -- A new (experimental) feature that lets you create custom HTTP endpoints directly inside SurrealDB. Instead of building a separate API server, you write the endpoint logic in SurrealQL. This is useful for exposing specific operations to external services like n8n webhooks.

- **LIVE SELECT** -- Real-time subscriptions that push changes to your app whenever data matching your query changes. Only works over WebSocket connections (not HTTP), and currently limited to single-node deployments.

- **SECURITY DEFINER** -- A way to run certain queries with elevated permissions, bypassing row-level security. Like an admin override for specific operations. Useful when a regular user needs to trigger something that touches data they normally cannot access.

- **Auth system** -- SurrealDB handles user signup, signin, tokens, sessions, and row-level permissions all natively. No need for a separate auth service. The `$auth` variable in queries always refers to the currently signed-in user.

## Patterns & Best Practices

### Use COMPUTED fields instead of futures

Futures are removed in v3. Anywhere you had a future in v2, replace it with COMPUTED.

```sql
-- v2 (old way -- no longer works)
DEFINE FIELD age ON person VALUE <future> { time::year(time::now()) - time::year(born) };

-- v3 (new way)
DEFINE FIELD age ON person COMPUTED time::year(time::now()) - time::year(born);
```

Rules for COMPUTED fields:
- They can only be defined at the top level of a table (not inside nested objects)
- You cannot combine COMPUTED with VALUE or DEFAULT on the same field
- The value is not stored on disk; it is recalculated every time you SELECT the record
- Use them for derived data: age from birthdate, full name from first + last, status from timestamps

### Set up record references with ON DELETE behavior

When one table references another, define what happens if the referenced record is deleted.

```sql
-- Define a field that references another table
DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;

-- ON DELETE options:
-- IGNORE   = silently remove the broken reference from the array
-- UNSET    = set the field to NONE (like clearing a cell)
-- CASCADE  = delete the parent record too
-- REJECT   = block the delete entirely (like a protected reference)
-- THEN     = run custom logic

DEFINE FIELD author ON book TYPE record<person> REFERENCE ON DELETE CASCADE;
```

### Use reverse references to query relationships from the other side

```sql
-- "Which people own this comic book?" -- without storing owners on comic_book
SELECT *, <~person AS owners FROM comic_book;

-- This follows the person.comics reference backwards
-- No need to maintain the relationship in both tables
```

### Choose the right connection mode

| Mode | Protocol | Best for | LIVE SELECT? |
|------|----------|----------|--------------|
| WebSocket | `ws://` or `wss://` | Apps that need real-time updates, long-lived connections | Yes |
| HTTP | `http://` or `https://` | Stateless REST calls, webhooks, n8n integrations | No |

- Use WebSocket for your Nuxt app (real-time updates, subscriptions)
- Use HTTP for n8n workflows and one-off API calls

### Structure authentication properly

```sql
-- Define a record-based access method (users sign up and sign in)
DEFINE ACCESS account ON DATABASE TYPE RECORD
    SIGNUP (
        CREATE user SET
            email = $email,
            pass = crypto::argon2::generate($pass)
    )
    SIGNIN (
        SELECT * FROM user WHERE
            email = $email
            AND crypto::argon2::compare(pass, $pass)
    )
    DURATION FOR TOKEN 15m, FOR SESSION 12h;
```

- **TOKEN duration** = how long the JWT is valid (short -- 15 minutes is good)
- **SESSION duration** = how long before the user has to sign in again (12 hours is reasonable for internal tools)
- Use `crypto::argon2::generate()` to hash passwords -- never store plaintext
- Use `crypto::argon2::compare()` to verify passwords during signin

### Set row-level and field-level permissions

```sql
-- Only let users see their own records
DEFINE TABLE notes PERMISSIONS
    FOR select WHERE created_by = $auth.id
    FOR create WHERE $auth.id IS NOT NONE
    FOR update WHERE created_by = $auth.id
    FOR delete WHERE created_by = $auth.id;

-- Hide sensitive fields from non-admin users
DEFINE FIELD salary ON employee
    PERMISSIONS
        FOR select WHERE $auth.role = 'admin'
        FOR update WHERE $auth.role = 'admin';
```

### Use SECURITY DEFINER for elevated operations

```sql
-- A function that needs to read data the user normally can't access
DEFINE FUNCTION fn::get_team_stats($team_id: string) {
    RETURN SELECT count() FROM team_member WHERE team = type::record('team', $team_id) GROUP ALL;
} SECURITY DEFINER;
-- Runs with full permissions regardless of who calls it
```

### Use DEFINE API for custom endpoints (experimental)

```sql
-- Create a custom GET endpoint
DEFINE API "/users/:id" FOR get THEN {
    LET $user = SELECT * FROM type::record('user', $id);
    RETURN {
        status: 200,
        body: $user
    };
};

-- Create a custom POST endpoint
DEFINE API "/users" FOR post THEN {
    LET $data = $request.body;
    LET $new_user = CREATE user SET name = $data.name, email = $data.email;
    RETURN {
        status: 201,
        body: $new_user
    };
};
```

- Supports path parameters (`:id` becomes `$id` in the handler)
- Can add custom middleware
- Use `--deny-arbitrary-query` server flag to lock down access so only your defined API endpoints work (good for production security)

## Common Gotchas

### Futures no longer exist

**Symptom:** Query fails or `<future>` syntax throws an error.
**Cause:** v3 removed futures entirely.
**Fix:** Replace with COMPUTED fields. See the Patterns section above.

### LET is now required for parameters

**Symptom:** `$myvar = 'hello'` throws a parse error.
**Cause:** v3 requires the `LET` keyword.
**Fix:**
```sql
-- v2 (old -- no longer works)
$name = 'Scott';

-- v3 (required)
LET $name = 'Scott';
```

### Non-existing tables now throw errors

**Symptom:** `SELECT * FROM nonexistent_table` returns an error instead of an empty array.
**Cause:** v3 is stricter. Querying a table that has not been defined (or has no records and no DEFINE TABLE) produces an error.
**Fix:** Always DEFINE TABLE before querying, or handle the error in your app code. In v2 you could query anything and get `[]` back -- that is no longer the case.

### Renamed functions

| v2 function | v3 replacement |
|-------------|---------------|
| `type::thing()` | `type::record()` |
| `rand::guid()` | `rand::id()` |

**Fix:** Find and replace throughout your queries. The old names will throw "function not found" errors.

### Similarity operators removed

**Symptom:** Operators like `~=` or similar fuzzy matching operators throw parse errors.
**Cause:** Removed in v3 in favor of built-in functions.
**Fix:** Use the equivalent function calls instead of operators.

### SEARCH ANALYZER renamed to FULLTEXT ANALYZER

**Symptom:** `DEFINE INDEX ... SEARCH ANALYZER` fails.
**Fix:**
```sql
-- v2 (old)
DEFINE INDEX title_search ON article FIELDS title SEARCH ANALYZER ascii;

-- v3 (new)
DEFINE INDEX title_search ON article FIELDS title FULLTEXT ANALYZER ascii;
```

### MTREE replaced by HNSW for vector indexes

**Symptom:** `DEFINE INDEX ... MTREE` fails.
**Cause:** v3 switched to HNSW (Hierarchical Navigable Small World) for vector similarity search. HNSW is faster for lookups.
**Fix:**
```sql
-- v2 (old)
DEFINE INDEX embedding_idx ON document FIELDS embedding MTREE DIMENSION 1536;

-- v3 (new)
DEFINE INDEX embedding_idx ON document FIELDS embedding HNSW DIMENSION 1536;
```

### Optional chaining syntax changed

**Symptom:** `$val?` throws a parse error.
**Fix:** Use `$val.?` (dot before question mark).
```sql
-- v2 (old)
$user.address?.city

-- v3 (new)
$user.address.?.city
```

### COMPUTED fields cannot be nested or combined

**Symptom:** Trying to use COMPUTED inside a nested object field, or combining it with VALUE or DEFAULT, throws an error.
**Cause:** COMPUTED is top-level only and exclusive.
**Fix:** Only define COMPUTED on top-level fields. If you need a default AND a computation, choose one. Use COMPUTED for derived values and DEFAULT/VALUE for input values.

### LIVE SELECT limitations

**Symptom:** LIVE SELECT does not work or does not fire.
**Cause:** You are either using HTTP instead of WebSocket, or running a multi-node cluster.
**Fix:** LIVE SELECT requires a WebSocket connection (`ws://` or `wss://`) and currently only works on single-node deployments. Switch to WebSocket for real-time features.

## Examples

### Basic CRUD operations

```sql
-- Create a record with a specific ID
CREATE user:scott SET
    name = 'Scott',
    email = 'scott@example.com',
    role = 'admin',
    created_at = time::now();

-- Read records
SELECT * FROM user WHERE role = 'admin';

-- Update a record
UPDATE user:scott SET role = 'superadmin';

-- Delete a record
DELETE user:scott;
```

### COMPUTED field for a full name

```sql
DEFINE FIELD first_name ON person TYPE string;
DEFINE FIELD last_name ON person TYPE string;
DEFINE FIELD full_name ON person COMPUTED string::concat(first_name, ' ', last_name);

CREATE person SET first_name = 'Scott', last_name = 'O\'Dell';
SELECT full_name FROM person;
-- Returns: "Scott O'Dell"
```

### Record references with ON DELETE

```sql
-- Setup
DEFINE TABLE team SCHEMAFULL;
DEFINE FIELD name ON team TYPE string;

DEFINE TABLE member SCHEMAFULL;
DEFINE FIELD name ON member TYPE string;
DEFINE FIELD team ON member TYPE record<team> REFERENCE ON DELETE CASCADE;

-- Create data
CREATE team:engineering SET name = 'Engineering';
CREATE member:alice SET name = 'Alice', team = team:engineering;
CREATE member:bob SET name = 'Bob', team = team:engineering;

-- Reverse reference: "Who is on the engineering team?"
SELECT *, <~member AS members FROM team:engineering;

-- If you DELETE team:engineering, Alice and Bob are also deleted (CASCADE)
```

### LIVE SELECT for real-time updates

```sql
-- Subscribe to new orders over $100
LIVE SELECT * FROM order WHERE total > 100;

-- Subscribe to just the changes (diff format)
LIVE SELECT DIFF FROM order;

-- Kill a live query when done
KILL $live_query_id;
```

### Authentication flow

```sql
-- 1. Define the access method (run once during setup)
DEFINE ACCESS account ON DATABASE TYPE RECORD
    SIGNUP (CREATE user SET email = $email, pass = crypto::argon2::generate($pass))
    SIGNIN (SELECT * FROM user WHERE email = $email AND crypto::argon2::compare(pass, $pass))
    DURATION FOR TOKEN 15m, FOR SESSION 12h;

-- 2. Sign up (from your app via the SDK)
-- db.signup({ access: 'account', variables: { email: 'scott@example.com', pass: 'secret' } })

-- 3. Sign in (from your app via the SDK)
-- db.signin({ access: 'account', variables: { email: 'scott@example.com', pass: 'secret' } })

-- 4. After signin, $auth is available in all queries
SELECT * FROM notes WHERE author = $auth.id;
```

### JWT-based access for external services

```sql
-- Define JWT access for n8n or other services
DEFINE ACCESS api_access ON DATABASE TYPE JWT
    ALGORITHM HS512
    KEY 'your-secret-key-here'
    DURATION FOR TOKEN 1h;
```

Supported algorithms: HS512, RS256.

### Custom API endpoint

```sql
-- Public health check endpoint
DEFINE API "/health" FOR get THEN {
    RETURN {
        status: 200,
        body: { ok: true, time: time::now() }
    };
};

-- Protected endpoint that uses auth
DEFINE API "/my/notes" FOR get THEN {
    LET $notes = SELECT * FROM notes WHERE author = $auth.id;
    RETURN {
        status: 200,
        body: $notes
    };
};
```

## Integration Points

### SurrealDB + Nuxt (via WASM SDK or WebSocket)

- The Nuxt app connects to SurrealDB using either the JavaScript WASM SDK (for embedded use) or a WebSocket client (for server mode)
- Use `useDatabase()` composable pattern to manage the connection
- LIVE SELECT feeds directly into Vue reactive state for real-time UI updates
- All the v2 WASM SDK gotchas (RecordId objects, null vs NONE, etc.) documented in `~/.claude/skills/surrealdb-patterns/SKILL.md` still apply -- verify against v3 SDK when upgrading

### SurrealDB + Tauri (embedded via Rust crate)

```toml
# Cargo.toml
[dependencies]
surrealdb = { version = "3", features = ["kv-surrealkv"] }
```

- **kv-surrealkv** is recommended for Tauri apps (pure Rust, no C dependencies)
- **kv-rocksdb** also works but has C compilation requirements
- **Important:** Tauri's default logging can significantly slow down SurrealDB. Disable it or filter out SurrealDB log messages
- **Important:** Set thread stack size to at least 10 MiB for the SurrealDB thread to avoid stack overflows:

```rust
std::thread::Builder::new()
    .stack_size(10 * 1024 * 1024) // 10 MiB
    .spawn(|| {
        // SurrealDB initialization here
    })
    .expect("Failed to spawn SurrealDB thread");
```

### SurrealDB + n8n

- Use HTTP mode (`http://`) for n8n webhook integrations -- n8n does not maintain long-lived WebSocket connections
- DEFINE API endpoints can serve as clean interfaces for n8n workflows
- Use JWT access for n8n to authenticate against SurrealDB without user credentials
- The `--deny-arbitrary-query` flag can lock down the database so only your defined API endpoints are accessible, which is safer when exposing SurrealDB to external services

### SurrealDB + GraphQL (stable in v3)

- GraphQL is now a stable feature in v3
- Useful if you have external consumers who prefer GraphQL over SurrealQL
- For Scott's stack (Nuxt + SurrealDB), direct SurrealQL queries are simpler and more powerful

## Resources

- [SurrealDB v3 docs](https://surrealdb.com/docs) -- Official documentation
- [SurrealDB v3 release notes](https://surrealdb.com/releases) -- Full changelog and migration details
- [SurrealDB GitHub](https://github.com/surrealdb/surrealdb) -- Source code, issues, discussions
- [WASM SDK (JavaScript)](https://github.com/surrealdb/surrealdb.js) -- JavaScript/TypeScript client
- [Rust SDK](https://docs.rs/surrealdb/latest/surrealdb/) -- Rust crate documentation for Tauri embedding
- Scott's existing v2 patterns: `~/.claude/skills/surrealdb-patterns/SKILL.md`
- Context7 library ID for fast doc lookups: `/surrealdb/docs.surrealdb.com`
