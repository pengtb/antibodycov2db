---
title: QuickStart Guide
layout: single
author_profile: false
permalink: /

toc: true
toc_label: Overview
toc_sticky: true

excerpt: "A database for antibodies targeting SARS-CoV2 spike WT/mutant RBD."
header: 
    overlay_color: "#333"
    # actions:
    #  - label: "Download All"
    #    url: "https://github.com"

---
# What is this db for?
This database aims to provide more detailed information for antibodies binding to RBD of SARS-CoV2 spike protein. Compared with other similar ones, the database provides following features:
- **Sources** of binding evidence and sequence records are curated.
- **Quantitative binding affinities** (a.k.a. *K<sub>D</sub>*) are provided for antibodies with SPR/BLI experiments.
- **Variable domains** are annotated using [InterProScan][interproscan] and their sequences are **numbered** with [IMGT scheme][imgt] using [ANARCI][anarci] as well.
- **Apoform structures** of antibodies are predicted using [IgFold][igfold] and can be visualized.
- **Datasets** for either binary or quantitative binding affinities prediction against WT/mutant RBD are provided.

<!-- Reference links -->
[anarci]: <https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabpred/anarci/>
[interproscan]: <https://www.ebi.ac.uk/interpro/about/interproscan/>
[igfold]: <https://github.com/Graylab/IgFold>
[imgt]: <https://www.imgt.org/IMGTScientificChart/Numbering/IMGTcorrespondence.html>