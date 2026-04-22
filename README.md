<div align="center" style="display: flex; justify-content: center; align-items: center; gap: 20px; margin-bottom: 20px;">
  <img src="images/vipr_logo_small.png" alt="VIPR Logo" 
  width="100" style="box-shadow: 0 0px 0px 0 rgba(0, 0, 0, 0.2);"/> </div>

## What is VIPR?
TL;DR: VIPR (Versatile Inverse Problem Software Framework) is a modular machine learning framework designed for inverse problems in physics.

Quick detail: VIPR (Versatile Inverse Problem Software Framework) is a plugin-based framework for reproducible machine-learning-driven solutions to scientific inverse problems.  It addresses ill-posed reconstruction tasks caused by loss of phase information during measurement, where direct inversion is not possible. It implements a modular microkernel architecture with domain-specific plugins to produce configurable machine learning workflows, including both deterministic and probabilistic models. Workflows are defined via declarative YAML configurations and can be executed through a command-line interface or a containerized web application. For a given experimental dataset, VIPR produces standardized analysis artifacts, including visualizations and statistical summaries.

## Repository structure:
vipr-demonstrator has directories titled `vipr-<component>`, which are basically submodules hosted at https://codebase.helmholtz.cloud/vipr/
```
vipr-demonstrator/
├── README.md                    # This file
├── images                       # Images featured in this readme file
├── vipr-api					 # REST API backend (FastAPI) with async processing (Celery)
├── vipr-core					 # Provides fundamental infrastructure (e.g. CLI, config management, plugin ecosystem w/o domain logic).
├── vipr-frontend				 # Web interface (Nuxt.js)
├── vipr-framework               # Implements containerization of VIPR-project using docker-compose. Also features source files for the documentation hosted at (https://vipr-docs.pages.dev/) which might be of interest to developers (and extra-keen users ;) )
└── vipr-reflectometry-plugin    # Example domain-plugin, registers domain-specific handlers, filters, and hooks with the core.
```

## Inference Pipeline Overview

Users can perform analysis via either the command-line interface (CLI) or the web interface (check 'Steps to reproduce...' subsection for details). Regardless of the choice of interface, at the core of VIPR is a domain-agnostic, five-step, standardized inference pipeline ensuring a reproducible workflow. The pipeline ensures that data flows through a predictable sequence of operations:

1.  :floppy_disk: **Load Data:** Experimental files (e.g., CSV, HDF5) are loaded
    and standardized into a unified `DataSet` structure containing
    input given as a vector of arguments ($x$), a mapped vector of
    values/results ($y$) optionally with uncertainties ($dx, dy$) by the corresponding file format-defined data handler.

2.  :robot: **Load Model:** A pre-trained model is loaded and prepared
    for inference on the configured device (CPU/GPU) by the model handler.

3.  :hammer_and_pick: **Preprocess:** A chain of filters transforms the `DataSet` for  model consumption. Filters are executed in deterministic
    weight-sorted order and configured via YAML, enabling operations
    such as normalization, data cleaning, outlier removal, or
    interpolation to the model's input grid.

4.  :tada: **Predict:** The model generates predictions based on the
    preprocessed input.

5. :bar_chart: **Postprocess:** Results are formatted, and hooks are triggered to generate artifacts (plots, tables) or persist results.


## Software architecture in a nutshell (in the interest of developers especially):

VIPR is built upon the [Cement application framework](https://github.com/datafolklabs/cement/) and
[NF4IP](https://github.com/Photon-AI-Research/NF4IP). The framework uses three primary mechanisms to ensure extensibility
without modifying the core codebase:
-   **Handlers** extend handler base classes (e.g., `DataLoaderHandler`,
    `ModelLoaderHandler`, `PredictorHandler`) to implement
    domain-specific data loading, model loading, and prediction.
-   **Filters** are chainable functions that transform data passing
    through the pipeline (e.g., interpolation, normalization). They are
    executed in a deterministic, weight-sorted order.
-   **Hooks** are callbacks used for non-transforming tasks such as
    logging, validation, or real-time visualization.

![Overview of VIPR](images/pictorial_overview_handmade.png)

### Deployment architecture
![Deployment architecture](images/vipr_deployment_alternate.png)


[Here](https://vipr-docs.pages.dev/) is a link to an in-depth documentation!

## Steps to reproduce the results presented in the [arXiv paper](http://arxiv.org/abs/nnnn.nnnnn) 
### Using CLI:
```bash
git clone --recursive https://github.com/Photon-AI-Research/vipr_demonstrator.git
# Setup a virtual environment and install all the dependencies
cd vipr_demonstrator/vipr-api
python3 -m venv venv
source venv/bin/activate
pip install .
pip install -r requirements.txt 
####################
# reflectorch plugin example: deterministic workflow on experimental X-ray reflectometry data
# (PTCDI-C3 on thin SiO~x~ on Si substrate)
vipr --config @vipr_reflectometry/reflectorch/examples/configs/PTCDI-C3.yaml inference run
# Output path: storage/results/inference/PTCDI-C3/
####################
# normalizing flow example: probabilistic workflow on experimental neutron reflectometry data
# (Pt/Fe MARIA dataset)
# TODO(public-model-release): switch these downloads to public Hugging Face artifacts.
# Current GitLab URLs require authenticated access to codebase.helmholtz.cloud.
mkdir -p storage/reflectometry/flow_models/configs/ storage/reflectometry/flow_models/saved_models/
curl -L "https://codebase.helmholtz.cloud/vipr/models/reflectometry-nsf-nr-maria/-/raw/models/models/fxc34ran/config.yaml" -o storage/reflectometry/flow_models/configs/fxc34ran.yaml
curl -L "https://codebase.helmholtz.cloud/vipr/models/reflectometry-nsf-nr-maria/-/raw/models/models/fxc34ran/model.pt" -o storage/reflectometry/flow_models/saved_models/fxc34ran.pt
vipr --config @vipr_reflectometry/flow_models/examples/configs/Fe_Pt_DN_NSF.yaml inference run
# Output path: storage/results/inference/<result_id>/
```

### Using VIPR Web App Without Docker
```bash
# From the demonstrator root
cd vipr_demonstrator

# 1) Install and start Redis (Ubuntu/Debian)
sudo apt update
sudo apt install -y redis-server redis-tools
sudo systemctl enable redis-server
sudo systemctl start redis-server
redis-cli ping   # should print PONG

# Alternative without system-wide install:
# cd ~
# wget https://download.redis.io/redis-stable.tar.gz
# tar xzf redis-stable.tar.gz
# cd redis-stable
# make
# ~/redis-stable/src/redis-server --port 6379 --daemonize yes --logfile ~/redis.log --dir ~/

# 2) Prepare VIPR API environment
cd vipr-api
# If you already ran the CLI setup above, reuse that environment:
# source venv/bin/activate
# Otherwise create it now:
python3 -m venv venv
source venv/bin/activate
pip install .
pip install -r requirements.txt
cp example.env .env
export $(grep -v '^#' .env | xargs)

# 3) Create required storage directories
mkdir -p \
  storage/reflectometry/flow_models/configs \
  storage/reflectometry/flow_models/saved_models \
  storage/huggingface_cache \
  storage/results

# 4) Start API (terminal 1)
python -m vipr_api.main

# Optional: skip model download during startup if models are already present
# VIPR_NO_MODEL_DOWNLOAD=true python -m vipr_api.main

# 5) Start worker (terminal 2)
cd vipr_demonstrator/vipr-api
source venv/bin/activate
export $(grep -v '^#' .env | xargs)
celery -A vipr_api.celery.celery_app worker --loglevel=debug -c 1

# 6) Start frontend (terminal 3)
cd vipr_demonstrator/vipr-frontend
# If nvm is not installed:
# curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
nvm install
nvm use
npm i
npm run dev

# 7) Open UI and run examples
# Frontend: http://localhost:3000
# API docs: http://localhost:8000/docs
# In the frontend, click "Load Examples" and run "PTCDI C3" or "fxc34ran".

# 8) If fxc34ran model files are missing:
# Reuse the download commands from the CLI section above
# (normalizing flow example for the Pt/Fe MARIA dataset).

# 9) Shutdown
# Stop API, worker, and frontend with Ctrl+C in their terminals.
sudo systemctl stop redis-server
# If you used the local Redis build, stop it with:
# ~/redis-stable/src/redis-cli shutdown
```

## License

This project is licensed under the GNU Lesser General Public License v3.0 or later (LGPL-3.0-or-later).

See [LICENSE.txt](vipr-framework/LICENSE.txt) for the full license text and [NOTICE.txt](vipr-framework/NOTICE.txt) for component information.

## How to Cite VIPR
If you use this code in your research, please kindly cite the following paper

```text
@article{rustamov2026vipr,
  title={VIPR: Versatile Inverse Problem Software Framework},
  author={Rustamov, Jeyhun and Creutzburg, Sascha},
  journal={arXiv preprint arXiv:nnnn.nnnnn},
  year={2026}
}
```

## Acknowledgments

This project is developed by the VIPR Project Consortium with contributions from:
- [Helm & Walter IT-Solutions GmbH](https://www.helmundwalter.de/)
- [Forschungszentrum Jülich (FZJ) - Jülich Centre for Neutron Science (JCNS)](https://www.fz-juelich.de)
- [Technische Universität München (TUM)](https://www.tum.de)
- [University of Tübingen](https://www.uni-tuebingen.de)
- [University of Siegen](https://www.uni-siegen.de)
- [Helmholtz-Zentrum Dresden-Rossendorf (HZDR)](https://www.hzdr.de)
- [DESY (Deutsches Elektronen-Synchrotron)](https://www.desy.de)

For more information, visit: https://vipr-project.de
