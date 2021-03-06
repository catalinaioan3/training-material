---
layout: tutorial_hands_on

title: Label-free data analysis using MaxQuant
zenodo_link: 'https://zenodo.org/record/3774452'
level: Introductory
questions:
- How to perform label-free analysis in Maxquant? 
- Which are the most abundant proteins in serum?
- How successful was the depletion of those in our experiment? 
objectives:
- Analysis of human serum proteome samples with label-free quantification in MaxQuant
time_estimation: 1H
key_points:
- MaxQuant offers a single tool solution for protein identification and quantification.
- Label-free quantitation reveals the most abundant proteins in serum samples. 
contributors:
- foellmelanie
- matthias313

---


# Introduction
{:.no_toc}

The proteome refers to the entirety of proteins in a biological system (e.g cell, tissue, organism). Proteomics is the large-scale experimental analysis of proteins and proteomes, most often performed by mass spectrometry that enables great sensitivity and throughput. Especially for complex protein mixtures, bottom-up mass spectrometry is the standard approach. In bottom-up proteomics, proteins are digested with a specific protease into peptides and the measured peptides are *in silico* reassembled into the corresponding proteins. Inside the mass spectrometer, not only the peptides are measured (MS1 level), but the peptides are also fragmented into smaller peptides which are measured again (MS2 level). This is referred to as tandem-mass spectrometry (MS/MS). Identification of peptides is performed by peptide spectrum matching of the theoretical spectra generated from the input protein database (fasta file) with the measured spectra. Peptide quantification is most often performed by measuring the area under the curve of the MS1 level peptide peaks, but special techniques such as TMT allow to quantify peptides on MS2 level. Nowadays, bottom-up tandem-mass spectrometry approaches allow for the identification and quantification of several thousand proteins.

![MQ_lcmsms](../../images/maxquant_lfq_lcmsms.png "Proteomics using liquid chromatography tandem-mass spectrometry (LC-MS/MS). Adapted from <a href="https://en.wikipedia.org/wiki/Tandem_mass_spectrometry#/media/File:MS_MS.png">wikipedia</a>.")


A plethora of software solutions were developed for the analysis of proteomics data. MaxQuant is one of the most popular proteomics softwares because it is an easy to use and free software that offers functionalities for nearly all kinds of proteomics data analysis challenges {% cite Cox2008 %}. Mass spectrometry raw data is normally obtained in a vendor specific, proprietary file format. MaxQuant can directly take those raw files as input. For peptide identification MaxQuant uses a search engine called "Andromeda". MaxQuant offers highly accurate functionalities for many different proteomics quantification strategies, e.g. label-free, SILAC, TMT. 

Blood is a commonly used biofluid for diagnostic procedures. The cell-free liquid blood portion is called plasma and after coagulation serum. Plasma and serum proteomics are frequently performed to find new biomarkers e.g. for diagnostic purposes and personalized medicine ({% cite Geyer2017 %}). Serum and Plasma proteomics are particularily challenging due to protein concentration differences in the orders of ten magnitudes. Therefore, most sample preparation protocols include a depletion step in which the most abundant proteins are (partially) depleted from the sample e.g. via columns with immobilized antibodies.

This stand-alone tutorial introduces the data analysis from raw data files to protein identification and quantification of two label-free human serum samples with the MaxQuant software. One sample is a pure serum sample, while the other sample has been depleted for several abundant blood proteins. One of the questions in this tutorial is to find out which sample was depleted and which not. 

For more advanced proteomics workflows, please consult the OpenMS [identification]({{site.baseurl}}/topics/proteomics/tutorials/protein-id-oms/tutorial.html), [quantification]({{site.baseurl}}/topics/proteomics/tutorials/protein-quant-sil/tutorial.html) as well as [SearchGUI/PeptideShaker]({{site.baseurl}}/topics/proteomics/tutorials/protein-id-sg-ps/tutorial.html) tutorials.


> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Get data

The serum proteomic samples and the fasta file for this training were deposited at [Zenodo](https://zenodo.org/record/3774452). It is of course possible to use other fasta files that contain human proteome sequences, but to ensure that the results are compatible we recommend to use the provided fasta file. MaxQuant not only adds known contaminants to the fasta file, but also generates the "decoy" hits for false discovery rate estimation itself, therefore the fasta file is not allowed to have decoy entries. To learn more about fasta files, have a look at [Protein FASTA Database Handling]({{site.baseurl}}/topics/proteomics/tutorials/database-handling/tutorial.html).


> ### {% icon hands_on %} Hands-on: Data upload
>
> 1. Create a new history for this tutorial and give it a meaningful name
>
>    {% include snippets/create_new_history.md %}
>
> 2. Import the fasta and raw files from [Zenodo](https://zenodo.org/record/3774452)
>
>    ```
>    https://zenodo.org/record/3774452/files/Protein_database.fasta
>    https://zenodo.org/record/3774452/files/Sample1.raw
>    https://zenodo.org/record/3774452/files/Sample2.raw
>    ```
>    {% include snippets/import_via_link.md %}
>
> 3. Once the files are green, rename the raw datasets into 'sample1' and 'sample2' and the fasta file into 'protein database'
>    {% include snippets/rename_dataset.md %}
> 4. Set the data type to thermo.raw for 'sample1' and 'sample2'
>
>    {% include snippets/change_datatype.md datatype="thermo.raw" %}
>
{: .hands_on}

# MaxQuant Analysis

The MaxQuant Galaxy implementation contains the most important MaxQuant parameters. As an alternative, **MaxQuant (using mqpar.xml)** {% icon tool %} can be used with a preconfigured mqpar.xml file. We will explain the parameters after starting the MaxQuant run which takes some time to finish.


> ### {% icon hands_on %} Hands-on: MaxQuant analysis
>
> 1. **MaxQuant** {% icon tool %} with the following parameters:
>    - In *"Input Options"*:
>        - {% icon param-file %} *"FASTA files"*: `protein database`
>    - In *"Search Options"*:
>        - *"minimum unique peptides"*: `1`
>    - In *"Parameter Group"*:
>        - {% icon param-files %} *"Infiles"*: `sample1`, `sample2`
>        - *"missed cleavages"*: `1`
>        - *"variable modifications"*: ` `
>        - *"Quantitation Methods"*: `label free quantification`
>    - *"Generate PTXQC (proteomics quality control pipeline) report?"*: `Yes`
>    - In *"Output Options"*:
>        - *"Select the desired outputs."*: `Protein Groups` `Peptides` `mqpar.xml`
>
>    > ### {% icon comment %} Comment: Protein Groups
>    > Proteins that share all their peptides with other proteins cannot be unambiguously identified. Therefore, MaxQuant groups such proteins into one protein group and only one common quantification will be calculated. The different protein properties are separated by semicolon. 
>    {: .comment}
{: .hands_on}

## More details on MaxQuant Parameters 

The *"minimum peptide length"* defines the minimum number of amino acids a peptide should have to be included for protein identification and quantification. Below 7 amino acids a peptide cannot be unique and is therefore not informative, thus typical values are in the range 7-9.

Several, even longer peptides are not unique, meaning that they are shared by several proteins e.g. when they are part of a common protein domain. During protein inference the peptides are statistically assembled into the corresponding proteins and the decision should be mainly based on the unique peptides. Therefore, we set *"min unique peptides"* to `1` - only proteins that have at least one unique peptide are reported in the output table.

In most bottom-up proteomics experiments Trypsin is used as a protease because it has many advantages such as it's accurate cleavage specificity. Trypsin cleaves peptides c-terminal of Arginine (R) and Lysine (K), except when those are followed by Proline (P). Therefore, in MaxQuant the default *"enzyme"* is set to `Trypsin/P`. This trypsin specific cleavage rule is used by MaxQuant to perform an *in silico* digestion of the protein database that was provided in the fasta file.

Protease digestion is not always complete, therefore we set the *"number of missed cleavages"* to `1`, meaning that the *in silico* digestion also includes peptides that have an additional Arginine or Lysine in their sequence.

![MQ_param_missed](../../images/maxquant_param_missedcl.png "Trypsin specificity and missed cleavages. With one missed cleavage all three peptides will be part of the *in silico* peptide database")

From the *in silico* generated peptide database the masses of the peptides are calculated and matched to the measured masses in order to identify them. A peptide's mass will change due to peptide modifications such as chemical labelling for example applied in different quantitation strategies or biological post translational modifications. Therefore, it is important to also include possible peptide modifications in the *in silico* generated peptide mass list. *"Fixed modifications"* are modifications that occur on every occurence of the specific amino acid. Those are often artificially introduced modifications such as Carbamidomethylation of cysteins (C) to prevent re-formation of the disulfid bridges. This is a common procedure in proteomics sample preparation and therefore also the default option in MaxQuant: `Carbamidomethyl (C)`. *"Variable modifications"* are modifications that do not occur on every amino acid such as Oxidation of Methionine might only occur on some Methionines and not all, but only a few peptide N-termini are acetylated. Variable modifications increase the *in silico* peptide database because each peptide's mass is calculated once with and once without the additional modification. To keep computation times as short as possible we did not use any variable modification this training, despite the MaxQuant defaults Oxidation of Methionine and Acetylation of N-termini would have been completely valid to use. 

MaxQuant supports different *"Quantitation Methods"*. The three main categories are `label-free quantification`, `label-based quantification` and `reporter ion MS2` quantification. In this tutorial we have chosen `label-free` because we did not apply any specific labeling/quantitation strategy to the samples.

![MQ_quant_methods](../../images/maxquant_lfq_quant_methods.png "Overview of MaxQuant quantification methods")

The PTXQC software ({% cite Bielow2015 %}) was built to enable direct proteomcs quality control from MaxQuant result files. This quality control can be directly used in the Galaxy MaxQuant wrapper by setting *"Generate PTXQC"* to `yes`. This will generate a pdf file with multiple quality control plots. Be aware that the cutoffs set in PTXQC might not be applicable to your experiment and mass spectrometer type and therefore "under performing" and "fail" do not necessarily mean that the quality is poor. 

![PTXQC_overview](../../images/maxquant_ptxqc_overview.png "Overview of PTXQC quality measures for sample1 and sample2")


MaxQuant automatically generates several output files. In the *"Output Options"* all or some of the output files can be selected. The protein information can be obtained by selecting `Protein Groups`, while the peptide information is obtained by choosing `Peptides`. The applied MaxQuant parameters are stored in the `mqpar.xml` This file can be re-used as an input file in the **MaxQuant (using mqpar.xml)** {% icon tool %}.


> ### {% icon details %} More MaxQuant parameters
> - The *"parse rules"* in the input section are applied to the fasta sequence headers. The default automatically extracts the Uniprot ID from fasta files that were downloaded from uniprot. Regular expressions can be adjusted to keep other information from the fasta file.
>
> - For pre-fractionated data an *"experimental design template"* has to be used. This has to be a tab-separated text file with a column for the fractions (e.g. 1-10) and a column for the experiment (sample1, sample2, sample3) and a column for post translational modifications (PTM). Examples are given in the help section of the MaxQuant tool. 
>
> - *"Match between run"* allows to transfer identifications (peptide sequences) from one run to another. If the MS1 (full length peptide) signal is present in both runs, but was only selected for fragmentation in one of them, MaxQuant can transfer the resulting peptide sequence to the run where the MS1 peptide was not fragmented. The Information if a peptides was identified via fragmentation (MS/MS) or match between run (matching) can be found in the evidence output.
>
> - MaxQuant allows to process different raw files with different parameters. In this tutorial we have loaded both files into the same *"parameter group"* in order to process them with the same parameters. To apply different parameters, new parameter groups can be added by clicking on the {% icon param-repeat %} *"insert parameter group"* button. In each *"parameter group"* one or several raw files can be specified and for them only the parameter specified within this parameter group section are applied. 
{: .details}

> ### {% icon tip %} Tip: Continue with results from Zenodo
>
> In case the MaxQuant run is not yet finished, the results can be downloaded from Zenodo to be able to continue the tutorial
> 1. Import the files from [Zenodo](https://zenodo.org/record/3774452)
>
>    ```
>    https://zenodo.org/record/3774452/files/PTXQC_report.pdf
>    https://zenodo.org/record/3774452/files/MaxQuant_Protein_Groups.tabular
>    https://zenodo.org/record/3774452/files/MaxQuant_Peptides.tabular
>    https://zenodo.org/record/3774452/files/MaxQuant_mqpar.xml
>    ```
{: .tip}

> ### {% icon question %} Questions
>
> 1. How many proteins were found in total?
> 2. How many peptides were found in total?
> 3. How many proteins identified and quantified? (Tipp: There is a tool called "filter data on any column", select the LFQ column for both files and remove rows containing "0") 
>
>
> > ### {% icon solution %} Solution
> >
> > 1. 271 protein (groups) were found in total.
> > 2. 2387 peptides were found in total. 
> > 3. Sample1: 237, Sample2: 123 (**filter data on any column** {% icon tool %} on the `protein groups` file *"with following condition"* `c32!=0` or `c33!=0` and *"Number of header lines" `1`)
> >
> {: .solution}
>
{: .question}


# Quality control results

To get a first overview of the MaxQuant results, the PTXQC report is helpful. Click on the {% icon galaxy-eye %} eye of the PTXQC pdf file to open it in Galaxy. Screening through the different plots might already give you a hint which of the samples was pure and which was depleted of abundant proteins. Both samples failed in some categories (see Figure 4 above), especially due to low peptide and protein numbers, which is expected in serum samples and therefore not a quality problem. 

> ### {% icon question %} Questions
>
> 1. How good was the tryptic digestion (percentage of zero missed cleavages)?
> 2. Which sample yielded more protein identifications?
> 3. In which sample were more contaminantes quantified?
> 4. Do you already have a guess on which sample was depleted?
>
>
> > ### {% icon solution %} Solution
> >
> > 1. The digestion was not ideal but good enough to work with. The proportion of zero missed cleavages was 75% for sample1 and around 85% for sample2. ![PTXQC_missed](../../images/maxquant_lfq_missedcleavages.png)
> > 2. Sample 1 yielded more protein identifications ![PTXQC_ids](../../images/maxquant_lfq_proteinid.png)
> > 3. Sample 2 has more contaminants, especially serum albumin is high abundant. ![PTXQC_contaminants](../../images/maxquant_lfq_contaminants.png)
> > 4. Sample1, more information can be found in the next section. 
> >
> {: .solution}


# Serum composition 

To explore the proteomic composition of the two serum samples some postprocessing steps are necessary. The protein groups file has many different columns, therefore the first step is to extract only columns that are of interest for this task. This are the columns with the fasta headers (this includes the protein name as it was written in the fasta file) and the two columns with LFQ intensities for both files. To find the most abundant proteins the LFQ intensities can be sorted. On this sorted dataset we will explore the composition of the serum proteins within both samples using an interactive pie charts diagram. 

> ### {% icon hands_on %} Hands-on: Exploring serum composition
>
> 1. **Cut columns from a table** {% icon tool %} with the following parameters:
>    - *"Cut columns"*: `c8,c32,c33`
>    - {% icon param-file %} *"From"*: `proteinGroups` (output of **MaxQuant** {% icon tool %})
>
> 2. **Sort data in ascending or descending order** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"Sort Query"*: `cut_file` (output of **Cut** {% icon tool %})
>    - *"Number of header lines"*: `1`
>    - In *"Column selections"*:
>        - *"on column"*: `c2`
>        - *"in"*: `Descending order`
>        - *"Flavor"*: `General numeric sort ( scientific notation -g)`
>
> 3. **Sort data in ascending or descending order** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"Sort Query"*: `cut_file` (output of **Cut** {% icon tool %})
>    - *"Number of header lines"*: `1`
>    - In *"Column selections"*:
>        - *"on column"*: `c3`
>        - *"in"*: `Descending order`
>        - *"Flavor"*: `General numeric sort ( scientific notation -g)`
> 4. Click {% icon galaxy-barchart %} “Visualize this data” on the last **Sort** {% icon tool %} result. 
>   - Select `Pie chart (NVD3)`
>   - *"Provide a title"*: `Serum compositions`
>   - Click `Select data` {% icon galaxy-chart-select-data %}
>   - *"Provide a label"*: `sample1`
>   - *"Labels"*: `Column: 1`
>   - *"Values"*: `Column: 2`
>   - Click `Insert data series`
>   - *"Provide a label"*: `sample2`
>   - *"Labels"*: `Column: 1`
>   - *"Values"*: `Column: 3`
>   - Save   {% icon galaxy-save %} (file is saved under "User" --> "Visualizations")
{: .hands_on}


![serum_composition](../../images/maxquant_lfq_serum_composition.png "Quantitative serum composition. In Galaxy one can hover over the graph to see the protein names")


> ### {% icon question %} Questions
>
> 1. What are the top 5 most abundant proteins in both files? Do they reflect typical serum proteins? 
> 2. Which sample was depleted for the top serum proteins?
> 3. How much did the serum albumin abundance percentage decrease? Was the depletion overall succesful?
>
> > ### {% icon solution %} Solution
> >
> > 1. Sample1: Complement C4-A, Ceruloplasmin, Hemopexin, Serum albumin, Complement factor B. Sample2: Serum albumin, Immunoglobulin heavy constant gamma 1, Serotransferrin, Immunoglobulin kappa constant, Haptoglobin. All of those proteins are typical (high abundant) serum proteins [plasma proteins found by MS](https://www.proteinatlas.org/humanproteome/blood/proteins+detected+in+ms). 
> > 2. Sample1 was depleted, sample2 was pure serum. 
> > 3. In the depleted sample1, there is a depletion in some of the most abundant proteins, especially Albumin, which proportion of the total sample intensities decreased by 58 percentage. Compared to the pure serum the depleted sample showed a duplication of identified and quantified proteins rendering it quite successful. However, there is still room for improvement as some of the most abundant proteins which should have been depleted did not change their abundance compared to the overall protein abundance.
> >
> {: .solution}
>
{: .question}


