# build-a-pim — build & run notes

Verified 2026-09-08 on Ubuntu 22.04, kernel 5.15.
Status: **build OK / run OK** (127/127 tests pass, all 6 demos run).

## Gotcha 1 — submodule URL is SSH-only
`.gitmodules` points at `git@github.com:UVA-LavaLab/pimDRAMsim`, which fails without
a deploy key. Override it with HTTPS:
```bash
git config submodule.dramsim3.url https://github.com/UVA-LavaLab/pimDRAMsim
git submodule update --init --recursive dramsim3
```

## Gotcha 2 — needs Python 3.14
`lib/cores/components/pipeline.py` annotates `Stage` with `Ptr[Stage]` *inside its
own class body*. That only works under PEP 649 deferred annotations, i.e. Python
**3.14+**. On 3.12 every test errors with `NameError: name 'Stage' is not defined`.
Do not "fix" this with `from __future__ import annotations` — just use 3.14.

A repo-local interpreter (no root, nothing outside the repo):
```bash
conda create -y -p ./.conda-py314 python=3.14
./.conda-py314/bin/pip install -r requirements.txt
```

## Build
```bash
./build.sh          # cmake + make of the bundled pimDRAMsim3 fork
```
Produces `dramsim3/build/libdramsim3.so` and `dramsim3/build/dramsim3main`.

## Run
```bash
./.conda-py314/bin/python -m pytest -q        # 127 passed in ~37s
# demos MUST run from the repo root (they use relative imports)
./.conda-py314/bin/python demo/vec-add.py
./.conda-py314/bin/python demo/redsum.py
./.conda-py314/bin/python demo/scalar-vec-add.py
./.conda-py314/bin/python demo/streaming-vec-add.py
./.conda-py314/bin/python demo/streaming-redsum.py
./.conda-py314/bin/python demo/streaming-scalar-vec-add.py
```
Each demo prints a correctness check plus `cycles taken` / `time taken`.

## Data prep
None.
