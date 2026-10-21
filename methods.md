# Methods

Generated 2026-10-21 by build_site_data.py v1.12. Datasets: Green et al. 2024 (`green`); Mathys et al. 2019 (`mathys2019`).

## Which test produced which result

| Result | Test | Unit compared | Threshold | Multiple-testing correction |
|---|---|---|---|---|
| Gene change in AD, main columns (default) | Wilcoxon rank-sum (nucleus-level, step 11b) | nuclei of AD vs control donors, within one cell type | padj < 0.05 | Benjamini-Hochberg over the tested genes of each cell type |
| Donor-level (pseudobulk) test | DESeq2 Wald test (step 11) | donors (pseudobulk samples), within one cell type | padj < 0.05 | Benjamini-Hochberg over the tested genes of each cell type, with DESeq2 independent filtering |
| Co-expression module in AD | Wilcoxon rank-sum (step 12) | donor-mean module eigengenes | padj < 0.05 | Benjamini-Hochberg over the modules of each cell type |
| AD-likeness by cell type | Wilcoxon rank-sum (step 13) | donor-mean DEGAS scores | padj < 0.05 | Benjamini-Hochberg over cell types |
| Drug scores | DrugCLIP z-score against a fixed reference library (step 14) | drugs | none (ranking only) | none (no p-values) |

## Gene change in AD: the nucleus-level test (main columns)

Within each cell type, every nucleus from an AD donor is compared with every nucleus from a control donor (scanpy `rank_genes_groups`, Wilcoxon rank-sum, tie-corrected). Counts are normalized to 10,000 per nucleus and log-transformed (log1p). A gene is tested only if it is detected in at least 10% of the nuclei of either group. The log2 fold change is scanpy's (rank_genes_groups, scanpy 1.11.5, the version the pipeline pins): log2((expm1(mean_AD) + 1e-9) / (expm1(mean_control) + 1e-9)), where mean_AD and mean_control are the means over nuclei of the log1p-transformed normalized counts; it is a ratio of back-transformed mean log values, not of mean counts. The test has no covariates and treats nuclei as independent, so it does not account for which donor a nucleus came from; its p-values are much smaller than a donor-level test would give.

## Gene change in AD: the donor-level (pseudobulk) test

Raw counts are summed per donor and cell type (pseudobulk; samples summed from fewer than 10 nuclei are dropped). Genes are kept if they have at least 10 counts in at least as many samples as the smaller group. DESeq2 fits the design below and tests the condition coefficient (AD vs control, control as reference) with a Wald test. Adjusted p-values are Benjamini-Hochberg with DESeq2's independent filtering at the same level: low-count genes can be left out of the correction, and the site labels them 'filtered (low counts, not tested at FDR)'. Genes removed by the pre-filter are 'not tested'.

| Dataset | DESeq2 design | Note |
|---|---|---|
| Green et al. 2024 | `~ sex + condition` | Sex was added as a covariate, chosen before any result of this test was seen, because the two groups differ in sex balance (AD 80 female / 20 male donors, control 40 female / 27 male). |
| Mathys et al. 2019 | `~ condition` | No covariates are set in this dataset's configuration. |

## Reference gene annotation

| Dataset | Reference annotation used to count reads |
|---|---|
| Green et al. 2024 | GENCODE version 32 (Ensembl 98), the 10x Genomics GRCh38-2020-A reference (from the header of the reference GTF; Cell Ranger v6.0.0 per the dataset documentation) |
| Mathys et al. 2019 | reads aligned to the hg38 genome (GRCh38.p5, NCBI GCA_000001405.20) with Cell Ranger 2.0.0, counting reads mapped to pre-mRNA (as stated in the publication's Methods); the gene-annotation release is not stated in the publication |

## Co-expression modules

hdWGCNA groups genes whose activity rises and falls together in each cell type. Each module's eigengene is averaged per donor over that donor's nuclei and compared between AD and control donors (Wilcoxon rank-sum, two-sided); p-values are Benjamini-Hochberg adjusted across the modules of the cell type.

## AD-likeness (DEGAS)

DEGAS learns AD vs control from bulk RNA-seq of the Mount Sinai Brain Bank (MSBB; AD = Braak stage 5-6, control = Braak stage 0-2) and scores each nucleus. Patient-level cross-validation on the bulk patients (held-out ROC-AUC; 0.5 = chance, 1 = perfect):

| Dataset | Bulk patients (AD / control) | Pooled held-out ROC-AUC | Mean over folds | Range over folds |
|---|---|---|---|---|
| Green et al. 2024 | 113 / 75 | 0.792 | 0.801 (10 folds) | 0.560-0.977 |
| Mathys et al. 2019 | 113 / 75 | 0.769 | 0.787 (10 folds) | 0.571-0.966 |

Both datasets use the same MSBB patients and the same patient folds, but each dataset's model is trained together with that dataset's nuclei and uses genes selected partly from them, so the held-out predictions and AUCs differ between datasets. Each held-out fold has only 18-19 patients, so fold AUCs vary widely; the pooled AUC is the steadier figure.

Per cell type, each donor's mean nucleus score is compared between AD and control donors (Wilcoxon rank-sum); p-values are Benjamini-Hochberg adjusted across cell types.

## Drug targets

For Green et al. 2024 and Mathys et al. 2019: in each cell type, the top 10 genes higher in AD (log2 fold change > 0) in the nucleus-level test (Wilcoxon, step 11b), ranked by the Wilcoxon rank-sum z-score (largest first; ties by p-value, then gene ID). Mitochondrially encoded and ribosomal protein genes are removed before ranking, so the next genes fill the list. A gene chosen in several cell types is one target. Choosing targets never depends on whether drugs or structures exist for them.

## Drug scores

Approved drugs (step 14b): DrugCLIP scores the approved small-molecule drugs of ChEMBL against the binding pockets of each target's experimental structures (residues within 6 Å of a bound ligand), as a z-score against a fixed reference compound library; a target with no suitable structure is not scored. These are rankings, not statistical tests.

Screening hits (step 14a): GenomeScreenDB holds precomputed DrugCLIP screens of a large compound library against predicted pockets of human proteins (Jia Y et al., “Deep contrastive learning enables genome-wide virtual screening,” Science 391(6781):eads9530, 2026, [doi:10.1126/science.ads9530](https://doi.org/10.1126/science.ads9530)). For each target protein in the database, the site lists the top 10 compounds per pocket by the database's own score; these are candidate hits for discovery, not approved drugs, and not tested by us.

Column definitions: [data_dictionary.md](data_dictionary.md).

## Genes with different Ensembl IDs in the two datasets

15 gene symbols are attached to different Ensembl gene IDs in the two datasets (or, for 2 of them, to two IDs within one dataset): AGBL1, ARHGAP11B, C2orf81, CCDC39, CCDC7, CRHR1, CRIPAK, FAM169B, GGT1, HSPA14, IQCJ-SCHIP1, PIK3R3, SCO2, TBCE, TIMM10B. Ensembl IDs of the same gene change between annotation releases, which is the likely reason, but the release used for Mathys et al. 2019 is not recorded in the data files, so this is not confirmed. Each gene page is keyed by Ensembl ID and named by the current NCBI symbol; IDs that NCBI gives the same symbol share one page, with a section per ID.

## Figures, structures and links on the gene pages

Gene-page figures are drawn by this site's build from the values in the page's own table (donor-level unshrunk DESeq2 log2 fold change with ± 1.96 standard errors; nucleus-level log2 fold change on a separate axis; share of nuclei expressing the gene). Drug-target pages show the experimental structures DrugCLIP scored (Protein Data Bank, wwPDB, data CC0 1.0; entry titles and citations from the RCSB PDB Data API), each with its pocket ligand's name (PDB Chemical Component Dictionary, via the RCSB PDB Data API), how step 14 chose it (the entry's subject of investigation, else the heaviest non-additive ligand of 150-900 Da) and a class assigned by hand from the dictionary name and entry title: drug-like or studied compound, cofactor or substrate, or lipid, detergent, buffer or other additive. Step 14 scores every such pocket alike; a cofactor, lipid or additive pocket is not a drug-binding site, so drug rankings on it should be read with that in mind. On request, a 3D view with PDBe Mol* (pdbe-molstar 3.12.0, Apache-2.0; Mol*: Sehnal et al., Nucleic Acids Research 2021) and AlphaFold DB models (Jumper et al., Nature 2021; Varadi et al., Nucleic Acids Research 2022; CC BY 4.0). Every gene page links to NCBI Gene, UniProt, the Human Protein Atlas, GTEx and STRING; no content from those sites is copied.

## Data sources

The data analyzed here were generated by the following studies:

- Mathys H, et al. Single-cell transcriptomic analysis of Alzheimer's disease. Nature 2019;570(7761):332-337. [doi:10.1038/s41586-019-1195-2](https://doi.org/10.1038/s41586-019-1195-2)
- Green GS, et al. Cellular communities reveal trajectories of brain ageing and Alzheimer's disease. Nature 2024;633(8030):634-645. [doi:10.1038/s41586-024-07871-6](https://doi.org/10.1038/s41586-024-07871-6)
- Wang M, et al. The Mount Sinai cohort of large-scale genomic, transcriptomic and proteomic data in Alzheimer's disease. Scientific Data 2018;5:180185. [doi:10.1038/sdata.2018.185](https://doi.org/10.1038/sdata.2018.185)

## Attribution

The results published here are in whole or in part based on data obtained from the AD Knowledge Portal (NIH-NIA, U24AG061340).

### Study acknowledgments

**ROSMAP (Mathys et al. 2019, Synapse syn18485175; Green et al. 2024, Synapse syn31512863)**

The results published here are in whole or in part based on data obtained from The AD Knowledge Portal (https://doi.org/10.7303/9618239). Study data were provided by the Rush Alzheimer's Disease Center, Rush University Medical Center, Chicago. Data collection was supported through funding by NIA grants P30AG10161 (ROS), R01AG15819 (ROSMAP; genomics and RNAseq), R01AG17917 (MAP), R01AG30146, R01AG36042 (5hC methylation, ATACseq), RC2AG036547 (H3K9Ac), R01AG36836 (RNAseq), R01AG48015 (monocyte RNAseq) RF1AG57473 (single nucleus RNAseq), U01AG32984 (genomic and whole exome sequencing), U01AG46152 (ROSMAP AMP-AD, targeted proteomics), U01AG46161(TMT proteomics), U01AG61356 (whole genome sequencing, targeted proteomics, ROSMAP AMP-AD), P30AG072975, the Illinois Department of Public Health (ROSMAP), and the Translational Genomics Research Institute (genomic). Additional phenotypic data can be requested at www.radc.rush.edu.

snRNA-seq data: Study data were generated from postmortem brain tissue provided by the Religious Orders Study and Rush Memory and Aging Project (ROSMAP) cohort at Rush Alzheimer's Disease Center, Rush University Medical Center, Chicago. This work was funded by NIH grants U01AG061356 (De Jager/Bennett), RF1AG057473 (De Jager/Bennett), and U01AG046152 (De Jager/Bennett) as part of the AMP-AD consortium, as well as NIH grants R01AG066831 (Menon) and U01AG072572 (De Jager/St George-Hyslop).

Study DOI: [https://doi.org/10.7303/9618239](https://doi.org/10.7303/9618239)

**MSBB bulk RNA-seq (Wang et al. 2018), used by DEGAS**

The results published here are in whole or in part based on data obtained from The AD Knowledge Portal (https://doi.org/10.7303/9618238). These data were generated from postmortem brain tissue collected through the Mount Sinai VA Medical Center Brain Bank and were provided by Dr. Eric Schadt from Mount Sinai School of Medicine.

Study DOI: [https://doi.org/10.7303/9618238](https://doi.org/10.7303/9618238)

### Data availability

The ROSMAP single-nucleus RNA sequencing data from Mathys et al. 2019 (Synapse syn18485175) and Green et al. 2024 (Synapse syn31512863), and the Mount Sinai Brain Bank (MSBB) bulk RNA sequencing data, are available via the AD Knowledge Portal (https://adknowledgeportal.org). The AD Knowledge Portal is a platform for accessing data, analyses, and tools generated by the Accelerating Medicines Partnership (AMP-AD) Target Discovery Program and other National Institute on Aging (NIA)-supported programs to enable open-science practices and accelerate translational learning. The data, analyses and tools are shared early in the research cycle without a publication embargo on secondary use. Data is available for general research use according to the following requirements for data access and data attribution (https://adknowledgeportal.synapse.org/Data%20Access). For access to the data resources described here, see https://doi.org/10.7303/9618239 (ROSMAP) and https://doi.org/10.7303/9618238 (MSBB).

### Gene descriptions

Gene descriptions on the gene pages are shown exactly as fetched, never generated or rewritten: the NCBI Gene summary (RefSeq), retrieved through MyGene.info (https://mygene.info), and the Function text of the reviewed human UniProtKB/Swiss-Prot entry (The UniProt Consortium, https://www.uniprot.org; licensed CC BY 4.0). NCBI places no restrictions on the use of these data (https://www.ncbi.nlm.nih.gov/home/about/policies/). Each text names its source and retrieval date.
