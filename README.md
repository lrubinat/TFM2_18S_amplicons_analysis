# TFM2_18S_amplicons_analysis

Original dataset:
	- Folder name: "Malaspina_18S_Surface"
	- Content: Malaspina 18S iTags; 99% Uparse clustering; 124 stations; surface waters (3-5m); size fraction 0.2-3um.

		--> blastn_vs_BM_V4_vgb203_evalue10min4.txt (/share/data/protist/DB/DB_Biomarks/new_DB?)
		--> blastn_vs_MAS_V4_v9059_evalue10min4.txt (/share/data/databases/PROTIST_MAS_V4_9059/MAS_V4_9059?)
		--> blastn_vs_SILVA_v119.1_evalue10min4 (/share/data/databases/SILVA/SILVA119.1?)
		--> otu_table99_MalaspinaSurf18S_v2.xlsx
		--> otus99_repset_clean.fa

#####################################

1) OTU table curation

	- Original dataset: otu_table99_MalaspinaSurf18S_v2.xlsx [56117 125]
	- Translation into txt:
		--> otus_18S_nohyphen.txt [56117 125]

	1.1. Remove repeated OTUs:
		
		$ python3 remove_SILVA_BLAST_repeated_otus.py 
			--> input: blastn_vs_SILVA_v119.1_evalue10min4 [56358 hits]
			--> output: BLAST_on_SILVA_unique_otus.txt [55997 hits]

		$ python3 remove_MAS_BLAST_repeated_otus.py
			--> input: blastn_vs_MAS_V4_v9059_evalue10min4.txt [55764 hits]
			--> output: BLAST_on_MAS_unique_otus.txt [55669 hits]

		$ python3 remove_BM_BLAST_repeated_otus.py
			--> input: blastn_vs_BM_V4_vgb203_evalue10min4.txt [56342 hits]
			--> output: BLAST_on_BM_unique_otus.txt [55948 hits]


	1.2. Identify BLAST hits with similarity 96% and coverage >= 0.7:

		$ python3 A_filter_SILVA_OTUs.py --> BLAST_SILVA_filtered_OTUs.txt [14456 out of 55997 don't pass the filter]
		$ python3 B_filter_MAS_OTUs.py --> BLAST_MAS_filtered_OTUs.txt [13042 out of 55669 don't pass the filter]
		$ python3 C_filter_BM_OTUs.py --> BLAST_BM_filtered_OTUs.txt [12295 out of 55948 don't pass the filter]

		(onliner used to count excluded hits: $ grep "NA" BLAST_SILVA_filtered_OTUs.txt | awk '$4!=$8{print}{}' | wc -l)


	1.3. Add OTUs classification in the OTU table:

		$ python3 D_add_OTUtb_classif.py --> D_add_classif.out

	1.4. Exclude metazoas:

		$ grep -v -E "Metazoa|metazoa" D_add_classif.out > E_OTUtb_nometazoa.txt

		--> E_OTUtb_nometazoa.out [55472 OTUs]


	1.5. Delete non-classified OTUs:

		$ python3 F_remove_nonclassif_OTUs.py

		--> F_OTUtb_classif.out [46808 OTUs]

		($ awk '{print $NF}' F_OTUtb_classif.out | gsort -V | uniq -c | gsort -nr)

		32300 NA
		12056 Dinophyceae
		 458 Pelagophyceae
		 426 Chrysophyceae
		 275 Dictyochophyceae
		 261 Prasinophyceae
		 160 MOCH-2
		 148 Prasinophyceae_clade-VII
		 124 Bacillariophyceae
		 106 Cryptophyceae
		  90 Prasinophyceae_clade-IX
		  86 MOCH-1
		  83 Chlorarachniophyceae
		  78 Prymnesiophyceae
		  40 Mamiellophyceae
		  29 Pinguiophyceae
		  28 MOCH-5
		  18 Bolidophyceae
		  15 other_Prasinophyceae
		  11 Eustigmatales
		   6 Pyramimonadaceae
		   4 Raphydophyceae
		   3 Ulvophyceae
		   3 Trebouxiophyceae


	1.6. Add header with samples complete name and taxonomic classification columns

		$ python3 G_add_complete_station_names_header.py
		$ cat G_complete_st_names_header.out F_OTUtb_classif.out > OTUtb_18S_curated_46808_OTUs.txt
