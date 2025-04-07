# linking-gcn-edges-with-a-grn

# Linking GCN Edges with a GRN.
There are 78,724 known genes (19,433 protein coding) that encode 385,659 unique RNA transcripts (89,832 protein coding) (GENCODE v47; https://www.gencodegenes.org/human/stats_47.html).  Through RNAseq analysis, it is possible to measure the output of all these genes for a given sample using gene co-expression networks (GCNs). 

RNAseq sample expression profiles can be stitched together into a Gene Expression Matrix (GEM) compendium.  Then one can compare transcriptome groups in the GEMs to identify differentially expressed genes (DEGs) or genes that are co-expressed (GCN).  Any of these monogenic or bigenic expression differences imply that the genes are differentially regulated via condition-specific gene regulatory networks (GRNs). 

Fortunately, tissue specific GRNs are being published for human samples.  For example, many human GRNs are described in Sonawane et al. “Understanding Tissue-Specific Gene Regulation Article Understanding Tissue-Specific Gene Regulation. 1077-1088 (2017).” These GRNs can be mapped to differentially regulated genes in a tissue specific context. This map of normal gene relationships can be mapped to aberrant gene expression states in non-wild-type samples and broken regulatory machinery can be identified. 

In this lab, we will link the genes in a human GCN with a tissue specific human GRN to generate gene regulation mechanism hypotheses.  In essence, one can use these mapping to explain why a GCN edge exists in the first place.  One twist is that we will need to rename genes in the GRN.  To achieve this, we will learn how to use a simple command line database, another useful Linux skill. How fun!!!

# Objectives
```
Obtain and preprocess a GRN and GCN file.
Visualize the network in Cytoscape.
Install sqlite3 database software.
Load the relevant tables into a sqlite3 database.
Perform SQL SELECT queries to build a preprocessed merged directional GRN and unidirectional GCN network.
```

# Step A. Download that GRN files.

Example gene regulatory networks (GRNs) can be found in Sonawane, A. R. et al. Understanding Tissue-Specific Gene Regulation Article Understanding Tissue-Specific Gene Regulation (2017). This comma-separated value (CSV) file maps the GRN genes with target genes for various human tissues.  Obtain this file using this command from this site: 

```
https://www.cell.com/cms/10.1016/j.celrep.2017.10.001/attachment/e7309c03-e579-4119-a95e-376ab2066cbb/mmc2.csv
```

The GRNs in this file are specific for these tissues:

```
Adipose_visceral
Adrenal_gland
Artery_aorta
Artery_coronary
Artery_tibial
Brain_basal_ganglia
Brain_cerebellum
Brain_other
Breast
Colon_sigmoid
Colon_transverse
Esophagus_mucosa
Esophagus_muscularis
Fibroblast_cell_line
Gastroesophageal_junction
Heart_atrial_appendage
Heart_left_ventricle
Intestine_terminal_ileum
Kidney_cortex
Liver
Lung
Lymphoblastoid_cell_line
Minor_salivary_gland
Ovary
Pancreas
Pituitary
Prostate
Skeletal_muscle
Skin
Spleen
Stomach
Testis
Thyroid
Tibial_nerve
Tissues
Uterus
Vagina
Whole_blood
```

The directed egges in these files can be found in the first and seconds columns with the tissue in the 5th column.   Note that the Transcription Factor column (TF) is an official gene symbol ID and the Transcriptional Target (TargetGene) column is an ENSEMBL ID, so you may need to translate these identifiers.  A database solution can be found below.

```
"TF","TargetGene","prior","Multiplicity","Tissues"
"AHR","ENSG00000000003","0","1","Colon_transverse,"
"ARID2","ENSG00000000003","0","2","Pituitary,Testis,"
"ARID3A","ENSG00000000003","1","0","none"
"ARNT","ENSG00000000003","0","1","Colon_transverse,"
"ARNTL","ENSG00000000003","0","1","Colon_transverse,"
"ATF1","ENSG00000000003","0","1","Testis,"
"ATF2","ENSG00000000003","0","1","Testis,"
"ATF7","ENSG00000000003","0","1","Testis,"
"ATOH8","ENSG00000000003","0","1","Colon_transverse,"
```

Use bash to reduce this file to the tissue-specific GRN and save to a file (e.g. grn.csv).

```
(head -n 1 mmc2.csv; grep 'Uterus' mmc2.csv) > grn.csv
```
 

# Step B. Build the GCN file.

Gene co-expression networks (GCN) can be found in numerous publications and databases.  Here are some examples:  

(Uterus) Hickmann A et al "Identification of condition-specific biomarker systems in uterine cancer." (2021); https://academic.oup.com/g3journal/article/12/1/jkab392/6427626
(Lung) Hang et al. "Identification of condition-specific regulatory mechanisms in normal and cancerous human lung tissue. (2022); https://link.springer.com/article/10.1186/s12864-022-08591-9
(Brain) Hang et al. "Exploration into biomarker potential of region-specific brain gene co-expression networks." (2020); https://www.nature.com/articles/s41598-020-73611-1

Download the GCN which is an undirected graph of GeneA---GeneB interactions. 

For example, here are undirected edges specific for normal uterus and uterine cancer (UCEC, UCS).  

```
https://gsajournals.figshare.com/ndownloader/files/31186971
```

You will to manually parse out the undirected GCN edge list using Excel for the tissue of interest from Supplemental Table 1. Save as a tab-delimeted file (e.g. gcn.tab). Add 'GeneA GeneB' column headers.

# Step B.Translate Gene Identifiers using Biomart Mapping Tables and an sqlite database

If the GRN contains gene target names as ENSEMBL IDs (e.g. ENSG00000278025 >>> NCR1), so we will want to change them to their corresponding official gene symbols.  To do this, we will need a gene identifier mapping table form Ensembl following these steps:

# Step B1. Obtain a geneID mapping table.

Go to https://www.ensembl.org/
Select Biomart
Select human genes

Select these attributes and save as a CSV file.  It will likely save as mart_export.txt
```
Gene stable ID
Gene stable ID version
Transcript stable ID
Transcript stable ID version
Gene name
```

# Step B2. Prepare the files for database loading.

```
cat mmc2.csv | sed 's/\"//g' | sed 's/,/\t/4;s/,/\t/3;s/,/\t/1;s/,/\t/1' > grn.tab #Convert to tab-delimited format
```

Make sure your gcn file is tab-delimited.

# Step B3: Install sqlite database software

You are probably familiar with spreadsheets like Excel.  Excel facilitates analysis of row x column data. However, when you want to find relationships between Excel spreadsheets, one need to write custom code or identify relationships in a relational database management system (rdbms).  We will use the command line friendly sqlite3 database software to identify relationships in our data.  You can find excellent help in these sqlite resources:  https://sqlite.org/cli.html and https://www.sqlitetutorial.net/.

Install sqlite3 in Linux computer using these commands:

```
PROMPT: What is sqlite? Please provide a simple overview of a database, tables, and joins.  Please also provide instructions to install sqlite on a shared linux cluster?
```

# Step B4. Start the sqlite3 shell and create the database called mapping.
```
sqlite3 mapping
```

# Step B5. Using the sqlite shell, load 3 tables into the mapping database (grn, gcn, names)
```
.separator "\t"
.import grn.tab grn
.separator "\t"
.import gcn.tab gcn
.mode csv
.import mart_export.txt names
```
#Get fields from the tables to make sure everything loaded properly.
```
.schema grn
.schema gcn
.schema names
```

# Step B6. Make an INNER JOIN to create a tab-delimited GRN file with the target gene mapped to official gene symbols.  Note that Structured Query Language (SQL) tutorials are all over the Internet.  Here is one example: https://www.w3schools.com/sql/sql_intro.asp. 

In this step we will perform SELECT..FROM, INNER JOIN, and WHERE SQL statements to relate and map the gene IDs, keeping only edges with the work ‘Kidney’ in the Tissue fields, and write the renamed data to a file.

```
.separator "\t"
.output GRN-REMAPPED.tsv
SELECT TF, names."Gene Name", Tissues, TargetGene
FROM grn
INNER JOIN names on names."Gene stable ID"=grn.TargetGene
WHERE Tissues LIKE '%Uterus%'; 
```

# Step C. Make a merged GRN and GCN edge list with bash.

Make a GRN and GCN edge files with the GRN and GCN edge labels.  Remember the GRN edges are directed (TF > target gene) edges while the GCN edges are undirected GeneA-GeneB).  Write the GRN and GCN edge lists and then merge them and remove duplicate edges.

#Obtain the edge lists and add the GCN or GRN label to each line

#Make GRN Edge File
```
cat GRN-REMAPPED.tsv| awk '{print $1,$2}' > temp
awk '{print "GRN " $0}' temp   > GRN_edges.tab
```
```
cat gcn.tab | awk '{print $1,$2}' > temp
awk '{print "GCN " $0}' temp   > GCN_edges.tab
```
#Concatenate the files
```
cat GRN_edges.tab GCN_edges.tab > merged.tissueX.gcn.grn.tab
```

#Remove duplicate lines (edges)
```
cat merged.tissueX.gcn.grn.tab | uniq | sed 's/\s/\t/g' > merged.tissueX.gcn.grn.unique.tab
```

# Step D. Make a merged GRN and GCN edge list with bash.
Make a Cytoscape network of Merged GCN and GRN. 

Transfer the file to your local computer and load the network into Cytoscape.  See if you can display GRN edges as directed in Cytoscape. 

When you are done, upload your merged Cytoscape file in to the class Googe Drive Folder.


