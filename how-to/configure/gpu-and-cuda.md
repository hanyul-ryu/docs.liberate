# GPU & CUDA

To use Liberate.FHE, a GPU is required. This document explains how to set up and install the Nvidia graphics card driver and CUDA. It provides steps to identify and install the GPU, update package lists, install Nvidia drivers, verify the installation, install the CUDA toolkit, set environment variables, and verify the CUDA installation.



## Setting graphic card

### GPU

#### Step 1: Identify your GPU

Before you begin, it's essential to identify the Nvidia GPU model in your system. You can use the following commands to check:

```bash
lshw -numeric -C display
lspci | grep -i nvidia
```

If the Nvidia graphics card is not installed or an incompatible version of CUDA is installed, install a new version.

#### Step 2: Update package lists

Ensure that your package lists are up to date.

```bash
sudo apt update
```

Check available graphic cards.

```bash
ubuntu-drivers devices
```

#### Step 3: Install nvidia driver

Automatically install the recommended drivers.

```bash
sudo ubuntu-drivers autoinstall
```

Alternatively, manually install the desired version.

```bash
sudo apt install nvidia-driver-535-server
```

Don't forget to restart the server after installing the graphic card driver.

#### Step 4: Verify installation

Please ensure that the graphics card is in working order.

```bash
nvidia-smi
```

If you encounter any issues during the installation or need further customization, you can refer to the official [Nvidia documentation](https://www.nvidia.com/download/index.aspx) or seek help from the Ubuntu community forums.

***

### CUDA

#### Step 1: Verify your GPU

Make sure your GPU is CUDA-compatible. You can find a list of CUDA-enabled GPUs on the [Nvidia website](https://docs.nvidia.com/deploy/cuda-compatibility/).

#### Step 2: Install the nvidia GPU drivers

Please refer to the instructions provided above.

#### Step 3: Download CUDA toolkit

Visit the Nvidia [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads) download page.

#### Step 4: Install the CUDA toolkit

Assuming you've downloaded the toolkit:

{% code fullWidth="false" %}
```sh
# ex)cuda 12.1.0
wget https://developer.download.nvidia.com/compute/cuda/12.1.0/local_installers/cuda_12.1.0_530.30.02_linux.run
sudo sh cuda_12.1.0_530.30.02_linux.run
```
{% endcode %}

#### Step 5: Set environment variables

Add CUDA to your `PATH` and update `LD_LIBRARY_PATH`. You can do this by adding the following lines to your `~/.bashrc` or `~/.zshrc` file:

```bash
export PATH=/usr/local/cuda-12.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-12.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

Remember to provide the source for your updated profile:

```bash
source ~/.bashrc
```

#### Step 6: Verify installation

Please verify if CUDA has been installed correctly by executing the following command:

```bash
nvcc --version
```

This should display the CUDA compiler version.

***
