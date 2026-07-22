# ParEval-WF: Task-Based Parallel Code Generation

This repository extends the [ParEval](https://github.com/parallelcodefoundry/ParEval)
benchmark to **task-based parallel programming models**, demonstrated on
[**PyCOMPSs**](https://compss.bsc.es).

While the original ParEval evaluates LLM-generated **C++ kernel** parallelism
(OpenMP, MPI, Kokkos, CUDA), this fork evaluates whether LLMs can generate
correct and efficient **task-based** parallel code, where parallelism is
expressed across functions rather than through in-function annotations.

## What this fork adds

- A **Python execution backend** for evaluating LLM-generated task-based code
  (`drivers/python/`), currently targeting PyCOMPSs but extensible to other
  Python-based models.
- A **multi-tier prompt set** ranging from bare kernel prompts to
  infrastructure-aware workflow prompts. This work evaluates Tiers 0 and 1
  (`prompts/`).
- **Analysis scripts** reproducing the correctness, failure-mode, and
  performance results in the paper (`analysis/`).

## Reproducing the study

The pipeline has four stages:

1. **Prompt construction** — `prompts/generate-pycompss-prompts.py` builds the
   Tier-0 and Tier-1 prompt sets.
2. **Generation** — `generate/generate-all.sh` (open-weight, via vLLM) and
   `generate/generate-all-api.sh` (commercial models) produce completions in
   `generate/output/`.
3. **Evaluation** — `drivers/run-all-models-slurm-parallel.sh` (correctness) and
   `drivers/run-all-models-slurm-scaling.sh` (scaling) run the PyCOMPSs driver.
4. **Analysis** — `analysis/run_analysis_all.sh` computes pass@k, speedup, and
   efficiency and generates the figures and tables.

### PyCOMPSs setup

In addition to the base dependencies below, the task-based pipeline requires:

- [PyCOMPSs](https://compss-doc.readthedocs.io/en/3.4/) 3.4.post2603

Open-weight models are served locally with vLLM on GPU nodes; commercial models
are queried through their provider APIs (set the relevant API keys as
environment variables).

---

# Original ParEval

The sections below document the upstream ParEval benchmark, on which this work
builds.

[![HPDC 2024](https://img.shields.io/badge/Paper-HPDC'24-e87053.svg?style=flat)](https://pssg.cs.umd.edu/assets/papers/2024-06-pareval-hpdc.pdf)&nbsp;[![arXiv](https://img.shields.io/badge/arXiv-2401.12554-b31b1b.svg)](https://arxiv.org/abs/2401.12554)&nbsp;[![GitHub license](https://badgen.net/github/license/parallelcodefoundry/ParEval)](https://github.com/parallelcodefoundry/ParEval/blob/develop/LICENSE)


This repo contains the Parallel Code Evaluation (ParEval) Benchmark for
evaluating the ability of Large Language Models to write parallel code. See the
[ParEval Leaderboard](https://pssg.cs.umd.edu/blog/2024/pareval/) for
up-to-date results on different LLMs.


## Overview

The organization of the repo is as follows.

- `prompts/` -- the prompts in ParEval alongside some utility scripts
- `generate/` -- scripts for generating LLM outputs
- `drivers/` -- scripts to evaluate LLM outputs
- `analysis/` -- scripts to analyze driver results and compute metrics
- `tpl/` -- git submodule dependencies

Each subdirectory has further documentation on its contents. The general
workflow is to use `generate/generate.py` to generate LLM outputs, run
`drivers/run-all.py` to evaluate outputs, and `analysis/metrics.py` to
post-process the results.

## Setup and Installation

A couple core systems software are assumed to be installed: Python >=3.7, a C++
compiler that supports C++17 and OpenMP, Make, CMake, and an MPI implementation.
If you are testing the CUDA and HIP prompts, then you will need access to NVIDIA
and AMD GPUs alongside their respective software stacks.

First, clone the repo.

```sh
git clone --recurse-submodules https://github.com/parallelcodefoundry/ParEval.git
```

Next, you need to build Kokkos (if you want to include it in testing).

```sh
cd tpl/kokkos

mkdir build
cd build

# depending on your system you may need to pass your c++ compiler to CMAKE_CXX_COMPILER
cmake .. -DCMAKE_INSTALL_PREFIX=. -DKokkos_ENABLE_THREADS=ON
make install -j4
```

You will need to build the main C++ drivers before running ParEval. The included
makefile will skip CUDA, HIP, and/or Kokkos if their respective libraries cannot
be found.

```sh
# from the repository root, step into the cpp drivers directory and run make
cd drivers/cpp
make
```

Finally, you need to install the Python dependencies. `requirements.txt` has
the set of dependencies pinned at the version they were tested with. Other
versions may also work. Note that some of these are only required for parts of
the pipeline i.e. PyTorch and Transformers are only needed for generating LLM
outputs.

```sh
pip install -r requirements.txt
```

## Citing ParEval

```
@misc{nichols2024large,
      title={Can Large Language Models Write Parallel Code?}, 
      author={Daniel Nichols and Joshua H. Davis and Zhaojun Xie and 
              Arjun Rajaram and Abhinav Bhatele},
      year={2024},
      publisher = {Association for Computing Machinery},
      address = {New York, NY, USA},
      booktitle = {Proceedings of the 33rd International Symposium on High-Performance Parallel and Distributed Computing},
      series = {HPDC '24}
}
```

## License

ParEval is distributed under the terms of the [MIT license](/LICENSE).
