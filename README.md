<img width="1920" height="1016" alt="2026-10-02_12-27-02" src="https://github.com/user-attachments/assets/2beb5c56-7e3a-4628-8d48-533a9b3b8739" />

# MD Water Services Data Audit & Quality Control

## Summary
This repository contains the complete SQL data auditing, cleanup, and quality assurance pipeline for the Maji Ndogo water services database (`md_water_services`). The primary objective of this project is to audit qualitative survey records, resolve human data entry errors, align descriptive logs with lab-tested biological metrics, and correct misclassified public water sources to support reliable public health reporting.

---

## Repository Structure

The project SQL code is organized across two primary scripts:

| File Name | Description |
| :--- | :--- |
| **`01_db_setup_and_inspection.sql`** | Initial database schema discovery, table verification, and preliminary previews (Task 1). |
| **`02_water_pollution_audit_and_cleaning.sql`** | Exploratory queries, queue time tracking, anomaly detection, pattern matching, and safe batch updates (Tasks 2–9). |

---

## Technical Workflow & Project Tasks

### Task 1: Initial Database Inspection, Table Discovery, and Data Preview
**Objective:** Verify database connectivity, validate table structures, and establish baseline previews across core tables in `md_water_services`.

```sql
SHOW TABLES;

SELECT * FROM data_dictionary LIMIT 5;
SELECT * FROM employee LIMIT 5;
SELECT * FROM global_water_access LIMIT 5;
SELECT * FROM location LIMIT 5;
SELECT * FROM visits LIMIT 5;
SELECT * FROM water_quality LIMIT 5;
SELECT * FROM water_source LIMIT 5;
SELECT * FROM well_pollution LIMIT 5;
```
### Tasks 2 – 6: Exploratory Data Analysis & Schema Audit
**Objective:** Evaluate queue times, water source distributions, and cross-examine qualitative field survey labels against quantitative laboratory contamination measurements.
```sql
-- Task 2: Find all unique water source types
SELECT DISTINCT 
    type_of_water_source 
FROM 
    water_source;

-- Task 3: Identify Severe Queue Times (> 500 minutes / ~8.3 hours)
SELECT 
    * 
FROM 
    visits
WHERE 
    time_in_queue > 500;

-- Task 4: Look up Water Source Types for High Queue Time Sources
SELECT
    *
FROM
    water_source
WHERE 
    source_id IN ('AkKi00881224', 'SoRu37635224', 'SoRu36096224', 'AkRu05234224', 'HaZa21742224');

-- Task 5: Audit Water Quality for Second Visits
SELECT
    *
FROM 
    water_quality
WHERE
    subjective_quality_score = 10 
    AND visit_count = 2;

-- Task 6: Inspect Well Pollution Laboratory Records
SELECT
    *
FROM 
    well_pollution 
LIMIT 5;
```
### Task 7: Detecting Contamination Anomalies
**Objective:** Identify water sources mislabeled as 'Clean' in survey results despite exceeding the $0.01\text{ CFU/mL}$ biological contamination threshold.
```sql
SELECT *
FROM well_pollution
WHERE results = 'Clean' 
  AND biological > 0.01;
```
### Task 8: Identifying Incorrect Descriptions with Trailing Text
**Objective:** Isolate records where field surveyors erroneously prepended "Clean " to descriptive notes detailing active biological contaminants.
```sql
SELECT 
    *
FROM 
    well_pollution
WHERE 
    description LIKE 'Clean %'
    AND biological > 0;
```
### Task 9: Cleaning Description Strings and Updating Contaminated Survey Results
**Objective:** Perform safe batch updates of mislabeled records.
```sql
-- Disable safe updates for this session
SET SQL_SAFE_UPDATES = 0;

-- Fix E. coli descriptions
UPDATE well_pollution
SET 
    description = 'Bacteria: E. coli'
WHERE 
    description = 'clean Bacteria: E. coli';

-- Fix Giardia Lamblia descriptions
UPDATE well_pollution
SET 
    description = 'Bacteria: Giardia Lamblia'
WHERE 
    description = 'clean Bacteria: Giardia Lamblia';

-- Update survey results status for all wells with biological contamination exceeding 0.01 CFU/mL
UPDATE 
    well_pollution
SET 
    results = 'Contaminated: Biological'
WHERE 
    results = 'Clean'
    AND biological > 0.01;

-- Re-enable safe updates
SET SQL_SAFE_UPDATES = 1;
```
### Tools & Technologies Used

* **Database Management System:** MySQL Server 8.0 & MySQL Workbench
* **Language & Syntax:** Structured Query Language (SQL — DQL & DML)
* **Data Quality Techniques:** Data Auditing, Pattern Matching (`LIKE`), String Cleaning, Metric Thresholding (`biological > 0.01`), and Safe Updates Control (`SQL_SAFE_UPDATES`)

---

### Background & Context

This project was completed as part of the ALX Africa Data Science Program to demonstrate practical relational database auditing, exploratory data analysis (DQL), record remediation (DML), and data quality control using MySQL Workbench.
