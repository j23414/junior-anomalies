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

4. Get coverage report

```bash
cat align-results/samtools_coverage/*tophit* > all-top-coverage.tsv
cat align-results/countreads/*_readcount.tsv > all-readcount.tsv

# TODO: Run this as part of the alignment command
wget https://raw.githubusercontent.com/j23414/illumina-align/refs/heads/main/bin/coverage_to_wide.py
python coverage_to_wide.py \
  --top-coverage all-top-coverage.tsv \
  --readcount all-readcount.tsv \
  --merged results-coverage.tsv
```

5. Generate consensus sequence

```bash
mkdir 03_Consensus; cd 03_Consensus
ln -s ../02_Align/align-results/bwamem2_mem .
ln -s ../02_Align/reference .

echo "sample,bam,reference" > samplesheet.csv
cat ../02_Align/results-coverage.tsv \
| awk -F'\t' 'NR>1, OFS="," {print $1,"bwamem2_mem/"$1".bam",$4}' \
>> samplesheet.csv
# $4=HA, repeat for other segment columns, add to samplesheet

nextflow run j23414/bam-consensus \
  --samplesheet samplesheet.csv \
  --reference reference/references.fna \
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
