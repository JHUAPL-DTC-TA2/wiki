# Accessible Domains and Websites

The DTC AWS environment uses an outbound network firewall allowlist. The domains below are allowed for HTTP and HTTPS traffic by the current firewall rules.

Entries shown as `*.example.com` represent domain suffix rules in the allowlist. If a website depends on assets, APIs, authentication services, or downloads from domains that are not listed here, parts of that site may not load from inside the environment.

## AWS and SageMaker

| Category | Accessible domains and websites | Typical use |
|---|---|---|
| AWS services | `*.amazonaws.com`, `*.aws.amazon.com` | AWS service endpoints and AWS documentation sites |
| SageMaker and AWS web apps | `*.sagemaker.aws`, `*.awsapps.com`, `*.aws.dev` | SageMaker Studio, AWS-hosted applications, and AWS development tools |
| AWS-hosted static content | `*.cloudfront.net`, `*.awsstatic.com` | Static assets and content delivery for AWS services |

## Operating Systems and Platform Updates

| Category | Accessible domains and websites | Typical use |
|---|---|---|
| Microsoft and Windows | `*.microsoft.com`, `*.windowsupdate.com`, `*.windows.net` | Microsoft services, Windows updates, and Microsoft-hosted platform content |
| Ubuntu and Canonical | `*.ubuntu.com`, `*.canonical.com` | Ubuntu package repositories and Canonical resources |
| Amazon Linux | `*.amazonlinux.com` | Amazon Linux package repositories and updates |

## Development Tools and Editors

| Category | Accessible domains and websites | Typical use |
|---|---|---|
| Visual Studio and VS Code | `*.visualstudio.com`, `code.visualstudio.com`, `update.code.visualstudio.com`, `marketplace.visualstudio.com`, `download.visualstudio.microsoft.com`, `vscode.download.prss.microsoft.com`, `vscode.blob.core.windows.net` | VS Code downloads, updates, extensions, and Visual Studio services |
| VS Code extension assets | `*.vsassets.io`, `*.gallerycdn.vsassets.io`, `*.gallery.vsassets.io`, `*.vscode-unpkg.net` | VS Code Marketplace extension metadata and asset delivery |
| OpenVSX | `*.open-vsx.org`, `openvsx.eclipsecontent.org`, `openvsxorg.blob.core.windows.net` | OpenVSX extension registry and extension downloads |
| JetBrains | `*.jetbrains.com` | JetBrains IDE downloads, updates, plugins, and documentation |
| Docker | `*.docker.com` | Docker documentation, tools, and downloads |

## Package Repositories and Language Ecosystems

| Category | Accessible domains and websites | Typical use |
|---|---|---|
| Python and PyPI | `*.python.org`, `*.pypi.org`, `*.pythonhosted.org` | Python documentation, PyPI package metadata, and Python package downloads |
| Anaconda | `*.anaconda.com`, `*.anaconda.org` | Conda package repositories, Anaconda tools, and documentation |
| R and RStudio | `*.r-project.org`, `*.rstudio.org` | R project resources and RStudio resources |
| JavaScript packages | `*.unpkg.com`, `*.jsdelivr.net` | JavaScript package CDN access |

## Machine Learning and Scientific Computing

| Category | Accessible domains and websites | Typical use |
|---|---|---|
| PyTorch | `*.pytorch.org` | PyTorch documentation, wheels, and package indexes |
| Hugging Face | `*.huggingface.co` | Hugging Face models, datasets, documentation, and package resources |
| NVIDIA | `*.nvidia.com` | NVIDIA documentation, CUDA resources, and GPU software downloads |
| MathWorks | `*.mathworks.com` | MATLAB and MathWorks documentation, downloads, and support resources |

## Notes

- The firewall rules are based on HTTP host headers and TLS Server Name Indication (SNI) for ports 80 and 443.
- DNS names, package mirrors, and download hosts can change over time. Contact the JHU APL team if a challenge-related website or package repository is blocked.
