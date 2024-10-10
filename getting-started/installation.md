# Installation

## Liberate.FHE Installation Guide

Liberate.FHE natively supports python programming language. To install and use Liberate.FHE, please follow the steps below:

## [GPU](../how-to/configure/gpu-and-cuda.md#gpu) and [CUDA](../how-to/configure/gpu-and-cuda.md#cuda) setup

Liberate.FHE runs on single or multiple GPUs. Running Liberate.FHE on GPUs requires installing [nvidia-driver](https://www.nvidia.co.kr/Download/index.aspx?lang=kr). Additionally, you need to install [CUDA](https://developer.nvidia.com/cuda-toolkit) that matches the version of [PyTorch](https://pytorch.org/) you intend to use. Theses settings are necessary for GPU-based operations.

## [Python setup](../how-to/configure/python.md) (including Virtual Environment)

To build Liberate.FHE, you need to install [poetry](https://python-poetry.org/), a dependency manager and packaging tool for Python. It is recommended to set up a virtual environment to manage the project in an isolated environment.

## Install Liberate.FHE

### Clone Liberate.FHE github repository

Clone the [github repository](https://github.com/Desilo/Liberate-dev) of Liberate.FHE to obtain the latest source codes.

{% code overflow="wrap" %}
```bash
git https://github.com/Desilo/liberate-fhe.git
```
{% endcode %}

### Install dependencies

Use poetry to install the project dependencies. Open a terminal and run the command `poetry install`. This will automatically install all the required packages for the Liberate.FHE.

```bash
poetry install
```

### Run CUDA compile script

To compile CUDA by running the `setup.py` file. In the terminal, run the command `poetry run python setup.py install`. This command will compile CUDA files.

```bash
python setup.py install
# poetry run python setup.py install
```

### Build a python package

Build the project by running the command `poetry build` in the terminal. This will create a distributable format of the package.

```sh
poetry build
```

### Install Liberate.FHE library

Install the Liberate.FHE by running the command `poetry run python -m pip install .` in the  terminal. This will install the Liberate.FHE library into your system.

```bash
pip install .
# poetry run python -m pip install .
```

***

## System Requirements

* **Operating System** : Liberate.FHE is compatible with [Ubuntu](https://ubuntu.com/).
* **Python** : Liberate.FHE requires [Python](https://www.python.org/) version _3.10_ or above. You can download and install Python from the official website. And we recommend using the python virtual environment.
* **PyTorch** : Liberate.FHE uses the [pyTorch](https://pytorch.org/) package. When you install Liberate.FHE, it automatically installs PyTorch.
* **CUDA** : If you want to utilize the GPU, install [CUDA](https://developer.nvidia.com/cuda-toolkit). Ensure that you choose a version of CUDA that is compatible with PyTorch, and install it accordingly.
