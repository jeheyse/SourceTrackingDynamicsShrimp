# SourceTrackingDynamicsShrimp

The full analysis of the paper [***Source tracking and community assembly of rearing water microbiomes in white leg shrimp (Litopenaeus vannamei) larviculture***](https://github.com/jeheyse/SourceTrackingDynamicsShrimp) by Jasmine Heyse, Ruben Props, Celine De Maesschalck, Geert Rombaut, Tom Defoirdt and Nico Boon.

Click [here](https://help.github.com/en/articles/cloning-a-repository) to get instructions on how to clone a repository to your local computer.

Before starting the analysis, the FCM data should be dowloaded from FlowRepository under accession ID FR-FCM-Z2LM (on-site measurements) and ID FR-FCM-Z2LN (off-site measurements) and stored in folders named _Data/FCM/OnSite_, _Data/FCM/OffSite_ and _Data/FCM/OffSite_.


The analysis pipeline of the raw Illumina data, using MOTHUR, can be found in _MOTHUR.html_. 

The data analysis for the processed Illumina data and flow cytometry data can be found in _AnalysisSourceTracking.html_.

The final file structure should be: 

```
├── AnalysisSourceTracking.Rmd
├── AnalysisSourceTracking.html
├── AnalysisSourceTracking.md
├── MOTHUR.Rmd
├── MOTHUR.html
├── MOTHUR.md
├── /Metadata
├── /Data
    ├── Metadata_FCM.csv
    ├── Metadata_Raman.csv
    ├── /FCSfiles
    └── /Ramanfiles
```