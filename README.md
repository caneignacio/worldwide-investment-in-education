# Worldwide Investment in Education

## Table of Contents

- [Overview](#overview)
- [Tools and Technologies](#tools-and-technologies)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Modelling](#data-modelling)
- [Power BI Report](#power-bi-report)
- [Key Insights](#key-insights)
- [Informational Video](#informational-video)
- [Data Source](#data-source)
- [References](#references)


### Overview

The following data analysis project investigates the level of education in different countries all over the world by studying three sets of data, which respectively represent: a) government expenditure in education, b) quantity of graduates of tertiary programmes, c) quantity of researchers.

Education constitutes a critical factor for individual and social development. It can be safely affirmed that strong investment in education drives economic growth, mitigates inequality, and enables countries to remain competitive in the global economic (Gupta, 2025).

The main goal of this work is to examine the correlation between public investment, education level, and research workforce, producing a report which can be consumed by people all over the world. Moreover, an informational video will be developed, showcasing the main conclusions, designed as if it were authored by _Policy Checkers_, a fictional NGO.

### Tools and Technologies

- Python (Exploratory Data Analysis)
- Power BI (Data Modelling and Data Visualization)
- DAX (Advanced Indicators)
- Adobe After Effects and Adobe Illustrator (Informational video)

### Exploratory Data Analysis

The available data for this project consists in two datasets and their respective metadata files. They are available in the [UNESCO Institute for Statistics](https://databrowser.uis.unesco.org/) website. According to the information in the metadata files, the datasets contain the following data:

- data-xgdp.csv: Government expenditure on education as a percentage of GDP, for different countries and years.

- data-ea.csv: Percentage of people that completed short-cycle tertiary education or higher, for different countries and years.

- data-resden.csv: Number of researchers per million inhabitants.

As a first step, an initial exploration was performed in Python for each of the files. The code can be found in the respective “initial-exploration” python files (one for each dataset) on this Github repository.

The steps taken and main results can be summed up as follows:

**General information**:

All datasets were queried to investigate their number of rows, number of columns, and name of each column. A short random sample of each dataset was taken too. Some of the commands used were:

`df = pd.read_csv('./data.csv')`

`df.shape[0] `

`df.info()`

`df.sample(n=5, random_state=1) `

The results of these initial queries are the following:

|Table|Num. of Rows|Num. of Columns|
|---|---|---|
|data-xgdp|3551|6|
|data-ea|1793|6|
|data-resden|2857|6|

**Null values**:

The number of null values per column was obtained with the `df.isnull().sum()` method. In all datasets, there are two columns which have little to no values. These columns represent additional characteristics of the values measured for any given row and will not be considered for the analysis.

data-xgdp.csv:

|Column|% of Null values|
|---|---|
|qualifier|96.8%|
|magnitude|100%|

data-ea.csv:

|Column|% of Null values|
|---|---|
|qualifier|55.3%|
|magnitude|99.7%|

data-resden.csv:

|Column|% of Null values|
|---|---|
|qualifier|97%|
|magnitude|100%|

**Data types**:

The data type of each column was checked by using `df.dtypes`. For all the datasets, two columns are text-type or mixed (“object” type in Pandas, representing codes), whereas other two are number-type (as was already stated, there are two other columns that will not be considered).

The text-type or mixed columns are:

- indicatorId: text that represents the value measured with a code (e.g. percentage of graduates).

- geoUnit: text that represents the country with a code. 

The number-type columns are:

- year: integer that represents years (to be considered date-type).

- value: float that represents the measured value.

**Statistical summary**:

Some general statistical indicators were drawn for numeric columns, using the `df.describe()` method.

data-xgdp.csv

|Column|Minimum|Maximum|Mean|Standard deviation|
|---|---|---|---|---|
|year|2000|2024|2012.189524|6.758061|
|value|0.127174|16.390530|4.423399|1.916085|

data-ea.csv

|Column|Minimum|Maximum|Mean|Standard deviation|
|---|---|---|---|---|
|year|2000|2024|2014.517568|5.595144|
|value|0|73.910004|21.667260|13.493808|

data-resden.csv

|Column|Minimum|Maximum|Mean|Standard deviation|
|---|---|---|---|---|
|year|2000|2023|2011.529226|6.597518|
|value|5.937380|20047.046520|1760.505542|1905.507707|

It should be noted that, for the “year” columns, the mode, i.e., the most frequent value, might be a better measure than the arithmetic mean. Therefore, it was also calculated using `df[“Column”].mode()`:

- data-xgdp.csv: 2017

- data-ea.csv: 2019

- data-resden.csv: 2015

**Cardinality**:

For each dataset, the number of distinct values per column was checked with the following code:

```
for col in df.columns:
    dupe = df[col].duplicated().any()
    if not dupe:
        print(f"{col}", df[col].duplicated().any())

```

For the three tables, all columns have duplicates, so none of them can work as a key on its own. However, for all datasets the concatenation of “geoUnit” and “year” works as a natural key.

### Data Modelling

A semantic model was built on Power BI, with three fact tables: _Expenditures_, _Graduates_, and _Researchers_ (respectively based on data-xgdp.csv, data-fosgp.csv, and data-resden.csv), and two other dimension tables: _Dates_ and _Countries_.

The data from the datasets was first uploaded to Power BI and transformed in the PowerQuery editor. The following transformations were performed all fact tables:

- The columns “qualifier” and “magnitude” were dropped, since they would not be considered for this analysis. The “indicatorId" column was also dropped, since it assigned the same value to all rows.

- Data types were revised and changed when needed.

- Some column names and values were replaced into a more user-friendly wording.

- A “Date” calculated column was created for better application of DAX time intelligence functions.

After generating the fact tables, a _Dates_ calculated table was created to work as a calendar table, and a one-to-many relationship was established from this table’s “Date” column to the fact tables’ “Date” columns.

In addition, a _Countries_ calculated table was created with a web connection to the [CODE FOR IATI](https://codelists.codeforiati.org/RegionM49/) site, that replicates UN’s “Standard Country or Area Codes for Statistical Use”. A one-to-many relationship was established from this table’s “Code” column to the fact tables’ “Country” columns.

Finally, two calculated columns were created in the _Countries_ table: “Cluster_Graduates” and “Cluster_Researchers”. These columns classify countries in tertiles, regarding their Expenditure, Graduates, and Researchers values.

The “Cluster_Graduates” column assigns the “Low” value to the country if it is on the lower tertile for Expenditure or Graduates, the “Medium” value if both are in the medium tertile or higher, and the “High” value if both are in the higher tertile. “Cluster_Researchers” works the same way, but considering Expenditure and Researchers values.

The following image shows the definitive semantic model:

![](/model.bmp)

### Power BI Report

A report was built upon the model. The editable version can be found in the [unescoeducation-uis.pbix] file on this Github repository. There is also an open link to a read-only version of the report on [this link](report.link).

The report consists on a front cover and three more pages, with the following elements:

- _Front Cover_:

  - This page shows the title of the project and buttons to navigate to the rest of the report, with icons representing the different pages. The rest of the pages have a panel on the top section with the same icons, which allows for navigation to other pages, and a button to return to this cover.

- _Top Ranked_:

  - On the top of the page, there is the navigation panel, with the title and different buttons.

![](/top-ranked-top.png.bmp)

  - On the left side, there is a map with colour fills of different intensity, regarding the level of Expenditure, Graduates and Researchers. The right side shows a bar chart with the top countries by Expenditure, Graduates and Researchers. For both visualizations, the three buttons on the right can be clicked to choose which one of the three variables to display.

![](/top-ranked-bottom.png.bmp)

- _All Countries_:

  - On the top of the page, there is the navigation panel, with the title and different buttons. There is no additional info button in this page, since it was not considered necessary.

![](/all-countries-top.png.bmp)

  - There are two scatter plot visualizations in this page, representing the distribution of Expenditure vs Graduates level, and Expenditure vs Researchers level, respectively. The colours on the visuals correspond to “Cluster_Graduates” and “Cluster_Researchers”, the cluster columns already discussed.

![](/all-countries-bottom.png.bmp)

- _Evolution_:

  - Once again, the top of the page shows the navigation panel, with the title and its buttons.

![](/evolution-top.png.bmp)

  - The left side of the page has a line chart visualization that compares the level of Expenditure, Graduates, and Researchers, over time. The variables can be chosen a pair at a time.

![](/evolution-left.png.bmp)

  - There are three buttons on the right side, that can be used to choose the desired pair of variables.

![](/evolution-right1.png.bmp)

  - Finally, there are three slicers. The first one can be used to lag the Expenditure variable, to check if there is a delayed correlation between this variable and the others. The second and third slicers allow filtering by Graduates level or Researchers level (these levels are the tertiles shown in the scatter plot visuals on the previous page).

![](/evolution-right2.png.bmp)

### Key Insights

This report is intended to allow people all around the world to draw insights about investment in education and its potential correlations to the numbers of tertiary graduates and researchers. The following conclusions could be pointed out:

- The top 10 countries for each variable span different continents. In particular, the top 3 countries for each variable are:

  - Top 3 expenditure (Oceania):
    - Tuvalu
    - American Samoa 
    - Kiribati

  - Top 3 graduates:
    - Kazakhstan (Asia)
    - Canada (America)
    - Uzbekistan (Asia)

  - Top 3 researchers (Europe):
    - Lichtenstein
    - Finland
    - Iceland

- There are 10 countries in the higher tertile both for expenditure and percentage of graduates. These countries are Belgium, Cyprus, Denmark, Finland, Iceland, Israel, Montserrat, Norway, Sweden, and USA. There are no countries in the higher tertile both for expenditure and number of researchers.

- Considering all countries, there is no simple correlation between the level of expenditure in education, percentage of tertiary graduates, and number of researchers. However, when filtering by the higher tertile for expenditure and percentage of graduates, a positive correlation can be seen between all variables.

## Informational Video

As it was previously stated, an informational video was developed with the main conclusions of this analysis. It was designed as if it were authored by _Policy Checkers_, a fictional NGO which informs the public about different policies worldwide and their impact.

The informational video can be browsed via Youtube, by clicking [this link](https://www.youtube.com/watch?v=aaXTHNQGvqU).

### Data Source

UNESCO Institute for Statistics (UIS), [https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG4Monitoring/t4.4/i4.4.3](https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG4Monitoring/t4.4/i4.4.3), July 18th 2025.

UNESCO Institute for Statistics (UIS), [https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG9Monitoring/t9.5/i9.5.2?highlightId=i9.5.2](https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG9Monitoring/t9.5/i9.5.2?highlightId=i9.5.2), July 16th 2025.

UNESCO Institute for Statistics (UIS), [https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG4Monitoring/t1.a/i1.a.2?highlightId=XGOVEXP.IMF&highlightGroupId=IG-XGOVEXP.IMF](https://databrowser.uis.unesco.org/browser/EDUCATION/UIS-SDG4Monitoring/t1.a/i1.a.2?highlightId=XGOVEXP.IMF&highlightGroupId=IG-XGOVEXP.IMF), June 27th 2025.

UN Statistics Division (UNSD), [https://unstats.un.org/unsd/methodology/m49/overview/](https://unstats.un.org/unsd/methodology/m49/overview/), extracted from [CODE FOR IATI](https://codelists.codeforiati.org/RegionM49/).

### References

Gutpa S. (2025, February 25). Funding education as an investment in the future. [https://www.unesco.org/sdg4education2030/en/articles/funding-education-investment-future]( https://www.unesco.org/sdg4education2030/en/articles/funding-education-investment-future)

World map image in informational video by Al MacDonald [2]/ twitter account @F1LT3R, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=7469140

Music in informational video: Expedition by Alex-Productions https://soundcloud.com/alexproductionsmusic
License: Creative Commons — Attribution 3.0 Unported — CC BY 3.0
Free Download / Stream: https://www.audiolibrary.com.co/alex-productions/expedition
Music promoted by Audio Library: https://youtu.be/-_CEmB_dHpA
