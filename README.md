Making R/Bioconductor tool integration easier in Galaxy
=====

Nitesh Turaga (nitesh.turaga@gmail.com)1, Dannon Baker (dannon.baker@gmail.com)1, John Chilton (jmchilton@gmail.com)2, Galaxy Team1,2, Anton Nekrutenko (anton@nekrut.org)2, and James Taylor (james@taylorlab.org)1

1Department of Biology, Johns Hopkins University, Baltimore, MD 21218, USA
2Department of Biochemistry and Molecular Biology, Penn State University, University Park, PA 16802, USA

# Abstract

Galaxy provides a web platform for interactive large scale data analysis where different bioinformatics tools written in any language can be integrated. A substantial amount of work in the world of bioinformatics is done in the R programming language as it allows for powerful analysis and visualization of complex data. This work is represented by the Bioconductor Project that provides highly used open source bioinformatics tools. While some of these tools are available in Galaxy, both communities would benefit greatly if they were integrated on a larger scale. Tool development in Galaxy is one of the early entry points to Galaxy developers, biologists, and bioinformaticians who want to make their work more accessible to the community. In this article we present a solution that makes Bioconductor and R tool integration into Galaxy easier and address dependency resolution using existing software including Planemo, Bioconda and BioaRchive.

# Keywords

Interoperability, Bioconductor, R, Galaxy, Open Source, Bioinformatics, Planemo
	
# Introduction

The Bioconductor project provides one of the largest suites of open source bioinformatics tools, covering analyses from highly used microarrays to the latest sequencing technology. This gives Bioconductor a large user base consisting of people from all biomedical professions analyzing genomics data. Developers often want to distribute their R/Bioconductor scripts and packages, and Galaxy has a large community of users who might not be able to otherwise access or effectively utilize these tools.

The Galaxy Project provides an open source web platform that allows command-line bioinformatics tools written in different languages to be integrated and displayed with a consistent interface focused on usability. The tool integration process involves writing a tool definition file which provides the interface between Galaxy and the tool being integrated. It also requires dependency management that makes software dependencies of the tool being integrated available to Galaxy.  Galaxy tool development is one of the early entry points for developers, and making this process of tool development and integration easier opens up the space for non-developers as well.

Recognizing the need to make tool development and integration an easier and more intuitive process, we describe a method that simplifies tool integration with R/Bioconductor packages in Galaxy by providing new functionality to an existing command-line tool Planemo. We describe how Planemo leverages Bioconda for dependency management for R/Bioconductor tools. Bioconda locates specific versioned packages in Bioconductor via BioaRchive and R via CRAN. This addition to Planemo provides a template for the tool definition file specific to R/Bioconductor tool development and also solves the issue of having the tool developer deal with R/bioconductor dependencies, helping them focus on providing a better and more intuitive user interface for the user.

# Methods

> WE CAN REMOVE THIS (MF). We describe the methods section in two parts: implementaion and operation. Implementation includes how the software was designed and implemented by adding new functionality to Planemo, Bioconda recipe generation, and BioaRchive. We also discuss how the three of them tie into each other to make tool integration easier. Operation includes minimal system requirements needed to run the software and . Use Cases, the specific use cases demonstrating the command line tools. A sample Tool development workflow, that gives a brief overview of the workflow to generate new tools.

### Implementation

Planemo provides command-line utilities to assist in building and publishing Galaxy tools. Planemo is easily extendable and different commands provide a good reference for how to add new functionality. We model our addition, the command `planemo bioc_tool_init`, on the command `planemo tool_init` with options and arguments tailored to address Bioconductor integration. This new command creates a Bioconda recipe using `bioconductor_skeleton.py`, and annotates the tool definition file to enable Galaxy to leverage the recipe. 

The Bioconda recipe generated is used as the dependency for the Bioconductor package. This eliminates the need to recursively parse the Bioconductor dependency tree to create an extra Galaxy-specific tool dependency file, which was the case previously. This approach has the added benefit of creating an artifact which, while created to use with a Galaxy tool, is potentially useful outside of the Galaxy ecosystem by anyone using Bioconda.

We also provide a way to contribute to Bioconda via planemo, through the `planemo bioc_conda_recipe_init` command line tool. This makes it easier to build Bioconda recipes for the Bioconductor package being integrated.

Bioconda (`bioconductor_skeleton.py`) in turn uses BioaRchive, a Bioconductor package version archive, to get the correct package versions if they exist in BioaRchive. The `bioconductor_skeleton.py` script has been modified to not only find missing Bioconductor, r-package dependencies but also create them in the local Bioconda repository specified by the user. BioaRchive improves reproducibility of the Bioconda recipes of different versions of the same Bioconductor package. The amalgamation of these three can be found in the command `planemo bioc_tool_init`, which allows developers to focus on creating better tools in Galaxy by reducing focus on dependencies and their versions.

New commands were included in the commands folder in planemo, planemo/commands/cmd_bioc_tool_init.py, planemo/commands/cmd_bioc_conda_recipe_init.py.
 


*Command Usage 1: (limited options are shown)*

```
bash$ planemo bioc_tool_init --help
Usage: planemo bioc_tool_init [OPTIONS]

  Generate a bioconductor tool outline from supplied arguments.

Options:
  --bioconda_path TEXT      Give path to bioconda repository, if left empty,path will be made in home directory
  --requirement TEXT        Give the name of the bioconductor package,requirements will be set using bioconda eg: 'motifbreakR'
  --output TEXT             An output location (e.g. output.bam), the Galaxy datatype is inferred from the extension.
  --input TEXT              An input description (e.g. input.fasta)
  -n, --name TEXT           Name for new Bioconductor tool (user facing)
  -t, --tool PATH           Output path for new tool (default is <id>.xml)
  -i, --id TEXT             Short identifier for new tool (no whitespace)
  --help                    Show this message and exit.
```

*Command Usage 2:*

```
$ planemo bioc_conda_recipe_init --help
Usage: planemo bioc_conda_recipe_init [OPTIONS]

  Make a conda recipe, given a package name.

  Package_name = motifbreakR, 
  bioconda_dir = '/path/workspace'.

Options:
 -u, --update / --no-update    Update an existing bioconda recipe
 -b, --bioconda_dir_path TEXT  Give the path to folder containing bioconda repository
 -c, --clone / --no-clone      Clone bioconda repository from github or not?
 -p, --package_name TEXT       Give the name of a Bioconductor package to create a new conda recipe
 --help                        Show this message and exit.
```

### Operation

The minimal system requirements needed to run the software are Python and R. We specify detailed library requirements in the supplementary section. 

> Need to add specifics here. Versions of R/Python. Operating system. Memory/space requirements. Also include here an overview of the workflow.

Tool Development workflow

We describe the tool development workflow with a bioconductor package “motifbreakR” as an example. We assume the user has planemo installed and on their path along with the requirements for the software.

### Step 1: 

Use the  planemo command bioc_tool_init, with the following parameters,

```	
planemo bioc_tool_init 
--name motifbreakR 
--id motifbreakR 
--requirement 'motifbreakR'
--bioconda_path /Users/Documents/workspace
```

This gives an output file,  motifbreakR.xml,

```
<tool id="motifbreakr" name="motifbreakR" version="0.1.0">
    <requirements>
       <requirement type="package" version="1.2.2">motifbreakR</requirement>
    </requirements>
    <stdio>
       <exit_code range="1:" />
    </stdio>
    <command><![CDATA[
       TODO: Fill in command template.
    ]]></command>
    <inputs>
    </inputs>
    <outputs>
    </outputs>
    <help><![CDATA[
        We introduce motifbreakR, which allows the biologist to judge in the first place whether the sequence surrounding the polymorphism is a good match, and in the second place how much information is gained or lost in one allele of the polymorphism relative to another. MotifbreakR is both flexible and extensible over previous offerings; giving a choice of algorithms for interrogation of genomes with motifs from public sources that users can choose from; these are 1) a weighted-sum probability matrix, 2) log-probabilities, and 3) weighted by relative entropy. MotifbreakR can predict effects for novel or previously described variants in public databases, making it suitable for tasks beyond the scope of its original design. Lastly, it can be used to interrogate any genome curated within Bioconductor (currently there are 22).
 
http://bioconductor.org/packages/release/bioc/html/motifbreakR.html
    ]]></help>
</tool>
```

The tool creates a Bioconda recipe at the path specified by the user i.e ‘/Users/Documents/workspace’ and a tool definition file in the location where the command was executed ‘motifbreakR.xml’. The Bioconda recipe is created after the Bioconda repository is cloned at   ‘/Users/Documents/workspace/bioconda-recipes/recipes/bioconductor-motifbreakr’. This allows the user to have a local copy of the recipe and all of the bioconda package dependencies for that recipe.

The tool definition file being automatically created helps the user get started quickly, and with a good blueprint.  If more options are used with bioc_tool_init, i.e, --input and --output, given an R script which takes command line inputs and outputs, bioc_tool_init will generate a tool definition file with inputs, outputs and other options specified. The output of such a command is shown in the supplementary material. A full list of options can be seen in the help section of the tool. An example R script using the motifbreakR package would be executed on the command line by:
```
$ Rscript motifbreakR-sample-script.R 
--input <inputfile> 
--output <outputfile>
```

### Step 2: 

If the user does not have conda installed, it is possible to do it via planemo using `planemo conda_init --conda_ensure_channels bioconda,r` This will install conda locally and ensures that the Bioconda and R channels are available for dependency resolution.

### Step 3: 

If the user uploads the Bioconda recipe (bioconductor-motifbreakr) created in Step 1 to the Bioconda channel, then the recipe will be available when using planemo conda_install motifbreakR.xml. This command, conda_install, parses the specified files and installs required dependencies on the local machine using conda.

It is also possible to install the requirement for motifbreakR without planemo. Since the recipe is available on the user's local machine, assuming the user has conda installed on their machine, 

```
$ conda build /Users/Documents/workspace/bioconda-recipes/recipes/bioconductor-motifbreakr --channel bioconda --channel r
```

# Use Cases

Here we describe two use cases for integrating R/Bioconductor tools. We use a toy tool example here, `test_tool.R`.

### Generate Bioconductor tool with Planemo: 

```
planemo bioc_tool_init 
--name motifbreakR 
--id motifbreakR 
--requirement ‘motifbreakR’
```

The package `motifbreakR` is used as an example to explain the usage. The command creates a new tool definition file, motifbreakR.xml, with the Bioconda recipe as a requirement in the tool definition file. It clones the Bioconda repository in the given bioconda_path and creates a new recipe, if, it did not exist already. If it is a new recipe it can be further contributed to the Bioconda repository if the user chooses. This command will be further improved in the following versions of the tool, to provide a more holistic tool definition file, which reduces workload on the tool developer. 

### Generate Bioconda recipe with Planemo: 

```
planemo bioc_conda_recipe_init 
--clone
--update
--package_name ‘motifbreakR’
```

The command creates a new conda recipe, which can be contributed to Bioconda. The user should contribute the package back to the Bioconda repository if they make the Galaxy tool public. The Bioconda repository serves as a hosting service for recipes, and other users of the Galaxy tool can access it from there on their local instances. This command can also be used if the user needs to update a Bioconductor package recipe to the latest version. The bioc_tool_init command is mainly used to create, not update, the dependency requirements and provide a blueprint for the tool definition file. 

## Conclusions and future work

This work extends the existing Planemo software adding not only easier R/Bioconductor tool development capabilities but also a new way of dependency management that is compatible with different platforms using Bioconda and BioaRchive. We present command line utilities that will aid in integrating a large body of bioinformatics tools in Galaxy. Integrating Bioconductor tools within Galaxy will open up bioinformatics analysis for larger network of researchers and provide a reproducible environment. The area of integrating Bioconductor tools into Galaxy shows plenty of scope. There are currently 1580 Bioconductor packages (https://www.bioconductor.org/packages/stats/) which consist of plotting tools, database interface tools, data manipulation tools, and analysis tools. Out of these 1580 tools, there are only around 13 currently available on the main Galaxy toolshed (https://toolshed.g2.bx.psu.edu/). Although, not all 1580 are easy to integrate into Galaxy, this work tries to encourage more Bioconductor tool developers to share their tools on Galaxy.

Future work includes improved versions of command line tool bioc_tool_init, will represent a more holistic tool definition files for R/Bioconductor packages. We envision passing a text file as argument containing a list (pipeline) of R/Bioconductor functions, which can be wrapped as a galaxy tool with a single command forming a comprehensive analysis tool. 

We also provide a detailed manual for how tool integration for galaxy is done for R/Bioconductor tools in the document hosted on github (Link 1). This document gives an overview of manual tool integration which we make easier with new feature additions to Planemo. It also provides a list of best practices for development processes in R/Bioconductor to make it easier for the developer.  planemo bioc_tool_init in its current version, provides a basic structure for use in quickly integrating Bioconductor tools along with providing requirements generated with Bioconda.

Link 1 : https://github.com/nturaga/bioc-galaxy-integration/blob/master/README.md

## Data and Software Availability

Link to Planemo feature (https://github.com/nturaga/planemo/tree/add_bioc_tool_init)
Link to Bioconda (https://bioconda.github.io/)
Link to BioaRchive (http://bioarchive.github.io/)
Link to Planemo(https://pypi.python.org/pypi/planemo)

## Author Contributions

NT designed and implemented the features presented with advice from DB, JC, AN and JT. JC implemented the Planemo tool on which this work is built. NT and DB wrote the paper. Members of the Galaxy Team developed that Galaxy framework this work relies on and provided advice on the project. All authors were involved in the revision of the draft manuscript and have agreed to the final content.


## Acknowledgments

We are grateful to the Galaxy Team members and the developer community for the Galaxy Framework which was developed over many years. Their guidance has been extremely valuable to this project. Bioconda has been providing a great service to the community, and we would like to thank the developers of Bioconda. We give special mention to Ryan Dale, for developing the script to create Bioconda recipes for Bioconductor packages. We also thank the contributors of BioaRchive. This project is funded by National Institutes of Health (NIH) [U41 HG006620]. 

## Funding Source

Funding for open access charge: National Institutes of Health (NIH) [U41 HG006620]

## Competing interests : No competing interests were declared.

## Software License 

Academic Free License version 3.0

## References

Goecks, J, Nekrutenko, A, Taylor, J and The Galaxy Team. Galaxy: a comprehensive approach for supporting accessible, reproducible, and transparent computational research in the life sciences. Genome Biol. 2010 Aug 25;11(8):R86.
Gentleman R.C., Carey V.J., Bates D.M., Bolstad B., Dettling M., Dudoit S., Ellis B., Gautier L., Ge Y., Gentry J., Hornik K., Hothorn T., Huber W., Iacus S., Irizarry R., Leisch F., Li C., Maechler M., Rossini A.J., Sawitzki G., Smith C., Smyth G., Tierney L., Yang J.Y. and Zhang J. (2004) Bioconductor: open software development for computational biology and bioinformatics. Genome Biol. 5(10): R80.
Planemo (https://pypi.python.org/pypi/planemo).
Bioconda (https://bioconda.github.io/).
R Core Team (2016). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria(https://www.R-project.org/).
BioaRchive (https://bioarchive.galaxyproject.org)
Supplementary Material

Here we describe an example of a planemo bioc_tool_init command with two inputs, and a single output defined. The options ‘--name’ and ‘--id’ describe the name of the tool in your Galaxy environment, and the id for the xml file (motifbreakR.xml). The ‘--requirement’ option is the exact name of the Bioconductor package, which is case-sensitive. The `--bioconda_path’ option specifies where the bioconda directory is cloned and recipes are created. The options defined as ‘--input’ mentions the inputs the R script takes, and ‘--output’ mentions the result generated after the tool run. The option ‘--example_command’ fills the command tag in the tool definition file generated which is executed by Galaxy.

```
planemo bioc_tool_init 
--name motifbreakR 
--id motifbreakR 
--requirement 'motifbreakR' 
--bioconda_path /Users/nturaga/Documents/workspace 
--input 'variants.csv' 
--input 'motifdb.csv' 
--output 'motifbreakr.results' 
--example_command \ 
'Rscript motifbreakR-sample-script.R 
--input test-data/variants.csv 
--input test-data/motifdb.csv
--output test-data/output.csv'
```

The command above generates a tool definition file with all the specified options. The tool definition file, is usable in Galaxy if there is a motifbreakR-sample-script.R. The script will need to have the options to specify the inputs and the output as command line arguments. 

```
<tool id="motifbreakR" name="motifbreakR" version="0.1.0">
    <requirements>
        <requirement type="package" version="1.2.2">motifbreakR</requirement>
    </requirements>
    <stdio>
        <exit_code range="1:" />
    </stdio>
    <command><![CDATA[
        Rscript motifbreakR-sample-script.R --input test-data/variants.csv --input test-data/motifdb.csv --output test-data/output.csv'
    ]]></command>
    <inputs>
        <param type="data" name="variants" format="csv" />
        <param type="data" name="motifdb" format="csv" />
    </inputs>
    <outputs>
        <data name="motifbreakr" format="results" />
    </outputs>
    <help><![CDATA[
        We introduce motifbreakR, which allows the biologist to judge in the first place whether the sequence surrounding the polymorphism is a good match, and in the second place how much information is gained or lost in one allele of the polymorphism relative to another. MotifbreakR is both flexible and extensible over previous offerings; giving a choice of algorithms for interrogation of genomes with motifs from public sources that users can choose from; these are 1) a weighted-sum probability matrix, 2) log-probabilities, and 3) weighted by relative entropy. MotifbreakR can predict effects for novel or previously described variants in public databases, making it suitable for tasks beyond the scope of its original design. Lastly, it can be used to interrogate any genome curated within Bioconductor (currently there are 22).

http://bioconductor.org/packages/release/bioc/html/motifbreakR.html
    ]]></help>
</tool>
```

Operation: Minimal system requirements needed to run the software including R session information and python package requirements.
```
System requirements:
Python 2.7.10
R version 3.2.4 (2016-03-10)
Platform: x86_64-apple-darwin13.4.0 (64-bit)
Running under: OS X 10.11.4 (El Capitan)
```

```
Pip Freeze: (these are the python package requirements, which can be automatically installed alongside Planemo when you ‘pip install planemo’)
aenum==1.4.0
bioblend==0.7.0
boto==2.39.0
click==6.6
docutils==0.12
galaxy-lib==16.7.0
glob2==0.4.1
Jinja2==2.8
MarkupSafe==0.23
PyYAML==3.11
requests==2.9.1
requests-toolbelt==0.6.0
six==1.10.0
virtualenv==15.0.1
```
