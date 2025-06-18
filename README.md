# 🧬 BigQUERY for Identifying Taxon-Specific Sequencing Runs

Use **Google BigQuery** to identify all sequencing runs that contain reads attributed to your **taxon of interest**.  
This approach is particularly useful for detecting pathogens from **off-target reads** in host sequencing runs.

---

## 📌 1. Prerequisites

### 1.1 Find the NCBI Taxonomy ID

Identify the NCBI Taxonomy ID (TaxID) for your target organism here:  
🔗 [NCBI Taxonomy Browser](https://www.ncbi.nlm.nih.gov/taxonomy)

---

### 1.2 Set Up Google BigQuery

Create a Google BigQuery account:  
🔗 [Google BigQuery Console](https://console.cloud.google.com/bigquery)

> **Note:** You can also use AWS Athena or other services, but only Google BigQuery is documented here.

---

## 🧾 2. Writing and Running Your SQL Query

### 2.1 Use the Template Query

Use the following SQL query as a template. Replace `31744` with your **target taxon’s TaxID**.

<details>
<summary>Click to expand SQL query</summary>

```sql
SELECT 
  m.sample_name,
  m.organism,
  m.acc,
  m.sample_acc,
  m.biosample,
  m.sra_study,
  m.bioproject,
  tax.self_count,
  taxinfo.identified_spot_count,
  tax.total_count,
  tax.tax_id
FROM 
  `nih-sra-datastore.sra.metadata` AS m,
  `nih-sra-datastore.sra_tax_analysis_tool.tax_analysis` AS tax,
  `nih-sra-datastore.sra_tax_analysis_tool.tax_analysis_info` AS taxinfo
WHERE 
  m.acc = tax.acc
  AND tax.tax_id = 31744
  AND taxinfo.acc = m.acc
ORDER BY  
  tax.total_count,
  m.bioproject,
  m.sra_study,
  m.biosample,
  m.sample_acc,
  m.sample_name,
  m.organism
