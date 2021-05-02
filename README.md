<!-- README.md is generated from README.Rmd. Please edit that file -->
microdadosBrasil
================

work in progress
----------------

### NEW:

-   Don't use R? See: [using the package from Stata and Python](https://github.com/lucasmation/microdadosBrasil/blob/master/vignettes/Running_from_other_software.Rmd)

### COMMING SOON:

-   Vignettes and Portuguese documentation
-   Censo 1991, PNADs before 2001
-   Support for data not fitting into memory.

In the near future:

-   Variable name harmonization

Description
-----------

This package contains functions to read the most commonly used Brazilian microdata easily and quickly, given that importing these microdata can be tedious. Most data is provided in fixed width files (fwf) with import instructions only for SAS and SPSS. The Data sometimes comes subdivided into serveral files, by state or macro regions. Also, it is common for file and variable names of the same dataset to vary overtime. `microdadoBrasil` handles all these idiosyncrasies for you. In the background the package is running `readr` for fwf data and `data.table` for .csv data. Therefore reading is reasonably fast.

Currently the package includes import functions for:

    #> Warning: package 'printr' was built under R version 3.4.4

| Source | Dataset                 | Import\_function            | Period             | Subdataset                 |
|:-------|:------------------------|:----------------------------|:-------------------|:---------------------------|
| IBGE   | PNAD                    | read\_PNAD                  | 1976 to 2015       | domicilios, pessoas        |
| IBGE   | Pnad Contínua           | read\_PNADContinua          | 2012-1q to 2017-4q | pessoas                    |
| IBGE   | Censo Demográfico       | read\_CENSO                 | 2000 and 2010      | domicilios, pessoas        |
| IBGE   | PME                     | read\_PME                   | 2002.01 to 2015.12 | vinculos                   |
| IBGE   | POF                     | read\_POF                   | 2008               | several, see details       |
| INEP   | Censo Escolar           | read\_CensoEscolar          | 1995 to 2014       | escolas, ..., see details  |
| INEP   | Censo da Educ. Superior | read\_CensoEducacaoSuperior | 1995 to 2014       | see details                |
| MTE    | CAGED                   | read\_CAGED                 | 2009.01 to 2016.05 | vinculos                   |
| MTE    | RAIS                    | read\_RAIS                  | 1998 to 2014       | estabelecimentos, vinculos |

For the datasets in fwf format, the package includes, internally, a list of import dictionaries. These were constructed with the `import_SASdictionary` function, which users can use to import dictionaries from datasets not included here. Import dictionaries for the datasets included in the package can be accessed with the `get_import_dictionary` function.

The package also harmonizes folder names, folder structure and file name that change overtime through a metadata table. It also unifies data that comes subdivides by regional subgroups (UF or região) into a single data object.

Installation
------------

``` r
install.packages("devtools")
devtools::install_github("lucasmation/microdadosBrasil")
library('microdadosBrasil')
```

Basic Usage
-----------

``` r
##############
# Censo Demográfico 2000
download_sourceData("CENSO", 2000, unzip = T)
d <- read_CENSO('domicilios', 2000)
d <- read_CENSO('pessoas', 2000)

#To import data from a specific path in your machine use:
d <- read_CENSO('domicilios', 2000, root_path ="C:/....")
#To restrict the import to only one State use:
d <- read_CENSO('pessoas', 2000, UF = "DF")


##############
# PNAD 2002
download_sourceData("PNAD", 2002, unzip = T)
d  <- read_PNAD("domicilios", 2002)
d2 <- read_PNAD("pessoas", 2002)

##############
# PNAD Continua
# quarterly version
download_sourceData("PnadContinua", i = "2014-4q", unzip = T)
d <- read_PNADcontinua("pessoas", i = "2014-4q")

# yearly version - 1st interview
download_sourceData("PnadContinua", i = "2016-anual-1v", unzip = T)
d <- read_PNADcontinua("pessoas", i = "2016-anual-1v")

# yearly version - 5th interview (available for some years, check get_available_periods)
download_sourceData("PnadContinua", i = "2016-anual-5v", unzip = T)
d <- read_PNADcontinua("pessoas", i = "2016-anual-5v")

##############
# Censo Escolar
download_sourceData('CensoEscolar', 2005, unzip=T)
d <- read_CensoEscolar('escola',2005)
d <- read_CensoEscolar('escola',2005,harmonize_varnames=T)

##############
#RAIS
#It will download files for all states and the selected year
download_sourceData("RAIS", i = "2000")
#To import data from all UFs in a single data.frame
d<- read_RAIS('vinculos', i = 2000)
#To import data from a selected group of UFs
d<- read_RAIS('vinculos', i = 2000, UF = c("DF","GO"))

##############
#PME

#It will download files for all months and the selected year:
download_sourceData("PME", i = "2012.01")
#'Period' argument should with quotes and formated as "YYYY.MM"
d <- read_PME("vinculos", "2012.01")
```

Other options
-------------

### Subsetting datasets

To read only a selected subset of the data, use the argument `vars_subset`, the default is `vars_subset = NULL`, this would result in no subseting.

Example, To read only sex and *percapita* income variable of PNAD 2014, type:

``` r

d<- read_PNAD("pessoas",i = 2014, root_path = path.expand("~/Datasets/PNAD"), 
              vars_subset =  c("V0302","V4750"))
```

### Ignore metadata and read the dataset directly from selected file

If for some reason you renamed a file or folder, our metadata won't work for you and you will need to use the argument `file` to point to which file is to be imported.

In this situation, the command would look like this:

``` r

d <- read_PNAD("pessoas",i = 2014, file = path.expand("~/Datasets/PNAD/pnad_pes.txt"))
```

In this case you will also receive a warning:

`You have specified  the 'file' argument, in this case we will assume that data is in a .txt or .csv file stored in the address specified by the 'file' parameter`

### Get import dictionaries

If you only need the import dictionaries and don't want to use the import functions of the package. Use the function `get_import_dictionary`

``` r

pnad_dic<- get_import_dictionary(dataset = "PNAD",i = 2014, ft = "pessoas")
```

Related efforts
---------------

This package is highly influenced by similar efforts, which are great time savers, vastly used and often unrecognized:

-   Anthony Damico's [scripts to read most IBGE surveys](http://www.asdfree.com/). Great if you your data does not fit into memory and you want speed when working with complex survey design data.
-   [Data Zoom](http://www.econ.puc-rio.br/datazoom/) by Gustavo Gonzaga, Cláudio Ferraz and Juliano Assunção. Similar ease of use and harmonization of Brazilian microdada for Stata.
-   [dicionariosIBGE](https://cran.r-project.org/web/packages/dicionariosIBGE/index.html), by Alexandre Rademaker. A set of data.frames containing the information from SAS import dictionaries for IBGE datasets.
-   [IPUMS](https://international.ipums.org/international/). Harmonization of Census data from several countries, including Brasil. Import functions for R, Stata, SAS and SPSS.

`microdadosBrasil` differs from those packages in that it:

-   updates import functions to more recent years
-   includes non-IBGE data, such as INEP Education Census and MTE RAIS (de-identified)
-   separates import code from dataset specific metadata, as explained below.

How the package works
---------------------

### Traditional Import Workflow

Nowadays packages are normally provided on-line (or in a physical CD for the older IBGE publications) as .zip files with the following structure:

dataset\_year.zip

-   dataset\_year
    -   DICTIONARIES
        -   import\_dictionary\_subdatasetA.SAS
    -   DATA
        -   subdatasetA\_state1.txt
        -   subdatasetA\_state2.txt
        -   ...
        -   subdatasetA\_stateN.txt
    -   ADITIONAL DOCUMENTATION
        -   subdatasetA\_variables\_and\_cathegories\_dictionary.xls

Users then normally manually reconstruct the import dictionaries in R by hand. Then, using this dictionary, run the import function, pointing to the DATA folder. Larger datasets (such as CENSUS or RAIS) come subdivided by state (or region), so the function must be repeated for all states. Then if the user needs more than one year of the dataset, the user repeats all the above, adjuting for changes fine and folder names.

### microdadosBrasil aproach

(soon)

#### Design principles

The main design principle was separating details of each dataset in each year - such as folder structure, data files and import dictionaries - from the original data and placing these into metadata tables (saved as csv files at the `extdata` folder). The elements in these tables, along with the list of import dictionaries extracted from the SAS import instructions from the data provider, serve as parameters to import a dataset for a specific year. This separation of dataset specific details from the actual code makes code shorter and easier to extend to new packages.

ergonomics over speed (develop)
