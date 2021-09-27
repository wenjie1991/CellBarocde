# CellBarcode

**CellBarcode** is an R package for dealing with **Cellular DNA barcoding** sequencing data.

The R package were created by Wenjie SUN, Anne-Marie Lyne, and Leïla Perié at Institut Curie.

## Kinds of barcodes

**CellBarcode** can handle all kinds of DNA barcodes, as long as:

- The barcodes have a pattern which can be matched by a regular expression.
- Each barcode is within a single sequencing read.

## What you can do with **CellBarcode**

- Performs quality control for the DNA sequence results, and filters the sequences according
  to their quality metrics.

- Identifies barcode (and UMI) information in sequencing results.

- Performs quality control and deals with the spurious sequence which comes from potential PCR & sequence errors.

- Provides toolkits to make it easier to manipulate samples and barcodes with metadata.

## Installing

### Install devel version from GitHub

```
if(!requireNamespace("remotes", quietly = TRUE))
    install.packages("remotes")
remotes::install_github("wenjie1991/CellBarcode")
```

## Get start

Here is an example of a basic workflow:

```{r}
library(CellBarcode)
library(magrittr)

# The example data is a mix of MEF lines with known barcodes
# 2000 reads for each file have been sampled for this test dataset
# TODO: Citation of the paper:
example_data <- system.file("extdata", "mef_test_data", package = "CellBarcode")
fq_files <- dir(example_data, "gz", full=TRUE)

# prepare metadata
metadata <- stringr::str_split_fixed(basename(fq_files), "_", 10)[, c(4, 6)]
metadata <- data.frame(metadata)
sample_name <- apply(metadata, 1, paste, collapse = "_")
colnames(metadata) = c("cell_number", "replication")
rownames(metadata) = sample_name
metadata

# extract UMI barcode with regular expression
bc_obj <- bc_extract(
  fq_files,
  pattern = "(.{12})CTCGAGGTCATCGAAGTATCAAG(.+)TAGCAAGCTCGAGAGTAGACCTACT", 
  pattern_type = c("UMI" = 1, "barcode" = 2),
  sample_name = sample_name,
  metadata = metadata
)
bc_obj

# sample subset operation, select 'mixa'
bc_sub <- bc_subset(bc_obj, sample=replication == "mixa")
bc_sub 

# filter the barcode, UMI barcode amplicon >= 2 & UMI counts >= 2
bc_sub <- bc_cure_umi(bc_sub, depth = 2) %>% bc_cure_depth(depth = 2)

# select barcodes with a white list
bc_sub[c("AAGTCCAGTACTATCGTACTA", "AAGTCCAGTACTGTAGCTACTA"), ]

# export the barcode counts to data.frame
head(bc_2df(bc_sub))

# export the barcode counts to matrix
head(bc_2matrix(bc_sub))
```


## License

[MIT](https://choosealicense.com/licenses/mit/)
