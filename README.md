# CVE Lakehouse - Medallion Architecture

**DIC 587 - Data Intensive Computing | Fall 2025**  
**Author**: Kumar Satyam  
**Platform**: Databricks Community Edition

A cybersecurity data lakehouse implementing the Medallion Architecture (Bronze → Silver → Gold) to analyze 40,000+ CVE records from 2024.

## 🎯 Overview

This project builds a production-grade data pipeline that:

- **Ingests** 38,727 CVE JSON files from GitHub
- **Transforms** nested JSON into normalized relational tables
- **Analyzes** cybersecurity trends with business intelligence
- **Implements** Delta Lake ACID guarantees and Unity Catalog

## 🏗️ Architecture

```
Bronze (Raw)           Silver (Normalized)      Gold (Analytics)
───────────────   →    ──────────────────  →    ────────────────
• 40,000+ CVEs         • Core CVE table         • Vendor dashboards
• Raw JSON             • Affected products      • Trend analysis
• Delta Lake           • CVSS scores            • Risk profiles
• Column mapping       • Data validation        • Monthly summaries
```

**Key Features:**

- Delta Lake column mapping for nested JSON
- Unity Catalog for data governance
- 2024-only optimization (38K vs 200K+ files)
- ACID transactions and schema evolution

## 🚀 Quick Start

### Prerequisites

- Databricks Community Edition (DBR 13.x+)
- Unity Catalog volume: `CREATE VOLUME IF NOT EXISTS workspace.default.assignment1;`

### Execution

**1. Bronze Layer** (~15 min) - `01_bronze_layer_2024_starter.ipynb`

```python
# Downloads CVE data, filters to 2024, writes to Delta
# Output: cve_bronze.records (40,123 records)
```

**2. Silver Layer** (~10 min) - `02_bronzetosilver.ipynb`

```python
# Normalizes JSON, extracts CVSS, explodes vendor/products
# Output: cve_silver.core + cve_silver.affected_products
```

**3. Gold Layer** (~5 min) - `03_exploratory_analysis.ipynb`

```python
# Creates analytics views and business intelligence
# Output: 3 Gold views for dashboards
```

## 📊 Pipeline Details

### Bronze Layer

- **Table**: `cve_bronze.records` (40,000+ records)
- **Purpose**: Raw JSON preservation with Delta Lake
- **Key Feature**: Column mapping handles special characters in nested fields

### Silver Layer

- **Tables**:
  - `cve_silver.core` - One CVE per row with CVSS scores
  - `cve_silver.affected_products` - Exploded vendor/product relationships (120K+ rows)
- **Purpose**: Normalized relational structure for analytics

### Gold Layer

- **Views**:
  - `vulnerability_dashboard` - Comprehensive vulnerability view
  - `vendor_risk_profile` - Vendor-level risk metrics
  - `monthly_vulnerability_summary` - Time-series aggregates
- **Purpose**: Business-ready analytics and dashboards

## 🔍 Key Insights

**2024 Vulnerability Stats:**

- 40,000+ total CVE disclosures (record year)
- 7.5% Critical, 20% High severity
- 6.95 average CVSS score
- Top vendors: Linux Kernel, Microsoft, Google

**Sample Queries:**

```sql
-- Find critical vulnerabilities by vendor
SELECT c.cve_id, c.cvss_base_score, a.vendor, a.product
FROM cve_silver.core c
JOIN cve_silver.affected_products a ON c.cve_id = a.cve_id
WHERE c.cvss_base_score >= 9.0;

-- Monthly trends
SELECT DATE_TRUNC('month', date_published) as month,
       COUNT(*) as vuln_count, AVG(cvss_base_score) as avg_cvss
FROM cve_silver.core
GROUP BY month ORDER BY month;
```

## 🛠️ Technical Highlights

**Delta Lake Column Mapping:**

```python
# Enables special characters in nested JSON field names
.option("delta.columnMapping.mode", "name")
```

**Unity Catalog Volumes:**

```python
# Managed storage for Community Edition
volume_root = "/Volumes/workspace/default/assignment1"
```

**2024-Only Optimization:**

```python
# 70% time reduction (15 min vs 45+ min)
target_2024_dir = os.path.join(cves_dir, "2024")  # 38K vs 200K+ files
```

## 🧪 Validation

**Data Quality Checks:**

```python
✓ Count >= 30,000 records
✓ Zero null CVE IDs
✓ All CVE IDs unique
✓ CVSS scores in valid range [0.0, 10.0]
```

**Verification Queries:**

```sql
-- Bronze count
SELECT COUNT(*) FROM cve_bronze.records;  -- Expected: 40,000+

-- Silver explosion ratio
SELECT COUNT(*) FROM cve_silver.affected_products;  -- Expected: 3x Bronze

-- Gold aggregation
SELECT SUM(cve_count) FROM cve_gold.monthly_vulnerability_summary;
```

## 🎓 Learning Outcomes

**Data Engineering:**

- Medallion Architecture (Bronze/Silver/Gold)
- Delta Lake (ACID transactions, time travel)
- Unity Catalog (data governance)
- PySpark (distributed processing)

**Domain Knowledge:**

- CVE vulnerability lifecycle
- CVSS risk scoring
- Cybersecurity vendor analysis

## 🔧 Troubleshooting

**Volume not found:**

```sql
CREATE VOLUME IF NOT EXISTS workspace.default.assignment1;
```

**Schema error with special characters:**

```python
-- Already fixed with column mapping
.option("delta.columnMapping.mode", "name")
```

**Timeout during download:**

```python
spark.conf.set("spark.sql.shuffle.partitions", "8")
```

## 📚 Resources

- [CVE Project](https://github.com/CVEProject/cvelistV5)
- [Delta Lake Docs](https://docs.delta.io/)
- [Unity Catalog](https://docs.databricks.com/data-governance/unity-catalog/)
- [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)

---

**Project Status**: ✅ Complete (November 2025)  
**Total Runtime**: ~30 minutes  
**Final Output**: 3-layer lakehouse with 6 Delta tables/views
