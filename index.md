---
title       : Processing the French Home Office elections return files
subtitle    : Coursera Developing Data Products Assignment
author      : Joël Gombin
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
---


## The problem

The French Home Office releases elections returns, broken down by local subdivisions. Unfortunately, these files are not directly usable for data analysis, because they are not in tabular format - in particular when the electoral offer is the same everywhere. Transforming it into tabular data is quite difficult in a spreadsheet editor - that would necessit to write a macro. 

The files look somewhat like this:

<!-- html table generated in R 3.1.0 by xtable 1.7-3 package -->
<!-- Sun Jun 22 01:36:53 2014 -->
<TABLE border=1>
<TR> <TH>  </TH> <TH> Nom </TH> <TH> Prénom </TH> <TH> Nuance </TH> <TH> Voix </TH> <TH> Nom.1 </TH> <TH> Prénom.1 </TH> <TH> Nuance.1 </TH> <TH> Voix.1 </TH>  </TR>
  <TR> <TD align="right"> 1 </TD> <TD> THOMAS </TD> <TD> GILBERT </TD> <TD> DVG </TD> <TD align="right"> 2548 </TD> <TD> BILLOUDET </TD> <TD> GUY </TD> <TD> UMP </TD> <TD align="right"> 2452 </TD> </TR>
  <TR> <TD align="right"> 2 </TD> <TD> LARMANJAT </TD> <TD> Guy </TD> <TD> SOC </TD> <TD align="right"> 2921 </TD> <TD> PETIT </TD> <TD> REGIS </TD> <TD> UMP </TD> <TD align="right"> 1781 </TD> </TR>
  <TR> <TD align="right"> 3 </TD> <TD> TRAVERS </TD> <TD> J.CLAUDE </TD> <TD> UMP </TD> <TD align="right"> 2239 </TD> <TD> DUPONT </TD> <TD> MARC </TD> <TD> SOC </TD> <TD align="right"> 1236 </TD> </TR>
   </TABLE>


So I decided to write a dedicated R package, available on my [Github Page](https://github.com/joelgombin/LireMinInterieur). It can be installed by running `devtools::github_install("joelgombin/LireMinInterieur")`.



---

## The solution

At the heart of the solution is a double loop. Loops are supposed to be slow in R, and a first it was. But by using matrixes rather than dataframes, and by creating an object of the proper size at first, it actually gets quite efficient (because R doesn't have to make copies of the target object).


```r
res2 <- matrix(0, nrow = dim(res1)[1], ncol = length(nuances) + length(candidats), 
    dimnames = list(c(), c(nuances, candidats)), byrow = TRUE)



for (i in 1:length(nuances)) {
    for (j in 1:dim(etiquettes)[1]) {
        index <- which(etiquettes[j, ] == nuances[i])
        res2[j, nuances[i]] <- sum(valeurs[j, index])
        res2[j, candidats[i]] <- sum(length(index))
    }
}
```

---

## The shiny app

The ShinyApp lives at [http://www.joelgombin.fr:3838/shiny/LireMinInterieurEng/](http://www.joelgombin.fr:3838/shiny/LireMinInterieurEng/) (it couldn't manage to have it running smoothly on ShinyApps.io, apparently for encoding issues - I reported the issue to ShinyApps.io).

<iframe src="http://www.joelgombin.fr:3838/shiny/LireMinInterieurEng" width=650 heigt=300></iframe>

---

## Example

An example dataset can be downloaded [here](https://raw.githubusercontent.com/joelgombin/DataProductsPres/gh-pages/data/Canto%2004%20R%C3%A9sultats%20Can%20%20FE%20T1.csv).

* Go to [the app](http://www.joelgombin.fr:3838/shiny/LireMinInterieurEng/).
* Upload the file.
* Field separators are semi-colons; decimal separators are commas.
* Load the dataset.
* Keep the `Code.du.département, Code.du.canton, Inscrits, Abstentions, Votants, Blancs.et.nuls, Exprimés` columns. You shouldn't need to rename them.
* The first column with political labels is called `Nuance`. 
* There are 8 columns between each political label column.
* There is 1 column between the political labels and the vote count.
* Press the `process file button`, switch to the `File after processing` tab, *et voilà* !

