# cpip

## What is it?
This tool builds a `conda` environment with `pip` dependencies together
in a way that allows for cross-distro/cross-version compatibility.

They are two ways to use `cpip`:

- `cpip pack` will package the environment into a portable tarball.
- `cpip create` will simply create a ready-to-use environment.

The advantage of having a `create` command gives the user the option
to create the environment in the exact same way as the one that will
be deployed via the `pack` command.

## How does it work?
1. Assembles an environment with basic `conda` commands from
given `conda` environment files
1. Installs cross-compilers and downloads/compiles `pip` packages
1. Packages `conda` environment via `conda-pack` tool
1. Unpacks environment to desired location (only for `create` command)

## Supported Operating Systems
- Linux
- macOS

## System Requirements
- [conda](https://conda.io/docs/) (version 4.6 or higher)
- [poetry](https://poetry.eustace.io/docs/) (optional)

## Environment Dependencies
- [conda-pack](https://conda.github.io/conda-pack/) (installed via `conda`)
- [jq](https://stedolan.github.io/jq/) (installed via `conda`)
- [python](https://www.python.org/) (installed via `conda`)

## Usage

### 1) Add cpip to your PATH environment variable

The command below will add `cpip` to your `PATH` environment variable;
make sure to substitute `<cpip_repository>` for the absolute path
of the `cpip` repository.

`export PATH=<cpip_repository>/bin>:$PATH`

To keep this configuration every time you open a new shell,
make sure to add this command to your shell configuration file.
- for login `bash` shells this will be `~/.bash_profile`
- for non-login `bash` shells this will be `~/.bashrc`

### 2) Update Conda
`conda update -n base -c defaults conda`

### 3) Update Terminal Environment
`conda init` + restart terminal

### 4) Create Build Environment
`conda env create` from base of `cpip` directory

### 5) Activate Build Environment
`conda activate cpip`

### 6) Run 'cpip pack' or 'cpip create'
For more information: `cpip pack --help` or `cpip create --help`

### 7) Unpack and Activate
    Initial Setup (if using 'cpip pack'):
      1. untar archive        --> tar -xf <archive>
      2. activate environment --> source <env-root>/bin/activate
      3. fix path prefixes    --> conda-unpack
    
    Normal Use:
      *  activate             --> source <env-root>/bin/activate
      *  deactivate           --> source deactivate

    Info:
      *  conda dependencies     @ <env-root>/dependencies/<env-name>.yml
      *  poetry lockfile        @ <env-root>/dependencies/poetry.lock

**NOTE:** It's possible to change the path of `<env-root>` after
untarring, as long that is done prior to running `conda-unpack`.

## Cleaning Caches
Use `cpip clean` command to clean the `conda` and/or `pip` caches

## Other Things to Note

### Installing pip dependencies with Conda vs. Poetry
`pip` dependencies can be installed via `conda` or `poetry`.
It is recommended to use a `poetry` project to define all the `pip`
packages in your project; this allows for lockfiles to pin package
versions properly. If you decide to use `poetry`, it makes sense to
move ALL `pip` dependencies from the `conda` environment file(s) to
a `poetry` project file.

### Cross-Compilers
Cross-compilers will be installed if any `pip` dependencies are
detected, or the `--command` option is used. They will always be
uninstalled automatically before the environment is finalized.

The cross-compilers used are Anaconda compiler tools
(see [here](https://conda.io/docs/user-guide/tasks/build-packages/compiler-tools.html)).
They are not true cross-compilers since they are not configured to work cross-platform.
However, they are configured to give a high degree of compatibility from one
distro/version to another.

**NOTE:** If any Anaconda compiler tools or associated runtime libraries are
present in the environment file(s), they will be removed completely. Only the
compiler runtime libraries will be reinstalled, and potentially of different
versions than before.

### Running Commands in the Newly Created Environment
Immediately after the environment is created, it may be desired to run
certain commands in the active environment. To do this, we can use the
`--command` option as many times as desired to pass in commands that we
want to run while the environment is active. `cpip` will ensure that
cross-compilers are installed in the environment, so that the build
environment for running commands is consistent with the build environment
for compiling `pip` packages.

The following are internal environment variables available to the commands:
- `CPIP_CONDA_LOCKFILE`   <-- path to `conda` lockfile
- `CPIP_POETRY_LOCKFILE`  <-- path to `poetry` lockfile

### Order of Environments
This tool can take more than one `conda` environment file.
These files will be loaded in the order they are given on
the command line. Different command line orders may produce
slightly different environments.

### Conda Configuration
Changing certain configuration values may yield unexpected results.

### Poetry Configuration
`cpip pack` internally sets `settings.virtualenvs.create` to `false`,
but restores the setting to its original value upon failure or completion.

### Concurrency
Running more than one `cpip` command simultaneously is not supported.

### TODO: Non-Unix Systems
For systems that don't respect the `XDG_CACHE_HOME` environment variable,
Poetry may have concurrency issues, however, if using a `poetry.lock` file,
this will relieve any potential cache conflicts.

The cache issues can be solved completely if `poetry` allows for the
something more flexible like a `--poetry-cache-dir` command line option.
