
# Metabarcoding-of-human-guts
Fecal microbiota transplant (FMT) study. Metabarcoding of human guts.
# Name:
Kara Frasier
# Code 
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

# Cutadapt
qiime cutadapt trim-single --i-demultiplexed-sequences import_qiime.qza --p-front TACGTATGGTGCA --p-discard-untrimmed --p-match-adapter-wildcards --verbose --o-trimmed-sequences import_cutadapt

qiime demux summarize --i-data import_cutadapt.qza --o-visualization import_demux

# Denoising
qiime dada2 denoise-single --i-demultiplexed-seqs import_cutadapt.qza --p-trunc-len 125 --p-trim-left 0 --p-n-threads 4 --o-denoising-stats denoising-stats.qza --o-table feature_table.qza --o-representative-sequences rep-seqs.qza

qiime metadata tabulate --m-input-file denoising-stats.qza --o-visualization denoising-stats.qzv

qiime feature-table tabulate-seqs --i-data rep-seqs.qza --o-visualization rep-seqs.qzv
# Taxonomy
qiime feature-classifier classify-sklearn --i-classifier /tmp/gen711_project_data/reference_databases/classifier.qza --i-reads rep-seqs.qza --o-classification FMT-taxonomy.qza

qiime taxa barplot --i-table feature_table.qza --i-taxonomy FMT-taxonomy.qza --o-visualization barplot-1.qzv
