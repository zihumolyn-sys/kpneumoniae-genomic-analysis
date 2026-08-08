# K. pneumoniae Analysis Pipeline — Scripts

This document contains all scripts used in the K. pneumoniae bioinformatics pipeline, in order of execution.

---

## 1. Quality Control — `01_qc.sh`

Tools: FastQC, MultiQC

```bash
#!/bin/bash
# Quality Control
# Tools: FastQC, MultiQC
mkdir -p ~/kpneumoniae/fastqc_results
fastqc ~/kpneumoniae/raw_data/*.fastq \
    -o ~/kpneumoniae/fastqc_results/ \
    --threads 4

cd ~/kpneumoniae/fastqc_results
multiqc .
```

---

## 2. Read Trimming — `02_trimming.sh`

Tool: fastp

```bash
#!/bin/bash
# Read Trimming
# Tool: fastp

mkdir -p ~/kpneumoniae/fastp_trimmed

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    fastp -i ~/kpneumoniae/raw_data/${SRR}_1.fastq \
          -I ~/kpneumoniae/raw_data/${SRR}_2.fastq \
          -o ~/kpneumoniae/fastp_trimmed/${SRR}_1.fastq \
          -O ~/kpneumoniae/fastp_trimmed/${SRR}_2.fastq \
          -h ~/kpneumoniae/fastp_trimmed/${SRR}_report.html \
          -j ~/kpneumoniae/fastp_trimmed/${SRR}_report.json
    echo "Done: $SRR"
done
```

---

## 3. Genome Assembly — `03_assembly.sh`

Tool: SPAdes

```bash
#!/bin/bash
# Genome Assembly
# Tool: SPAdes

mkdir -p ~/kpneumoniae/assembly

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    spades.py -1 ~/kpneumoniae/fastp_trimmed/${SRR}_1.fastq \
              -2 ~/kpneumoniae/fastp_trimmed/${SRR}_2.fastq \
              -o ~/kpneumoniae/assembly/${SRR} \
              --threads 4
    echo "Done: $SRR"
done
```

---

## 4. Assembly Quality Assessment — `04_assembly_qc.sh`

Tool: QUAST

```bash
#!/bin/bash
# Assembly Quality Assessment
# Tool: QUAST

mkdir -p ~/kpneumoniae/quast_results

quast.py ~/kpneumoniae/assembly/ERR3569676/contigs.fasta \
         ~/kpneumoniae/assembly/ERR3569677/contigs.fasta \
         ~/kpneumoniae/assembly/ERR3569679/contigs.fasta \
         ~/kpneumoniae/assembly/SRR21901333/contigs.fasta \
         ~/kpneumoniae/assembly/SRR21901335/contigs.fasta \
         -o ~/kpneumoniae/quast_results \
         --threads 4
```

---

## 5. Species Confirmation — `05_species_confirmation.sh`

Tools: BLASTn, FastANI

```bash
#!/bin/bash
# Species Confirmation
# Tools: BLASTn, FastANI

mkdir -p ~/kpneumoniae/fastani

# Extract largest contig for BLAST
for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    python3 -c "
from Bio import SeqIO
record = list(SeqIO.parse('/home/lyn-zihumo/kpneumoniae/assembly/${SRR}/contigs.fasta', 'fasta'))[0]
print('>' + record.id)
print(str(record.seq[:1000]))
" > ~/kpneumoniae/assembly/${SRR}/largest_contig_1000bp.fasta
done
# FastANI
ls ~/kpneumoniae/assembly/*/contigs.fasta > ~/kpneumoniae/fastani/query_list.txt

fastANI --ql ~/kpneumoniae/fastani/query_list.txt \
        -r ~/kpneumoniae/fastani/GCF_000240185.1_ASM24018v2_genomic.fna \
        -o ~/kpneumoniae/fastani/fastani_results.txt
```

---

## 6. AMR Gene Detection — `06_amr_detection.sh`

Tool: AMRFinderPlus v4.2.7, Database: 2026-03-24.1

```bash
#!/bin/bash
# AMR Gene Detection
# Tool: AMRFinderPlus v4.2.7, Database: 2026-03-24.1

mkdir -p ~/kpneumoniae/amr_results

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    amrfinder -n ~/kpneumoniae/assembly/${SRR}/contigs.fasta \
              --organism Klebsiella_pneumoniae \
              --output ~/kpneumoniae/amr_results/${SRR}_amr.txt \
              --threads 4
    echo "Done: $SRR"
done
```

---

## 7. Virulence Gene Detection — `07_virulence_detection.sh`

Tool: ABRicate v1.0.1, Database: VFDB 2022-12-02

```bash
#!/bin/bash
# Virulence Gene Detection
# Tool: ABRicate v1.0.1, Database: VFDB 2022-12-02

mkdir -p ~/kpneumoniae/abricate_results

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    abricate --db vfdb \
             --minid 50 \
             --mincov 30 \
             ~/kpneumoniae/assembly/${SRR}/contigs.fasta \
             > ~/kpneumoniae/abricate_results/${SRR}_vfdb.txt
    echo "Done: $SRR"
done

abricate --summary ~/kpneumoniae/abricate_results/*_vfdb.txt \
         > ~/kpneumoniae/abricate_results/summary_vfdb.txt
```

---

## 8. Phylogenetic Analysis — `08_phylogenetics.sh`

Tools: Snippy v4.6.0, IQ-TREE v2.0.7

```bash
#!/bin/bash
# Phylogenetic Analysis
# Tools: Snippy v4.6.0, IQ-TREE v2.0.7

mkdir -p ~/kpneumoniae/snippy

conda activate snippy_env

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
    snippy --cpus 4 \
           --outdir ~/kpneumoniae/snippy/${SRR} \
           --ref ~/kpneumoniae/fastani/GCF_000240185.1_ASM24018v2_genomic.fna \
           --R1 ~/kpneumoniae/fastp_trimmed/${SRR}_1.fastq \
           --R2 ~/kpneumoniae/fastp_trimmed/${SRR}_2.fastq \
           --force
    echo "Done: $SRR"
done
cd ~/kpneumoniae/snippy
snippy-core --ref ~/kpneumoniae/fastani/GCF_000240185.1_ASM24018v2_genomic.fna \
            ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335

iqtree2 -s ~/kpneumoniae/snippy/core.aln \
        -m GTR+G \
        -bb 1000 \
        -nt AUTO \
        -pre ~/kpneumoniae/snippy/kpneumoniae_tree
```

---

## 9. MLST Typing — `09_mlst.sh`

Tool: mlst v2.33.1, Pasteur K. pneumoniae scheme

```bash
#!/bin/bash
# MLST Typing
# Tool: mlst v2.33.1, Pasteur K. pneumoniae scheme

mkdir -p ~/kpneumoniae/mlst_results

for SRR in ERR3569676 ERR3569677 ERR3569679 SRR21901333 SRR21901335; do
echo "=== $SRR ==="
    mlst ~/kpneumoniae/assembly/${SRR}/contigs.fasta
    echo ""
done > ~/kpneumoniae/mlst_results/mlst_results.txt
```
