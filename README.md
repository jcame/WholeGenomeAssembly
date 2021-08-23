# Steps to Hybrid Whole Genome Assembly (WGA) (Illumina | ONT)

Make sure you have install the following dependencies:
- trimmomatic
- seqkit
- bbmap
- fastq-pair
- spades
- nanofilt

If you miss any of them you can install them through ANACONDA as folllows:

```
conda install -c bioconda trimmomatic
conda install -c bioconda seqkit 
conda install -c bioconda bbmap 
conda install -c bioconda fastq-pair 
conda install -c bioconda spades  
conda install -c bioconda nanofilt
```

---

Once installed let us proceed with the assembly:

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
bbduk.sh in=DPF1.fq out=forward.fq  ref=/Users/mxl641/Desktop/Software/phiX/phi_X174_phage.fa k=31 hdist=1
bbduk.sh in=DPF2.fq out=reverse.fq  ref=/Users/mxl641/Desktop/Software/phiX/phi_X174_phage.fa k=31 hdist=1

rm DPF1.fq
rm DPF2.fq
```
