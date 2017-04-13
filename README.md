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

