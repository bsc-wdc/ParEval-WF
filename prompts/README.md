# Prompts

This directory contains the ParEval prompts.
The prompts for the generation task are contained in `generation-prompts.json`
and the prompts for the translation task are in `translation-prompts.json`.
Prompts exist for several parallelism models (`serial`, `omp`, `cuda`,
`pycompss`, ...); the per-model source files live under `kernel/`, `kernel-guided/`,
and `translate/` and are gathered into the composed JSON files above.


The format of the prompts dataset is as follows:

```json
[
    {
        "problem_type": "stencil",
        "language": "cpp",
        "name": "17_problem_name",
        "parallelism_model": "serial",
        "prompt": "/* prompt for the model here */\nvoid foo(int x) {"
    },
    ...
]
```

## Other Utilities

`gather-raw-prompts.py` -- gather the raw per-model source files into a single
composed JSON in the correct format.

`generate-pycompss-prompts.py` / `generate-python-serial-prompts.py` -- derive
the `pycompss` and `serial-python` source files from the `omp` ones.

`create-serial-tests.py` -- this script will parse out the sequential baselines
for each problem from the drivers and create a "fake" output file with these
as the sequential solutions. This can be used to test the driver setup and make
sure it's working.

`count-tokens.py` -- this can be used to estimate the number of tokens passed
to the OpenAI API and, thus, estimate the cost of generated outputs.