# ITS_metabarcoding_analyses
Meghan's ITS pipeline


## Database Download from http://qiime.org/home_static/dataFiles.html

mkdir its_database && cd its_database

wget 'https://github.com/downloads/qiime/its-reference-otus/its_12_11_otus.tar.gz'



##### Be sure to unzip the database, all of them



### Make blast database for taxonomy assignment




## Make fresh directory for raw reads, all reads should be gzipped at this point.

mkdir raw_reads & cd raw_reads

mv /path/to/project/dir/*/*R1* ./
mv /path/to/project/dir/*/*R3* ./




## rename files to this format --> SampleName_barcode_L001_R1_001.fastq.gz
##### for your data I simply needed to fix the ones with ITS in it. (remove the -n to actually make the change)
rename -v -n 's/(.*)-(.*)__(.*)/$1$2_$3/' *



## Merge reads, add forward reads that did not merge to merged reads



