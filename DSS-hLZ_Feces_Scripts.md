
Change directories to access unzipped fastq file. Lines 1 and 2 compile names and barcodes; only run these lines once per study.

Line 1 runs quickly and ensures any new files are saved to this location. Make sure that all the files you need are located in this directory.

```cd /home/qiime/Desktop/Shared_Folder/IlluminaFilesDSShLZ_CDRFAug2015/```

Line 2 grabs the sequencing data. This also runs quickly.

```cat DSS-hLZ_S1_L001_R1_001.fastq | perl -ne '$h1=$_; $s=<>; $h2=<>; $q=<>; print $h1.substr($s,0,8)."\n".$h2.substr($q,0,8)."\n"' > barcode001_DSS-hLZ.txt```

Line 3 checks the quality of the mapping file, but you may need to adjust as needed. The mapping file must be located in the current directory. If there are errors in the mapping file, check to make sure there are no spaces and that no sections are too long. Ensure that the mapping file is saved as tab delimited. Assuming that everything is set up correctly, this line should run quickly.

```validate_mapping_file.py -m MappingFiles_DSS-hLZ_Feces.txt -o mapping_output```

Line 4 matches samples in the Mapping file to barcodes in the library. Creates a new output file. This line runs quickly.

```split_libraries_fastq.py -i DSS-hLZ_S1_L001_R1_001.fastq -o split_library_output_DSS-hLZ_Feces -b barcode001_DSS-hLZ.txt -m MappingFiles_DSS-hLZ_Feces.txt --barcode_type 8```

Line 5 takes a few minutes; assigns OTUs based on Greengenes reference database.

```pick_open_reference_otus.py -a -O 8 -i /home/qiime/Desktop/Shared_Folder/IlluminaFilesDSShLZ_CDRFAug2015/split_library_output_DSS-hLZ_Feces/seqs.fna -o /home/qiime/Desktop/Shared_Folder/IlluminaFilesDSShLZ_CDRFAug2015/split_library_output_DSS-hLZ_Feces/otus -r /home/qiime/qiime_software/gg_otus-13_8-release/rep_set/97_otus.fasta```

Line 6 will take hours, so set this to run overnight. It builds alpha rarefaction plots. Alpha diversity is a measure of diversity based on the number of new OTUs found as the number of sequences/samples increases (qualitative), or the relative abundance of OTUs within each sample (quantitative). It essentially helps gauge species richness.

```alpha_rarefaction.py -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -o diversity/ -m MappingFiles_DSS-hLZ_Feces.txt -t split_library_output_DSS-hLZ_Feces/otus/rep_set.tre```

Line 7 produces a results summary with the number of reads found that correspond to each sample. Use this to gauge the minimum number of reads that you want the program to analyze. This line runs quickly.

```biom summarize-table -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -o split_library_output_DSS-hLZ_Feces/otus/results_summary.txt```

Line 8 takes approximately one hour. Forms a workflow which can be used for subsequent diversity analyses. Must specify the minimum number of reads you wish to be analyzed, as determined from the previous line.

```core_diversity_analyses.py -o diversity/core_div_2500/ -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -m MappingFiles_DSS-hLZ_Feces.txt -t split_library_output_DSS-hLZ_Feces/otus/rep_set.tre -e 2500```

Line 9 creates a summary of the taxonomic groups represented in each sample, in the form of a table. This line runs quickly.

```summarize_taxa.py -o summarize_taxa_map/ -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -m MappingFiles_DSS-hLZ_Feces.txt``` 

Line 10 produces relative abundance barcharts with samples grouped together from a specified column from the mapping file (as opposed to individual samples). 

```summarize_taxa_through_plots.py -o summarize_taxa_plots_TimeTrtmtDiet/ -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -m MappingFiles_DSS-hLZ_Feces.txt -c TimeTrtmtDiet```

Line 11 compares OTU frequencies across sample groups and determines whether there are statistically significant differences in OTU abundance in different sample groups.

```group_significance.py -i split_library_output_DSS-hLZ_Feces/otus/otu_table_mc2_w_tax_no_pynast_failures.biom -m MappingFiles_DSS-hLZ_Feces.txt -o cat_sig -s g_test -c TimeTrtmtDiet```

