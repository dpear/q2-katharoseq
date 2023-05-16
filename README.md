# KatharoSeq

An implementation of the KatharoSeq protocol, originally defined in [Minich et al 2018 mSystems](https://journals.asm.org/doi/10.1128/mSystems.00218-17).

## Installation

Installation assumes a working QIIME 2 environment with a minimum version of 2021.8. Details on installing QIIME 2 can be found [here](https://docs.qiime2.org/2021.11/install/).

```
git clone https://github.com/biocore/q2-katharoseq.git
cd q2-katharoseq
pip install -e .
```

## Use

Computation assumes that the user has classified their 16S features against SILVA, and that the `FeatureTable[Frequency]` has been collapsed to the genus level. Please see the [`q2-feature-classifier`](https://docs.qiime2.org/2022.2/plugins/available/feature-classifier/classify-sklearn/) for detail on how to perform taxonomy classification, and the [`q2-taxa`](https://docs.qiime2.org/2022.2/plugins/available/taxa/collapse/) plugin for information on collapsing to a taxonomic level. If you need more information on how to process your data, please refer to one of the relevant tutorials that can be found [here](https://docs.qiime2.org/2022.2/tutorials/). For these examples, data from the Fish Microbiome Project (FMP): [Fish microbiomes 101: disentangling the rules governing marine fish mucosal microbiomes across 101 species](https://www.biorxiv.org/content/10.1101/2022.03.07.483203v1) paper will be used, and can be found in the `example` folder.


## Read Count Threshold

In order to obtain a read count threshold, computation of a minimum read count threshold can be performed with the
`read-count-threshold` plugin action. Test data can be found under the `example` folder.

```
qiime katharoseq read-count-threshold \
    --i-table example/fmp_collapsed_table.qza \
    --p-threshold 90 \
    --p-positive-control-value control \
    --m-positive-control-column-file example/fmp_metadata.tsv \
    --m-positive-control-column-column control_rct \
    --m-cell-count-column-file example/fmp_metadata.tsv \
    --m-cell-count-column-column control_cell_into_extraction \
    --p-control classic \
    --o-visualization result_fmp_example.qzv
```
### Description of parameters:
If you type `qiime katharoseq read-count-threshold` into the command line, the following information will be displayed.

- `table`:  A qiime FeatureTable collapsed to the genus level (level 6) that contains the control samples.
- `threshold`: Threshold to use in calculating minimum frequency. Must be int in 0 .. 100.
- `positive-control-value`: The value in the control column that demarks which samples are the positive controls.
- `positive-control-column-file`: .tsv metadata filename
- `positive-control-column-column`: The column name for the column in the sample metadata that describes which samples are and are not controls.
- 
- `cell-count-column-file`: .tsv metadata filename (probably the same as `positive-control-column-file`)
- `cell-count-column-column`: Column name in metadata that contains cell count information
- `control`: Community of controls used. IMPORTANT: please see the below section on possible control communities. If there is a species in your control community that is not in any of the provided builtin options, please contact the authors of this tool. Custom control communities are not yet supported but will be in the future.
- `visualization`: Path name of output visualization file


## Estimating Biomass

 Estimate the biomass of samples using KatharoSeq controls. After obtaining a read count threshold using the action above, use the same metadata and collapsed table as input. The `--p-pcr-template-vol` and `--p-dna-template-vol` values are numeric values that should come from your experimental procedures.
 
```
qiime katharoseq estimating-biomass \
    --i-table example/fmp_collapsed_table.qza \
    --m-control-cell-extraction-file example/fmp_metadata.tsv \
    --m-control-cell-extraction-column control_cell_into_extraction \
    --p-min-total-reads 1315 \
    --p-positive-control-value control \
    --m-positive-control-column-file example/fmp_metadata.tsv \
    --m-positive-control-column-column control_rct \
    --p-pcr-template-vol 5 \
    --p-dna-extract-vol 60 \
    --m-extraction-mass-g-column extraction_mass_g \
    --m-extraction-mass-g-file example/fmp_metadata.tsv \
    --o-estimated-biomass estimated_biomass_fmp_rct
```

To get a description of parameters type `qiime katharoseq estimating-biomass` into the command line.

## Biomass Plot

Finally in order to visualize the results from `estimating-biomass`, run `biomass-plot`.

```
qiime katharoseq biomass-plot \
    --i-table example/fmp_collapsed_table.qza \
    --m-control-cell-extraction-file example/fmp_metadata_mod.tsv \
    --m-control-cell-extraction-column control_cell_into_extraction \
    --p-min-total-reads 1315 \
    --p-positive-control-value control \
    --m-positive-control-column-file example/fmp_metadata_mod.tsv \
    --m-positive-control-column-column control_rct \
    --o-visualization biomass_plot_fmp
```

To get a description of parameters type `qiime katharoseq biomass-plot` into the command line.

## Control Communities for read-count-threshold

If your control community is a subset of any of these, use that category, however if your control community contains speciese that are not in that group, or if they are different altogether, please contact dsperry@ucsd.edu. Support for custom controls will be included soon.

`atcc`:
- d__Bacteria;p__Firmicutes;c__Clostridia;o__Clostridiales;f__Clostridiaceae;g__Clostridium,
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Enterobacterales;f__Enterobacteriaceae;g__,
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Enterobacterales;f__Enterobacteriaceae;g__Escherichia-Shigella,
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Staphylococcales;f__Staphylococcaceae;g__Staphylococcus

`zymobiomics`:
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Lactobacillales;f__Listeriaceae;g__Listeria,
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Pseudomonadales;f__Pseudomonadaceae;g__Pseudomonas,
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Bacillales;f__Bacillaceae;g__Bacillus,
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Enterobacterales;f__Enterobacteriaceae;__,
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Enterobacterales;f__Enterobacteriaceae;g__Escherichia-Shigella,
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Lactobacillales;f__Lactobacillaceae;g__Lactobacillus,
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Lactobacillales;f__Enterococcaceae;g__Enterococcus,
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Staphylococcales;f__Staphylococcaceae;g__Staphylococcus

`classic`:
- d__Bacteria;p__Firmicutes;c__Bacilli;o__Bacillales;f__Bacillaceae;g__Bacillus,
- d__Bacteria;p__Proteobacteria;c__Alphaproteobacteria;o__Rhodobacterales;f__Rhodobacteraceae;g__Paracoccus

`single`:
- d__Bacteria;p__Proteobacteria;c__Gammaproteobacteria;o__Burkholderiales;f__Comamonadaceae;g__Variovorax
