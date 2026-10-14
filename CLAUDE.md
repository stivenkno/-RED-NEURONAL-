# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Teaching material on neural networks built as **marimo notebooks** (not Jupyter). There is no package, test suite, or linter. The two notebooks are:

- `nn.py`: an interactive walkthrough from perceptron → NAND gate → sigmoid neuron → a 2-4-1 XOR network trained with hand-written NumPy backprop. It includes the full backprop derivation in LaTeX markdown and a step-by-step training demo.
- `mnist.py`: a from-scratch NumPy 784-10-10 classifier (sigmoid hidden layer, softmax output) on MNIST. torchvision is used **only** to download/load the dataset into `./data/MNIST`.

The PNG/SVG images referenced by `mo.image(src=...)` are exported from the `.excalidraw` sources in the repo root. If you edit a diagram, re-export the PNG with the same filename.

## Running

Each notebook declares its dependencies inline (PEP 723 `# /// script` header, `requires-python >= 3.14`). The intended workflow, from `.claude/settings.local.json`:

```bash
uvx marimo@latest edit --no-token --sandbox nn.py      # or mnist.py
uv run nn.py                                            # run as a script
```

`--no-token` lets the marimo-pair skill discover the running server. The local `venv/` (Python 3.12, marimo/numpy/matplotlib/plotly) does **not** include torch/torchvision, so `mnist.py` won't run there. Use the `uv` sandbox instead, or install those packages.

## marimo conventions to respect when editing

- Cells are `@app.cell` functions whose parameters are the variables they read and whose `return` tuple lists the variables they export. marimo builds the reactive dependency graph from these. **Every global name may be defined in only one cell**, which is why the notebooks use `_`-prefixed locals (cell-private) and unique aliases like `import plotly.graph_objects as _go_sig`. Keep that pattern when adding cells.
- Shared imports (`mo`, `np`, `plt`, `torchvision`) are defined in the **last** cell of each file. Placement doesn't matter because marimo orders execution by dependencies.
- Explanatory/UI cells use `hide_code=True`.
- Interactive widgets (`mo.ui.slider/switch/matrix/button`) must be defined in one cell and have their `.value` read in a *different* cell.
- The stepped XOR trainer in `nn.py` uses `mo.state` (`get_step_state`/`set_step_state`) with button `on_change` callbacks, so training state persists across UI interactions.
- `__marimo__/session/` holds marimo's cached session outputs. Don't hand-edit them.

## Math/shape conventions

- `nn.py` (XOR) uses a **row-vector** convention in code: `X` is `(N, 2)`, `W1` is `(2, 4)`, `z1 = X @ W1 + b1`. The markdown math and the `mo.ui.matrix` widgets show the **transposed** `W^{(1)} ∈ R^{4×2}`, so `.T` appears whenever values move between widgets and code. Loss is MSE with sigmoid output.
- `mnist.py` uses a **column-vector** convention: `X` is `(784, m)`, `W1` is `(10, 784)`, `Z1 = W1.dot(X) + b1`. Loss is cross-entropy with softmax, so `dZ2 = A2 - one_hot(Y)`. It trains on the first 50k of the 60k training images and evaluates on the last 10k (the official test split is unused).
- `macros.tex` defines `\RR` and `\boldx` LaTeX macros.

## Agent skill

`.agents/skills/marimo-pair/` (see `SKILL.md` and `reference/gotchas.md`) provides `scripts/discover-servers.sh` and `scripts/execute-code.sh --port <port>` for running code inside a live marimo kernel (default port 2718). Prefer it for editing cells in a running notebook so the user sees changes live.
