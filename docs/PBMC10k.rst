PBMC10k
=======

Prepare 10x input data
----------------------

This tutorial shows the steps to analyze a typical scRNA-seq dataset.
We will use PBMC data available from the 10 Genomics support website.

The "Feature / cell matrix (filtered)" data was downloaded from 10x Genomics,
`here <https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.0/pbmc_10k_v3>`_.

.. code-block:: bash

    wget http://cf.10xgenomics.com/samples/cell-exp/3.0.0/pbmc_10k_v3/pbmc_10k_v3_filtered_feature_bc_matrix.tar.gz

When using 10x data as an input, the pipeline assumes the files are in the typical Cell Ranger directory structure.
This is not the case when downloading the processed counts from the 10x website, so we will put them into the proper format::

    mkdir -p pbmc10k/outs/
    tar xvf pbmc_10k_v3_filtered_feature_bc_matrix.tar.gz -C pbmc10k/outs/

which results in::

    $ tree pbmc10k
    pbmc10k
    └── outs
        └── filtered_feature_bc_matrix
            ├── barcodes.tsv.gz
            ├── features.tsv.gz
            └── matrix.mtx.gz

    2 directories, 3 files

So, in the nextflow config file, the tenx input channel should be set to::

    params.data.tenx.cellranger_mex = '/ddn1/vol1/staging/leuven/stg_00002/lcb/cflerin/analysis/pbmc10k/dsl2/pbmc10k/outs'


Setup the VSN-pipelines project
-------------------------------

Update the repository
*********************

Pull/update the vsn-pipelines repository cached by nextflow::

    nextflow pull vib-singlecell-nf/vsn-pipelines

Build the config file
*********************

We use a combination of profiles to build the config file:

* ``tenx``: defines the input data type
* ``single_sample_scenic``: loads the basic parameters to run the single_sample and scenic workflows
* ``scenic_use_cistarget_motifs`` and ``scenic_use_cistarget_tracks``: includes parameters to specify the location of the cistarget database files
* ``hg38``: specifies the genome

.. code-block:: bash

    nextflow config vib-singlecell-nf/vsn-pipelines \
        -profile tenx,single_sample_scenic,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,hg38,singularity \
        > pbmc10k.vsn-pipelines.complete.config

Important variables to check in the config:

* ``singularity.runOptions`` (or ``docker.runOptions``): making sure the correct volume mounts are specified (requires the user home folder (included by default in Singularity), and the location of the data).
* ``params.global.project_name`` (optional): will control the naming of the output files.
* ``params.sc.scope.tree.level_${X}`` (optional): controls the labeling of the loom file when uploaded to the SCope viewer.
* ``params.sc.scanpy.filter``
* ``params.sc.scanpy.feature_selection``
* ``params.sc.scanpy.clustering``

The complete config file used here is available at: `pbmc10k/pbmc10k.vsn-pipelines.complete.config`_.

.. _`pbmc10k/pbmc10k.vsn-pipelines.complete.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/pbmc10k/pbmc10k.vsn-pipelines.complete.config

Run the VSN-pipelines project
-----------------------------

First pass
**********

While the overall goal is to run the "best practices steps" and SCENIC together, we can first skip running SCENIC, and focus on getting the filtering and preprocessing steps correct.
Then, we can move on to run the resource-intensive SCENIC steps.
Even though we created a profile with single_sample and scenic options together, we can run just the single_sample workflow first::

    nextflow -C pbmc10k.vsn-pipelines.complete.config \
        run vib-singlecell-nf/vsn-pipelines \
        -entry single_sample

Now, the QC reports can be inspected, and the cell and gene filters can be updated by editing the config file.

Second pass
***********

Once the filters look ok, we can re-start the pipeline with the full SCENIC steps enabled::

    nextflow -C pbmc10k.vsn-pipelines.complete.config \
        run vib-singlecell-nf/vsn-pipelines \
        -entry single_sample_scenic \
        -resume


