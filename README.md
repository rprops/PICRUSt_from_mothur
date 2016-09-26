# PICRUSt_from_mothur
Guidelines for running PICRUSt with mothur biom output

Tested with mothur v1.37
# Required files:
Reclassified otu taxonomy using greengenes: 
```R
stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.0.03.cons.taxonomy
```
Shared file: 
```R
stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared
```
Reference taxonomy downloaded from mothur website (both 13_8 and 13_5 will work with PICRUSt): 
```R
gg_13_8_99.gg.tax
```
Mapping file (gg IDs) is retrieved from PICRUSt website: 
```R 97_otu_map.txt
```

#Create mothur biom file

First make biom file in mothur.
```R
make.biom(shared=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.shared, label=0.03, reftaxonomy=gg_13_8_99.gg.tax, constaxonomy=stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.0.03.cons.taxonomy, picrust=97_otu_map.txt)
```
Check if the biom is one of the correct version (mothur will produce one of 0.9.1 while you need 1.0.0).
```R
biom validate-table -i stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.0.03.biom
```
Output:
```R
Invalid format 'Biological Observation Matrix 0.9.1', must be '1.0.0'
Timestamp does not appear to be ISO 8601
The input file is not a valid BIOM-formatted file.
```
# Convert biom to 1.0.0 version 
Code requires biom 2.x.x or higher (this is standard version, to check version: biom show-install-info):
```R
biom convert --table-type="OTU table" -i stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.pick.an.unique_list.0.03.biom -o your_OTU_table.txt --to-tsv --header-key taxonomy
biom convert -i your_OTU_table.txt -o your_OTU_table.biom --table-type="OTU table" --to-json --process-obs-metadata taxonomy
```
Check again for correct version.
```R
biom validate-table -i your_OTU_table.biom
```

# Generate 16S copy number normalized OTU table

Run the PICRUSt command for copy number normalization of the OTU table.
Code requires biom 1.3.1 (contact administrator (FM) to quickly change version, PICRUSt runs on biom 1.3.1):
```R
normalize_by_copy_number.py -i your_OTU_table.biom -o normalized_otus.biom
```

Convert the output biom file to a standard OTU table file.
Code requires biom 2.x.x (contact administrator to quickly change version back to 2.x, format your normalized biom file to a readable table format):
```R
biom convert --table-type="OTU table" -i normalized_otus.biom -o normalized_otus_table.txt --header-key taxonomy --to-tsv
biom convert -i normalized_otus.biom -o normalized_otus_table.txt  --table-type "otu table"
```
# Calculate NSTI scores
Checking whether your OTUs are closely matching with exisisting reference genomes. NSTI scores are explained here: https://picrust.github.io/picrust/tutorials/quality_control.html#basic-qc-checklist. NSTI scores are provided in the file nsti.txt (you have to create a blank .txt file for this). In general <0.03 is good.
```R
predict_metagenomes.py -f -i normalized_otus.biom -o predicted_metagenomes.txt -a nsti.txt
```
