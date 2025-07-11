# MetaboliteNameStandardization: a system prompt for Llama3.3 to facilitate standardization of metabolite names
# Authors
Willson Tjioe, Katja Danielzik

# Overview
One of the crucial steps in the Metabolomics workflow is the matching of metabolite names provided by a collaborator to database entry, to for example retrieve IDs for downstream analysis.
Although most database APIs accept multiple synonyms, some metabolite names remain unmatched although a human would find the respective entry in the database. Additionally multiple file conversions during the life cycle of a dataset might introduce special characters like "?" that impede recognition by databases.

Large Language models like Llama3.3 have learned which metabolite names are often used in published literature (i.e. metabolite names without special characters) and could assist in the standardization of metabolite names that would otherwise consume a lot of time by hand.

# Modelfile specifications

- build on llama3.3
- 
# Installation and usage in R

``

# Example workflow in R with rollama and RefMet

```{r}
# install and load needed packages
if(!require("rollama")) install_github("JBGruber/rollama")
if (!require("RefMet")) install_github("metabolomicsworkbench/RefMet")
if (!require("dplyr")) install.packages("dplyr")
library(RefMet)
library(rollama)
library(dplyr)
``
# Example usage 
We applied MetaboliteNameStandardization to a set of ??? metabolite names from untargeted LC-MS, which did not contain any lipids and found that the model sufficiently converted ????/??? in a name that could be recognized by the R RefMet API.
Although the temperature and ??? are set to low number, reducing hallucinations of the model, we would suggest to follow this workflow: first feed all your metabolite names into a database, filter for unrecognized ones and only use MetaboliteNameStandardization for the remaining ones.

# Limitations
- Llama3.3 70B requires substential computational capacities, especially a NIVIDA GPU
Llama 3.3 70B Requirements:
GPU Minimum: 24GB VRAM Recommended: NVIDIA GPU with at least 35GB VRAM (e.g., A100 or H100) Optimal setup: Dual NVIDIA RTX 3090 (48GB combined VRAM)
RAM Minimum: 32GB Recommended: 64GB or more, especially for larger datasets
CPU Minimum: 8-core processor

- Problems that we are aware of:
- Nucleotide: sometimes alterations of deoxy Nucleotides (e.g. dATP) to Nucleotides (ATP) or vice versa. As we found that many databases are not consistent either in their IDs regarding these metabolites this behavior might be caused by the database entries. 
