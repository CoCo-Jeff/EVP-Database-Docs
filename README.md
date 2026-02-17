EVP Database Documentation Project
Overview
This repository exists to create clear, consistent, AI-friendly documentation for the EVP database schema.
The target audience is both human developers/analysts and AI agents that need to reason about the data model, generate SQL, and understand business flows built on top of this schema.

The database supports a multi-channel e-commerce system (sales and supply channels, catalog, inventory, orders, settlements, returns, shipping, etc.).
This repo is the canonical place where that physical schema (tables/columns/keys) is linked to business meaning, domain concepts, and common query patterns.

Inputs available to AI agents
An AI agent working with this repo can assume the following core inputs exist (or will exist) here:

Schema.XLSX

Sheet All Fields: complete list of tables and columns, including:

Schema name, table name, column name.

Data type, length/precision/scale, nullability.

Primary-key and foreign-key flags.

Referenced schema/table/column for each FK.

Domain assignment per table (e.g., Channel, Inventory, Orders, Settlements, Shipping, Work Orders, etc.).

Sheet Domains:

Domain abbreviation (e.g., cha, inv, ord, set).

Domain description (e.g., Channel, Inventory, Orders, Settlements).

Domain importance (High/Medium/Low) to guide documentation priority.

Existing narrative documentation (outside this repo, but content may be copied here)

A document that already describes core channel and inventory tables (channels, locations, items, item codes, sales/supply extensions, attributes, availability) and key design patterns (channel isolation, shared-Id extension tables, entity–attribute–value attributes).

The agent’s job is to transform these raw inputs into human-readable markdown documentation, organized by domain and table.

Documentation goals
The goals for this project are:

Comprehensive coverage

Every table in the database should have at least a minimal markdown section: purpose, keys, and relationships.

High-importance domains (Channel, Inventory, Merchant, Data Warehouse, Orders) should receive deeper documentation first.

Business-oriented descriptions

Go beyond column lists; explain what each table represents in the business domain and how it participates in real workflows (e.g., order lifecycle, inventory movements, settlements, automated jobs).

Reusable patterns for AI and humans

Standardize a per-table template so multiple agents (and humans) can extend documentation consistently.

Capture common join patterns and query examples to reduce ambiguity and support automated SQL generation.

Domain-oriented navigation

Organize docs by domain (Channel, Inventory, Orders, Settlements, Shipping, Returns, Work Orders, etc.) so that readers can follow end-to-end flows within a business area.

Target documentation structure
AI agents should aim to generate and maintain the following structure (filenames are suggestions, not strict requirements):

schema-overview.md

High-level description of the EVP database and its purpose.

Summary of global patterns:

Channels and channel isolation.

Items vs item codes vs channel-specific extensions.

Entity–attribute–value patterns.

Common audit fields and Id conventions.

Domain-specific markdown files, for example:

domain-channel.md (Channel)

domain-inventory.md (Inventory)

domain-orders.md (Orders)

domain-settlements.md (Settlements)

domain-shipping.md (Shipping)

domain-returns.md (Returns)

domain-work-orders.md (Work Orders)

domain-catalog-sources.md (Catalog Sources)

domain-business-processes.md (Business Processes)

domain-pricing.md (Pricing)

domain-reference.md (Reference)

Additional domains from the Domains sheet as needed.

Within each domain file:

A short domain overview (what this domain covers and why it matters).

One subsection per table in that domain.

Per-table markdown template
All AI agents should use the same structure for table documentation. Given a table dbo.TableName, the recommended template is:

text
### dbo.TableName

**Domain**  
<Domain name from Schema.XLSX>

**Purpose**  
Short description of what this table represents in the business domain.

**Logical role**  
Where this table sits in the process flow (e.g., “Root entity for settlements”, “Line-level breakdown for shipping cost models”).

**Primary key**
- Column(s) marked as Is Primary Key in Schema.XLSX.

**Foreign keys**
- One bullet per FK inferred from Schema.XLSX (Referenced Table Name + Referenced Column Name).

**Key attributes**

| Column     | Type           | Null | Purpose                          |
|-----------|----------------|------|----------------------------------|
| ColumnName| data_type(...) | Yes/No | Brief description of behavior. |

**Relationships**

- Parent tables:
  - List parent tables and the FK columns that reference them.
- Child tables:
  - List child tables that reference this table.

**Lifecycle and states** (optional but recommended)

- Describe relevant status/state fields (Status, ListingState, etc.) and typical state transitions.

**Business rules**

- Important invariants, uniqueness constraints, and soft rules enforced in code (not just the database).

**Common query patterns**

- 2–3 typical queries or reporting scenarios involving this table, described in words and optionally with example SQL.

**Implementation notes / gotchas**

- Misleading column names, legacy quirks, or special patterns (e.g., extensions that share Id with a parent table rather than having a separate FK).
Agents should automatically populate all structural information (columns, types, PK/FK, domain) from Schema.XLSX, and then enrich with business semantics when provided by the user.

Prioritization strategy for agents
The Domains sheet includes an Importance column that indicates which domains should be documented first (e.g., Channel, Inventory, Merchant, Data Warehouse are marked High).

Agents should:

Respect domain importance

Focus on High-importance domains before Medium or Low.

Within a domain, prioritize central tables

Prefer “root” or central tables (e.g., headers) before satellites (e.g., history, log, or junction tables).

A simple heuristic:

Tables referenced by many FKs are likely central.

Tables with business-sounding names (Order, Settlement, WorkOrder, Return, Item, Channel) are often more important than pure reference tables.

Iterate domain by domain

Complete a minimal pass for all tables in a domain (Purpose, PK/FK, basic relationships).

Then deepen documentation for the most critical tables (lifecycles, rules, query patterns).

How future AI agents should work with the user
An AI agent picking up this project should follow this collaborative pattern:

Identify current domain focus

Ask the user which domain to work on next (e.g., “Channel”, “Inventory”, “Orders”), or read any project notes indicating current focus.

Select tables within that domain

Use Schema.XLSX to list tables for that domain.

Optionally sort by centrality (FK relationships) to pick core tables first.

Generate first-pass docs automatically

For each table, generate:

Domain, purpose (best-effort guess from name + domain).

Primary key and foreign-key lists from Schema.XLSX.

A full field table (columns, types, nullability).

Mark sections that need user input (e.g., lifecycle, business rules, gotchas).

Refine with user input

Ask focused questions:

“What does this table represent in your business?”

“Which 5–10 columns matter the most?”

“Any non-obvious rules or common pitfalls?”

Update the corresponding .md sections accordingly.

Keep docs idempotent and editable

When regenerating docs, avoid breaking human edits.

Prefer appending new sections or clearly labeled updates rather than rewriting entire files unless the user requests it.

Expectations for this repository
This repo is the source of truth for human-readable EVP database documentation.

AI agents should treat Schema.XLSX and these markdown files as authoritative for:

Table and column definitions.

Relationships between tables.

Domain boundaries.

Business meaning of key tables and fields.

Any new documentation or changes generated by an AI agent should:

Use the shared per-table template.

Be grouped into the appropriate domain file.

Preserve existing human-authored content whenever possible.

How to extend this project
If you are an AI agent with access to this repo, you can:

Load Schema.XLSX and determine domains and tables.

For the current domain, generate or update the corresponding domain-*.md file using the template above.

Where business meaning is unclear, emit explicit questions for the human maintainer to answer in future interactions.

Once answers are provided, incorporate them into the markdown sections without altering unrelated content.

This README is intended to give you enough context to continue the documentation effort consistently and safely.
