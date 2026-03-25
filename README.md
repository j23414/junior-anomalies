# junior-anomalies

Temporary repository, exploring a few minor variant pipelines

## Notes

* Pipeline Option: https://nf-co.re/viralrecon/2.6.0/docs/usage/
* Pipeline or data option: [McCrone et al, 2018](https://elifesciences.org/articles/35962) - flu focus, pull dataset with BioProject PRJNA412631
* [Xue et al, 2018](https://pmc.ncbi.nlm.nih.gov/articles/PMC6097882/) - glance and see if there are ways to mitigate limitations
* [Akande et al, 2025](https://pmc.ncbi.nlm.nih.gov/articles/PMC12282516/) - example paper using viralrecon 

## Explore

From an HPC

1. Pull data

```bash
nextflow run nf-core/fetchngs \
   -profile singularity,stjude \
   --nf_core_pipeline viralrecon \
   --input data/ids.csv \
   --outdir data \
   --download_method aspera \
   -resume
```

2. Try viralrecon


```bash
# Test against a reference
nextflow run nf-core/viralrecon \
-profile singularity,stjude \
--input data/samplesheet/samplesheet.csv \
--platform 'illumina' \
--protocol metagenomic \
--skip_assembly \
--variant_caller lofreq \
--min_allele_freq 0.02 \
--min_base_quality 30 \
--min_mapping_quality 30 \
--deduplication true \
--outdir results \
--genome 'CY207731'
```