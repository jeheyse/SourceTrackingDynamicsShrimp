---
title: <font size = "6">Analysis</font>
author: Jasmine Heyse <sup>a</sup>, Ruben Props <sup>a</sup>, Pantipa Kongnuan <sup>b</sup>, Peter De Schryver <sup>b</sup>, Geert Rombaut<sup>b</sup>, Tom Defoirdt <sup>a</sup>, Nico Boon <sup>a</sup>
output:
  html_document:
    code_folding: show
    highlight: haddock
    keep_md: yes
    theme: flatly
    toc: yes
    number_sections: true
    toc_float:
      collapsed: no
      smooth_scroll: yes
      toc_depth: 4
editor_options:
  chunk_output_type: console
---

<style>
#TOC {
  margin: 15px 0px 15px 0px;
}
@media (max-width: 768px) {
#TOC {
  position: relative;
  width: 100%;
}
}
.toc-content {
  padding-left: 20px;
  padding-right: 20px;
}
div.tocify {
  width: 20%;
  max-width: 700px;
  max-height: 85%;
}
  body {text-align: justify}
  .main-container {max-width: 2100px !important;}
  code.r{ font-size: 11px; }
  pre{ font-size: 15px }
  pre code, pre, code {
  white-space: pre !important;
  overflow-x: scroll !important;
  word-break: keep-all !important;
  word-wrap: initial !important;
}
</style>


<font size="2">
&#160; &#160; &#160; &#160;<sup>a</sup>  Center for Microbial Ecology and Technology (CMET), Department of Biochemical and Microbial Technology, Ghent University, Coupure Links 653, B-9000 Gent, Belgium <br>
&#160; &#160;&#160; &#160; <sup>b</sup>  INVE Technologies NV, Hoogveld 93, B-9200 Dendermonde, Belgium
</font>
<br>
<br>
<br>

The full analysis for the paper _Rearing water microbiomes in white leg shrimp (<i>Litopenaeus vannamei</i>) larviculture assemble stochastically and are influenced by the microbiomes of live feed products_.



# Libraries and working directory




```r
# Packages
library("Phenoflow")
library("plyr")
library("dplyr")
library("ggplot2")
library("gridExtra")
library("grid")
library("RColorBrewer")
library("grid")
library("tidyr")
library("reshape2")
library("gganimate")
library("gifski")
library("ellipse")
library("glmnet")
library("mboost")
library("openxlsx")
library("stabs")
library("stringr")
library("cowplot")
library("viridis")
library("SpiecEasi")
library("scales")   
library("ggplot2")  
library("plyr")     
library("dplyr")
library("gtable")
library("tidyr")
library("data.table")
library("reshape2")
library("vegan")
library("phyloseq") 
library("ape")      
library("openxlsx") 
library("readxl")   
library("cowplot")
library("boot")
library("Biostrings")
library("htmltools")
library("CMETNGS")
library("VennDiagram")
library("adegenet")
library("ggtree")
library("stringr")
library("knitr")
library("stringdist")
library("DECIPHER")
library("dada2")
library("seqinr")
library("RColorBrewer")
library("ggalluvial")
library("ggfittext")
library("otu2ot")
library("network")
library("GGally")
library("igraph")
library("ggrepel")
library("ggraph")
library("DESeq2")
library("betapart")
library("picante")
library("microbiome")
library("VennDiagram")
# Load the FEAST function from the sourcefile
source("./Functions/FEAST_src/src.R")
# Load the Raup Crick function to estimate community assembly (Stegen et al. 2013)
source("./Functions/Raup_Crick_Abundance.r")
# Load convenience funtions
source("./Functions/ConvenienceFunctions.R")

set.seed(458)
```


```r
# Define colors for plotting
TankColors <- c("#82487d", "#edd958", "#3c4175", "#60be8f", "#6c72dd")
AllTankColors <- c("#82487d", "#edd958", "#3c4175", "#60be8f", "#6c72dd", "#5a7e48", "#d1568f")
AllTankAndSourcesColors <- c("#82487d", "#edd958", "#3c4175", "#60be8f", "#6c72dd", "#5a7e48", "#a74472", "#6296a3", "#d47e37")
SourceColors <- c("#5a7e48", "#a74472", "#6296a3", "#d47e37")
SingleColor <- c("#3c4175")
SingleColor2 <- c("#5a7e48")
ColorBlocksFacet <- c("#e0e0e0")
Series8 <- c("#990241", "#c74a65", "#c95b38", "#e3b549", "#8fbd75",  "#498f75", "#515b9c", "#470161")
Series16 <- c("#f5bdc5","#e65c6e", "#ba1822", "#750019", "#8f501b", "#bd7a28", "#f5af00", "#f2ff91", "#a1d179", "#53a358",  "#3a6e5b", "#3eb8a7", "#7581d1", "#091d9e", "#5c0778", "#a600a3")
ColorsCommunityAssembly <- c("#dc9d00", "#f2ff91", "#9199b9", "#151f45", "#394881")
ColorsCommunityAssemblyII <- c("#dc9d00", "#f2ff91", "#394881")
```

# Larval health 


```r
# Load continuous larval health parameters
ContinuousData <- xlsx::read.xlsx("./Metadata/MetadataLarvalHealth.xlsx", stringsAsFactors = F, sheetIndex = 1)
ContinuousData$Tank[ContinuousData$Tank == "T150"] <- "T1"
ContinuousData$Tank[ContinuousData$Tank == "T135"] <- "T2"
ContinuousData$Tank[ContinuousData$Tank == "T113"] <- "T3"
ContinuousData$Tank[ContinuousData$Tank == "T109"] <- "T4"
ContinuousData$Tank[ContinuousData$Tank == "T120"] <- "T5"
ContinuousData$Tank <- as.factor(ContinuousData$Tank)
# Importing caused conversion to characters
ContinuousData$Time_cont <- as.numeric(paste(ContinuousData$Time_cont))
ContinuousData$pct_activity <- as.numeric(paste(ContinuousData$pct_activity))
ContinuousData$pct_mortality <- as.numeric(paste(ContinuousData$pct_mortality))
# NA's are no mortality or activity reduction observed 
ContinuousData$pct_activity[is.na(ContinuousData$pct_activity)] <- 1
ContinuousData$pct_mortality[is.na(ContinuousData$pct_mortality)] <- 0


# Plot larval health for the different tanks
p_LarvalActivity <- ContinuousData %>%
                    dplyr::filter(Tank %in% c("T1","T2","T3","T4","T5")) %>% 
                    ggplot(data = ., aes(x = Time_cont, y = 100*pct_activity)) +
                    geom_line(aes(x = Time_cont, y = 100*pct_activity)) +                     
                    geom_point(shape = 21, size = 2, alpha = 1, aes(fill = Tank)) +
                    facet_grid(. ~ Tank) +
                    geom_vline(xintercept = 0, size = 1) +
                    scale_fill_manual(values = TankColors) +
                    theme_cowplot() +
                    theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), strip.background = element_rect(color = ColorBlocksFacet, fill = ColorBlocksFacet, size = 1), axis.line.y = element_blank()) +
                    labs(color = "", y = "Activity (%)", x = "Time (d)") +
                    guides(color = FALSE, fill = FALSE) +
                    scale_x_continuous(limits = c(0, 18), expand = c(0, 0))
print(p_LarvalActivity)


p_LarvalMortality <- ContinuousData %>%
                    dplyr::filter(Tank %in% c("T1","T2","T3","T4","T5")) %>% 
                    ggplot(data = ., aes(x = Time_cont, y = 100*pct_mortality)) +
                    geom_line(aes(x = Time_cont, y = 100*pct_mortality)) +                     
                    geom_point(shape = 21, size = 2, alpha = 1, aes(fill = Tank)) +
                    facet_grid(. ~ Tank) +
                    geom_vline(xintercept = 0, size = 1) +
                    scale_fill_manual(values = TankColors) +
                    theme_cowplot() +
                    theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), strip.background = element_rect(color = ColorBlocksFacet, fill = ColorBlocksFacet, size = 1), axis.line.y = element_blank()) +
                    labs(color = "", y = "Mortality (%)", x = "Time (d)") +
                    guides(color = FALSE, fill = FALSE) +
                    scale_x_continuous(limits = c(0, 18), expand = c(0, 0))
print(p_LarvalMortality)


# Assemble plot with both mortality and activity
g1 <- plot_grid(p_LarvalActivity, p_LarvalMortality, labels = c("A", "B"), ncol = 1, nrow = 2, scale = 0.95)
ggsave(file = "Figures/HEALTH-MortalityActivity.png", width = 9, height = 5.5, dpi = 300, units = "in", g1)


# The data of the larval activity and mortality are not used further
remove(p_LarvalActivity, p_LarvalMortality, g1, ContinuousData)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/DataContinuousLarvalHealth-1.png" width="50%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/DataContinuousLarvalHealth-2.png" width="50%" style="display: block; margin: auto;" />

# Processing flow cytometry data 

## On-site flow cytometry measurements

### Load data


```r
# Upload fcs-files
Datapath <- c("Data/FCM/OnSite/")
fcsfiles <- list.files(path = Datapath, recursive = TRUE, pattern = ".fcs", full.names = TRUE)
flowData <- flowCore::read.flowSet(files = fcsfiles, transformation = FALSE, pattern = ".fcs", ignore.text.offset = TRUE, emptyValue = FALSE)
# Remove variables that are no longer needed
remove(Datapath, fcsfiles)
```


```r
# Make metadata from sample names
metadata <- data.frame(flowCore::sampleNames(flowData), do.call(rbind, strsplit(substring(flowCore::sampleNames(flowData), 5), split = "_")))
# Clean-up the metadata
metadata <- cbind(metadata, do.call(rbind, strsplit(as.character(metadata$X1), split = ".", fixed = TRUE)))
colnames(metadata) <- c("Sample_names", "Sample", "Type", "Stain", "Dilution", "Tank", "Day", "Timepoint", "Feeding_status")
metadata$Dilution <- as.character(gsub(".fcs", "", metadata$Dilution))
metadata$Dilution <- as.character(gsub("x", "", metadata$Dilution))
metadata$Dilution <- as.numeric(metadata$Dilution)
# Make a continuous variable for time
metadata$Time_cont <- as.numeric(gsub("D", "", metadata$Day)) + (((as.numeric(as.character(metadata$Timepoint)))*3) - 1)/24
```

### Preprocessing and calculating cell densities


```r
# Select phenotypic features of interest and transform parameters
flowData_transformed <- flowCore::transform(flowData,
                                  `FL1-H` = asinh(`FL1-H`), 
                                  `SSC-H` = asinh(`SSC-H`), 
                                  `FL3-H` = asinh(`FL3-H`), 
                                  `FSC-H` = asinh(`FSC-H`))
remove(flowData)
```


```r
# Bacterial population
sqrcut1 <- matrix(c(8.5,8.5,16,16,
                    2,7,15,2),
                  ncol = 2, 
                  nrow = 4)
colnames(sqrcut1) <- c("FL1-H", "FL3-H")
polyGateTotalCells <- polygonGate(.gate = sqrcut1, filterId = "Total Cells")

# Algal population
sqrcut3 <- matrix(c(7,7,15,15,
                    13,16.5,16.5,13),
                  ncol = 2, 
                  nrow = 4)
colnames(sqrcut3) <- c("FL1-H", "FL3-H")
polyGateAlgae <- polygonGate(.gate = sqrcut3, filterId = "Algal Cells")

# Remove variables that are no longer needed
remove(sqrcut1, sqrcut3)
```


```r
# Count bacteria and algae
BacterialCount <- flowCore::filter(flowData_transformed, polyGateTotalCells) 
BacterialCount <- toTable(summary(BacterialCount))
metadata <- dplyr::left_join(metadata, BacterialCount[c("sample", "true")], by = c("Sample_names" = "sample"))
colnames(metadata)[11] <- "Count"
AlgalCount <- flowCore::filter(flowData_transformed, polyGateAlgae)
AlgalCount <- toTable(summary(AlgalCount))
metadata <- dplyr::left_join(metadata, AlgalCount[c("sample", "true")], by = c("Sample_names" = "sample"))
colnames(metadata)[12] <- "AlgalCount"
# Get the volume (is expressed as µL)
vol <- as.numeric(flowCore::fsApply(flowData_transformed, FUN = function(x) x@description$`$VOL`))/1000
# Calculate the bacterial and algal densities (multiply with 1000 to get cells/mL)
metadata$BacterialDensity <- 1000*metadata$Count*metadata$Dilution/vol
metadata$AlgalDensity <- 1000*metadata$AlgalCount*metadata$Dilution/vol
# Remove all variables that are no longer needed
remove(AlgalCount, BacterialCount, polyGateAlgae, polyGateTotalCells, vol, flowData_transformed)
```


```r
# Wite the counting-results to a csv to reupload them later
write.csv(x = metadata, file = "Results/DYNAMICS-OnSite-Counting.csv")

# remove variables that are no longer needed
remove(metadata)
```

## Off-site flow cytometry measurements

### Load data


```r
# Upload fcs-files
Datapath <- c("Data/FCM/OffSite/")
fcsfiles <- list.files(path = Datapath, recursive = TRUE, pattern = ".fcs", full.names = TRUE)
flowData <- flowCore::read.flowSet(files = fcsfiles, transformation = FALSE, pattern = ".fcs", ignore.text.offset = TRUE, emptyValue = FALSE)
# Remove variables that are no longer needed
remove(Datapath, fcsfiles)
```


```r
# Make metadata from sample names
metadata <- data.frame(flowCore::sampleNames(flowData), do.call(rbind, strsplit(substring(flowCore::sampleNames(flowData), 5), split = "_")))
# Clean-up the metadata
metadata <- cbind(metadata, do.call(rbind, strsplit(as.character(metadata$X1), split = ".", fixed = TRUE)))
colnames(metadata) <- c("Sample_names", "Sample", "Type", "Stain", "Dilution", "Tank", "Day", "Timepoint", "Feeding_status")
metadata$Dilution <- as.character(gsub(".fcs", "", metadata$Dilution))
metadata$Dilution <- as.character(gsub("x", "", metadata$Dilution))
metadata$Dilution <- as.numeric(metadata$Dilution)
# Make a continuous variable for time
metadata$Time_cont <- as.numeric(gsub("D", "", metadata$Day)) + (((as.numeric(as.character(metadata$Timepoint)))*3) - 1)/24
```

### Preprocessing and calculating cell densities


```r
# Select phenotypic features of interest and transform parameters
flowData_transformed <- flowCore::transform(flowData, 
                                  `FL1-H` = asinh(`FL1-H`), 
                                  `SSC-H` = asinh(`SSC-H`), 
                                  `FL3-H` = asinh(`FL3-H`),
                                  `FSC-H` = asinh(`FSC-H`))
remove(flowData)
```


```r
# Bacterial population
sqrcut1 <- matrix(c(8,8,16,16,
                    2,7,15.3,2),
                  ncol = 2, 
                  nrow = 4)
colnames(sqrcut1) <- c("FL1-H", "FL3-H")
polyGate1 <- polygonGate(.gate = sqrcut1, filterId = "Total Cells")

# Algal population
sqrcut3 <- matrix(c(7,7,15,15,
                    13,16.5,16.5,13),
                  ncol = 2, 
                  nrow = 4)
colnames(sqrcut3) <- c("FL1-H", "FL3-H")
polyGateAlgae <- polygonGate(.gate = sqrcut3, filterId = "Algal Cells")

# Combine the filters
filters <- filters(list(polyGate1, polyGateAlgae))
flist <- list(filters)

# Gating quality check
for (i in base::sample(which(metadata$Stain == "SG"), size = 2)){
  names(flist) <- flowCore::sampleNames(flowData_transformed[i])
  filter <- flist
  print(xyplot(`FL3-H` ~ `FL1-H`, data = flowData_transformed[i], 
              filter = filter,
              scales = list(y = list(limits = c(0,16)),
                            x = list(limits = c(6,17))),
              axis = axis.default, 
              nbin = 125, 
              par.strip.text = list(col = "black", font = 3, cex = 0.75), 
              smooth = FALSE)) 
}

# Remove all variables that are no longer needed
remove(sqrcut1, sqrcut3, i)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_GateCheck-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_GateCheck-2.png" width="70%" style="display: block; margin: auto;" />


```r
# Make a figure with both bacterial and algal gate for in the supplemenary information file
i <- 249 # random sample
names(flist) <- flowCore::sampleNames(flowData_transformed[i])
filter <- flist
p_gate <- xyplot(`FL3-H` ~ `FL1-H`, data = flowData_transformed[i], 
           filter = filter,
           scales = list(y = list(limits = c(1.5,17.25)),
                         x = list(limits = c(6.5,16.5))),
           axis = axis.default, 
           nbin = 125, 
           par.strip.text=list(col="white", font=2, cex=0), 
           smooth = FALSE)
print(p_gate)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_GateFigure-1.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/QC-Gate.png", width = 6, height = 4, res = 300, units = "in")
print(p_gate)
dev.off()
```

```
## png 
##   2
```

```r
# Remove all variables that are no longer needed
remove(filter, filters, flist, i)
```


```r
# Count bacteria and algae
BacterialCount <- flowCore::filter(flowData_transformed, polyGate1) 
BacterialCount <- toTable(summary(BacterialCount))
metadata <- dplyr::left_join(metadata, BacterialCount[c("sample", "true")], by = c("Sample_names" = "sample"))
colnames(metadata)[11] <- "Count"
AlgalCount <- flowCore::filter(flowData_transformed, polyGateAlgae)
AlgalCount <- toTable(summary(AlgalCount))
metadata <- dplyr::left_join(metadata, AlgalCount[c("sample", "true")], by = c("Sample_names" = "sample"))
colnames(metadata)[12] <- "AlgalCount"
# Get the volume (is expressed as µL)
vol <- as.numeric(flowCore::fsApply(flowData_transformed, FUN = function(x) x@description$`$VOL`))/1000
# Calculate the bacterial and algal densities (multiply with 1000 ti get cells/mL)
metadata$BacterialDensity <- 1000*metadata$Count*metadata$Dilution/vol
metadata$AlgalDensity <- 1000*metadata$AlgalCount*metadata$Dilution/vol
# Remove all variables that are no longer needed
remove(AlgalCount, BacterialCount, polyGateAlgae, polyGate1, vol, flowData_transformed)
```


```r
# Get the datasets for 'A' (after) and 'B' (before) separately
metadataB <- metadata[metadata$Feeding_status == "B",]
metadataA <- metadata[metadata$Feeding_status == "A",]
# Join the datasets
Combined <- left_join(metadataB[c("Tank", "Day", "Timepoint", "BacterialDensity")], metadataA[c("Tank", "Day", "Timepoint", "BacterialDensity")], by = c("Tank", "Day", "Timepoint"))
Combined <- Combined[complete.cases(Combined), ] # Remove NA's
# Quantify variance between 'A' and 'B'
Combined$Diff <- 100*(Combined$BacterialDensity.x - Combined$BacterialDensity.y)/rowMeans(Combined[c("BacterialDensity.x", "BacterialDensity.y")])
# 
mean(Combined$Diff)
```

```
## [1] 0.4427631
```

```r
median(Combined$Diff)
```

```
## [1] -0.3753104
```

```r
# Remove all variables that are no longer needed
remove(metadataB, metadataA, Combined)
```


```r
# Reorganise plot levels so 'B' (= before) will be plotted before 'A' (= after)
metadata$Feeding_status <- factor(metadata$Feeding_status, levels = c("B", "A", "Algae", "Artemia", "SEDIMENT", "0"))

# Plot results for the samples that were measured on site, but fixed
p_cells_Fixed <- metadata %>%
                 dplyr::filter(Tank %in% c("T1", "T2", "T3", "T4", "T5") & Feeding_status %in% c("A", "B")) %>% 
                 ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                 geom_point(shape = 21, size = 2, alpha = 1, aes(fill = Tank)) +
                 facet_grid(. ~ Tank) +
                 scale_fill_manual(values = TankColors) +
                 geom_vline(xintercept = 0, size = 1) +
                 labs(color = "", y = "Bacterial density (cells/mL)", x = "Time (d)") +
                 guides(color = FALSE) +
                 geom_smooth(col = "black", span = 0.25) +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.line.y = element_blank()) +
                 scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 2e8)) +
                 scale_x_continuous(limits = c(0, 19), expand = c(0, 0))
png("Figures/DYNAMICS-Tanks-BacterialDensity-Fixed.png", width = 14, height = 5, res = 500, units = "in")
print(p_cells_Fixed)
dev.off()
```

```
## png 
##   2
```

```r
# Select only subset from before the crash of T1 and T4
InfoTanksNotCrashed <- metadata[metadata$Tank %in% c("T2", "T3", "T5"),]
InfoTanksT1 <- metadata[metadata$Tank %in% c("T1") & metadata$Time_cont <= 13.5,]
InfoTanksT4 <- metadata[metadata$Tank %in% c("T4") & metadata$Time_cont <= 10.5,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)
# Plot
p_cells_Fixed_v <- InfoTanks %>%
                 dplyr::filter(Tank %in% c("T1", "T2", "T3", "T4", "T5") & Feeding_status %in% c("A", "B")) %>% 
                 ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                 geom_point(shape = 21, size = 1, alpha = 1, fill = "dimgray") +
                 facet_grid(Tank ~ .) +
                 # scale_fill_manual(values = TankColors) +
                 geom_vline(xintercept = 0, size = 1) +
                 labs(color = "", y = "Bacterial density (cells/mL)", x = "Time (d)") +
                 guides(fill = FALSE) +
                 geom_smooth(col = "black", span = 0.25, size = 0.5) +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.line.y = element_blank()) +
                 scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 2e8)) +
                 scale_x_continuous(limits = c(0, 19), expand = c(0, 0))

# Remove variables that are no longer needed 
remove(InfoTanksNotCrashed, InfoTanks, InfoTanksT1, InfoTanksT4)
```


```r
# Plot algal cell densities of the fixed samples
InfoTanksNotCrashed <- metadata[metadata$Tank %in% c("T2", "T3", "T5"),]
InfoTanksT1 <- metadata[metadata$Tank %in% c("T1") & metadata$Time_cont <= 13.5,]
InfoTanksT4 <- metadata[metadata$Tank %in% c("T4") & metadata$Time_cont <= 10.5,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)
p_algae_Fixed <- InfoTanks %>%
                 dplyr::filter(Tank %in% c("T1", "T2", "T3", "T4", "T5") & Feeding_status %in% c("A", "B") & AlgalDensity > 1e3) %>% 
                 ggplot(data = ., aes(x = Time_cont, y = AlgalDensity)) +
                 geom_point(shape = 21, size = 3, alpha = 1, aes(fill = Tank)) +
                 facet_wrap(. ~ Tank, nrow = 2) +
                 scale_fill_manual(values = TankColors) +
                 geom_vline(xintercept = 0, size = 1) +
                 labs(color = "", y = "Algal density (cells/mL)", x = "Time (d)") +
                 guides(color = FALSE) +
                 geom_smooth(col = "black", span = 0.25) +              
                 scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(5e2, 1e5)) +
                 scale_x_continuous(limits = c(0, 19), minor_breaks = seq(1,18), expand = c(0, 0)) +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.line.y = element_blank())
print(p_algae_Fixed)
png("Figures/DYNAMICS-Tanks-AlgalDensity-Fixed.png", width = 8, height = 7, res = 300, units = "in")
grid.draw(shift_legend(p_algae_Fixed))
dev.off()
```

```
## png 
##   2
```

```r
# Remove variables that are no longer needed
remove(p_algae_Fixed)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_PlotAlgalDensitiesTanks-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Make a dataframe to draw lines between samples of the same batch
Subset <- metadata %>% dplyr::filter(Tank  == "Algae")
Subset$Time_cont <- round(Subset$Time_cont , 2)
df <- as.data.frame(c(Subset$Time_cont, 1:10))
colnames(df) <- "Time_cont"
df <- left_join(df, Subset[c("Time_cont", "BacterialDensity")], by = c("Time_cont"))
 
# Plot bacterial density in algae stocks over time, for fixed samples
ColorsBackgoundBlocks <- c("#5a7e48", "#ffffff")
p_AlgaeStock_Fixed <- metadata %>%
                      dplyr::filter(Tank  == "Algae") %>% 
                      ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                      annotate("rect", xmin = 1, xmax = 2, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                      annotate("rect", xmin = 2, xmax = 3, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                      annotate("rect", xmin = 3, xmax = 4, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                      annotate("rect", xmin = 4, xmax = 5, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                      annotate("rect", xmin = 5, xmax = 6, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                      annotate("rect", xmin = 6, xmax = 7, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                      annotate("rect", xmin = 7, xmax = 8, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                      annotate("rect", xmin = 8, xmax = 9, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                      annotate("rect", xmin = 9, xmax = 10, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                      geom_line(data = df, aes(x = Time_cont, y = BacterialDensity)) + 
                      geom_point(shape = 21, size = 2, alpha = 1, fill = SourceColors[1]) +
                      labs(color = "", y = "Bacterial density (cells/mL)", x = "Time (d)") +
                      theme_cowplot() +
                      theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet)) +
                      scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 3e7)) + #, limits = c(1e5, 3e7)) +
                      scale_x_continuous(limits = c(1, 10), expand = c(0, 0), breaks = seq(2,10,2))
print(p_AlgaeStock_Fixed)
png("Figures/DYNAMICS-AlgalTank-BacterialDensity-Fixed.png", width = 10, height = 5, res = 500, units = "in")
print(p_AlgaeStock_Fixed)
dev.off()
```

```
## png 
##   2
```

```r
# Remove all variables that are no longer needed
remove(df, Subset, ColorsBackgoundBlocks)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_PlotBacterialDensitiesAlgaeStock-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Reorganise time points in order to start at 11 a.m. and finish at 8 a.m. (for easier interpretation of the plot)
metadata$Timepoint <- factor(metadata$Timepoint, levels = c("4", "5", "6", "7", "8", "1", "2", "3", "0", "PWE"))

# Make a dataframe to draw lines between samples of the same batch
Subset <- metadata %>% dplyr::filter(Tank  == "Artemia")
Subset$Time_cont <- round(Subset$Time_cont , 2)
df <- as.data.frame(c(Subset$Time_cont, (4:18)+0.39))
colnames(df) <- "Time_cont"
df <- left_join(df, Subset[c("Time_cont", "BacterialDensity")], by = c("Time_cont"))

# Plot bacterial density in Artemia stocks over time, for fixed samples
ColorsBackgoundBlocks <- c("#ffffff", "#eb1c91")
p_ArtemiaStock_Fixed <- metadata %>%
                        dplyr::filter(Tank  == "Artemia") %>% 
                        ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                        annotate("rect", xmin = 3.7, xmax = 4.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 4.3958, xmax = 5.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 5.3958, xmax = 6.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 6.3958, xmax = 7.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 7.3958, xmax = 8.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 8.3958, xmax = 9.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 9.3958, xmax = 10.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 10.3958, xmax = 11.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 11.3958, xmax = 12.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 12.3958, xmax = 13.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 13.3958, xmax = 14.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 14.3958, xmax = 15.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 15.3958, xmax = 16.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        annotate("rect", xmin = 16.3958, xmax = 17.3958, ymin = 1e5, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                        annotate("rect", xmin = 17.3958, xmax = 18.3958, ymin = 1e5, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                        geom_line(data = df, aes(x = Time_cont, y = BacterialDensity)) +                   
                        geom_point(shape = 21, size = 2, alpha = 1, aes(fill = Timepoint)) +
                        scale_fill_brewer("", palette = "PuRd", labels = c("11 a.m.", "2 p.m.", "5 p.m.", "8 p.m.", "11 p.m.", "2 a.m.", "5 a.m.", "8 a.m.")) +
                        labs(color = "", y = "Bacterial density (cells/mL)", x = "Time (d)") +
                        theme_cowplot() +
                        theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet)) +
                        scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 3e7)) +
                        scale_x_continuous(limits = c(4.2, 18.5), expand = c(0, 0), breaks = seq(4,18,2))
print(p_ArtemiaStock_Fixed)
png("Figures/DYNAMICS-ArtemiaTank-BacterialDensity-Fixed.png", width = 10, height = 4, res = 500, units = "in")
print(p_ArtemiaStock_Fixed)
dev.off()
```

```
## png 
##   2
```

```r
# Remove all variables that are no longer needed
remove(df, Subset, ColorsBackgoundBlocks)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_PlotBacterialDensitiesArtemiaStock-1.png" width="70%" style="display: block; margin: auto;" />


```r
# The time labels for the WE samples were not converted to a continuous time value
metadata$Time_cont[metadata$Tank == "IncomingW"] <- as.numeric(gsub("D", "", metadata$Day[metadata$Tank == "IncomingW"]))

# Plot percentage intact cells of the first treatment steps
ColorsBackgoundBlocks <- c("#ffffff", "#78bdcf")
p_cells_ExchangeWaters <- metadata %>%
                          dplyr::filter(Tank == "IncomingW") %>%
                          ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                          annotate("rect", xmin = 6.51, xmax = 8, ymin = 1e4, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                          annotate("rect", xmin = 8, xmax = 10, ymin = 1e4, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                          annotate("rect", xmin = 10, xmax = 12, ymin = 1e4, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                          annotate("rect", xmin = 12, xmax = 14, ymin = 1e4, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                          annotate("rect", xmin = 14, xmax = 16, ymin = 1e4, ymax = 3e7, alpha = 0.15, fill = ColorsBackgoundBlocks[2]) +
                          annotate("rect", xmin = 16, xmax = 18, ymin = 1e4, ymax = 3e7, alpha = 0.1, fill = ColorsBackgoundBlocks[1]) +
                          geom_point(shape = 21, size = 2, alpha = 0.8, fill = SourceColors[3]) +
                          labs(color = "", y = "Bacterial density (cells/mL)", x = "Time (d)") +
                          scale_x_continuous(limits = c(6.5, 17.5), breaks = c(7, 9, 11, 13, 15, 17)) +
                          scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e4, 3e7)) + 
                          theme_cowplot() +
                          theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_cells_ExchangeWaters)
png("Figures/DYNAMICS-ExchangeWaters-CellDensities.png", width = 5, height = 4, res = 500, units = "in")
print(p_cells_ExchangeWaters)
dev.off()
```

```
## png 
##   2
```

```r
# Remove all variables that are no longer needed
remove(ColorsBackgoundBlocks)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6Plus_PlotBacterialDensitiesExchangeWaters-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Assemble to one plot
g1 <- plot_grid(p_AlgaeStock_Fixed, p_cells_ExchangeWaters, labels = c("A", "B"), ncol = 2, nrow = 1, rel_widths = c(2,2), scale = 1)
g2 <- plot_grid(g1, p_ArtemiaStock_Fixed, labels = c("", "C"), ncol = 1, nrow = 2, rel_widths = c(2,2), scale = 1)
ggsave(file = "Figures/DYNAMICS-SourceCounts.png", width = 9, height = 7, dpi = 700, units = "in", g2)
ggsave(file = "FiguresPublication/DYNAMICS-SourceCounts_300.tiff", width = 9, height = 7, dpi = 300, units = "in", plot = g2, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-SourceCounts_500.tiff", width = 9, height = 7, dpi = 500, units = "in", plot = g2, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-SourceCounts_700.tiff", width = 9, height = 7, dpi = 700, units = "in", plot = g2, compression = "lzw")
# Remove all variables that are no longer needed
remove(p_cells_Fixed, p_ArtemiaStock_Fixed, p_AlgaeStock_Fixed, p_cells_ExchangeWaters, g1, g2)
```


```r
# Wite the counting-results to a csv to reupload them later
write.csv(x = metadata, file = "Results/DYNAMICS-OffSite-Counting.csv")

# Remove variables that are no longer needed
remove(metadata)
```

## Off-site feed flow cytometry measurements

### Load data


```r
# Upload fcs-files
Datapath <- c("Data/FCM/OffSiteFeeds/")
fcsfiles <- list.files(path = Datapath, recursive = TRUE, pattern = ".fcs", full.names = TRUE)
flowData <- flowCore::read.flowSet(files = fcsfiles, transformation = FALSE, pattern = ".fcs", ignore.text.offset = TRUE, emptyValue = FALSE)
# Remove variables that are no longer needed
remove(Datapath, fcsfiles)
```


```r
# Make metadata from sample names
metadata <- data.frame(flowCore::sampleNames(flowData), do.call(rbind, strsplit(substring(flowCore::sampleNames(flowData), 5), split = "_")))
# Clean-up the metadata
colnames(metadata) <- c("Sample_names", "Sample", "Type", "Stain", "Dilution")
metadata$Dilution <- as.character(gsub(".fcs", "", metadata$Dilution))
metadata$Dilution <- as.numeric(metadata$Dilution)
# Import the exact mass of feed that was sonicated
MassInfo <- xlsx::read.xlsx("./Metadata/MetadataFeeds.xlsx", stringsAsFactors = F, sheetIndex = 1)
metadata <- left_join(metadata, MassInfo, by = c("Sample"))
# Remove variables that are no longer needed
remove(MassInfo)
```

### Preprocessing and calculating cell densities


```r
# Select phenotypic features of interest and transform parameters
flowData_transformed <- flowCore::transform(flowData,
                                   `FL1-H` = asinh(`FL1-H`), 
                                   `SSC-H` = asinh(`SSC-H`), 
                                   `FL3-H` = asinh(`FL3-H`), 
                                   `FSC-H` = asinh(`FSC-H`))
remove(flowData)
```


```r
# Bacterial population
sqrcut1 <- matrix(c(9,9,13,13,
                    5,8,13,5),
                  ncol = 2, 
                  nrow = 4)
colnames(sqrcut1) <- c("FL1-H", "FL3-H")
polyGate1 <- polygonGate(.gate = sqrcut1, filterId = "Total Cells")
# Gating quality check
for (i in 1:length(flowData_transformed)){
  print(xyplot(`FL3-H` ~ `FL1-H`, data = flowData_transformed[i],
              filter = polyGate1,
              scales = list(y = list(limits = c(4.5,16)),
                            x = list(limits = c(8,13.2))),
              axis = axis.default,
              nbin = 125,
              par.strip.text = list(col = "black", font = 3, cex = 0.75),
              smooth = FALSE))
}
# Make figure for supplementary information
  # Feed product 1 where cells and background are clearly separated
  p1 <- xyplot(`FL3-H` ~ `FL1-H`, data = flowData_transformed[1],
                filter = polyGate1,
                scales = list(y = list(limits = c(4.5,16)),
                              x = list(limits = c(8,13.2))),
                axis = axis.default,
                nbin = 125,
                par.strip.text = list(cex = 0),
                smooth = FALSE)
  # Feed product 4 where cells and background cannot be separated clearly
  p2 <- xyplot(`FL3-H` ~ `FL1-H`, data = flowData_transformed[2],
                filter = polyGate1,
                scales = list(y = list(limits = c(4.5,16)),
                              x = list(limits = c(8,13.2))),
                axis = axis.default,
                nbin = 125,
                par.strip.text = list(cex = 0),
                smooth = FALSE)
g <- plot_grid(p1, p2, labels = c("A", "B"), ncol = 2, nrow = 1)
ggsave(file = "Figures/QC-GatingFeeds.png", width = 10, height = 4, dpi = 300, units = "in", g)
# Remove variables that are no longer needed
remove(sqrcut1, i, p1, p2, g)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6PlusFeeds_GateCheck-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6PlusFeeds_GateCheck-2.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6PlusFeeds_GateCheck-3.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6PlusFeeds_GateCheck-4.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6PlusFeeds_GateCheck-5.png" width="70%" style="display: block; margin: auto;" />


```r
# Count bacteria
BacterialCount <- flowCore::filter(flowData_transformed, polyGate1) 
BacterialCount <- toTable(summary(BacterialCount))
# Get the analysed volume
vol <- as.numeric(flowCore::fsApply(flowData_transformed, FUN = function(x) x@description$`$VOL`))/1000
# Calculate cell density (cells/mL)
BacterialDensity <- 1000*1000*BacterialCount$true/vol # *1000 to go from µL to mL and *1000 because the samples were 1000x diluted
# 1 mL of dilution buffer was added to the feed particles, so the absolute number of cells is the same as the density
BacterialDensity <- cbind.data.frame(BacterialCount$sample, BacterialDensity)
colnames(BacterialDensity) <- c("Sample_names", "CellCount")
# Recalculate per mass of feedparticles
Results <- left_join(metadata, BacterialDensity, by = c("Sample_names"))
Results$DensityPerG <- Results$CellCount/Results$Mass
# Only for feed 1 the gating will be accurate, for the other samples it was not possible to distinghuish between background and cells
Results$DensityPerG[2:5] <- 0
# Remove variables that are no longer needed
remove(BacterialCount, flowData_transformed, polyGate1, BacterialDensity, vol, metadata)
```


```r
# Read the counting-results of the liquid samples
ResultsLiquidSamples <- read.csv(file = "Results/DYNAMICS-OffSite-Counting.csv")

# Organise the results from the feed in the same way as those of the liquid samples
ResultsFeeds <- cbind.data.frame(0, Results$Sample_names, Results$Sample, Results$Type, Results$Stain, Results$Dilution, 0, 0, 0, 0, 0, 0, 0, Results$DensityPerG, 0)
colnames(ResultsFeeds) <- colnames(ResultsLiquidSamples)

# Combine the results of the liquid samples and those of the solid samples and export the results
Results <- rbind(ResultsLiquidSamples, ResultsFeeds)
# write.csv(x = Results, file = "Results/DYNAMICS-OffSite-Counting-WithDryFeeds.csv")

# Remove variables that are no longer needed
remove(Results, ResultsFeeds, ResultsLiquidSamples)
```

# Evaluate the reliability of the fixation for flow cytometry


```r
# Load the results of the Accuri C6 and Accuri C6Plus measurements
ResultsAccuriC6 <- read.csv("Results/DYNAMICS-OnSite-Counting.csv")
ResultsAccuriC6Plus <- read.csv("Results/DYNAMICS-OffSite-Counting.csv")
# Small changes to make the labels in the datasets compatible
ResultsAccuriC6Plus$Sample <- as.character(gsub("Artemia", "Art", ResultsAccuriC6Plus$Sample))
Results <- dplyr::left_join(ResultsAccuriC6, ResultsAccuriC6Plus, by = c("Sample" = "Sample"))
Results$Tank.x <- as.character(Results$Tank.x)
Results <- Results[!is.na(Results$BacterialDensity.y),] # Not all timepoints were measured on site: remove all those that were only measured off site since we cannot evaluate the fixation effect on these samples

# Fit linear model
LM_Counts <- lm(BacterialDensity.x ~ BacterialDensity.y, data = Results) 
# Plot relation between cell counts of all samples together
p_CorrCounts_Fixed <- Results %>%
                       dplyr::filter(Tank.x %in% c("T1", "T2", "T3", "T4", "T5", "Algae", "Art")) %>%
                       dplyr::mutate(Tank.x = factor(Tank.x, levels = c("T1", "T2", "T3", "T4", "T5", "Algae", "Art"))) %>% 
                       ggplot(data = ., aes(x = BacterialDensity.x, y = BacterialDensity.y)) +
                       geom_point(shape = 21, size = 3, alpha = 1, aes(fill = Tank.x)) +
                       scale_fill_manual(values = AllTankColors, labels = c("T1","T2","T3","T4","T5","Algae","Artemia")) +
                       geom_vline(xintercept = 0, size = 1) +
                       labs(fill = "", y =  "Bacterial density fixed samples (cells/mL)", x = "Bacterial density fresh samples (cells/mL)") +
                       guides(color = FALSE) +
                       geom_abline(intercept = 0, slope = 1) +
                       scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 1e8)) +
                       scale_x_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 1e8)) +
                       coord_fixed() +
                       guides(fill = guide_legend(override.aes = list(size = 4))) +
                       theme_cowplot() +
                       theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.line.y = element_blank(), legend.title = element_text(size = 13), legend.text = element_text(size = 13)) + 
                       annotate(geom = "text", x = 0.9e7, y = 1.7e5, label = paste0("Adj.R.sq. = ", format(summary(LM_Counts)$r.squared, digits = 2), "\nCp = ", formatC(cor(log10(Results$BacterialDensity.x), log10(Results$BacterialDensity.y)), format = "f", digits = 2), "\nn = ", dim(Results)[1]), hjust = 0)

# Fit linear model
LM_Counts <- lm(AlgalDensity.x ~ AlgalDensity.y, data = Results) 
# Plot relation between cell counts of all samples together, for the algae samples
p_CorrCounts_Fixed_Alg <- Results %>%
                       dplyr::filter(Tank.x %in% c("T1", "T2", "T3", "T4", "T5", "Algae", "Art") & AlgalDensity.x > 1e3 & AlgalDensity.y > 1e3) %>%
                       dplyr::mutate(Tank.x = factor(Tank.x, levels = c("T1", "T2", "T3", "T4", "T5", "Algae", "Art"))) %>% 
                       ggplot(data = ., aes(x = AlgalDensity.x, y = AlgalDensity.y)) +
                       geom_point(shape = 21, size = 3, alpha = 1, aes(fill = Tank.x)) +
                       scale_fill_manual(values = AllTankColors, labels = c("T1","T2","T3","T4","T5","Algae","Artemia")) +
                       geom_vline(xintercept = 0, size = 1) +
                       labs(fill = "", y = "Algal density fixed samples (cells/mL)", x = "Algal density fresh samples (cells/mL)") +
                       guides(color = FALSE) +
                       geom_abline(intercept = 0, slope = 1) +
                       scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e3, 5e6)) +
                       scale_x_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e3, 5e6)) +
                       coord_fixed() +
                       theme_cowplot() +
                       theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.line.y = element_blank(), legend.title = element_text(size = 13), legend.text = element_text(size = 13)) + 
                       annotate(geom = "text", x = 2e5, y = 1.8e3, label = paste0("Adj.R.sq. = ", format(summary(LM_Counts)$r.squared, digits = 2), "\nCp = ", formatC(cor(log10(Results$BacterialDensity.x), log10(Results$BacterialDensity.y)), format = "f", digits = 2), "\nn = ", dim(Results)[1]), hjust = 0)

# Get the legend from 1 plot and remove the legends from the plots
Legend <- cowplot::get_legend(p_CorrCounts_Fixed)
p_CorrCounts_Fixed <- p_CorrCounts_Fixed + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
p_CorrCounts_Fixed_Alg <- p_CorrCounts_Fixed_Alg + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
# Assemble to one plot
g <- plot_grid(p_CorrCounts_Fixed, p_CorrCounts_Fixed_Alg, Legend, labels = c("B", "C", ""), ncol = 3, nrow = 1, rel_widths = c(2, 2, 1))
g1  <- plot_grid(p_gate, g, labels = c("A", "", ""), ncol = 1, nrow = 2)
print(g1)
ggsave(file = "Figures/QC-FCMControls.png", width = 10, height = 10, dpi = 500, units = "in", g1)


# Remove all variables that will not be used further
remove(ResultsAccuriC6, ResultsAccuriC6Plus, Results, p_CorrCounts_Fixed_Tank, p_CorrCounts_Fixed_Alg, p_CorrCounts_Fixed, LM_Counts, Legend, g)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/ComparisonCountsFixation-1.png" width="70%" style="display: block; margin: auto;" />

# Processing Illumina data

## Upload sample metadata


```r
# Upload metadata
Metadata <- read.xlsx(xlsxFile = "Metadata/MetadataSequencing.xlsx")
# Split the sample-identifiers
Metadata <- cbind(Metadata, do.call(rbind, strsplit(as.character(Metadata$SampleIdentifierFCM), split = ".", fixed = TRUE)))
colnames(Metadata)[6:9] <- c("Tank", "Day", "Timepoint", "Feeding_status")
Metadata$Day <- as.numeric(gsub("D", "", Metadata$Day))
```

## Evaluate the loss of reads over the processing steps


```r
# file direcrory
fldr <- "Data/Illumina/fastq/"
# Load info from the summary-files
filelist <- list.files(fldr)
crfn <- grep(".*contigs.report", filelist, value = TRUE)
fn <- sub(".contigs.report", "", crfn)
ini <- data.table::fread(paste(fldr, "/", crfn, sep = ""), header = TRUE)
csum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.summary", sep = ""), header = TRUE)
firsttrimsum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.summary", sep = ""), header = TRUE)
uniquesum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.summary", sep = ""), header = TRUE)
postalnsum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.good.filter.summary", sep = ""), header = TRUE)
preclussum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.good.filter.unique.precluster.summary", sep = ""), header = TRUE)
postuchimeclasssum <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.summary", sep = ""), header = TRUE)

# Remove all variables that will not be used further
remove(filelist, crfn, ini)
```


```r
countdfprepr <- data.frame(uniqueseqs = c(nrow(csum),
                                        nrow(firsttrimsum),
                                        nrow(uniquesum),
                                        nrow(postalnsum),
                                        nrow(preclussum),
                                        nrow(postuchimeclasssum)),
                           totalseqs = c(nrow(csum),
                                       nrow(firsttrimsum),
                                       sum(uniquesum$numSeqs),
                                       sum(postalnsum$numSeqs),
                                       sum(preclussum$numSeqs),
                                       sum(postuchimeclasssum$numSeqs)),
                           step = c("Contigs", 
                                    "Initial trim",
                                    "Unique",
                                    "Alignment",
                                    "Precluster",
                                    "UChime"))
countdfprepr.m <- melt(countdfprepr)
p_removalreads <- ggplot(data = countdfprepr.m, aes(x = step, y = value, fill = variable)) +
                     geom_bar(stat = "identity", position = "dodge",  color = "black") +
                     scale_fill_manual(labels = c("Unique", "Total"), name = "", values = c(SingleColor, SingleColor2)) +
                     scale_x_discrete(limits = c("Contigs","Initial trim", "Unique", "Alignment", "Precluster", "UChime")) +
                     geom_text(aes(y = value, label = format(round(as.numeric(value), 1), big.mark = ",")), vjust = -0.25, color = "black", size = 3, position = position_dodge(width = 1)) +
                     labs(x = "Processing step", y = "Number of reads") +
                     scale_y_continuous(labels = function(x) format(x, big.mark = ",", scientific = FALSE)) +
                     theme_cowplot() + 
                     theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_removalreads)
png("Figures/QC-RemovalReadsDuringProcessing.png", width = 10, height = 5, res = 500, units = "in")
print(p_removalreads)
dev.off()

# Remove all variables that will not be used further
remove(csum, firsttrimsum, uniquesum, postalnsum, preclussum, postuchimeclasssum, countdfprepr, countdfprepr.m, p_removalreads)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/stepwiseseqremoval-1.png" width="800px" height="400px" style="display: block; margin: auto;" />

## Upload the classification results


```r
# Load taxonomy from Mothur output files
otutaxonomy <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.0.03.cons.taxonomy", sep = ""), header = TRUE)
taxonomy.spl <- preformattax(otutaxonomy)
taxonomy.np <- taxonomy.spl %>% dplyr::select(-dplyr::contains("Prob"))

# Load the shared file for getting the OTU abundances per sample
shared <- data.table::fread(paste(fldr, "/", fn, ".trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.shared", sep = ""), header = TRUE)
shared <- as.data.frame(shared)
desgroups <- shared$Group # the samplenames
shared.x <- shared[, 4:ncol(shared)] # only the OTU info
rownames(shared.x) <- desgroups
shared.t <- as.data.frame(t(shared.x))

# Removal of singletons from the OTU table and the taxonomy results (singletons = OTU's that only occur once over the entire dataset)
shared.t.ns <- shared.t[which(rowSums(shared.t)!=1),]
taxonomy.np.ns <- taxonomy.np[which(rownames(taxonomy.np) %in% rownames(shared.t.ns)),]

# Save the OTU table (with and without singletons) in an excel sheet
tmp.otu <- cbind(rownames(shared.t), shared.t)
colnames(tmp.otu) <- c("OTU", colnames(shared.t))
tmp.otu.ns <- cbind(rownames(shared.t.ns), shared.t.ns)
colnames(tmp.otu.ns) <- c("OTU", colnames(shared.t.ns))
wb <- openxlsx::write.xlsx(x = tmp.otu, file = "Results/DYNAMICS-OTUTable.xlsx", sheetName = "OTU_table")
openxlsx::addWorksheet(wb,sheetName = "OTU_table_NoSingletons")
openxlsx::writeData(wb, sheet = "OTU_table_NoSingletons", x = tmp.otu.ns)
openxlsx::addWorksheet(wb,sheetName = "Classification")
openxlsx::writeData(wb, sheet = "Classification", x = taxonomy.np)
openxlsx::addWorksheet(wb,sheetName = "Classification_NoSingletons")
openxlsx::writeData(wb, sheet = "Classification_NoSingletons", x = taxonomy.np.ns)
openxlsx::saveWorkbook(wb, file = "Results/DYNAMICS-OTUTable.xlsx", overwrite = TRUE)

# Remove all variables that will not be used further
remove(fldr, fn, otutaxonomy, taxonomy.spl, taxonomy.np, shared, desgroups, shared.x, shared.t, wb, tmp.otu, tmp.otu.ns)
```

## Upload cell densities to calculate absolute taxon abundances


```r
# Load the Accuri C6+ data
Counts <- read.csv("Results/DYNAMICS-OffSite-Counting-WithDryFeeds.csv")
Counts <- Counts[c("Sample", "BacterialDensity")]
Counts$Sample <- gsub("Artemia", "Art", Counts$Sample)
# Sample names are not perfectly matching: "Artemia" --> "Art"
Metadata <- left_join(Metadata, Counts, by = c("SampleIdentifierFCM" = "Sample"))

# For three samples the cell densities of the fixed samples went wrong, so for these we will use the cell densities that were measured on site
  # Samples for which we still need cell densities
  tmp <- Metadata$SampleIdentifierFCM[is.na(Metadata$BacterialDensity) & Metadata$Feeding_status %in% c("Algae", "Art", "B")]
  tmp2 <- gsub(".B", ".A", tmp)
  # For the algal sample, the fresh one is also missing
  Counts <- read.csv("Results/DYNAMICS-OnSite-Counting.csv")
  Counts <- Counts[c("Sample", "BacterialDensity")]
  InFresh <- tmp2[tmp2 %in% Counts$Sample]
  Metadata$BacterialDensity[Metadata$SampleIdentifierFCM %in% gsub(".A", ".B", InFresh)] <- Counts$BacterialDensity[Counts$Sample %in% InFresh]
  # Algae sample
  tmp <- Metadata$SampleIdentifierFCM[is.na(Metadata$BacterialDensity) & Metadata$Feeding_status %in% c("Algae", "Art", "B")]
  Metadata$BacterialDensity[Metadata$SampleIdentifierFCM == tmp] <- mean(Metadata$BacterialDensity[Metadata$Feeding_status == "Algae"], na.rm = TRUE)

# Remove all variables that will not be used further
remove(Counts, tmp, tmp2, InFresh)
```

## Evaluate control samples


```r
# Upload the real composition of the Zymo mock
MockComposition <- read.xlsx(xlsxFile = "./Metadata/Mock_ZRC190811.xlsx")
MockComposition$Group <- "Mock"
colnames(MockComposition) <- c("Genus", "Species", "RelativePercentage", "Group")
# Get samples to compare dilution series of the zymo and chelex extractions
Identifiers <- c("250", "251", "252", "253", "254", "255", "Z")
shared.controls <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.t.ns[Identifiers], rowSums(shared.controls))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:8]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Add the real mock composition to the table
Aggregated <- ddply(df.all, .(Genus), numcolwise(sum))
df.tmp <- left_join(Aggregated[c("Genus", Identifiers)], MockComposition[c("Genus", "RelativePercentage")], by = c("Genus"))
colnames(df.tmp) <- c("Genus", "Zymo8", "Zymo7", "Zymo6", "Zymo5", "Zymo4", "Zymo3", "Zymo0", "Theoretical")
# Phyloseq object
SubsetOTU <- otu_table(as.matrix(df.tmp[c("Zymo8", "Zymo7", "Zymo6", "Zymo5", "Zymo4", "Zymo3", "Zymo0", "Theoretical")]), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.tmp["Genus"]))
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX)
Subsetphyseqobj <- Subsetphyseqobj %>%
                    tax_glom(taxrank = "Genus") %>%
                    transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                    psmelt()

# Subset to 10^8 - 10^5 
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX)
Subsetphyseqobj <- prune_samples(samples = c("Zymo8", "Zymo7", "Zymo6", "Zymo5", "Theoretical"), x = Subsetphyseqobj)
Subsetphyseqobj <- Subsetphyseqobj %>%
                    tax_glom(taxrank = "Genus") %>%
                    transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                    psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_comp_mock <- Subsetphyseqobj %>%
                  dplyr::mutate(Sample = factor(Sample, levels = c("Zymo8", "Zymo7", "Zymo6", "Zymo5", "Zymo4", "Zymo3", "Zymo0", "Chelex8", "Chelex7", "Chelex6", "Chelex5", "Chelex4", "Chelex3", "Chelex0", "Theoretical"))) %>% 
                  dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                  ggplot(data = ., aes(x = Sample, y = 100*Abundance, fill = Genus)) + 
                  geom_bar(stat = "identity", colour = "black") +
                  scale_fill_manual(values = c(ColorBlocksFacet, Series8), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                  scale_x_discrete("", labels = c(expression(atop("Zymo", paste("10"^"8"))),expression(atop("Zymo", paste("10"^"7"))), expression(atop("Zymo", paste("10"^"6"))), expression(atop("Zymo", paste("10"^"5"))), "Theoretical")) +
                  guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                  ylab("Relative abundance (%)") +
                  theme_cowplot() +
                  theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.text.align = 0, axis.title.x = element_blank()) 
print(p_comp_mock)
png("Figures/QC-DilutionSeries-ZymoMock.png", width = 8, height = 4,  res = 300, units = "in")
print(p_comp_mock)
dev.off()

# Get median abundance of contaminating OTU's
Identifiers <- c("250", "251", "252", "253")
shared.controls <- shared.t.ns[Identifiers]
shared.controls <- sweep(shared.controls, 2, colSums(shared.controls), `/`)
shared.controls$Means <- rowMeans(shared.controls)
shared.controls$Medians <- rowMedians(as.matrix(shared.controls[c("250", "251", "252", "253")]))
OTUsFromMock <- c("Otu00025", "Otu00039", "Otu00038", "Otu00055", "Otu00060", "Otu00071", "Otu00059", "Otu00083")
shared.controlsContaminants <- shared.controls[!(row.names(shared.controls) %in% OTUsFromMock), ]
# Get relative abundances of the contaminants
x <- shared.controlsContaminants[c("250", "251", "252", "253")]
x <- x[x > 0]
max(x)
median(x)

# Remove all variables that will not be used further
remove(MockComposition, Aggregated, df.all, df.tmp, shared.controls, Subsetphyseqobj, SubsetOTU, SubsetTAX, Top, shared.controlsContaminants, x, Identifiers, OTUsFromMock)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/Zymo_DilutionMock-1.png" width="70%" style="display: block; margin: auto;" />

## Composition blank


```r
# Select the samples of the water in the rearing tanks
Identifiers <- c("Z")
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples from the rearing water
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_blank_rel <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Info$Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = WholeCommunityOrSorted, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", color = "black") + 
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative abundance (%)") +
                      xlab("") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, axis.title.x = element_blank(), axis.text.x = element_blank(), axis.ticks.x = element_blank())
print(p_blank_rel)
png("Figures/QC-Blank-Genus.png", width = 7, height = 5,  res = 300, units = "in")
print(p_blank_rel)
dev.off()

# Remove all variables that will not be used further
remove(Info, shared.selection, SubsetINFO, SubsetOTU, SubsetTAX, Top, Identifiers, Subsetphyseqobj, df.all, Aggregated)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/Zymo_Blank-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Assemble to one plot
g1 <- plot_grid(NULL, p_blank_rel, NULL, ncol = 3, nrow = 1, rel_widths = c(0.75,2,0.75), scale = 1)
g2 <- plot_grid(p_comp_mock, g1, labels = c("A", "B"), ncol = 1, nrow = 2, scale = 1)

ggsave(file = "Figures/QC-ControlsSequencing.png", width = 11, height = 10, dpi = 300, units = "in", g2)
# Remove all variables that are no longer needed
remove(p_comp_mock, p_blank_rel, g1, g2)
```

## Scaling


```r
# Make phyloseq object from the data without all the control samples (don't take into account their reads for scaling)
IdentifierControlSamples <- c("250", "251", "252", "253", "254", "255", "Z")
otumat.ns <- as.matrix(shared.t.ns[Metadata$Identifier[!Metadata$Identifier %in% IdentifierControlSamples]])
taxmat.ns <- as.matrix(taxonomy.np.ns)
info <- Metadata[Metadata$Identifier %in% colnames(otumat.ns),]
OTU <- otu_table(otumat.ns, taxa_are_rows = TRUE)
TAX <- tax_table(taxmat.ns)
INFO <- sample_data(info)
rownames(INFO) <- INFO$Identifier
physeqobj <- phyloseq(OTU, TAX, INFO)

# Do the scaling
physeqobj <- scale_reads(physeqobj)

# Save the scaled data as the otu-table
shared.t.ns.old <- shared.t.ns # Keep this to check later which OTU's were not used (needed in the community assembly part)
shared.t.ns <- as.data.frame(otu_table(physeqobj))
taxonomy.np.ns <- as.data.frame(physeqobj@tax_table@.Data)

# Save the OTU table (with and without singletons) in an excel sheet
tmp.otu.ns <- cbind(rownames(shared.t.ns), shared.t.ns)
colnames(tmp.otu.ns) <- c("OTU", colnames(shared.t.ns))
wb <- openxlsx::write.xlsx(x = tmp.otu.ns, file = "Results/DYNAMICS-OTUTable-Scaling.xlsx", sheetName = "OTU_table")
openxlsx::addWorksheet(wb,sheetName = "Classification")
openxlsx::writeData(wb, sheet = "Classification", x = taxonomy.np.ns)
openxlsx::saveWorkbook(wb, file = "Results/DYNAMICS-OTUTable-Scaling.xlsx", overwrite = TRUE)

# Remove all variables that will not be used further
remove(OTU, TAX, INFO, info, physeqobj, otumat.ns, taxmat.ns, IdentifierControlSamples)
```

# Community composition and dynamics

## Composition of the rearing water


```r
# Select the part from before the crash of T1 and T4
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)


# Select the samples of the water in the rearing tanks
Identifiers <- InfoTanks$Identifier
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples from the rearing water
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_tanks_rel <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Info$Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = Day, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", color = "black") + 
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"])), name = "Genus   ") +
                      facet_grid(Tank ~ .) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1, nrow = 5)) +
                      ylab("Relative abundance (%)") +
                      xlab("Time (d)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, legend.position = "bottom", legend.margin=margin(c(0,0,0,0)))
print(p_tanks_rel)

# Remove all variables that will not be used further
remove(Identifiers, shared.selection, df.all, SubsetINFO, SubsetOTU, SubsetTAX, Top, Aggregated, InfoTanks, InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesTanks_RelAbundanceOnlyNotCrashed-1.png" width="90%" style="display: block; margin: auto;" />


```r
# Get legend 
LegendComposition <- cowplot::get_legend(p_tanks_rel)
p_tanks_rel <- p_tanks_rel + theme(legend.position = "none")
# Assemble to one plot
g1 <- plot_grid(p_cells_Fixed_v, p_tanks_rel, labels = c("A", "B"), ncol = 2, nrow = 1, rel_widths = c(2,3), scale = 1)
g2 <- plot_grid(NULL, LegendComposition, ncol = 2, nrow = 1, rel_widths = c(0.1,3), scale = 1)
g3 <- plot_grid(g1, g2, ncol = 1, nrow = 2, rel_heights = c(4,1), scale = 1)
ggsave(file = "Figures/DYNAMICS-Tanks-AbundanceAndComposition.png", width = 11, height = 10, dpi = 300, units = "in", g3)
ggsave(file = "FiguresPublication/DYNAMICS-Tanks-AbundanceAndComposition_300.tiff", width = 11, height = 10, dpi = 300, units = "in", plot = g3, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-Tanks-AbundanceAndComposition_500.tiff", width = 11, height = 10, dpi = 500, units = "in", plot = g3, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-Tanks-AbundanceAndComposition_700.tiff", width = 11, height = 10, dpi = 700, units = "in", plot = g3, compression = "lzw")
# Remove all variables that are no longer needed
remove(p_tanks_rel, p_cells_Fixed_v, LegendComposition, g1, g2, g3)
```


```r
# Upload the count data
ResultsAccuriC6Plus <- read.csv("Results/DYNAMICS-OffSite-Counting.csv")
ResultsAccuriC6Plus <- ResultsAccuriC6Plus %>% dplyr::filter(Tank %in% c("T1", "T2", "T3", "T4", "T5") & Feeding_status %in% c("B"))

# Select the illumina samples
Identifiers <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples from the rearing water
shared.selection <- shared.t.ns[Identifiers]
# Make phyloseq
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
SubsetphyseqobjTanks <- Subsetphyseqobj %>%
                        dplyr::mutate(Sample = factor(Sample, levels = Info$Identifiers)) %>%
                        dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"])))

# Correlation with the algal abundance
CombinedData <- left_join(SubsetphyseqobjTanks[c("OTU", "Day", "Abundance", "SampleIdentifierFCM", "Regnum", "Phylum", "Classis", "Ordo", "Familia", "Genus")], ResultsAccuriC6Plus[c("Sample", "BacterialDensity", "AlgalDensity")], by = c("SampleIdentifierFCM" = "Sample"))
CombinedData <- CombinedData[complete.cases(CombinedData),]
CombinedData$AbsoluteAbundance <- CombinedData$Abundance*CombinedData$BacterialDensity

CorrelationResults <- NULL
for (i in unique(CombinedData$OTU)){
  # Calculate the correlation
  DataSelectedOTU <- CombinedData[CombinedData$OTU == i,]
  Correlation <- cor.test(DataSelectedOTU$AbsoluteAbundance, DataSelectedOTU$AlgalDensity, method = "pearson")
  # Get some info on the importance of this OTU in the tanks
  MaxRelAbund <- max(DataSelectedOTU$Abundance)
  MaxAbsAbund <- max(DataSelectedOTU$AbsoluteAbundance)  
  # Save the results
  Result <- cbind.data.frame(i, DataSelectedOTU[1, c("Regnum", "Phylum", "Classis", "Ordo", "Familia", "Genus")], Correlation$estimate, Correlation$p.value, MaxRelAbund, MaxAbsAbund)
  CorrelationResults <- rbind(CorrelationResults, Result)
}
colnames(CorrelationResults) <- c("OTU", "Regnum", "Phylum", "Classis", "Ordo", "Familia", "Genus", "Pearson", "Pvalue", "MaxRelAbund", "MaxAbsAbund")
CorrelationResults <- CorrelationResults[complete.cases(CorrelationResults),]
# Keep significant correlations only
CorrelationResultsSignificant <- CorrelationResults[CorrelationResults$Pvalue < 0.001,]
# Save the results of the correlation with the algae
write.csv(x = CorrelationResultsSignificant, file = "Results/DYNAMICS-CorrelationAlgae.csv")

# Remove all variables that will not be used further
remove(Identifiers, Info, shared.selection, SubsetINFO, SubsetOTU, SubsetTAX, Subsetphyseqobj, SubsetphyseqobjTanks, CombinedData, Correlation, CorrelationResults, CorrelationResultsSignificant, DataSelectedOTU, Result, ResultsAccuriC6Plus, MaxAbsAbund, MaxRelAbund, i)
```

## Community composition during start-up


```r
Identifiers <- Metadata$Identifier[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & Metadata$Day %in% c(1, 2) & !(Metadata$Feeding_status %in% c("Sed", "Mix"))]
Identifiers <- Identifiers[!is.na(Identifiers)]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the startup
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
# Plot
p_tanks <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes_string(x = "Day", y = "Abundance", fill = "Genus")) + 
                      geom_bar(stat = "identity", color = "black") + # colour = "black"
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      facet_grid(. ~ Tank) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      scale_x_continuous(breaks = c(1, 2)) +
                      ylab("Relative abundance") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, plot.margin = margin(1, 0, 0, 1, "cm"))
print(p_tanks)


# Get absolute abundances
Subsetphyseqobj$AbsoluteAbundance <- Subsetphyseqobj$Abundance*Subsetphyseqobj$BacterialDensity
p_tanks_abs <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes_string(x = "Day", y = "AbsoluteAbundance", fill = "Genus")) + 
                      geom_bar(stat = "identity", color = "black") + # colour = "black"
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      facet_grid(. ~ Tank) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      scale_x_continuous(breaks = c(1, 2)) +
                      ylab("Absolute abundance (cells/mL)") +
                      scale_y_continuous(labels = function(x) format(x, scientific = TRUE)) +
                      theme_cowplot() +
                      theme(legend.text.align = 0, plot.margin = margin(1, 0, 0, 1, "cm"))
print(p_tanks_abs)

# Remove all variables that will not be used further
remove(Identifiers, Info, shared.selection, SubsetINFO, SubsetOTU, SubsetTAX, Aggregated, df.all, Subsetphyseqobj, Top)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesTanks_Startup-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesTanks_Startup-2.png" width="70%" style="display: block; margin: auto;" />


```r
ResultsAccuriC6 <- read.csv("Results/DYNAMICS-OnSite-Counting.csv")
# The first data points is from right after filling the tank and the MIC pretreatment
FirstTimePoints <- ResultsAccuriC6 %>% dplyr::filter(Time_cont < 2.34 & Tank %in% c("T1", "T2", "T3", "T4", "T5", "Trans", "Desinf"))
FirstTimePoints$Time_cont[FirstTimePoints$Time_cont == unique(FirstTimePoints$Time_cont)[1]] <- "Day 2" # 
FirstTimePoints$Time_cont[FirstTimePoints$Time_cont == unique(FirstTimePoints$Time_cont)[2]] <- "Day 0" # Before MIC
FirstTimePoints$Time_cont[FirstTimePoints$Time_cont == unique(FirstTimePoints$Time_cont)[3]] <- "Day 1" # After MIC
FirstTimePoints$Time_cont[FirstTimePoints$Tank == "Trans"] <- "Transport water"
FirstTimePoints$Time_cont[FirstTimePoints$Tank == "Desinf"] <- "After desinfection"
FirstTimePoints$Tank <- as.character(FirstTimePoints$Tank)
FirstTimePoints$Tank[FirstTimePoints$Tank == "Trans"] <- "All"
FirstTimePoints$Tank[FirstTimePoints$Tank == "Desinf"] <- "All"

# Plot cell densities of the first treatment steps
p_cells_FirstTreatmentSteps <- FirstTimePoints %>%
                               dplyr::filter(Stain == "SG" & Type == "FRESH") %>%
                               dplyr::mutate(Time_cont = factor(Time_cont, levels = c("Transport water", "After desinfection", "Day 0", "Day 1", "Day 2"))) %>% 
                               ggplot(data = ., aes(x = Time_cont, y = BacterialDensity)) +
                               geom_point(shape = 21, size = 4, alpha = 1, aes(fill = Tank)) +
                               scale_fill_manual(values = c(ColorBlocksFacet, TankColors)) +
                               labs(color = "", y = "Bacterial density (cells/mL)", x = "") +
                               theme_cowplot() +
                               scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1e5, 4e6)) +
                               scale_x_discrete(labels = c(expression(atop("Transport", paste("water"))), expression(atop("After", paste("desinfection"))), expression(atop("Day 1", paste("(after filling)"))), expression(atop("Day 1", paste("(after adding larvae)"))) , "Day 2")) +
                               theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), plot.margin = margin(1, 5, 0, 1, "cm"))
print(p_cells_FirstTreatmentSteps)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AccuriC6_PlotBacterialDensitiesInitialTreatmentSteps-1.png" width="70%" style="display: block; margin: auto;" />

```r
# Remove variables that are no longer needed
remove(ResultsAccuriC6, FirstTimePoints)
```


```r
# Assemble to one plot
g <- plot_grid(p_cells_FirstTreatmentSteps, p_tanks, p_tanks_abs, labels = c("A", "B", "C"), ncol = 1, nrow = 3)
ggsave(file = "Figures/DYNAMICS-StartUp-DensityComposition.png", width = 11, height = 14, dpi = 700, units = "in", g)
# Remove variables that are no longer needed
remove(p_cells_FirstTreatmentSteps, p_tanks, p_tanks_abs, g)
```

## Community composition in the live feeds, dry feeds and exchange waters

### Algae


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "Algae"]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the algae
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_algae <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = Day, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      scale_x_continuous(breaks = c(2, 4, 6, 8, 10, 12, 14, 16, 18)) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative abundance (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0)
print(p_algae)
png("Figures/SOURCES-Algae-Genus-Relative.png", width = 8, height = 4,  res = 300, units = "in")
print(p_algae)
dev.off()

# Remove all variables that will not be used further
remove(Aggregated, df.all, shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Top, Info, SubsetINFO, Identifiers)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesSources_Algae-1.png" width="70%" style="display: block; margin: auto;" />


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "Algae"]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the algae
shared.selection <- shared.t.ns[Identifiers]
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)})

# Core community
core.taxa.standard <- core_members(x = Subsetphyseqobj, detection = 0, prevalence = 75/100, include.lowest = FALSE)
CoreComposition <- NULL
AbsoluteAbundances <- as.data.frame(sample_data(Subsetphyseqobj))[,c("Identifier", "BacterialDensity")]
for (i in core.taxa.standard){
  # Get the info of the selected OTU
  Abundances <- as.data.frame(otu_table(Subsetphyseqobj)[i,])
  AbsoluteAbundance <- Abundances*t(AbsoluteAbundances[colnames(Abundances), "BacterialDensity"])
  Classification <- as.data.frame(tax_table(Subsetphyseqobj)[i, c("Familia","Genus")])

  # Check how often this OTU is present and how frequently it's abundance is higher than 1 %
  Abundant <- sum(Abundances > 0.01)

  # Check the average ebundance when this OTU is present - relative abundances
  MeanAbundance <- rowMeans(Abundances)
  sdAbundance <- sd(Abundances)
  MinAbundance <- min(Abundances)
  MaxAbundance <- max(Abundances)

  # Check the average ebundance when this OTU is present - absolute abundances
  MeanAbsAbundance <- rowMeans(AbsoluteAbundance)
  sdAbsAbundance <- sd(AbsoluteAbundance)
  MinAbsAbundance <- min(AbsoluteAbundance)
  MaxAbsAbundance <- max(AbsoluteAbundance)

  # Save the results
  Result <- cbind.data.frame(i, Classification, Abundant, MeanAbundance, sdAbundance, MinAbundance, MaxAbundance, MeanAbsAbundance, sdAbsAbundance, MinAbsAbundance, MaxAbsAbundance)
  CoreComposition <- rbind(CoreComposition, Result)
}
# Write the results to an excel
write.csv(x = CoreComposition, file = "Results/DYNAMICS-CoreMembers-Algae.csv")

# Quantify batch differences
Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
meanBray <- mean(Bray)
sdBray <- sd(Bray)
Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
meanJaccard <- mean(Jaccard)
sdJaccard <- sd(Jaccard)
Bray.part <- bray.part(t(as.matrix(shared.selection)))
meanBray.turnover <- mean(Bray.part$bray.gra)
sdBray.turnover <- sd(Bray.part$bray.gra)
meanBray.abundancevariation <- mean(Bray.part$bray.bal)
sdBray.abundancevariation <- sd(Bray.part$bray.bal)

# Remove all variables that will not be used further
remove(shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Info, SubsetINFO, Identifiers, CoreComposition, Abundances, Classification, Bray, meanBray, sdBray, Jaccard, meanJaccard, sdJaccard, Bray.part, meanBray.turnover, sdBray.turnover, meanBray.abundancevariation, sdBray.abundancevariation, Result, sdAbundance, i, MinAbundance, MaxAbundance, MeanAbundance, Abundant)
```

### Artemia


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "Art"]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the Artemia
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_artemia <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = Day, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      scale_x_continuous(breaks = c(2, 4, 6, 8, 10, 12, 14, 16, 18)) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative abundance (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0)
print(p_artemia)
png("Figures/SOURCES-Artemia-Genus-Relative.png", width = 10, height = 4,  res = 300, units = "in")
print(p_artemia)
dev.off()

# Remove all variables that will not be used further
remove(Aggregated, df.all, shared.selection, SubsetOTU, SubsetTAX, Top, SubsetINFO, Identifiers) # Subsetphyseqobj, Info
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesSources_Artemia-1.png" width="70%" style="display: block; margin: auto;" />


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "Art"]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the Artemia
shared.selection <- shared.t.ns[Identifiers]
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)})

# Core community
core.taxa.standard <- core_members(x = Subsetphyseqobj, detection = 0, prevalence = 75/100, include.lowest = FALSE)
CoreComposition <- NULL
AbsoluteAbundances <- as.data.frame(sample_data(Subsetphyseqobj))[,c("Identifier", "BacterialDensity")]
for (i in core.taxa.standard){
  # Get the info of the selected OTU
  Abundances <- as.data.frame(otu_table(Subsetphyseqobj)[i,])
  AbsoluteAbundance <- Abundances*t(AbsoluteAbundances[colnames(Abundances), "BacterialDensity"])
  Classification <- as.data.frame(tax_table(Subsetphyseqobj)[i, c("Familia","Genus")])

  # Check how often this OTU is present and how frequently it's abundance is higher than 1 %
  Abundant <- sum(Abundances > 0.01)

  # Check the average ebundance when this OTU is present - relative abundances
  MeanAbundance <- rowMeans(Abundances)
  sdAbundance <- sd(Abundances)
  MinAbundance <- min(Abundances)
  MaxAbundance <- max(Abundances)

  # Check the average ebundance when this OTU is present - absolute abundances
  MeanAbsAbundance <- rowMeans(AbsoluteAbundance)
  sdAbsAbundance <- sd(AbsoluteAbundance)
  MinAbsAbundance <- min(AbsoluteAbundance)
  MaxAbsAbundance <- max(AbsoluteAbundance)

  # Save the results
  Result <- cbind.data.frame(i, Classification, Abundant, MeanAbundance, sdAbundance, MinAbundance, MaxAbundance, MeanAbsAbundance, sdAbsAbundance, MinAbsAbundance, MaxAbsAbundance)
  CoreComposition <- rbind(CoreComposition, Result)
}
# Write the results to an excel
write.csv(x = CoreComposition, file = "Results/DYNAMICS-CoreMembers-Artemia.csv")

# Quantify batch differences
Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
meanBray <- mean(Bray)
sdBray <- sd(Bray)
Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
meanJaccard <- mean(Jaccard)
sdJaccard <- sd(Jaccard)
Bray.part <- bray.part(t(as.matrix(shared.selection)))
meanBray.turnover <- mean(Bray.part$bray.gra)
sdBray.turnover <- sd(Bray.part$bray.gra)
meanBray.abundancevariation <- mean(Bray.part$bray.bal)
sdBray.abundancevariation <- sd(Bray.part$bray.bal)

# Remove all variables that will not be used further
remove(shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Info, SubsetINFO, Identifiers, CoreComposition, Abundances, Classification, Bray, meanBray, sdBray, Jaccard, meanJaccard, sdJaccard, Bray.part, meanBray.turnover, sdBray.turnover, meanBray.abundancevariation, sdBray.abundancevariation, Result, sdAbundance, i, MinAbundance, MaxAbundance, MeanAbundance, Abundant)
```

### Dry feed


```r
Identifiers <- Metadata$Identifier[Metadata$SampleIdentifier %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5")]
Identifiers <- Identifiers[!is.na(Identifiers)]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the dry feed
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_dryfeed <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = SampleIdentifierFCM, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative abundance (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, axis.title.x = element_blank())
print(p_dryfeed)
png("Figures/SOURCES-DryFeed-Genus-Relative.png", width = 8, height = 4,  res = 300, units = "in")
print(p_dryfeed)
dev.off()

# Remove all variables that will not be used further
remove(Aggregated, df.all, shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Top, Info, SubsetINFO, Identifiers)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesSources_DryFeed-1.png" width="70%" style="display: block; margin: auto;" />

### Exchange water


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "IncomingW"]
Identifiers <- Identifiers[!is.na(Identifiers)]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the exchange water
shared.selection <- shared.t.ns[Identifiers]
# Generate matrix with both abundance and taxonomy info in order to be able to select the names of the top 10 genera
df.all <- cbind.data.frame(taxonomy.np.ns, shared.selection, rowSums(shared.selection))
colnames(df.all)[dim(df.all)[2]] <- "Summed"
Aggregated <- aggregate(df.all$Summed, by = list(Category = df.all[,"Genus"]), FUN = sum) # Aggregate at the genus level
Top <- Aggregated$Category[order(Aggregated$x, decreasing = TRUE)][1:16]
# All genera that do not belong to the top genera should be collapsed into 1 class called "Other"
df.all$Genus <- as.character(df.all$Genus)
df.all$Genus[!(df.all$Genus %in% Top)] <- "Other"
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(df.all[colnames(taxonomy.np.ns)]))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
# Plot
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                      psmelt()
Subsetphyseqobj$Genus <- as.factor(Subsetphyseqobj$Genus)
p_exchange <- Subsetphyseqobj %>%
                      dplyr::mutate(Sample = factor(Sample, levels = Identifiers)) %>% 
                      dplyr::mutate(Genus = factor(Genus, levels = c("Other", levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) %>% 
                      ggplot(data = ., aes(x = Day, y = 100*Abundance, fill = Genus)) + 
                      geom_bar(stat = "identity", colour = "black") + 
                      scale_fill_manual(values = c(ColorBlocksFacet, Series16), labels = c("Other", make.italic(levels(Subsetphyseqobj$Genus)[!levels(Subsetphyseqobj$Genus) == "Other"]))) +
                      scale_x_continuous(breaks = c(7,9,11,13,15,17)) +
                      theme(axis.title.x = element_blank()) + 
                      guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative abundance (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0)
print(p_exchange)
png("Figures/SOURCES-ExchangeWaters-Genus-Relative.png", width = 8, height = 4,  res = 300, units = "in")
print(p_exchange)
dev.off()

# Remove all variables that will not be used further
remove(Aggregated, df.all, shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Top, Info, SubsetINFO, Identifiers)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/CommunitiesSources_ExchangeWater-1.png" width="70%" style="display: block; margin: auto;" />


```r
Identifiers <- Metadata$Identifier[Metadata$Tank == "IncomingW"]
Info <- Metadata[Metadata$Identifier %in% Identifiers,]
Metadata[Metadata$Identifier %in% Identifiers,]
rownames(Info) <- Info$Identifier
# Get samples of the exchange water
shared.selection <- shared.t.ns[Identifiers]
# Subset to top instances
SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
SubsetINFO <- sample_data(Info)
rownames(SubsetINFO) <- SubsetINFO$Identifier
Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
Subsetphyseqobj <- Subsetphyseqobj %>%
                      transform_sample_counts(function(x) {x/sum(x)})

# Core community
core.taxa.standard <- core_members(x = Subsetphyseqobj, detection = 0, prevalence = 75/100, include.lowest = FALSE)
CoreComposition <- NULL
AbsoluteAbundances <- as.data.frame(sample_data(Subsetphyseqobj))[,c("Identifier", "BacterialDensity")]
for (i in core.taxa.standard){
  # Get the info of the selected OTU
  Abundances <- as.data.frame(otu_table(Subsetphyseqobj)[i,])
  AbsoluteAbundance <- Abundances*t(AbsoluteAbundances[colnames(Abundances), "BacterialDensity"])
  Classification <- as.data.frame(tax_table(Subsetphyseqobj)[i, c("Familia","Genus")])

  # Check how often this OTU is present and how frequently it's abundance is higher than 1 %
  Abundant <- sum(Abundances > 0.01)

  # Check the average ebundance when this OTU is present - relative abundances
  MeanAbundance <- rowMeans(Abundances)
  sdAbundance <- sd(Abundances)
  MinAbundance <- min(Abundances)
  MaxAbundance <- max(Abundances)

  # Check the average ebundance when this OTU is present - absolute abundances
  MeanAbsAbundance <- rowMeans(AbsoluteAbundance)
  sdAbsAbundance <- sd(AbsoluteAbundance)
  MinAbsAbundance <- min(AbsoluteAbundance)
  MaxAbsAbundance <- max(AbsoluteAbundance)

  # Save the results
  Result <- cbind.data.frame(i, Classification, Abundant, MeanAbundance, sdAbundance, MinAbundance, MaxAbundance, MeanAbsAbundance, sdAbsAbundance, MinAbsAbundance, MaxAbsAbundance)
  CoreComposition <- rbind(CoreComposition, Result)
}
# Write the results to an excel
write.csv(x = CoreComposition, file = "Results/DYNAMICS-CoreMembers-ExchangeWater.csv")

# Quantify batch differences
Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
meanBray <- mean(Bray)
sdBray <- sd(Bray)
Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
meanJaccard <- mean(Jaccard)
sdJaccard <- sd(Jaccard)
Bray.part <- bray.part(t(as.matrix(shared.selection)))
meanBray.turnover <- mean(Bray.part$bray.gra)
sdBray.turnover <- sd(Bray.part$bray.gra)
meanBray.abundancevariation <- mean(Bray.part$bray.bal)
sdBray.abundancevariation <- sd(Bray.part$bray.bal)

# Remove all variables that will not be used further
remove(shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Info, SubsetINFO, Identifiers, CoreComposition, Abundances, Classification, Bray, meanBray, sdBray, Jaccard, meanJaccard, sdJaccard, Bray.part, meanBray.turnover, sdBray.turnover, meanBray.abundancevariation, sdBray.abundancevariation, Result, sdAbundance, i, MinAbundance, MaxAbundance, MeanAbundance, Abundant)
```

### Combined plot sources


```r
# Assemble to one plot (separate the legends first and then recombine to have them aligned for all subplots)
# Get legends
Legend_algae <- cowplot::get_legend(p_algae)
Legend_artemia <- cowplot::get_legend(p_artemia)
Legend_dryfeed <- cowplot::get_legend(p_dryfeed)
Legend_exchange <- cowplot::get_legend(p_exchange)
# Remove legends from plot
p_algae <- p_algae + theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "none")
p_artemia <- p_artemia + theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "none")
p_dryfeed <- p_dryfeed + theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "none")
p_exchange <- p_exchange + theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "none")
# Combine plots and legends
g3 <- plot_grid(p_algae, Legend_algae, p_artemia, Legend_artemia, p_dryfeed, Legend_dryfeed, p_exchange, Legend_exchange, labels = c("A", "", "B", "", "C", "", "D", ""), ncol = 2, nrow = 4, rel_widths = c(1.5,0.65))
ggsave(file = "Figures/SOURCES-All-Genus-Relative.png", width = 10, height = 18, dpi = 300, units = "in", g3)

# Remove all variables that will not be used further
remove(Legend_algae, Legend_artemia, Legend_dryfeed, Legend_exchange, p_algae, p_artemia, p_dryfeed, p_exchange, g3)
```

### Resembence between sources and rearing water microbiomes 


```r
# Exchange water and rearing water
    IdentifiersTanks <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
    IdentifiersWE <- Metadata$Identifier[Metadata$Tank == "IncomingW"]
    Identifiers <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5", "IncomingW") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
    Info <- Metadata[Metadata$Identifier %in% Identifiers,]
    Metadata[Metadata$Identifier %in% Identifiers,]
    rownames(Info) <- Info$Identifier
    # Get samples of the sorting buffers
    shared.selection <- shared.t.ns[Identifiers]
    # Subset to top instances
    SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
    SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
    SubsetINFO <- sample_data(Info)
    rownames(SubsetINFO) <- SubsetINFO$Identifier
    Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
    # Plot
    Subsetphyseqobj <- Subsetphyseqobj %>%
                          transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                          psmelt()
    # Quantify batch differences
    Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
    Jaccard <- as.data.frame(as.matrix(Jaccard))
    Jaccard <- Jaccard[IdentifiersTanks, IdentifiersWE] 
    mean(as.matrix(Jaccard))
    sd(as.matrix(Jaccard))
    Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
    Bray <- as.data.frame(as.matrix(Bray))
    Bray <- Bray[IdentifiersTanks, IdentifiersWE] # Select only comparions between tanks and exchange water (and not within the tank-samples or within the exchange water samples)
    mean(as.matrix(Bray))
    sd(as.matrix(Bray))
    Bray.part <- bray.part(t(as.matrix(shared.selection)))
    Bray.turnover <- as.data.frame(as.matrix(Bray.part$bray.gra))
    Bray.turnover <- Bray.turnover[IdentifiersTanks, IdentifiersWE]
    meanBray.turnover <- mean(as.matrix(Bray.turnover))
    sdBray.turnover <- sd(as.matrix(Bray.turnover))
    Bray.abundancevariation <- as.data.frame(as.matrix(Bray.part$bray.bal))
    Bray.abundancevariation <- Bray.abundancevariation[IdentifiersTanks, IdentifiersWE]
    meanBray.abundancevariation <- mean(as.matrix(Bray.abundancevariation))
    sdBray.abundancevariation <- sd(as.matrix(Bray.abundancevariation))

    
    
# Artemia and rearing water
    IdentifiersTanks <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
    IdentifiersArtemia <- Metadata$Identifier[Metadata$Tank == "Art"]
    Identifiers <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5", "Art") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae"))]
    Info <- Metadata[Metadata$Identifier %in% Identifiers,]
    Metadata[Metadata$Identifier %in% Identifiers,]
    rownames(Info) <- Info$Identifier
    # Get samples of the sorting buffers
    shared.selection <- shared.t.ns[Identifiers]
    # Subset to top instances
    SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
    SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
    SubsetINFO <- sample_data(Info)
    rownames(SubsetINFO) <- SubsetINFO$Identifier
    Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
    # Plot
    Subsetphyseqobj <- Subsetphyseqobj %>%
                          transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                          psmelt()
    # Quantify batch differences
    Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
    Jaccard <- as.data.frame(as.matrix(Jaccard))
    Jaccard <- Jaccard[IdentifiersTanks, IdentifiersArtemia] 
    mean(as.matrix(Jaccard))
    sd(as.matrix(Jaccard))
    Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
    Bray <- as.data.frame(as.matrix(Bray))
    Bray <- Bray[IdentifiersTanks, IdentifiersArtemia]
    mean(as.matrix(Bray))
    sd(as.matrix(Bray))
    Bray.part <- bray.part(t(as.matrix(shared.selection)))
    Bray.turnover <- as.data.frame(as.matrix(Bray.part$bray.gra))
    Bray.turnover <- Bray.turnover[IdentifiersTanks, IdentifiersArtemia]
    meanBray.turnover <- mean(as.matrix(Bray.turnover))
    sdBray.turnover <- sd(as.matrix(Bray.turnover))
    Bray.abundancevariation <- as.data.frame(as.matrix(Bray.part$bray.bal))
    Bray.abundancevariation <- Bray.abundancevariation[IdentifiersTanks, IdentifiersArtemia]
    meanBray.abundancevariation <- mean(as.matrix(Bray.abundancevariation))
    sdBray.abundancevariation <- sd(as.matrix(Bray.abundancevariation))
    
# Algae and rearing water
    IdentifiersTanks <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
    IdentifiersAlgae <- Metadata$Identifier[Metadata$Tank == "Algae"]
    Identifiers <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5", "Algae") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Art"))]
    Info <- Metadata[Metadata$Identifier %in% Identifiers,]
    Metadata[Metadata$Identifier %in% Identifiers,]
    rownames(Info) <- Info$Identifier
    # Get samples of the sorting buffers
    shared.selection <- shared.t.ns[Identifiers]
    # Subset to top instances
    SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
    SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
    SubsetINFO <- sample_data(Info)
    rownames(SubsetINFO) <- SubsetINFO$Identifier
    Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
    # Plot
    Subsetphyseqobj <- Subsetphyseqobj %>%
                          transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                          psmelt()
    # Quantify batch differences
    Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
    Jaccard <- as.data.frame(as.matrix(Jaccard))
    Jaccard <- Jaccard[IdentifiersTanks, IdentifiersAlgae] 
    mean(as.matrix(Jaccard))
    sd(as.matrix(Jaccard))
    Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
    Bray <- as.data.frame(as.matrix(Bray))
    Bray <- Bray[IdentifiersTanks, IdentifiersAlgae] 
    mean(as.matrix(Bray))
    sd(as.matrix(Bray))
    Bray.part <- bray.part(t(as.matrix(shared.selection)))
    Bray.turnover <- as.data.frame(as.matrix(Bray.part$bray.gra))
    Bray.turnover <- Bray.turnover[IdentifiersTanks, IdentifiersAlgae]
    meanBray.turnover <- mean(as.matrix(Bray.turnover))
    sdBray.turnover <- sd(as.matrix(Bray.turnover))
    Bray.abundancevariation <- as.data.frame(as.matrix(Bray.part$bray.bal))
    Bray.abundancevariation <- Bray.abundancevariation[IdentifiersTanks, IdentifiersAlgae]
    meanBray.abundancevariation <- mean(as.matrix(Bray.abundancevariation))
    sdBray.abundancevariation <- sd(as.matrix(Bray.abundancevariation))

# Dry feed and rearing water
    IdentifiersTanks <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Algae", "Art"))]
    IdentifiersDryFeed <- Metadata$Identifier[Metadata$SampleIdentifier %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5")]
    Identifiers <- Metadata$Identifier[Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5", "Algae") & !(Metadata$Feeding_status %in% c("Sed", "Mix", "Art", "Algae")) | Metadata$SampleIdentifier %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5")]
    Info <- Metadata[Metadata$Identifier %in% Identifiers,]
    Metadata[Metadata$Identifier %in% Identifiers,]
    rownames(Info) <- Info$Identifier
    # Get samples of the sorting buffers
    shared.selection <- shared.t.ns[Identifiers]
    # Subset to top instances
    SubsetOTU <- otu_table(as.matrix(shared.selection), taxa_are_rows = TRUE)
    SubsetTAX <- tax_table(as.matrix(taxonomy.np.ns))
    SubsetINFO <- sample_data(Info)
    rownames(SubsetINFO) <- SubsetINFO$Identifier
    Subsetphyseqobj <- phyloseq(SubsetOTU, SubsetTAX, SubsetINFO)
    # Plot
    Subsetphyseqobj <- Subsetphyseqobj %>%
                          transform_sample_counts(function(x) {x/sum(x)}) %>% # Transform to relative abundances
                          psmelt()
    # Quantify batch differences
    Jaccard <- vegan::vegdist(t(SubsetOTU), method = "jaccard", binary = TRUE)
    Jaccard <- as.data.frame(as.matrix(Jaccard))
    Jaccard <- Jaccard[IdentifiersTanks, IdentifiersDryFeed] 
    mean(as.matrix(Jaccard))
    sd(as.matrix(Jaccard))
    Bray <- vegan::vegdist(t(SubsetOTU), method = "bray", binary = FALSE)
    Bray <- as.data.frame(as.matrix(Bray))
    Bray <- Bray[IdentifiersTanks, IdentifiersDryFeed] 
    mean(as.matrix(Bray))
    sd(as.matrix(Bray))
    Bray.part <- bray.part(t(as.matrix(shared.selection)))
    Bray.turnover <- as.data.frame(as.matrix(Bray.part$bray.gra))
    Bray.turnover <- Bray.turnover[IdentifiersTanks, IdentifiersDryFeed]
    meanBray.turnover <- mean(as.matrix(Bray.turnover))
    sdBray.turnover <- sd(as.matrix(Bray.turnover))
    Bray.abundancevariation <- as.data.frame(as.matrix(Bray.part$bray.bal))
    Bray.abundancevariation <- Bray.abundancevariation[IdentifiersTanks, IdentifiersDryFeed]
    meanBray.abundancevariation <- mean(as.matrix(Bray.abundancevariation))
    sdBray.abundancevariation <- sd(as.matrix(Bray.abundancevariation))

# Remove all variables that will not be used further
remove(shared.selection, Subsetphyseqobj, SubsetOTU, SubsetTAX, Info, SubsetINFO, Identifiers, Bray, Jaccard, Bray.part, meanBray.turnover, sdBray.turnover, meanBray.abundancevariation, sdBray.abundancevariation, Bray.abundancevariation, Bray.turnover, IdentifiersAlgae, IdentifiersArtemia, IdentifiersDryFeed, IdentifiersTanks, IdentifiersWE)
```

# Beta diversity


```r
# Make phyloseq object from the data without all the control samples (don't need to calculate their alpha diveristy)
IdentifierControlSamples <- c("250", "251", "252", "253", "254", "255", "Z")
otumat.ns <- as.matrix(shared.t.ns[Metadata$Identifier[!Metadata$Identifier %in% IdentifierControlSamples]])
taxmat.ns <- as.matrix(taxonomy.np.ns)
info <- Metadata[Metadata$Identifier %in% colnames(otumat.ns),]
OTU <- otu_table(otumat.ns, taxa_are_rows = TRUE)
TAX <- tax_table(taxmat.ns)
INFO <- sample_data(info)
rownames(INFO) <- INFO$Identifier
physeqobj <- phyloseq(OTU, TAX, INFO)
# Remove all variables that will not be used further
remove(OTU, TAX, INFO, info, taxmat.ns)
```
 
## Distance metrics


```r
# Only the samples of the entire communities are needed for this
InfoSources <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & !(Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5")),]
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)

# For every day
ResultsDistance <- NULL
ResultsDistanceTanks <- NULL
for(i in 1:18){
  # Get the identifiers for the day
  DayBefore <- i
  DayAfter <- i+1
  # Get the corresponding samples
  InfoSelectedSamples <- InfoTanks[InfoTanks$Day %in% c(DayBefore, DayAfter),]
  
  # For every tank store the distance metrics between the previous and the following day
  for (j in c("T1", "T2", "T3", "T4", "T5")){
    IdentifiersOfIntrest <- InfoSelectedSamples$Identifier[InfoSelectedSamples$Tank == j]
    if (length(IdentifiersOfIntrest) == 2){
        # Distance metrics
        Bray <- vegan::vegdist(t(shared.t.ns)[IdentifiersOfIntrest,], method = "bray", binary = FALSE)
        Jaccard <- vegan::vegdist(t(shared.t.ns)[IdentifiersOfIntrest,], method = "jaccard", binary = TRUE)
        
        # Distance metrics with partitioning
        Bray.part <- bray.part(t(shared.t.ns)[IdentifiersOfIntrest,])
        
        # Save the results
        Result <- cbind.data.frame(i, i+1, j, as.numeric(Bray), as.numeric(Jaccard), as.numeric(Bray.part$bray), as.numeric(Bray.part$bray.bal), as.numeric(Bray.part$bray.gra))
        ResultsDistance <- rbind(ResultsDistance, Result)
    }
  }

  
  # For every day get the average distance metrics between the tanks
  Bray <- vegan::vegdist(t(shared.t.ns)[InfoSelectedSamples$Identifier[InfoSelectedSamples$Day == i],], method = "bray", binary = FALSE)
  meanBray <- mean(Bray)
  sdBray <- sd(Bray)
  Jaccard <- vegan::vegdist(t(shared.t.ns)[InfoSelectedSamples$Identifier[InfoSelectedSamples$Day == i],], method = "jaccard", binary = TRUE)
  meanJaccard <- mean(Jaccard)
  sdJaccard <- sd(Jaccard)
  Bray.part <- bray.part(t(shared.t.ns)[InfoSelectedSamples$Identifier[InfoSelectedSamples$Day == i],])
  meanBrayTot <- mean(Bray.part$bray)
  sdBrayTot <- sd(Bray.part$bray)
  meanBrayBal <- mean(Bray.part$bray.bal)
  sdBrayBal <- sd(Bray.part$bray.bal)
  meanBrayGra <- mean(Bray.part$bray.gra)
  sdBrayGra <- sd(Bray.part$bray.gra)
  
  Result <- cbind.data.frame(i, meanBray, sdBray, meanJaccard, sdJaccard, meanBrayTot, sdBrayTot, meanBrayBal, sdBrayBal, meanBrayGra, sdBrayGra)
  ResultsDistanceTanks <- rbind(ResultsDistanceTanks, Result)
}
colnames(ResultsDistance) <- c("DayBefore", "DayAfter", "Tank", "BrayCurtis", "Jaccard", "BrayTotal", "BrayBal", "BrayGra")
colnames(ResultsDistanceTanks) <- c("Day", "meanBray", "sdBray", "meanJaccard", "sdJaccard", "meanBrayTot", "sdBrayTot", "meanBrayBal", "sdBrayBal", "meanBrayGra", "sdBrayGra")


# Plot Bray curtis of subsequent days with partitioning
tmp <- ResultsDistance[c("DayAfter", "Tank", "BrayBal", "BrayGra")]
tmp <- melt(tmp, id.vars = c("DayAfter", "Tank"))
p_Bray <- tmp %>%
              ggplot(data = ., aes(x = DayAfter, y = value, fill = variable)) +
              geom_bar(stat = "identity", color = "black") +
              scale_fill_manual(values = TankColors[3:4], labels = c("Abundance variation", "Turnover")) +
              facet_grid(Tank ~ .) +
              labs(fill = "                ", x = "Time (d)", y = "Bray-Curtis dissimilarity") +
              ylim(0, 1) +
              scale_x_continuous(minor_breaks = seq(1,18), limits = c(1,18.6)) +
              theme_cowplot() +
              theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom") 
print(p_Bray)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityMetrics-1.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/DYNAMICS-BrayCurtisPartitioning-PerDay.png", width = 6, height = 8, res = 500, units = "in")
print(p_Bray)
dev.off()
```

```
## png 
##   2
```

```r
# Plot Bray curtis of subsequent days with partitioning
tmpmean <- ResultsDistanceTanks[c("Day", "meanBrayBal", "meanBrayGra")]
tmpsd <- ResultsDistanceTanks[c("Day", "sdBrayBal", "sdBrayGra")]
tmpmean <- melt(tmpmean, id.vars = c("Day"))
tmpsd <- melt(tmpsd, id.vars = c("Day"))
p_Bray <- ggplot(data = tmpmean, aes(x = Day, y = value, fill = variable)) +
              geom_bar(stat = "identity", color = "black") +
              scale_fill_manual(values = TankColors[3:4], labels = c("Abundance variation", "Turnover")) + # labels = c("Abundance\nvariation", "Turnover\n ")
              labs(fill = "", x = "Time (d)", y = "Bray-Curtis dissimilarity") +
              guides(fill = guide_legend(title.position = "left", reverse = TRUE, keywidth = 1, keyheight = 1, nrow = 2)) +
              scale_y_continuous(limits = c(0, 1), expand = c(0, 0)) +
              scale_x_continuous(limits = c(0, 19), expand = c(0, 0)) +
              theme_cowplot() +
              theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom", legend.margin = margin(c(0,0,0,0)))
print(p_Bray)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityMetrics-2.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/DYNAMICS-BrayCurtisPartitioning-AveragesTank.png", width = 7, height = 4, res = 500, units = "in")
print(p_Bray)
dev.off()
```

```
## png 
##   2
```

```r
# Remove variables that are no longer needed
remove(InfoSources, InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4, i, j, DayBefore, DayAfter, IdentifiersOfIntrest, InfoSelectedSamples, Bray, Bray.part, Jaccard, meanBray, sdBray, meanJaccard, sdJaccard, meanBrayTot, sdBrayTot, meanBrayBal, sdBrayBal, meanBrayGra, sdBrayGra, Result, ResultsDistance, ResultsDistanceTanks, tmp, tmpmean, tmpsd)
```


```r
# For everyday: average diversity from one tank as compared to the others
ResultsDistanceIndividualTanks <- NULL
for (i in 1: 18){
  # Get the corresponding samples
  InfoSelectedSamples <- InfoTanks[InfoTanks$Day == i,]
  
  # For every tank store the distance metrics between this and all other tanks
  for (j in c("T1", "T2", "T3", "T4", "T5")){
      if (sum(InfoSelectedSamples$Tank == j) == 1){ # If we have this sample
      # Identifiers for all tanks
      IdentifierTankOfIntrest <- InfoSelectedSamples$Identifier[InfoSelectedSamples$Tank == j]
      IdentifiersOtherTanks <- InfoSelectedSamples$Identifier[!InfoSelectedSamples$Tank == j]
      
      # Calculate distance metrics
      Bray <- vegan::vegdist(t(shared.t.ns)[c(IdentifiersOtherTanks, IdentifierTankOfIntrest),], method = "bray", binary = FALSE)
      Bray <- as.data.frame(as.matrix(Bray))
      
      # The value for each pairwise combination with the tank of intrest will be the last row
      meanBray <- mean(Bray[IdentifierTankOfIntrest][-dim(Bray)[2],]) # Need to remove the combination of the tank with itself
      sdBray <- sd(Bray[IdentifierTankOfIntrest][-dim(Bray)[2],])

      # Save the results
      Result <- cbind.data.frame(i, j, meanBray, sdBray)
      ResultsDistanceIndividualTanks <- rbind(ResultsDistanceIndividualTanks, Result)
      }
  }
}
colnames(ResultsDistanceIndividualTanks) <- c("Day", "Tank", "meanBray", "sdBray")

# Plot
p_DistancesIndividualTanks <- ResultsDistanceIndividualTanks %>%
                               ggplot(data = ., aes(x = Day, y = meanBray)) +
                               geom_line(alpha = 1, aes(color = Tank)) +
                               geom_point(shape = 21, size = 3, alpha = 1, aes(fill = Tank)) +
                               scale_fill_manual(values = TankColors) +
                               scale_color_manual(values = TankColors) +
                               labs(color = "", x = "Time (d)", y = "Bray-Curtis dissimilarity") +
                               guides(color = FALSE, fill = guide_legend(keywidth = 1, keyheight = 1, nrow = 3, byrow = FALSE, title.position = "left")) +
                               scale_y_continuous(limits = c(0, 1), expand = c(0, 0)) +
                               scale_x_continuous(limits = c(0, 19), expand = c(0, 0)) +
                               theme_cowplot() +
                               theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom", legend.margin = margin(c(0,0,0,0)))
print(p_DistancesIndividualTanks)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BrayComparisonTanks-1.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/DYNAMICS-BrayCurtis-AveragesTank-PerTank.png", width = 7, height = 5, res = 500, units = "in")
print(p_DistancesIndividualTanks)
dev.off()
```

```
## png 
##   2
```

```r
# Remove variables that are no longer needed
remove(i, j, InfoSelectedSamples, Bray, meanBray, sdBray, Result, ResultsDistanceIndividualTanks, InfoTanks, IdentifierTankOfIntrest, IdentifiersOtherTanks)
```

## PCoA

### All samples 


```r
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoPeripheral <- Metadata[Metadata$Tank %in% c("Algae", "Art", "IncomingW" ) | Metadata$SampleIdentifier %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5"),]
InfoAll <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4, InfoPeripheral)
```


```r
IDsNotCrashed <- InfoAll$Identifier
# Get only the data from the rearingwater
otumat.ns <- as.matrix(shared.t.ns[IDsNotCrashed])
# Calculate Bray-Curtis dissimilarities and run a PCoA on all the samples
Bray <- vegan::vegdist(t(otumat.ns), method = "bray", binary = FALSE)
PCoARes <- stats::cmdscale(Bray, k = 2, eig = TRUE, add = TRUE)
var <- base::format(round(vegan::eigenvals(PCoARes)/sum(vegan::eigenvals(PCoARes))*100,1), nsmall = 1)
# Add metadata for plotting
PCoARes <- data.frame(PCoARes$points)
PCoARes$Identifier <- rownames(PCoARes)
PCoARes <- dplyr::left_join(PCoARes, Metadata, by = c("Identifier"))
# Add day-variables for the feeds (day is the first day this dry feed product was aded to the water)
PCoARes$Tank <- as.character(PCoARes$Tank)
PCoARes$Tank[PCoARes$Tank %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5")] <- "Feed"
PCoARes$Day[PCoARes$Tank == "Feed"] <- c(2,5,7,12,15)

# Plot the ordination
p_beta_all <- PCoARes %>%
                 dplyr::mutate(Tank = factor(Tank, levels = c("T1", "T2", "T3", "T4", "T5", "Algae", "Art", "IncomingW", "Feed"))) %>% 
                 ggplot(data = ., aes(x = X1, y = X2, fill = Tank)) +
                 geom_point(alpha = 1, size = 3 , shape = 21, color = "black") +
                 scale_fill_manual(values = AllTankAndSourcesColors, labels = c("Tank 1", "Tank 2", "Tank 3", "Tank 4", "Tank 5", "Algae", "Artemia", "Exchange water", "Feed")) +
                 guides(fill = guide_legend(override.aes = list(shape = 21, size = 4))) +
                 labs(x = paste0("PCoA axis 1 (",var[1], "%)"), y = paste0("PCoA axis 2 (",var[2], "%)")) +
                 coord_fixed() +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_beta_all)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityAll-1.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/DYNAMICS-All-Beta.png", width = 7, height = 5, res = 500, units = "in")
print(p_beta_all)
dev.off()
```

```
## png 
##   2
```

```r
# Treat the rearing water as 1 group
PCoARes$Tank[PCoARes$Tank %in% c("T1","T2","T3","T4","T5")] <- "Tank"
p_beta_all <- PCoARes %>%
                 dplyr::mutate(Tank = factor(Tank, levels = c("Tank", "Algae", "Art", "IncomingW", "Feed"))) %>% 
                 ggplot(data = ., aes(x = X1, y = X2, fill = Tank)) +
                 geom_point(alpha = 1, size = 3 , shape = 21, color = "black") +
                 scale_fill_manual(values = c(SingleColor, SourceColors), labels = c("Rearing water", "Algae", "Artemia", "Exchange water", "Feed")) +
                 guides(fill = guide_legend(override.aes = list(shape = 21, size = 4))) +
                 labs(x = paste0("PCoA axis 1 (",var[1], "%)"), y = paste0("PCoA axis 2 (",var[2], "%)"), fill = "") +
                 scale_y_continuous(limits = c(-0.6,0.4), breaks = c(-0.6,-0.4,-0.2,0,0.2,0.4)) +
                 scale_x_continuous(limits = c(-0.6,0.4), breaks = c(-0.6,-0.4,-0.2,0,0.2,0.4)) +
                 coord_fixed() +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_beta_all)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityAll-2.png" width="70%" style="display: block; margin: auto;" />

```r
# PERMANOVA
  # Checking the homogeneity condition
  dist <- vegdist(t(otumat.ns))
  anova(betadisper(dist, PCoARes$Tank)) # Significant difference detected, so it is not possible to run PERMANOVA
```

```
## Analysis of Variance Table
## 
## Response: Distances
##            Df  Sum Sq  Mean Sq F value   Pr(>F)   
## Groups      4 0.16468 0.041170  3.6987 0.007511 **
## Residuals 100 1.11310 0.011131                    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
# Remove all variables that will not be used further
remove(Bray, PCoARes, var, dist)
```

### The rearing water


```r
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)
```


```r
RearingWaterIDs <- InfoTanks$Identifier
# Get only the data from the rearingwater
otumat.ns <- as.matrix(shared.t.ns[RearingWaterIDs])
# Calculate Bray-Curtis dissimilarities and run a PCoA on all the samples
Bray <- vegan::vegdist(t(otumat.ns), method = "bray", binary = FALSE)
PCoARes <- stats::cmdscale(Bray, k = 2, eig = TRUE, add = TRUE)
var <- base::format(round(vegan::eigenvals(PCoARes)/sum(vegan::eigenvals(PCoARes))*100,1), nsmall = 1)
# Add metadata for plotting
PCoARes <- data.frame(PCoARes$points)
PCoARes$Identifier <- rownames(PCoARes)
PCoARes <- dplyr::left_join(PCoARes, Metadata, by = c("Identifier"))


# Plot the ordination
p_beta_water <- PCoARes %>%
                 dplyr::mutate(Tank = factor(Tank, levels = c("T1", "T2", "T3", "T4", "T5"))) %>% 
                 ggplot(data = ., aes(x = X1, y = X2)) +
                 geom_point(alpha = 1, shape = 21, aes(fill = Tank, size = Day)) +
                 scale_fill_manual(values = TankColors) +
                 guides(fill = FALSE, size = guide_legend(keywidth = 1, keyheight = 1, nrow = 3)) +
                 labs(x = paste0("PCoA axis 1 (",var[1], "%)"), y = paste0("PCoA axis 2 (",var[2], "%)")) +
                 coord_fixed() +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom", legend.margin = margin(c(0,0,0,0)))
print(p_beta_water)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityRearingWater-1.png" width="70%" style="display: block; margin: auto;" />

```r
png("Figures/DYNAMICS-Tanks-Beta.png", width = 7, height = 5, res = 500, units = "in")
print(p_beta_water)
dev.off()
```

```
## png 
##   2
```

```r
# PERMANOVA
  # Checking the homogeneity condition
  dist <- vegdist(t(otumat.ns))
  anova(betadisper(dist, PCoARes$Tank))
```

```
## Analysis of Variance Table
## 
## Response: Distances
##           Df  Sum Sq  Mean Sq F value   Pr(>F)   
## Groups     4 0.16583 0.041456  3.9629 0.005935 **
## Residuals 69 0.72182 0.010461                    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
  anova(betadisper(dist, PCoARes$Day))
```

```
## Analysis of Variance Table
## 
## Response: Distances
##           Df Sum Sq  Mean Sq F value    Pr(>F)    
## Groups    17 1.3223 0.077783  3.6995 0.0001093 ***
## Residuals 56 1.1774 0.021025                      
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
  # Run permanova
  permanova <- vegan::adonis(t(otumat.ns) ~ PCoARes$Day * PCoARes$Tank , permutations = 999, method = "bray")
  permanova
```

```
## 
## Call:
## vegan::adonis(formula = t(otumat.ns) ~ PCoARes$Day * PCoARes$Tank,      permutations = 999, method = "bray") 
## 
## Permutation: free
## Number of permutations: 999
## 
## Terms added sequentially (first to last)
## 
##                          Df SumsOfSqs MeanSqs F.Model      R2 Pr(>F)    
## PCoARes$Day               1    4.0935  4.0935 16.0388 0.15808  0.001 ***
## PCoARes$Tank              4    2.5880  0.6470  2.5350 0.09994  0.001 ***
## PCoARes$Day:PCoARes$Tank  4    2.8797  0.7199  2.8208 0.11121  0.001 ***
## Residuals                64   16.3343  0.2552         0.63078           
## Total                    73   25.8955                 1.00000           
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
  # P-values
  print(as.data.frame(permanova$aov.tab)["PCoARes$Day", "Pr(>F)"])
```

```
## [1] 0.001
```

```r
  print(as.data.frame(permanova$aov.tab)["PCoARes$Tank", "Pr(>F)"])
```

```
## [1] 0.001
```

```r
# Remove all variables that will not be used further
remove(Bray, PCoARes, var, RearingWaterIDs, InfoTanks, InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4, permanova, dist)
```

### The pheripheral microbiomes


```r
SourceIDs <- Metadata$Identifier[(Metadata$Tank == "IncomingW") | (Metadata$Feeding_status %in% c("Algae", "Art", "Feed1", "Feed2", "Feed3", "Feed4", "Feed5"))]
# Get only the data from the rearingwater
otumat.ns <- as.matrix(shared.t.ns[SourceIDs])
# Calculate Bray-Curtis dissimilarities and run a PCoA on all the samples
Bray <- vegan::vegdist(t(otumat.ns), method = "bray", binary = FALSE)
PCoARes <- stats::cmdscale(Bray, k = 2, eig = TRUE, add = TRUE)
var <- base::format(round(vegan::eigenvals(PCoARes)/sum(vegan::eigenvals(PCoARes))*100,1), nsmall = 1)
# Add metadata for plotting
PCoARes <- data.frame(PCoARes$points)
PCoARes$Identifier <- rownames(PCoARes)
PCoARes <- dplyr::left_join(PCoARes, Metadata, by = c("Identifier"))
# Change the feed labels for plotting
PCoARes$Tank <- as.character(PCoARes$Tank)
PCoARes$Tank[PCoARes$Tank %in% c("Feed1", "Feed2", "Feed3", "Feed4", "Feed5")] <- "Dry feed"
PCoARes$Tank[PCoARes$Tank == "IncomingW"] <- "Exchange water"
PCoARes$Tank[PCoARes$Tank == "Art"] <- "Artemia"

# Plot the ordination
p_beta <- PCoARes %>%
                 dplyr::mutate(Tank = factor(Tank, levels = c("Algae", "Artemia", "Exchange water", "Dry feed"))) %>% 
                 ggplot(data = ., aes(x = X1, y = X2)) +
                 geom_point(alpha = 1, size = 3, shape = 21, aes(fill = Tank)) +
                 scale_fill_manual(values = SourceColors) +
                 guides(fill = guide_legend(override.aes = list(size = 4))) +
                 labs(x = paste0("PCoA axis 1 (",var[1], "%)"), y = paste0("PCoA axis 2 (",var[2], "%)"), fill = "") +
                 scale_y_continuous(limits = c(-0.4,0.6), breaks = c(-0.4,-0.2,0,0.2,0.4,0.6)) +
                 scale_x_continuous(limits = c(-0.5,0.5), breaks = c(-0.4,-0.2,0,0.2,0.4)) +
                 coord_fixed(ratio = 1) +
                 theme_cowplot() +
                 theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_beta)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/BetaDiversityPheripheralMicrobiomes-1.png" width="70%" style="display: block; margin: auto;" />

```r
# Make a combined plot
g1 <- plot_grid(p_beta_all, p_beta, labels=c("A", "B"), ncol = 1, nrow = 2, scale = 0.95)
ggsave(file = "Figures/SOURCES-Sources-Beta-Combined.png", width = 7, height = 9, dpi = 500, units = "in", g1)

# PERMANOVA
  # Checking the homogeneity condition
  dist <- vegdist(t(otumat.ns))
  anova(betadisper(dist, PCoARes$Tank))
```

```
## Analysis of Variance Table
## 
## Response: Distances
##           Df  Sum Sq   Mean Sq F value Pr(>F)
## Groups     3 0.00606 0.0020208  0.1282 0.9426
## Residuals 27 0.42570 0.0157665
```

```r
  # Run permanova
  permanova <- vegan::adonis(t(otumat.ns) ~ PCoARes$Tank, permutations = 999, method = "bray")
  permanova
```

```
## 
## Call:
## vegan::adonis(formula = t(otumat.ns) ~ PCoARes$Tank, permutations = 999,      method = "bray") 
## 
## Permutation: free
## Number of permutations: 999
## 
## Terms added sequentially (first to last)
## 
##              Df SumsOfSqs MeanSqs F.Model     R2 Pr(>F)    
## PCoARes$Tank  3    4.1993 1.39977  4.6903 0.3426  0.001 ***
## Residuals    27    8.0578 0.29844         0.6574           
## Total        30   12.2571                 1.0000           
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

```r
# Remove all variables that will not be used further
remove(Bray, PCoARes, SourceIDs, var, p_beta, permanova, dist, otumat.ns, g1, p_beta_all)
```

## Assembled plot


```r
# Get legends
LegendA <- cowplot::get_legend(p_beta_water)
LegendB <- cowplot::get_legend(p_DistancesIndividualTanks)
LegendC <- cowplot::get_legend(p_Bray)
# Remove legends
p_beta_water <- p_beta_water + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
p_DistancesIndividualTanks <- p_DistancesIndividualTanks + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
p_Bray <- p_Bray + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
# Assemble
g1 <- plot_grid(p_beta_water, p_DistancesIndividualTanks, p_Bray, rel_widths = c(1,1,1), labels = c("A","B","C"), ncol = 3, nrow = 1, scale = 1)

g2 <- plot_grid(NULL, LegendA, LegendB, NULL, LegendC, rel_widths = c(0.5,1.2,1,0.2,1), ncol = 5, nrow = 1, scale = 1)
g3 <- plot_grid(g1, g2, p_Bray, rel_heights = c(2.7,1), ncol = 1, nrow = 2, scale = 1)

# Save
ggsave(file = "FiguresPublication/DYNAMICS-AssembledBeta_300.tiff", width = 12, height = 5,  dpi = 300, units = "in", plot = g3, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-AssembledBeta_500.tiff", width = 12, height = 5,  dpi = 500, units = "in", plot = g3, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-AssembledBeta_700.tiff", width = 12, height = 5,  dpi = 700, units = "in", plot = g3, compression = "lzw")

# Remove variables that will no longer be used
remove(LegendA, LegendB, LegendC, p_beta_water, p_DistancesIndividualTanks, p_Bray, g1, g2, g3)
```

# Source tracking

## Based on absolute OTU abundances


```r
# Only the samples of the entire communities are needed for this
InfoSources <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & !(Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5")),]
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)
# Read the excel file with the info regarding the timing of the source introduction
SourceInfo <- read.xlsx(xlsxFile = "Metadata/MetadataSourceTracking.xlsx")

SourceTrackingResults <- NULL
AbsoluteLoadSources <- NULL
# For every day
for(i in 1:17){
  # Get the identifiers for the day
  DayBefore <- i
  DayAfter <- i+1
  # Get the corresponding samples
  InfoSelectedSamples <- InfoTanks[InfoTanks$Day %in% c(DayBefore, DayAfter),]
  # Make "Day" a factor with a specific order of the levels
  InfoSelectedSamples$Day <- factor(InfoSelectedSamples$Day, levels = c(DayBefore, DayAfter))
  # Get the corresponding samples
  InfoSelectedSamples <- InfoTanks[InfoTanks$Day %in% c(DayBefore, DayAfter),]
  OtuSelectedSamples <- cbind(sweep(shared.t.ns[InfoSelectedSamples$Identifier], 2, colSums(shared.t.ns[InfoSelectedSamples$Identifier]), `/`))
  # Get the absolute bacterial count of the OTU's in the tanks (175 L)
  OtuSelectedSamples <- OtuSelectedSamples*InfoSelectedSamples$BacterialDensity*1000*175

  # For every tank, calculate the absolute increase
  AbundanceDifferences <- data.frame(matrix(, nrow = dim(OtuSelectedSamples)[1], ncol = 0))
  for (Tank in c("T1","T2","T3","T4","T5")){
    SampleDayBefore <- InfoSelectedSamples$Identifier[InfoSelectedSamples$Tank == Tank & InfoSelectedSamples$Day == DayBefore]
    SampleDayAfter <- InfoSelectedSamples$Identifier[InfoSelectedSamples$Tank == Tank & InfoSelectedSamples$Day == DayAfter]
    if (sum(!is.na(c(SampleDayBefore,SampleDayAfter))) == 2) {
      AbundanceDifference <- OtuSelectedSamples[SampleDayBefore] - OtuSelectedSamples[SampleDayAfter]
      colnames(AbundanceDifference) <- Tank
      AbundanceDifferences <- cbind.data.frame(AbundanceDifferences, AbundanceDifference)
    }
  }
  
  # Add this to the results
  AverageDayBefore <- rowMeans(as.matrix(OtuSelectedSamples[InfoSelectedSamples$Identifier[InfoSelectedSamples$Day == DayBefore]]))
  AverageDayAfter <- rowMeans(as.matrix(OtuSelectedSamples[InfoSelectedSamples$Identifier[InfoSelectedSamples$Day == DayAfter]]))
  TankAverages <- cbind.data.frame(rownames(OtuSelectedSamples), AverageDayBefore, AverageDayAfter)
  colnames(TankAverages)[1] <- "OTU"

  # Get the OTU's for which an increase was observed
  OTUsRetained <- rownames(AbundanceDifferences)[rowMeans(as.matrix(AbundanceDifferences)) < 0 & rowSums(AbundanceDifferences < 0) >= floor(ncol(AbundanceDifferences)/2)]

  # Get the sources that may have influenced the microbiome on this day
  RelevantSources <- SourceInfo[SourceInfo$DayBefore == i,]
  # For every source
  for (j in 1:dim(RelevantSources)[1]){
    # Check which OTUs are present in the source sample(s) of this day
    SourceSampleIdentifier <- RelevantSources$SampleIdentifierSource[j]
    RelevantSourceOTUs <- cbind.data.frame(rownames(shared.t.ns), shared.t.ns[as.character(SourceSampleIdentifier)]/sum(shared.t.ns[as.character(SourceSampleIdentifier)]))
    colnames(RelevantSourceOTUs) <- c("OTU", "RelAb")
    RelevantSourceOTUs <- RelevantSourceOTUs[RelevantSourceOTUs$RelAb > 0,] # drop the OTUs that are absent from the source
    
    # Check which of the OTU's in the source were increased in absolute abundance in the tanks
    RelevantSourceOTUs <- RelevantSourceOTUs[RelevantSourceOTUs$OTU %in% OTUsRetained,]

    # Calculate for each source the absolute amount of cells that was added from a source Algae = L; Artemia = mL; Dry feed = g; Water exchange = %
    SourceType <- RelevantSources$Type[j]
    Amount <- RelevantSources$Amount_g_perc_or_mL[j]
    if (SourceType == "Algae"){ # Added amount is expressed in L, bacterial density cells/mL
        AbsoluteAmountAdded <- Amount*(Metadata$BacterialDensity[Metadata$Identifier == SourceSampleIdentifier]*1000)
    } else if (SourceType == "Artemia"){ # Added amount is expressed in mL
        AbsoluteAmountAdded <- Amount*(Metadata$BacterialDensity[Metadata$Identifier == SourceSampleIdentifier])
    } else if (SourceType == "ExchangeWater"){ # Added amount is expressed as a percentage of the total tank volume (approx. 175l)
        AbsoluteAmountAdded <- 175*(Amount/100)*Metadata$BacterialDensity[Metadata$Identifier == SourceSampleIdentifier]
    } else if (SourceType == "Dry feed"){ # Added amount is expressed as number of grams
        AbsoluteAmountAdded <- Amount*(Metadata$BacterialDensity[Metadata$Identifier == SourceSampleIdentifier])
    }
    # Multiply the absolute amount of bacteria added to the relative abundance of the OTU to get the abolute amount of bacteria for this specific OTU
    RelevantSourceOTUs$CellCount <- RelevantSourceOTUs$RelAb*AbsoluteAmountAdded
    
    # Check wether the amount that is added is bigger than the amount that was already present on day i
    Result <- cbind.data.frame(TankAverages[TankAverages$OTU %in% RelevantSourceOTUs$OTU,], RelevantSourceOTUs)
    Result$RatioSourceBefore <- Result$CellCount/Result$AverageDayBefore
    
    # Calculate the foldchange from day i to day i+1 (if there were none present on the day before, logfold as compared to what is added is calculated)
    Result$AverageDayBefore[Result$AverageDayBefore == 0] <- Result$CellCount[Result$AverageDayBefore == 0]
    Result$FoldChange <- Result$AverageDayAfter/Result$AverageDayBefore
    
    # Only consider the OTU's that were added more though the source as compared to what was already there
    Result <- Result[(Result$RatioSourceBefore > 1),]
    Result$NewlyIntroduced <- Result$RatioSourceBefore == Inf # Make an additional label that shows wether the source was newly introduced or not
    OTUSRetained <- Result[c("OTU", "RatioSourceBefore", "RelAb", "NewlyIntroduced", "FoldChange")]
    
    # Make a results-table for all OTUs that are not retained (easier for plotting later if all samples are present)
    OTUSNotRetained <- as.data.frame(rownames(OtuSelectedSamples)[!(rownames(OtuSelectedSamples) %in% OTUSRetained$OTU)])
    OTUSNotRetained$RatioSourceBefore <- 0
    OTUSNotRetained$RelRelAb <- 0
    OTUSNotRetained$NewlyIntroduced <- FALSE
    OTUSNotRetained$FoldChange <- 0
    colnames(OTUSNotRetained) <- c("OTU", "RatioSourceBefore", "RelAb", "NewlyIntroduced", "FoldChange")
    
    # Save the results
    Result <- rbind(OTUSRetained, OTUSNotRetained)
    Result$Day <- i
    Result$SourceType <- RelevantSources$Type[j]
    Result$Source <- RelevantSources$Source[j]
    SourceTrackingResults <- rbind(SourceTrackingResults, Result)
    
    # Save the absolute cell counts that are added to the tanks
    ResultLoads <- cbind.data.frame(i, RelevantSources$Type[j], RelevantSources$Source[j], AbsoluteAmountAdded)
    AbsoluteLoadSources <- rbind(AbsoluteLoadSources, ResultLoads)
  }
}
colnames(AbsoluteLoadSources) <- c("Day", "Type", "Source", "AbsoluteAmountAdded")
SourceTrackingResults <- SourceTrackingResults[!is.na(SourceTrackingResults$OTU),]

# Remove variables that are no longer needed
remove(AbsoluteAmountAdded, Amount, AverageDayAfter, AverageDayBefore, DayAfter, DayBefore, i, j, SampleDayAfter, SampleDayBefore, SourceSampleIdentifier, SourceType, Tank, AbundanceDifference, AbundanceDifferences, InfoSelectedSamples, InfoSources, InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4, OtuSelectedSamples, OTUSRetained, OTUSNotRetained, TankAverages, RelevantSourceOTUs, RelevantSources, Result, ResultLoads, OTUsRetained)
```


```r
# Keep only the OTU's that were increased due to the presence of a source at least once
RelevantOTUs <- unique(SourceTrackingResults$OTU[SourceTrackingResults$RatioSourceBefore > 1])

# Get the sourcetracking results for these OTU's alone
SourceTrackingResultsOTUs <- SourceTrackingResults %>% dplyr::filter(OTU %in% RelevantOTUs)

# Add taxonomic information so this can be used in the plot
tmp <- taxonomy.np.ns[SourceTrackingResultsOTUs$OTU,]["Genus"]
SourceTrackingResultsOTUs$Genus <- tmp$Genus

# Re-order the levels to have a prettier plot
SourceTrackingResultsOTUs$Source <- factor(SourceTrackingResultsOTUs$Source, levels = c("AlgaeDay1", "AlgaeDay2", "AlgaeDay3", "AlgaeDay4", "AlgaeDay6", "AlgaeDay9", "AlgaeDay10", "ArtemiaDay4", "ArtemiaDay5", "ArtemiaDay6", "ArtemiaDay7", "ArtemiaDay8", "ArtemiaDay9", "ArtemiaDay10", "ArtemiaDay11", "ArtemiaDay12", "ArtemiaDay13", "ArtemiaDay14", "ArtemiaDay15", "ArtemiaDay16", "ArtemiaDay17", "ArtemiaDay18", "DryFeed1.D2", "DryFeed1.D3", "DryFeed1.D4", "DryFeed1.D5", "DryFeed2.D5", "DryFeed2.D6", "DryFeed2.D7", "DryFeed3.D7", "DryFeed3.D8", "DryFeed3.D9", "DryFeed3.D10", "DryFeed3.D11", "DryFeed3.D12", "DryFeed4.D12", "DryFeed4.D13", "DryFeed4.D14", "DryFeed4.D15", "DryFeed5.D15", "DryFeed5.D16", "DryFeed5.D17", "WaterExchangeDay7", "WaterExchangeDay9", "WaterExchangeDay11", "WaterExchangeDay13", "WaterExchangeDay15", "WaterExchangeDay17"))

# Drop levels of the unused OTUs
SourceTrackingResultsOTUs$OTU <- as.factor(SourceTrackingResultsOTUs$OTU)
SourceTrackingResultsOTUs$OTU <- droplevels(x = SourceTrackingResultsOTUs$OTU)

# Make nicer x-axis labels
XLabels <- c("AlgaeDay1" = "Algae day 1", "AlgaeDay2" = "Algae day 2", "AlgaeDay3" = "Algae day 3", "AlgaeDay4" = "Algae day 4", "AlgaeDay6" = "Algae day 6", "AlgaeDay9" = "Algae day 9", "AlgaeDay10" = "Algae day 10", "ArtemiaDay5" = "Artemia day 4", "ArtemiaDay7" = "Artemia day 6", "ArtemiaDay8" = "Artemia day 7", "ArtemiaDay9" = "Artemia day 8", "ArtemiaDay10" = "Artemia day 9", "ArtemiaDay11" = "Artemia day 10", "ArtemiaDay12" = "Artemia day 11", "ArtemiaDay13" = "Artemia day 12", "ArtemiaDay14" = "Artemia day 13", "ArtemiaDay15" = "Artemia day 14", "ArtemiaDay16" = "Artemia day 15", "ArtemiaDay17" = "Artemia day 16", "ArtemiaDay18" = "Artemia day 17", "DryFeed1.D2" = "Dry feed 1 day 2", "DryFeed1.D3" = "Dry feed 1 day 3", "DryFeed1.D4" = "Dry feed 1 day 4", "DryFeed1.D5" = "Dry feed 1 day 5", "DryFeed2.D5" = "Dry feed 2 day 5", "DryFeed2.D6" = "Dry feed 2 day 6", "DryFeed2.D7" = "Dry feed 2 day 7", "DryFeed3.D7" = "Dry feed 3 day 7", "DryFeed3.D8" = "Dry feed 3 day 8", "DryFeed3.D9" = "Dry feed 3 day 9", "DryFeed3.D10" = "Dry feed 3 day 10", "DryFeed3.D11" = "Dry feed 3 day 11", "DryFeed3.D12" = "Dry feed 3 day 12", "DryFeed4.D12" = "Dry feed 4 day 12", "DryFeed4.D13" = "Dry feed 4 day 13", "DryFeed4.D14" = "Dry feed 4 day 14", "DryFeed4.D15" = "Dry feed 4 day 15", "DryFeed5.D15" = "Dry feed 5 day 15","DryFeed5.D16" = "Dry feed 5 day 16", "DryFeed5.D17" = "Dry feed 5 day 17","WaterExchangeDay7" = "Exchange water day 7", "WaterExchangeDay9" = "Exchange water day 9", "WaterExchangeDay11" = "Exchange water day 11", "WaterExchangeDay15" = "Exchange water day 15", "WaterExchangeDay17" = "Exchange water day 17")

# Plot
p_sourcetracking <- ggplot() +
      geom_count(data = SourceTrackingResultsOTUs, aes(y = paste(OTU, " (", Genus, ")", sep = ""), x = Source , size = log2(FoldChange), fill = SourceType, shape = NewlyIntroduced), alpha = 1) + 
      scale_fill_manual(values = c(SourceColors[1], SourceColors[2], SourceColors[4], SourceColors[3]), labels = c("Algae", "Artemia", "Dry feed", "Exchange water")) +
      scale_shape_manual(values = c(21, 23), labels = c("No", "Yes")) +
      scale_size_continuous(range = c(-1, 6), breaks = c(1,5,10,15)) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 0.5, xmax = 7.5), fill = "#5a7e48", alpha = 0.3) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 7.5, xmax = 20.5), fill = "#d97ea8", alpha = 0.3) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 20.5, xmax = 24.5), fill = "#d49f37", alpha = 0.5) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 24.5, xmax = 27.5), fill = "#d49f37", alpha = 0.4) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 27.5, xmax = 33.5), fill = "#d49f37", alpha = 0.3) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 33.5, xmax = 37.5), fill = "#d49f37", alpha = 0.2) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 37.5, xmax = 40.5), fill = "#d49f37", alpha = 0.1) +
      geom_rect(aes(ymin = 0, ymax = length(unique(SourceTrackingResultsOTUs$OTU)) + 0.5 , xmin = 40.5, xmax = 46), fill = "#88b2b8", alpha = 0.3) +
      labs(y = "", x = "", fill = "Source", size = "log2(fold change)", shape = "Newly introduced") +
      guides(fill = guide_legend(override.aes = list(size = 5, shape = 21)), shape = guide_legend(override.aes = list(size = 5, shape = c(21, 23), colour = "black", fill = c("white")))) +
      scale_x_discrete(labels = XLabels) +
      theme_cowplot() +
      theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.text.x = element_text(angle = 90, hjust = 1, vjust = 0.5))
print(p_sourcetracking)
png("Figures/SOURCES-AbsAb-AllOTUsFromSources.png", width = 14, height = 10,  res = 300, units = "in")
print(p_sourcetracking)
dev.off()

# Remove variables that are no longer needed
remove(SourceTrackingResults, tmp, RelevantOTUs, XLabels)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/SourceTrackingPlot-1.png" width="95%" style="display: block; margin: auto;" />


```r
SourceTrackingResultsOTUs <- SourceTrackingResultsOTUs[SourceTrackingResultsOTUs$FoldChange > 0,]
TotalEvents <- dim(SourceTrackingResultsOTUs)[1]
TotalOTUs <- length(unique(SourceTrackingResultsOTUs$OTU))

# OTU's associated with each of the sources
AlgalOTUs <- length(unique(SourceTrackingResultsOTUs$OTU[SourceTrackingResultsOTUs$SourceType == "Algae"]))
ArtemiaOTUs <- length(unique(SourceTrackingResultsOTUs$OTU[SourceTrackingResultsOTUs$SourceType == "Artemia"]))
ExchangeWaterOTUs <- length(unique(SourceTrackingResultsOTUs$OTU[SourceTrackingResultsOTUs$SourceType == "ExchangeWater"]))
DryFeedOTUs <- length(unique(SourceTrackingResultsOTUs$OTU[SourceTrackingResultsOTUs$SourceType == "Dry feed"]))

# Remove unused variables
remove(TotalEvents, TotalOTUs, AlgalOTUs, ArtemiaOTUs, ExchangeWaterOTUs, DryFeedOTUs)
```


```r
# Get only OTU's that were introduced
RelevantOTUs <- SourceTrackingResultsOTUs[SourceTrackingResultsOTUs$RatioSourceBefore > 0,]
# Plot
p_AbsoluteContributionSources <- RelevantOTUs %>%
                                  dplyr::mutate(SourceType = factor(SourceType, levels = c("Algae", "Artemia", "ExchangeWater", "Dry feed"))) %>%
                                  ggplot(data = ., aes(x = 100*RelAb, y = paste(OTU, " (", Genus, ")", sep = ""), fill = SourceType)) + 
                                  geom_point(shape = 21, size = 3) + 
                                  labs(color = "", y = "", x =  expression(atop("Relative abundance in ", paste("the source (%)")))) +
                                  scale_fill_manual(values = SourceColors) +
                                  scale_x_continuous(minor_breaks = seq(0,40,5)) +
                                  guides(fill = FALSE) +
                                  theme_cowplot() +
                                  theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AbsoluteContributionSources)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/ContributionsSources-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Make phyloseq object from the data without all the control samples (this will make the calculations a little bit faster)
otumat.ns <- as.matrix(shared.t.ns[Metadata$Identifier[!Metadata$Identifier %in% IdentifierControlSamples]])
taxmat.ns <- as.matrix(taxonomy.np.ns)
info <- Metadata[Metadata$Identifier %in% colnames(otumat.ns),]
rownames(info) <- info$Identifier
OTU <- otu_table(otumat.ns, taxa_are_rows = TRUE)
TAX <- tax_table(taxmat.ns)
INFO <- sample_data(info)
physeqobj <- phyloseq(OTU, TAX, INFO)
physeqobj <- physeqobj %>%
                transform_sample_counts(function(x) {x/sum(x)}) %>%
                psmelt()
physeqobj$AbsoluteAbundance <- physeqobj$Abundance*physeqobj$BacterialDensity

# Check whether the introduced sources reach high abundances in the tanks?
ResultsNumberOfDaysPresent <- NULL
ResultsAbundanceAfterIntro <- NULL
for (i in 1:dim(RelevantOTUs)[1]){
  Selected <- RelevantOTUs[i,]
  
  # Get relative abundances of this OTU in the rearingwater after the introduction (i.e. from day i+1 on)
  Subset <- physeqobj %>% dplyr::filter(OTU == Selected$OTU  & Tank %in% c("T1", "T2", "T3", "T4", "T5") & Feeding_status == "B" & Day > Selected$Day)
  Subset <- Subset[c("OTU", "Abundance", "Tank", "Day", "Genus", "AbsoluteAbundance")]
  for (Tank in c("T1", "T2", "T3", "T4", "T5")){
    SubsetTank <-  Subset[Subset$Tank == Tank,]
    # Sort the dataframe according to day
    SubsetTank <- SubsetTank[order(SubsetTank$Day),]
    SubsetTank$AbsoluteAbundance[is.na(SubsetTank$AbsoluteAbundance)] <- 1 # If the sample is missing we estimate it was there, to make sure we have the worst case scenaria
    # label when OTU is present or absent
    SubsetTank$Present <- SubsetTank$AbsoluteAbundance[SubsetTank$Tank == Tank] > 0
    # Count number of days abundant after introduction
    NumberOfDaysPresent <- sum(cumprod(SubsetTank$Present))
    
    # Save the result
    ResultsNumberOfDaysPresent <- rbind(ResultsNumberOfDaysPresent, cbind.data.frame(Selected[c("OTU", "NewlyIntroduced", "Day", "SourceType", "Genus")], Tank, NumberOfDaysPresent))
    
    if (dim(SubsetTank[SubsetTank$Present,])[1] > 0){
          # Store the abundances
          ResultsAbundanceAfterIntro <- rbind(ResultsAbundanceAfterIntro, cbind.data.frame(SubsetTank[SubsetTank$Present,], i, Selected[c("NewlyIntroduced", "SourceType")]))
    }
  }
}
colnames(ResultsAbundanceAfterIntro)[8] <- "IntroDay"

# Get % of total community over the cultivation
PercentageCommunity <- sum(ResultsAbundanceAfterIntro$Abundance)/(dim(InfoTanks)[1])

# Remove variables that are no longer needed
remove(info, INFO, physeqobj, RelevantOTUs, Selected, Subset, SubsetTank, taxmat.ns, i, NumberOfDaysPresent, IdentifierControlSamples, OTU, Tank, TAX, PercentageCommunity, otumat.ns)
```


```r
# Plot number of days in the tank
p_NumberOfDaysPresentInTankAllTanks <- ResultsNumberOfDaysPresent %>%
                                  dplyr::mutate(SourceType = factor(SourceType, levels = c("Algae", "Artemia", "ExchangeWater", "Dry feed"))) %>%
                                  ggplot(data = ., aes(x = NumberOfDaysPresent, y = paste(OTU, " (", Genus, ")", sep = ""), fill = SourceType)) +
                                  geom_point(shape = 21, size = 3) + 
                                  labs(fill = "Source", y = "", x = expression(atop("Number of days present in ", paste("tanks after introduction (d)")))) +
                                  scale_x_continuous(minor_breaks = seq(2,18), breaks = seq(2,18,2), limits = c(1,18)) +
                                  scale_fill_manual(values = SourceColors, labels = c("Algae", "Artemia", "Exchange water", "Dry feed") ) +
                                  guides(fill = guide_legend(override.aes = list(size = 5, shape = 21))) +
                                  theme_cowplot() +
                                  theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom")
print(p_NumberOfDaysPresentInTankAllTanks)  

# Plot abundances after introduction
p_AbundanceAfterIntroductionAllTanks <- ResultsAbundanceAfterIntro %>%
                                  dplyr::mutate(SourceType = factor(SourceType, levels = c("Algae", "Artemia", "ExchangeWater", "Dry feed"))) %>%
                                  ggplot(data = ., aes(x = 100*Abundance, y = paste(OTU, " (", Genus, ")", sep = ""), fill = SourceType)) +
                                  geom_point(shape = 21, size = 3) + 
                                  labs(color = "", y = "", x = expression(atop("Relative abundance in tanks", paste("after introduction (%)")))) +
                                  scale_fill_manual(values = SourceColors) +
                                  theme_cowplot() +
                                  theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AbundanceAfterIntroductionAllTanks)

# Remove variables that are no longer needed
remove(ResultsNumberOfDaysPresent, ResultsAbundanceAfterIntro)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/ContributionsSourcesPlots-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/ContributionsSourcesPlots-2.png" width="70%" style="display: block; margin: auto;" />


```r
# Make plots ready
Legend <- cowplot::get_legend(p_NumberOfDaysPresentInTankAllTanks)
p_AbsoluteContributionSources <- p_AbsoluteContributionSources + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
p_NumberOfDaysPresentInTankAllTanks <- p_NumberOfDaysPresentInTankAllTanks + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.text.y = element_blank())
p_AbundanceAfterIntroductionAllTanks <- p_AbundanceAfterIntroductionAllTanks + ggplot2::theme(legend.position = "none", panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), axis.text.y = element_blank())
# Assemble and save
g1 <- plot_grid(p_AbsoluteContributionSources, p_NumberOfDaysPresentInTankAllTanks, p_AbundanceAfterIntroductionAllTanks, labels=c("B", "C", "D"), ncol = 3, nrow = 1, rel_widths = c(2.2, 1, 1), scale = 0.95)
g2 <- plot_grid(NULL, Legend, NULL, labels=c("", "", ""), ncol = 3, nrow = 1, rel_widths = c(2,0.9,0.9))
g3 <- plot_grid(g1, g2, labels = c("", ""), ncol = 1, nrow = 2, rel_heights = c(2, 0.2))
g4 <- plot_grid(p_sourcetracking, g3, labels = c("A", ""), ncol = 1, nrow = 2, rel_heights = c(2, 1.7))
ggsave(file = "Figures/SOURCES-AbsAb-Combined.png", width = 13, height = 17, dpi = 500, units = "in", g4)
ggsave(file = "FiguresPublication/SOURCES-AbsAb-Combined_300.tiff", width = 13, height = 17, dpi = 300, units = "in", plot = g4, compression = "lzw")
ggsave(file = "FiguresPublication/SOURCES-AbsAb-Combined_500.tiff", width = 13, height = 17, dpi = 500, units = "in", plot = g4, compression = "lzw")
ggsave(file = "FiguresPublication/SOURCES-AbsAb-Combined_700.tiff", width = 13, height = 17, dpi = 700, units = "in", plot = g4, compression = "lzw")
# Remove variables that are no longer needed
remove(Legend, p_AbsoluteContributionSources, p_NumberOfDaysPresentInTankAllTanks, p_AbundanceAfterIntroductionAllTanks, p_sourcetracking, SourceTrackingResultsOTUs, g1, g2, g3, g4)
```


```r
# Add info regarding the frequency of which the sources were added to the rearing water
AddFrequency <- data.frame(matrix(c("Algae", "Artemia", "ExchangeWater", "Dry feed", 10, 14, 6, 18), nrow = 4))
colnames(AddFrequency) <- c("Type", "Frequency")
AbsoluteLoadSources <- left_join(AbsoluteLoadSources, AddFrequency, by = c("Type"))
AbsoluteLoadSources$Frequency <- as.numeric(paste(AbsoluteLoadSources$Frequency))
AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$AbsoluteAmountAdded == 0] <- 1 # for plotting on log-scale

# Averages per source
Averages <- NULL
for (i in unique(AbsoluteLoadSources$Type)){
  Mean <- mean(AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$Type == i])
  Result <- cbind.data.frame(i, Mean, AbsoluteLoadSources$Frequency[AbsoluteLoadSources$Type == i][1])
  Averages <- rbind(Averages, Result)
}
colnames(Averages) <- c("Type", "Mean", "Frequency")

# Plot
p_AbsoluteContributionSources <- AbsoluteLoadSources %>%
                                  dplyr::mutate(Type = factor(Type, levels = c("Algae", "Artemia", "ExchangeWater", "Dry feed"))) %>%
                                  ggplot(data = ., aes(x = Frequency, y = AbsoluteAmountAdded, fill = Type)) + 
                                  geom_violin() +
                                  geom_point() + 
                                  labs(fill = "Source", y = "Bacterial load (cells/d)", x = "Frequency of addition (number of days)") +
                                  scale_y_continuous(labels = function(x) format(x, scientific = TRUE), trans = "log10", limits = c(1,1e10)) +
                                  scale_x_continuous(limits = c(1,20)) +
                                  scale_fill_manual(values = SourceColors, labels = c("Algae", "Artemia", "Exchange water", "Dry feed")) +
                                  geom_linerange(data = Averages, aes(y = Mean, xmin = Frequency - 1, xmax = Frequency + 1), size = 1.2) +
                                  theme_cowplot() +
                                  theme(panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AbsoluteContributionSources)
png("Figures/SOURCES-AbsAb-AmountAddedPerSourcePerDay.png", width = 7, height = 4,  res = 300, units = "in")
print(p_AbsoluteContributionSources)
dev.off()

# OTU's associated with each of the sources
AlgalContribution <- mean(AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$Type == "Algae"])
Artemiaontribution <- mean(AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$Type ==  "Artemia"])
ExchangeWaterContribution <- mean(AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$Type == "ExchangeWater"])
DryFeedContribution <- mean(AbsoluteLoadSources$AbsoluteAmountAdded[AbsoluteLoadSources$Type == "Dry feed"])

# Remove variables that are no longer needed
remove(AddFrequency, AbsoluteLoadSources, p_AbsoluteContributionSources, AlgalContribution, Artemiaontribution, ExchangeWaterContribution, DryFeedContribution, Mean, Averages, Result)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/AdditionFrequency-1.png" width="70%" style="display: block; margin: auto;" />

# Evaluate community assembly processes

## In the rearing water over time (using only tank OTU's)


```r
# For tanks 1 and 4, which, crashed, we only need the data untill before the crash
InfoTanksNotCrashed <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T2", "T3", "T5") & Metadata$Feeding_status == "B",]
InfoTanksT1 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T1") & Metadata$Feeding_status == "B" & Metadata$Day < 13,]
InfoTanksT4 <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & Metadata$Tank %in% c("T4") & Metadata$Feeding_status == "B" & Metadata$Day < 10,]
InfoTanks <- rbind.data.frame(InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4)

# OTU-table tanks
OTUTanks <- shared.t.ns[InfoTanks$Identifier]
OTUTanks <- OTUTanks[rowSums(OTUTanks) > 1,] # Remove singletons
```

### Estimate importance of selection

Note: Since running this analysis takes a long time, the results of the itterations were stored in a rds-file and reloaded into the workspace. When running this code for the first time, make sure to run also the parts that are currently marked as commented sections.


```r
# # Generate a phylogenetic tree on the OTU's
#     # Read the fasta file with the (aligned) reference sequences
#     RefSeqs <- Biostrings::readDNAStringSet(filepath = "Data/Illumina/fastq/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.0.03.rep.fasta", format = "fasta")
#     # Only keep the OTU's that were still retained after the scaling
#     RefSeqs <- RefSeqs[which(rownames(shared.t.ns.old) %in% rownames(OTUTanks))]
#     length(RefSeqs)
#     # Convert to a DNAbin object to make trees
#     RefSeqs <- as.DNAbin(RefSeqs)
#     # Calculate distances between the sequences
#     Distances <- dist.dna(RefSeqs, model = "TN93")
#     # Building the tree
#     NJ.tree <- nj(Distances) # neighbourhood joining (from ape)
#     # Prepare for plotting
#     NJ.tree <- ladderize(NJ.tree)
#     NJ.tree$tip.label <- str_extract(NJ.tree$tip.label, "Otu[0-9]{5}") # Extract from the sting "Otu" and then the following 5 numbers
# 
# # Add the OTU table to the tree and save it as a list
# communitydata <- list("phylo" = NJ.tree, "otutable" = t(OTUTanks))
# 
# # Calculate the weighted βMNTD for the communities
# beta.mntd.weighted <- as.matrix(comdistnt(communitydata$otutable, cophenetic(communitydata$phylo), abundance.weighted = TRUE))
# 
# # Sanity check
# identical(rownames(communitydata$otutable), colnames(beta.mntd.weighted))
# identical(rownames(communitydata$otutable), rownames(beta.mntd.weighted))
# 
# # Calculate the distribution of βMNTD under the null-hypothesis (i.e stochastic community assembly)
#     # Define the number of itterations
#     NumberOfItterations = 999 # as in Stegen et al. 2013
#     # Initialise
#     random.weighted.bMNTD.comp <- array(c(-999), dim = c(nrow(communitydata$otutable), nrow(communitydata$otutable), NumberOfItterations))
#     # For every itteration, shuffle the taxa labels on the tree and calculate the weighted βMNTD for the dataset
#     for (rep in 1:NumberOfItterations) {
#         random.weighted.bMNTD.comp[,,rep] = as.matrix(comdistnt(communitydata$otutable, taxaShuffle(cophenetic(communitydata$phylo)), abundance.weighted = TRUE, exclude.conspecifics = F))
#         print(c(date(),rep))
#     }
#     # Calculate the βNTI
#     weighted.bNTI <- matrix(c(NA), nrow = nrow(communitydata$otutable), ncol = nrow(communitydata$otutable))
#     for (columns in 1:(nrow(communitydata$otutable)-1)) {
#         for (rows in (columns+1):nrow(communitydata$otutable)) {
#             random.vals <- random.weighted.bMNTD.comp[rows,columns,]
#             weighted.bNTI[rows,columns] <- (beta.mntd.weighted[rows,columns] - mean(random.vals)) / sd(random.vals)
#             rm("random.vals")
#         }
#     }
#     rownames(weighted.bNTI) <- rownames(communitydata$otutable)
#     colnames(weighted.bNTI) <- rownames(communitydata$otutable)
# 
# # Save the results
# # saveRDS(object = weighted.bNTI, file = "Results/DYNAMICS-weightedbNTI-999iter.rds")
```


```r
# Upload the results
weighted.bNTI <- readRDS(file = "Results/DYNAMICS-weightedbNTI-999iter.rds")

# Get the values per tank so these can be plotted
ResultsWeighted.bNTI <- NULL
for (Tank in c("T1","T2","T3","T4","T5")){
  SamplesSelectedTank <- InfoTanks[InfoTanks$Tank == Tank & InfoTanks$Feeding_status == "B",]
  for (j in 1:(length(SamplesSelectedTank$Day)-1)){
    # Get the identifiers of the samples
    IDDayBefore <- SamplesSelectedTank$Identifier[SamplesSelectedTank$Day == sort(SamplesSelectedTank$Day)[j]]
    IDDayAfter <- SamplesSelectedTank$Identifier[SamplesSelectedTank$Day == sort(SamplesSelectedTank$Day)[j+1]]
    # Get the weighted bNTI for the pair (not necessarily in the order of the metadata, so check both combinations)
    Pair1 <- weighted.bNTI[IDDayAfter, IDDayBefore]
    Pair2 <- weighted.bNTI[IDDayBefore, IDDayAfter]
    # Get the value
    weighted.bNTI.tank <- c(Pair1, Pair2)[!is.na(c(Pair1, Pair2))]
    # Save the results
    Result <- cbind.data.frame(Tank, j, j+1, weighted.bNTI.tank)
    ResultsWeighted.bNTI <- rbind(ResultsWeighted.bNTI, Result)
  }
}
colnames(ResultsWeighted.bNTI) <- c("Tank", "DayBefore", "DayAfter", "Weighted.bNTI")
ResultsWeighted.bNTI$DayMid <- rowMeans(ResultsWeighted.bNTI[c("DayBefore", "DayAfter")])

# Remove variables that are no longer needed
remove(IDDayAfter, IDDayBefore, j, Pair1, Pair2, Tank, weighted.bNTI.tank, InfoTanksNotCrashed, InfoTanksT1, InfoTanksT4, Result, SamplesSelectedTank)
```

### Estimate importance of drift


```r
# # Prepare data
# spXsite <- t(OTUTanks)
# # Calculate RCbray
# RCout <- raup_crick_abundance(spXsite = spXsite, plot_names_in_col1 = FALSE, reps = 9999)
# # Save the results
# saveRDS(object = RCout, file = "Results/DYNAMICS-RaupCrick-BrayCurtis-9999iter.rds")

# Upload the results
RCout <- readRDS(file = "Results/DYNAMICS-RaupCrick-BrayCurtis-9999iter.rds")
RCout <- as.matrix(RCout)

# Get the values per tank 
ResultsRC <- NULL
for (Tank in c("T1","T2","T3","T4","T5")){
  SamplesSelectedTank <- InfoTanks[InfoTanks$Tank == Tank & InfoTanks$Feeding_status == "B",]
  for (j in 1:(length(SamplesSelectedTank$Day)-1)){
    # Get the identifiers of the samples
    IDDayBefore <- SamplesSelectedTank$Identifier[SamplesSelectedTank$Day == sort(SamplesSelectedTank$Day)[j]]
    IDDayAfter <- SamplesSelectedTank$Identifier[SamplesSelectedTank$Day == sort(SamplesSelectedTank$Day)[j+1]]
    # Get the RC for the pair
    RC.tank <- RCout[IDDayAfter, IDDayBefore]
    # Save the results
    Result <- cbind.data.frame(Tank, j, j+1, RC.tank)
    ResultsRC <- rbind(ResultsRC, Result)
  }
}
colnames(ResultsRC) <- c("Tank", "DayBefore", "DayAfter", "RC")
ResultsRC$DayMid <- rowMeans(ResultsRC[c("DayBefore", "DayAfter")])

# Remove the values for which selection was detected, since it does not make sense to check for drift in these samples
TransitionsWithSelection <- ResultsWeighted.bNTI[abs(ResultsWeighted.bNTI$Weighted.bNTI) > 2,]
for (i in 1:dim(TransitionsWithSelection)[2]){
  # Get the day and tank number
  Day <- TransitionsWithSelection$DayBefore[i]
  Tank <- TransitionsWithSelection$Tank[i]
  # Remove the RC values for this transition
  ResultsRC$RC[ResultsRC$Tank == Tank & ResultsRC$DayBefore == Day] <- NA  
}


# Remove variables that are no longer needed
remove(IDDayAfter, IDDayBefore, Day, i, j, RC.tank, Tank, OTUTanks, Result, ResultsRC, ResultsWeighted.bNTI, SamplesSelectedTank, TransitionsWithSelection)
```

### Plot community assembly


```r
RC <- RCout
bNTI <- weighted.bNTI

# Get the average for every day
CommunityAssemblyResults <- NULL
for (i in 1:17){
  # Get days
  DayBefore <- i
  DayAfter <- i + 1
  # For every tank get the value of bNTI and RC
  Results <- NULL
  for (Tank in c("T1","T2","T3","T4","T5")){
    # Get samples
    IDDayBefore <- InfoTanks$Identifier[InfoTanks$Day == DayBefore & InfoTanks$Tank == Tank]
    IDDayAfter <- InfoTanks$Identifier[InfoTanks$Day == DayAfter & InfoTanks$Tank == Tank]
    if (length(InfoTanks$Identifier[InfoTanks$Day %in% c(DayBefore,DayAfter) & InfoTanks$Tank == Tank]) == 2){ # If we have both samples
        # bNTI
        Pair1 <- bNTI[IDDayAfter, IDDayBefore]
        Pair2 <- bNTI[IDDayBefore, IDDayAfter]
        weighted.bNTI.tank <- c(Pair1, Pair2)[!is.na(c(Pair1, Pair2))]
        # RC
        RC.tank <- RC[IDDayAfter, IDDayBefore]
        # Store
        Result <- cbind.data.frame(weighted.bNTI.tank, RC.tank)
        Results <- rbind(Results, Result)
    }  
  }
  
  # Evaluate selection 
      # Calculate fraction bNTI > 2
      fractionVariableSelection <- sum(Results$weighted.bNTI.tank > 2)/length(Results$weighted.bNTI.tank)
      
      # Calculate fraction bNTI < -2
      fractionHomogenuousSelection <- sum(Results$weighted.bNTI.tank < -2)/length(Results$weighted.bNTI.tank)
      
      # Calculate fraction no selection
      fractionNoSelection <- 1 - fractionVariableSelection - fractionHomogenuousSelection
      
  # Evaluate drift (ONLY ON SAMPLES FOR WHICH THERE WAS NO INDICATION FOR SELECTION - remove the values for which selection was detected)
      ResultsNoSelection <- Results[!abs(Results$weighted.bNTI.tank) > 2,]
      
      # Calculate fraction RC > 0.95
      fractionDispersalLimitation <- sum(ResultsNoSelection$RC.tank >= 0.95)/length(ResultsNoSelection$RC.tank)
      
      # Calculate fraction RC < -0.95
      fractionHomogenisingDispersal <- sum(ResultsNoSelection$RC.tank <= -0.95)/length(ResultsNoSelection$RC.tank)
      
      # Calculate |RC| < 0.95
      fractionDrift <- sum(abs(ResultsNoSelection$RC.tank) < 0.95)/length(ResultsNoSelection$RC.tank)
      
      # Recalculate the no-selection fractions
      fractionDispersalLimitation <- fractionDispersalLimitation*fractionNoSelection
      fractionHomogenisingDispersal <- fractionHomogenisingDispersal*fractionNoSelection
      fractionDrift <- fractionDrift*fractionNoSelection
  
  # Store results
  CommunityAssemblyResult <- cbind.data.frame(i, i+1, fractionVariableSelection, fractionHomogenuousSelection, fractionDispersalLimitation, fractionHomogenisingDispersal, fractionDrift)
  CommunityAssemblyResults <- rbind(CommunityAssemblyResults, CommunityAssemblyResult)
}
colnames(CommunityAssemblyResults) <- c("DayBefore", "DayAfter", "Variable selection", "Homogenuous selection", "Dispersal limitation and drift", "Homogenising dispersal", "Drift alone")


CommunityAssemblyResults <- melt(CommunityAssemblyResults, id.vars = c("DayBefore", "DayAfter"))
# Plot
p_AssemblyPerDay <- CommunityAssemblyResults %>%
                      ggplot(data = ., aes(x = DayAfter, y = 100*value, fill = variable)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = ColorsCommunityAssembly) +
                      labs(fill = "") +
                      guides(fill = guide_legend(reverse = FALSE, keywidth = 1, keyheight = 1)) +
                      scale_x_continuous(minor_breaks = seq(2,18), limits = c(1,19)) + 
                      labs(y = "Relative contribution (%)", x = "Time (d)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AssemblyPerDay)
png("Figures/DYNAMICS-CommunityAssemblyEstimation-fractions-PerDay.png", width = 8, height = 4, res = 500, units = "in")
print(p_AssemblyPerDay)
dev.off()


# Remove variables that will not be used further
remove(DayAfter, DayBefore, fractionDrift, fractionDispersalLimitation, fractionHomogenisingDispersal, fractionHomogenuousSelection, fractionVariableSelection, fractionNoSelection, i, IDDayAfter, IDDayBefore, Pair1, Pair2, RC.tank, p_AssemblyPerDay, Result, Results, ResultsNoSelection, CommunityAssemblyResult, CommunityAssemblyResults, Tank)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/PlotCommunityAssemblyPerDay-1.png" width="70%" style="display: block; margin: auto;" />


```r
# Get the average for every tank
CommunityAssemblyResults <- NULL
for (Tank in c("T1","T2","T3","T4","T5")){
    Results <- NULL
    # For every day get the value of bNTI and RC
    for (i in 1:17){
    # Get days
    DayBefore <- i
    DayAfter <- i + 1
    # Get samples
    IDDayBefore <- InfoTanks$Identifier[InfoTanks$Day == DayBefore & InfoTanks$Tank == Tank]
    IDDayAfter <- InfoTanks$Identifier[InfoTanks$Day == DayAfter & InfoTanks$Tank == Tank]
    if (length(InfoTanks$Identifier[InfoTanks$Day %in% c(DayBefore,DayAfter) & InfoTanks$Tank == Tank]) == 2){ # If we have both samples
        # bNTI
        Pair1 <- bNTI[IDDayAfter, IDDayBefore]
        Pair2 <- bNTI[IDDayBefore, IDDayAfter]
        weighted.bNTI.tank <- c(Pair1, Pair2)[!is.na(c(Pair1, Pair2))]
        # RC
        RC.tank <- RC[IDDayAfter, IDDayBefore]
        # Store
        Result <- cbind.data.frame(weighted.bNTI.tank, RC.tank)
        Results <- rbind(Results, Result)
    }  
  }
  
  # Evaluate selection 
        # Calculate fraction bNTI > 2
        fractionVariableSelection <- sum(Results$weighted.bNTI.tank > 2)/length(Results$weighted.bNTI.tank)
        
        # Calculate fraction bNTI < -2
        fractionHomogenuousSelection <- sum(Results$weighted.bNTI.tank < -2)/length(Results$weighted.bNTI.tank)
        
        # Calculate fraction no selection
        fractionNoSelection <- 1 - fractionVariableSelection - fractionHomogenuousSelection
      
  # Evaluate drift (ONLY ON SAMPLES FOR WHICH THERE WAS NO INDICATION FOR SELECTION - remove the values for which selection was detected)
        ResultsNoSelection <- Results[!abs(Results$weighted.bNTI.tank) > 2,]    
    
        # Calculate fraction RC > 0.95
        fractionDispersalLimitation <- sum(ResultsNoSelection$RC.tank >= 0.95)/length(ResultsNoSelection$RC.tank)
        
        # Calculate fraction RC < -0.95
        fractionHomogenisingDispersal <- sum(ResultsNoSelection$RC.tank <= -0.95)/length(ResultsNoSelection$RC.tank)
        
        # Calculate |RC| < 0.95
        fractionDrift <- sum(abs(ResultsNoSelection$RC.tank) < 0.95)/length(ResultsNoSelection$RC.tank)
        
        # Recalculate the no-selection fractions
        fractionDispersalLimitation <- fractionDispersalLimitation*fractionNoSelection
        fractionHomogenisingDispersal <- fractionHomogenisingDispersal*fractionNoSelection
        fractionDrift <- fractionDrift*fractionNoSelection
  
  # Calculate the amount of samples we had for this tank to be able to rescale the bargraphs
  FractionTime <- dim(Results)[1]/17
        
  # Store results
  CommunityAssemblyResult <- cbind.data.frame(Tank, fractionVariableSelection, fractionHomogenuousSelection, fractionDispersalLimitation, fractionHomogenisingDispersal, fractionDrift, FractionTime)
  CommunityAssemblyResults <- rbind(CommunityAssemblyResults, CommunityAssemblyResult)
}
colnames(CommunityAssemblyResults) <- c("Tank", "Variable selection", "Homogenuous selection", "Dispersal limitation and drift", "Homogenising dispersal", "Drift alone", "FractionOfSamples")


# Plot, without correcting for the number of samples
CommunityAssemblyResultsAll <- CommunityAssemblyResults[c("Tank", "Variable selection", "Homogenuous selection", "Dispersal limitation and drift", "Homogenising dispersal", "Drift alone")]
CommunityAssemblyResultsAll <- melt(CommunityAssemblyResultsAll, id.vars = c("Tank"))
p_AssemblyPerTank <- CommunityAssemblyResultsAll %>%
                      ggplot(data = ., aes(x = Tank, y = 100*value, fill = variable)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = ColorsCommunityAssembly, labels = c("Variable selection", "Homogenuous selection", "Dispersal limitation and drift", "Homogenising dispersal", "Drift alone")) +
                      labs(fill = "", x = "") +
                      guides(fill = guide_legend(reverse = FALSE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative contribution (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AssemblyPerTank)
png("Figures/DYNAMICS-CommunityAssemblyEstimation-fractions-PerTank.png", width = 8, height = 5, res = 500, units = "in")
print(p_AssemblyPerTank)
dev.off()


# Plot, with correcting for the number of samples
CommunityAssemblyResults$`Variable selection` <- CommunityAssemblyResults$FractionOfSamples*CommunityAssemblyResults$`Variable selection`
CommunityAssemblyResults$`Homogenuous selection` <- CommunityAssemblyResults$FractionOfSamples*CommunityAssemblyResults$`Homogenuous selection`
CommunityAssemblyResults$`Dispersal limitation and drift` <- CommunityAssemblyResults$FractionOfSamples*CommunityAssemblyResults$`Dispersal limitation and drift`
CommunityAssemblyResults$`Homogenising dispersal` <- CommunityAssemblyResults$FractionOfSamples*CommunityAssemblyResults$`Homogenising dispersal`
CommunityAssemblyResults$`Drift alone` <- CommunityAssemblyResults$FractionOfSamples*CommunityAssemblyResults$`Drift alone`
CommunityAssemblyResults <- CommunityAssemblyResults[c("Tank", "Variable selection", "Homogenuous selection", "Dispersal limitation and drift", "Homogenising dispersal", "Drift alone")]
CommunityAssemblyResults <- melt(CommunityAssemblyResults, id.vars = c("Tank"))
p_AssemblyPerTankWeighted <- CommunityAssemblyResults %>%
                      ggplot(data = ., aes(x = Tank, y = 100*value, fill = variable)) + 
                      geom_bar(stat = "identity", colour = "black") +
                      scale_fill_manual(values = ColorsCommunityAssembly) +
                      labs(fill = "") +
                      guides(fill = guide_legend(reverse = FALSE, keywidth = 1, keyheight = 1)) +
                      ylab("Relative contribution (%)") +
                      theme_cowplot() +
                      theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AssemblyPerTankWeighted)


# Remove variables that will not be used further
remove(DayAfter, DayBefore, fractionDrift, fractionDispersalLimitation, fractionHomogenisingDispersal, fractionHomogenuousSelection, fractionVariableSelection, fractionNoSelection, i, IDDayAfter, IDDayBefore, Pair1, Pair2, RC.tank, Result, Results, ResultsNoSelection, CommunityAssemblyResult, CommunityAssemblyResults, CommunityAssemblyResultsAll, Tank, p_AssemblyPerTankWeighted, FractionTime, bNTI, RC, RCout, weighted.bNTI, weighted.bNTI.tank, p_Weighted.bNTI, p_RC)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/PlotCommunityAssemblyPerTank-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/PlotCommunityAssemblyPerTank-2.png" width="70%" style="display: block; margin: auto;" />

## From the pheripheral microbiomes to the rearing water (using tank and sources OTUs)


```r
# Only the samples of the entire communities are needed for this
InfoSources <- Metadata[Metadata$WholeCommunityOrSorted == "Whole community" & !(Metadata$Tank %in% c("T1", "T2", "T3", "T4", "T5")),]
# Combine tanks and sources
InfoAll <- rbind.data.frame(InfoSources, InfoTanks)

# OTU-table tanks
OTUTanks <- shared.t.ns[InfoAll$Identifier]
OTUTanks <- OTUTanks[rowSums(OTUTanks) > 1,] # Remove singletons
```

### Estimate importance of selection


```r
# # Generate a phylogenetic tree on the OTU's
#     # Read the fasta file with the (aligned) reference sequences
#     RefSeqs <- Biostrings::readDNAStringSet(filepath = "Data/Illumina/fastq/stability.trim.contigs.good.unique.good.filter.unique.precluster.pick.pick.opti_mcc.0.03.rep.fasta", format = "fasta")
#     # Only keep the OTU's that were still retained after the scaling
#     RefSeqs <- RefSeqs[which(rownames(shared.t.ns.old) %in% rownames(OTUTanks))]
#     length(RefSeqs)
#     # Convert to a DNAbin object to make trees
#     RefSeqs <- as.DNAbin(RefSeqs)
#     # Calculate distances between the sequences
#     Distances <- dist.dna(RefSeqs, model = "TN93")
#     # Building the tree
#     NJ.tree <- nj(Distances) # neighbourhood joining (from ape)
#     # Prepare for plotting
#     NJ.tree <- ladderize(NJ.tree)
#     NJ.tree$tip.label <- str_extract(NJ.tree$tip.label, "Otu[0-9]{5}") # Extract from the sting "Otu" and then the following 5 numbers
# 
# # Add the OTU table to the tree and save it as a list
# communitydata <- list("phylo" = NJ.tree, "otutable" = t(OTUTanks))
# 
# # Calculate the weighted βMNTD for the communities
# beta.mntd.weighted <- as.matrix(comdistnt(communitydata$otutable, cophenetic(communitydata$phylo), abundance.weighted = TRUE))
# 
# # Sanity check
# identical(rownames(communitydata$otutable), colnames(beta.mntd.weighted))
# identical(rownames(communitydata$otutable), rownames(beta.mntd.weighted))
# 
# # Calculate the distribution of βMNTD under the null-hypothesis (i.e stochastic community assembly)
#     # Define the number of itterations
#     NumberOfItterations = 999 # as in Dini-andreote et al. 2015
#     # Initialise
#     random.weighted.bMNTD.comp = array(c(-999), dim = c(nrow(communitydata$otutable), nrow(communitydata$otutable), NumberOfItterations))
#     # For every itteration, shuffle the taxa labels on the tree and calculate the weighted βMNTD for the dataset
#     for (rep in 1:NumberOfItterations) {
#         random.weighted.bMNTD.comp[,,rep] = as.matrix(comdistnt(communitydata$otutable, taxaShuffle(cophenetic(communitydata$phylo)), abundance.weighted = TRUE, exclude.conspecifics = F))
#         print(c(date(),rep))
#     }
#     # Calculate the βNTI
#     weighted.bNTI <- matrix(c(NA), nrow = nrow(communitydata$otutable), ncol = nrow(communitydata$otutable))
#     for (columns in 1:(nrow(communitydata$otutable)-1)) {
#         for (rows in (columns+1):nrow(communitydata$otutable)) {
#             random.vals <- random.weighted.bMNTD.comp[rows,columns,]
#             weighted.bNTI[rows,columns] <- (beta.mntd.weighted[rows,columns] - mean(random.vals)) / sd(random.vals)
#             rm("random.vals")
#         }
#     }
#     rownames(weighted.bNTI) <- rownames(communitydata$otutable)
#     colnames(weighted.bNTI) <- rownames(communitydata$otutable)
# 
# # Save the results
# saveRDS(object = weighted.bNTI, file = "Results/DYNAMICS-weightedbNTI-WithSources-999iter.rds")
```

### Estimate importance of drift


```r
# # Prepare data
# spXsite <- t(OTUTanks)
# # Calculate RCbray
# RCout <- raup_crick_abundance(spXsite = spXsite, plot_names_in_col1 = FALSE, reps = 999)
# # Save the results
# saveRDS(object = RCout, file = "Results/DYNAMICS-RaupCrick-BrayCurtis-WithSources-999iter.rds")
```

### Plot community assembly


```r
# Read the excel file with the info regarding the timing of the source introduction
SourceInfo <- read.xlsx(xlsxFile = "Metadata/MetadataSourceTracking.xlsx")
weighted.bNTI <- readRDS(file = "Results/DYNAMICS-weightedbNTI-WithSources-999iter.rds")
RCout <- readRDS(file = "Results/DYNAMICS-RaupCrick-BrayCurtis-WithSources-999iter.rds")
RCout <- as.matrix(RCout)


Results <- NULL
# For every source
for(i in unique(SourceInfo$Type)){
  # Info selected source types
  SelectedSourceInfo <- SourceInfo[SourceInfo$Type == i,]
  
  # For every time this source was added: get the bNTI and RC values
  for (j in 1:dim(SelectedSourceInfo)[1]){
    # Get samples from the tank the first day after the addition
    DayAfter <- SelectedSourceInfo$DayBefore[j] + 1
    TanksDayAfter <- InfoTanks[InfoTanks$Day == DayAfter,]
    
    # Get the bNTI and RC values for every combination of source and day after source
    IdentifiersOfIntrest <- c(SelectedSourceInfo$SampleIdentifierSource[j], TanksDayAfter$Identifier)
    SelectedbNTI <- weighted.bNTI[IdentifiersOfIntrest,IdentifiersOfIntrest]
    SelectedRCout <- RCout[IdentifiersOfIntrest,IdentifiersOfIntrest]
    SelectedbNTI <- SelectedbNTI[,as.character(SelectedSourceInfo$SampleIdentifierSource[j])][c(-1)]
    SelectedRCout <- SelectedRCout[,as.character(SelectedSourceInfo$SampleIdentifierSource[j])][c(-1)]
    
    #
    Result <- cbind.data.frame(SelectedSourceInfo$DayBefore[j], SelectedSourceInfo$Type[j], SelectedSourceInfo$Source[j], SelectedbNTI, SelectedRCout)
    colnames(Result) <- c("DayBefore", "Type", "Source", "bNTI", "RC") 
    
    # Determine community assembly process
          # Evaluate selection 
                # Calculate fraction bNTI > 2
                fractionVariableSelection <- sum(Result$bNTI > 2)/length(Result$bNTI)
                
                # Calculate fraction bNTI < -2
                fractionHomogenuousSelection <- sum(Result$bNTI < -2)/length(Result$bNTI)
                
                # Calculate fraction no selection
                fractionNoSelection <- 1 - fractionVariableSelection - fractionHomogenuousSelection
          
    
    # Get the community assambly result
    CommunityAssemblyResult <- cbind.data.frame(SelectedSourceInfo$DayBefore[j], SelectedSourceInfo$Type[j], SelectedSourceInfo$Source[j], fractionVariableSelection, fractionHomogenuousSelection, fractionNoSelection)
    
    # Save the results  
    Results <- rbind(Results, CommunityAssemblyResult)
  }
  
}
colnames(Results)[1:3] <- c("DayBefore", "Type", "Source") 
Results[is.nan(Results)] <- 0 # Replace NaN values with 0

# For the dry feed there is always 1 day where both dry feeds are being added therefore the total fraction on those days is 2 in stead of 1: normalise these for plotting
DaysTwoTypesOfDryFeed <- c(5, 7, 12, 15)
for (i in DaysTwoTypesOfDryFeed){
      # On the day 2 dry feeds were added normalise the fractions to one
      Results[Results$Type == "Dry feed" & Results$DayBefore == i, c("fractionVariableSelection", "fractionHomogenuousSelection",  "fractionNoSelection")] <- rbind(colSums(0.5*Results[Results$Type == "Dry feed" & Results$DayBefore == i, c("fractionVariableSelection", "fractionHomogenuousSelection",  "fractionNoSelection")]), c(0,0,0,0,0))

      # 
      Remove <- rownames(Results[Results$Type == "Dry feed" & Results$DayBefore == i, c("fractionVariableSelection", "fractionHomogenuousSelection",  "fractionNoSelection")])[2]
      Results <- Results[!row.names(Results) == Remove,]
}



Results <- melt(Results, id.vars = c("DayBefore", "Type", "Source"))
# Make names nicer for plotting
Results$variable <- as.character(Results$variable)
Results$variable[Results$variable == "fractionVariableSelection"] <- c("Variable selection")
Results$variable[Results$variable == "fractionHomogenuousSelection"] <- c("Homogenuous selection")
Results$variable[Results$variable == "fractionNoSelection"] <- c("No selection")
Results$Type <- as.character(Results$Type)
Results$Type[Results$Type == "ExchangeWater"] <- c("Exchange water")

# Fix levels
Results$variable <- factor(Results$variable, levels = c("Variable selection", "Homogenuous selection", "No selection"))

# Plot
p_AssemblyForSources <- Results %>%
                          ggplot(data = ., aes(x = DayBefore+1, y = 100*value, fill = variable)) + 
                          geom_bar(stat = "identity", colour = "black") +
                          scale_fill_manual(values = ColorsCommunityAssemblyII) +
                          labs(fill = "") +
                          facet_wrap(. ~ Type) +
                          guides(fill = guide_legend(reverse = FALSE, keywidth = 1, keyheight = 1)) +
                          scale_x_continuous(minor_breaks = seq(2,18), limits = c(1,19)) + 
                          labs(y = "Relative contribution (%)", x = "Time (d)", legend = "             ") +
                          theme_cowplot() +
                          theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet))
print(p_AssemblyForSources)
png("Figures/DYNAMICS-CommunityAssemblyEstimation-fractions-FromSources-PerDay.png", width = 8, height = 6, res = 500, units = "in")
print(p_AssemblyForSources)
dev.off()


# Calculate average for each sources
ResultsPerSource <- NULL
for (i in unique(Results$Type)){
  # For each assembly process, get average
  for (j in unique(Results$variable)){
    Result <- mean(Results[Results$Type == i & Results$variable == j , "value"])
    ResultsPerSource <- rbind(ResultsPerSource, cbind.data.frame(i, j, Result))
  }
}
colnames(ResultsPerSource) <- c("Type", "Assembly", "Value")
# Plot
p_AssemblyPerSourceType <- ResultsPerSource %>%
                            dplyr::mutate(Tank = factor(Assembly, levels = c("Homogenuous selection", "Variable selection", "No selection"))) %>% 
                            ggplot(data = ., aes(x = Type, y = 100*Value, fill = Assembly)) + 
                            geom_bar(stat = "identity", colour = "black") +
                            scale_fill_manual(values = c(ColorsCommunityAssemblyII[2], ColorsCommunityAssemblyII[3], ColorsCommunityAssemblyII[1])) +
                            labs(fill = "") +
                            guides(fill = guide_legend(reverse = FALSE, keywidth = 1, keyheight = 1)) +
                            labs(y = "Relative contribution (%)", x = " ", legend = "             ") +
                            theme_cowplot() +
                            theme(legend.text.align = 0, panel.grid.major = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), panel.grid.minor = element_line(size = 0.3, linetype = 'solid', colour = ColorBlocksFacet), legend.position = "bottom")
print(p_AssemblyPerSourceType)

# Remove variables that are no longer needed
remove(CommunityAssemblyResult, g1, InfoAll, InfoSources, InfoTanks, Legend, OTUTanks, p_AssemblyForSources, RCout, Results, ResultsPerSource, SelectedSourceInfo, SourceInfo, TanksDayAfter, weighted.bNTI, DayAfter, DaysTwoTypesOfDryFeed, fractionHomogenuousSelection, fractionNoSelection, fractionVariableSelection, i, IdentifiersOfIntrest, j, Remove, Result, SelectedbNTI, SelectedRCout)
```

<img src="AnalysisSourceTrackingFinal_files/figure-html/SelectinfFromSources-1.png" width="70%" style="display: block; margin: auto;" /><img src="AnalysisSourceTrackingFinal_files/figure-html/SelectinfFromSources-2.png" width="70%" style="display: block; margin: auto;" />


```r
p_AssemblyPerTank_bottom <- p_AssemblyPerTank + guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1, nrow = 5)) + theme(legend.position = "bottom", legend.margin=margin(0,0,0,2,"cm"))
p_AssemblyPerSourceType_bottom <- p_AssemblyPerSourceType + guides(fill = guide_legend(reverse = TRUE, keywidth = 1, keyheight = 1, nrow = 5)) + theme(legend.position = "bottom", legend.margin=margin(0,0,0,2,"cm"))
# Assemble
g1 <- plot_grid(p_AssemblyPerTank_bottom, p_AssemblyPerSourceType_bottom, labels = c("A", "B"), rel_widths = c(4,4), ncol = 2, nrow = 1)
ggsave(file = "Figures/DYNAMICS-CommunityAssemblyEstimation-TanksAndSources.png", width = 10, height = 5, dpi = 700, units = "in", g1)
ggsave(file = "FiguresPublication/DYNAMICS-CommunityAssemblyEstimation-TanksAndSources_300.tiff", width = 10, height = 5, dpi = 300, units = "in", plot = g1, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-CommunityAssemblyEstimation-TanksAndSources_500.tiff", width = 10, height = 5, dpi = 500, units = "in", plot = g1, compression = "lzw")
ggsave(file = "FiguresPublication/DYNAMICS-CommunityAssemblyEstimation-TanksAndSources_700.tiff", width = 10, height = 5, dpi = 700, units = "in", plot = g1, compression = "lzw")

# Remove variables that are no longer needed
remove(g1, p_AssemblyPerSourceType, p_AssemblyPerSourceType_bottom, p_AssemblyPerSourceType_no, p_AssemblyPerTank, p_AssemblyPerTank_bottom)
```