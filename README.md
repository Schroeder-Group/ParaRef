# Parasites-project
Shotgun metagenomics is a valuable tool for detecting parasite DNA, but contamination in reference genomes can lead to false positives. To address this, we curated a database by quantifying and removing contamination from 831 published endoparasite genomes. Testing this database on modern and ancient metagenomic datasets showed a significant reduction in false positive detections, providing a more reliable resource for parasite identification.

### Workflow to Create the Curated Parasite Genome Database

1. **Compilation of Reference Genomes**:
   - Gather parasite genome sequences from public databases such as **NCBI** and **VEuPathDB** Release 61.
   
2. **Contamination Detection and Removal**:
   - Apply **FCS-GX** and **Contaminator** to detect contamination within the collected genomes.
   - Remove contaminated regions to clean the genome data using custom script.

3. **Masking Low-Complexity Regions**:
   - Use a custom script to mask low-complexity regions in both the original and decontaminated genomes.
   - Create separate **KrakenUniq** databases for the masked original genomes and the decontaminated genomes.

4. **Preprocessing of Sequencing Data**:
   - Preprocess sequencing data from datasets using the ancient DNA pipeline **nf-core/eager 2.4.7**.
   - Align non-host reads to the respective host organism to filter out host-derived sequences.

5. **Processing of Non-Host Reads**:
   - Use the **snakemake** workflow **pathopipe** for further processing of non-host reads.
     - Perform metagenomic classification with **KrakenUniq**.
     - Map reads at the genus level using **Bowtie2**.
     - Perform authentication steps for accurate species identification.

6. **Species-Level Filtering**:
   - Include species that have over 300 unique kmers across assigned reads for further alignment and verification.
   - Extract all reads for each species from the genus-level assignment.
   - Perform alignment using **Bowtie2** in "global very sensitive" mode, allowing for 1 mismatch between reads and the reference genome.

7. **Authentication of Hits**:
   - For each genus, select the species with the highest number of unique kmers.
   - Authenticate true positive hits using the following criteria:
     - Hits with more than 1,000 aligned reads.
     - Average edit-distance between the read and reference genome.
     - Detection of damage patterns on both ends of reads.
     - Evenness of coverage (with a score above 0.8).
   
8. **Evaluation of Decontamination Efficiency**:
   - Assess the number of positive hits, differences in read assignments, and coverage statistics between the original and decontaminated genomes.
   - Use these metrics to evaluate the efficiency of contamination-masking and confirm the reliability of the curated database for parasite detection.

This workflow ensures a rigorous process to decontaminate, evaluate, and authenticate parasite genomes, providing a robust database for accurate parasite detection in metagenomic studies.
