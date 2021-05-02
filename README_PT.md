
<!-- README.md is generated from README.Rmd. Please edit that file -->
microdadosBrasil
================

Trabalho em andamento
---------------------

### NOVIDADES:

-   Censo 2010
-   RAIS
-   CAGED
-   PME

-   Não usa R? Veja: [using the package from Stata and Python](https://github.com/lucasmation/microdadosBrasil/blob/master/vignettes/Running_from_other_software.Rmd)

### EM BREVE:

-   Suporte para leitura de dados fora da memória RAM
-   Harmonização do nome de variáveis ao longo dos anos

DESCRIÇÃO
---------

Esse pacote disponibiliza funções para importar as bases mais comuns de microdados brasileiros. Importar estes microdados pode ser tedioso. A maior parte dos dados é disponibilizada em arquivos do tipo txt colunado (fixed width files, fwf) e, geralmente, contém scripts de importação somente para SAS e SPSS. Os dados algumas vezes vêm subdivididos em muitos arquivos, por UF ou Região. Além disso é comum que nomes de arquivos e de variáveis de certa base de dados variem ao longo do tempo. `microdadoBrasil` cuida desses detalhes pra você. Internamente o pacote está rodando `readr` para arquivos fwf e `data.table` aquivos separados por delimitadores (csv). Assim, a importação é rápida.

Atualmente, o pacote inclui funções de importação para as seguintes bases de dados:

| Fonte | Dataset                 | Função                      | Período            | Subdataset                 |
|:------|:------------------------|:----------------------------|:-------------------|:---------------------------|
| IBGE  | PNAD                    | read\_PNAD                  | 2001 to 2014       | domicilios, pessoas        |
| IBGE  | Censo Demográfico       | read\_CENSO                 | 2000               | domicilios, pessoas        |
| IBGE  | PME                     | read\_PME                   | 2002.01 to 2015.12 | vinculos                   |
| IBGE  | POF                     | read\_POF                   | 2008               | several, ver detalhes      |
| INEP  | Censo Escolar           | read\_CensoEscolar          | 1995 to 2014       | escolas, ..., ver detalhes |
| INEP  | Censo da Educ. Superior | read\_CensoEducacaoSuperior | 1995 to 2014       | ver detalhes               |
| MTE   | CAGED                   | read\_CAGED                 | 2009.01 to 2016.05 | vinculos                   |
| MTE   | RAIS                    | read\_RAIS                  | 1998 to 2014       | estabelecimentos, vinculos |

Para os dados em formato fwf, o pacote inclui, internamente, dicionários de importação. Esses dicionários foram criados com a função `import_SASdictionary()`, que pode ser utilizado pelo usuário para construir, a partir de um dicionário SAS, dicionários não incluídos no pacote. Dicionário incluídos no pacote podem ser acessados com a função `get_import_dictionary`.

O pacote também harmoniza nomes de arquivos e a estrutura das pastas ao longo tempo, através de uma tabela de metadados, tornando possível a importação de bases de dados que usualmente vem dividadas em subgroupos regionais (por UF ou região) em um único objeto.

INSTALAÇÃO
----------

``` r
install.packages("devtools")
install.packages("stringi") 
devtools::install_github("lucasmation/microdadosBrasil")
library('microdadosBrasil')
```

UTILIZAÇÃO
----------

``` r
# Censo Demográfico 2000
#Depois de ter baixado e descompactado os arquivos em seu diretório de trabalho , rode:
d <- read_CENSO('domicilios',2000)
d <- read_CENSO('pessoas',2000)

#Para importar os dados a partir de uma pasta diferente de seu atual diretório de trabalho, use 
d <- read_CENSO('domicilios',2000, root_path ="C:/....")
#Para restringir a importação para apenas uma UF, use:
d <- read_CENSO('pessoas',2000, UF = "DF")

# PNAD 2002
download_sourceData("PNAD", 2002, unzip = T)
d  <- read_PNAD("domicilios", 2002)
d2 <- read_PNAD("pessoas", 2002)

# Censo Escolar
download_sourceData('CensoEscolar', 2005, unzip=T)
d <- read_CensoEscolar('escola',2005)
d <- read_CensoEscolar('escola',2005,harmonize_varnames=T)

#RAIS
#Para tentar baixar os dados de todo o ano de 2000 e todas as UFs
download_sourceData("RAIS", i = "2000")
#Para ler os dados de todas as UFs:
d<- read_RAIS('vinculos', i = 2000)
#Para ler os dados de UFs selecionadas:
d<- read_RAIS('vinculos', i = 2000, UF = c("DF","GO"))

#PME

#Irá baixar os dados para todo o ano de 2012, pois estes vem em um único arquivo:
download_sourceData("PME", i = "2012.01")
#O período deve ser inserido entre aspas e no formato YYYY.MM
d <- read_PME("vinculos", "2012.01")
```

ESFORÇOS RELACIONADOS
---------------------

Esse pacote foi altamente influenciado por esforços similares, que são grande poupadores de tempo, muito utilizados e, algumas vezes, não reconhecidos:

-   [Scripts para ler a maioria das pesquisas do IBGE](http://www.asdfree.com/) de Anthony Damico. Excelente se seus dados não cabem na memória RAM e você quer velociadade para trabalhar com dados de amostras complexas.
-   [Data Zoom](http://www.econ.puc-rio.br/datazoom/) por Gustavo Gonzaga, Cláudio Ferraz e Juliano Assunção. Esforço de simplificação para o software Stata. Além da importação, harmoniza nomes das variáveis.
-   [dicionariosIBGE](https://cran.r-project.org/web/packages/dicionariosIBGE/index.html), por Alexandre Rademaker. Conjunto de data.frames contendo a informação dos dicionários de importação do SAS. .
-   [IPUMS](https://international.ipums.org/international/). Harmonização de dados microdados de CENSO de vários países, incluindo o Brasil. Funções de importação para R, Stata, SAS e SPSS.

`microdadosBrasil` Se diferencia destes pacotes por:

-   Trazer opções de importação para períodos mais recentes
-   Incluir dados de outras fontes, além do IBGE, como Censo Escolar (do INEP) e a RAIS (do MTE).
-   Separar código pra importação e os metadados específicos de cada base de dados, como explicado abaixo:

#### Princípios de concepção do pacote

O principal princípio utilizado na construção do pacote foi separar os detalhes de cada base de dados, como a estrutura de pastas e nome de arquivos em tabelas de metadados (salvos como arquivos .csv na pasta `extdata`). O conteúdo dessas tabelas, assim como uma lista contendo os dicionários de importação extraídos dos dicionários oficiais em formato SAS, seve como parâmetro para a importação dos microdados para cada ano. Essa separação entre detalhes específicos de cada base de dados e código torna o código mais simples e generalizável, facilitando a extensão para novas base de dados.

ergonomics over speed (develop)
