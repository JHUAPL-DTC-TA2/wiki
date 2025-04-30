# Getting Started with BluelightAI Cobalt

## What is Cobalt?

Cobalt is a toolbox that helps you diagnose and repair problems with AI models. It uses Topological Data Analysis (TDA) to uncover patterns in the ways models transform their input data into output data. You can use Cobalt with any type of model or data, as long as you can produce vector embeddings of that data. Cobalt uses these to build representations of your data and reveal ways the model fails.

## How can I access Cobalt within the DTC SageMaker environment?

Cobalt is pre-installed into a dedicated `conda` environment, `cobalt-env`, as part of the default SageMaker Lifecycle Configuration (LCC) script. Run `conda env list` from the SageMaker terminal to verify that the environment has been installed. If the environment is not available, you may need to restart your SageMaker space, making sure it runs with the Lifecycle Configuration script containing the text "cobalt"; this script is what performs the installation and setup. It will not disturb your existing `conda` environments.

You can access this environment within JupyterLab Notebooks by selecting “Python 3 (cobalt-env)” from the Select Kernel dropdown, located at the top-right of each JupyterLab Notebook.

## How can I learn more about using Cobalt?

We recommend the example notebooks as a great place to start. These are placed at `~/.config/cobalt/cobalt-notebooks.tar.gz` by the LCC script mentioned above.

Please refer to the [Cobalt documentation](https://docs.cobalt.bluelightai.com/index.html) for a complete guide. The [Concept Overview](https://docs.cobalt.bluelightai.com/concepts.html) section is a good starting point to gain a high-level understanding of Cobalt use cases and concepts.

