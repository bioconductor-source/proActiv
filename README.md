
<!-- README.md is generated from README.Rmd. Please edit that file -->

# proActiv

<!-- badges: start -->

<!-- badges: end -->

The goal of proActiv is to estimate promoter activity using RNA-Seq
data. The users only need to provide an input txdb object (annotation
version of their choice) and the input junction bed files. The proActiv
package will identify the promoter regions and estimate absolute and
relative promoter activity as well as gene expression. The details of
the method can be found at <https://doi.org/10.1101/176487>.

## Installation

proActiv is currently available only as the development version and can
be installed from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("GoekeLab/proActiv")
```

## Examples

This is a basic example which shows you how to solve a common problem:
The input data used for the example below can be downloaded from here:
[inputFiles](https://drive.google.com/drive/folders/1R8sI97h1ZTdyxbQxG4latR9xN9FF2tq8?usp=sharing)
The preprocessed annotation data for Gencode v19 to be used for simple
examples can be downloaded from here:
[preprocessedAnnotation](https://drive.google.com/drive/folders/1dtuP2QIKBTIQd8HecUmDZz2fCI_VaLMl?usp=sharing)

### Simple example (TopHat2)

``` r
library(GenomicRanges)
library(GenomicFeatures)
library(GenomicAlignments)
library(dplyr)
library(proActiv)

# Loads the exonReducedRanges, promoterIdMapping, intronRanges.annotated and promoterCoordinates objects
load('preprocessedAnnotation/preprocessedAnnotation.RData')

### TopHat2 Junction Files Example 

# The paths and labels for samples
tophatJunctionFiles <- list.files('./inputFiles/tophat/', full.names = TRUE)
tophatJunctionFileLabels <- paste0('s', 1:length(tophatJunctionFiles), '-tophat')

# Count the total number of junction reads for each promoter
promoterCounts.tophat <- calculatePromoterReadCounts(exonReducedRanges, 
                                                      intronRanges.annotated, 
                                                      junctionFilePaths = tophatJunctionFiles, 
                                                      junctionFileLabels =  tophatJunctionFileLabels, 
                                                      junctionType = 'tophat', 
                                                      numberOfCores)

# Normalize the promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.tophat <- normalizePromoterReadCounts(promoterCounts.tophat)

# Calculate the absolute promoter activity
absolutePromoterActivity.tophat <- getAbsolutePromoterActivity(normalizedPromoterCounts.tophat, 
                                                               promoterIdMapping, 
                                                               log2 = TRUE, 
                                                               pseudocount = 1)
# Calculate the gene expression
geneExpression.tophat <- getGeneExpression(absolutePromoterActivity.tophat)
# Calculate the relative promoter activity
relativePromoterActivity.tophat <- getRelativePromoterActivity(absolutePromoterActivity.tophat, 
                                                               geneExpression.tophat)
```

### Simple example (STAR)

``` r
library(GenomicRanges)
library(GenomicFeatures)
library(GenomicAlignments)
library(dplyr)
library(proActiv)

# Loads the exonReducedRanges, promoterIdMapping, intronRanges.annotated and promoterCoordinates objects
load('preprocessedAnnotation/preprocessedAnnotation.RData')

### STAR Junction Files Example 

# The paths and labels for samples
starJunctionFiles <- list.files('./inputFiles/star/', full.names = TRUE)
starJunctionFileLabels <- paste0('s', 1:length(starJunctionFiles), '-star')

# Count the total number of junction reads for each promoter
promoterCounts.star <- calculatePromoterReadCounts(exonReducedRanges, 
                                                      intronRanges.annotated, 
                                                      junctionFilePaths = starJunctionFiles, 
                                                      junctionFileLabels =  starJunctionFileLabels, 
                                                      junctionType = 'star', 
                                                      numberOfCores)

# Normalize the promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.star <- normalizePromoterReadCounts(promoterCounts.star)

# Calculate the absolute promoter activity
absolutePromoterActivity.star <- getAbsolutePromoterActivity(normalizedPromoterCounts.star, 
                                                             promoterIdMapping, 
                                                             log2 = TRUE, 
                                                             pseudocount = 1)
# Calculate the gene expression
geneExpression.star <- getGeneExpression(absolutePromoterActivity.star)
# Calculate the relative promoter activity
relativePromoterActivity.star <- getRelativePromoterActivity(absolutePromoterActivity.star, 
                                                             geneExpression.tophat)
```

### Advanced example

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

# Reduce first exons to identify transcripts belonging to each promoter
exonReducedRanges <- getUnannotatedReducedExonRanges(txdb, 
                                                     species,
                                                     numberOfCores)

# Prepare the id mapping transcripts, TSSs, promoters and genes
promoterIdMapping <- preparePromoterIdMapping(txdb, 
                                              species,
                                              exonReducedRanges)

# Prepare the annotated intron ranges to be used as input for junction read counting
intronRanges.annotated <- prepareAnnotatedIntronRanges(txdb, 
                                                        species, 
                                                        promoterIdMapping)

# Annotate the reduced exons with promoter metadata
exonReducedRanges <- prepareAnnotatedReducedExonRanges(txdb, 
                                                       species, 
                                                       promoterIdMapping, 
                                                       exonReducedRanges)

# Retrieve promoter coordinates 
promoterCoordinates <- preparePromoterCoordinates(exonReducedRanges,
                                                    promoterIdMapping)

### TopHat2 Junction Files Example 

# The paths and labels for samples
tophatJunctionFiles <- list.files('./inputFiles/tophat/', full.names = TRUE)
tophatJunctionFileLabels <- paste0('s', 1:length(tophatJunctionFiles), '-tophat')

# Count the total number of junction reads for each promoter
promoterCounts.tophat <- calculatePromoterReadCounts(exonReducedRanges, 
                                                      intronRanges.annotated, 
                                                      junctionFilePaths = tophatJunctionFiles, 
                                                      junctionFileLabels =  tophatJunctionFileLabels, 
                                                      junctionType = 'tophat', 
                                                      numberOfCores)

# Normalize the promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.tophat <- normalizePromoterReadCounts(promoterCounts.tophat)

# Calculate the absolute promoter activity
absolutePromoterActivity.tophat <- getAbsolutePromoterActivity(normalizedPromoterCounts.tophat, 
                                                               promoterIdMapping, 
                                                               log2 = TRUE, 
                                                               pseudocount = 1)
# Calculate the gene expression
geneExpression.tophat <- getGeneExpression(absolutePromoterActivity.tophat)
# Calculate the relative promoter activity
relativePromoterActivity.tophat <- getRelativePromoterActivity(absolutePromoterActivity.tophat, 
                                                               geneExpression.tophat)


### STAR Junction Files Example 

# The paths and labels for samples
starJunctionFiles <- list.files('./inputFiles/star/', full.names = TRUE)
starJunctionFileLabels <- paste0('s', 1:length(starJunctionFiles), '-star')

# Count the total number of junction reads for each promoter
promoterCounts.star <- calculatePromoterReadCounts(exonReducedRanges, 
                                                      intronRanges.annotated, 
                                                      junctionFilePaths = starJunctionFiles, 
                                                      junctionFileLabels =  starJunctionFileLabels, 
                                                      junctionType = 'star', 
                                                      numberOfCores)

# Normalize the promoter read counts by DESeq2 (optional)
normalizedPromoterCounts.star <- normalizePromoterReadCounts(promoterCounts.star)

# Calculate the absolute promoter activity
absolutePromoterActivity.star <- getAbsolutePromoterActivity(normalizedPromoterCounts.star, 
                                                             promoterIdMapping, 
                                                             log2 = TRUE, 
                                                             pseudocount = 1)
# Calculate the gene expression
geneExpression.star <- getGeneExpression(absolutePromoterActivity.star)
# Calculate the relative promoter activity
relativePromoterActivity.star <- getRelativePromoterActivity(absolutePromoterActivity.star, 
                                                             geneExpression.tophat)
```