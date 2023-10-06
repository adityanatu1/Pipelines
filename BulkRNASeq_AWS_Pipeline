#!/bin/bash


##INTENDED USE: Described Below
#Loads data from the S3 bucket to the EC2 instance using the aws s3 sync command.
#Activates the Conda environment where Trimmomatic is installed.
#Runs Trimmomatic on the fastq.gz files.
#Uses a loop to unzip all the fastq.gz files in the directory.
#Deactivates the Conda environment.
#Activates the Conda environment where STAR is installed.
#Runs STAR on the trimmed fastq files.
#Deactivates the Conda environment.
#Activates the Conda environment where fcounts is installed.
#Specifies the paths to the required files.
#Runs featureCounts with the specified settings.
#Deactivates the Conda environment.

##ASSUMPTIONS: 
#1. You have a folder called Data and which holds the .fastq files from your study + the correct adapter sequence specified by sequencing facility
#2. You have the paths indicated below in each segment of the code 
#3. You assembled the indices yourself and are now ready to use and have the .fa and gtf file in your intended directories 
#4. You generate the quality report outside of this script to make sure your data passes your preferred sanity checks 
#5. Your species is Mmusculus 
#6. Every tool you have for this pipeline is using conda install and a separate conda env
#7. A specific file naming convention for the input files. It expects the R1 reads to have the pattern "_R1_001.fastq.gz" and the R2 reads to have the pattern "_R2_001.fastq.gz". Make sure your input files follow this naming convention.
#8. ALL the necessary input files are located in the "Data" folder. It will process all the R1 and R2 pairs of input files present in that folder. Make sure you have all the required input files in the specified folder.
#9. You have a working AWS CLI setup and the necessary permissions to sync data from the S3 bucket to your local directory. It also assumes that the specified S3 bucket contains the relevant input files.
#10. The code assumes that all the necessary tools (Trimmomatic, STAR, and featureCounts) are installed and configured correctly in separate conda environments. It also assumes that the conda environments have the required dependencies for each tool.
#11.The code assumes that the output files from each tool will be generated in the current working directory. Make sure you have sufficient write permissions and disk space in the directory where you run the script.


# Activate conda environment and initialize shell
eval "$(conda shell.bash hook)"

# Sync data from S3 bucket to local directory
aws s3 sync s3://2208210/ /home/ec2-user/Data

# Activate the Conda environment where Trimmomatic is installed
conda activate trimmomatic-env

for i in *_R1_001.fastq.gz
do
    R2=${i//_R1_001.fastq.gz/_R2_001.fastq.gz}
    R1paired=${i//.fastq.gz/_paired.fastq.gz}
    R1unpaired=${i//.fastq.gz/_unpaired.fastq.gz}
    R2paired=${R2//.fastq.gz/_paired.fastq.gz}
    R2unpaired=${R2//.fastq.gz/_unpaired.fastq.gz}

    # Run Trimmomatic with 90 threads (assuming it is accessible from anywhere within the activated Conda environment)
    trimmomatic PE -phred33 -threads 90 $i $R2 $R1paired $R1unpaired $R2paired $R2unpaired ILLUMINACLIP:/home/ec2-user/Data/Adapter/Adapter.fa:2:30:10:2:keepBothReads LEADING:3 TRAILING:3

    
done

# Unzip all the fastq.gz files in the directory before moving to the mapping step!
for file in *.fastq.gz; do
    gunzip "$file"
done

# Deactivate the Conda environment
conda deactivate

# Activate the Conda environment where STAR is installed
conda activate STAR-env

for i in *R1_001_paired.fastq
do
    R2=${i//R1_001_paired.fastq/R2_001_paired.fastq}
    PREFIX=${i//_R1_001_paired.fastq}

    # Run STAR with the specified settings (assuming it is accessible from anywhere within the activated Conda environment)
    STAR --outSAMstrandField intronMotif --genomeDir /home/ec2-user/Data/Genome/STAR_reference --runThreadN 90 --readFilesIn $i $R2 --outFileNamePrefix $PREFIX --outSAMtype BAM SortedByCoordinate
done

# Deactivate the Conda environment
conda deactivate

# Activate the Conda environment where fcounts is installed
conda activate fcounts

# Specify the paths to the required files
GENOME_DIR="/home/ec2-user/GTF_reference"
GTF_FILE="Mus_musculus.GRCm39.109.gtf"
OUTPUT_FILE="feature_counts.txt"

# Run featureCounts with the specified settings (assuming it is accessible from anywhere within the activated Conda environment)
featureCounts -p -T 60 -a "$GENOME_DIR/$GTF_FILE" -o "$OUTPUT_FILE" /home/ec2-user/Data/*.bam

# Deactivate the Conda environment
conda deactivate
