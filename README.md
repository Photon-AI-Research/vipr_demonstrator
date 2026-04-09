# What is VIPR?
TL;DR: VIPR (Versatile Inverse Problem Software Framework) is a modular machine learning framework designed for inverse problems in physics.

Quick detail: VIPR (Versatile Inverse Problem Software Framework) is a plugin-based framework for reproducible machine-learning-driven solutions to scientific inverse problems.  It addresses ill-posed reconstruction tasks, e.g., caused by loss of phase information during measurement, where direct inversion is not possible. It implements a modular microkernel architecture with domain-specific plugins to produce configurable machine learning workflows, including both deterministic and probabilistic models. Workflows are defined via declarative YAML configurations and can be executed through a command-line interface or a containerized web application. For a given experimental dataset, VIPR produces standardized analysis artifacts, including visualizations and statistical summaries.

📊 **[TLDR Introduction](framework/docs-sphinx/docs/TLDR%20VIPR%20Introduction.pdf)** - Quick framework overview (PDF)

**Getting Started:**
- 🎉 `git clone --recursive https://github.com/Photon-AI-Research/vipr_demonstrator.git`
- 🖥️ [CLI Tutorial](framework/VIPR_CLI.md) - Command-line interface
- 🌐 [Web Application Tutorial](framework/VIPR_web_app.md) - Docker-based web UI
- 🌐 [Web Application Without Docker](framework/VIPR_web_app_no_docker.md) - Local development setup

## Overview

**VIPR** is a modular ML framework with:
- **vipr-frontend**: Web interface (Nuxt.js)
- **vipr-api**: REST API backend (FastAPI) with async processing (Celery)
- **vipr-core**: Extensible plugin framework (Python CLI)

## Getting Started

### 🖥️ Command Line Interface (CLI)
- **[Getting Started Tutorial](framework/VIPR_CLI.md)** - Installation to first prediction
- **[CLI Introduction Presentation](framework/docs-sphinx/docs/VIPR%20CLI%20introduction.pdf)** 📊 - CLI + Architecture deep-dive (PDF)
- **[Installation Reference](framework/docs-sphinx/docs/installation/cli.md)** - Quick install & commands

### 🌐 Web Application (Docker)
- **[Getting Started Tutorial](framework/VIPR_web_app.md)** - Setup and first use
- **[Without Docker](framework/VIPR_web_app_no_docker.md)** - Local development setup with Redis, API, worker, and frontend
- **[Inference Flow](framework/docs-sphinx/docs/web-app/inference-flow.md)** - How inference works (Frontend → Backend → Core)
- **[Deployment Reference](framework/docs-sphinx/docs/installation/docker.md)** - Services, configuration, troubleshooting
- **Access**: http://localhost:3000 (UI), http://localhost:8000/docs (API)

### 🔌 API Integration
- **Setup**: Follow [Web App Tutorial](VIPR_web_app.md)
- **API Docs**: http://localhost:8000/docs

## Quick Links

- [Documentation](framework/docs-sphinx/docs/) - Complete documentation
- [Docker Deployment](framework/docs-sphinx/docs/installation/docker.md) - Web UI + API setup
- [Web App Without Docker](framework/VIPR_web_app_no_docker.md) - Local development setup
- [CLI Installation](framework/docs-sphinx/docs/installation/cli.md) - Local CLI setup
- [CLI Commands](framework/docs-sphinx/docs/cli/commands.md) - Command reference

## System Requirements

- Python 3.10+


## Project Structure

```
vipr-framework/
├── README.md                    # This file
├── docker-compose.yml           # Docker orchestration
├── common.env                   # Default environment config
├── docs/
│   ├── README.md                # Documentation index
│   ├── installation/
│   │   ├── cli.md               # CLI installation guide
│   │   └── docker.md            # Docker deployment guide
│   └── cli/
│       └── commands.md          # CLI command reference
└── storage/                     # Persistent data (created on startup)
```

## License

This project is licensed under the GNU Lesser General Public License v3.0 or later (LGPL-3.0-or-later).

See [LICENSE.txt](framework/LICENSE.txt) for the full license text and [NOTICE.txt](framework/NOTICE.txt) for component information.

### VIPR Components

The VIPR Framework orchestrates multiple components, each with their own repositories and LGPL-3.0-or-later license:
- **vipr-core**: Core Python framework - https://codebase.helmholtz.cloud/vipr/vipr-core
- **vipr-api**: FastAPI backend service - https://codebase.helmholtz.cloud/vipr/vipr-api
- **vipr-reflectometry-plugin**: Reflectometry analysis plugin - https://codebase.helmholtz.cloud/vipr/vipr-reflectometry-plugin
- **vipr-frontend**: Nuxt.js web application - https://codebase.helmholtz.cloud/vipr/vipr-frontend

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
