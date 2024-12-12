# Getting Started with BluelightAI Cobalt

## What is Cobalt?

Cobalt is a toolbox that helps you diagnose and repair problems with AI models. It uses Topological Data Analysis (TDA) to uncover patterns in the ways models transform their input data into output data. You can use Cobalt with any type of model or data, as long as you can produce vector embeddings of that data. Cobalt uses these to build representations of your data and reveal ways the model fails.

## How can I access Cobalt within the DTC SageMaker environment?

Cobalt is pre-installed into a dedicated `conda` environment, `cobalt-env`, as part of the default SageMaker lifecycle configuration script. Run `conda env list` from the SageMaker terminal to verify that the environment has been installed. You can easily access this environment within JupyterLab Notebooks by selecting “Python 3 (cobalt-env)” from the Select Kernel dropdown, located at the top-right of each JupyterLab Notebook.

## How can I learn more about using Cobalt?

Please refer to the [Cobalt documentation](https://docs.cobalt.bluelightai.com/index.html) for a complete guide. The [Concept Overview](https://docs.cobalt.bluelightai.com/concepts.html) section is a great place to start to gain a high-level understanding of Cobalt use cases and concepts.

