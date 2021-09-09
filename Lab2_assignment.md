Lab2\_assignment
================
Matt Harris
9/8/2021

``` r
library(tidyverse)
library(tidycensus)
library(sf)
library(tmap) # mapping, install if you don't have it
set.seed(717)
```

This assignment if for you to complete a short version of the lab notes,
but you have to complete a number of the steps yourself. You will then
knit this to a markdown (not an HTML) and push it to your GitHub repo.
Unlike HTML, the RMarkdown knit to `github_document` can be viewed
directly on GitHub. You will them email your lab instructor with a link
to your repo.

Steps in this assignment:

1.  Make sure you have successfully read, run, and learned from the
    `MUSA_508_Lab2_sf.Rmd` Rmarkdown

2.  Find two new variables from the 2019 ACS data to load. Use `vars <-
    load_variables(2019, "acs5")` and `View(vars)` to see all of the
    variable from that ACS. Note that you should not pick something
    really obscure like count\_38yo\_cabinetmakers because you will get
    lots of NAs.

3.  Pick a neighborhood of the City to map. You will need to do some
    googling to figure this out. Use the [PHL Track Explorer](c) to get
    the `GEOID10` number from each parcel and add them to the `myTracts`
    object below. This is just like what was done in the exercise, but
    with a different neighborhood of your choice. Remember that all
    GEOIDs need to be 10-characters long.

4.  In the first code chunk you will do that above and then edit the
    call-outs in the dplyr pipe sequence to `rename` and `mutate` your
    data.

5.  You will transform the data to `WGS84` by adding the correct EPSG
    code. This is discussed heavily in the exercise.

6.  You will produce a map of one of the variables you picked and
    highlight the neighborhood you picked. There are call-out within the
    `ggplot` code for you to edit.

7.  You can run the code chunks and lines of code as you edit to make
    sure everything works.

8.  Once you are done, hit the `knit` button at the top of the script
    window (little blue knitting ball) and you will see the output. Once
    it is what you want…

9.  Use the `Git` tab on the bottom left of right (depending on hour
    your Rstudio is laid out) and click the check box to `stage` all of
    your changes, write a commit note, hit the `commit` button, and then
    the `Push` button to push it to Github.

10. Check your Github repo to see you work in the cloud.

11. Email your lab instructor with a link\!

12. Congrats\! You made a map in code\!

## Load data from {tidycensus}

``` r
Keep <- c(88,89,93,94,99,100,104,105)

ACS2019_Asian_Codes <- load_variables(2019,
                                      "acs5",
                                      cache = TRUE) %>%
                        filter(grepl('ASIAN', concept)) %>%
                        slice(Keep)
ACS2019_Asian_Codes$label <- str_remove(ACS2019_Asian_Codes$label, "Estimate!!Total!!")
ACS2019_Asian_Codes$label <- str_replace(ACS2019_Asian_Codes$label, "!!"," ")
ACS2019_Asian_Codes$label <- str_replace(ACS2019_Asian_Codes$label, "!!"," ")

ACS2019_Asian_Codes <- ACS2019_Asian_Codes %>%
  mutate(Sex = ifelse(grepl('Male',label) == TRUE, 'Male', 'Female'),
         Immigrant_status = ifelse(grepl('Native',label) == TRUE, 'Native', 'Foriegn'),
         Age = ifelse(grepl('Under',label) == TRUE, 'Under_18', 'Over_18'),
         Code = paste(substr(Sex,1,1),substr(Immigrant_status,1,1),Age, sep = ""),
         Merge_name = paste(name,"E", sep ="")) %>%
  select(-3)

myTracts <- c("42101037200","42101980700","42101005000","42101980600","42101037300", "42101003902")

acsTractsPHL.2019.sf <- get_acs(geography = "tract",
                             year = 2019,
                             variables = ACS2019_Asian_Codes$name,
                             geometry = TRUE,
                             state  = "PA",
                             county = "Philadelphia",
                             output = "wide") %>%
                             dplyr::select (GEOID, NAME, all_of(paste0(ACS2019_Asian_Codes$name,"E"))) %>%
  rename(MNUnder_18 = 3,
         MFUnder_18 = 4,
         MNOver_18 = 5,
         MFOver_18 = 6,
         FNUnder_18 = 7,
         FFUnder_18 = 8,
         FNOver_18 = 9,
         FFOver_18 = 10) %>%
  mutate(Neighborhood = ifelse(GEOID %in% myTracts,
                               "SOUTH PHILLY",
                               "REST OF PHILADELPHIA"),
         Natural_Total = MNUnder_18 + MNOver_18 + FNUnder_18 + FNOver_18,
         Foriegn_Total = MFUnder_18 + MFOver_18 + FFUnder_18 + FFOver_18,
         Percent_Natural = Natural_Total/(Natural_Total + Foriegn_Total))
```

## Transform to WGS84 with {sf}

``` r
acsTractsPHL.2019.sf <- acsTractsPHL.2019.sf %>% 
  st_transform(crs = "EPSG:4326")
```

## Plot with {ggplot2}

![](Lab2_assignment_files/figure-gfm/ggplot_geom_sf-1.png)<!-- -->
