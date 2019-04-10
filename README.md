## Reproducing the paper: Variant analysis of Tetralogy of Fallot

### Overview
This workspace reproduces the work described by Matthieu Miossec and collaborators in the bioRxiv preprint "Deleterious genetic variants in NOTCH1 are a major contributor to the incidence of non-syndromic Tetralogy of Fallot (ToF).” The ToF study is a classic example of research to understand the genetics that underlie a particular phenotype, making it an excellent candidate to illustrate these key points. Additionally, the workspace demonstrates how to reproduce someone else’s work using a cloud-based analysis platform, and serves as a template of best practices for making your own work easily reproducible.

### Summary of original ToF study
By analysing high-throughput exome sequence data from 867 cases and 1252 controls, the authors identified 49 deleterious variants within the NOTCH1 gene that appeared to correspond with the ToF congenital heart disease. Others had previously identified NOTCH1 variants in families with congenital heart defects, including ToF. However the work by Miossec et al. is the first to scale variant analysis of ToF to a cohort of nearly a thousand case samples and show that NOTCH1 is a significant contributor to ToF risk.


- **Paper URL:** https://www.ahajournals.org/doi/abs/10.1161/CIRCRESAHA.118.313250 (This workspace is based on the [preprint](https://www.biorxiv.org/content/early/2018/04/13/300905))
- **Workflows:** see [Tools](https://app.terra.bio/#workspaces/workshop-ashg18/ASHG18-ToF-Reproducible-Paper/tools)
- **Notebook:** [download from Google Cloud Storage](https://storage.cloud.google.com/firecloud-workshops/181017-ashg18/notebooks/cluster_analysis.ipynb)
- **Precomputed results:** [view in Google Cloud Storage](https://console.cloud.google.com/storage/browser/firecloud-workshops/181017-ashg18 )

### Adapting the original experimental design
Our goal was to recapitulate the ToF study results by using similar methodology and data - to run the same analysis on the same data and get the same results. To tackle this, we divide the study into main phases - data input, processing, and analysis - as described by Justin Kitzes et al. in their book, "The Practice of Reproducible Research." We used information provided in the preprint and its supplemental materials, as well as input from the study author, to reconstruct these phases. 

Our guiding principle, where deciding what compromises to use, was to be as close to the original study as possible. To this end 1) basic data processing should be fairly robust to adaptations as long as it is applied the same way to all of the data and 2) the annotations and clustering analysis at the core of the study were critical to reproduce exactly. 

For the **processing phase**, we created a synthetic dataset to get around the lack of appropriate public data to use as input, and applied a variant discovery workflow that we judged equivalent (see guiding principle 1, above).  For the **analysis phase**, we obtained the original scripts and commands from Dr. Miossec (following guiding principle 2, above). With his assistance, we reimplemented the original scripts in two parts: the prediction of variant effects as a workflow in WDL ([Workflow Description Language](http://www.openwdl.org)) and the clustering analysis as R code in a [Jupyter notebook](http://jupyter.org/). We did all the work in the Broad Institute’s open-source analysis platform, FireCloud/Terra. 

#### Note on synthetic dataset creation
We chose not to use the existing read alignments from 1000G due to technical shortcomings of that dataset (file validation errors, insufficient coverage, etc.) and instead opted to generate synthetic sequence data from the Phase 3 callset using the NEAT toolkit. We then introduced the desired variant alleles into reads in those synthetic BAM files using BAMSurgeon. For control samples, we used a synonymous SNP as neutral variant to ensure that the processing applied to controls would mimic cases as closely as possible.

Additional details will be provided in an upcoming revision.

### Results & Discussion
We were able to reproduce the core analysis with our compromises (i.e. synthetic data set, adapted processing workflows). Encouragingly, these changes from the original study had less impact on the results than the size of the synthetic cohort. As the synthetic cohort size approached that of the ToF study, the agreement improved. This underscores the importance of developing community resources to support reproducing this kind of work at the appropriate scale.

In particular, in a test run on a synthetic cohort of 100 samples (of which 8 were case samples), our reimplementation of the original methods was able to identify the spiked-in NOTCH1 variants among the highest scoring hits in the clustering analysis. However, they were not the top hit, which suggests that the test run was statistically underpowered, as is to be expected. Analyzing a larger synthetic cohort of 500 samples, NOTCH-1 rose to the top, reproducing the paper’s results accurately. As we expected, the greater numbers of samples provided sufficient statistical power to reproduce the original results more closely. 

### Detailed contents of this workspace

#### Data
The workspace model contains precomputed per-sample output files for two synthetic participant_sets derived from participants in the 1000 Genomes project : Test_cohort_A100 with 100 samples and Test_cohort-B500 with 500 samples. The workflows take a long time to run on the larger set, but the larger numbers provide greater statistical muscle and agreement with the original study (noted for its large cohort size). 
The workspace data model also contains IDs for all 3,500 participants of the 1000 Genomes Project, the foundation of our synthetic data. The data are stored in an external Google bucket that is fully public. The data sharing policy has yet to be defined formally, but will be largely inherited from the progenitor dataset, the 1000 Genomes Project.

#### Workspace attributes
All required and optional references and resources for the methods are included in the Workspace Attributes section at the bottom of this page. Of note for this workspace: we use the reference genome from the original study, hg19 (aka GRCh37, or b37), to ensure the most accurate reproduction.

#### Methods
We provide preset method configurations for the following workflows:

| Name | Entity Type| Synopsis |
|---|---|---|
| Collect-1000G-participant | participant | Collect the variants for a single participant from 1000G Phase 3 |
| Generate-synthetic-reads | participant | Generate synthetic read data based on intervals and a VCF of variants |
| Mutate-reads-with-BAMSurgeon | participant | Introduce specific mutations into an analysis-ready BAM file with BAMSurgeon |
| Call-single-sample-GVCF-GATK4 | participant | Call variants per-sample and produce a GVCF file with GATK4 HaplotypeCaller |
| Joint-call-and-hard-filter-GATK4 | participant_set | Apply joint variant discovery analysis and hard filtering |
| Predict-variant-effects-GEMINI | participant_set | Predict and annotate the functional effects of variants using SnpEff and GEMINI

##### Note on software versions
Throughout this project, we strove to apply the same versions of all the tools used in the original study. For the GATK variant calling portion, however, it is technically much more straightforward to work with very recent versions and avoid redeveloping a lot of pipeline code available in FireCloud/Terra. Based on our team's experience with GATK we estimated that the results should be functionally equivalent for the purposes of this project. The rest of the tools used the same versions as in the paper, including the corresponding WDL workflow and Jupyter Notebook, which were developed from scratch. The table below lists the tools and their version.

| Tool Name | Version  |
|---|---|
| GATK | 4.0.9.0 |
| SNPEff | 4.0e |
| Gemini | 0.18.2 |
| CADD | 1.3 |

###Notebooks - Clustering Analysis###
This workspace contains Jupyter notebooks that reproduce the clustering analysis at the core of the ToF study results. Annotations explain each analysis step and the cell blocks contain the code. The notebooks contained in the Featured Workspace are set to run on the 100-sample cohort, and running the notebook will identify the NOTCH-1 gene as a candidate for deleterious variants responsible for ToF, but not number one. Changing to the 500-sample cohort (by editing the code block in the notebook) delivers the analysis from the original paper, with NOTCH-1 the top candidate. Thus the critical clustering analysis reveals how boosting the statistical power of the synthetic sample set allows us to more closely emulate the ToF study.

##Best Practices for “reproducible” research ##
Several features within this workspace enabled us to reproduce the ToF study. Incorporating these will help others reproduce and validate your own work precisely and exactly.

- **Version-controlled code** Capturing the version used for processing and analysing data is critical for exact reproducibility.

- **Open data** Open -access data is of course the first choice for a reproducible study. If that option isn’t available, as in this case, creating an equivalent synthetic data can make the study reproducible.

- **Automation** Both batch processing (WDLs) and interactive analysis (in Jupyter notebooks) can be automated , with shared code to ensure consistency throughout the processing and analysis stages .

- **Built-in documentation** Comments in Jupyter notebooks are a convenient way to ensure that your analysis process is transparent and understandable. 

- **Free and open tools** Similar in theme to open data, using free and open tools ensures that others can use the same tools for reproducing your work.


### Credits and acknowledgments
This workspace was developed for an ASHG2018 Invited Workshop. The work was carried out as a collaboration between our team in the Data Sciences Platform at the Broad Institute in Cambridge, MA, USA and Dr. Matthieu J. Miossec of the Center for Bioinformatics and Integrative Biology (CBB) at Universidad Andrés Bello, Santiago, Chile.

### License
#### Copyright Broad Institute, 2018 | BSD-3
All code provided in this workspace is released under the WDL open source code license (BSD-3) (full license text at https://github.com/openwdl/wdl/blob/master/LICENSE). Note however that the programs called by the scripts may be subject to different licenses. Users are responsible for checking that they are authorized to run all programs before running these tools.
