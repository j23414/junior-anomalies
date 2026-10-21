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
   -profile stjude \
   -c queue-limit.config \
   --nf_core_pipeline viralrecon \
   --input data/ids.csv \
   --outdir illumina \
   -resume
```

2. QC illumina data

```bash
echo "sample,R1,R2" | tr ',' '\t' > samplesheet.tsv

ls -1 illumina/fastq/*_1.fastq.gz \
| sed 's:illumina/fastq/::g' \
| sed 's:_1.fastq.gz::g' \
| awk 'OFS="\t" {print $1,"illumina/fastq/"$1"_1.fastq.gz","illumina/fastq/"$1"_2.fastq.gz"}' \
>> samplesheet.tsv

nextflow run j23414/illumina-qc \
  --samplesheet samplesheet.tsv \
  --trim_adaptors fastp \
  -profile stjude
```

3. Align against H3N2 and H1N1 references

```bash
echo "sample,R1,R2" > samplesheet.csv

ls -1 ../01_QC/qc-results_fastp/fastp/*_R1.fastp.fastq.gz \
| sed 's:illumina/fastq/::g' \
| sed 's:_1.fastq.gz::g' \
| awk 'OFS="," {print $1,"illumina/fastq/"$1"_R1.fastp.fastq.gz","illumina/fastq/"$1"_R2.fastp.fastq.gz "}' \
>> samplesheet.csv

nextflow run j23414/illumina-align \
  --samplesheet samplesheet.csv \
  --reference "data/references.fna" \
  -profile stjude
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
