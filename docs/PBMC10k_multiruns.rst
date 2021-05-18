10k PBMCs with multiple SCENIC iterations
=========================================


This tutorial follows closely the following 10x PBMC case study:

* `Analysis of 10k PBMCs from a single healthy human donor <https://vsn-pipelines-examples.readthedocs.io/en/latest/PBMC10k.html>`_

However, here we focus on setting up the workflow for running SCENIC multiple times and automatically integrating the results.
Please refer to the above case study for the explanation of the remainder of the setup steps.


Setup the VSN-pipelines project
-------------------------------

Update the repository
*********************

Pull/update the vsn-pipelines repository cached by nextflow.
Here, we use the ``-r`` flag to specify the pipeline version to use::

    nextflow pull vib-singlecell-nf/vsn-pipelines -r v0.25.0

Build the config file
*********************

We use a combination of profiles to build the config file:

* ``tenx``: defines the input data type
* ``single_sample_scenic``: loads the basic parameters to run the single_sample and scenic workflows
* ``scenic_multiruns``: loads the multiruns parameters
* ``scenic_use_cistarget_motifs`` and ``scenic_use_cistarget_tracks``: includes parameters to specify the location of the cistarget database files
* ``hg38``: specifies the genome. Other options are: ``hg19``, ``dm6``, ``mm10``.
* ``singularity`` (or ``docker``): specifies container system to use to run the processes

.. code-block:: bash

    nextflow config vib-singlecell-nf/vsn-pipelines \
        -profile tenx,single_sample_scenic,scenic_multiruns,scenic_use_cistarget_motifs,scenic_use_cistarget_tracks,hg38,singularity \
        > pbmc10k.vsn-pipelines.complete.config

Important **SCENIC multi-runs parameters** to check in the config:

  * ``params.sc.scenic.numRuns``: the number of SCENIC iterations to run. Using ``10`` will give a good balance with computation time, while ``100`` will provide an exhaustive evaluation of the regulon and target gene characteristics.
  * ``params.sc.scenic.aucell.min_genes_regulon``: The threshold used for filtering the regulons based on the number of target genes. Regulons are keps only if they contain this number of genes or greater. This parameter should be considered in proportion to the number of runs (default: 5).
  * ``params.sc.scenic.aucell.min_regulon_gene_occurrence``: The threshold used for filtering the genes bases on their occurrence. Target genes that occur this many times or less across all of the runs are discarded. This parameter should be considered in proportion to the number of runs (default: 5).

Specifying compute resource usage in the config:

* The global executor (``process.executor``) is set to ``local`` by default. It can be changed to ``qsub``, etc. to run specific processes as jobs. The executor parameter can be added to specific labels to run only these processes as jobs. Typically the GRN step should be submitted as a job (``compute_resources__scenic_grn``), especially for the multi-runs approach.
* The number of cpus and memory usage can be adjusted for each label.

The complete config file used here is available at: `pbmc10k/pbmc10k.vsn-pipelines.scenic-multiruns.complete.config`_.

.. _`pbmc10k/pbmc10k.vsn-pipelines.scenic-multiruns.complete.config`: https://github.com/vib-singlecell-nf/vsn-pipelines-examples/blob/master/pbmc10k/pbmc10k.vsn-pipelines.scenic-multiruns.complete.config

Run the VSN-pipelines project
-----------------------------

Testing the settings (optional)
*******************************

With the SCENIC multi-runs approach, it is highly recommended to test the approach with ``params.sc.scenic.numRuns`` set to a small number (e.g. ``2``).

It may also be useful to use a small test dataset for this purpose::

    wget https://raw.githubusercontent.com/aertslab/SCENICprotocol/master/example/sample_data_small.tar.gz
    tar xzvf sample_data_small.tar.gz

This can be loaded in the config by temporarily replacing the input data with::

    data {
        tenx {
            cellranger_mex = 'sample_data/outs'
        }
    }

As this is a small dataset and the ``numRuns`` parameter is set to a small number, also change these settings::

    scanpy {
            filter {
                cellFilterMinNGenes = 1
            }
            neighborhood_graph {
                nPcs = 2
            }
            dim_reduction {
                pca {
                    nComps = 2
                }
            }
        }
        scenic {
            aucell {
                min_genes_regulon = 0
                min_regulon_gene_occurrence = 0
            }
        }


First pass
**********

As in the original PBMC10k tutorial, we can first run without the SCENIC steps to get the filtering and other parameters set correctly.
Re-run the pipeline as many times as needed (with ``resume`` to skip alread-completed steps) to select the proper filters::

    nextflow -C pbmc10k.vsn-pipelines.complete.config \
        run vib-singlecell-nf/vsn-pipelines \
        -entry single_sample \
        -r v0.25.0 -resume


Second pass
***********

With the parameters set, the full multi-runs workflow can be run::

    nextflow -C pbmc10k.vsn-pipelines.complete.config \
        run vib-singlecell-nf/vsn-pipelines \
        -entry single_sample_scenic \
        -r v0.25.0 -resume

This can take a long time to run, depending on the number of iterations used.


Results
-------

Once the pipeline is complete, the output will be the following files (display truncated)::

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
            │   ├── run_1
            │   │   └── sample_data__run_1__adj.tsv
            │   └── run_2
            │       └── sample_data__run_2__adj.tsv
            ├── aucell
            │   ├── run_1
            │   │   └── sample_data__run_1__auc_mtf.loom
            │   └── run_2
            │       └── sample_data__run_2__auc_mtf.loom
            ├── cistarget
            │   ├── run_1
            │   │   └── sample_data__run_1__reg_mtf.csv.gz
            │   └── run_2
            │       └── sample_data__run_2__reg_mtf.csv.gz
            ├── multi_runs_aucell
            │   └── multi_runs_regulons_auc_mtf.tsv
            ├── multi_runs_cistarget
            │   ├── multi_runs_features_mtf.csv.gz
            │   └── multi_runs_regulons_mtf.pkl.gz
            ├── multi_runs_looms
            │   └── multi_runs_regulons_auc_mtf.loom
            ├── multi_runs_regulons_mtf
            │   ├── BRF1(+).tsv
            │   ├── FOS(+).tsv
            │   └── regulons.tsv
            ├── notebooks
            │   ├── SCENIC_report.html
            │   └── SCENIC_report.ipynb
            └── SCENIC_SCope_output.loom


The final output is similar to that of a SCENIC pipeline with a single iteration, with the exception of some additional files being stored in ``out/scenic/<sampleId>``:

* GRN, cisTarget, and AUCell outputs for each run/iteration in their respective directories.
* In the ``out/scenic/<sampleId>/multi_runs_regulons_mtf/``  directory (and optionally ``multi_runs_regulons_trk`` if track databases were used):

  * ``regulons.tsv``: Contains a list of all regulons found, along with their occurrence (count) across all SCENIC iterations.
  * A file for each regulon (e.g. ``BRF1(+).tsv``). Each file contains two columns: 1) the target gene name, and 2) the number of times that gene occurred across all SCENIC iterations.


The final SCENIC output is packaged into a loom file, which includes the results of the multi-runs analysis.
This can be found at ``out/scenic/<sampleId>/SCENIC_SCope_output.loom``, and is ready to be uploaded to a `SCope <http://scope.aertslab.org/>`_ session.

