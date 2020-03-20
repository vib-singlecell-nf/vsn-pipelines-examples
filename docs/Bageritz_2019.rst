Bageritz, J., Willnow, P., Valentini, E. et al., 2019 - Gene expression atlas of a developing tissue by single cell expression correlation analysis.
----------------------------------------------------------------------------------------------------------------------------------------------------

Some links related to the case study:

- Paper: https://www.nature.com/articles/s41592-019-0492-x
- GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE127832

Analysis of 10x Genomics Samples
********************************

The following command was used to generate the config:

.. code:: bash
    nextflow pull vib-singlecell-nf/vsn-pipelines -r v0.14.2
    nextflow config vib-singlecell-nf/vsn-pipelines -profile \
      sra,cellranger,pcacv,bbknn,dm6,scenic,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,singularity \
      > nextflow.config


The generated config is available at the ``vsn-pipelines-examples`` GitHub repository: `bageritz_2019/10x_bbknn_scenic.config`_.  You should provide the lines commented with "TO EDIT" with the correct information.

.. _`bageritz_2019/10x_bbknn_scenic.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/bageritz_2019/10x_bbknn_scenic.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run ~/vib-singlecell-nf/vsn-pipelines \
            -entry sra_cellranger_bbknn_scenic


The resulting loom file is available at `bageritz_2019_bbknn_scenic.loom`_, and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`bageritz_2019_bbknn_scenic.loom`: https://cloud.aertslab.org/index.php/s/tCiKLHEwdDsFftq

