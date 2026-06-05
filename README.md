<div align="center">

# 🎵 SQL Music Store Analysis

### *Answering real business questions for a digital music store using PostgreSQL*

<br>

[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org)
[![SQL](https://img.shields.io/badge/SQL-Advanced-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)](https://github.com/AmanManiTiwari/SQL-Music-Store-Analysis)
[![Difficulty](https://img.shields.io/badge/Queries-Easy%20→%20Advanced-orange?style=for-the-badge)](https://github.com/AmanManiTiwari/SQL-Music-Store-Analysis)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)](https://github.com/AmanManiTiwari/SQL-Music-Store-Analysis)

<br>

> **What does a real analyst actually do with a database?**
> They ask business questions - and then write the SQL to answer them precisely.
> This project does exactly that, across 11 progressively complex queries on a live music store schema.

</div>

---

## 📌 Overview

**SQL Music Store Analysis** is a business intelligence project built entirely in SQL on a relational music store database. It covers three tiers of analytical complexity - from basic aggregations to advanced window functions and recursive CTEs - producing actionable insights on sales performance, customer behaviour, artist popularity, and market penetration by country.

This project demonstrates not just SQL syntax, but the analyst's mindset: translating business problems into queries, and queries into decisions.

---

## 🗄️ Database Schema

The analysis runs on an 11-table relational schema covering the full music store lifecycle:

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   employee   │     │   customer   │     │    artist    │
│──────────────│     │──────────────│     │──────────────│
│ employee_id  │     │ customer_id  │     │  artist_id   │
│ first_name   │     │ first_name   │     │  name        │
│ last_name    │     │ last_name    │     └──────┬───────┘
│ title        │     │ email        │            │
│ levels       │     │ country      │     ┌──────▼───────┐
└──────────────┘     └──────┬───────┘     │    album     │
                            │             │──────────────│
                     ┌──────▼───────┐     │  album_id    │
                     │   invoice    │     │  title       │
                     │──────────────│     │  artist_id   │
                     │ invoice_id   │     └──────┬───────┘
                     │ customer_id  │            │
                     │ billing_city │     ┌──────▼───────┐
                     │ total        │     │    track     │
                     └──────┬───────┘     │──────────────│
                            │             │  track_id    │
                     ┌──────▼───────┐     │  name        │
                     │ invoice_line │     │  album_id    │
                     │──────────────│     │  genre_id    │
                     │ invoice_id   │     │  milliseconds│
                     │ track_id     ├─────│  unit_price  │
                     │ unit_price   │     └──────┬───────┘
                     │ quantity     │            │
                     └──────────────┘     ┌──────▼───────┐
                                          │    genre     │
                                          │──────────────│
                                          │  genre_id    │
                                          │  name        │
                                          └──────────────┘
```

![Database Schema](https://github.com/AmanManiTiwari/SQL-Music-Store-Analysis/raw/main/MusicDatabaseSchema.png)

---

## 🔍 Business Questions Answered

The queries are structured across three difficulty levels, each with increasing analytical complexity.

---

### 🟢 Level 1 - Easy: Operational Queries

| # | Business Question | SQL Concepts Used |
|---|-------------------|-------------------|
| Q1 | Who is the most senior employee? | `ORDER BY`, `LIMIT` |
| Q2 | Which countries generate the most invoices? | `GROUP BY`, `COUNT`, `ORDER BY` |
| Q3 | What are the top 3 highest invoice values? | `ORDER BY DESC`, `LIMIT` |
| Q4 | Which city should we host the music festival in? | `SUM`, `GROUP BY`, `ORDER BY` |
| Q5 | Who is our single best (highest-spending) customer? | `JOIN`, `SUM`, `GROUP BY` |

**Sample Query - Best City for a Music Festival:**
```sql
SELECT billing_city, SUM(total) AS InvoiceTotal
FROM invoice
GROUP BY billing_city
ORDER BY InvoiceTotal DESC
LIMIT 1;
```

---

### 🟡 Level 2 - Moderate: Customer & Genre Intelligence

| # | Business Question | SQL Concepts Used |
|---|-------------------|-------------------|
| Q1 | Who are all the Rock music listeners? (email list) | Multi-table `JOIN`, `DISTINCT`, Subquery |
| Q2 | Which are the top 10 rock bands by track count? | 4-table `JOIN`, `COUNT`, `GROUP BY` |
| Q3 | Which tracks are longer than the average song length? | Scalar Subquery, `AVG`, `WHERE` |

**Sample Query - Top 10 Rock Artists:**
```sql
SELECT artist.name, COUNT(artist.artist_id) AS number_of_songs
FROM track
JOIN album   ON album.album_id   = track.album_id
JOIN artist  ON artist.artist_id = album.artist_id
JOIN genre   ON genre.genre_id   = track.genre_id
WHERE genre.name LIKE 'Rock'
GROUP BY artist.artist_id
ORDER BY number_of_songs DESC
LIMIT 10;
```

---

### 🔴 Level 3 - Advanced: Window Functions, CTEs & Recursive Queries

| # | Business Question | SQL Concepts Used |
|---|-------------------|-------------------|
| Q1 | How much did each customer spend on the top-selling artist? | `WITH` CTE, 6-table `JOIN`, `SUM`, aggregation |
| Q2 | What is the most popular genre in each country? | `WITH` CTE + `ROW_NUMBER()` window function; also solved with **Recursive CTE** |
| Q3 | Who is the top-spending customer in each country? | `WITH` CTE + `ROW_NUMBER()` window function; also solved with **Recursive CTE** |

**Sample Query - Most Popular Genre Per Country (CTE + Window Function):**
```sql
WITH popular_genre AS (
    SELECT
        COUNT(invoice_line.quantity)  AS purchases,
        customer.country,
        genre.name,
        genre.genre_id,
        ROW_NUMBER() OVER(
            PARTITION BY customer.country
            ORDER BY COUNT(invoice_line.quantity) DESC
        ) AS RowNo
    FROM invoice_line
    JOIN invoice  ON invoice.invoice_id   = invoice_line.invoice_id
    JOIN customer ON customer.customer_id = invoice.customer_id
    JOIN track    ON track.track_id       = invoice_line.track_id
    JOIN genre    ON genre.genre_id       = track.genre_id
    GROUP BY customer.country, genre.name, genre.genre_id
)
SELECT * FROM popular_genre WHERE RowNo <= 1;
```

---

## 💡 SQL Techniques Demonstrated

| Technique | Used In |
|-----------|---------|
| `GROUP BY` + `HAVING` | Sales aggregation, customer ranking |
| Multi-table `JOIN` (up to 6 tables) | Customer–invoice–track–artist chains |
| Correlated & scalar **subqueries** | Track length filtering, top-N selection |
| **Common Table Expressions (CTEs)** | Advanced Q1, Q2, Q3 |
| **Window functions** - `ROW_NUMBER() OVER (PARTITION BY ...)` | Top genre/customer per country |
| **Recursive CTEs** | Alternative solutions for Q2 & Q3 |
| `DISTINCT` for deduplication | Email list generation |
| Aggregate functions - `SUM`, `COUNT`, `AVG`, `MAX` | Throughout all levels |

---

## 📁 Repository Structure

```
SQL-Music-Store-Analysis/
│
├── 🗃️  Music_Store_database.sql     ← Full database: schema + seed data
├── 🔍  Music_Store_Query.sql        ← All 11 analytical queries (3 difficulty levels)
├── 🖼️  MusicDatabaseSchema.png      ← Visual ER diagram of all 11 tables
└── 📄  README.md                    ← You are here
```

---

## 🚀 Getting Started

### Prerequisites
- **PostgreSQL** 13+ (recommended) - or any SQL-compatible RDBMS

### 1. Clone the Repository
```bash
git clone https://github.com/AmanManiTiwari/SQL-Music-Store-Analysis.git
cd SQL-Music-Store-Analysis
```

### 2. Create the Database
```sql
-- In psql terminal
CREATE DATABASE music_store;
\c music_store
```

### 3. Load Schema & Data
```bash
psql -U postgres -d music_store -f Music_Store_database.sql
```

### 4. Run the Queries
```bash
psql -U postgres -d music_store -f Music_Store_Query.sql
```
Or open `Music_Store_Query.sql` in **pgAdmin**, **DBeaver**, or any SQL IDE and run queries section by section.

---

## 📈 Key Business Insights Unlocked

> 🏙️ **Best city for a music festival** - identified by aggregating total invoice revenue per billing city

> 👑 **Best customer** - found by joining `customer` and `invoice` tables and ranking by total spend

> 🎸 **Top 10 rock bands** - ranked by track catalogue size via a 4-table join chain

> 🌍 **Most popular genre per country** - solved two ways: using `ROW_NUMBER()` windowing and a recursive CTE, demonstrating SQL flexibility

> 💰 **Top spender per country** - window function partitioned by country delivers country-level customer leaderboards in a single query

---

## 🛠️ Tools & Environment

| Tool | Purpose |
|------|---------|
| **PostgreSQL** | Primary database engine |
| **pgAdmin / DBeaver** | Query execution & result visualization |
| **SQL** | All analysis - no Python or BI tools |

---

## 💼 Real-World Relevance

The skills exercised here directly map to day-to-day analyst work at companies like:

- **Spotify / Apple Music** - genre popularity by region, artist performance analytics
- **Retail & e-commerce** - top customer identification, city-level revenue analysis
- **Any data team** - writing clean, performant SQL that answers stakeholder questions without ambiguity

---

## 🙋 About the Author

**Aman Mani Tiwari** - Data analyst building a portfolio that spans SQL, Excel dashboards, and machine learning.

[![GitHub](https://img.shields.io/badge/GitHub-AmanManiTiwari-181717?style=flat-square&logo=github)](https://github.com/AmanManiTiwari)

---

<div align="center">

*If this project was useful, a ⭐ helps others find it - thank you!*

</div>
