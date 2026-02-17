# EVP Database Documentation Project

## Overview

This repository provides clear, consistent, AI-friendly documentation for the EVP database schema. The target audience includes both human developers/analysts and AI agents that need to reason about the data model, generate SQL, and understand business flows built on top of this schema.

The database supports a multi-channel e-commerce system covering sales and supply channels, catalog, inventory, orders, settlements, returns, shipping, and more. This repo is the canonical place where the physical schema (tables/columns/keys) is linked to business meaning, domain concepts, and common query patterns.

## Database Domains Reference

The EVP database is organized into **37 domains**, each representing a functional area of the system. The table below lists all domains with their abbreviations and documentation priority:

| Abbreviation | Domain | Importance |
|--------------|--------|------------|
| amz | Amazon | Low |
| bus | Business Processes | Low |
| cat | Catalog Sources | Medium |
| cha | Channel | **High** |
| chg | Change Management | Low |
| cla | Classification | Medium |
| com | Competitors | Low |
| con | Contact | Low |
| cus | Customers | Low |
| dac | Data Access | Medium |
| doc | Documentation | Low |
| dwh | Data Warehouse | **High** |
| eBa | Ebay | Low |
| edi | EDI | Low |
| etl | Etl | Low |
| fba | FBA | Low |
| inv | Inventory | **High** |
| isu | Issues | Low |
| med | Media | Low |
| mer | Merchant | **High** |
| mon | Monitors | Low |
| ord | Orders | Medium |
| pay | Payments | Low |
| pla | Planning | Low |
| prc | Pricing | Low |
| pro | Automated Jobs | Low |
| ref | Reference | Low |
| ret | Returns | Low |
| rpt | Reporting | Low |
| sec | Security | Low |
| set | Settlements | Low |
| shp | Shipping | Low |
| sup | Supplier | Low |
| sys | System | Low |
| wal | Walmart | Low |
| web | Website | Low |
| wrk | Work Orders | Low |

**High-priority domains** (Channel, Data Warehouse, Inventory, Merchant) should be documented first, as they represent core business functionality.

## Inputs Available to AI Agents

An AI agent working with this repo can assume the following core inputs exist (or will exist) here:

### schema-all-fields.json

JSON file containing the complete database schema with 379 tables and 5,365 columns, including:
- Schema name, table name, domain assignment
- Column names, data types, constraints (nullable, primary key, foreign key)
- Foreign key relationships (referenced schema/table/column)
- Structured for easy programmatic access by AI agents

### Schema.XLSX (Original Source)

**Sheet: All Fields** - Complete list of tables and columns, including:
- Schema name, table name, column name
- Data type, length/precision/scale, nullability
- Primary-key and foreign-key flags
- Referenced schema/table/column for each FK
- Domain assignment per table

**Sheet: Domains** - Domain definitions including:
- Domain abbreviation (e.g., cha, inv, ord, set)
- Domain description (e.g., Channel, Inventory, Orders, Settlements)
- Domain importance (High/Medium/Low) to guide documentation priority

### Existing Narrative Documentation

Existing documentation (outside this repo, but content may be copied here) describes core channel and inventory tables, including:
- Channels, locations, items, item codes
- Sales/supply extensions, attributes, availability
- Key design patterns (channel isolation, shared-Id extension tables, entity–attribute–value attributes)

The agent's job is to transform these raw inputs into human-readable markdown documentation, organized by domain and table.

## Documentation Goals

### Comprehensive Coverage
- Every table in the database should have at least a minimal markdown section: purpose, keys, and relationships
- High-importance domains (Channel, Inventory, Merchant, Data Warehouse) should receive deeper documentation first

### Business-Oriented Descriptions
- Go beyond column lists; explain what each table represents in the business domain and how it participates in real workflows (e.g., order lifecycle, inventory movements, settlements, automated jobs)

### Reusable Patterns for AI and Humans
- Standardize a per-table template so multiple agents (and humans) can extend documentation consistently
- Capture common join patterns and query examples to reduce ambiguity and support automated SQL generation

### Domain-Oriented Navigation
- Organize docs by domain (Channel, Inventory, Orders, Settlements, Shipping, Returns, Work Orders, etc.) so that readers can follow end-to-end flows within a business area

## Target Documentation Structure

AI agents should aim to generate and maintain the following structure (filenames are suggestions, not strict requirements):

### High-Level Documentation
- `schema-overview.md` - High-level description of the EVP database and its purpose, including:
  - Summary of global patterns
  - Channels and channel isolation
  - Items vs item codes vs channel-specific extensions
  - Entity–attribute–value patterns
  - Common audit fields and Id conventions

### Domain-Specific Files

Prioritize documentation in this order based on domain importance:

**High Priority:**
- `domain-channel.md` (cha - Channel)
- `domain-data-warehouse.md` (dwh - Data Warehouse)
- `domain-inventory.md` (inv - Inventory)
- `domain-merchant.md` (mer - Merchant)

**Medium Priority:**
- `domain-catalog-sources.md` (cat - Catalog Sources)
- `domain-classification.md` (cla - Classification)
- `domain-data-access.md` (dac - Data Access)
- `domain-orders.md` (ord - Orders)

**Low Priority:**
- Additional domain files as needed based on the domains reference table above

### Domain File Structure

Within each domain file:
- A short domain overview (what this domain covers and why it matters)
- One subsection per table in that domain

## Per-Table Markdown Template

All AI agents should use the same structure for table documentation. Given a table `dbo.TableName`, the recommended template is:

```markdown
### dbo.TableName

**Domain**  
<Domain name from schema-all-fields.json>

**Purpose**  
Short description of what this table represents in the business domain.

**Logical Role**  
Where this table sits in the process flow (e.g., "Root entity for settlements", "Line-level breakdown for shipping cost models").

**Primary Key**
- Column(s) marked as Is Primary Key in schema-all-fields.json

**Foreign Keys**
- One bullet per FK inferred from schema-all-fields.json (Referenced Table Name + Referenced Column Name)

**Key Attributes**

| Column     | Type           | Null | Purpose                          |
|-----------|----------------|------|----------------------------------|
| ColumnName| data_type(...) | Yes/No | Brief description of behavior. |

**Relationships**
- Parent tables:
  - List parent tables and the FK columns that reference them
- Child tables:
  - List child tables that reference this table

**Lifecycle and States** (optional but recommended)
- Describe relevant status/state fields (Status, ListingState, etc.) and typical state transitions

**Business Rules**
- Important invariants, uniqueness constraints, and soft rules enforced in code (not just the database)

**Common Query Patterns**
- 2–3 typical queries or reporting scenarios involving this table, described in words and optionally with example SQL

**Implementation Notes / Gotchas**
- Misleading column names, legacy quirks, or special patterns (e.g., extensions that share Id with a parent table rather than having a separate FK)
```

Agents should automatically populate all structural information (columns, types, PK/FK, domain) from `schema-all-fields.json`, and then enrich with business semantics when provided by the user.

## Prioritization Strategy for Agents

Use the **Database Domains Reference** table above to determine documentation priority. Domains marked as **High** importance should be documented first.

### Agent Prioritization Guidelines

1. **Respect Domain Importance**
   - Focus on High-importance domains before Medium or Low
   - High priority: Channel (cha), Data Warehouse (dwh), Inventory (inv), Merchant (mer)
   - Medium priority: Catalog Sources (cat), Classification (cla), Data Access (dac), Orders (ord)

2. **Within a Domain, Prioritize Central Tables**
   - Prefer "root" or central tables (e.g., headers) before satellites (e.g., history, log, or junction tables)
   - Simple heuristic:
     - Tables referenced by many FKs are likely central
     - Tables with business-sounding names (Order, Settlement, WorkOrder, Return, Item, Channel) are often more important than pure reference tables

3. **Iterate Domain by Domain**
   - Complete a minimal pass for all tables in a domain (Purpose, PK/FK, basic relationships)
   - Then deepen documentation for the most critical tables (lifecycles, rules, query patterns)

## How Future AI Agents Should Work with the User

An AI agent picking up this project should follow this collaborative pattern:

### 1. Identify Current Domain Focus
- Ask the user which domain to work on next (e.g., "Channel", "Inventory", "Orders"), or read any project notes indicating current focus
- Reference the **Database Domains Reference** table to confirm domain abbreviation and importance level

### 2. Select Tables Within That Domain
- Use `schema-all-fields.json` to list tables for that domain
- Optionally sort by centrality (FK relationships) to pick core tables first

### 3. Generate First-Pass Docs Automatically
For each table, generate:
- Domain, purpose (best-effort guess from name + domain)
- Primary key and foreign-key lists from `schema-all-fields.json`
- A full field table (columns, types, nullability)
- Mark sections that need user input (e.g., lifecycle, business rules, gotchas)

### 4. Refine with User Input
Ask focused questions:
- "What does this table represent in your business?"
- "Which 5–10 columns matter the most?"
- "Any non-obvious rules or common pitfalls?"
- Update the corresponding `.md` sections accordingly

### 5. Keep Docs Idempotent and Editable
- When regenerating docs, avoid breaking human edits
- Prefer appending new sections or clearly labeled updates rather than rewriting entire files unless the user requests it

## Expectations for This Repository

This repo is the source of truth for human-readable EVP database documentation.

AI agents should treat `schema-all-fields.json` and these markdown files as authoritative for:
- Table and column definitions
- Relationships between tables
- Domain boundaries
- Business meaning of key tables and fields

Any new documentation or changes generated by an AI agent should:
- Use the shared per-table template
- Be grouped into the appropriate domain file
- Preserve existing human-authored content whenever possible

## How to Extend This Project

If you are an AI agent with access to this repo, you can:

1. Load `schema-all-fields.json` to access all table and column definitions
2. Reference the **Database Domains Reference** table to prioritize work
3. For the current domain, generate or update the corresponding `domain-*.md` file using the template above
4. Where business meaning is unclear, emit explicit questions for the human maintainer to answer in future interactions
5. Once answers are provided, incorporate them into the markdown sections without altering unrelated content

This README is intended to give you enough context to continue the documentation effort consistently and safely.