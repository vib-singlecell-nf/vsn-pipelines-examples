Kurmangaliyev et al., 2019 - Modular transcriptional programs separately define axon and dendrite connectivity
--------------------------------------------------------------------------------------------------------------

Some links related to the case study:

- Paper: https://elifesciences.org/articles/50822
- GEO: https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE126139

Analysis of 10x Genomics Samples
********************************

BBKNN and SCENIC
++++++++++++++++

The following command was used to generate the config:

.. code:: bash

    nextflow pull vib-singlecell-nf/vsn-pipelines
    nextflow config vib-singlecell-nf/vsn-pipelines \
        -profile sra,cellranger,pcacv,bbknn,dm6,scenic,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,singularity \
      > nextflow.config


The generated config is available at the ``vsn-pipelines-examples`` GitHub repository: `kurmangaliyevyz_2019/10x_bbknn_scenic.config`_.  You should provide the lines commented with "TO EDIT" with the correct information.

.. _`kurmangaliyevyz_2019/10x_bbknn_scenic.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/kurmangaliyevyz_2019/10x_bbknn_scenic.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run vib-singlecell-nf/vsn-pipelines \
            -entry sra_cellranger_bbknn_scenic


The resulting loom file is available at `kurmangaliyevyz_2019_bbknn_scenic.loom`_, and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`kurmangaliyevyz_2019_bbknn_scenic.loom`: https://cloud.aertslab.org/index.php/s/ygtZgFM29QdzFjT

Harmony and SCENIC (append mode)
++++++++++++++++++++++++++++++++

.. code:: bash

    nextflow config \
        vib-singlecell-nf/vsn-pipelines \
        -profile tenx,pcacv,harmony,scenic_append_only,singularity \
        > nextflow.config

The generated config is available at the ``vsn-pipelines`` GitHub repository: `kurmangaliyevyz_2019/10x_harmony_scenic_append_only.config`_. You should update The lines commented with "TO EDIT" with the correct information.

.. _`kurmangaliyevyz_2019/10x_harmony_scenic_append_only.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/kurmangaliyevyz_2019/10x_harmony_scenic_append_only.config

To start the pipeline, run the following command:

.. code:: bash

    nextflow \
        -C nextflow.config \
        run vib-singlecell-nf/vsn-pipelines \
            -entry harmony_scenic -resume

The resulting loom file is available at `kurmangaliyevyz_2019/harmony_scenic_append_only`_ and is ready to be explored in `SCope <http://scope.aertslab.org/>`_.

.. _`kurmangaliyevyz_2019/harmony_scenic_append_only`: https://cloud.aertslab.org/index.php/s/Jy4adwppT6wwoRi