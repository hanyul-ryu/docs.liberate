# Liberate

## Prerequisite Package

### GPU

Please refer to the respective page for instructions on how to install a [GPU](gpu-and-cuda.md#gpu).

### CUDA

Please refer to the respective page for instructions on how to install a [CUDA](gpu-and-cuda.md#cuda).

### Python

Please refer to the respective page for instructions on how to install a [Python](python.md).

## Liberate.FHE

### Step 1: Clone the github repository

Clone the [Github repository](https://github.com/Desilo/Liberate-dev) of Liberate.FHE to obtain the latest source code.

{% code overflow="wrap" %}
```bash
git clone https://github.com/Desilo/liberate-fhe.git
```
{% endcode %}

### Step 2: Install dependencies

Use poetry to install the project dependencies. Open the terminal and run the command `poetry install`. This will automatically install all the required packages for the Liberate.FHE.

```bash
poetry install
```

### Step 3: Run CUDA build script&#x20;

To build CUDA by running the `setup.py` file. In the terminal, run the command `poetry run python setup.py install`. This command will build CUDA files.

```bash
python setup.py install
# poetry run python setup.py install
```

### Step 4: Install Liberate.FHE library

Install the Liberate.FHE by running the command `poetry run python -m pip install .`in the  terminal. This will install the Liberate.FHE library into your system.

```bash
pip install .
# poetry run python -m pip install .
```



