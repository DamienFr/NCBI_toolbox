# BigQUERY for Identifying Taxon-Specific Sequencing Runs

Use **Google BigQuery** to identify all sequencing runs that contain reads attributed to your **taxon of interest**.  
This approach is particularly useful for detecting pathogens from **off-target reads** in host sequencing runs.
NCBI also provides guides and tutorials for [SRA Taxonomy Analysis Tool](https://www.ncbi.nlm.nih.gov/sra/docs/sra-taxonomy-analysis-tool/) and [BigQuery](https://www.ncbi.nlm.nih.gov/sra/docs/sra-bigquery/) that partially overlap with the present content.

---

## 📌 1. Prerequisites

### 1.1 Find the NCBI Taxonomy ID

Identify the NCBI Taxonomy ID (TaxID) for your target organism here:  
🔗 [NCBI Taxonomy Browser](https://www.ncbi.nlm.nih.gov/taxonomy)

---

### 1.2 Set Up Google BigQuery

Create a Google BigQuery account:  
🔗 [Google BigQuery Console](https://console.cloud.google.com/bigquery)

</details>**Note:** You can also use AWS Athena or other services, but only Google BigQuery is documented here.

---

## 🧾 2. Writing and Running Your SQL Query

### 2.1 Use the Template Query

Use the following SQL query as a template. Replace `31744` with your **target taxon’s TaxID**. This code is to be used on google's BigQuery interface. It will run on their server, not on your computer.

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
```

> ⚠️ Important: BigQuery offers limited free queries per month.
>    Each query consumes your free monthly data processing quota, so test cautiously.

---

### 2.2 Interpreting the Output

The output is a CSV file listing all sequencing runs that contain reads matching your specified TaxID (e.g., 31744).

tax.self_count: Number of reads mapped to your taxon.
Use this as a lower bound for estimating usable reads.

You should filter runs based on this value for sufficient coverage.

>    💡 Example:
>    If your organism’s genome is ~40,000 bp and your reads are 150 bp long, filter for runs with self_count > 500 to aim for at least ~2× coverage.

---

### ⬇️ 2.3 Downloading the libraries

For a few sequencing runs, it is possible to download them manually from NCBI SRA, but in most cases dedicated tools are more convenient and efficient.
I usually use the bit of code below to do that. I can provide more details upon request.

```bash
sra_acc=SRR8534768
path=temp

prefetch ${sra_acc} --output-directory ${path} --max-size u
fasterq-dump ${path}/${sra_acc} --outdir ${path}/${sra_acc}
```

Then typical bioinformatics pipeline might be used: de novo assemblies, mapping etc ...
In some cases it can be useful to map against a reference and discard all unmapped reads to keep the dataset size small.






