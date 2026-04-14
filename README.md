# 🗄️ PostgreSQL Financial Transactions Pipeline

A Python/SQL project that builds a relational PostgreSQL database from a financial transactions dataset, including data modeling, ingestion, analytical queries, and KPI views.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Database Schema](#database-schema)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Data Pipeline](#data-pipeline)
- [SQL Queries & Analytics](#sql-queries--analytics)
- [KPI Views](#kpi-views)
- [Technologies Used](#technologies-used)

---

## Overview

This project connects to a PostgreSQL database and:

1. Creates a normalized relational schema from a raw financial CSV file
2. Loads and segments data into dedicated tables (agencies, clients, products, transactions)
3. Runs analytical SQL queries for business insights
4. Exposes reusable KPI views for reporting and dashboarding

---

## Database Schema

The database is composed of four main tables:

```
agences               segments_client
──────────            ───────────────────
agence_id (PK)        segment_id (PK)
agence                segment_client
                      Categorie_risque

clients               produits
──────────            ──────────
client_id (PK)        produit_id (PK)
segment_client        produit
score_credit_client

transactions
────────────────────────────────────────
transaction_id (PK)
client_id (FK → clients)
date_transaction, montant, devise
taux_change_eur, montant_eur
categorie, produit, agence
type_operation, statut
score_credit_client, segment_client
solde_avant, anomaly_montant, anomaly_score
is_anomaly, Années, Mois, Trimestre
jour-semaine, montant_eur_verifie
Comparaison, Categorie_risque, taux_rejet
```

---

## Project Structure

```
├── PostgreSQL.ipynb        # Main notebook: setup, ingestion, queries, views
├── financecore_clean.csv   # Source dataset (not tracked)
├── .env                    # Database credentials (not tracked)
└── README.md
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- PostgreSQL server running locally or remotely
- The following Python packages:

```bash
pip install sqlalchemy psycopg2-binary pandas python-dotenv
```

### Environment Variables

Create a `.env` file at the root of the project:

```env
DB_USER=your_username
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
```

### Run the Notebook

```bash
jupyter notebook PostgreSQL.ipynb
```

Execute cells in order: connection → table creation → data loading → queries → views.

---

## Data Pipeline

The pipeline follows these steps:

1. **Connect** to PostgreSQL using SQLAlchemy + psycopg2
2. **Create tables** with proper constraints and foreign keys
3. **Load the CSV** with pandas (`financecore_clean.csv`)
4. **Split into sub-tables**: agencies, client segments, clients, products, and transactions
5. **Insert** each sub-table into its corresponding PostgreSQL table

---

## SQL Queries & Analytics

The notebook includes several analytical queries:

| Query | Description |
|---|---|
| Top transactions by agency & product | Grouped by agency, product, year, month — filtered on total > €1,000 |
| Below-average balance clients | Clients whose `solde_avant` is below the overall average |
| Default rate by risk category | Counts rejected/refused/defaulted transactions per `Categorie_risque` |
| Full joined transaction view | Joins transactions with clients, segments, agencies, and products |

---

## KPI Views

Three SQL views are created for easy reuse in BI tools or dashboards:

### `vw_kpi_transactions`
Aggregates transaction counts, total and average EUR amounts by agency, product, year, and month.

### `vw_kpi_taux_defaut`
Calculates the default rate (%) per risk category using filtered status values (`Rejeté`, `Refusé`, `Défaut`).

### `vw_kpi_performance_agence`
Summarizes per-agency performance: transaction count, total amount, average amount, and unique client count.

---

## Technologies Used

- **Python** — data processing and orchestration
- **pandas** — CSV loading and DataFrame manipulation
- **SQLAlchemy** — database engine and ORM-free SQL execution
- **psycopg2** — PostgreSQL adapter
- **PostgreSQL** — relational database
- **Jupyter Notebook** — interactive development environment
- **python-dotenv** — secure credential management

---

## ⚠️ Notes

- The `.env` file and `financecore_clean.csv` are not included in the repository for security and size reasons.
- The `clients` table is dropped and recreated separately to avoid foreign key conflicts during initial load.
- Column names with special characters (e.g., `Années`, `jour-semaine`) are quoted in SQL using double quotes.
