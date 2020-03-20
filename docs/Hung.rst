Hung, R.-J. et al., 2019 - A cell atlas of the adult Drosophila midgut
----------------------------------------------------------------------

Some links related to the case study:

- Paper: https://www.pnas.org/content/117/3/1514.abstract
- GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE120537

Analysis of 10x Genomics Samples
********************************

BBKNN and SCENIC
++++++++++++++++

The following command was used to generate the config:

.. code:: bash
    nextflow pull vib-singlecell-nf/vsn-pipelines -r v0.14.2
    nextflow config vib-singlecell-nf/vsn-pipelines -profile \
      sra,cellranger,pcacv,bbknn,dm6,scenic,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,singularity \
      > nextflow.config


The generated config is available at the ``vsn-pipelines-examples`` GitHub repository: `hungr_2019/10x_bbknn_scenic.config`_.  You should provide the lines commented with "TO EDIT" with the correct information.

.. _`hungr_2019/10x_bbknn_scenic.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/hungr_2019/10x_bbknn_scenic.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run vib-singlecell-nf/vsn-pipelines \
            -entry sra_cellranger_bbknn_scenic


The resulting loom file is available at `hungr_2019_bbknn_scenic.loom`_, and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`hungr_2019_bbknn_scenic.loom`: https://cloud.aertslab.org/index.php/s/3MwXaecjSQqEGHD
