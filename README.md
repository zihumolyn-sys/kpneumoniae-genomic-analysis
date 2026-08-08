# Genomic Characterisation of Multidrug-Resistant *Klebsiella pneumoniae*

## Overview
Whole genome sequencing (WGS) analysis of 5 clinical *Klebsiella pneumoniae* isolates from South African hospitals, focusing on antimicrobial resistance, virulence and phylogenetic relationships.

## Dataset
| Sample | BioProject | Location | Year | ST |
|--------|-----------|----------|------|----|
| ERR3569676 | PRJEB34702 | Tygerberg Hospital, Western Cape, SA | 2016-17 | ST307 |
| ERR3569677 | PRJEB34702 | Tygerberg Hospital, Western Cape, SA | 2016-17 | ST307 |
| ERR3569679 | PRJEB34702 | Tygerberg Hospital, Western Cape, SA | 2016-17 | ST101 |
| SRR21901333 | PRJNA850834 | Tembisa Hospital, Pretoria, SA | 2019-20 | ST307 |
| SRR21901335 | PRJNA850834 | Tembisa Hospital, Pretoria, SA | 2019-20 | ST307 |

## Pipeline
1. Data Retrieval — SRA Toolkit
2. Quality Control — FastQC + MultiQC
3. Read Trimming — fastp
4. Genome Assembly — SPAdes
5. Assembly QC — QUAST
6. Species Confirmation — BLASTn + FastANI
7. AMR Detection — AMRFinderPlus
8. Virulence Detection — ABRicate + VFDB
9. Phylogenetics — Snippy + IQ-TREE + iTOL
10. MLST Typing — mlst

## Key Findings
- All isolates confirmed as K. pneumoniae (ANI 98.86-99.05%)
- 4/5 isolates are ST307 — a globally disseminated high-risk clone
- ERR3569679 is ST101 carrying yersiniabactin — potentially hypervirulent
- SRR21901333 and SRR21901335 carry blaOXA-181 and blaOXA-48 (carbapenem resistance) and mgrB (colistin resistance)
- Clonal dissemination confirmed in neonatal outbreak isolates

## Tools Used
FastQC, MultiQC, fastp, SPAdes, QUAST, BLASTn, FastANI, AMRFinderPlus, ABRicate, Snippy, IQ-TREE, iTOL, mlst

## Author
Lyn Zihumo-
Midlands State University-
Bioinformatics Project1 2026
