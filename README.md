# linking-gcn-edges-with-a-grn

# Linking GCN Edges with a GRN.
There are 78,724 known genes (19,433 protein coding) that encode 385,659 unique RNA transcripts (89,832 protein coding) (GENCODE v47; https://www.gencodegenes.org/human/stats_47.html).  Through RNAseq analysis, it is possible to measure the output of all these genes for a given sample using gene co-expression networks (GCNs). 

RNAseq sample expression profiles can be stitched together into a Gene Expression Matrix (GEM) compendium.  Then one can compare transcriptome groups in the GEMs to identify differentially expressed genes (DEGs) or genes that are co-expressed (GCN).  Any of these monogenic or bigenic expression differences imply that the genes are differentially regulated via condition-specific gene regulatory networks (GRNs). 

Fortunately, tissue specific GRNs are being published for human samples.  For example, many human GRNs are described in Sonawane et al. “Understanding Tissue-Specific Gene Regulation Article Understanding Tissue-Specific Gene Regulation. 1077-1088 (2017).” These GRNs can be mapped to differentially regulated genes in a tissue specific context. This map of normal gene relationships can be mapped to aberrant gene expression states in non-wild-type samples and broken regulatory machinery can be identified. 

In this lab, we will link the genes in a human GCN with a tissue specific human GRN to generate gene regulation mechanism hypotheses.  In essence, one can use these mapping to explain why a GCN edge exists in the first place.  One twist is that we will need to rename genes in the GRN.  To achieve this, we will learn how to use a simple command line database, another useful Linux skill. How fun!!!

# Objectives
```
Install sqlite3 database software.
Obtain and preprocess a GRN and GCN file.
Load the relevant tables into a sqlite3 database.
Perform SQL SELECT queries to build a preprocessed merged directional GRN and unidirectional GCN network.
Visualize the network in Cytoscape.
Install sqlite3 database software.
```


# Step A. Download that GRN files from Sonawane, A. R. et al. Understanding Tissue-Specific Gene Regulation Article Understanding Tissue-Specific Gene Regulation (2017). This file maps the GRN genes with target genes for various human tissues.  Obtain this file using this command:
```
wget https://www.cell.com/cms/10.1016/j.celrep.2017.10.001/attachment/e7309c03-e579-4119-a95e-376ab2066cbb/mmc2.csv
```

# Step B. Download that GCN files from Hickmann A et al Identification of condition-specific biomarker systems in uterine cancer (2021).

```
wget https://gsajournals.figshare.com/ndownloader/files/31186971
```

# Step C. The GRN contain gene target names as ENSEMBL IDs (e.g. ENSG00000278025= NCR1), so we will want to change them to their corresponding official gene symbols.  To do this, we will need a gene identifier mapping table form Ensembl following these steps:

Go to https://www.ensembl.org/

Go to Biomart

Select these attributes and save as a CSV file.  It will likely save as mart_export.csv

```
Gene stable ID
Gene stable ID version
Transcript stable ID
Transcript stable ID version
Gene name
```

# Step D. Prepare the files for database loading.

cat mmc2.csv | sed 's/\"//g' | sed 's/,/\t/4;s/,/\t/3;s/,/\t/1;s/,/\t/1' > grn.tab
#Change space delimiters to tabs in GCN file
cat merged-gtex-kirp-kich-kirp-gem.log2.quantile.coexpnet.txt | sed 's/\s/\t/g' > gcn.tab
Build a database of the GRN, GCN, and names tables.

# Step E: Install sqllite database software

You are probably familiar with spreadsheets like Excel.  Excel facilitates analysis of row x column data. However, when you want to find relationships between Excel spreadsheets, one need to write custom code or identify relationships in a relational database management system (rdbms).  We will use the command line friendly sqlite3 database software to identify relationships in our data.  You can find excellent help in these sqlite resources:  https://sqlite.org/cli.html and https://www.sqlitetutorial.net/.

Install sqllite3 in your Ubunti VM using these commands:

```
PROMPT: How do i install sqllite on a shared linux cluster?
```

# Step F. Start the sqlite3 shell and create the database called mapping.
```
sqlite3 mapping
```

# Step G. Using the sqlite shell, load 3 tables into the mapping database (grn, gcn, names)
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

# Step H. Make an INNER JOIN to create a tab-delimited GRN file with the target gene mapped to official gene symbols.  Note that Structured Query Language (SQL) tutorials are all over the Internet.  Here is one example: https://www.w3schools.com/sql/sql_intro.asp. 

In this step we will perform SELECT..FROM, INNER JOIN, and WHERE SQL statements to relate and map the gene IDs, keeping only edges with the work ‘Kidney’ in the Tissue fields, and write the renamed data to a file.

```
.separator "\t"
.output GRN-REMAPPED.txt
SELECT TF, names."Gene Name", Tissues, TargetGene
FROM grn
INNER JOIN names on names."Gene stable ID"=grn.TargetGene
WHERE Tissues LIKE '%Kidney%';
```

# Step I. Make a merged GRN and GCN edge list with bash.

Make a GRN and GCN edge files with the GRN and GCN edge labels.  Remember the GRN edges are directed (TF > target gene) edges while the GCN edges are undirected GeneA-GeneB).  Write the GRN and GCN edge lists and then merge them and remove duplicate edges.

#Obtain the edge lists and add the GCN or GRN label to each line

#Make GRN Edge File
```
cat GRN-REMAPPED.txt | awk '{print $1,$2}' > temp
awk '{print "GRN " $0}' temp   > GRN_edges.tab
```

#Make GCN Edge File
```
cat merged-gtex-kirp-kich-kirp-gem.log2.quantile.coexpnet.txt| awk '{print $1,$2}' > temp
awk '{print "GCN " $0}' temp   > GCN_edges.tab
```

#Concatenate the files
```
cat GRN_edges.tab GCN_edges.tab > merged.kidney.gcn.grn.tab

#remove duplicate lines (edges)
cat merged.kidney.gcn.grn.tab | uniq | sed 's/\s/\t/g' > merged.kidney.gcn.grn.unique.tab
```

# Step J. Make a merged GRN and GCN edge list with bash.
Make a Cytoscape network of Merged GCN and GRN. 

Transfer the file to your local computer and load the network into Cytoscape.  See if you can display GRN edges as directed in Cytoscape. 

When you are done, upload your merged Cytoscape file in to the class Googe Drive Folder.



