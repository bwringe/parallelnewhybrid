# parallelnewhybrid 
[![Travis-CI Build Status](https://travis-ci.org/thierrygosselin/parallelnewhybrid.svg?branch=master)](https://travis-ci.org/thierrygosselin/parallelnewhybrid)
[![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/bwringe/parallelnewhybrid?branch=master&svg=true)](https://ci.appveyor.com/project/bwringe/parallelnewhybrid)
[![DOI](https://zenodo.org/badge/21962/bwringe/parallelnewhybrid.svg)](https://zenodo.org/badge/latestdoi/21962/bwringe/parallelnewhybrid)

### ***parallelnewhybrid*** is an R package designed to parallelize *NewHybrids* analyses


***  
Please ensure that you have the correct version of **NewHybrids** installed. The source code and instructions for installation can be found at (<https://github.com/eriqande/newhybrids>)


###Package Installation
**parallelenewhybrid** can be installed from GitHub using the R package *devtools* and the function *install_github*:  
```
devtools::install_github("bwringe/parallelnewhybrid")
```
<span style = "color:red"> <strong>NOTE:</strong></span> : **parallelnewhybrid** relies on functions from the R packages *parallel*, *plyr*, *stringdist*, *stringr*, and *tidyr*. The user should ensure these are installed from CRAN prior to installing **parallelnewhybrid**.
```
if (!require("pacman")) install.packages("pacman")
pacman::p_load(parallel, plyr, stringr, tidyr)
```

###Function descriptions
<h4 class="text-primary">parallelnh_xx.R</h4>
Allows *NewHybrids* (Anderson and Thompson 2002) to be run in parallel. A job (*NewHybrids* analysis) is assigned to each of the *c* cores available in the computer. As each task finishes, a new analysis is assigned to the idled core.  
**parallelnewhybrid** will attempt to analyze all *NewHybrids* format files in the folder specified by the user through the *folder.data* argument. Therefore, it is essential this folder contain only the files the user wishes to analyze, and optionally their associated individual file(s).  
The user can must also specify the length of the Markov chain Monte Carlo (MCMC) burn-in and subsequent run length using the *burnin* and *sweeps* parameters.  
<span style = "color:red"> <strong>NOTE:</strong></span> There are **three operating system-specific versions** of the **parallelnh_xx** function because of the different ways in which the operating systems handle forking of processes.  

**parallelnh version**|**Operating system**
------------|----------
**parallelnh_OSX** | OS X
**parallelnh_WIN** | Windows
**parallelnh_LINUX** | Linux (Ubuntu)

***  

###How to use:  
####Example datasets:  
Example datasets have been provided as R images (.rda files). These can be loaded into your workspace using the ``data()`` function.

**Example dataset** | **Contents**  
------------|---------------------------------------------------------------  
*SimPops\_S1R1_NH* | A *NewHybrids* format file. To analyze this file using the function **parallelnh_xx**, save it with the extension ".txt" to an empty folder on your hard drive, then provide **parallelnh_xx** with the file path to the folder. To run in parallel, after saving the file, copy it and give the copies unique names. **parallelnh_xx** will attempt to analyze all files which do not contain "individuals.txt" within the file name, so it is essential that only *NewHybrids* formatted files, and their associated individual files be present in the folder provided to **parallelnh_xx**.
*SimPops\_S1R1_NH_individuals* | The individual file associated with *SimPops\_S1R1_NH*. A single copy of this file should be saved to the same folder in which *SimPops\_S1R1_NH* is saved. The filename must end in "individuals.txt".

***  
<h4 class="text-primary">parallelnh_xx</h4>  

**Parameter** | **Description**
------------|---------------------------------------------------------------  
*folder.data*| A file path to the folder in which the *NewHybrids* formatted files to be analyzed, and their associated individual file reside.
*where.NH* | A file path to the *NewHybrids* installation folder. NOTE: The name of this folder must be named "newhybrids". If it is named anything else the function will fail.
*burnin* | An integer specifying how many burn-in steps *NewHybrids* is to run
*sweeps* | An integer specifying how many sweep steps *NewHybrids* is to run  

```r

### ANALYSIS OF EXAMPLE DATA

## Get the file path to the working directory, will be used to allow a universal example
path.hold <- getwd()

## Get the individual file included along with the parallelnewhybrid package and make it an object
sim_inds <- parallelnewhybrid::SimPops_S1R1_NH_individuals

## Get the genotype data file included along with the parallelnewhybrid package and make it an object
sim_data <- parallelnewhybrid::SimPops_S1R1_NH

## Save the individual data to the working directory as a file called "SimPops_S1R1_NH_individuals.txt"
write.table(x = sim_inds, file = paste0(path.hold, "/SimPops_S1R1_NH_individuals.txt"), row.names = FALSE, col.names = FALSE, quote = FALSE)

## Save the genotype data to the working directory as a file called "SimPops_S1R1_NH.txt"
write.table(x = sim_data, file = paste0(path.hold, "/SimPops_S1R1_NH.txt"), row.names = FALSE, col.names = FALSE, quote = FALSE)

## Create an empty folder within the working directory. Recall, parallelnewhybrids will analyze all files within the folder it is specified, but if there are files that are not NewHybrids format, or individual files, it will fail.
dir.create(paste0(path.hold, "/parallelnewhybrids example"))

## Copy the individual file to the new folder
file.copy(from = paste0(path.hold, "/SimPops_S1R1_NH_individuals.txt"), to = paste0(path.hold, "/parallelnewhybrids example"))

## Copy the genotype data file to the new folder
file.copy(from = paste0(path.hold, "/SimPops_S1R1_NH.txt"), to = paste0(path.hold, "/parallelnewhybrids example"))

## Create two copies of the genotype data file to act as technical replicates of the NewHybrids simulation based analysis. This will also serve demonstrate the parallel capabilities of parallelnewhybrid.
file.copy(from = paste0(path.hold, "/parallelnewhybrids example/SimPops_S1R1_NH.txt"), to = paste0(path.hold, "/parallelnewhybrids example/SimPops_S1R2_NH.txt"))

file.copy(from = paste0(path.hold, "/parallelnewhybrids example/SimPops_S1R1_NH.txt"), to = paste0(path.hold, "/parallelnewhybrids example/SimPops_S2R3_NH.txt"))

## Clean up the working directory by deleting the two files
file.remove(paste0(path.hold, "/SimPops_S1R1_NH_individuals.txt"))

file.remove(paste0(path.hold, "/SimPops_S1R1_NH.txt"))

## Create an object that is the file path to the folder in which NewHybrids is installed. Note: this folder must be named "newhybrids"
your.NH <- "YOUR PATH/newhybrids/"

## Execute parallelnh. NOTE: "xx" must be replaced by the correct designation for your operating system. burnin and sweep values have been chosen for demonstration only.
parallelnh_xx(folder.data = paste0(path.hold, "/parallelnewhybrids example/"), where.NH = your.NH, burnin = 100, sweeps = 100)


## Clean up everything by deleting the example folder. Note: comment characters have been added to prevent this command being run accidently.
unlink(paste0(path.hold, "/parallelnewhybrids example/"), recursive = TRUE)


```

***  

### Important Notes:

* The folder in which *NewHybrids* resides must be named "newhybrids". If it is named anything else, the function will fail.  
* All file paths must end in "/". If they do not, the function may fail. As well, it is preferable to place all folders such that the file path to them does not contain spaces.  
* The first time the function is run, it may trigger warnings from the operating system, this is normal. Further, the function may not work properly in newer versions of Windows unless R is run as Administrator.  

***

### **parallelnewhybrid** Contributors

* Brendan Wringe <https://github.com/bwringe>; bwringe(at)gmail.com; Corresponding author  
* Dr. Ian Bradbury <http://tinyurl.com/h3qrhkv>  
* Dr. Nick Jeffery <https://github.com/NickJeff13>  
* Dr. Ryan Ronald Edward Stanley <https://github.com/rystanley> 

* If you don't understand anything, please contact me (bwringe[at]gmail.com)
* Any ideas on how to improve the functionality are very much appreciated.
* If you spot a typo, feel free to edit and send a pull request.  

**Reference**
Anderson EC, Thompson EA. A model-based method for identifying
species hybrids using multilocus genetic data.
Genetics. Genetics Society of America;
2002;160: 1217-1229.


To cite the current version of **parallelnewhybrid**, please refer to our zenodo DOI: [![DOI](https://zenodo.org/badge/21962/bwringe/parallelnewhybrid.svg)](https://zenodo.org/badge/latestdoi/21962/bwringe/parallelnewhybrid)

***
***
