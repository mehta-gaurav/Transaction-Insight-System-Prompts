# Transaction Insight System – Database Design

## 📌 Objective

Design an **optimized relational database schema** for the Transaction Insight System backend. This schema will support:

- High-volume **financial transactions**
- **Dynamic filtering** on hundreds of parameters
- Efficient **Transaction Selection Sets** retrieval
- Scalable **insights** and related data views

---

## 📂 Database Overview

| **Component**        | **Details**                                                                |
|----------------------|----------------------------------------------------------------------------|
| **Database Type**    | Relational Database (PostgreSQL recommended)                               |
| **Workload Focus**   | OLTP (high-speed transaction processing) + OLAP (analytical querying)      |
| **Scale Strategy**   | Partitioning, indexing, and denormalization for specific use cases         |

---

## 🗂️ Core Entities

### 1. Transactions

Captures the core financial transaction data.

| **Field Name**        | **Data Type**   | **Description**                                          |
|-----------------------|-----------------|----------------------------------------------------------|
| `transaction_id`      | BIGINT (PK)     | Unique identifier for each transaction                   |
| `source_account`      | VARCHAR(64)     | Source account identifier                                |
| `destination_account` | VARCHAR(64)     | Destination account identifier                           |
| `amount`              | NUMERIC(18,4)   | Transaction amount                                       |
| `currency`            | CHAR(3)         | ISO currency code                                        |
| `transaction_type`    | VARCHAR(32)     | Type of transaction (e.g., transfer, payment)            |
| `status`              | VARCHAR(16)     | Status (e.g., completed, pending, failed)                |
| `transaction_date`    | TIMESTAMP       | Date and time of the transaction                         |
| `reference_id`        | VARCHAR(64)     | External reference or correlation ID                     |
| `created_at`          | TIMESTAMP       | Record creation timestamp                                |
| `updated_at`          | TIMESTAMP       | Record last updated timestamp                            |

---

### 2. Transaction Details

Stores additional dynamic attributes or metadata for transactions (semi-structured).

| **Field Name**       | **Data Type**   | **Description**                                          |
|----------------------|-----------------|----------------------------------------------------------|
| `detail_id`          | BIGINT (PK)     | Unique identifier for the detail entry                   |
| `transaction_id`     | BIGINT (FK)     | Linked to `transactions.transaction_id`                  |
| `attribute_name`     | VARCHAR(64)     | Name of the attribute                                    |
| `attribute_value`    | VARCHAR(256)    | Value of the attribute                                   |

> **Note:** Allows storage of dynamic key-value pairs to support flexible filtering on hundreds of fields.

---

### 3. Transaction Selection Sets

Defines user-created selection sets for reusable transaction queries.

| **Field Name**          | **Data Type**   | **Description**                                         |
|-------------------------|-----------------|---------------------------------------------------------|
| `selection_set_id`      | BIGINT (PK)     | Unique ID for the selection set                         |
| `name`                  | VARCHAR(128)    | User-defined name for the set                           |
| `description`           | TEXT            | Description of the selection set                        |
| `filter_criteria_json`  | JSONB           | Filter criteria saved in JSON format                    |
| `created_by_user_id`    | BIGINT          | User who created the selection set                      |
| `created_at`            | TIMESTAMP       | Record creation timestamp                               |

---

### 4. Users

Users who interact with the system.

| **Field Name**   | **Data Type**   | **Description**                        |
|------------------|-----------------|----------------------------------------|
| `user_id`        | BIGINT (PK)     | Unique identifier for the user         |
| `username`       | VARCHAR(64)     | Unique username                        |
| `email`          | VARCHAR(128)    | Email address                          |
| `role`           | VARCHAR(32)     | Role (e.g., admin, viewer)             |
| `created_at`     | TIMESTAMP       | User account creation date             |
| `updated_at`     | TIMESTAMP       | Last update timestamp                  |

---

### 5. Transaction Insights

Stores analytical data derived from transactions (aggregates, trends, flags).

| **Field Name**    | **Data Type**   | **Description**                                          |
|-------------------|-----------------|----------------------------------------------------------|
| `insight_id`      | BIGINT (PK)     | Unique identifier for the insight record                 |
| `transaction_id`  | BIGINT (FK)     | Linked transaction                                       |
| `insight_type`    | VARCHAR(64)     | Insight type (e.g., risk_score, anomaly_flag)            |
| `insight_value`   | VARCHAR(256)    | Insight value (e.g., high, low, score number)            |
| `created_at`      | TIMESTAMP       | Record creation timestamp                                |

---

## 🔑 Primary & Foreign Keys

- `transaction_id` as PK in `Transactions`
- `transaction_id` as FK in `Transaction Details` and `Transaction Insights`
- `selection_set_id` as PK in `Transaction Selection Sets`
- `user_id` as PK in `Users` and FK in `Transaction Selection Sets`

---

## ⚙️ Indexes & Query Optimization

| **Table**                  | **Index Type**  | **Index Fields**                              | **Purpose**                               |
|----------------------------|-----------------|-----------------------------------------------|-------------------------------------------|
| **Transactions**           | BTREE           | `transaction_date`                            | Optimize date range queries               |
|                            | BTREE           | `source_account`, `destination_account`       | Optimize account-based filtering          |
|                            | BTREE           | `status`                                      | Speed up status queries                   |
| **Transaction Details**    | BTREE           | `attribute_name`, `attribute_value`           | Support attribute-based filtering         |
|                            | HASH (Optional) | `transaction_id`                              | Fast lookup on parent transaction ID      |
| **Transaction Selection Sets** | GIN (JSONB) | `filter_criteria_json`                        | Efficient JSON querying                   |
| **Transaction Insights**   | BTREE           | `insight_type`, `insight_value`               | Filter insights by type/value             |

---

## 🚀 Partitioning / Sharding Strategy

### Transaction Table Partitioning

- **Partition By**: `transaction_date` (Time-based partitioning)
- **Partition Interval**: Monthly or quarterly partitions based on data volume
- **Benefits**:
  - Speeds up time-based queries
  - Improves insert performance
  - Eases data archival and maintenance

### Sharding Strategy (Optional for High Scale)

- **Shard Key**: `source_account` or `transaction_id` hash
- **Justification**: For extremely high volume, sharding across multiple database nodes ensures horizontal scalability.

---

## 📏 Normalization / Denormalization Decisions

| **Approach**         | **Justification**                                                                 |
|----------------------|-----------------------------------------------------------------------------------|
| **Normalization**    | Core entities (Transactions, Users) are normalized for data consistency.          |
| **Denormalization**  | `Transaction Details` allows flexible storage for high-cardinality attributes, supporting dynamic filters without schema changes. |
| **Pre-aggregated Insights** | Stored in `Transaction Insights` for fast retrieval in analytics views. |

---

## 🔗 Entity-Relationship Model (ER Model) Description

- **Transactions** is the central table, linked to:
  - **Transaction Details** (1:M relationship)
  - **Transaction Insights** (1:M relationship)
- **Transaction Selection Sets** references **Users**
- **Users** manage and own **Transaction Selection Sets**

> ER Diagram available upon request.

---

## ✅ Justifications for Design Choices

1. **Dynamic filtering** supported via `Transaction Details` key-value design.
2. **Efficient queries** achieved with indexing, partitioning, and JSONB for complex criteria.
3. **Scalability** via partitioning and potential sharding strategies.
4. **Flexibility** to adapt future attributes and filters without schema changes.
5. **Relational integrity** maintained via normalized core entities and foreign keys.

---

### ✅ Next Step

We will focus on building the backend services that interact with the database designed.
