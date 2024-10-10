# Python

## Installing python

This document offers comprehensive guidelines for installing Python on Ubuntu. It discusses multiple installation methods, such as utilizing APT, `Anaconda`, or `pyenv` for Python version management. The document also provides detailed instructions for installing `Anaconda`, setting up a virtual environment with Anaconda, and installing and configuring `pyenv` for Python version and virtual environment management.



### Installing python using _apt_

Before installing [Python](https://www.python.org/) on Ubuntu, it is important to check if Python is already installed on your system. You can do this by running the following command:

```bash
python3 --version
```

If Python `^3.10` is not already installed, you have the option to use APT to install it on your system. To do so, please follow these steps:

To update the system repository list, use the following command:sudo apt install python3

```sh
sudo apt update
```

1.  To set up Python, install the required dependencies using the following command:sudo apt&#x20;

    {% code overflow="wrap" %}
    ```sh
    sudo apt install build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
    ```
    {% endcode %}
2.  Install the latest version of Python with the command:

    ```
    sudo apt install python3
    python3 --version
    ```

***

{% hint style="info" %}
_(Recommended)_ Using Anaconda or pyenv

Instead of directly installing Python on your system, it is recommended to use Python version management tools like _Anaconda_ or _pyenv_.
{% endhint %}

### Installing python using anaconda

[Anaconda](https://www.anaconda.com/) is a software package that includes Python and R programming languages. It is widely used in scientific computing for various fields like data science, machine learning, and large-scale data processing.

If you are using Linux and want to use _GUI packages_, you will need to install additional dependencies for Qt.

{% code overflow="wrap" %}
```sh
apt-get install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6
```
{% endcode %}

To install the latest version of Anaconda, follow these steps:

1.  Download the latest version of Anaconda from the official website:shasum -a 256&#x20;

    {% code overflow="wrap" %}
    ```sh
    apt-get install libgl1-mesa-glx libegl1-mesa libxrandr2 libxrandr2 libxss1 libxcursor1 libxcomposite1 libasound2 libxi6 libxtst6
    ```
    {% endcode %}
2.  (Recommended) Ensure the integrity of the installer's data by verifying it with SHA-256. For additional details on hash verification, please refer to the documentation on cryptographic hash validation.

    ```bash
    shasum -a 256 /PATH/ANACONDA
    # Replace /PATH/FILENAME with your installation's path and filename.
    ```
3.  Install Anaconda by running the downloaded installer :&#x20;

    ```sh
    bash ~/PATH/TO/ANACONDA
    # bash ./Anaconda3-2023.09-0-Linux-x86_64.sh
    ```
4.  Update all packages in Anaconda

    ```sh
    conda update --all
    ```
5.  Set up a Python Virtual Environment using Anaconda :&#x20;

    ```sh
    conda create -n <Environment Name> python=<Version>
    # e.g., conda create -n Liberate python=3.11
    ```
6.  Activate the virtual environment :&#x20;

    ```sh
    conda activate <Environment Name>
    # e.g., conda activate Liberate
    ```

For more information on how to use Anaconda and its features, please refer to the [official documentation](https://www.anaconda.com/).

***

### Installing python using pyenv

[Pyenv](https://github.com/pyenv/pyenv) is a tool that enables you to conveniently handle multiple Python versions and virtual environments.

To configure pyenv, you can simply follow these steps :

1.  Download pyenv using the following command:

    ```sh
    curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
    ```
2.  Update pyenv to the latest version :

    ```sh
    pyenv update
    ```
3. To ensure proper configuration, please add the necessary settings to your shell startup file (`~/.bashrc`, `~/.zshrc`, etc.), based on your shell environment.
   *   For Bash :&#x20;

       ```sh
       echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
       echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
       echo 'eval "$(pyenv init -)"' >> ~/.bashrc

       ```
   *   For Zsh :

       ```sh
       echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
       echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
       echo 'eval "$(pyenv init -)"' >> ~/.zshrc
       ```
4.  To install the Python version you want, you can use pyenv. First check the available Python versions using the command `pyenv install --list`. Then, install a specific version by running the appropriate command.

    ```sh
    pyenv install --list
    pyenv install <PYTHON VERSION>
    # ex) pyenv install 3.11.6
    ```


5.  To start, create a Python virtual environment and activate it :&#x20;

    ```sh
    pyenv virtualenv <PYTHON VERSION> <VIRTULAENV NAME>
    # ex) pyenv virtualenv 3.11.6 liberate
    pyenv activate <VIRTUALENV NAME>
    # ex) pyenv activate liberate
    ```

To learn more about using `pyenv` and its features, please refer to the [official documentation](https://github.com/pyenv/pyenv).
