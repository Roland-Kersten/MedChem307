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

# Transcriptome assembly – single dataset
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
#SBATCH --time=01:00:00
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
#SBATCH --job-name=sra-download-start
#SBATCH --account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --time=01:00:00
#SBATCH --mem=10g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./sratools-%j
source /etc/profile.d/http_proxy.sh
module load Bioinformatics
module load sratoolkit/2.10.9-udmejx7
fasterq-dump SRA# --split-files
```
**2. Trimming - paired-end datasets**
- Trimming improved precursor peptide assembly and core peptide detection in our study. To trim an RNA-seq dataset with TrimGalore (v0.6.7) default settings, run the script:
```
#!/bin/bash
#SBATCH --job-name=trimgalore
#SBATCH –-account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
trim_galore --cores 4 --paired ./SRA#_1.fastq ./ SRA#_2.fastq
```
**2. Trimming - single-end datasets**
- Trimming improved precursor peptide assembly and core peptide detection in our study. To trim an RNA-seq dataset with TrimGalore (v0.6.7) default settings, run the script:
```
#!/bin/bash
#SBATCH --job-name=trimgalore
#SBATCH –-account=your_account
#SBATCH --partition=standard
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
trim_galore --cores 4 ./SRA#.fastq
```
**3. Transcriptome assembly** - *Three assembler softwares were used for de novo transcriptome assembly from trimmed RNA-seq datasets.*
- **a. SPAdes – paired-end datasets**
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
- **b. SPAdes – single-end datasets**
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
