Bageritz, J., Willnow, P., Valentini, E. et al., 2019 - Gene expression atlas of a developing tissue by single cell expression correlation analysis.
----------------------------------------------------------------------------------------------------------------------------------------------------

Some links related to the case study:

- Paper: https://www.nature.com/articles/s41592-019-0492-x
- GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE127832

Analysis of 10x Genomics Samples
********************************

BBKNN and SCENIC
++++++++++++++++

The following command was used to generate the config:

.. code:: bash

    nextflow pull vib-singlecell-nf/vsn-pipelines
    nextflow config vib-singlecell-nf/vsn-pipelines -profile \
      sra,cellranger,pcacv,bbknn,dm6,scenic,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,singularity \
      > nextflow.config


The generated config is available at the ``vsn-pipelines-examples`` GitHub repository: `bageritzj_2019/10x_bbknn_scenic.config`_.  You should provide the lines commented with "TO EDIT" with the correct information.

.. _`bageritzj_2019/10x_bbknn_scenic.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/bageritzj_2019/10x_bbknn_scenic.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run vib-singlecell-nf/vsn-pipelines \
            -entry sra_cellranger_bbknn_scenic


The resulting loom file is available at `bageritzj_2019_bbknn_scenic.loom`_, and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`bageritzj_2019_bbknn_scenic.loom`: https://cloud.aertslab.org/index.php/s/DecJoZfFxmBqLpc

Harmony and SCENIC (append mode)
++++++++++++++++++++++++++++++++

.. code:: bash

    nextflow config \
        vib-singlecell-nf/vsn-pipelines \
        -profile tenx,pcacv,harmony,scenic_append_only,singularity \
        > nextflow.config

The generated config is available at the ``vsn-pipelines`` GitHub repository: `bageritzj_2019/10x_harmony_scenic_append_only.config`_. You should update The lines commented with "TO EDIT" with the correct information.

.. _`bageritzj_2019/10x_harmony_scenic_append_only.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/bageritzj_2019/10x_harmony_scenic_append_only.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run vib-singlecell-nf/vsn-pipelines \
            -entry harmony_scenic -resume

The resulting loom file is available at `bageritzj_2019/harmony_scenic_append_only`_ and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`bageritzj_2019/harmony_scenic_append_only`: https://cloud.aertslab.org/index.php/s/NeN67EfNYA9GfP2