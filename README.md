# Welcome this is my first tutorial using Kedro
- [Kedro Concepts](#kedro-concepts)
- [Kedro Spaceflights](#kedro-spaceflights)
  - [Overview](#overview)
  - [Rules and guidelines](#rules-and-guidelines)
  - [Set up the spaceflights tutorial project](#set-up-the-spaceflights-tutorial-project)
  - [How to install dependencies](#how-to-install-dependencies)
  - [How to run your Kedro pipeline](#how-to-run-your-kedro-pipeline)
  - [How to test your Kedro project](#how-to-test-your-kedro-project)
  - [Project dependencies](#project-dependencies)
  - [How to work with Kedro and notebooks](#how-to-work-with-kedro-and-notebooks)
    - [Jupyter](#jupyter)
    - [JupyterLab](#jupyterlab)
    - [IPython](#ipython)
    - [How to convert notebook cells to nodes in a Kedro project](#how-to-convert-notebook-cells-to-nodes-in-a-kedro-project)
    - [How to ignore notebook output cells in `git`](#how-to-ignore-notebook-output-cells-in-git)
  - [Package an entire Kedro project](#package-an-entire-kedro-project)
  - [Add documentation to your project](#add-documentation-to-your-project)
    - [Set up the Sphinx project files](#set-up-the-sphinx-project-files)
    - [Build HTML documentation](#build-html-documentation)
    - [Documentation from docstrings](#documentation-from-docstrings)
  - [Package your project](#package-your-project)
    - [Package recipients](#package-recipients)
    - [Docker, Airflow and other deployment targets](#docker-airflow-and-other-deployment-targets)

# Kedro concepts

It is time to introduce the most basic elements of Kedro. You can find further information about these and more advanced Kedro concepts in the [Kedro glossary](https://docs.kedro.org/en/stable/resources/glossary.html).

## Nodes
In Kedro, a node is a wrapper for a pure Python function that names the inputs and outputs of that function. Nodes are the building block of a pipeline, and the output of one node can be the input of another.

Here are two simple nodes as an example:
```
from kedro.pipeline import node

# First node
def return_greeting():
    return "Hello"


return_greeting_node = node(func=return_greeting, inputs=None, outputs="my_salutation")

# Second node
def join_statements(greeting):
    return f"{greeting} Kedro!"


join_statements_node = node(
    join_statements, inputs="my_salutation", outputs="my_message"
)
```

## Pipelines

A pipeline organises the dependencies and execution order of a collection of nodes and connects inputs and outputs while keeping your code modular. The pipeline determines the **node execution order** by resolving dependencies and does not necessarily run the nodes in the order in which they are passed in.

Here is a pipeline comprised of the nodes shown above:

```
from kedro.pipeline import pipeline

# Assemble nodes into a pipeline
greeting_pipeline = pipeline([return_greeting_node, join_statements_node])
```

## Data Catalog

The Kedro Data Catalog is the registry of all data sources that the project can use to manage loading and saving data. It maps the names of node inputs and outputs as keys in a `DataCatalog`, a Kedro class that can be specialised for different types of data storage.

[Kedro provides numerous different built-in datasets](https://docs.kedro.org/en/stable/kedro.datasets.htmlhttps://docs.kedro.org/en/stable/kedro.datasets.html) for various file types and file systems, so you don’t have to write the logic for reading/writing data.

## Kedro project directory structure

Kedro projects follow a default template that uses specific folders to store datasets, notebooks, configuration and source code. We advise you to retain the structure to make it easy to share your projects with other Kedro users, but you can adapt the folder structure if you need to.

A standard Kedro project structure is as follows:
```
project-dir         # Parent directory of the template
├── .gitignore      # Hidden file that prevents staging of unnecessary files to `git`
├── conf            # Project configuration files
├── data            # Local project data (not committed to version control)
├── docs            # Project documentation
├── logs            # Project output logs (not committed to version control)
├── notebooks       # Project-related Jupyter notebooks (can be used for experimental code before moving the code to src)
├── pyproject.toml  # Identifies the project root and [contains configuration information](../faq/architecture_overview.md#kedro-project)
├── README.md       # Project README
├── setup.cfg       # Configuration options for `pytest` when doing `kedro test` and for the `isort` utility when doing `kedro lint`
└── src             # Project source code
```

`conf`

The `conf` folder contains two subfolders for storing configuration information: `base` and `local`.

`conf/base`

Use the `base` subfolder for project-specific settings to share across different installations (for example, with other users).

The folder contains three files for the example, but you can add others as you require:

* `catalog.yml` - [Configures the Data Catalog](https://docs.kedro.org/en/stable/data/data_catalog.html#use-the-data-catalog-within-kedro-configuration) with the file paths and load/save configuration needed for different datasets

* `logging.yml` - Uses Python’s default `logging` library to set up logging

* `parameters.yml` - Allows you to define parameters for machine learning experiments, for example, train/test split and the number of iterations

`conf/local`

The `local` subfolder is specific to each user and installation and its contents is ignored by `git` (through inclusion in `.gitignore`).

Use the `local` subfolder for **settings that should not be shared**, such as **access credentials**, **custom editor configuration**, **personal IDE configuration** and **other sensitive or personal content**.

By default, Kedro creates one file, `credentials.yml`, in `conf/local`.

`data`

The `data` folder contains multiple subfolders to store project data. We recommend you put `raw` data into raw and move processed data to other subfolders according to the [data engineering convention](https://docs.kedro.org/en/stable/faq/faq.html#what-is-data-engineering-convention).

`src`

This subfolder contains the project’s source code in one subfolder and another folder that you can use to add unit tests for your project. Projects are preconfigured to run tests using `pytest` when you call `kedro test` from the project’s root directory.


# Kedro Spaceflights

Kedro spaceflights tutorial : https://kedro.readthedocs.io/en/stable/tutorial/spaceflights_tutorial.html

## Overview

This is your new Kedro project, which was generated using `Kedro 0.18.3`.

Take a look at the [Kedro documentation](https://kedro.readthedocs.io) to get started.

## Rules and guidelines

In order to get the best out of the template:

* Don't remove any lines from the `.gitignore` file we provide
* Make sure your results can be reproduced by following a [data engineering convention](https://kedro.readthedocs.io/en/stable/faq/faq.html#what-is-data-engineering-convention)
* Don't commit data to your repository
* Don't commit any credentials or your local configuration to your repository. Keep all your credentials and local configuration in `conf/local/`

## Set up the spaceflights tutorial project

The setup steps are as follows:

1. Create a new project with `kedro new`

2. Install project dependencies with `pip install -r src/requirements.txt`

3. Configure the following in the `conf` folder:

* Credentials and any other sensitive information

* Logging

## How to install dependencies

Declare any dependencies in `src/requirements.txt` for `pip` installation and `src/environment.yml` for `conda` installation.

To install them, run:

```
pip install -r src/requirements.txt
```

## How to run your Kedro pipeline

You can run your Kedro project with:

```
kedro run
```

## How to test your Kedro project

Have a look at the file `src/tests/test_run.py` for instructions on how to write your tests. You can run your tests as follows:

```
kedro test
```

To configure the coverage threshold, go to the `.coveragerc` file.

## Project dependencies

To generate or update the dependency requirements for your project:

```
kedro build-reqs
```

This will `pip-compile` the contents of `src/requirements.txt` into a new file `src/requirements.lock`. You can see the output of the resolution by opening `src/requirements.lock`.

After this, if you'd like to update your project requirements, please update `src/requirements.txt` and re-run `kedro build-reqs`.

[Further information about project dependencies](https://kedro.readthedocs.io/en/stable/kedro_project_setup/dependencies.html#project-specific-dependencies)

## How to work with Kedro and notebooks

> Note: Using `kedro jupyter` or `kedro ipython` to run your notebook provides these variables in scope: `context`, `catalog`, and `startup_error`.
>
> Jupyter, JupyterLab, and IPython are already included in the project requirements by default, so once you have run `pip install -r src/requirements.txt` you will not need to take any extra steps before you use them.

### Jupyter
To use Jupyter notebooks in your Kedro project, you need to install Jupyter:

```
pip install jupyter
```

After installing Jupyter, you can start a local notebook server:

```
kedro jupyter notebook
```

### JupyterLab
To use JupyterLab, you need to install it:

```
pip install jupyterlab
```

You can also start JupyterLab:

```
kedro jupyter lab
```

### IPython
And if you want to run an IPython session:

```
kedro ipython
```

### How to convert notebook cells to nodes in a Kedro project
You can move notebook code over into a Kedro project structure using a mixture of [cell tagging](https://jupyter-notebook.readthedocs.io/en/stable/changelog.html#release-5-0-0) and Kedro CLI commands.

By adding the `node` tag to a cell and running the command below, the cell's source code will be copied over to a Python file within `src/<package_name>/nodes/`:

```
kedro jupyter convert <filepath_to_my_notebook>
```
> *Note:* The name of the Python file matches the name of the original notebook.

Alternatively, you may want to transform all your notebooks in one go. Run the following command to convert all notebook files found in the project root directory and under any of its sub-folders:

```
kedro jupyter convert --all
```

### How to ignore notebook output cells in `git`
To automatically strip out all output cell contents before committing to `git`, you can run `kedro activate-nbstripout`. This will add a hook in `.git/config` which will run `nbstripout` before anything is committed to `git`.

> *Note:* Your output cells will be retained locally.

## Package an entire Kedro project

This section explains how to build your project documentation, and how to bundle your entire project into a Python package.

Kedro also has an advanced feature which supports packaging on a pipeline level allowing you share and reuse pipelines across projects! To read more about this please look at the [section on micro-packaging](../nodes_and_pipelines/micro_packaging.md).

## Add documentation to your project

There are several documentation frameworks for Python projects. This section describes how to use [Sphinx](https://www.sphinx-doc.org) to build the documentation of your Kedro project.

To install Sphinx, run the following:

```bash
pip install sphinx
```

### Set up the Sphinx project files

```{warning}
Currently, Kedro projects are created with a `docs/source` subdirectory, which gets pre-populated with two Sphinx configuration files (`conf.py`, and `index.rst`), needed by the `kedro build-docs` command. This command is deprecated; it will be removed in Kedro version 0.19, along with those dummy files.

Before proceeding with these instructions, back up the contents of `docs/source/index.rst` and remove both `docs/source/conf.py` and `docs/source/index.rst`.
```

First, run the following command:

```bash
sphinx-quickstart docs
```

Sphinx will ask a series of configuration questions. The first is as follows:

```text
You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path,
or you separate "source" and "build" directories within the root path.

> Separate source and build directories (y/n)? [n]:
```

Select `y` to separate the build files from the source files, and enter any additional information that Sphinx requests such as the project name and the documentation language, which defaults to English.

### Build HTML documentation

```{warning}
If you previously backed up the contents of `index.rst`, restore them before proceeding.
```

After the quickstart process is complete, you can build the documentation by **navigating to the `docs` directory** and running the following:

```bash
make html
```

Your project documentation will be written to the `docs/build/html` directory.

You may want to add project-specific Markdown documentation within the `docs/source` folder of your Kedro project. To be able to build it, follow the [introduction instructions of MyST-Parser](https://myst-parser.readthedocs.io/en/stable/intro.html) and update your `docs/source/index.rst` file to add the markdown files to the table of contents.

### Documentation from docstrings
If you wish to add documentation built from [`docstrings`](https://datacamp.com/community/tutorials/docstrings-python) within your project, you need to make some changes to the Sphinx configuration files found in the `docs/source` directory to use [automatic documentation generation from code](https://www.sphinx-doc.org/en/master/tutorial/automatic-doc-generation.html).

In `conf.py`, add the following to ensure that the `sphinx.ext.autodoc` and `sphinx.ext.autosummary` extensions are specified, and `autosummary_generate` is enabled:

```python
extensions = ["sphinx.ext.autodoc", "sphinx.ext.autosummary"]
autosummary_generate = True
```

Finally, to ensure that you include the autodoc modules in your build, run the following command once **from the `docs` folder**:

```bash
sphinx-apidoc --module-first -o source ../src/<project_name>

```

This will generate a `docs/src/modules.rst` file, as well as other files containing references to your docstrings. To include those in your documentation, make sure your `docs/src/index.rst` has a `modules` entry in the table of contents:

```text
.. toctree::

   modules
```

**From the `docs` folder** run the following:

```text
pip install -e ../src
```

Finally, **from the `docs folder`**, run this command to build a full set of documentation that automatically includes docstrings:

```text
make html
```

```{note}
Consult the Sphinx project documentation for [additional options to pass to `sphinx-build`](https://www.sphinx-doc.org/en/master/man/sphinx-build.html). To customise your documentation beyond the basic template, you'll need to adjust the [Sphinx configuration settings](https://www.sphinx-doc.org/en/master/usage/configuration.html) which are stored in `docs/source/conf.py` file.
```

## Package your project

To package your project, run the following in your project root directory:

```bash
kedro package
```

Kedro builds the package into the `dist` folder of your project, and creates one `.egg` file and one `.whl` file, which are [Python packaging formats for binary distribution](https://packaging.python.org/).

The resulting package only contains the Python source code of your Kedro pipeline, not any of the `conf`, `data` and `logs` subfolders. This means that you can distribute the project to run elsewhere, such as on a separate computer with different configuration information, dataset and logging locations.

We recommend that you document the configuration required (parameters and catalog) in the local `README.md` file for any project recipients.

### Package recipients

Recipients of the `.egg` and `.whl` files need to have Python and `pip` on their machines, but do not need to have Kedro installed.

A recipient can install the project by calling:

```bash
pip install <path-to-wheel-file>
```

An executable, `kedro-tutorial`, is placed in the `bin` subfolder of the Python install folder, so the project can be run as follows:

```bash
python -m kedro_tutorial
```

```{note}
The recipient will need to add a `conf` subfolder. They also need to add `data` and `logs` if the pipeline loads/saves local data or uses logging.
```

Once your project is installed, to run your pipelines from any Python code, simply import it:

```python
from kedro_tutorial.__main__ import main

main(
    ["--pipeline", "__default__"]
)  # or simply main() if you don't want to provide any arguments
```

This is equivalent to running `kedro run`, and you can provide all the parameters described by `kedro run --help`.

### Docker, Airflow and other deployment targets

There are various methods to deploy packaged pipelines via Kedro plugins:

* [Kedro-Docker](https://github.com/kedro-org/kedro-plugins/tree/main/kedro-docker) plugin for packaging and shipping Kedro projects within [Docker](https://www.docker.com/) containers.
* [Kedro-Airflow](https://github.com/kedro-org/kedro-plugins/tree/main/kedro-airflow) to convert your Kedro project into an [Airflow](https://airflow.apache.org/) project.
* The [Deployment guide](../deployment/deployment_guide) touches on other deployment targets such as AWS Batch and Prefect, and there is a [range of third-party plugins for deployment](extend_kedro/plugins.md#community-developed-plugins).
