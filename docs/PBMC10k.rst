10k PBMCs (10x Genomics)
========================


This tutorial shows the steps to analyze a typical scRNA-seq dataset with a single sample.
We will use PBMC data available from the 10x Genomics support website.
The same dataset was used in the DSL1 version of this pipeline, described in the 
`SCENIC protocol <https://github.com/aertslab/SCENICprotocol>`_ tutorial 
(`here <https://github.com/aertslab/SCENICprotocol/blob/master/notebooks/PBMC10k_SCENIC-protocol-CLI.ipynb>`_).

Prepare 10x input data
----------------------

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

So, in the nextflow config file, generated in the following step, the tenx input channel should point to the ``outs`` folder.
For example::

    params.data.tenx.cellranger_mex = '/home/cflerin/analysis/pbmc10k/dsl2_0.19.0/pbmc10k/outs'


Setup the VSN-pipelines project
-------------------------------

Update the repository
*********************

Pull/update the vsn-pipelines repository cached by nextflow.
Here, we use the ``-r`` flag to specify the pipeline version to use::

    nextflow pull vib-singlecell-nf/vsn-pipelines -r v0.19.0

Build the config file
*********************

We use a combination of profiles to build the config file:

* ``tenx``: defines the input data type
* ``single_sample_scenic``: loads the basic parameters to run the single_sample and scenic workflows
* ``scenic_use_cistarget_motifs`` and ``scenic_use_cistarget_tracks``: includes parameters to specify the location of the cistarget database files
* ``hg38``: specifies the genome
* ``singularity`` (or ``docker``): specifies container system to use to run the processes

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
        -entry single_sample \
        -r v0.19.0

Now, the QC reports can be inspected (see ``out/notebooks/intermediate/pbmc10k.SC_QC_filtering_report.html``, either the original ipynb, or the converted html file).
The cell and gene filters can be updated by editing the config file.
For example, the filters used here are::

    params {
        sc {
            scanpy {
                filter = {
                    cellFilterMinNGenes = 200
                    cellFilterMaxNGenes = 4000
                    cellFilterMaxPercentMito = 0.15
                    geneFilterMinNCells = 3
                }
            }
        }
    }


Second pass
***********

Once the cell and gene filters look ok, we can re-start the pipeline with the full SCENIC steps enabled.
This will re-run the filtering steps and all following steps that depend on the filtering output, while skipping the initial conversion, etc. when the ``-resume`` option is used::

    nextflow -C pbmc10k.vsn-pipelines.complete.config \
        run vib-singlecell-nf/vsn-pipelines \
        -entry single_sample_scenic \
        -r v0.19.0 -resume


Results
-------

Once the pipeline is complete (approximately 2 hours on a HPC system using 15 processes for the SCENIC GRN step), the output will be the following files (display truncated)::

    $ tree out
    out/
    ├── data
    │   ├── intermediate
    │   │   └── [...]
    │   └── pbmc10k.PBMC10k_DSL2.single_sample.output.h5ad
    ├── loom
    │   ├── pbmc10k.SCENIC_SCope_output.loom
    │   └── pbmc10k.SCope_output.loom
    ├── nextflow_reports
    │   ├── execution_report.html
    │   ├── execution_timeline.html
    │   ├── execution_trace.txt
    │   └── pipeline_dag.dot
    ├── notebooks
    │   ├── intermediate
    │   ├── pbmc10k.merged_report.html
    │   ├── pbmc10k.merged_report.ipynb
    │   ├── pbmc10k.merged_report.louvain_0.4.html
    │   ├── pbmc10k.merged_report.louvain_0.4.ipynb
    │   ├── pbmc10k.merged_report.louvain_0.8.html
    │   └── pbmc10k.merged_report.louvain_0.8.ipynb
    └── scenic
        └── pbmc10k
            ├── arboreto_with_multiprocessing
            │   ├── pbmc10k__adj.tsv
            │   └── pbmc10k.filtered.loom
            ├── aucell
            │   ├── pbmc10k__auc_mtf.loom
            │   ├── pbmc10k__auc_trk.loom
            │   └── pbmc10k.filtered.loom
            ├── cistarget
            │   ├── pbmc10k.filtered.loom
            │   ├── pbmc10k__reg_mtf.csv
            │   └── pbmc10k__reg_trk.csv
            ├── notebooks
            │   ├── SCENIC_report.html
            │   └── SCENIC_report.ipynb
            ├── SCENIC_output.loom
            └── SCENIC_SCope_output.loom


The final SCENIC output is packaged into a loom file, which includes the results of the parallel expression analysis (based on highly variable genes).
This can be found at ``out/loom/pbmc10k.SCENIC_SCope_output.loom``, and is ready to be uploaded to a `SCope <http://scope.aertslab.org/>`_ session.
The output loom file from this analysis can be found on the `SCENIC protocol SCope session <http://scope.aertslab.org/#/Protocol_Cases/Protocol_Cases/welcome>`_.

Also included is ``out/data/pbmc10k.PBMC10k_DSL2.single_sample.output.h5ad``, an anndata file generated by the Scanpy section of the pipeline, including the results of the expression analysis (but not results from SCENIC).
