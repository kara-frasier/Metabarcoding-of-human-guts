
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

 ./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-2 import_fastp
 
 ./fastp-single.sh 120 /tmp/gen711_project_data/FMT_3/fmt-tutorial-demux-1 import_fastp2
 
conda activate qiime2-2022.8

qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path import_fastp --output-path import_qiime

qiime tools import --type "SampleData[SequencesWithQuality]" --input-format CasavaOneEightSingleLanePerSampleDirFmt --input-path import_fastp2 --output-path import_qiime

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

# Metadata and Background
qiime taxa barplot --i-table feature_table.qza --i-taxonomy FMT-taxonomy.qza --m-metadata-file sample-metadata.tsv --o-visualization barplot-1.qzv

qiime taxa barplot \
    --i-table feature_table.qza \
    --i-taxonomy FMT-taxonomy.qza \
    --m-metadata-file sample-metadata.tsv \
    --o-visualization barplot-1.qzv

qiime taxa barplot --i-table feature_table.qza --i-taxonomy FMT-taxonomy.qza --m-metadata-file sample-metadata.tsv --o-visualization barplot-1.qzv

# Filtered Phylogenetic Tree
qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs.qza \
  --o-alignment alignments \
  --o-masked-alignment masked-alignment \
  --o-tree unrooted-tree \
  --o-rooted-tree rooted-tree \
  --p-n-threads 4
  
 qiime diversity core-metrics-phylogenetic \
  --i-phylogeny rooted-tree.qza \
  --i-table feature_table.qza \
  --p-sampling-depth 500 \
  --m-metadata-file sample-metadata.tsv \
  --p-n-jobs-or-threads 4 \
  --output-dir core-metrics
  
qiime feature-table relative-frequency \
  --i-table core-metrics/rarefied_table.qza \
  --o-relative-frequency-table core-metrics/relative_rarefied_table
  
qiime diversity pcoa-biplot \
  --i-features core-metrics/relative_rarefied_table.qza \
  --i-pcoa core-metrics/unweighted_unifrac_pcoa_results.qza \
  --o-biplot core-metrics/unweighted_unifrac_pcoa_biplot
  
 qiime emperor biplot \
  --i-biplot core-metrics/unweighted_unifrac_pcoa_biplot.qza \
  --m-sample-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics/unweighted_unifrac_pcoa_biplot
  
qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metrics/shannon_vector.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization core-metrics/alpha-group-significance

qiime diversity beta-group-significance \
  --i-distance-matrix core-metrics/unweighted_unifrac_distance_matrix.qza \
  --m-metadata-file sample-metadata.tsv \
  --m-metadata-column treatment-group \
  --p-pairwise \
  --o-visualization core-metrics/unweighted_unifrac-beta-group-significance
  
# Plots
![image](https://github.com/kara-frasier/Metabarcoding-of-human-guts/assets/130784549/251dd70b-1cb6-4297-905c-3587cdeaa7f0)
![image](https://github.com/kara-frasier/Metabarcoding-of-human-guts/assets/130784549/d47aa2d7-8f41-47d6-9a44-d7c7814c7980)


  
