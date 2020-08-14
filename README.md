# SourceTrackingDynamicsShrimp

The full analysis of the paper [***Rearing water microbiomes in white leg shrimp (Litopenaeus vannamei) larviculture assemble stochastically and are influenced by the microbiomes of live feed products***](https://github.com/jeheyse/SourceTrackingDynamicsShrimp) by Jasmine Heyse, Ruben Props, Pantipa Kongnuan, Peter De Schryver, Geert Rombaut, Tom Defoirdt and Nico Boon.

Before starting the analysis, the FCM data should be dowloaded from FlowRepository under accession ID FR-FCM-Z2LM (on-site measurements) and ID FR-FCM-Z2LN (off-site measurements) and stored in folders named _Data/FCM/OnSite_, _Data/FCM/OffSite_ and _Data/FCM/OffSite_. The raw sequence data of the natural and mock communities can be dowloaded from the NCBI Sequence Read Archive (SRA) under accession ID PRJNA637486. These should be stored in a folder named _Data/Illumina/fastq_. All metadata files are available on this repository.

The analysis pipeline of the raw Illumina data, using MOTHUR, can be found in _MOTHUR.html_. 

The data analysis for the processed Illumina data and flow cytometry data can be found in _AnalysisSourceTrackingFinal.html_.

The final file structure should be: 

```
├── AnalysisSourceTrackingFinal.Rmd
├── AnalysisSourceTrackingFinal.html
├── AnalysisSourceTrackingFinal.md
├── MOTHUR.Rmd
├── MOTHUR.html
├── MOTHUR.md
├── /Functions
├── /Metadata
├── /Data
    ├── /FCM
	    ├── OnSite
	    ├── OffSite
	    ├── OffSiteFeeds
    ├── /Illumina
	    ├── /fastq
```