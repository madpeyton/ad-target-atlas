# Data dictionary

Written by build_site_data.py v1.12. Main gene test: nucleus-level test (Wilcoxon, step 11b) (Green et al. 2024, Mathys et al. 2019). Tests and thresholds: [methods.md](methods.md).

## de_genes (one row per gene and cell type; every gene tested by the main test or by the donor-level test)

| Column | Meaning | Produced by |
|---|---|---|
| `cell_type` | cell type label used by the dataset's authors or the pipeline's annotation | step 09 annotation |
| `gene_symbol, gene_id` | gene symbol as the dataset names it, and Ensembl gene ID | step 01 (data loading) |
| `direction` | higher in AD / lower in AD; 'unclear' when the fold change and the test statistic disagree in sign | main gene test |
| `strong_evidence` | true when padj < 0.05 in the main test | main gene test |
| `fdr_tested` | false only when the main test is DESeq2 and the gene was left out of the FDR correction by independent filtering | main gene test |
| `padj` | Benjamini-Hochberg adjusted p-value | main gene test |
| `effect` | log2 fold change, AD vs control | main gene test |
| `stat` | test statistic (Wilcoxon z-score, or DESeq2 Wald statistic) | main gene test |
| `pct_ad, pct_control` | share of the cell type's nuclei from AD (control) donors with at least one UMI count of the gene (nucleus-level test only) | nucleus-level test (Wilcoxon, step 11b) |
| `donor_level` | whether the donor-level DESeq2 Wald statistic points the same way as the main test (nucleus-level main test only); 'not tested' if the gene is absent from step 11 | donor-level test (pseudobulk DESeq2, step 11) |
| `donor_test` | 'significant' (padj < 0.05), 'not significant', 'filtered (low counts, not tested at FDR)' (DESeq2 independent filtering left padj empty) or 'not tested' (gene removed by step 11's pre-filter) | donor-level test (pseudobulk DESeq2, step 11) |
| `nucleus_tested` | false for genes tested only by the donor-level test (nucleus-level fields then read 'not tested (detected in < 10% of nuclei)' and carry no numbers; donor_level then gives the donor-level direction itself) | nucleus-level test (Wilcoxon, step 11b) detection filter |

## modules (one row per co-expression module and cell type)

| Column | Meaning | Produced by |
|---|---|---|
| `module, n_genes` | hdWGCNA module (named by color; grey = no module, left out) and its size | step 12 hdWGCNA |
| `direction, median_difference` | difference in donor medians of the module eigengene, AD minus control | step 12 Wilcoxon rank-sum across donors |
| `padj, strong_evidence` | Benjamini-Hochberg across modules within the cell type; strong = padj < 0.05 | step 12 Wilcoxon rank-sum across donors |
| `n_donors_AD, n_donors_control` | donors in the test | step 12 |

## degas.celltypes (one row per cell type)

| Column | Meaning | Produced by |
|---|---|---|
| `n_nuclei, median_score, q25, q75` | DEGAS AD-likeness score over all nuclei of the cell type | step 13 DEGAS |
| `donor_median_AD, donor_median_control, donor_difference, direction, padj` | per-donor mean score compared between AD and control donors; Benjamini-Hochberg across cell types | step 13 Wilcoxon rank-sum across donors |
| `n_donors_AD, n_donors_control` | donors in the test | step 13 |

## downloads/results.csv and results.json (one row per dataset, cell type and gene)

| Column | Meaning | Produced by |
|---|---|---|
| `dataset` | dataset key (green = Green et al. 2024; mathys2019 = Mathys et al. 2019) | - |
| `cell_type, gene_symbol, gene_id` | cell type label, gene symbol as used by the dataset, Ensembl gene ID | - |
| `gene_symbol_current` | current NCBI symbol of the Ensembl ID (the gene page's name; the dataset symbol when NCBI gives no single symbol) | MyGene.info (fetch_gene_annotations.py) |
| `nucleus_tested` | false if the gene was detected in < 10% of the nuclei of both groups (not tested) | nucleus-level test (Wilcoxon, step 11b) |
| `nucleus_significant` | nucleus_padj < 0.05 (empty when not tested) | nucleus-level test (Wilcoxon, step 11b) |
| `nucleus_direction` | higher in AD / lower in AD / unclear (log2 fold change and z-score disagree in sign) / not tested | nucleus-level test (Wilcoxon, step 11b) |
| `nucleus_padj` | Benjamini-Hochberg adjusted p-value over the tested genes of the cell type | nucleus-level test (Wilcoxon, step 11b) |
| `nucleus_log2fc` | scanpy's log2 fold change: log2((expm1(mean_AD) + 1e-9) / (expm1(mean_control) + 1e-9)), means over nuclei of log1p-transformed normalized counts | nucleus-level test (Wilcoxon, step 11b) |
| `nucleus_score` | Wilcoxon rank-sum z-score (tie-corrected) | nucleus-level test (Wilcoxon, step 11b) |
| `pct_ad, pct_control` | share of the cell type's nuclei from AD (control) donors with at least one UMI count | nucleus-level test (Wilcoxon, step 11b) |
| `donor_test` | significant (donor_padj < 0.05) / not significant / filtered (low counts, not tested at FDR) / not tested (removed by the pre-filter) | donor-level test (pseudobulk DESeq2, step 11) |
| `donor_padj` | Benjamini-Hochberg adjusted p-value with DESeq2 independent filtering (empty when filtered) | donor-level test (pseudobulk DESeq2, step 11) |
| `donor_log2fc, donor_lfcse` | unshrunk log2 fold change, AD vs control, and its standard error | donor-level test (pseudobulk DESeq2, step 11) |
| `donor_log2fc_shrunk` | apeglm-shrunk log2 fold change (used to pick drug targets when de_source is pseudobulk) | donor-level test (pseudobulk DESeq2, step 11) |

## Gene pages: table "Change in AD by cell type" (genes/<SYMBOL>/)

| Column | Meaning | Produced by |
|---|---|---|
| Page name (genes/<SYMBOL>/) | one page per gene (Ensembl ID), named by the current NCBI symbol (the dataset symbol when NCBI gives no single symbol); every other symbol a dataset uses for the gene has a page that redirects to it | MyGene.info (fetch_gene_annotations.py) |
| Gene description (top of the page) | NCBI Gene summary (RefSeq) and UniProtKB/Swiss-Prot Function text, as fetched (description in the gene JSON), with source, retrieval date and links; 'No curated summary available.' when neither exists | fetch_gene_annotations.py (MyGene.info, UniProt REST API) |
| Nucleus-level test | significant / not significant / not tested (detected in < 10% of nuclei) | nucleus-level test (Wilcoxon, step 11b) |
| Nucleus-level direction | nucleus_direction | nucleus-level test (Wilcoxon, step 11b) |
| Nucleus-level log2FC, Nucleus-level adjusted p | nucleus_log2fc, nucleus_padj | nucleus-level test (Wilcoxon, step 11b) |
| % nuclei expressing, AD / control | pct_ad / pct_control | nucleus-level test (Wilcoxon, step 11b) |
| Donor-level (pseudobulk) test | donor_test | donor-level test (pseudobulk DESeq2, step 11) |
| Donor-level log2FC (SE) | the UNSHRUNK DESeq2 log2 fold change, AD vs control (donor_log2fc), followed by its standard error in parentheses (donor_lfcse); not shrunk, not a confidence interval | donor-level test (pseudobulk DESeq2, step 11) |
| Donor-level adjusted p | donor_padj (empty when filtered) | donor-level test (pseudobulk DESeq2, step 11) |
