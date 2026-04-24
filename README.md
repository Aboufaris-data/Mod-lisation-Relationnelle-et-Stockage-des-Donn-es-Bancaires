# 🏦 Modélisation Relationnelle et Stockage des Données Bancaires

A relational data modeling and storage project for banking transactions using **PostgreSQL** and **Python (pandas + SQLAlchemy)**.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Database Schema](#database-schema)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Data Pipeline](#data-pipeline)
- [SQL Views & KPIs](#sql-views--kpis)
- [Analytical Queries](#analytical-queries)
- [Integrity Checks](#integrity-checks)
- [Technologies](#technologies)

---

## Overview

This project implements a normalized relational banking database that:

- Stores clients, transactions, products, agencies, and customer segments
- Loads data from a cleaned CSV (`financecore_clean.csv`) into PostgreSQL
- Creates SQL views for KPI reporting and anomaly detection
- Validates referential integrity with orphan checks
- Supports analytical queries for risk assessment and monthly reporting

---

## Database Schema

The database follows a **star-schema** pattern centered around the `transactions` table:

```
segment ──────┐
              │
              ▼
client ────► transactions ◄──── produit
                  │
                  ▼
               agence
```

### Tables

| Table          | Primary Key       | Description                              |
|----------------|-------------------|------------------------------------------|
| `segment`      | `segment_id`      | Client segment labels (e.g., Premium)    |
| `client`       | `client_id`       | Clients with credit scores & segment FK  |
| `agence`       | `agence_id`       | Bank branches                            |
| `produit`      | `produit_id`      | Financial products with category         |
| `transactions` | `transaction_id`  | Core fact table — all banking operations |

### `transactions` Columns

| Column             | Type            | Description                          |
|--------------------|-----------------|--------------------------------------|
| `transaction_id`   | VARCHAR(60) PK  | Unique transaction identifier        |
| `client_id`        | VARCHAR(60) FK  | References `client`                  |
| `produit_id`       | INT FK          | References `produit`                 |
| `agence_id`        | INT FK          | References `agence`                  |
| `date_transaction` | VARCHAR(60)     | Transaction date                     |
| `montant`          | DECIMAL(12,2)   | Transaction amount                   |
| `devise`           | VARCHAR(10)     | Currency                             |
| `taux_change_eur`  | DECIMAL(10,4)   | Exchange rate to EUR                 |
| `type_operation`   | VARCHAR(60)     | Operation type                       |
| `statut`           | VARCHAR(30)     | Transaction status                   |
| `solde_avant`      | DECIMAL(12,2)   | Balance before transaction           |
| `is_anomalie`      | VARCHAR(60)     | Anomaly flag                         |

---

## Project Structure

```
├── data/
│   └── financecore_clean.csv         # Cleaned source data
├── financiere_bank_modelesitation.ipynb  # Main notebook
├── .env                              # DB credentials (not committed)
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- PostgreSQL running locally or remotely
- Required Python packages:

```bash
pip install pandas sqlalchemy psycopg2-binary python-dotenv
```

### Environment Variables

Create a `.env` file at the root of the project:

```env
DB_USER=your_user
DB_PASS=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database
```

> ⚠️ Never commit your `.env` file. Add it to `.gitignore`.

### Run the Notebook

```bash
jupyter notebook financiere_bank_modelesitation.ipynb
```

Execute cells in order — the notebook handles table creation, data loading, and view creation sequentially.

---

## Data Pipeline

The ingestion pipeline follows these steps:

1. **Connect** to PostgreSQL using SQLAlchemy + `.env` credentials
2. **Create tables** (`segment`, `client`, `agence`, `produit`, `transactions`) with foreign key constraints
3. **Load CSV** into a pandas DataFrame
4. **Insert dimension tables** first (segment → client → agence + produit)
5. **Merge IDs** back into the DataFrame to resolve foreign keys
6. **Insert fact table** (`transactions`) with all FK references resolved
7. **Create indexes** on `client_id`, `date_transaction`, `agence_id` for query performance

---

## SQL Views & KPIs

Three analytical views are created automatically:

### `jointure_table`
Full denormalized join across all tables — useful as a base for reporting.

### `kpi_total_transactions`
```sql
SELECT
    COUNT(*)        AS nb_transactions,
    SUM(montant)    AS total_montant,
    AVG(montant)    AS moyenne_montant
FROM transactions;
```

### `kpi_risk_segment`
Default rate per client segment, based on credit score threshold of 500.

### `kpi_transactions_mensuelles`
Monthly transaction volume and total amounts, grouped by truncated date.

---

## Analytical Queries

### Monthly Aggregation by Agency & Product
Groups transactions by agency, product, and month — filtered to totals above 1,000.

### Below-Average Credit Score Clients
Returns all clients whose credit score falls below the overall average.

### Default Rate by Segment
Counts clients with `score_credit_client < 500` per segment and computes a default rate ratio.

### Full Transaction Join
Left joins all dimension tables onto transactions to produce a fully enriched view.

---

## Integrity Checks

After loading, orphan checks are run to validate referential integrity:

| Check              | Query Logic                                      |
|--------------------|--------------------------------------------------|
| `orphan_client`    | Clients with no associated transactions          |
| `orphan_segment`   | Segments with no associated clients              |
| `orphan_produit`   | Products with no associated transactions         |
| `orphan_agence`    | Agencies with no associated transactions         |

All counts should return `0` for a clean load.

---

## Technologies

| Tool            | Purpose                          |
|-----------------|----------------------------------|
| Python 3        | Data processing & orchestration  |
| pandas          | DataFrame manipulation           |
| SQLAlchemy      | ORM & DB connection              |
| PostgreSQL      | Relational database engine       |
| python-dotenv   | Secure credential management     |
| Jupyter         | Interactive development          |

---

## License

This project is for educational and portfolio purposes.
