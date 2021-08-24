# Whole Genome Assembly (WGA) Workflow (Illumina | ONT)

Make sure you have install the following dependencies:
- trimmomatic
- seqkit
- bbmap
- fastq-pair
- spades
- nanofilt
- blast


If you miss any of them you can install them through ANACONDA as folllows:

```
conda install -c bioconda trimmomatic
conda install -c bioconda seqkit 
conda install -c bioconda bbmap 
conda install -c bioconda fastq-pair 
conda install -c bioconda spades  
conda install -c bioconda nanofilt
conda install -c bioconda blast
```

----
---

Once installed let us proceed with the assembly, download phi_X174_phage.fa (spiked QC DNA) and NexteraPE-PE.fa (adapters list) provided in the files of this repository.

Also download the illumina and ONT data tol be assembled here: https://www.dropbox.com/sh/vesi0sbbwp5z6ri/AABaLcer6hOyaJLPS6y10yc-a?dl=0


## Quality Control
```
trimmomatic PE -threads 4 -phred33 *R1* *R2* PF1.fq UF1.fq PF2.fq UF2.fq ILLUMINACLIP:NexteraPE-PE.fa:2:30:10 LEADING:20 TRAILING:20 MINLEN:50

cat PF1.fq UF1.fq > forward.fq
cat PF2.fq UF2.fq > reverse.fq

rm PF1.fq 
rm UF1.fq
rm PF2.fq
rm UF2.fq
```

## Dereplication

```
seqkit rmdup forward.fq -s -o DPF1.fq -j 4
seqkit rmdup reverse.fq -s -o DPF2.fq -j 4

rm forward.fq
rm reverse.fq
```

## Deconvolution

```
bbduk.sh in=DPF1.fq out=forward.fq  ref=phi_X174_phage.fa k=31 hdist=1
bbduk.sh in=DPF2.fq out=reverse.fq  ref=phi_X174_phage.fa k=31 hdist=1

rm DPF1.fq
rm DPF2.fq
```

## Pairing sequences (paired & unpaired)

```
fastq_pair forward.fq reverse.fq

rm forward.fq
rm reverse.fq

cat *single.fq > unpaired.fq
rm *single.fq
```

## Denovo Assembly using (i) Illumina

```
spades.py --pe1-1 forward.fq.paired.fq --pe1-2 reverse.fq.paired.fq --pe1-s unpaired.fq -o spades_folder_illumina -t 4 -m 7 --only-assembler
```

## Selecting long scaffolds (i) Illumina

```
seqkit seq spades_folder_illumina/scaffolds.fasta -m 2000 -g > scaffolds_2k_illumina.fasta
```

## results (i) Illumina

```
grep ">" scaffolds_2k_illumina.fasta
```


----
----

## Quality Control for ONT reads

```
cat BC10.fastq | NanoFilt -q 7 -l 500 --readtype 1D > BC10_HQ.fq
```

## Denovo Assembly using (ii) Illumina hybrid ONT

```
spades.py --pe1-1 forward.fq.paired.fq --pe1-2 reverse.fq.paired.fq --pe1-s unpaired.fq -o spades_folder_hybrid   -t 4 -m 7 --only-assembler  --nanopore BC10_HQ.fq
```

## Selecting long scaffolds (ii) Illumina hybrid ONT

```
seqkit seq spades_folder_hybrid/scaffolds.fasta   -m 2000 -g > scaffolds_2k_hybrid.fasta
```

## results (ii) Illumina hybrid ONT

```
grep ">" scaffolds_2k_hybrid.fasta
```

## Cleaning

```
rm *fq
```

----
----
----
----


## Comparing Assemblies

```
makeblastdb -in scaffolds_2k_hybrid.fasta -dbtype nucl
blastn -query scaffolds_2k_illumina.fasta -db scaffolds_2k_hybrid.fasta -outfmt 6 -evalue 10e-5 -num_threads 4 -strand both -subject_besthit  > OutSelfBlast.txt 
```
