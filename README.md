Open crime data (in R)
================
Matt Ashby
9 December 2020

These brief notes are for the workshop ‘Open crime data in R’ at the
[Methods@Manchester](https://www.methods.manchester.ac.uk/) hackathon on
‘Exploring the Relationship between Crowds, Safety and Crime’, held
remotely on 9 December 2020.

## What is open crime data?

“[Open data and content can be freely used, modified, and shared by
anyone for any purpose](https://opendefinition.org/)”, typically under a
[Creative Commons](https://creativecommons.org/) or similar licence.
Open crime data typically refers to incident-level data about crime (or
related topics such as police activity) that is made available under an
open-data licence.

## Why use open crime data?

Open crime data is often much faster to access, compared to obtaining
data directly from an agency. Negotiating data-sharing agreement and
obtaining security clearances for researchers often delays research for
months (or even years), whereas open data can be downloaded immediately
it is released (and many agencies update their open data daily or
weekly). This means open data can be used to carry out fast-time
research, which can be particularly useful where an answer to a research
question is needed quickly (e.g. to respond to a government
consultation).

Not needing to negotiate data access directly with agencies also means
that it is more feasible to do comparative research using data from
multiple agencies. This is important because it allows us to identify
how generaliable the results of our work are to other settings.

Since open crime data can be accessed by anyone, it does not have to be
handled on secure servers or the results restricted to a few people.
This can be particularly useful when using data internationally, since
there is no need to comply with restrictions on the handling of
sensitive data that existing in many countries.

## Why not to use open crime data

If you need data fields that aren’t included in the open data provided
by a particular agency, obviously open data is of no use to you. It’s
very rare for agencies to release all the fields in their own data as
part of their open data extracts, so there will almost always be more
fields available if you approach the agency directly.

Open data is typically not suitable for research that is using an
individual or a point location as the unit of analysis, since names are
typically redacted ([although not in
Dallas](https://www.dallasopendata.com/Public-Safety/Police-Incidents/qv6i-rri7))
and locations often obfuscated (e.g. by snapping locations to the
centroid of the nearest street segment) to maintain victim privacy.

## A tour of open crime data sources

For England, Wales and Northern Ireland (but not Scotland),
[data.police.uk](https://data.police.uk) provides incident-level open
crime data, but there is no date stamp (only the month in which the
crime occurred) and offences are grouped into a small number of broad
categories.

The [Police Data Initiative](https://www.policedatainitiative.org)
provides a directory of open crime and other police data (e.g. arrests,
traffic stops) from various US agencies, but there are lots of US
agencies that produce open crime data but aren’t listed in the
directory.

The [Open Data Network](http://www.opendatanetwork.com) provides a tool
for searching the metadata of datasets hosted by Socrata, the main
provider of open-data platform software in the US, but there is no
advanced search feature so this often involves trawling through large
numbers of results. Note that many agencies refer to their crime records
as ‘incidents’ rather than ‘crimes’.

[ArcGIS Hub](https://hub.arcgis.com/search) provides a similar search
tool for open-data websites that they host, although again the tagging
of metadata by different agencies is very hit and miss, so it’s often
better to do a more general search and then browse through the results
manually.

Some US agencies provide open data but don’t use one of the big
platforms (e.g. [Portland,
OR](https://www.portlandoregon.gov/police/76454)), making their data
easier to find by Googling ‘\[city name\] open data’ and browsing from
there.

The [Crime Open Database](https://osf.io/zyaqn/) provides harmonised
crime data for 16 large US cities, although the process of harmonising
data from different cities operating under different laws is [inevitably
imperfect](https://doi.org/10.1163/24523666-00401007).

## Using the `crimedata` package in R

The [Crime Open Database](https://osf.io/zyaqn/) (CODE) contains about
22.5 million crime records from 16 of the largest cities in the United
States from 2007 onwards. Each record includes the type, date/time and
approximate location of a crime.

You can access CODE directly in R using the `crimedata` package. Use the
function `get_crime_data()` to download data for specific cities or
specific years:

``` r
# install package if not already installed
# install.packages("crimedata")

library("crimedata")

crime_data <- get_crime_data(years = 2019, cities = "San Francisco")
```

    ## Downloading list of URLs for data files. This takes a few seconds but is only done once per session.

    ## Downloading sample data for San Francisco in 2019

    ##   |                                                                              |                                                                      |   0%  |                                                                              |======================================================================| 100%
    ##   |                                                                              |                                                                      |   0%  |                                                                              |====================                                                  |  29%  |                                                                              |==========================================                            |  60%  |                                                                              |=================================================                     |  70%  |                                                                              |========================================================              |  79%  |                                                                              |==================================================================    |  94%  |                                                                              |======================================================================| 100%

    ## Warning: Unknown columns: `location_type`, `location_category`

The first call to `get_crime_data()` takes a while to run (you may need
to get a coffee) because it downloads a list of URLs in the background
for the current locations of the data files. This is only done once per
session, so subsequent calls are faster.

Some cities do not provide as many data fields as others, so you may see
(as above) a warning that some columns (in this case, `location_type`
and `location_category`) are missing from the data you have requested –
this is not an error with your code, just a reflection of San Francisco
open crime data not including a location-type field.

``` r
crime_data
```

    ## # A tibble: 1,040 x 10
    ##       uid city_name offense_code offense_type offense_group offense_against
    ##     <int> <fct>     <fct>        <fct>        <fct>         <fct>          
    ##  1 1.99e7 San Fran… 13B          simple assa… assault offe… persons        
    ##  2 1.99e7 San Fran… 290          destruction… destruction/… property       
    ##  3 1.99e7 San Fran… 22B          non-residen… burglary/bre… property       
    ##  4 1.99e7 San Fran… 90J          trespass of… trespass of … society        
    ##  5 1.99e7 San Fran… 520          weapon law … weapon law v… society        
    ##  6 1.99e7 San Fran… 23F          theft from … larceny/thef… property       
    ##  7 1.99e7 San Fran… 22A          residential… burglary/bre… property       
    ##  8 1.99e7 San Fran… 23H          all other l… larceny/thef… property       
    ##  9 1.99e7 San Fran… 23H          all other l… larceny/thef… property       
    ## 10 1.99e7 San Fran… 290          destruction… destruction/… property       
    ## # … with 1,030 more rows, and 4 more variables: date_single <dttm>,
    ## #   longitude <dbl>, latitude <dbl>, census_block <chr>

Because the data available is quite large, by default `get_crime_data()`
downloads only a 10% sample of the crime data requested. This allows you
to refine your request before downloading the full dataset by adding the
`type = "core"` parameter to your function.

There is a lot more information about CODE data, including details of
the crime and location categories, on the [CODE project
website](https://osf.io/zyaqn/wiki).
