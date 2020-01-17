
<!-- README.md is generated from README.Rmd. Please edit that file -->
proActiv
--------

<!-- badges: start -->
<!-- badges: end -->
proActiv is an R package that estimates promoter activity from RNA-Seq data. proActiv uses aligned reads and genome annotations as input, and provides absolute and relative promoter activity as output. The package can be used to identify active promoters and alternative promoters, the details of the method are described at <https://doi.org/10.1101/176487>.

Additional data on differential promoters in tissues and cancers can be downloaded here: <https://jglab.org/data-and-software/>

### Installation

proActiv can be installed from [GitHub](https://github.com/) with:

``` r
library("devtools")
devtools::install_github("GoekeLab/proActiv")
```

### Annotation and Example Data

Pre-calculated promoter annotation data for Gencode v19 (GRCh37) is available as part of the proActiv package. The PromoterAnnotation object has 4 slots:

-   reducedExonRanges : The reduced first exon ranges for each promoter with promoter metadata for Gencode v19
-   promoterIdMapping : The id mapping between transcript ids, names, TSS ids, promoter ids and gene ids for Gencode v19
-   annotatedIntronRanges : The intron ranges annotated with the promoter information for Gencode v19
-   promoterCoordinates : Promoter coordinates (TSS) with gene id and internal promoter state for Gencode v19

Example junction files as produced by TopHat2 and STAR are available as external data. The reference genome used for alignment is Gencode v19 (GRCh37). The TopHat2 and STAR example files (5 files each) can be found at 'extdata/tophat2' and 'extdata/star' folders respectively.

Example TopHat2 files:

-   extdata/tophat2/sample1.bed
-   extdata/tophat2/sample2.bed
-   extdata/tophat2/sample3.bed
-   extdata/tophat2/sample4.bed
-   extdata/tophat2/sample5.bed

Example STAR files:

-   extdata/tophat2/sample1.junctions
-   extdata/tophat2/sample2.junctions
-   extdata/tophat2/sample3.junctions
-   extdata/tophat2/sample4.junctions
-   extdata/tophat2/sample5.junctions

### Estimate Promoter Activity (TopHat2 alignment)

This is a basic example to estimate promoter activity from a set of RNA-Seq data which was aligned with TopHat2. proActiv will use the junction file from the TopHat2 alignment (see below for an example with STAR-aligned reads), and a set of annotation objects that describe the associations of promoters, transcripts, and genes, to calculate promoter activity.

``` r
library(proActiv)

# Preprocessed data is available as part of the package for the human genome (hg19):
# Available data: proActiv::promoterAnnotationData.gencode.v19

### TopHat2 Junction Files Example

# The paths and labels for samples

tophatJunctionFiles <- list.files(system.file('extdata/tophat2', package = 'proActiv'), full.names = TRUE)
tophatJunctionFileLabels <- paste0('s', 1:length(tophatJunctionFiles), '-tophat')

# Count the total number of junction reads for each promoter
promoterCounts.tophat <- calculatePromoterReadCounts(proActiv::promoterAnnotationData.gencode.v19,
                                                      junctionFilePaths = tophatJunctionFiles,
                                                      junctionFileLabels =  tophatJunctionFileLabels,
                                                      junctionType = 'tophat')

# Normalize promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.tophat <- normalizePromoterReadCounts(promoterCounts.tophat)

# Calculate absolute promoter activity
absolutePromoterActivity.tophat <- getAbsolutePromoterActivity(normalizedPromoterCounts.tophat,
                                                               proActiv::promoterAnnotationData.gencode.v19)
# Calculate gene expression
geneExpression.tophat <- getGeneExpression(absolutePromoterActivity.tophat)

# Calculate relative promoter activity
relativePromoterActivity.tophat <- getRelativePromoterActivity(absolutePromoterActivity.tophat,
                                                               geneExpression.tophat)
```

### Estimate Promoter Activity (STAR alignment)

``` r
library(proActiv)

# Preprocessed data is available as part of the package for the human genome (hg19):
# Available data: proActiv::promoterAnnotationData.gencode.v19

### STAR Junction Files Example

# The paths and labels for samples
starJunctionFiles <- list.files(system.file('extdata/star', package = 'proActiv'), full.names = TRUE)
starJunctionFileLabels <- paste0('s', 1:length(starJunctionFiles), '-star')

# Count the total number of junction reads for each promoter
promoterCounts.star <- calculatePromoterReadCounts(proActiv::promoterAnnotationData.gencode.v19,
                                                      junctionFilePaths = starJunctionFiles,
                                                      junctionFileLabels =  starJunctionFileLabels,
                                                      junctionType = 'star')

# Normalize promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.star <- normalizePromoterReadCounts(promoterCounts.star)

# Calculate absolute promoter activity
absolutePromoterActivity.star <- getAbsolutePromoterActivity(normalizedPromoterCounts.star,
                                                             proActiv::promoterAnnotationData.gencode.v19)

# Calculate gene expression
geneExpression.star <- getGeneExpression(absolutePromoterActivity.star)

# Calculate relative promoter activity
relativePromoterActivity.star <- getRelativePromoterActivity(absolutePromoterActivity.star,
                                                             geneExpression.star)
```

### Creating your own promoter annotations

proActiv provides functions to create promoter annotation objects for any genome. Here we describe how the annotation can be created using a TxDb object (please see the TxDb documentation for how to create annotations from a GTF file).

A TxDb object for the human genome version hg19 (Grch37) can be downloaded here: [inputFiles](http://s3.ap-southeast-1.amazonaws.com/all-public-data.store.genome.sg/DemirciogluEtAl2019/annotations/gencode.v19.annotation.sqlite)

``` r
library(GenomicRanges)
library(GenomicFeatures)
library(GenomicAlignments)
library(dplyr)
library(proActiv)

# Load the txdb object for your annotation of choice (Gencode v19 used here)
txdb <- loadDb('./inputFiles/annotation/gencode.v19.annotation.sqlite')

# The species argument to be used for GenomeInfoDb::keepStandardChromosomes
species <- 'Homo_sapiens'
# The number of cores to be used for parallel execution (mc.cores argument for parallel::mclappy), optional
numberOfCores <- 1

### Annotation data preparation
### Needs to be executed once per annotation. Results can be saved and loaded later for reuse

promoterAnnotationData <- preparePromoterAnnotationData(txdb, species = 'Homo_sapiens', numberOfCores = 1)

# Retrieve the id mapping between transcripts, TSSs, promoters and genes
head(promoterIdMapping(promoterAnnotationData))

# Retrieve promoter coordinates
head(promoterCoordinates(promoterAnnotationData))
```

Limitations
-----------

proActiv will not provide promoter activity estimates for promoters which are not uniquely identifiable from splice junctions (single exon transcripts, promoters which overlap with internal exons).

Citing proActiv
---------------

If you use proActiv, please cite: Demircioğlu, Deniz, et al. "A Pan-Cancer Transcriptome Analysis Reveals Pervasive Regulation through Tumor-Associated Alternative Promoters." bioRxiv (2018): 176487.

Contributors
------------

ProActiv is developed and maintained by Deniz Demircioglu and Jonathan Göke.
