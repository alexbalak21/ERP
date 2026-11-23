# Building a Metadata-Driven ERP Inspired by Sage X3

## Goal

Build an ERP platform where administrators can create and customize:

- Tables
- Fields
- Links
- Indexes
- Screens
- Objects
- Actions
- Menus
- Local Menus
- Business Rules
- Workflows
- Custom Code

without modifying application source code.

The ERP should work similarly to Sage X3, where most functionality is generated from metadata stored in a dictionary.

---

# High Level Architecture

```text
+-------------------+
|  React Frontend   |
+-------------------+
          |
          v
+-------------------+
| Metadata Engine   |
+-------------------+
          |
          v
+-------------------+
| Business Runtime  |
+-------------------+
          |
          v
+-------------------+
| PostgreSQL        |
+-------------------+
          |
          +------------------+
          | Metadata Tables  |
          +------------------+
          |
          +------------------+
          | Business Tables  |
          +------------------+
```

---

# Core Concept

Never create screens manually.

Instead:

```text
Administrator creates metadata
↓
ERP generates:
    Tables
    APIs
    Forms
    Menus
    Security
    Reports
```

---

# Phase 1 - Metadata Schema

The metadata repository drives the entire ERP.

---

# Meta Tables

Stores all business tables.

```sql
CREATE TABLE meta_tables
(
    id UUID PRIMARY KEY,
    code VARCHAR(50),
    label VARCHAR(255),
    description TEXT,
    module VARCHAR(50),

    is_system BOOLEAN,
    is_custom BOOLEAN,

    created_at TIMESTAMP,
    updated_at TIMESTAMP
);
```

Example:

```text
CUSTOMER
SUPPLIER
PRODUCT
SALES_ORDER
```

---

# Meta Fields

Stores all field definitions.

```sql
CREATE TABLE meta_fields
(
    id UUID PRIMARY KEY,

    table_id UUID,

    field_name VARCHAR(50),

    label VARCHAR(255),

    field_type VARCHAR(50),

    length INTEGER,

    decimals INTEGER,

    mandatory BOOLEAN,

    default_value TEXT,

    local_menu_id UUID,

    lookup_table VARCHAR(50),

    is_custom BOOLEAN
);
```

Example:

```text
CUSTOMER.CODE
CUSTOMER.NAME
CUSTOMER.COUNTRY
```

---

# Supported Field Types

```text
String
Text
Integer
Decimal
Boolean
Date
Datetime
Guid
Blob
Json
```

Additional ERP types:

```text
Customer
Supplier
Product
Site
User
Currency
Company
```

These generate automatic selection lists.

---

# Meta Relations

Equivalent to Sage Links.

```sql
CREATE TABLE meta_relations
(
    id UUID PRIMARY KEY,

    source_table VARCHAR(50),
    source_field VARCHAR(50),

    target_table VARCHAR(50),
    target_field VARCHAR(50),

    relation_type VARCHAR(20)
);
```

Examples:

```text
CUSTOMER -> COUNTRY
PRODUCT -> CATEGORY
ORDER -> CUSTOMER
```

---

# Meta Indexes

Equivalent to Sage indexes.

```sql
CREATE TABLE meta_indexes
(
    id UUID PRIMARY KEY,

    table_name VARCHAR(50),

    index_name VARCHAR(100),

    unique_flag BOOLEAN
);
```

---

# Meta Index Fields

```sql
CREATE TABLE meta_index_fields
(
    index_id UUID,

    field_name VARCHAR(50),

    sequence INTEGER
);
```

---

# Phase 2 - Dynamic Database Engine

When user creates metadata, ERP updates PostgreSQL automatically.

---

# Create Table

User creates:

```text
CUSTOMER
```

System generates:

```sql
CREATE TABLE customer
(
    id UUID PRIMARY KEY
);
```

---

# Add Field

User adds:

```text
CODE
VARCHAR(30)
```

System generates:

```sql
ALTER TABLE customer
ADD COLUMN code VARCHAR(30);
```

---

# Add Relation

User creates relation:

```text
ORDER.CUSTOMER_ID
→ CUSTOMER.ID
```

System generates:

```sql
ALTER TABLE sales_order
ADD CONSTRAINT fk_order_customer
FOREIGN KEY(customer_id)
REFERENCES customer(id);
```

---

# Migration System

Never execute SQL directly.

Generate migrations.

Example:

```text
Migration 0001

CREATE TABLE CUSTOMER
CREATE COLUMN CODE
CREATE COLUMN NAME
```

Store every migration.

```sql
meta_migrations
```

Like Entity Framework.

---

# Phase 3 - Screen Engine

Equivalent to Sage Screens.

---

# Meta Screens

```sql
CREATE TABLE meta_screens
(
    id UUID PRIMARY KEY,

    code VARCHAR(50),

    title VARCHAR(255),

    table_name VARCHAR(50)
);
```

Example:

```text
CUSTOMER_SCREEN
```

---

# Screen Sections

```sql
CREATE TABLE meta_screen_sections
(
    id UUID PRIMARY KEY,

    screen_id UUID,

    title VARCHAR(255),

    position INTEGER
);
```

Example:

```text
General
Address
Accounting
Statistics
```

---

# Screen Fields

```sql
CREATE TABLE meta_screen_fields
(
    screen_id UUID,

    field_id UUID,

    section_id UUID,

    row_number INTEGER,

    column_number INTEGER
);
```

This controls the layout.

---

# Dynamic Form Generator

Frontend reads metadata:

```json
{
  "field": "COUNTRY",
  "type": "lookup"
}
```

Automatically renders:

```text
Country
[ France ▼ ]
```

No coding required.

---

# Phase 4 - Object Engine

Equivalent to Sage X3 Objects.

Object = CRUD behavior.

---

# Meta Objects

```sql
CREATE TABLE meta_objects
(
    id UUID PRIMARY KEY,

    code VARCHAR(50),

    table_name VARCHAR(50),

    screen_code VARCHAR(50)
);
```

Example:

```text
CUSTOMER

TABLE:
CUSTOMER

SCREEN:
CUSTOMER_SCREEN
```

---

# Object Capabilities

```text
Create
Read
Update
Delete
Duplicate
Import
Export
Print
```

Enabled using metadata.

---

# Phase 5 - Menus

---

# Menu Definitions

```sql
CREATE TABLE meta_menus
(
    id UUID PRIMARY KEY,

    parent_id UUID,

    label VARCHAR(255),

    icon VARCHAR(50),

    sequence INTEGER,

    target_type VARCHAR(20),

    target_code VARCHAR(50)
);
```

Example:

```text
Sales
 └ Customers
 └ Orders
 └ Deliveries
```

Generated dynamically.

---

# Phase 6 - Local Menus

Equivalent to Sage Local Menus.

---

# Local Menu

```sql
CREATE TABLE meta_local_menus
(
    id UUID PRIMARY KEY,

    code VARCHAR(50),

    description VARCHAR(255)
);
```

---

# Values

```sql
CREATE TABLE meta_local_menu_values
(
    id UUID PRIMARY KEY,

    menu_id UUID,

    value_code INTEGER,

    label VARCHAR(255)
);
```

---

Example

```text
Order Status

1 Draft
2 Open
3 Closed
4 Cancelled
```

---

# Phase 7 - Event Engine

Equivalent to Actions.

---

# Event Types

```text
BEFORE_CREATE
AFTER_CREATE

BEFORE_UPDATE
AFTER_UPDATE

BEFORE_DELETE
AFTER_DELETE

BEFORE_VALIDATE
AFTER_VALIDATE

ON_DISPLAY
ON_IMPORT
ON_EXPORT
```

---

# Meta Actions

```sql
CREATE TABLE meta_actions
(
    id UUID PRIMARY KEY,

    object_code VARCHAR(50),

    event_name VARCHAR(50),

    script_id UUID
);
```

---

# Phase 8 - Business Logic

This is the most important part.

---

# Rule Engine

Simple business rules.

Example:

```text
IF CreditLimit < 0
THROW Error
```

Stored as metadata.

---

# Recommended Script Engine

Instead of creating a language immediately:

Store Python scripts.

```python
if record["credit_limit"] < 0:
    raise Exception(
        "Credit limit cannot be negative"
    )
```

or

JavaScript:

```javascript
if(record.credit_limit < 0)
{
    throw "Invalid value";
}
```

---

# Script Storage

```sql
CREATE TABLE meta_scripts
(
    id UUID PRIMARY KEY,

    code VARCHAR(50),

    language VARCHAR(20),

    content TEXT
);
```

---

# Execution Example

```text
Customer Save
↓
BEFORE_UPDATE
↓
Execute Script
↓
Continue Save
```

---

# Phase 9 - Workflow Engine

---

# Workflow Definition

```sql
CREATE TABLE meta_workflows
(
    id UUID PRIMARY KEY,

    code VARCHAR(50),

    name VARCHAR(255)
);
```

---

Example

```text
Purchase Request

Step 1:
Manager Approval

Step 2:
Finance Approval

Step 3:
Order Creation
```

No code required.

---

# Phase 10 - API Generator

Every object automatically exposes:

```http
GET     /api/customers
GET     /api/customers/1

POST    /api/customers

PUT     /api/customers/1

DELETE  /api/customers/1
```

Generated from metadata.

---

# Phase 11 - Security

---

# Roles

```sql
roles
```

Example:

```text
ADMIN
SALES
PURCHASING
ACCOUNTING
```

---

# Permissions

```sql
permissions
```

Examples:

```text
CUSTOMER.READ
CUSTOMER.CREATE
CUSTOMER.UPDATE
CUSTOMER.DELETE
```

---

# Phase 12 - Audit Trail

Track every change.

---

```sql
audit_log
```

Stores:

```text
User
Date
Table
Record
Field
Old Value
New Value
```

Like Sage traceability.

---

# Phase 13 - Versioning

Every customization should be exportable.

Example:

```text
YCUSTOMERS
YSALES
YFIELDS
```

Package as:

```json
customization.json
```

Import into another environment.

Equivalent to:

```text
Sage patches
```

---

# Final Goal

An administrator should be able to:

1. Create a table
2. Add fields
3. Create links
4. Add indexes
5. Create menus
6. Create screens
7. Create objects
8. Create workflows
9. Add business logic
10. Publish

Without writing backend code.

The platform generates:

- Database structure
- APIs
- Forms
- Menus
- Security
- Workflows
- Reports

from metadata, exactly like the philosophy behind Sage X3.