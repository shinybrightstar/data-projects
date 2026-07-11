---
title: SQL Data Warehouse
tagline: Medallion-architecture warehouse built from raw CSVs to analytics-ready star schema.
flow: Raw CSVs -> Bronze -> Silver -> Gold -> Reporting
stack: [SQL Server, T-SQL, Medallion Architecture, ETL]
repo: https://github.com/shinybrightstar/data-projects
---

## Overview

_Replace this with your project summary: what data source, what problem, what the warehouse enables._

## Architecture

Bronze → Silver → Gold layers, following medallion architecture: raw ingestion, cleaned/conformed data,
then business-ready aggregates.

## Key queries

```sql
-- Paste a representative query here, e.g. your gold-layer aggregation
```

## What I'd do differently

_A short, honest note — this is what makes a portfolio project read as real rather than a tutorial copy._
