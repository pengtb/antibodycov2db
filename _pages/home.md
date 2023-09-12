---
title: QuickStart Guide
layout: single
author_profile: false
permalink: /

excerpt: "A database for antibodies targeting SARS-CoV2 spike WT/mutant RBD."
header: 
    # overlay_color: "#333"
    overlay_image: "/assets/images/header.png"
    overlay_filter: 0.5
    caption: "Diagram Credit: [**EPIGENTEK**](https://www.epigentek.com/)"
    # actions:
    #  - label: "Download All"
    #    url: "https://github.com"

classes: wide
---
# What is this db for?
This database aims to provide more detailed information for antibodies binding to RBD of SARS-CoV2 spike protein. Compared with other similar ones, the database provides following features:
- **Sources** of binding evidence and sequence records are curated.
- **Quantitative binding affinities** (a.k.a. *K<sub>D</sub>*) are provided for antibodies with SPR/BLI experiments.
- **Variable domains** are annotated using [InterProScan][interproscan] and their sequences are **numbered** with [IMGT scheme][imgt] using [ANARCI][anarci] as well.
- **Apoform structures** of antibodies are predicted using [IgFold][igfold] and can be visualized.
- **Datasets** for either binary or quantitative binding affinities prediction against WT/mutant RBD are provided.

# How to use?
- To **download** the curated database or build datasets, please use [Download all in one][download-all] to download all the tables in one go, or download the table needed by [Select one table to download][download-one].
- To **browse** the databset, please use [Browse][browse] for online browsing. You can also click the link of each entry for more details. Besides, to check the details of the database, you can first download the sqlite database by [Download sqlite db][download-sqlite] and then use [SQLite Viewer][sqlite-viewer] to browse.
- To find a specific antibody, please use [Search][search] to **search** by ID, name, or sequence.
- Some **analyses** like data distribution and correlations with binding affinities are provided in [Analysis][analysis].

<!-- Reference links -->
[anarci]: <https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabpred/anarci/>
[interproscan]: <https://www.ebi.ac.uk/interpro/about/interproscan/>
[igfold]: <https://github.com/Graylab/IgFold>
[imgt]: <https://www.imgt.org/IMGTScientificChart/Numbering/IMGTcorrespondence.html>
[download-all]: <antibodycov2db/download/#download-all>
[download-one]: <antibodycov2db/download/#download-one>
[download-sqlite]: <[antibodycov2db/assets/db/collected_0825.db]>
[browse]: <antibodycov2db/browse/>
[sqlite-viewer]: <http://inloop.github.io/sqlite-viewer>
[search]: <antibodycov2db/search/>
[analysis]: <antibodycov2db/analysis/>