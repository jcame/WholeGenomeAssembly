# Whole Genome Assembly (WGA) Workflow (Illumina | ONT Hybrid)

Let us create a new environment with specific packages

```
conda create -n HybridAssembly python=3.9
conda activate HybridAssembly
```

Make sure you have install the following dependencies:
- trimmomatic (Only if hybrid assembly will be used)
- bbmap (Only if hybrid assembly will be used)
- fastq-pair (Only if hybrid assembly will be used)
- spades (Only if hybrid assembly will be used)
- nanofilt
- blast
- racon
- unicycler

If you miss any of them you can install them through CONDA as folllows:

```
#conda install -c bioconda trimmomatic
#conda install -c bioconda bbmap 
#conda install -c bioconda fastq-pair 
#conda install -c bioconda spades  
conda install -c bioconda nanofilt
conda install -c bioconda blast
conda install -c bioconda racon

git clone https://github.com/rrwick/Unicycler.git  #Documentation: https://github.com/rrwick/Unicycler 
cd Unicycler
make
cd ..

```
----
---

Once installed let us proceed with the assembly, but first download the illumina and ONT data to be assembled here: https://www.dropbox.com/sh/cfdxwru5pe82bra/AACnojDJEOqGnzcwDnTc5Tjha?dl=0 


## Quality Control for ONT reads
```
cat ONT_M08.fastq | NanoFilt -q 7 -l 2000 --readtype 1D > ONT_M08_7Q_2000L.fq
```

## Denovo Assembly using (ii) Illumina hybrid ONT
```
./Unicycler/unicycler-runner.py  -l ONT_M08_7Q_2000L.fq -o output_dir -t 4 --mode bold --min_fasta_length 2000
```





# Hybrid Assembly takes over nearly 2 hours (RAM 8GB | 4 cores), feel free to do it in your spare time (Illumina | ONT Hybrid)

## Quality Control
```
trimmomatic PE -threads 4 -phred33 *R1* *R2* PF1.fq UF1.fq PF2.fq UF2.fq ILLUMINACLIP:bin/NexteraPE-PE.fa:2:30:10 LEADING:20 TRAILING:20 MINLEN:50

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
bbduk.sh in=DPF1.fq out=forward.fq  ref=bin/phi_X174_phage.fa k=31 hdist=1
bbduk.sh in=DPF2.fq out=reverse.fq  ref=bin/phi_X174_phage.fa k=31 hdist=1

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
./Unicycler/unicycler-runner.py -1 forward.fq.paired.fq -2 reverse.fq.paired.fq -s unpaired.fq -l BC10_HQ.fq -o output_dir -t 4 --mode normal --min_fasta_length 1000
```

## Cleaning

```
#rm *fq
```

----
----
----
----

