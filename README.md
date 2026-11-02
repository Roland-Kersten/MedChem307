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
**2b. Trimming - single-end dataset**
- Trimming improves transcript assembly and gene detection. To trim an RNA-seq dataset with TrimGalore (v0.6.7) default settings, run the script:
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
**2. Batch trimming - paired-end data**
- Trimming improves transcript assembly and gene detection. Paired-end RNA-seq data is trimmed with TrimGalore (v0.6.7) default settings in batches of 100 datasets.
- Generate directories for fwd reads and rev reads:
```
mkdir input_data_1
mkdir input_data_2
```
- Move fwd reads to input_data_1/ directory:
```
mv *_1.fastq /path/to/input_data_1/
```
- Move rev reads to input_data_2/ directory:
```
mv *_2.fastq /path/to/input_data_2/
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
#SBATCH --cpus-per-task=4
#SBATCH --time=02:00:00
#SBATCH --mem=7g
#SBATCH --mail-user=your@email.com
#SBATCH --mail-type=END
#SBATCH --output=./trimgalore-%j
module load Bioinformatics
module load trimgalore/0.6.7-ztb2tpz
file1=$(ls ./input_data_1/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
file2=$(ls ./input_data_2/ | sed -n ${SLURM_ARRAY_TASK_ID}p)
trim_galore --cores 4 --paired ./input_data_1/${file1} ./input_data_2/${file2}
```
**3. Transcriptome assembly** 
- SPAdes is used for de novo transcriptome assembly from trimmed RNA-seq datasets.
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
