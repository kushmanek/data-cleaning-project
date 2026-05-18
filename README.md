SQL Data Cleaning Project – Layoffs Dataset
Project Overview

This project focuses on cleaning and preparing a layoffs dataset using SQL in MySQL.
The main objective was to transform raw company layoff data into a clean and analysis-ready dataset by handling duplicates, standardizing values, fixing formatting issues, converting data types, and removing incomplete records.

The project demonstrates practical SQL skills commonly used in real-world data analyst and data engineering workflows.

Dataset

The dataset contains company layoff information, including:

Company name
Location
Industry
Total employees laid off
Percentage laid off
Date
Company stage
Country
Funds raised

Main table used:

layoffs
Project Workflow
1. Creating a Staging Table

A staging table was created to preserve the original raw dataset.

CREATE TABLE layoffs_staging LIKE layoffs;

INSERT layoffs_staging
SELECT * FROM layoffs;

Purpose:

Protect original data
Perform transformations safely
Enable repeatable cleaning processes
2. Identifying Duplicate Records

Duplicates were detected using the ROW_NUMBER() window function.

SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry,
total_laid_off, percentage_laid_off,
date, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;

A Common Table Expression (CTE) was used to isolate duplicates:

WITH duplicate_cte AS (
    SELECT *,
    ROW_NUMBER() OVER(
        PARTITION BY company, location, industry,
        total_laid_off, percentage_laid_off,
        date, stage, country, funds_raised_millions
    ) AS row_num
    FROM layoffs_staging
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
3. Creating a Clean Staging Table

A second staging table was created to store cleaned records.

CREATE TABLE layoffs_staging2 (
    company TEXT,
    location TEXT,
    industry TEXT,
    total_laid_off INT DEFAULT NULL,
    percentage_laid_off TEXT,
    date TEXT,
    stage TEXT,
    country TEXT,
    funds_raised_millions INT DEFAULT NULL,
    row_num INT
);

Data with row numbers was inserted:

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry,
total_laid_off, percentage_laid_off,
date, stage, country, funds_raised_millions
) AS row_num
FROM layoffs_staging;
4. Removing Duplicate Records

Duplicate rows were deleted using:

DELETE
FROM layoffs_staging2
WHERE row_num > 1;
5. Standardizing Data
Trimming Company Names
UPDATE layoffs_staging2
SET company = TRIM(company);

Example:

" Airbnb " → "Airbnb"
Standardizing Industry Names
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

Example:

"Crypto Currency" → "Crypto"
Standardizing Country Names
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country)
WHERE country LIKE 'United States%';

Example:

"United States." → "United States"
6. Fixing Date Formatting

The original date column was stored as text.

Converted to proper MySQL DATE format:

UPDATE layoffs_staging2
SET date = STR_TO_DATE(date, '%m/%d/%Y');

Modified column datatype:

ALTER TABLE layoffs_staging2
MODIFY COLUMN date DATE;
7. Handling Null and Blank Values
Replacing Blank Industries with NULL
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';
Filling Missing Industries

Missing industry values were populated using self-joins based on matching company names.

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL
AND t2.industry != '';

Example:

Missing Airbnb industry updated to "Travel"
8. Removing Incomplete Records

Rows with both:

total_laid_off
percentage_laid_off

missing were removed.

DELETE FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
9. Final Cleanup

The temporary helper column was removed.

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
Final Result

The final cleaned dataset:

Contains no duplicate rows
Has standardized company, industry, and country names
Uses proper date formatting
Handles missing values appropriately
Removes unusable records

Final clean table:

layoffs_staging2
SQL Concepts Used

This project demonstrates:

Window Functions
ROW_NUMBER()
Common Table Expressions (CTEs)
Data Cleaning Techniques
String Functions
Date Functions
Self Joins
NULL Handling
Table Alterations
Data Standardization
Duplicate Removal
Skills Demonstrated
SQL Data Cleaning
MySQL
Data Transformation
ETL Preparation
Analytical Data Preparation
Database Management
Real-world Data Wrangling
Future Improvements

Possible future enhancements:

Automate cleaning with stored procedures
Add data validation rules
Build dashboards using cleaned data
Perform exploratory data analysis (EDA)
Create automated ETL pipelines
Author

Created as part of a SQL Data Cleaning Portfolio Project for practicing real-world data preparation and transformation techniques.
