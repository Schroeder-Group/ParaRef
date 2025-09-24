# ParaRef: Decontaminated Parasite Reference Database
A curated, **decontaminated** collection of parasite genomes intended for **species-level parasite detection** in ancient and modern metagenomic datasets. ParaRef removes pervasive reference contamination that otherwise inflates false positives, while preserving sensitivity.

### Quick start
- **Download ParaRef** (masked & decontaminated genomes) from Zenodo and unpack into ParaRef_PathopipeDB/library/.
- **Build indices**: use Scripts/ParaRef_PathopipeDB (Snakemake) to create a **KrakenUniq** DB and **Bowtie2** indices from ParaRef_PathopipeDB/library/ using Scripts/ParaRef_PathopipeDB/refs.tsv.
- **Preprocess reads** with **nf-core/eager** to trim/merge and **remove host reads**.
- **Classify** non-host reads with **KrakenUniq** against **ParaRef**; map top candidate genera with **Bowtie2** (Pathopipe).
- **Validate** species hits (≥300 unique k-mers, ≥1,000 aligned reads, mode edit distance <2, even coverage entropy >0.8; add ancient DNA damage checks where relevant).
### Contents
- Scripts/env.yml – conda environment for building ParaRef pathopipe database and running Pathopipe
- Scripts/ParaRef_PathopipeDB/BuildDB_Snakefile – Snakemake workflow to build KrakenUniq DB and Bowtie2 indices
- Scripts/ParaRef_PathopipeDB/refs.tsv – example reference table (name, path) for ParaRef genomes
- Scripts/Pathopipe/targets.tsv – list of all genera present in the database for Pathopipe
- Scripts/Pathopipe/targets_priority.tsv – curated list of parasitic genera (prioritisation) for Pathopipe
- Scripts/Pathopipe/config.yml – configuration file for Pathopipe
- Scripts/Pathopipe/Input.tsv – example input file for Pathopipe
### Requirements
- Conda environment build from Scripts/env.yml 
- Snakemake (≥7.x)
- KrakenUniq
- Bowtie2 
- nf-core/eager
### 1) Compile / obtain reference genomes
#### Option A (recommended): Use ParaRef from Zenodo
1. Copy Scripts/ParaRef_PathopipeDB to your working directory 
2. Download and extract genomes into ParaRef_PathopipeDB/library/
#### Option B: Build user-defined decontaminated database
If rebuilding, **hard-mask** contaminant intervals, and **exclude contigs <1 kb** wherever possible (these disproportionately carry contamination).
##### Create a refs.tsv with two tab-separated columns:
```
assemblyId	fasta
name    /absolute/or/relative/path/to/ParaRef_PathopipeDB/library/<name>.fasta
```
**Example:**
```
assemblyId	fasta
FungiDB-61-Acandida2VRR	library/FungiDB-61-Acandida2VRR_Genome.decontaminated.fna.gz
```
### 2) Build databases (KrakenUniq + Bowtie2)
Run the Snakemake workflow from the ParaRef_PathopipeDB directory to generate both the **KrakenUniq** database and **Bowtie2** indices from library/ using your ParaRef_PathopipeDB/refs.tsv.
```
snakemake -s BuildDB_Snakefile
```
Outputs include:
   - ParaRef_PathopipeDB/database.* – KrakenUniq DB built from ParaRef
   - ParaRef_PathopipeDB/bt2/ – Bowtie2 indices per genome
   - ParaRef_PathopipeDB/library.seqInfo_bt2.tsv – per-reference index metadata
### 3) Preprocess sequencing reads (nf-core/eager)
Use nf-core/eager to trim/merge reads and **remove host sequences** by mapping against the relevant host reference (human, dog, pig, etc.). Keep **non-host** FASTQ for downstream taxonomic classification.
**Minimal example**:
```
nextflow run nf-core/eager \
  --input samplesheet.tsv \
  --fasta /path/to/HOST_REFERENCE.fa \
  --run_bam_filtering --bam_unmapped_type fastq
```
### 4) Classify and map non-host reads (Pathopipe)
We use a Snakemake-based workflow (Pathopipe) to orchestrate classification and validation.
**Inputs**
- ParaRef_PathopipeDB built from ParaRef
- Scripts/Pathopipe/targets.tsv – all genera represented in the DB
- Scripts/Pathopipe/targets_priority.tsv – curated parasitic genera
- Scripts/Pathopipe/config.yml – configuration file
- Scripts/Pathopipe/Input.tsv – non-host fastq files from nf-core/eager
- Snakemake and src/ from https://github.com/martinsikora/pathopipe  
  
**Run Pathopipe**:

```
snakemake -s Snakefile --configfile config.yml
```
**Steps**:
- **KrakenUniq classification** against **ParaRef**.
- **Genus-level mapping**: for each candidate genus, map reads with **Bowtie2** to **all** species references in that genus.
- **Species-level filtering** (see thresholds below).
- **Alignment-based validation** and summary metrics.
### 5) Species-level filtering (pre-validation)
Carry forward species that meet a minimal evidence threshold:
   **≥300 unique k-mers** (KrakenUniq) across assigned reads
For each retained species:
- **Extract reads** from the genus-level set
- **Bowtie2** alignment using **“–very-sensitive”** global settings
(allow up to **1 seed mismatch** as in our study)
### 6) Authentication & validation of hits
For each genus, **keep the species with the highest unique k-mer count**, then validate using:
- **Read count: ≥1,000 aligned reads**
- **Alignment quality**: **low mode edit distance** (e.g., <2)
- **Coverage evenness**: **coverage entropy** (e.g., covPosRelEntropy1000) **> 0.8**
- **Ancient DNA** (when applicable): **deamination** patterns (C→T/G→A at read ends) and **short fragment lengths**
These checks help separate **true positives** from residual contamination or misassignment.
### Output overview
- **KrakenUniq reports** (per sample)
- **Genus- and species-level alignments** (BAM)
- **Summary tables** (unique k-mers, aligned read counts, edit distance, coverage breadth, **coverage entropy**)
- **QC plots** (optional; edit distance, damage patterns for aDNA)
### Citation
If you use **ParaRef** or the accompanying workflows, please cite:
- Niemann et al.  Genome Biol. 2025 (ParaRef)
- Breitwieser FP, Baker DN, Salzberg SL. Genome Biol. 2018 (KrakenUniq)
- Fellows Yates JA et al. PeerJ 2021 (nf-core/eager)
- Steinegger M, Salzberg SL. Genome Biol. 2020 (Conterminator)
- Astashyn A et al. Genome Biol. 2024 (FCS-GX)
- Sikora M et al. Nature. 2025 (Pathopipe)





