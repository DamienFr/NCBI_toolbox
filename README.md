#1 BigQUERY

Using BigQUERY to identify all sequencing runs containing reads attributed to your taxon of interest. Particularly usefull to identify pathogens in off-target reads from host sequencing runs.

1.1 Identify the ncbi taxid of your taxon on NCBI taxonomy: https://www.ncbi.nlm.nih.gov/taxonomy

1.2 Create aGoogle BigQuery account (or AWS if you want but not documented here) : https://console.cloud.google.com/bigquery?

1.3 Create a SQL request. You can use the below template after changing the taxid to yours. 
Note: on google BigQuery, only a few monthly request are possible for freee so go easy with test queries. Each query takes of a volume of data off your free monthly allowance of data processing volume.

SELECT m.sample_name, m.organism, m.acc, m.sample_acc, m.biosample, m.sra_study, m.bioproject, tax.self_count, taxinfo.identified_spot_count, tax.total_count, tax.tax_id

FROM 
`nih-sra-datastore.sra.metadata` as m,
`nih-sra-datastore.sra_tax_analysis_tool.tax_analysis` as tax,
`nih-sra-datastore.sra_tax_analysis_tool.tax_analysis_info` as taxinfo

WHERE m.acc=tax.acc
and 
tax.tax_id=31744
and 
taxinfo.acc=m.acc

ORDER 
BY  tax.total_count, m.bioproject, m.sra_study, m.biosample, m.sample_acc, m.sample_name, m.organism

You obtain a csv table with all the sequencing runs that match the taxid, here 31744
The column tax.self_count represents the number of matched reads, so i think it can be seen as the lower bound for the number of usable reads for you in later stages. You should filter on that.
If your target organism has a genome size of 40 000 bp and read length is 150, you can specify 500 to target a minimum sequencing depth of 2X for example.

1.4 
