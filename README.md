# MedChem307
**Softwares** 

- **sequenceserver v3.1.0**: https://github.com/wurmlab/sequenceserver
- **SPAdes v3.15.5**: https://github.com/ablab/spades
- **sratoolkit v2.10.9**: https://github.com/ncbi/sra-tools
- **kallisto v0.46.0**: https://github.com/pachterlab/kallisto
- **trimgalore v0.6.7**: https://github.com/FelixKrueger/TrimGalore
- **EnTAP v2.0.0**: https://gitlab.com/PlantGenomicsLab/EnTAP 
- **Transeq**: https://www.ebi.ac.uk/jdispatcher/st/emboss_transeq (https://doi.org/10.1093/nar/gkae241)

The following scripts were submitted to a high performance computing cluster via SLURM (https://github.com/schedmd/slurm). All softwares above were pre-installed under a Bioinformatics module on the computing cluster except orfipy, which was installed as described below. Sequenceserver was used as a cloud-based version (https://sequenceserver.com/).

**1a. SRA-download-initial-run**
- Configure your cluster for sratools (v2.10.9) commands by running this script on the initial run.
- Target RNA-seq datasets for de novo transcriptome assembly should be primarily paired-end data. To download an RNA-seq dataset from the NCBI sequence read archive (SRA), run the script:
```
#!/bin/bash
#SBATCH --job-name=sra-download-start
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=04:00:00
#SBATCH --mem=10g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./sratools-%j
source /etc/profile.d/http_proxy.sh
module load Bioinformatics
module load sratoolkit/2.10.9-udmejx7
vdb-config --interactive
fasterq-dump SRA# --split-files
```
**1b. SRA-download**
- Run this script after the initial run.
- Target RNA-seq datasets for de novo transcriptome assembly should be primarily paired-end data. To download an RNA-seq dataset from the NCBI sequence read archive (SRA), run the script:
```
#!/bin/bash
#SBATCH --job-name=sra-download
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=04:00:00
#SBATCH --mem=10g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./sratools-%j
source /etc/profile.d/http_proxy.sh
module load Bioinformatics
module load sratoolkit/2.10.9-udmejx7
fasterq-dump SRA# --split-files
```
**1c. Batch SRA-download**
- For large-scale transcriptome mining, SRA datasets are downloaded in batches of up to 100 datasets per one directory with the following script. SRA accession numbers of target data are listed in an SRA.txt file with one SRA accession number per line.
```
#!/bin/bash
#SBATCH --job-name=sra-download
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./sratools-%j
source /etc/profile.d/http_proxy.sh
module load Bioinformatics
module load sratoolkit/2.10.9-udmejx7
sed -i 's/\r$//' SRA.txt &
cat SRA.txt |parallel -j 10 xargs -n 25 -P 0 fasterq-dump --split-files --outdir /path/to/directory/ & jobs -l
wait
printf "\n...done\n\n"
```
**2a. Trimming - paired-end dataset**
- Trimming improves transcript assembly and gene detection. To trim an RNA-seq dataset with TrimGalore (v0.6.7) default settings, run the script:
```
#!/bin/bash
#SBATCH --job-name=trimgalore
#SBATCH –-account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
trim_galore --cores 8 --paired ./SRA#_1.fastq ./ SRA#_2.fastq
```
**2b. Trimming - single-end dataset**
- Trimming improves transcript assembly and gene detection. To trim an RNA-seq dataset with TrimGalore (v0.6.7) default settings, run the script:
```
#!/bin/bash
#SBATCH --job-name=trimgalore
#SBATCH –-account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
trim_galore --cores 8 ./SRA#.fastq
```
**2c. Batch trimming - paired-end data**
- Trimming improves transcript assembly and gene detection. Paired-end RNA-seq data is trimmed with TrimGalore (v0.6.7) default settings in batches of 100 datasets.
- Generate directories for fwd reads and rev reads:
```
mkdir input_data_1
mkdir input_data_2
```
- Move fwd reads to input_data_1/ directory:
```
mv *_1.fastq input_data_1
```
- Move rev reads to input_data_2/ directory:
```
mv *_2.fastq input_data_2
```
- Run the batch TrimGalore-trimming script:
```
#!/bin/bash
#SBATCH --job-name=trimgalore
#SBATCH --account=your_account
#SBATCH --partition=standard 
#SBATCH --array=1-(insert-number-of-datasets-in-input_data_1_directory)
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
file1=$(ls ./input_data_1/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
file2=$(ls ./input_data_2/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
trim_galore --cores 8 --paired ./input_data_1/${file1} ./input_data_2/${file2}
```
**3. Transcriptome assembly** 
- SPAdes is used for de novo transcriptome assembly from trimmed RNA-seq datasets.

**3a. SPAdes – paired-end datasets**
```
#!/bin/bash
#SBATCH --job-name=SPAdes
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./SPAdes-%j
module load Bioinformatics
module load spades/3.15.5-jhe6qq2
spades.py --rna -1 ./SRA#_1_val_1.fq -2 ./SRA#_2_val_2.fq -o spades_SRA#
cd spades_SRA#
mv transcripts.fasta spades_SRA#.fasta
```
**3b. SPAdes – single-end datasets**
```
#!/bin/bash
#SBATCH --job-name=SPAdes
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./SPAdes-%j
module load Bioinformatics
module load spades/3.15.5-jhe6qq2
spades.py --rna -s ./SRA#_trimmed.fq -o spades_SRA#
cd spades_SRA#
mv transcripts.fasta spades_SRA#.fasta
```
**3c. SPAdes - batch assembly of multiple datasets**
- Generate directories for trimmed fwd reads and trimmed rev reads:
```
mkdir input_data_trimmed_1
mkdir input_data_trimmed_2
```
- Move trimmed fwd reads to input_data_trimmed_1/ directory:
```
mv *_1.fq /path/to/input_data_trimmed_1/
```
- Move trimmed rev reads to input_data_trimmed_2/ directory:
```
mv *_2.fq /path/to/input_data_trimmed_2/
```
- Run SPAdes batch assembly:
```
#!/bin/bash
#SBATCH --job-name=SPAdes
#SBATCH --account=your_account 
#SBATCH --partition=standard
#SBATCH --array=1-(insert-number-of-datasets-in-input_data_trimmed_1_directory)
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@mail.com
#SBATCH --mail-type=END
#SBATCH --output=./SPAdes-%j
module load Bioinformatics
module load spades/3.15.5-jhe6qq2
file1=$(ls ./input_data_trimmed_1/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
file2=$(ls ./input_data_trimmed_2/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
spades.py --rna -1 ./input_data_trimmed_1/${file1} -2 ./input_data_trimmed_2/${file2} -o spades_$file1
cd spades_$file1
mv transcripts.fasta spades_$file1\.fasta
```
**4. BLAST database formatting** 
- BLAST database generation of transcriptome assemblies by Sequenceserver requires reduction of fasta headers to less than 51 letters. The following script was used for SPAdes-specific assembly fasta file formatting for Sequenceserver-based BLAST database generation and commandline PHI-BLAST search.
```
#!/bin/bash
#SBATCH --job-name=rename
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=12:00:00
#SBATCH --mem=10g
#SBATCH --mail-user=your@mail.com
#SBATCH --mail-type=END
#SBATCH --output=./rename-%j

for i in *fasta; do n="${i%.fasta}"; sed -i.bak "s/>[^_]\+/>$n/" $i; done
cat *fasta
sed -i 's/_length//g' *fasta
sed -i 's/_cov//g' *fasta
sed -i 's/_g//g' *fasta
sed -i 's/_i//g' *fasta
```
**5. Kallisto read quantification**
- Read quantification in a given assembled transcriptome is done with kallisto from the trimmed read files.
-	First the an assembled *de novo* transcriptome is indexed with the following kallisto-index script:
  
**Kallisto indexing**
```
#!/bin/bash
#SBATCH --job-name=kallisto-index-SRA#
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@mail.com
#SBATCH --mail-type=END
#SBATCH --output=./kallisto-index-%j
module load Bioinformatics
module load kallisto/0.46.0
kallisto index -i SRA#.idx spades_SRA#.fasta
```

**Kallisto quantification**
- Quantify reads of each paired trimmed fastq dataset (SRA#) against indexed assembled transcriptome with the following kallisto script:
```
#!/bin/bash
#SBATCH --job-name=kallisto-quant-SRA#
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=7
#SBATCH --time=24:00:00
#SBATCH --mem=48g
#SBATCH --mail-user=your@mail.com
#SBATCH --mail-type=END
#SBATCH --output=./kallisto-quant-%j
module load Bioinformatics
module load kallisto/0.46.0
kallisto quant -i SRA#.idx -o output_SRA# SRA#_1_val_1.fq SRA#_2_val_2.fq
```

**6.EnTAP - gene function annotation**
- Annotate the transcript sequences in a fasta file with EnTAP. EnTAP requires the installation and configuration of a reference database such as Uniprot. Please refer to EnTAP gitlab (https://gitlab.com/PlantGenomicsLab/EnTAP) for database installation.
```
#!/bin/bash
#SBATCH --job-name=entap
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --time=24:00:00
#SBATCH --mem=14g
#SBATCH --mail-user=your@mail.com
#SBATCH --mail-type=END
#SBATCH --output=./entap-%j
module load Bioinformatics
module load EnTAP/2.0.0
EnTAP --runN --state 4x --run-ini ./entap_run.params --entap-ini ./entap_config.ini
```
- An entap_config.ini file and a entap_run.params file are required in the EnTAP directory to run EnTAP.
- Generate entap_config.ini:
```
nano entap_config.ini
```
with following input and save:
```
data-generate=false
data-type=0,
entap-db-bin=/sw/pkgs/arc/EnTAP/2.0.0/db/bin/entap_database.bin
entap-graph=/sw/pkgs/arc/EnTAP/2.0.0/bin/entap_graphing.py
rsem-calculate-expression=/sw/pkgs/med/RSEM/1.3.3/bin/rsem-calculate-expression
rsem-sam-validator=/sw/pkgs/med/RSEM/1.3.3/bin/rsem-sam-validator
rsem-prepare-reference=/sw/pkgs/med/RSEM/1.3.3/bin/rsem-prepare-reference
convert-sam-for-rsem=/sw/pkgs/med/RSEM/1.3.3/bin/convert-sam-for-rsem
transdecoder-long-exe=/sw/pkgs/arc/EnTAP/2.0.0/bin/libs/TransDecoder-TransDecoder-v5.7.1/TransDecoder.LongOrfs
transdecoder-predict-exe=/sw/pkgs/arc/EnTAP/2.0.0/bin/libs/TransDecoder-TransDecoder-v5.7.1/TransDecoder.Predict
interproscan-exe=interproscan.sh
diamond-exe=/sw/pkgs/arc/EnTAP/1.0.1/bin/libs/bin/diamond
```
- For Uniprot database annotation (fast, incomplete rich EnTAP annotation), generate the following	entap_run.params:
```
fpkm=0.5
align=
transdecoder-no-refine-starts=false
out-dir=entap_outfiles
overwrite=false
resume=false
input=/scratch/rkersten_root/rkersten1/rkersten/MC307/spades_SRR8195628.fasta
database=/nfs/turbo/phar-rkersten/Uniprot/bin/uniprot_sprot.dmnd
no-trim=false
threads=1
output-format=1,3,4,7,
hgt-donor=
hgt-recipient=
hgt-gff=
ontology_source=0,
eggnog-contaminant=false
eggnog-dbmem=false
eggnog-sensitivity=more-sensitive
interproscan-db=
qcoverage=50
tcoverage=50
e-value=1e-05
uninformative=conserved,predicted,unknown,unnamed,hypothetical,putative,unidentified,uncharacterized,uncultured,uninformative,
diamond-sensitivity=very-sensitive
```
- For NCBI nr database annotation (long, rich EnTAP annotation), generate	the following entap_run.params:
```
fpkm=0.5
align=
transdecoder-no-refine-starts=false
out-dir=entap_outfiles
overwrite=false
resume=false
input=/scratch/rkersten_root/rkersten1/rkersten/MC307/spades_SRR8195628.fasta
database=/nfs/turbo/phar-rkersten/nr_database.dmnd
no-trim=false
threads=1
output-format=1,3,4,7,
hgt-donor=
hgt-recipient=
hgt-gff=
ontology_source=0,
eggnog-contaminant=false
eggnog-dbmem=false
eggnog-sensitivity=more-sensitive
interproscan-db=
qcoverage=50
tcoverage=50
e-value=1e-05
uninformative=conserved,predicted,unknown,unnamed,hypothetical,putative,unidentified,uncharacterized,uncultured,uninformative,
diamond-sensitivity=very-sensitive
```
