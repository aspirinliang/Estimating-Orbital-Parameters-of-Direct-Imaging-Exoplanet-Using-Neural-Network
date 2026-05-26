# FM-MCMC: Estimating Orbital Parameters of Direct Imaging Exoplanets

[![arXiv](https://img.shields.io/badge/arXiv-2510.17459-b31b1b.svg)](https://arxiv.org/abs/2510.17459)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.x-ee4c2c.svg)](https://pytorch.org/)

This repository contains research code for the paper
**[Estimating Orbital Parameters of Direct Imaging Exoplanet Using Neural Network](https://arxiv.org/abs/2510.17459)**.

The project implements a flow-matching Markov chain Monte Carlo (FM-MCMC)
workflow for estimating orbital parameters of directly imaged exoplanets. The
main idea is to use flow matching posterior estimation (FMPE) to learn a fast,
data-conditioned proposal distribution, then use MCMC to refine that proposal
into an accurate Bayesian posterior.

On the beta Pictoris b experiment reported in the paper, FM-MCMC achieves
comparable posterior accuracy while running **77.8x faster than Parallel
Tempered MCMC (PTMCMC)** and **365.4x faster than nested sampling**.

## Method Overview

FM-MCMC is designed for single-planet direct-imaging systems where the
observations are relative astrometry time series.

1. **Simulate orbital systems**
   - Generate synthetic direct-imaging astrometry with `orbitize!`.
   - Sample the eight orbital/system parameters:
     `a`, `e`, `i`, `omega`, `Omega`, `tau`, `pi`, and `M_T`.

2. **Train an FMPE model**
   - Train a conditional continuous normalizing flow with flow matching.
   - The model maps noisy observational context, such as `Delta RA` and
     `Delta Dec`, to posterior samples over orbital parameters.

3. **Constrain the sampling region**
   - Draw fast neural posterior samples for the target system.
   - Use those samples to build tighter, data-adaptive priors or initial
     regions for classical sampling.

4. **Run flow-informed MCMC**
   - Use the FMPE output to warm-start or guide PTMCMC.
   - Let MCMC produce the final posterior distribution and likelihood-based
     diagnostics.

## Repository Structure

| Path | Description |
| --- | --- |
| `FMPE_train.ipynb` | Notebook for loading data, building the flow-matching posterior model, and training or loading checkpoints. |
| `test_5000pp.ipynb` | Validation notebook, including posterior sampling checks and P-P plot evaluation. |
| `FM-MCMC.ipynb` | Flow-informed MCMC experiment for beta Pictoris b using `orbitize!`. |
| `posterior_models/` | Posterior model implementations, including flow matching and normalizing-flow wrappers. |
| `utils/` | Training utilities, logging helpers, optimizer/scheduler builders, and statistical helpers. |
| `traing_4/best_model.pt` | Included trained checkpoint. The directory name is kept as it appears in the repository. |

## Installation

Create a clean Python environment and install the main scientific dependencies:

```bash
conda create -n fmmcmc python=3.10
conda activate fmmcmc
```

Install PyTorch with the CUDA or CPU build that matches your machine, then
install the remaining packages:

```bash
pip install numpy scipy matplotlib astropy pyyaml tqdm h5py threadpoolctl
pip install torchdiffeq zuko lampe orbitize emcee ptemcee dynesty
```

If you use a GPU, follow the official PyTorch installation selector:
https://pytorch.org/get-started/locally/

## Running the Notebooks

The current code is organized as research notebooks. Before running them, update
the hard-coded local paths in the notebooks, for example paths beginning with:

```text
/home/ps/4T/npe-astrometry-betapic/
```

Recommended workflow:

1. Open `FMPE_train.ipynb`.
2. Set the training and validation HDF5 dataset paths.
3. Build the model settings and train the FMPE model, or load the included
   checkpoint from `traing_4/best_model.pt`.
4. Use `test_5000pp.ipynb` to evaluate calibration with P-P plots.
5. Open `FM-MCMC.ipynb`, set `flow_path` to your generated FMPE sample file,
   and run the flow-informed MCMC experiment.

The beta Pictoris b astrometry used by the notebooks is loaded from the
`orbitize!` example data through:

```python
from orbitize import DATADIR, read_input

data_table = read_input.read_file(f"{DATADIR}/betaPic.csv")
data_table = data_table[:-1]  # discard the RV observation
```

## Data

The paper uses 16 million simulated single-planet systems analogous to beta
Pictoris b for training. The repository does not include the full training
dataset because of its size. To reproduce the training run, generate or place
the HDF5 datasets locally and update the notebook paths accordingly.

Expected dataset format in the notebooks follows `lampe.data.H5Dataset`, with
orbital parameters as targets and relative astrometry observations as context.

## Important Notes

This repository is currently a research snapshot. A few paths and helper imports
still reflect the original experiment environment:

- Some notebooks import local modules such as `helpers`.
- The model code imports `nn.cfnets` and `nn.nsf`.
- Some experimental cells refer to `posterior_models.score_matching`.
- Several notebooks contain absolute local paths that should be replaced before
  running.

If you plan to make the repository fully reproducible for external users, the
next useful cleanup steps are to add these helper modules, create a
`requirements.txt` or `environment.yml`, move paths into a config file, and add
standalone data-generation/training scripts.

## Citation

If you use this code or build on the FM-MCMC method, please cite:

```bibtex
@misc{liang2025estimatingorbitalparametersdirect,
  title         = {Estimating Orbital Parameters of Direct Imaging Exoplanet Using Neural Network},
  author        = {Bo Liang and Hanlin Song and Chang Liu and Tianyu Zhao and Yuxiang Xu and Zihao Xiao and Manjia Liang and Minghui Du and Wei-Liang Qian and Li-e Qiang and Peng Xu and Ziren Luo},
  year          = {2025},
  eprint        = {2510.17459},
  archivePrefix = {arXiv},
  primaryClass  = {astro-ph.EP},
  doi           = {10.48550/arXiv.2510.17459},
  url           = {https://arxiv.org/abs/2510.17459}
}
```

## License

No license file is currently included in this repository. Please add a license
before redistributing or reusing the code outside the original research context.
