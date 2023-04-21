
# Metabarcoding-of-human-guts
Fecal microbiota transplant (FMT) study. Metabarcoding of human guts.
# Name:
Kara Frasier
#Code 04/21
conda activate genomics
cd Metabarcoding-of-human-guts/
touch testfile.txt
echo 'a test' > testfile.txt
git add testfile.txt
git commit -m '...'

cp  /tmp/gen711_project_data/fastp-single.sh ~/fastq-single.sh
chmod +x ~/fastq-single.sh

conda activate qiime2-2022.8
qiime tools import --type "SampleData[PairedEndSequencesWithQuality]"  --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path trimmed_fastq --output-path trimmed_fastq/imported_files 
