# Progressive Diffusion Link Prediction

Link prediction in networks through progressive information diffusion. The program computes scores between node pairs and generates versions of the original graph with additional edges, using different diffusion steps and measures.

## Requirements

- Python 2.7.
- Python 2-compatible versions of NumPy, NetworkX, and SciPy.
- Weave (`from weave import inline`).
- GCC for the code compiled through Weave.

Processing runs on the CPU using multiple processes. This version relies on legacy dependencies and is not directly compatible with Python 3. Exact library versions are not specified in the source code; use an isolated environment with these dependencies installed.

## Usage

From the script directory, create the output folder and run:

```bash
mkdir -p results
python2 lpd_progressive_diffusion.py data/network.gml results 0.1 1.0
```

Replace `data/network.gml` with your graph path. The example assumes the script is named `lpd_progressive_diffusion.py`.

Provide the arguments in this order:

| Argument | Description |
| --- | --- |
| Input file | Path to the graph in `.gml` format. |
| Output directory | Existing folder where results will be saved. |
| `beta` | Information propagation parameter. |
| `gamma` | Parameter controlling when nodes stop spreading information. Use a value greater than zero. |

Use a simple, undirected, unweighted graph with no isolated nodes and consecutive integer IDs from `0` to `N-1`. Node ordering must match these IDs because the code uses them as matrix indices.

## Configuration

The following settings are defined near the beginning of the script:

```python
factors = [0.05, 0.1, 0.2]
maxSteps = 22
```

- `factors`: fractions of the original edge count used to select new connections — 5%, 10%, and 20%.
- `maxSteps`: number of temporal states computed by the main execution path, including the initial state.

With the default settings, results are generated at steps 3, 6, 9, 12, 15, 18, and 21. Combining two diffusion measures, two score aggregation methods, and three edge fractions produces **84 `.gml` files per run**.

## Output

### Graphs (`.gml`)

Each file contains the input graph with the selected edges added. Example filename:

```text
network__EtaPt3-RPS0-05L0_1.gml
```

| Identifier | Meaning |
| --- | --- |
| `Eta` | Uses the probability that a node has already received information (`I + R`). |
| `Inf` | Uses the probability that a node is currently spreading information (`I`). |
| `Pt3` | Diffusion step used. |
| `RPS` / `RPA` | Sum / average of the scores in both directions of a node pair. |
| `0-05` | Selected edge fraction: 5%. |
| `L0_1` | Ratio `beta / gamma`: 0.1. |

### Matrices (`.npz`)

The file `networkMATRIXS-Peak_0_1.npz` stores:

- **`listI`**: probability that each node is currently spreading information.
- **`listRS`**: probability that each node has already received information, whether it is still spreading or has stopped (`I + R = 1 - S`).

The arrays contain results indexed by source node, target node, and time step. They are not embeddings.

If the `.npz` file already exists, the program reuses its data. Use a different output folder or remove the cached file when changing the graph, step count, or parameters: the cache filename identifies only the graph name and the `beta / gamma` ratio.

## Reference

Related work on link prediction and the evaluation of diffusion processes in networks:

Vega-Oliveros, D. A.; Zhao, L.; Berton, L. **Evaluating link prediction by diffusion processes in dynamic networks**. Scientific Reports, 9, 10833, 2019. [DOI: 10.1038/s41598-019-47271-9](https://doi.org/10.1038/s41598-019-47271-9).
