# Virtual Environments

A Python Virtual Environment (venv) is created as a clean slate of libraries for one's project. A newly created venv contains only the necessary libraries, and the user may `pip install` others as required.

## Creating a new environment

An venv may be created via Command Prompt (Windows). Assuming `PATH` contains the base Python executable location, then

```shell
python -m venv c:\path\to\project\venv
```

This creates the necessary files for a virutual environment at the specified location. These are

```
|-Include
|-Lib
|-Scripts
|  |- activate
|  |- activate.bat
|  |- Activate.ps1
|  |- deactivate.shl
|  |- pip.py
|  |- pip3.10.py
|  |- pip3.py
|  |- python.exe
|  |- pythonw.exe
|-pyvenv.cfg
```

## Activating the Environment

Even though the files have been created, the environment must be 'activated'. On Windows Command Prompt, this is done through

```shell
C:\> <venv>\Scripts\activate.bat
```

Similarly, to deactivate the environment and return to your main Python executable 

```shell
C:\> <venv>\Scripts\deactivate.bat
```

To reiterate - one may enter the following in command prompt to view the installed libraries

```shell
pip list
```

and the results should show that only `pip` and `setuptools` are the installed packages.

