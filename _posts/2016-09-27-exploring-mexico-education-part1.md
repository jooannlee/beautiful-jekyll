---
layout: post
title: Exploring Mexico’s Education Landscape 
subtitle: Part1 - Basic mapping in R
tags: [R, map, mexico, education]
bigimg: /img/2016-09-27.png
---

Introduction
------------

This is the first of multiple short posts dedicated to exploring Mexico’s education landscape. Here I will introduce the basics of one way you can easily make a map in R.

What you will need
------------------

### Shapefiles

Shapefiles contain the necessary GIS data for us to create maps. There are many websites to download shapefiles from. For this project, I downloaded mine here: <http://www.diva-gis.org/gdata>. Download and unzip the shapefiles to your working directory.

### Packages

The two R packages that we will be using are: 
<br>- maptools 
<br>- ggplot2

OK, step 1
----------

Set your working directory to the folder containing the shapefiles, and read the shapefiles into R to be stored as an object. Here I called my object containing the Mexico shapefiles mxShp.

``` r
setwd("C:/MEX_adm")

library("maptools")
mxShp = readShapeSpatial("MEX_adm1.shp")
```

mxShp is an object with class “Spatial Polygons”, and within it are 16 categories corresponding to the GIS data that represents the states of Mexico.

``` r
class(mxShp)
```

    ## [1] "SpatialPolygonsDataFrame"
    ## attr(,"package")
    ## [1] "sp"

``` r
names(mxShp)
```

    ##  [1] "ID_0"       "ISO"        "NAME_0"     "ID_1"       "NAME_1"    
    ##  [6] "VARNAME_1"  "NL_NAME_1"  "HASC_1"     "CC_1"       "TYPE_1"    
    ## [11] "ENGTYPE_1"  "VALIDFR_1"  "VALIDTO_1"  "REMARKS_1"  "Shape_Leng"
    ## [16] "Shape_Area"

We can explore the categories by just calling it as part of mxShp using a $. Here, we see that there are 31 states in Mexico and 1 federal district called Distrito Federal.

``` r
mxShp$NAME_1
```

    ##  [1] Aguascalientes      Baja California     Baja California Sur
    ##  [4] Campeche            Chiapas             Chihuahua          
    ##  [7] Coahuila            Colima              Distrito Federal   
    ## [10] Durango             Guanajuato          Guerrero           
    ## [13] Hidalgo             Jalisco             México             
    ## [16] Michoacán           Morelos             Nayarit            
    ## [19] Nuevo León          Oaxaca              Puebla             
    ## [22] Querétaro           Quintana Roo        San Luis Potosí    
    ## [25] Sinaloa             Sonora              Tabasco            
    ## [28] Tamaulipas          Tlaxcala            Veracruz           
    ## [31] Yucatán             Zacatecas          
    ## 32 Levels: Aguascalientes Baja California Baja California Sur ... Zacatecas

``` r
mxShp$ID_1
```

    ##  [1] 1804 1805 1806 1807 1808 1809 1810 1811 1812 1813 1814 1815 1816 1817
    ## [15] 1818 1819 1820 1821 1822 1823 1824 1825 1826 1827 1828 1829 1830 1831
    ## [29] 1832 1833 1834 1835

``` r
mxShp$TYPE_1
```

    ##  [1] Estado           Estado           Estado           Estado          
    ##  [5] Estado           Estado           Estado           Estado          
    ##  [9] Distrito Federal Estado           Estado           Estado          
    ## [13] Estado           Estado           Estado           Estado          
    ## [17] Estado           Estado           Estado           Estado          
    ## [21] Estado           Estado           Estado           Estado          
    ## [25] Estado           Estado           Estado           Estado          
    ## [29] Estado           Estado           Estado           Estado          
    ## Levels: Distrito Federal Estado

Step 2
------

Now that we have our shapefiles loaded in R, we need to prepare our dataset to be compatible with the structure of the shapefiles.

I downloaded the Mexico education dataset here: <http://www.snie.sep.gob.mx/estadisticas_educativas.html>. For starters, I’ll focus on the enrollment data.

Convert the dataset into .csv and load the dataset into R. My dataset is loaded as a dataframe called dat.

``` r
dat = read.csv("C:/enrollment.csv")
```

We can see that the variable DES contains information on the states of Mexico in addition to the level of education. We can also see that variable CLV is a level index that marks what level the enrollment data is at.

``` r
head(dat, n=5)
```

    ##   ENT    CLV                               DES BRP X1990.1991 X1991.1992
    ## 1   1 100000   PREESCOLAR TOTAL AGUASCALIENTES BRT     26,612     27,049
    ## 2   1 100100                           GENERAL  BT     25,146     25,146
    ## 3   1 100200                          INDIGENA  BT          0          0
    ## 4   1 100300         CURSOS COMUNITARIOS TOTAL  BT        518        827
    ## 5   1 100400                             CENDI  BT        948      1,076
    ##   X1992.1993 X1993.1994 X1994.1995 X1995.1996 X1996.1997 X1997.1998
    ## 1     27,494     29,870     32,386     32,105     33,691     34,165
    ## 2     25,567     27,957     30,043     29,723     31,057     31,571
    ## 3          0          0          0          0          0          0
    ## 4        701        715      1,011        887        819        770
    ## 5      1,226      1,198      1,332      1,495      1,815      1,824
    ##   X1998.1999 X1999.2000 X2000.2001 X2001.2002 X2002.2003 X2003.2004
    ## 1     35,215     35,862     36,459     35,790     36,988     37,356
    ## 2     32,403     32,932     33,396     32,793     33,783     35,271
    ## 3          0          0          0          0          0          0
    ## 4        793        846        864        827      1,089      1,176
    ## 5      2,019      2,084      2,199      2,170      2,116        652
    ##   X2004.2005 X2005.2006 X2006.2007 X2007.2008 X2008.2009 X2009.2010
    ## 1     38,514     47,507     48,797     49,065     48,239     48,034
    ## 2     36,665     45,776     46,915     47,622     46,793     46,657
    ## 3          0          0          0          0          0          0
    ## 4      1,416      1,340      1,467      1,443      1,446      1,377
    ## 5        133         40          0          0          0          0
    ##   X2010.2011 X2011.2012 X2012.2013 X2013.2014
    ## 1     47,726     48,917     50,250     50,757
    ## 2     46,366     47,400     48,842     49,354
    ## 3          0          0          0          0
    ## 4      1,360      1,517      1,408      1,403
    ## 5          0          0          0          0

Let’s extract preschool enrollment information for all states (level 100000 CLV).

``` r
datCLV100000 = subset(dat, dat$CLV=="100000")
datCLV100000$DES
```

    ##  [1] PREESCOLAR TOTAL AGUASCALIENTES     
    ##  [2] PREESCOLAR TOTAL BAJA CALIFORNIA    
    ##  [3] PREESCOLAR TOTAL BAJA CALIFORNIA SUR
    ##  [4] PREESCOLAR TOTAL CAMPECHE           
    ##  [5] PREESCOLAR TOTAL COAHUILA           
    ##  [6] PREESCOLAR TOTAL COLIMA             
    ##  [7] PREESCOLAR TOTAL CHIAPAS            
    ##  [8] PREESCOLAR TOTAL CHIHUAHUA          
    ##  [9] PREESCOLAR TOTAL DISTRITO FEDERAL   
    ## [10] PREESCOLAR TOTAL DURANGO            
    ## [11] PREESCOLAR TOTAL GUANAJUATO         
    ## [12] PREESCOLAR TOTAL GUERRERO           
    ## [13] PREESCOLAR TOTAL HIDALGO            
    ## [14] PREESCOLAR TOTAL JALISCO            
    ## [15] PREESCOLAR TOTAL MÉXICO             
    ## [16] PREESCOLAR TOTAL MICHOACÁN          
    ## [17] PREESCOLAR TOTAL MORELOS            
    ## [18] PREESCOLAR TOTAL NAYARIT            
    ## [19] PREESCOLAR TOTAL NUEVO LEÓN         
    ## [20] PREESCOLAR TOTAL OAXACA             
    ## [21] PREESCOLAR TOTAL PUEBLA             
    ## [22] PREESCOLAR TOTAL QUERÉTARO          
    ## [23] PREESCOLAR TOTAL QUINTANA ROO       
    ## [24] PREESCOLAR TOTAL SAN LUIS POTOSÍ    
    ## [25] PREESCOLAR TOTAL SINALOA            
    ## [26] PREESCOLAR TOTAL SONORA             
    ## [27] PREESCOLAR TOTAL TABASCO            
    ## [28] PREESCOLAR TOTAL TAMAULIPAS         
    ## [29] PREESCOLAR TOTAL TLAXCALA           
    ## [30] PREESCOLAR TOTAL VERACRUZ           
    ## [31] PREESCOLAR TOTAL YUCATÁN            
    ## [32] PREESCOLAR TOTAL ZACATECAS          
    ## [33] PREESCOLAR TOTAL REPÚBLICA MEXICANA 
    ## 594 Levels:               EDUCACIÓN ESPECIAL CURSOS INTENSIVOS ...

According to our shapefiles, there are 31 states and 1 federal district in Mexico. However our dataset has 33 unique entries. In order to map, we need to get our dataset to be compatible with the shapefiles.

``` r
length(datCLV100000$DES)
```

    ## [1] 33

What’s going on? To find out, let’s extract and append the states from the preschool enrollment information, and convert them to all lower cases.

``` r
datCLV100000$states = substr(datCLV100000$DES, 18, 1000000L)
datCLV100000$states = tolower((datCLV100000$states))
datCLV100000$states
```

    ##  [1] "aguascalientes"      "baja california"     "baja california sur"
    ##  [4] "campeche"            "coahuila"            "colima"             
    ##  [7] "chiapas"             "chihuahua"           "distrito federal"   
    ## [10] "durango"             "guanajuato"          "guerrero"           
    ## [13] "hidalgo"             "jalisco"             "méxico"             
    ## [16] "michoacán"           "morelos"             "nayarit"            
    ## [19] "nuevo león"          "oaxaca"              "puebla"             
    ## [22] "querétaro"           "quintana roo"        "san luis potosí"    
    ## [25] "sinaloa"             "sonora"              "tabasco"            
    ## [28] "tamaulipas"          "tlaxcala"            "veracruz"           
    ## [31] "yucatán"             "zacatecas"           "república mexicana"

Let’s compare what we have with the information in shapefiles. Which is the additional entry from our dataset?

``` r
datShp = data.frame(states=mxShp$NAME_1, id=mxShp$ID_1)
datShp$states = tolower((datShp$states))
datShp$states
```

    ##  [1] "aguascalientes"      "baja california"     "baja california sur"
    ##  [4] "campeche"            "chiapas"             "chihuahua"          
    ##  [7] "coahuila"            "colima"              "distrito federal"   
    ## [10] "durango"             "guanajuato"          "guerrero"           
    ## [13] "hidalgo"             "jalisco"             "méxico"             
    ## [16] "michoacán"           "morelos"             "nayarit"            
    ## [19] "nuevo león"          "oaxaca"              "puebla"             
    ## [22] "querétaro"           "quintana roo"        "san luis potosí"    
    ## [25] "sinaloa"             "sonora"              "tabasco"            
    ## [28] "tamaulipas"          "tlaxcala"            "veracruz"           
    ## [31] "yucatán"             "zacatecas"

``` r
'%nin%' = Negate('%in%')
which(datCLV100000$states %nin% datShp$states)
```

    ## [1] 33

It's entry 33, "república mexicana".

``` r
datCLV100000$states[33]
```

    ## [1] "república mexicana"

Since we are interested in state level data, let’s remove “república mexicana”, and compare our dataset again.

``` r
datCLV100000 = datCLV100000[-c(33),]

datCLV100000 = datCLV100000[order(datCLV100000$states),]
datShp = datShp[order(datShp$states),]
all.equal(datCLV100000$states, datShp$states) 
```

    ## [1] TRUE

Now our dataset is compatible with the shapefiles, and we are ready to start mapping!

Step 3
------

Let’s create the mapping dataset. Here I will focus on data for year 2013 - 2014. Extract the appropriate data, merge it with states and id information, and convert enrollment to numeric.

``` r
enrollment = data.frame(enrollment=datCLV100000$X2013.2014, states=datCLV100000$states)
head(enrollment, n=5)
```

    ##   enrollment              states
    ## 1     50,757      aguascalientes
    ## 2    107,954     baja california
    ## 3     26,561 baja california sur
    ## 4     36,072            campeche
    ## 5    284,104             chiapas

``` r
datMap = merge(enrollment, datShp, by="states")
datMap$enrollment = as.numeric(gsub(",","",datMap$enrollment))
head(datMap, n=5)
```

    ##                states enrollment   id
    ## 1      aguascalientes      50757 1804
    ## 2     baja california     107954 1805
    ## 3 baja california sur      26561 1806
    ## 4            campeche      36072 1807
    ## 5             chiapas     284104 1808

Next, merge our mapping dataset with the shapefiles.

``` r
library("ggplot2")
datShp2 = fortify(mxShp, region="ID_1") #convert shapefiles into dataframe
head(datShp2, n=5)
```

    ##        long      lat order  hole piece   id  group
    ## 1 -102.1261 21.73960     1 FALSE     1 1804 1804.1
    ## 2 -102.1339 21.72841     2 FALSE     1 1804 1804.1
    ## 3 -102.1393 21.72210     3 FALSE     1 1804 1804.1
    ## 4 -102.1407 21.71793     4 FALSE     1 1804 1804.1
    ## 5 -102.1493 21.70852     5 FALSE     1 1804 1804.1

``` r
datMap2 = merge(datShp2, datMap, by="id", all.x=TRUE) 
head(datMap2, n=5)
```

    ##     id      long      lat order  hole piece  group         states
    ## 1 1804 -102.1261 21.73960     1 FALSE     1 1804.1 aguascalientes
    ## 2 1804 -102.1339 21.72841     2 FALSE     1 1804.1 aguascalientes
    ## 3 1804 -102.1393 21.72210     3 FALSE     1 1804.1 aguascalientes
    ## 4 1804 -102.1407 21.71793     4 FALSE     1 1804.1 aguascalientes
    ## 5 1804 -102.1493 21.70852     5 FALSE     1 1804.1 aguascalientes
    ##   enrollment
    ## 1      50757
    ## 2      50757
    ## 3      50757
    ## 4      50757
    ## 5      50757

Finally, we are ready to plot our basic map!

``` r
map = ggplot() + geom_polygon(data=datMap2, aes(x=long, y=lat, group=group, fill=enrollment)) + coord_map()

plot(map)
```

![](/img/2016-09-27-1.png)

Not very surprisingly, we see that Mexico City (used to be known as distrito federal) has the highest preschool level enrollment for year 2013 - 2014. Stay tuned for more!
