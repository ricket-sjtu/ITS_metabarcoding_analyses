# ITS_metabarcoding_analyses
Meghan's ITS pipeline


## Database Download from http://qiime.org/home_static/dataFiles.html

mkdir its_database && cd its_database

wget 'https://github.com/downloads/qiime/its-reference-otus/its_12_11_otus.tar.gz'
#
##### Be sure to unzip the database, all of them
#
### Make blast database for taxonomy assignment (go into database rep set) only if using joes taoxnomy assignment
makeblastdb -dbtype nucl -in 99_otus.fasta -out 99_otus
#
## Make fresh directory for raw reads, all reads should be gzipped at this point.

mkdir raw_reads & cd raw_reads

mv /path/to/project/dir/*/*R1* ./
mv /path/to/project/dir/*/*R3* ./
#
## rename files to this format --> SampleName_barcode_L001_R1_001.fastq.gz
##### for your data I simply needed to fix the ones with ITS in it. (remove the -n to actually make the change)
rename -v -n 's/(.*)-(.*)__(.*)/$1$2_$3/' *
#
## Merge reads, add forward reads that did not merge to merged reads
cd raw_reads
join_reads_and_prep_for_qiime.py ./



## Truncate primers

truncate_reverse_primer.py -f seqs.fna -m mapping_file.txt -o truncate_reverse_primers

## Open Reference OTU Picking

pick_open_reference_otus.py -i seqs.fna -o pick_references_output_open -r /home/genome/joseph7e/MEG_its_test/its_database/rep_set/97_otus.fasta -p parameters

### Run qiime2 enviornment



## IMPORT TO QIIME 2 table
qiime tools import \
  --input-path ../otu_table_mc2.biom \
  --type "FeatureTable[Frequency]" \
  --source-format BIOMV210Format \
  --output-path feature-table-from_qiime1.qza

### Import to qiime 2 taxonomy

qiime tools import \
  --type "FeatureData[Taxonomy]" \
  --input-path ../uclust_assigned_taxonomy/rep_set_tax_assignments.txt \
  --output-path taxonomy_from_qiime1

### Summarize data in table
qiime feature-table summarize \
  --i-table feature-table-from_qiime1.qza \
  --o-visualization table.qzv \
  --m-sample-metadata-file ../../../Microbiome_ITS_Mapping_File.csv
  
  
### Summarize through barplots
qiime taxa barplot \
  --i-table feature-table-from_qiime1.qza \
  --i-taxonomy taxonomy_from_qiime1.qza \
  --m-metadata-file Microbiome_ITS_Mapping_File.tsv \
  --o-visualization taxa-bar-plots.qzv



## Create a tree

## Import the rrep set form qiime2

qiime tools import \
  --type "FeatureData[Sequence]" \
  --input-path ../rep_set.fna \
  --output-path rep_set_from_qiime1

qiime alignment mafft \
  --i-sequences rep_set_from_qiime1.qza \
  --o-alignment aligned-rep-seqs.qza

qiime alignment mask \
  --i-alignment aligned-rep-seqs.qza \
  --o-masked-alignment masked-aligned-rep-seqs.qza


qiime phylogeny fasttree \
  --i-alignment masked-aligned-rep-seqs.qza \
  --o-tree unrooted-tree.qza


### root the tree

qiime phylogeny midpoint-root   --i-tree tree_unrooted_from_qiime1.qza   --o-rooted-tree rooted-tree.qza

### Rareify this shit

my_numbers : 6092, 2130, 10065


sampling_depth=6092

qiime diversity core-metrics \
  --i-phylogeny rooted-tree.qza \
  --i-table feature-table-from_qiime1.qza \
  --p-sampling-depth $sampling_depth \
  --output-dir core-metric_$sampling_depth


### ALpha diversity

qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metric_$sampling_depth/faith_pd_vector.qza \
  --m-metadata-file Microbiome_ITS_Mapping_File.tsv \
  --o-visualization core-metric_$sampling_depth/faith-pd-group-significance.qzv

qiime diversity alpha-group-significance \
  --i-alpha-diversity core-metric_$sampling_depth/evenness_vector.qza \
  --m-metadata-file Microbiome_ITS_Mapping_File.tsv \
  --o-visualization core-metric_$sampling_depth/evenness-group-significance.qzv
  
qiime diversity alpha-correlation \
  --i-alpha-diversity core-metric_$sampling_depth/faith_pd_vector.qza \
  --m-metadata-file Microbiome_ITS_Mapping_File.tsv \
  --o-visualization core-metric_$sampling_depth/faith-pd-correlation.qzv
  
qiime diversity alpha-correlation \
  --i-alpha-diversity core-metric_$sampling_depth/evenness_vector.qza \
  --m-metadata-file Microbiome_ITS_Mapping_File.tsv \
  --o-visualization core-metric_$sampling_depth/evenness-correlation.qzv  
