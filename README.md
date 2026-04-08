# Artifacts

The following is a description of artifact for the paper *"A Comparative Study of Productivity and Performance of Four Programming Languages"* under review.

## Table of Contents
- [Content of the repository](#content-of-the-repository)
  - [Benchmark Algorithms](#benchmark-algorithms)
  - [Computing System](#computing-system)
  - [Conda Environments](#conda-environments)
  - [Workflow](#workflow)
  - [SBATCH Scripts](#sbatch-scripts)
  - [Performance Results](#performance-results)
  - [Python Preliminary Experiments](#python-preliminary-experiments)
  - [Coding Productivity Metrics](#coding-productivity-metrics)
  - [Code Lines DIFF](#code-lines-diff)
  - [Figures and Table](#figures-and-table)
- [Snakemake 101](#snakemake-101)
- [References](#references)

## Content of the repository [](#content-of-the-repository)

### [Benchmark Algorithms](benchmark_scripts) [](#benchmark-algorithms)

We implemented 4 different algorithms (Connected Components, Merge Sort, Pi-Approximation, and N-Body Simulation) in four programming languages (Python, Julia, C++, and DaphneDSL). The first algorithm requires a matrix as input, the other three not.

Each of the folder contains the code for the sequential (Seq), local parallel (Par) and distributed (`-mpi`, Dist) version of each algorithm.

In each folder, there is one folder per language (`py`, `jl`, `cpp`, and `daph`).   
*Note*: There is no `daph` folder for the Par and Dist version, as those are the same as the sequential one.

### [Computing System](computing-system)  [](#computing-system)
- [Topology](computing-system/topology.png) view (generated with ```lstopo```) of a typical compute node (2x AMD EPYC 7H12 64-Core Processor) on Vega EuroHPC.  
- Hardware and Software [specifications](computing-system/collect_environment.txt) of the computing system used for the experiments collected with this [script](computing-system/collect_environment.sh). Vega is managed with Slurm.
- Software stack used for the experiments:  
    * Conda [1] v23.7.2 for package end environmental management. 
    * Snakemake [2] v9.9.0 workflow management system is used to run the entire workflow
    * Multimetric [3] v2.2.2 to collect the productivity metrics.
    * Containerization tool Singularity [4] PRO v4.1.6-1.el8 for software containers.

### [Conda Environments](conda_envs) [](#conda-environments)

- [`code_metrics.yml`](conda_envs/code_metrics.yml)
- [`snakemake.yml`](conda_envs/snakemake.yml)
- [`plotting.yml`](conda_envs/plotting.yml)  

To use them:  
```console
module load Anaconda
conda env create -f snakemake.yml 
conda activate snakemake
```

### [Workflow](workflow) [](#workflow)

This folder contains the heavy lifting of the experiments.

The workflow is managed by [Snakemake](https://snakemake.readthedocs.io/en/stable/). You can execute `Snakemake` from the provided Conda environment [`snakemake.yml`](conda_envs/snakemake.yml):

#### `.smk` files

- `matrices.smk`: contains the details about the matrices used in the experiments (i.e., where to download, and some metadata)

  - The rules of this file download and decompress the matrices, then set up the metadata file (`.mtx.meta`) and fix the `.mtx` file so that all the linear algebra libraries used in the experiments read the matrices the same way.

- `doe.smk`: file containing the configuration about the **D**esign **O**f **E**xperiments. This is where to change the important parameters. Most of the others `.smk` files read this file.

- `build.smk`: file containing the steps to build DAPHNE (download source code (commit in `doe.smk`), download singularity image, and compile)

- `experiments_with_matrices.smk`: file containing the rules for an experiment comparing the scaling ability of threads on a single node for algorithms with matrix input.

- `experiments_with_matrices_mpi_local.smk`: file containing the rules for an experiment comparing the scaling ability of `mpi` processes on a single node for algorithms with matrix input.

- `experiments_with_matrices_mpi_scale_nodes.smk`: file containing the rules for an experiment comparing the scaling ability of `mpi` processes on a multiple node for algorithms with matrix input.

- `experiments.smk`: file containing the rules for an experiment comparing the scaling ability of threads on a single node.

- `experiments_mpi_local.smk`: file containing the rules for an experiment comparing the scaling ability of `mpi` processes on a single node.

- `experiments_mpi_scale_nodes.smk`: file containing the rules for an experiment comparing the scaling ability of `mpi` processes on a multiple node.

### [SBATCH Scripts](sbatch_scripts) [](#sbatch-scripts)

This folder contains the `sbatch` scripts used to execute the different languages in the different context (sequential, parallel, distributed).

The interface is as follow:

1. For Connected Components with matrix input:  
    a) For sequential and local parallel:
    ```console
    sbatch ./sbatch_scripts/run_vega_{LANG}.sh {NUM_THREADS} {SRC_FILE_PATH} {MATRIX_PATH} {MATRIX_SIZE} {OUTPUT_FILE}
    ```  
    b) For distributed with MPI:
    ```console
    sbatch ./sbatch_scripts/run_vega_{LANG}_mpi.sh {NUM_THREADS} {SRC_FILE_PATH} {MATRIX_PATH} {MATRIX_SIZE} {OUTPUT_FILE}
    ```
2. For Merge Sort, Pi-Approximation, and N-Body Simulation:  
    a) For sequential and local parallel:
    ```console
    sbatch ./sbatch_scripts/nomat_run_vega_{LANG}.sh {NUM_THREADS} {SRC_FILE_PATH} {OUTPUT_FILE} {ARGS}
    ```  
    b) For distributed with MPI:
    ```console
    sbatch ./sbatch_scripts/nomat_run_vega_{LANG}_mpi.sh {NUM_THREADS} {SRC_FILE_PATH} {OUTPUT_FILE} {ARGS}
    ```    

where:
- `{LANG}` is the language short name (`cpp`, `daph`, `jl`, `py`)

- `{NUM_THREADS}` is the number of threads to use.

- `{SRC_FILE_PATH}` is the path to the source code file containing the benchmark

- `{MATRIX_PATH}` is the path to the `.mtx` file

- `{MATRIX_SIZE}` is the size of the matrix (i.e., the number of columns or rows (as we consider only square matrices, those are the same))

- `{OUTPUT_FILE}` is the path where to store the result of the execution

- `{ARGS}` are the arguments for each algorithm  
    * Merge Sort: ```array_size```, ```threshold```, for Par version additionally ```num_threads```
    * Pi-Approximation: ```num_intervals```, for Par version additionally ```num_threads```
    * N-Body Simulation: ```num_particles```, ```num_timesteps```, for Par version additionally ```num_threads```


### [Performance Results](results) [](#performance-results)
This folder contains all experimental results.
- [`Connected Components`](results/connected_components/)
- [`Merge Sort`](results/mergesort/)
- [`Pi-Approximation`](results/pi_approx/)
- [`N-Body Simulation`](results/nbody/)

Each folder contains:
- [`Sequential (seq)`](results/mergesort/seq/)
- [`Local parallel (par)`](results/mergesort/par/)
- [`Local MPI (mpi_local)`](results/mergesort/mpi_local/)
- [`MPI on multiple nodes (mpi_scale_nodes)`](results/mergesort/mpi_scale_nodes/)

The `.dat` files contain the end-to-end and the computation time together with the algorithm result for correctness check:  
`/{MATRIX}/{LANG}/{DAPHNE-SCHEDULING}/{PARALLELISM}/{REP}.dat`

where:
- `{MATRIX}` is the input matrix `ljournal-2008` or empty if no matrix input.

- `{LANG}` is the language short name (`cpp`, `daph`, `jl`, `py`).

- `{DAPHNE-SCHEDULING}` Daphne scheduling options. Empty for all other languages.

- `{PARALLELISM}` is the number of threads, mpi processes, or nodes used.

- `{REP}` is the repetition of each experiments.


### [Python Preliminary Experiments](python-pre-experiments) [](#python-preliminary-experiments)
Contains the material of our preliminary experiments with GIL-free and GIL-bound Python v3.14 with Pi-Approximation algorithm.
- [`Experiments`](python-pre-experiments/pi_experiments.sh)  
    * The script to run the experiments with and without profiling. All experimental configurations are in the script, but the actual execution requires building the GIL-free and GIL-bound Python Singularity images first (see below).
    * Using [pi_slurm.job](python-pre-experiments/pi_slurm.job) as template for the `sbatch` script to run the experiments on Vega.
    * Running the actual experiment script [pi.py](python-pre-experiments/pi.py) requires building the GIL-free and  Python Singularity image first (see below).
- [`Results-Py`](python-pre-experiments/results-py)  
    * Traces (`.json`) can not be provided as they are too large to share on this repository.
- [`Singularity Images`](python-pre-experiments/sif)  
    Resources to build the Singularity images with GIL-free and GIL-bound Python v3.14 and Python v3.12.
    *Note*: The actual images (`.sif`) are not provided as they are to large to share on this repository.


### [Coding Productivity Metrics](coding_productivity) [](#coding-productivity-metrics)

- The adapted version of [`multimetric`](coding_productivity/multimetric/) [3] which includes language support for DaphneDSL and collects the source lines of code (SLOC).
- The collected [metrics](coding_productivity/metrics/) in `.json` format.

Execute `multimetric` from within the created Conda environment from the provided [code_metrics.yml](conda_envs/code_metrics.yml)  
Build it with:
```
conda activate code-metrics
cd ./coding_productivity/multimetric
pip install -e .
```

```
 multimetric {SCRIPT_FILE} > {OUTPUT_FILE}
```

where:

- `{SCRIPT_FILE}` is the path if the script to analyze, e.g., `/benchmark_scripts/connected_components/cpp/connected_components.cpp`

- `{OUTPUT_FILE}` is the path where to store the result of the execution

To collect all coding productivity metric results, execute [collect_metrics.sh](coding_productivity/collect_metrics.sh). The resulting metrics (`.json`) are provided in the directory [metrics](coding_productivity/metrics).


### [Code Lines DIFF](plots) [](#code-lines-diff)
- [Script](plots/code-diff.sh) to compute the difference in lines of code between two versions of an algorithm  (e.g., Seq $\rightarrow$ Par version of the same algorithm and language). It uses `git diff` command to compute the difference and outputs the number of lines added and removed to go from one version to the other.
- [Differences](plots/diff_results.txt) between benchmarks scripts.  
  Reading example:  
  ```=== mergesort | jl | seq vs par ===```  
  ```39	5	../benchmark_scripts/{mergesort-seq => mergesort-par}/jl/mergesort.jl```  
  Changing the **Julia** implementation for **Merge Sort** from **sequential** (_Sep_) to **local parallel** (_Par_) version required adding **39** lines and removing **5** lines.  
  ```MISSING``` denotes combinations where no corresponding version exists.  

### [Figures and Table](plots) [](#figures-and-table)
The Jupyter notebook `plots.ipynb` is used to create the final [performance figures](plots/performance/), [Python pre-experiment figures](plots/pre-experiments/), the coding productivity metrics result Table III, and the baseline execution times Table IV. To have all dependencies, execute the notebook from [plotting.yml](conda_envs/plotting.yml)

*Note*: Profiling plots can not be reproduced without running the profiling first, as the tracing results can not be provided as they are too large to share on this repository.

## Snakemake 101 [](#snakemake-101)

**All the Snakemake commands are to be ran from the root of the repository**

To dry-run a workflow:

```console
snakemake -s {WORKFLOW_FILE}.smk -n
```

To execute a workflow with 2 parallel processes and other important flags:

- `--cores`: run workflow with parallel processes

- `--jobs`: run workflow bt submitting this many jobs at a time

- `--latency-wait`: as the data will be written on the NFS it is good practice to tell Snakemake that there might be some latency in the filesystem.

- `--keep-going`: continue to execute the workflow even if one job failed.

- `--rerun-incomplete`: rerun jobs that have been stopped in a weird state before.

```console
snakemake -s {WORKFLOW_FILE}.smk --cores 2 --jobs 20 --latency-wait 60 --keep-going --rerun-incomplete
```

Tell Snakemake to check what output files already exist to avoid repetition of experiments if not needed:

```console
snakemake -s {WORKFLOW_FILE}.smk --touch --rerun-incomplete
```

## References [](#references)
[1] Anaconda Inc. 2025. Conda. [https://anaconda.org/anaconda/conda](https://anaconda.org/anaconda/conda). Accessed August 16, 2025.  
[2] Johannes Koester. 2025. Snakemake. [https://snakemake.readthedocs.io/en/stable/](https://snakemake.readthedocs.io/en/stable/). Accessed August 16, 2025.   
[3] Konrad Weihmann. multimetric. Version 2.2.2. [https://github.com/priv-kweihmann/multimetric](https://github.com/priv-kweihmann/multimetric). Accessed July 10, 2025.  
[4] Sylabs Inc. 2024. SingularityCD. [https://docs.sylabs.io/guides/4.1/user-guide/](https://docs.sylabs.io/guides/4.1/user-guide/). Accessed August 16, 2025.  



