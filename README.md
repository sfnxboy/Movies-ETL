# Extract, Transform, Load

### Tools Used:
- Python
- PostgreSQL

## Objective
In this project we use the ETL (extract, tranform, load) process to showcase a standard data pipeline.  A data pipeline moves data from a source to a destination, and the ETL process creates data pipelines that also transform the data along the way. Analysis is impossible without access to good data, so creating data pipelines is often the first step before any analysis can be performed. Therefore, understanding ETL is an essential skill for data analysis. We are tasked with creating data sets of popular movies from two data sources, a scrape of Wikipedia for all movies released since 1990, and rating data from Movie Land's website. We will extract the data from the two sources, transform the data into one clean data set, and load that data set into a SQL table.



## Notes

ETL is a flexible process for moving data. It can be as simple as a one-time migration from one database to another, or as complex as an ongoing automated collection of messy, real-time data from many different sources.

- In the Extract phase, data is pulled from external or internal sources, possibly disparate. The sources could be flat files, scraped webpages in HTML or JavaScript Object Notation (JSON) format, SQL tables, or even streams of sensor data. The extracted data is held in a staging area in between the data sources and data targets.

- After data is extracted, there are many transformations it may need to go through. The data may need to be filtered, parsed, translated, sorted, interpolated, pivoted, summarized, aggregated, merged, or more. The goal is to create a consistent structure in the data. Without a consistent structure in our data, it's almost impossible to perform any meaningful analysis.

- The transformation phase can be accomplished with Python and Pandas, pure SQL, or specialized ETL tools like Apache Airflow or Microsoft SQL Server Integrated Services (SSIS). Python and Pandas are especially good for prototyping an ETL transformation because they provide flexibility and interactivity (especially in a Jupyter Notebook), without enforcing any complicated frameworks. We use Python and Pandas to explore, document, and perform our data transformation.

### Extraction  
Its important to load the appropriate dependencies before attempting to extract data. To load a JSON file into a pandas dataframe, we must import the following libraries as so:  
```
import json
import pandas as pd
import numpy as np
```  
The following code allows us to load the JSON file into a list of dictionaries, and test to see if the data loaded properly.  
```
with open(f'{file_dir}/wikipedia.movies.json', mode='r') as file:
    wiki_movies_raw = json.load(file)
len(wiki_movies_raw)
# First 5 records
wiki_movies_raw[:5]
```  
There are 7,311 data points, or records of movies, but is this a reasonable number of records? One way to check that 7,311 is reasonable is to look at the rate of movies being released that it implies. Rough mental math here is the key—we want these calculations to be quick enough that these checks become a habit. So, let's say that it's about 7,200 movies over about 30 years. That's 240 movies released per year, or a little less than five movies released per week. That seems a little high if we're considering only major movies, but if for every two major motion pictures, there are three indie films, that doesn't seem like an outlandish number.  

Most data you'll work with in real life won't come to you in a flat format. One great thing about the JSON format is it's really flexible, and it can handle raw, messy data. But if you try to read raw, messy JSON data directly into a DataFrame, the DataFrame will be a mess too. It's very difficult to find and fix corrupted data in messy DataFrames, and it's also difficult to consolidate columns without headaches. This leads us to the transformation stage.

### Transformation  
