# ParaRef: A decontaminated reference database for parasite detection in ancient and modern metagenomic datasets 
Shotgun metagenomics is a valuable tool for detecting parasite DNA, but contamination in reference genomes can lead to false positives. To address this, we curated ParaRef, a cruated reference database by quantifying and removing contamination from 831 published endoparasite genomes. Testing this database on modern and ancient metagenomic datasets showed a significant reduction in false positive detections, providing a more reliable resource for parasite identification.

### Workflow to Create the Curated Parasite Genome Database

1. **Compilation of Reference Genomes**:
   - Gather parasite genome sequences from public databases such as **NCBI** and **VEuPathDB** Release 61.
   
2. **Contamination Detection and Removal**:
   - Apply **[FCS-adaptor](https://github.com/ncbi/fcs/wiki/FCS-adaptor-quickstart)**, **[FCS-GX](https://github.com/ncbi/fcs/wiki/FCS-GX-quickstart)** and **[Contaminator](https://github.com/steineggerlab/conterminator)** to detect contamination within the collected genomes.
   - We provide a selection of decontaminated parasite reference genomes on zenodo that can be used for the workflow below.

3. **Building KrakenUniq database**:
We applied a Snakemake workflow (Scripts/BuildPathopipeDB) to set up KrakenUniq databases and bowtie2 reference index. Requires a folder called library with the genomes, and refs.tsv file with names and paths to the reference genomes. If the provided decontaminated parasite genomes are used, the Scripts/refs.tsv file can be used.
   - Mask low-complexity regions in both the original and decontaminated genomes.
   - Build Bowtie2 index for each masked orginal and decontaminated genomes.
   - Create separate **KrakenUniq** databases for the masked original genomes and the decontaminated genomes.
   - Create library_SeqInfo_bt2.tsv

5. **Preprocessing of Sequencing Data**:
   - Preprocess sequencing data from datasets using the ancient DNA pipeline **[nf-core/eager 2.4.7](https://nf-co.re/eager/2.4.7)**.
   - Align non-host reads to the respective host organism to filter out host-derived sequences.

6. **Processing of Non-Host Reads**:
   - Use the **snakemake** workflow **[pathopipe](https://github.com/martinsikora/pathopipe)** for further processing of non-host reads. Requires Resources/targets.tsv file with list of all genera of the genomes within the database and Resources/targets_priority.tsv, which contains a curated list of parasitic genera.
     - Perform metagenomic classification with **KrakenUniq** against ParaRef database.
     - Map reads at the genus level using **Bowtie2**.
     - Perform authentication steps for accurate species identification.

7. **Species-Level Filtering**:
   - Include species that have over 300 unique kmers across assigned reads for further alignment and verification.
   - Extract all reads for each species from the genus-level assignment.
   - Perform alignment using **Bowtie2** in "global very sensitive" mode, allowing for 1 mismatch in the seed between reads and the reference genome.

8. **Authentication of Hits**:
   - For each genus, select the species with the highest number of unique kmers.
   - Authenticate true positive hits using the following criteria:
     - Hits with more than 1,000 aligned reads.
     - Average edit-distance between the read and reference genome.
     - Detection of damage patterns on both ends of reads.
     - Evenness of coverage (with a score above 0.8).
   
9. **Evaluation of Decontamination Efficiency**:
   - Assess the number of positive hits, differences in read assignments, and coverage statistics between the original and decontaminated genomes.
   - Use these metrics to evaluate the efficiency of contamination-masking and confirm the reliability of the curated database for parasite detection.

This workflow ensures a rigorous process to decontaminate, evaluate, and authenticate parasite genomes, providing a robust database for accurate parasite detection in metagenomic studies.

#### Reference
1. O’Leary NA, Cox E, Holmes JB, Anderson WR, Falk R, Hem V, et al. Exploring and retrieving sequence and metadata for species across the tree of life with NCBI Datasets. Sci Data. 2024;11:732.
2. Amos B, Aurrecoechea C, Barba M, Barreto A, Basenko EY, Bażant W, et al. VEuPathDB: the eukaryotic pathogen, vector and host bioinformatics resource center. Nucleic Acids Res. 2022;50:D898–911.
3. Astashyn A, Tvedte ES, Sweeney D, Sapojnikov V, Bouk N, Joukov V, et al. Rapid and sensitive detection of genome contamination at scale with FCS-GX. bioRxiv. 2023. 
4. Steinegger M, Salzberg SL. Terminating contamination: large-scale search identifies more than 2,000,000 contaminated entries in GenBank. Genome Biol. 2020;21:115.
5. Fellows Yates JA, Lamnidis TC, Borry M, Andrades Valtueña A, Fagernäs Z, Clayton S, et al. Reproducible, portable, and efficient ancient genome reconstruction with nf-core/eager. PeerJ. 2021;9:e10947.
6. Sikora M, Canteri E, Fernandez-Guerra A, Oskolkov N, Ågren R, Hansson L, et al. The spatiotemporal distribution of human pathogens in ancient Eurasia. Nature. 2025;643:1011–9.

