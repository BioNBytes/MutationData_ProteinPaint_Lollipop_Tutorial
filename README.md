# 2025-05-29 - 🧬 ProteinPaint Lollipop Plot Tutorial
Author: Osama Shiraz Shah
Date: 2025-05-29

---

## Prerequisites: 
- **Basic familiarity with cBioPortal**: Understand how to navigate studies and identify different data types (e.g., mutation, expression, clinical).
- **A working R environment**: R and RStudio (optional) installed and functional on your system.
- **Internet access** 😄: Required to install necessary packages, download datasets from cBioPortal and access ProteinPaint website.

---

## Step 1: Install and Load `cbioportalR`
The environment setup is quite straightforward and we only need two packages installed in R language. One is dyplr and cbioportalR. The code chunk below guides you in this part.

```r
## Install libraries
# install.packages("dplyr")
# install.packages("cbioportalR")
# Load libraries
library(dplyr)
library(cbioportalR)
```


## Step 2: Connecting to cBioPortal
Now we can ensure that we have access to cBioPortal public database and can explore available studies.
```r
# Set connection to public cBioPortal
set_cbioportal_db("public")

# Test connection
test_cbioportal_db()

# View available studies
available_studies <- available_studies()
head(available_studies)
```


## Step 3: Retrieve Mutation Data
For the sake of this tutorial will utilize the TCGA breast cancer study by [Ciriello et al](https://pubmed.ncbi.nlm.nih.gov/26451490/) from 2015. Here is the [link](https://www.cbioportal.org/study/summary?id=brca_tcga_pub2015) to the cBioPortal page of this study. We will use the unique study_id for this study to programmatically retrieve associated mutation data (in MAF format). Then we will subset it for the *CDH1* gene and view the identified mutation types.
```r
# Retrieve mutation data for CDH1 from TCGA Breast Cancer study
TCGA_BRCA_MAF <- get_mutations_by_study(study_id = "brca_tcga_pub2015")
TCGA_BRCA_CDH1_MAF <- subset(TCGA_BRCA_MAF, hugoGeneSymbol == "CDH1") %>% as.data.frame()

# View mutation types
table(as.character(TCGA_BRCA_CDH1_MAF$mutationType))
```


## Step 4: Make Mutation Names Compatible with ProteinPaint
Now we have to convert the MAF mutation type names to ProteinPaint recognized names. This will be done in two steps. First, convert mutation names from MAF to ProteinPaint supported format. Second, convert mutation names to symbols. For second step, we will define a list containing mapping between mutation names and symbols recognized by ProteinPaint.
```r
# Convert MAF mutation types names to ProteinPaint recognized names
TCGA_BRCA_CDH1_MAF$MUT_protein_paint <- ifelse(
  as.character(TCGA_BRCA_CDH1_MAF$mutationType) %in% c("Frame_Shift_Del", "Frame_Shift_Ins", "In_Frame_Del"), "FRAMESHIFT",
  ifelse(as.character(TCGA_BRCA_CDH1_MAF$mutationType) == "Missense_Mutation", "MISSENSE",
    ifelse(as.character(TCGA_BRCA_CDH1_MAF$mutationType) == "Nonsense_Mutation", "NONSENSE",
      ifelse(as.character(TCGA_BRCA_CDH1_MAF$mutationType) == "Splice_Site", "SPLICE", "NA")
    )
  )
)

# Define mutation type name-symbol dictionary as described in https://proteinpaint.stjude.org/
mutation_symbol_dict <- list(
  MISSENSE = "M",
  EXON = "E",
  FRAMESHIFT = "F",
  NONSENSE = "N",
  SILENT = "S",
  PROTEINDEL = "D",
  PROTEININS = "I",
  PROTEINALTERING = "ProteinAltering",
  SPLICE_REGION = "P",
  SPLICE = "L",
  INTRON = "Intron",
  `Not tested` = "Blank",
  Wildtype = "WT",
  UTR_3 = "Utr3",
  UTR_5 = "Utr5",
  NONSTANDARD = "X",
  NONCODING = "noncoding",
  SNV = "snv",
  MNV = "mnv",
  `Sequence insertion` = "insertion",
  `Sequence deletion` = "deletion"
)

# Map ProteinPaint mutations to symbols  
TCGA_BRCA_CDH1_MAF$MUT_protein_paint_class <- unlist(mutation_symbol_dict[TCGA_BRCA_CDH1_MAF$MUT_protein_paint])
```


## Step 5: Save Data in Format Compatible with ProteinPaint
ProteinPaint takes mutation input in the following format: Mutation format: mutation type name, genomic position, mutation type symbol, sample (optional). Our output file will have one mutation per line and each field being tab delimited. 
```r
# Format for output
TCGA_CDH1_protein_paint <- paste0(
  TCGA_BRCA_CDH1_MAF$proteinChange, ", ",
  "chr", TCGA_BRCA_CDH1_MAF$chr, ":", TCGA_BRCA_CDH1_MAF$startPosition, ", ",
  TCGA_BRCA_CDH1_MAF$MUT_protein_paint_class, ", ", TCGA_BRCA_CDH1_MAF$sampleId
)

# Export to tab-delimited file
write.table(
  x = TCGA_CDH1_protein_paint,
  file = "TCGA_BRCA_CDH1_protein_paint.txt",
  col.names = FALSE,
  row.names = FALSE,
  quote = FALSE,
  sep = "\t"
)
```
You can also download the saved output file from this link: ![[TCGA_BRCA_CDH1_protein_paint.txt]]

