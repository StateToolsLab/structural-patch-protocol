# DEV-PROTOCOL-CASE-DB-001

# Schema Mismatch Caused by Actual Artifact Violation

## 0. Status

Motivating failure case.

This case documents a schema mismatch caused by implementing against inferred database structure instead of inspecting the actual artifact.

## 1. Context

During an AI-assisted crawler implementation, the agent assumed that the database schema matched the design notes and previous sprint expectations.

The implementation proceeded without first verifying the actual database file, table names, or column names.

## 2. Observed Mismatches

The following mismatches were observed:

```text
industry.duckdb had no expected table
url column did not exist       → actual: crawl_url
raw_hash column did not exist  → actual: body_hash
source_id column did not exist → actual: company_id
```

## 3. Failure Mode

This is an Actual Artifact Violation.

The agent treated design assumptions as evidence.

```text
Designed ≠ implemented
Implemented ≠ current artifact
Previous sprint complete ≠ safe to reference without inspection
```

## 4. SPP Interpretation

This case motivates the Actual Artifact First Gate.

Before modifying code that depends on a database, the agent must inspect:

```text
DB file path
available tables
actual table schema
sample rows or query result
```

Examples:

```sql
SHOW TABLES;
DESCRIBE <table>;
SELECT * FROM <table> LIMIT 5;
```

## 5. Correct SPP Pattern

```text
Target Lock
→ Inspect actual DB path
→ SHOW TABLES
→ DESCRIBE target table
→ Apply minimal Probe Patch
→ Run integration query or test
→ Verified Merge
→ Stop
```

## 6. Rule Extracted

```text
The artifact itself is evidence.
```

And:

```text
Design notes are not evidence.
Previous sprint notes are not evidence.
Naming conventions are not evidence.
```
