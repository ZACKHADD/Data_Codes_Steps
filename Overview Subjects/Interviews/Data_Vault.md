# Complete Data Vault Guide for Snowflake Tech Leads

## Table of Contents
1. [What is Data Vault](#what-is-data-vault)
2. [Core Components and Architecture](#core-components-and-architecture)
3. [Data Vault vs Medallion Architecture](#data-vault-vs-medallion-architecture)
4. [Benefits of Data Vault](#benefits-of-data-vault)
5. [Limitations and Challenges](#limitations-and-challenges)
6. [Snowflake Implementation Guide](#snowflake-implementation-guide)
7. [Best Practices](#best-practices)
8. [Sample Implementation](#sample-implementation)

## What is Data Vault

Data Vault is a data modeling methodology designed specifically for data warehousing that provides long-term historical storage of data coming from multiple operational systems. Developed by Dan Linstedt, it's built on three foundational principles:

- **Flexibility**: Easily accommodate changes in business requirements
- **Scalability**: Handle growing data volumes and sources
- **Auditability**: Maintain complete data lineage and historical context

Data Vault combines the benefits of Third Normal Form (3NF) and Star Schema approaches while addressing their respective weaknesses in enterprise data warehousing scenarios.

## Core Components and Architecture

Data Vault uses three core components, each serving a specific purpose in creating a resilient, auditable data warehouse. The naming comes from the analogy of a bank vault system:

### Visual Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         DATA VAULT ARCHITECTURE                                 │
│                     (Showing Transactional Entities)                           │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐    ┌─────────────────────┐    ┌─────────────┐                  │
│  │    HUB      │    │        LINK         │    │    HUB      │                  │
│  │  CUSTOMER   │◄───┤   CUSTOMER_ORDER    ├───►│    ORDER    │                  │
│  │             │    │                     │    │             │                  │
│  │ customer_hk │    │  customer_order_hk  │    │  order_hk   │                  │
│  │ customer_id │    │    customer_hk      │    │  order_id   │ ◄─── Transactional │
│  │  load_date  │    │     order_hk        │    │ load_date   │      Entity       │
│  │record_source│    │    load_date        │    │record_source│                  │
│  └─────┬───────┘    │   record_source     │    └─────┬───────┘                  │
│        │            └─────────────────────┘          │                          │
│        │                                             │                          │
│        ▼                                             ▼                          │
│ ┌──────────────────┐                        ┌─────────────────────┐             │
│ │   SATELLITE      │                        │ Multiple Satellites │             │
│ │ CUSTOMER_DETAILS │                        │   for ORDER HUB     │             │
│ │                  │                        │                     │             │
│ │   customer_hk    │                        │ ┌─────────────────┐ │             │
│ │   load_date      │                        │ │ SAT_ORDER_      │ │             │
│ │  load_end_date   │                        │ │   STATUS        │ │             │
│ │   hash_diff      │                        │ │ • order_status  │ │ ◄─── Changes │
│ │  customer_name   │                        │ │ • status_reason │ │      Frequently│
│ │     email        │                        │ │ • updated_by    │ │             │
│ │     phone        │                        │ └─────────────────┘ │             │
│ │    address       │                        │                     │             │
│ └──────────────────┘                        │ ┌─────────────────┐ │             │
│                                             │ │ SAT_ORDER_      │ │             │
│                                             │ │  FINANCIAL      │ │             │
│                                             │ │ • order_amount  │ │ ◄─── Changes │
│                                             │ │ • tax_amount    │ │      Occasionally│
│                                             │ │ • discount      │ │             │
│                                             │ │ • currency      │ │             │
│                                             │ └─────────────────┘ │             │
│                                             │                     │             │
│                                             │ ┌─────────────────┐ │             │
│                                             │ │ SAT_ORDER_      │ │             │
│                                             │ │   DETAILS       │ │             │
│                                             │ │ • order_date    │ │ ◄─── Rarely  │
│                                             │ │ • delivery_date │ │      Changes │
│                                             │ │ • ship_address  │ │             │
│                                             │ │ • order_type    │ │             │
│                                             │ └─────────────────┘ │             │
│                                             └─────────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Key Insights for Transactional Data:

**1. Every Hub Can Have Multiple Satellites**
- **Master Data** (Customer): Usually 1-2 satellites (basic info, preferences)
- **Transactional Data** (Order): Often 3-5 satellites (status, financial, details, shipping, etc.)

**2. Satellite Separation Strategy**
```
┌─────────────────────────────────────────────────────────────┐
│                    SATELLITE DESIGN PRINCIPLES              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Separate satellites by:                                    │
│                                                             │
│  📊 CHANGE FREQUENCY                                        │
│     • High frequency: Status, workflow states              │
│     • Medium frequency: Financial amounts, calculations     │
│     • Low frequency: Basic details, references             │
│                                                             │
│  🏢 SOURCE SYSTEM                                          │
│     • Payment system updates → Financial satellite         │
│     • Workflow engine updates → Status satellite           │
│     • User interface updates → Details satellite           │
│                                                             │
│  📋 FUNCTIONAL AREA                                        │
│     • Financial team needs → Financial satellite           │
│     • Operations team needs → Status satellite             │
│     • Customer service needs → Details satellite           │
│                                                             │
│  🔄 UPDATE PATTERNS                                        │
│     • Real-time updates → Status satellite                 │
│     • Batch updates → Financial satellite                  │
│     • Manual updates → Details satellite                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1. HUB Tables - "The Vault's Safe Deposit Boxes"

**WHY HUBS?** Think of Hubs as the safe deposit boxes in a bank vault. Each box has a unique number (hash key) and contains the most important thing - the identity of what belongs to whom (business key).

**PURPOSE:**
- **Identity Storage**: Store unique business identifiers (Customer ID, Product Code, etc.)
- **Deduplication**: Ensure each business entity appears only once
- **Source Independence**: Same customer from different systems gets same hub record
- **Audit Trail**: Track when we first saw this entity and from which source

**Why called "Hub"?** Like the hub of a wheel, all information about an entity radiates out from this central point.

```sql
-- Example: Why we need hubs
-- Customer "CUST001" might appear in:
-- - CRM system as "CUST001" 
-- - Billing system as "CUST001"
-- - Support system as "CUST001"
-- Hub ensures we have ONE record for this customer

CREATE TABLE hub_customer (
    customer_hk VARCHAR(32) PRIMARY KEY,  -- Hash key (our vault box number)
    customer_id VARCHAR(50) NOT NULL,     -- Business key (what the business calls this customer)
    load_date TIMESTAMP_NTZ NOT NULL,     -- When we first saw this customer
    record_source VARCHAR(50) NOT NULL,   -- Which system first told us about this customer
    UNIQUE(customer_id)
);
```

### 2. LINK Tables - "The Vault's Relationship Records"

**WHY LINKS?** Links are like the vault's record book that tracks which safe deposit boxes are related to each other. They answer "Who is connected to what, when, and where did we learn about it?"

**PURPOSE:**
- **Relationship Storage**: Capture associations between business entities
- **Transaction Recording**: Track when relationships were first observed
- **Many-to-Many Support**: Handle complex relationships naturally
- **Source Attribution**: Know which system told us about each relationship

**Why called "Link"?** They literally link hubs together, creating the chain of relationships.

```sql
-- Example: Why we need links
-- Question: "Which customers placed which orders?"
-- Without links: We'd have to duplicate customer data in order table
-- With links: Clean separation - customer exists in hub, order exists in hub, 
--            relationship exists in link

CREATE TABLE link_customer_order (
    customer_order_hk VARCHAR(32) PRIMARY KEY,  -- Hash of the relationship
    customer_hk VARCHAR(32) NOT NULL,           -- Points to customer hub
    order_hk VARCHAR(32) NOT NULL,              -- Points to order hub  
    load_date TIMESTAMP_NTZ NOT NULL,           -- When we first saw this relationship
    record_source VARCHAR(50) NOT NULL,         -- Which system told us about it
    FOREIGN KEY (customer_hk) REFERENCES hub_customer(customer_hk),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);
```

### 3. SATELLITE Tables - "The Vault's Detailed Information Files"

**WHY SATELLITES?** Satellites are like detailed file folders that orbit around each safe deposit box (hub) or relationship record (link). They contain all the descriptive information and track how it changes over time.

**PURPOSE:**
- **Attribute Storage**: Store all descriptive data (names, addresses, amounts, etc.)
- **Historical Tracking**: Maintain complete history of changes
- **Change Detection**: Efficiently identify what data has changed
- **Context Preservation**: Keep different types of information separate

**Why called "Satellite"?** Like satellites orbiting a planet, they revolve around hubs and links, providing detailed information about them.

```sql
-- Example: Why we need satellites
-- Problem: Customer name changes from "John Smith" to "John Smith Jr."
-- Traditional approach: Update in place (lose history)
-- Satellite approach: Add new record, keep old one
-- Result: Complete audit trail of all changes

CREATE TABLE sat_customer_details (
    customer_hk VARCHAR(32) NOT NULL,        -- Which customer this describes
    load_date TIMESTAMP_NTZ NOT NULL,        -- When this version became active
    load_end_date TIMESTAMP_NTZ,             -- When this version became inactive
    record_source VARCHAR(50) NOT NULL,      -- Source of this information
    hash_diff VARCHAR(32) NOT NULL,          -- Hash of all attributes (change detection)
    -- THE ACTUAL BUSINESS DATA:
    customer_name VARCHAR(100),               -- Can change over time
    email VARCHAR(100),                       -- Can change over time
    phone VARCHAR(20),                        -- Can change over time
    address VARCHAR(200),                     -- Can change over time
    PRIMARY KEY (customer_hk, load_date),
    FOREIGN KEY (customer_hk) REFERENCES hub_customer(customer_hk)
);
```

### Real-World Example: Transactional Data (Orders) with Satellites

**CRITICAL CONCEPT**: Every hub (whether master data like Customer or transactional data like Order) can have multiple satellites to track different aspects and changes over time.

#### Order Hub + Multiple Satellites Example

```sql
-- 1. ORDER HUB (The transaction identity)
CREATE TABLE hub_order (
    order_hk VARCHAR(40) PRIMARY KEY,        -- Hash of order_id
    order_id VARCHAR(50) NOT NULL UNIQUE,   -- Business key (ORDER-2024-001)
    load_date TIMESTAMP_NTZ NOT NULL,        -- When we first saw this order
    record_source VARCHAR(50) NOT NULL      -- Which system created the order
);

-- 2. ORDER FINANCIAL SATELLITE (Financial details that can change)
CREATE TABLE sat_order_financial (
    order_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    -- Financial attributes that can change:
    order_amount DECIMAL(15,2),              -- Total amount
    tax_amount DECIMAL(15,2),                -- Tax (can be recalculated)
    discount_amount DECIMAL(15,2),           -- Discounts applied
    currency_code VARCHAR(3),                -- Currency
    exchange_rate DECIMAL(10,4),             -- Exchange rate (changes daily)
    PRIMARY KEY (order_hk, load_date),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);

-- 3. ORDER STATUS SATELLITE (Status changes - very dynamic)
CREATE TABLE sat_order_status (
    order_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    -- Status attributes that change frequently:
    order_status VARCHAR(50),                -- PENDING -> CONFIRMED -> SHIPPED -> DELIVERED
    status_reason VARCHAR(200),              -- Why status changed
    updated_by VARCHAR(50),                  -- Who changed the status
    PRIMARY KEY (order_hk, load_date),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);

-- 4. ORDER DETAILS SATELLITE (Basic order info - rarely changes)
CREATE TABLE sat_order_details (
    order_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    -- Basic order details:
    order_date DATE,                         -- When order was placed
    delivery_date DATE,                      -- Requested delivery date
    shipping_address VARCHAR(500),           -- Shipping address
    order_type VARCHAR(50),                  -- Online, Phone, In-store
    PRIMARY KEY (order_hk, load_date),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);
```

#### Why Multiple Satellites for Orders?

**Different Change Frequencies:**
- **Status Satellite**: Changes every few hours (pending → confirmed → shipped)
- **Financial Satellite**: Changes occasionally (price adjustments, tax recalculations)
- **Details Satellite**: Rarely changes (shipping address correction)

**Example Order Lifecycle:**

```sql
-- Day 1: Order created
INSERT INTO hub_order VALUES ('ABC123', 'ORDER-2024-001', '2024-01-01 10:00:00', 'ECOMMERCE');

INSERT INTO sat_order_details VALUES (
    'ABC123', '2024-01-01 10:00:00', NULL, 'ECOMMERCE', 'HASH1',
    '2024-01-01', '2024-01-05', '123 Main St', 'Online'
);

INSERT INTO sat_order_financial VALUES (
    'ABC123', '2024-01-01 10:00:00', NULL, 'ECOMMERCE', 'HASH2', 
    100.00, 8.50, 0.00, 'USD', 1.0000
);

INSERT INTO sat_order_status VALUES (
    'ABC123', '2024-01-01 10:00:00', NULL, 'ECOMMERCE', 'HASH3',
    'PENDING', 'Order received', 'system'
);

-- Day 1 - 2 hours later: Status changes (only status satellite gets new record)
UPDATE sat_order_status SET load_end_date = '2024-01-01 12:00:00' 
WHERE order_hk = 'ABC123' AND load_end_date IS NULL;

INSERT INTO sat_order_status VALUES (
    'ABC123', '2024-01-01 12:00:00', NULL, 'PAYMENT_SYSTEM', 'HASH4',
    'CONFIRMED', 'Payment approved', 'payment_gateway'
);

-- Day 2: Discount applied (only financial satellite gets new record)
UPDATE sat_order_financial SET load_end_date = '2024-01-02 09:00:00' 
WHERE order_hk = 'ABC123' AND load_end_date IS NULL;

INSERT INTO sat_order_financial VALUES (
    'ABC123', '2024-01-02 09:00:00', NULL, 'PROMOTION_ENGINE', 'HASH5',
    100.00, 8.50, 10.00, 'USD', 1.0000  -- Discount applied
);
```

#### Purchase/Invoice Example with Satellites

```sql
-- PURCHASE HUB
CREATE TABLE hub_purchase (
    purchase_hk VARCHAR(40) PRIMARY KEY,
    purchase_id VARCHAR(50) NOT NULL UNIQUE,  -- PO-2024-001
    load_date TIMESTAMP_NTZ NOT NULL,
    record_source VARCHAR(50) NOT NULL
);

-- PURCHASE APPROVAL SATELLITE (Approval workflow tracking)
CREATE TABLE sat_purchase_approval (
    purchase_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    approval_status VARCHAR(50),              -- DRAFT -> PENDING -> APPROVED -> REJECTED
    approved_by VARCHAR(50),                  -- Manager who approved
    approval_date TIMESTAMP_NTZ,             -- When approved
    approval_comments VARCHAR(1000),         -- Approval notes
    PRIMARY KEY (purchase_hk, load_date)
);

-- PURCHASE FINANCIAL SATELLITE (Financial tracking)
CREATE TABLE sat_purchase_financial (
    purchase_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    purchase_amount DECIMAL(15,2),
    budget_code VARCHAR(50),
    cost_center VARCHAR(50),
    payment_terms VARCHAR(50),               -- NET30, NET60, etc.
    payment_status VARCHAR(50),              -- PENDING, PAID, OVERDUE
    PRIMARY KEY (purchase_hk, load_date)
);
```

### Real-World Scenario: Order Status Change

**Scenario**: Order "ORDER-2024-001" changes from "SHIPPED" to "DELIVERED"

**Traditional Approach**:
```sql
-- This loses the shipping timestamp!
UPDATE orders 
SET status = 'DELIVERED', last_updated = NOW()
WHERE order_id = 'ORDER-2024-001';
```

**Data Vault Approach**:
```sql
-- Hub stays unchanged (order identity doesn't change)
-- Financial and details satellites stay unchanged (amounts didn't change)
-- Only status satellite gets new record:

-- Close current status record
UPDATE sat_order_status 
SET load_end_date = CURRENT_TIMESTAMP()
WHERE order_hk = 'ABC123...' AND load_end_date IS NULL;

-- Insert new status record (keeps full history)
INSERT INTO sat_order_status VALUES (
    'ABC123...', CURRENT_TIMESTAMP(), NULL, 'DELIVERY_SYSTEM', 'HASH_NEW',
    'DELIVERED', 'Package delivered to customer', 'delivery_driver_001'
);

-- Result: We can now see:
-- - When it was PENDING (with reason)
-- - When it was CONFIRMED (with reason) 
-- - When it was SHIPPED (with reason)
-- - When it was DELIVERED (with reason)
-- All with timestamps and who/what changed it!
```

## The Insert-Only Principle: Core of Data Vault

### The Golden Rules

**Data Vault Raw Layer: NEVER UPDATE - ONLY INSERT**
- Hubs: Insert-only (never update)
- Links: Insert-only (never update) 
- Satellites: Insert-only with `load_end_date` updates for closing records

**Information Marts: Present "Current" Views**
- Use flags, window functions, or filtering to show current state
- Rebuild-able from the raw vault at any time

### How Insert-Only Works in Practice

#### 1. Hub Tables - Pure Insert-Only

```sql
-- Hub Customer - NEVER gets updated, only new customers inserted
INSERT INTO hub_customer (customer_hk, customer_id, load_date, record_source)
SELECT 
    generate_hash_key(customer_id),
    customer_id,
    CURRENT_TIMESTAMP(),
    'CRM_SYSTEM'
FROM staging_customer s
WHERE NOT EXISTS (
    SELECT 1 FROM hub_customer h 
    WHERE h.customer_id = s.customer_id
);

-- Result: Hub grows, but existing records never change
-- CUST001 inserted 2024-01-01 → Stays forever unchanged
-- CUST002 inserted 2024-01-02 → Stays forever unchanged
-- CUST003 inserted 2024-01-03 → Stays forever unchanged
```

#### 2. Satellite Tables - Insert + End-Dating Pattern

```sql
-- When customer email changes from john@old.com to john@new.com

-- STEP 1: Close the current record (ONLY update allowed)
UPDATE sat_customer_details 
SET load_end_date = CURRENT_TIMESTAMP()
WHERE customer_hk = 'ABC123...' 
  AND load_end_date IS NULL;  -- Only the current record

-- STEP 2: Insert new record with changed data
INSERT INTO sat_customer_details (
    customer_hk, load_date, load_end_date, hash_diff,
    customer_name, email, phone, address
) VALUES (
    'ABC123...', 
    CURRENT_TIMESTAMP(), 
    NULL,  -- This is now the current record
    'NEW_HASH_VALUE',
    'John Smith', 
    'john@new.com',  -- Changed value
    '555-1234', 
    '123 Main St'
);
```

#### 3. The Historical Timeline Result

After several changes, your satellite looks like this:

```sql
-- sat_customer_details table contents:
customer_hk | load_date           | load_end_date       | email           | customer_name
------------|--------------------|--------------------|-----------------|---------------
ABC123...   | 2024-01-01 10:00  | 2024-02-15 14:30  | john@old.com   | John Smith
ABC123...   | 2024-02-15 14:30  | 2024-03-10 09:15  | john@new.com   | John Smith  
ABC123...   | 2024-03-10 09:15  | 2024-04-22 16:45  | john@new.com   | John Smith Jr.
ABC123...   | 2024-04-22 16:45  | NULL              | j.smith@new.com| John Smith Jr.
```

**Perfect Historical Preservation:**
- January: We can see he was "john@old.com" 
- February: Email changed to "john@new.com"
- March: Name changed to "John Smith Jr."
- April: Email changed to "j.smith@new.com" (current)

### Information Marts: How to Show "Current" Values

#### Method 1: Using load_end_date IS NULL Filter

```sql
-- Current Customer View - Most Common Approach
CREATE OR REPLACE VIEW mart_current_customers AS
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.phone,
    s.address,
    s.load_date as last_updated
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
WHERE s.load_end_date IS NULL;  -- Only current records

-- Result: Shows only latest values
-- customer_id | customer_name   | email            | last_updated
-- CUST001     | John Smith Jr.  | j.smith@new.com | 2024-04-22 16:45
```

#### Method 2: Using Row Number (Alternative Pattern)

```sql
-- If you prefer not to use load_end_date
CREATE OR REPLACE VIEW mart_current_customers_v2 AS
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.phone,
    s.address,
    s.load_date as last_updated
FROM hub_customer h
JOIN (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY customer_hk ORDER BY load_date DESC) as rn
    FROM sat_customer_details
) s ON h.customer_hk = s.customer_hk
WHERE s.rn = 1;  -- Most recent record only
```

#### Method 3: Historical Point-in-Time Views

```sql
-- "What did customer look like on specific date?"
CREATE OR REPLACE FUNCTION get_customer_as_of(as_of_date DATE)
RETURNS TABLE (
    customer_id VARCHAR(50),
    customer_name VARCHAR(100),
    email VARCHAR(100)
)
AS
$
SELECT 
    h.customer_id,
    s.customer_name,
    s.email
FROM hub_customer h
JOIN (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY customer_hk 
               ORDER BY load_date DESC
           ) as rn
    FROM sat_customer_details
    WHERE load_date <= as_of_date
) s ON h.customer_hk = s.customer_hk
WHERE s.rn = 1;
$;

-- Usage: SELECT * FROM get_customer_as_of('2024-02-20');
-- Shows customer data as it was on February 20th
```

### Why This Pattern is Powerful

#### 1. **Time Travel Queries**
```sql
-- Compare customer data between two dates
WITH customer_jan AS (
    SELECT * FROM get_customer_as_of('2024-01-31')
),
customer_mar AS (
    SELECT * FROM get_customer_as_of('2024-03-31')  
)
SELECT 
    j.customer_id,
    j.email as email_january,
    m.email as email_march,
    CASE WHEN j.email != m.email THEN 'CHANGED' ELSE 'SAME' END as status
FROM customer_jan j
JOIN customer_mar m ON j.customer_id = m.customer_id;
```

#### 2. **Audit Trail Queries**
```sql
-- "Show me all changes to customer CUST001"
SELECT 
    h.customer_id,
    s.load_date as change_date,
    s.customer_name,
    s.email,
    s.record_source as changed_by_system,
    LAG(s.email) OVER (ORDER BY s.load_date) as previous_email
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
WHERE h.customer_id = 'CUST001'
ORDER BY s.load_date;

-- Result: Complete change history
-- customer_id | change_date         | email           | previous_email
-- CUST001     | 2024-01-01 10:00   | john@old.com    | NULL
-- CUST001     | 2024-02-15 14:30   | john@new.com    | john@old.com
-- CUST001     | 2024-04-22 16:45   | j.smith@new.com | john@new.com
```

#### 3. **Data Quality Monitoring**
```sql
-- "Find customers whose email changed more than 3 times"
SELECT 
    h.customer_id,
    COUNT(*) as number_of_email_changes
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
GROUP BY h.customer_id
HAVING COUNT(*) > 3;
```

### The Business Value

**Traditional Approach:**
```sql
-- Lost forever: What was the customer's email in February?
-- Lost forever: How many times has this customer changed their address?
-- Lost forever: Which system updated this customer last month?
```

**Data Vault Approach:**
```sql
-- Available forever: Complete timeline of every change
-- Available forever: Full audit trail with source system attribution
-- Available forever: Point-in-time reconstruction for any date
```

### Performance Considerations in Snowflake

```sql
-- Cluster satellites by the hub key and load_date for optimal performance
ALTER TABLE sat_customer_details CLUSTER BY (customer_hk, load_date);

-- Use Snowflake's automatic clustering
ALTER TABLE sat_customer_details SET AUTOMATIC_CLUSTERING = TRUE;

-- Create materialized views for frequently accessed "current" data
CREATE MATERIALIZED VIEW mv_current_customers AS
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.load_date
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
WHERE s.load_end_date IS NULL;
```

### Key Takeaways

✅ **Data Vault Raw Layer**: Insert-only preserves perfect history
✅ **Information Marts**: Present business-friendly current views  
✅ **Time Travel**: Query any point in time
✅ **Audit Trail**: Complete lineage always available
✅ **Performance**: Materialize current views for fast queries

### Data Flow Architecture

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           COMPLETE DATA VAULT ARCHITECTURE                   │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SOURCE SYSTEMS              STAGING               DATA VAULT LAYERS         │
│                                                                              │
│  ┌─────────────┐            ┌─────────────┐      ┌─────────────────────────┐ │
│  │    CRM      │            │             │      │     RAW DATA VAULT      │ │
│  │  System     │──────────► │   STAGING   │────► │                         │ │
│  │             │            │    AREA     │      │  ┌─────┐  ┌─────┐       │ │
│  └─────────────┘            │             │      │  │ HUB │  │ HUB │       │ │
│                             │             │      │  │CUST │  │ORDER│       │ │
│  ┌─────────────┐            │             │      │  └──┬──┘  └──┬──┘       │ │
│  │   E-COMMERCE│            │ - Data      │      │     │        │          │ │
│  │   Platform  │──────────► │   Cleaning  │      │     │   ┌────┴────┐     │ │
│  │             │            │ - Hash Key  │      │     │   │  LINK   │     │ │
│  └─────────────┘            │   Generation│      │     └───┤CUST_ORD ├─────┘ │
│                             │ - Validation│      │         └─────────┘       │ 
│  ┌─────────────┐            │             │      │              │            │ 
│  │   BILLING   │            │             │      │         ┌────▼────┐       │ 
│  │   System    │──────────► │             │      │         │   SAT   │       │ 
│  │             │            │             │      │         │  CUST   │       │ 
│  └─────────────┘            └─────────────┘      │         │ DETAILS │       │ 
│                                                  │         └─────────┘       │ 
│                                 │                └───────────────────────────┘ 
│                                 │                                            │
│                                 ▼                                            │
│                         ┌─────────────────┐      ┌────────────────────────────┐ 
│                         │ BUSINESS DATA   │      │    INFORMATION MARTS       │ 
│                         │     VAULT       │────► │                            │ 
│                         │                 │      │  ┌─────────────────────┐   │ 
│                         │ - Business Rules│      │  │   Customer 360      │   │ 
│                         │ - Calculations  │      │  │       View          │   │ 
│                         │ - Derived Keys  │      │  └─────────────────────┘   │ 
│                         │                 │      │                            │ 
│                         └─────────────────┘      │  ┌─────────────────────┐   │ 
│                                                  │  │   Order Analytics   │   │ 
│                                                  │  │      Mart           │   │ 
│                                                  │  └─────────────────────┘   │ 
│                                                  └────────────────────────────┘
│                                                                               │
└───────────────────────────────────────────────────────────────────────────────┘
```

### Why This Layered Approach?

**1. Raw Data Vault Layer**
- **Purpose**: Exact replica of source systems in Data Vault format
- **Benefit**: Preserves original data integrity
- **Rule**: No business logic, only structural transformation

**2. Business Data Vault Layer**  
- **Purpose**: Apply business rules and create calculated fields
- **Benefit**: Clean separation between raw data and business interpretation
- **Rule**: Additive only - never changes raw vault

**3. Information Marts Layer**
- **Purpose**: Optimized views for reporting and analytics
- **Benefit**: Fast query performance for end users
- **Rule**: Denormalized views that can be rebuilt from vault

## Data Vault vs Medallion Architecture: The Critical Differences

### What Medallion Architecture CAN Do

| Capability | Medallion Approach | Medallion Limitations |
|------------|-------------------|----------------------|
| **Historical Data** | SCD Type 2 in Silver/Gold layers | ❌ Requires manual design for each table |
| **Data Quality** | Progressive cleaning Bronze→Silver→Gold | ❌ Quality rules mixed with business logic |
| **Performance** | Optimized materialized views at Gold | ✅ **Better than Data Vault** |
| **Schema Changes** | Rebuild affected layers | ❌ Can break downstream dependencies |
| **Multi-Source** | Union and merge in Silver layer | ❌ Complex conflict resolution logic needed |

### What ONLY Data Vault Can Do

#### 1. **Automatic Historical Preservation Without Design Effort**

**Medallion Approach** (Manual effort for each entity):
```sql
-- You must manually design SCD Type 2 for EVERY table
CREATE TABLE silver_customer (
    customer_id VARCHAR(50),
    customer_name VARCHAR(100),
    email VARCHAR(100),
    effective_date TIMESTAMP,
    end_date TIMESTAMP,      -- You must remember to add this
    is_current BOOLEAN,      -- You must remember to add this
    hash_key VARCHAR(32)     -- You must remember to add this
);

-- Complex merge logic for EVERY table (repeated hundreds of times)
MERGE INTO silver_customer tgt
USING (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY updated_at DESC) as rn
    FROM bronze_customer
) src ON tgt.customer_id = src.customer_id AND tgt.is_current = TRUE
WHEN MATCHED AND src.hash_key != tgt.hash_key THEN
    UPDATE SET end_date = CURRENT_TIMESTAMP(), is_current = FALSE
WHEN NOT MATCHED THEN
    INSERT (customer_id, customer_name, email, effective_date, is_current, hash_key)
    VALUES (src.customer_id, src.customer_name, src.email, CURRENT_TIMESTAMP(), TRUE, src.hash_key);
-- Repeat this complex logic for every single table!
```

**Data Vault Approach** (Automatic for ALL entities):
```sql
-- ONE satellite loading procedure works for ALL satellites
CREATE OR REPLACE PROCEDURE load_any_satellite(
    satellite_name STRING,
    hub_key STRING,
    source_table STRING
)
-- This same procedure handles historical preservation for ALL entities automatically
-- No need to design SCD logic for each table - it's built into the architecture
```

#### 2. **True Source System Independence**

**Problem**: Customer "CUST001" exists in CRM, Billing, and Support systems with different information.

**Medallion Approach** - Breaks down:
```sql
-- Silver layer: How do you handle conflicting data?
-- CRM says: name="John Smith", email="john@email.com"
-- Billing says: name="J. Smith", email="johnsmith@email.com"  
-- Support says: name="John Smith Jr.", email="john@email.com"

-- You're FORCED to choose ONE version or create complex resolution rules
CREATE TABLE silver_customer AS
SELECT 
    customer_id,
    CASE 
        WHEN crm.name IS NOT NULL THEN crm.name
        WHEN billing.name IS NOT NULL THEN billing.name  
        ELSE support.name
    END as customer_name  -- ❌ You've lost the other versions forever!
FROM bronze_crm_customer crm
FULL JOIN bronze_billing_customer billing USING (customer_id)
FULL JOIN bronze_support_customer support USING (customer_id);
```

**Data Vault Approach** - Preserves everything:
```sql
-- Each source system gets its own satellite - no data loss
CREATE TABLE sat_customer_crm (
    customer_hk VARCHAR(40),
    load_date TIMESTAMP,
    customer_name VARCHAR(100),    -- "John Smith"
    email VARCHAR(100)             -- "john@email.com"
);

CREATE TABLE sat_customer_billing (
    customer_hk VARCHAR(40), 
    load_date TIMESTAMP,
    customer_name VARCHAR(100),    -- "J. Smith"
    email VARCHAR(100)             -- "johnsmith@email.com"
);

CREATE TABLE sat_customer_support (
    customer_hk VARCHAR(40),
    load_date TIMESTAMP, 
    customer_name VARCHAR(100),    -- "John Smith Jr."
    email VARCHAR(100)             -- "john@email.com"
);

-- Business rules applied at Information Mart level
-- You can see ALL versions and decide which to use when
```

#### 3. **Regulatory Compliance and Auditability**

**Compliance Requirement**: "Show me exactly what data you had about customer X on date Y, and trace where it came from."

**Medallion Approach**:
```sql
-- ❌ If you didn't think to add audit columns, you're in trouble
-- ❌ If you overwrote data in Silver layer, it's gone forever
-- ❌ Lineage tracking is external and can be lost
SELECT * FROM silver_customer 
WHERE customer_id = 'CUST001' 
AND effective_date <= '2024-01-15'
AND (end_date > '2024-01-15' OR end_date IS NULL);
-- Result: Limited view, may not show source-specific versions
```

**Data Vault Approach**:
```sql
-- ✅ Complete audit trail built-in
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.load_date as "data_as_of",
    s.record_source as "came_from_system",
    s.hash_diff as "change_fingerprint"
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
WHERE h.customer_id = 'CUST001'
  AND s.load_date <= '2024-01-15'
  AND (s.load_end_date > '2024-01-15' OR s.load_end_date IS NULL);
-- Result: Exact data state with full lineage, always available
```

#### 4. **Schema Evolution Without Breaking Changes**

**Scenario**: Source system adds new field "customer_segment"

**Medallion Approach**:
```sql
-- ❌ Must modify Silver layer schema
ALTER TABLE silver_customer ADD COLUMN customer_segment VARCHAR(50);

-- ❌ Must update all transformation logic
-- ❌ May break existing Gold layer views
-- ❌ Downstream reports might fail
-- ❌ Requires coordinated deployment across all layers
```

**Data Vault Approach**:
```sql
-- ✅ Add new satellite - no existing structures change
CREATE TABLE sat_customer_segmentation (
    customer_hk VARCHAR(40),
    load_date TIMESTAMP,
    customer_segment VARCHAR(50),
    -- No impact on existing hubs, links, or other satellites
);

-- ✅ Existing views continue working
-- ✅ New functionality added independently
```

### The Medallion "Workaround" Problem

**You CAN make Medallion do some Data Vault things, but:**

1. **Manual Effort**: You must manually implement SCD Type 2 for every table
2. **Inconsistent Implementation**: Each developer implements it differently
3. **Maintenance Burden**: 100 tables = 100 different historical tracking implementations
4. **Error Prone**: Easy to forget audit fields or mess up the merge logic
5. **Performance**: Complex MERGE statements for every table load

### When Data Vault is ESSENTIAL (Medallion Cannot Deliver)

#### ✅ **Regulatory Industries**
- **Financial Services**: SOX compliance, risk management
- **Healthcare**: HIPAA audit trails, patient data lineage  
- **Pharmaceuticals**: FDA validation, clinical trial data integrity
- **Government**: Data governance, FOIA requests

#### ✅ **Complex Source Landscapes**
- **50+ source systems** with overlapping entities
- **Legacy systems** with inconsistent data formats
- **M&A scenarios** where you need to preserve data from acquired companies
- **Multi-tenant** SaaS platforms serving different industries

#### ✅ **Long-term Data Preservation**
- **10+ year** historical requirements
- **Forensic analysis** needs ("What did we know when?")
- **Data archaeology** (understanding how business processes evolved)

### When Medallion is SUFFICIENT

#### ✅ **Analytics-First Organizations**
- Primary goal is reporting and BI
- Data sources are well-controlled
- Schema changes are infrequent
- Performance is the top priority

#### ✅ **Simpler Environments**
- Less than 20 source systems
- Modern, well-designed source systems
- Tolerant of some data loss in exchange for simplicity

### The Bottom Line Decision Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                    DECISION FRAMEWORK                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Choose DATA VAULT when you need:                              │
│  ✅ Bulletproof compliance and auditability                    │
│  ✅ Source system independence                                 │
│  ✅ Automatic historical preservation                          │
│  ✅ Schema evolution without breaking changes                  │
│  ✅ Complex multi-source entity relationships                  │
│                                                                 │
│  Choose MEDALLION when you need:                               │
│  ✅ Faster development and simpler maintenance                 │
│  ✅ Better query performance for analytics                     │
│  ✅ Team familiarity with layered architectures               │
│  ✅ Primarily append-only data sources                         │
│                                                                 │
│  HYBRID APPROACH: Use Data Vault for core business entities    │
│  and Medallion for analytics-only datasets                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Benefits of Data Vault

### 1. Flexibility and Adaptability
- **Additive-Only**: New requirements met by adding tables, not modifying existing ones
- **Source System Independence**: Changes in one system don't affect others
- **Schema Evolution**: Business key changes handled gracefully

### 2. Auditability and Compliance
- **Complete History**: Every data change tracked with timestamps
- **Data Lineage**: Built-in traceability to source systems
- **Regulatory Compliance**: Meets strict audit requirements (SOX, GDPR, etc.)

### 3. Scalability
- **Parallel Loading**: Independent loading of hubs, links, and satellites
- **Horizontal Scaling**: Easy to distribute across multiple systems
- **Performance**: Insert-only operations optimize for high throughput

### 4. Data Quality and Consistency
- **Single Version of Truth**: Consistent business keys across systems
- **Duplicate Detection**: Natural deduplication through hub design
- **Error Isolation**: Bad data doesn't corrupt good data

## Limitations and Challenges

### 1. Complexity
- **Learning Curve**: Steep initial learning for teams
- **Modeling Complexity**: Requires deep understanding of business relationships
- **Development Time**: Initial setup more time-intensive than traditional approaches

### 2. Performance Considerations
- **Query Complexity**: Requires multiple joins for simple queries
- **Storage Overhead**: Normalized structure increases storage requirements
- **Reporting Performance**: May need denormalized views for BI tools

### 3. Tooling and Skills
- **Limited Tooling**: Fewer native tools compared to dimensional modeling
- **Specialized Skills**: Requires data vault-specific expertise
- **Documentation**: Extensive documentation needed for maintenance

### 4. Implementation Challenges
- **Business Key Management**: Requires stable, high-quality business keys
- **Hash Collisions**: Though rare, must be handled appropriately
- **Migration Complexity**: Challenging to migrate from existing architectures

## Snowflake Implementation Guide

### 1. Environment Setup

```sql
-- Database structure
CREATE DATABASE DV_ENTERPRISE;
CREATE SCHEMA DV_ENTERPRISE.RAW_VAULT;
CREATE SCHEMA DV_ENTERPRISE.BUSINESS_VAULT;
CREATE SCHEMA DV_ENTERPRISE.INFORMATION_MARTS;
CREATE SCHEMA DV_ENTERPRISE.STAGING;
```

### 2. Hash Key Generation

```sql
-- Create hash key generation function
CREATE OR REPLACE FUNCTION generate_hash_key(input_string STRING)
RETURNS STRING
LANGUAGE SQL
AS
$$
  SELECT UPPER(SHA1(UPPER(TRIM(input_string))))
$$;

-- Example usage
SELECT generate_hash_key('CUST001') as customer_hk;
```

### 3. Standard Table Templates

```sql
-- Hub template
CREATE OR REPLACE TABLE hub_template (
    [entity]_hk VARCHAR(40) PRIMARY KEY,
    [business_key] VARCHAR(100) NOT NULL,
    load_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    record_source VARCHAR(50) NOT NULL,
    UNIQUE([business_key])
);

-- Link template  
CREATE OR REPLACE TABLE link_template (
    [relationship]_hk VARCHAR(40) PRIMARY KEY,
    [parent1]_hk VARCHAR(40) NOT NULL,
    [parent2]_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    record_source VARCHAR(50) NOT NULL
);

-- Satellite template
CREATE OR REPLACE TABLE sat_template (
    [parent]_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    -- Descriptive attributes here
    PRIMARY KEY ([parent]_hk, load_date)
);
```

### 4. Loading Procedures

```sql
-- Hub loading procedure
CREATE OR REPLACE PROCEDURE load_hub_customer()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    INSERT INTO hub_customer (customer_hk, customer_id, load_date, record_source)
    SELECT DISTINCT
        generate_hash_key(customer_id) as customer_hk,
        customer_id,
        CURRENT_TIMESTAMP(),
        'SOURCE_SYSTEM_A'
    FROM staging.customer_data s
    WHERE NOT EXISTS (
        SELECT 1 FROM hub_customer h 
        WHERE h.customer_id = s.customer_id
    );
    
    RETURN 'Hub Customer loaded successfully';
END;
$$;
```

### 5. Satellite Loading with History

```sql
CREATE OR REPLACE PROCEDURE load_sat_customer_details()
RETURNS STRING
LANGUAGE SQL
AS
$$
BEGIN
    -- Close existing records
    UPDATE sat_customer_details
    SET load_end_date = CURRENT_TIMESTAMP()
    WHERE load_end_date IS NULL
    AND customer_hk IN (
        SELECT DISTINCT h.customer_hk
        FROM staging.customer_data s
        JOIN hub_customer h ON h.customer_id = s.customer_id
        JOIN sat_customer_details sat ON sat.customer_hk = h.customer_hk
        WHERE sat.load_end_date IS NULL
        AND sat.hash_diff != generate_hash_key(CONCAT(
            COALESCE(s.customer_name, ''), '|',
            COALESCE(s.email, ''), '|',
            COALESCE(s.phone, ''), '|',
            COALESCE(s.address, '')
        ))
    );
    
    -- Insert new/changed records
    INSERT INTO sat_customer_details (
        customer_hk, load_date, record_source, hash_diff,
        customer_name, email, phone, address
    )
    SELECT 
        h.customer_hk,
        CURRENT_TIMESTAMP(),
        'SOURCE_SYSTEM_A',
        generate_hash_key(CONCAT(
            COALESCE(s.customer_name, ''), '|',
            COALESCE(s.email, ''), '|',
            COALESCE(s.phone, ''), '|',
            COALESCE(s.address, '')
        )) as hash_diff,
        s.customer_name,
        s.email,
        s.phone,
        s.address
    FROM staging.customer_data s
    JOIN hub_customer h ON h.customer_id = s.customer_id
    WHERE NOT EXISTS (
        SELECT 1 FROM sat_customer_details sat 
        WHERE sat.customer_hk = h.customer_hk 
        AND sat.load_end_date IS NULL
        AND sat.hash_diff = generate_hash_key(CONCAT(
            COALESCE(s.customer_name, ''), '|',
            COALESCE(s.email, ''), '|',
            COALESCE(s.phone, ''), '|',
            COALESCE(s.address, '')
        ))
    );
    
    RETURN 'Satellite Customer Details loaded successfully';
END;
$$;
```

### 6. Information Mart Views

```sql
-- Current customer view
CREATE OR REPLACE VIEW mart_current_customers AS
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.phone,
    s.address,
    s.load_date as last_updated
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
WHERE s.load_end_date IS NULL;

-- Historical customer changes
CREATE OR REPLACE VIEW mart_customer_history AS
SELECT 
    h.customer_id,
    s.customer_name,
    s.email,
    s.phone,
    s.address,
    s.load_date as valid_from,
    COALESCE(s.load_end_date, '9999-12-31'::timestamp) as valid_to,
    s.record_source
FROM hub_customer h
JOIN sat_customer_details s ON h.customer_hk = s.customer_hk
ORDER BY h.customer_id, s.load_date;
```

## Best Practices

### 1. Naming Conventions
- **Hubs**: `hub_[entity]` (e.g., `hub_customer`)
- **Links**: `link_[entity1]_[entity2]` (e.g., `link_customer_order`)
- **Satellites**: `sat_[parent]_[context]` (e.g., `sat_customer_details`)
- **Hash Keys**: `[entity]_hk` (e.g., `customer_hk`)

### 2. Hash Key Strategy
- Use SHA-1 for hash keys (40 characters)
- Always uppercase and trim input before hashing
- Include all business key components for composite keys
- Handle NULL values consistently

### 3. Loading Patterns
- Load hubs first, then links, then satellites
- Use staging areas for data preparation
- Implement idempotent loading procedures
- Monitor for hash collisions

### 4. Performance Optimization
```sql
-- Clustering keys for better performance
ALTER TABLE hub_customer CLUSTER BY (customer_id);
ALTER TABLE sat_customer_details CLUSTER BY (customer_hk, load_date);

-- Automatic clustering
ALTER TABLE hub_customer SET AUTOMATIC_CLUSTERING = TRUE;
```

### 5. Data Quality Checks
```sql
-- Orphaned satellite records check
SELECT COUNT(*) as orphaned_records
FROM sat_customer_details s
LEFT JOIN hub_customer h ON s.customer_hk = h.customer_hk
WHERE h.customer_hk IS NULL;

-- Duplicate hash key check
SELECT hash_key, COUNT(*) as duplicate_count
FROM (
    SELECT customer_hk as hash_key FROM hub_customer
    UNION ALL
    SELECT order_hk as hash_key FROM hub_order
) 
GROUP BY hash_key 
HAVING COUNT(*) > 1;
```

## Sample Implementation

### Complete Customer-Order Example

```sql
-- 1. Create Hubs
CREATE TABLE hub_customer (
    customer_hk VARCHAR(40) PRIMARY KEY,
    customer_id VARCHAR(50) NOT NULL UNIQUE,
    load_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    record_source VARCHAR(50) NOT NULL
);

CREATE TABLE hub_order (
    order_hk VARCHAR(40) PRIMARY KEY,
    order_id VARCHAR(50) NOT NULL UNIQUE,
    load_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    record_source VARCHAR(50) NOT NULL
);

-- 2. Create Link
CREATE TABLE link_customer_order (
    customer_order_hk VARCHAR(40) PRIMARY KEY,
    customer_hk VARCHAR(40) NOT NULL,
    order_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ DEFAULT CURRENT_TIMESTAMP(),
    record_source VARCHAR(50) NOT NULL,
    FOREIGN KEY (customer_hk) REFERENCES hub_customer(customer_hk),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);

-- 3. Create Satellites
CREATE TABLE sat_customer_details (
    customer_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    phone VARCHAR(20),
    PRIMARY KEY (customer_hk, load_date),
    FOREIGN KEY (customer_hk) REFERENCES hub_customer(customer_hk)
);

CREATE TABLE sat_order_details (
    order_hk VARCHAR(40) NOT NULL,
    load_date TIMESTAMP_NTZ NOT NULL,
    load_end_date TIMESTAMP_NTZ,
    record_source VARCHAR(50) NOT NULL,
    hash_diff VARCHAR(40) NOT NULL,
    order_amount DECIMAL(10,2),
    order_status VARCHAR(50),
    order_date DATE,
    PRIMARY KEY (order_hk, load_date),
    FOREIGN KEY (order_hk) REFERENCES hub_order(order_hk)
);

-- 4. Create Information Mart
CREATE OR REPLACE VIEW mart_customer_orders AS
SELECT 
    h_c.customer_id,
    s_c.customer_name,
    s_c.email,
    h_o.order_id,
    s_o.order_amount,
    s_o.order_status,
    s_o.order_date
FROM hub_customer h_c
JOIN link_customer_order l ON h_c.customer_hk = l.customer_hk
JOIN hub_order h_o ON l.order_hk = h_o.order_hk
JOIN sat_customer_details s_c ON h_c.customer_hk = s_c.customer_hk
JOIN sat_order_details s_o ON h_o.order_hk = s_o.order_hk
WHERE s_c.load_end_date IS NULL 
AND s_o.load_end_date IS NULL;
```

## Conclusion

Data Vault provides a robust, auditable, and flexible approach to enterprise data warehousing, particularly well-suited for complex regulatory environments and multi-source system landscapes. While it requires significant upfront investment in modeling and development, it offers unparalleled historical tracking, data lineage, and adaptability to changing business requirements.

For Snowflake implementations, leverage the platform's columnar storage, automatic scaling, and time travel features to maximize Data Vault benefits while mitigating some traditional performance concerns. Success depends on proper planning, consistent standards, and team expertise in the methodology.
