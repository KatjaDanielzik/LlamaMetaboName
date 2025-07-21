# LlamaMetaboName: a system prompt for Llama3.3 to facilitate standardization of metabolite names

Llama 3.3 is licensed under the Llama 3.3 Community License, Copyright © Meta Platforms, Inc. All Rights Reserved.

# Overview
One of the crucial steps in the Metabolomics workflow is the matching of metabolite names, provided by a collaborator, to database entries, to for example retrieve IDs for downstream analysis.
Although most database APIs accept multiple synonyms, some metabolite names remain unmatched although a human would find the respective entry in the database. Additionally multiple file conversions during the life cycle of a dataset might introduce special characters like "?" that impede recognition by databases.

Large Language models like Llama3.3 have learned which metabolite names are often used in published literature (i.e. metabolite names without special characters) and could assist in the standardization of metabolite names that would otherwise consume a lot of time by hand.

## Example specifications in modelfile
While the system prompt itself is designed to be generalizable for other metabolite names the statements under **Additional information** were tailored to our example set of metabolite names.
**Additional information** refers to the examples that are listed at the bottom of the model file in the format **MESSAGE user metabolite name** **MESSAGE assistant standardized name**.
Other datasets may need adjustment of the additional informations and/or examples. 

# Modelfile specifications

System prompt and parameter settings for llama3.3 70B.

Llama 3.3 70B Requirements:
- GPU Minimum: 24GB VRAM Recommended: NVIDIA GPU with at least 35GB VRAM (e.g., A100 or H100) Optimal setup: Dual NVIDIA RTX 3090 (48GB combined VRAM)
- RAM Minimum: 32GB Recommended: 64GB or more, especially for larger datasets
- CPU Minimum: 8-core processor
 
# Install ollama
```{bash}
# install ollama
curl -fsSL https://ollama.com/install.sh | sh

# build model from modelfile
ollama create MetaboNameStandard --file ~/<path>/MetaboNameStandard.modelfile

# check that model was created
ollama list
```

# Example workflow in R with rollama (ollama API) and RefMet
```{r}
# install and load needed packages
if(!require("rollama")) install_github("JBGruber/rollama")
if (!require("RefMet")) install_github("metabolomicsworkbench/RefMet")
if (!require("dplyr")) install.packages("dplyr")
library(RefMet)
library(rollama)
library(dplyr)

# load data (stored as dataframe with "metabolite" specifying metabolite names)
data <- read.csv("path to folder")

# hand raw metabolite names to RefMet
refmet_output <- RefMet::refmet_map_df(data$metabolite)

# filter for missing metabolite names
llm_standardized <- refmet_output%>%filter(Standardized.name=="-")

# using MetaboNameStandard
## Selecting model
rollama::options(rollama_model = "MetaboNameStandard")

# Build prompt query
queries <- rollama::make_query(
              text = non_standardized$Input.name # unrecognized metabolite names
              prompt = "Only output the standardized metabolite name without any explanation.")

# run query and store query results as new column
llm_standardized$LLM_standardized <- rollama::query(queries,
                                                      screen = FALSE, 
                                                      output = "text"
                                                        )

# Print results
llm_standardized

# hand over standardized names to RefMet
refmet_LLM_standardized <- refmet_map_df(llm_standardized$LLM_standardized)
```

## Example output with comments
|Raw metabolite name| LlamaMetaboName output | Refmet Entry | Comment |
|---|---|---|---|
|Isocitric acid|Isocitrate|Isocitric acid| -acid/ -ate dilemma|
|5?-Deoxy-5?-(methylthio)adenosine |5'-Deoxy-5'-(methylthio)adenosine|5'-Methylthioadenosine| Removal of question marks (?)|
N'-Acetyl-L-glutamine (TL_regress)|N-Acetylglutamine|N-Acetylglutamine | processing comments (in brackets) removed|
|N-CARBAMOYL-DL-ASPARTIC ACID |N-Carbamoyl-aspartic acid|N-Carbamoylaspartic acid|upper case -> lower case for consistent capitalization|
|UDP| UDPGlucose|UDP-glucose|Hallucination: 2 different metabolite (UDP vs UDP-glucose)|
|TTP|dTTP|dTTP| Hallucination: Nucleotide -> deoxy-Nucleotide|
|N'-Acetyl-L-glutamine|N-Acetylglutamine|N-Acetylglutamine| removal of Apostrophe (')|
|N_N_N-Trimethyllysine|Trimethyllysine|N-6-Trimethyllysine| removal of underscore|
|Fructose(26)bisphosphate|Fructose-2,6-bisphosphate|Fructose 2,6-bisphosphate| added missing comma and hyphen|
|1;7-Dimethylxanthine|1,7-Dimethylxanthine|Paraxanthine| conversion of semicolon to comma|
|Flavin adenine dinucleotide | FAD | FAD | correctly assigned abbrevation|

# Example usage 
We applied MetaboliteNameStandardization to a set of 470 metabolite names from untargeted LC-MS, which did **not contain any lipids** and found that the model sufficiently converted 450/470 in a name that could be recognized by the R RefMet API.
Although the temperature, top_k and top_p and  are set to zero, reducing hallucinations of the model, we would suggest to **follow this workflow:**

First feed all your metabolite names into a database, filter for unrecognized ones and only use LlamaMetaboName for the remaining ones. This also helps to reduce the enviromental burden as less tokens are generated. 

# Known model behaviors
Preferable:
- Works well for standardizing punctuations formats (e.g. spacing, capitalization, hyphenation, symbols)
- Returns only one standardized metabolite per input row
- Returns a vector 
- Removes additional prefixes

Limitations:
- often hallucinates UDP -> UDP-Glucose. UDP and UDP-Glucose are very different metabolites (structure and function)
- LlamaMetaboName nearly always transforms the suffix -ate to -acid. Metabolites with transformed suffixes are still mostly recognized by the database RefMet(via RefMet API in R)
- LlamaMetaboName frequently converts abbrevated nucleotides (e.g. ATP, GTP, CTP, TTP) to the abbrevations of there deoxygenated forms (e.g. dATP, dGTP, dCTP, dTTP). This happens especially for TTP. But we found that many databases also not sufficiently discriminate between these two forms of nucleotides. 
