# README for PneumoCaT tool
---------------------------

This tool uses a two-step step approach to assign capsular type to *S.pneumoniae* genomic data. In the first step, reads from each readset are mapped to capsular locus sequences for all known capsular types (92 for S. pneumoniae plus 2 additional subtypes/molecular types). This step is considered successful if the readset matches > 90% to 1 or more capsular locus sequences. If it matches to a single capsular locus then PneumoCaT terminates here and reports this as the assigned capsular type. If more than 1 loci are matched then the tool moves to the second step; a variant based approach that utilises the capsular type variant (CTV) database to distinguish serotypes within a serogroup/genogroup. For more information you can refer to the publication.

## Table of content
---------------------------

* Dependencies
* Running PneumoCaT
* PneumoCaT Output
* CTV database
* Examples
* Troubleshooting

## Dependencies
---------------------------

PneumoCaT  is written with Python 2.7.5 and requires the following packages installed before running:
* bowtie2/2.1.0
* samtools/0.1.19
* yaml/1.1
* numpy/python2.7/1.7.1
* lxml/python2.7.0/3.2.3
* pysam/python2.7/0.7.5
* biopython/python2.7/1.61

## Running PneumoCaT
----------------------------

```
usage: python PneumoCaT.py [-h]
                           [--input_directory INPUT_DIRECTORY]
                           [--fastq_1 FASTQ_1] [--fastq_2 FASTQ_2]

Arguments:
  -h, --help            
        show this help message and exit
  --input_directory INPUT_DIRECTORY, -i INPUT_DIRECTORY
        please provide the path to the directory contains the fastq files; [REQUIRED - OPTION 1]
  --fastq_1 FASTQ_1, -1 FASTQ_1
        Fastq file pair 1 [REQUIRED - OPTION 2]
  --fastq_2 FASTQ_2, -2 FASTQ_2
        Fastq file pair 2 [REQUIRED - OPTION 2]
  --variant_database VARIANT_DATABASE, -d VARIANT_DATABASE
        variant database [OPTIONAL]; defaults to streptococcus-pneumoniae-ctvdb in the github directory
  --output_dir OUTPUT_DIR, -o OUTPUT_DIR
        please provide an output directory [OPTIONAL]; if none provided a pneumo_capsular_typing folder will be created in the directory containing the fastq files
  --bowtie BOWTIE, -b BOWTIE
        please provide the path for bowtie2 [OPTIONAL]; defaults to bowtie2
  --samtools SAMTOOLS, -s SAMTOOLS
        please provide the path for samtools [OPTIONAL]; defaults to samtools
```

## Output files
---------------

### STEP 1: COVERAGE-BASED APPROACH
1. **SAMPLEID.results.xml** - The XML file at step 1 is basic and it only contains the top two capsular types and their respective read coverage (% of the capsular locus length covered). If only one capsular type is matched with more than 90% coverage then the report from step 1 is considered the final result. If more than one capsular type are matched with more than 90% coverage then the software moves to step two and a second XML file is generated with the final result. Note that the output XML file from step 1 only reports two capsular types, when more could be matched and all will pass to step 2 for further distinction. If the top coverage is < 90% then no serotypes are reported and 'Failed' appears instead.
2. **SAMPLEID.sorted.bam** - BAM file generated during step 1 using the 94 capsular locus sequences as reference.
3. **SAMPLEID.sorted.bam.bai** - index file for the sorted BAM file
4. **ComponentComplete.txt** - indicates PneumoCaT analysis was completed succssfully
5. **coverage_summary.txt** - contains the coverage values for all serotypes. This is useful if the step has failed, epsecially if the top coverage falls close to the 90% threshold.

### STEP 2 - VARIANT-BASED APPROACH
1. **SAMPLEID.results.xml** - The XML file at step 2 details the assigned capsular type, total hits, the capsular types studied in this analysis as well as a full breakdown of the mutations used for this assignment. Total hits corresponds to the number of matched mutations vs the number of all mutations tested. A capsular type is assigned only if all mutations matched. If a complete match is not possible then 'Serotype Undetermined' is reported.
  Coverage statistic metrics are calculated for each gene locus used for this assignment. These include five values as detailed below:
  * Minimum depth: the minimum number of reads that maps to the gene sequence at any single position.
  * Maximum depth: the maximum number of reads that maps to the gene sequence at any single position.
  * Average depth: the average number of reads that maps across the length of the gene sequence.
  * Percent Coverage: records coverage across the length of each gene sequence.
  * Coverage Distribution: records the area of the gene sequence covered by reads.
2. **SAMPLEID.sorted.bam** - BAM file generated during step 2 using specific genes, assigned for each serogroup/genogroup in the CTV database, as reference 
3. **SAMPLEID.sorted.bam.bai** - index file for the sorted BAM file
4. **variant_summary.yml** - contains the variant matches for all serotypes tested. This is useful if a complete match was not possible and 'Serotype undetermined' was reported. It can give information on coverage for specific positions, mixed positions and other events that could lead to failure to assing a serotype.

## CTV database
---------------
Capsular Type Variant (CTV) database is supplied as a structured directory (streptococcus-pneumoniae-ctvdb). Within this directory you can find:
* reference.fasta - This file contains the capsular locus sequences for all 92 serotypes plus 6E and 23B1 first described in the publication.
* 19 subfolders corresponding to each genogroup recognised in publication. Each subfolder contains:
  * mutationdb.pickled - contains the variants that can distinguish the serotypes within this serogroup. This file is read by PneumoCaT during step 2.
  * mutationdb.yml - contains the same information as the mutationdb.pickled file but can be read using text editors.
  * reference.fasta - contains the sequences for the genes defined in the mutationdb file.

## Examples
-----------

### OPTION 1: Use input directory that contains fastq files for the isolate

Default for all optional arguments:

`python PneumoCaT.py -i <path-to-input-directory> `

With Optional arguments:

`python PneumoCaT.py -i <path-to-input-directory> -o <path-to-output-directory> -d <path-to-the-database> -b <path-to-bowties> -s <path-to-samtools>`

### Option 2: Use paths to fastq files
Default for all optional arguments:

`python PneumoCaT -1 <path-to-FASTQ1> -2 <path-to-FASTQ2>`

With Optional arguments:

`python PneumoCaT -1 <path-to-FASTQ1> -2 <path-to-FASTQ2> -o <path-to-output-directory> -d <path-to-the-database> -b <path-to-bowties> -s <path-to-samtools>`

## Troubleshooting
------------------
### Formats for the output of failure

Eight different formats of failure values are returned depending on the coverage statistic values and individual mutations:

* "Reference data not available ['24B', '24F']": Serogroups 24 and 32 have no available reference data.
* "Mixed ['01', '22A', '22F']": The combination of serotypes matched with more than 90% coverage during step 1 does not correspond to any known serogroup or genogroup. In this case mixed culture is suspected.
* “Low coverage”: Less than 90% coverage of the gene sequence.
* "SNP position 1002: Depth < 5”: Nucleotide position covered by less than 5 reads.
* “SNP position 1002 not covered”: No reads map at this nucleotide position.
* "Mixed: {'allele-2': 0.99, 'allele-1': 0.99}": Mixed samples based on presence of two alleles for the gene with breakdown of coverage for the alleles found. There could be two scenarios in this case:
  * {'allele-2': 0.99, 'allele-1': 0.99}: when both alleles has coverage more than 90% then a mixed culture is suspected.
  * {'allele-1': 0.75, 'allele-2': 0.25}: In this case, further bioinformatics investigation is required to determine chimeric state.
* "Mixed: (413, 414),TA/--,TA:0.67,--:0.33”: Mixed genotype at specific region with breakdown of frequency for each state. Usually corresponds to frameshift mutation responsible for loss of function (pseudogene).
* "Mixed: (215: {'A': 0, 'C': 64, 'T': 67, 'G': 0})”: Mixed genotype at specific position with breakdown of nucleotides found.

### Threshold values

Mutations can be confidently reported if the percentage coverage for the gene is more than 90% and a single fragment is reported in the coverage distribution. As well as this general threshold individual mutations have specific thresholds.
* Alleles: An allele is reported if it has coverage more than 90% and the other alleles have coverage < 10%, otherwise a mixed tag is reported.
* SNPs: When a specific nt position is investigated the hit will only be reported if 
  * depth in the codon area is more than 5
  * consensus base has more than 75% frequency
  * mapping quality is more than 30
  * no other snps are reported within ±5 nts area 
* Pseudogenes: The overall coverage threshold has been reduced to 70% to allow for reporting large indels/truncations as a causal agent for loss of function. A functional orf is reported if coverage is more than 80% and a single orf is present that match the functional orf for the gene tested. This lower threshold (80% instead of 90%) has been empirically adjusted to allow for poor mapping in the beginning and end of the gene locus. If specific frameshift mutations are investigated then the same threshold as described above are used (SNPs).

## Contact
----------
Georgia Kapatai 

Email: Georgia.Kapatai@phe.gov.uk 

Twitter: @gkapatai