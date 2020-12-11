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

kaggle_metadata = pd.read_csv(f'{file_dir}movies_metadata.csv', low_memory=False)
ratings = pd.read_csv(f'{file_dir}ratings.csv')
```  
There are 7,311 data points, or records of movies, but is this a reasonable number of records? One way to check that 7,311 is reasonable is to look at the rate of movies being released that it implies. Rough mental math here is the key—we want these calculations to be quick enough that these checks become a habit. So, let's say that it's about 7,200 movies over about 30 years. That's 240 movies released per year, or a little less than five movies released per week. That seems a little high if we're considering only major movies, but if for every two major motion pictures, there are three indie films, that doesn't seem like an outlandish number.  

Most data you'll work with in real life won't come to you in a flat format. One great thing about the JSON format is it's really flexible, and it can handle raw, messy data. But if you try to read raw, messy JSON data directly into a DataFrame, the DataFrame will be a mess too. It's very difficult to find and fix corrupted data in messy DataFrames, and it's also difficult to consolidate columns without headaches. This leads us to the transformation stage.

### Transformation  
Every messy dataset presents its own unique challenges. There's no one right way to clean data, but we can still have a rough game plan to follow. Bad data comes in three states:

- **Beyond repair:** Data beyond repair could be data that has been overwritten or has suffered severe data corruption during storage or transfer (such as power loss during writing, voltage spikes, or hard-drive failures). The worst-case example would be having data with every value missing. All the information is lost and unrecoverable. For data beyond repair, all we can do is delete it and move on.  
- **Badly damaged:** Data that is badly damaged may have good data that we can recover, but it will take time and effort to repair the damaged data. This can be garbled data, with a lot of missing values, from inconsistent sources, or existing in multiple columns. Consider trade-offs to pick the best solution (even if the "best" solution isn't perfect, but rather the "best-available" solution). You may consider replacing the data by substituting data from another source, interpolating between existing data points, or extrapolating from existing data. Its also useful to standardize units of measure, and to consolidate data from multiple columns.  
- **Wrong form:** Finally, data in the wrong form should usually be fixed—that is, the data is good but can't be used in its current form. "Good" data in the wrong form can be data that is too granular or detailed, numeric data stored as strings, or data that needs to be split into multiple columns (e.g., address data). To remedy good data in the wrong form, try reshaping the data, converting data types, parsing text data into the correct format, or splitting columns.

Theres in iterative provess for cleaning data. First, we need to inspect our data and identify a problem. Once we have identified the problem, we need to make a plan and decide whether it is worth the time and effort to fix it, and lastly we execute the repair. Early iterations focus on making the data easier to investigate: deleting obviously bad data, removing superfluous columns (e.g., columns with only one value or missing an overwhelming amount of data), removing duplicate rows, consolidating columns, and reshaping the data if necessary. As the data becomes easier to investigate, iterations focus on fixing the most obvious problems first. As obvious problems are resolved, more subtle problems become noticeable. As the iterations shift toward solving more subtle problems, we might discover an earlier step needs to change as well as all the iterations that follow that step. It's frustrating when work has to be undone, but at least you now have a better understanding of your data. There are a number of questions to ask yourself when cleaning data frames.

- Does it have a consistent structure (like a CSV table) or is it unstructured (like a collection of email messages)?  
- How is each data point identified—is there an explicit, unique ID for each data point, or will one need to be built?  
- If a column doesn't have the right data type, is it a problem with the whole column? Or are just a handful of rows causing the issues?  
- Do rows have outliers due to spurious data? Or are they valid data points?  
- When values are missing, will they need to be removed, replaced, or interpolated?  

While transforming your data, you might bounce between steps in the iteration—for example, making a plan, then realizing you need to inspect more; executing a plan, then realizing a step was missed and you need to quickly rework the plan. We offer these steps as a descriptive, not prescriptive, approach. Cleaning up messy data is a messy process. The best practice is to document every step of your thought process and actions in detail.

### Load  
After extracting and transforming the data, its time to load the new dataframe into a SQL database. The `to_sql()` method in Pandas allows a programmer to save the dataframe to an SQL table. First you must create a Database engine in your python program. The following code serves as a template.  
```
from sqlalchemy import create_engine

from config import db_password # Imports your database password from a config.py file
db_string = f"postgres://postgres:{db_password}@127.0.0.1:5432/{DATABASE_NAME}" # Connects to local server

movies_df.to_sql(name='movies', con=engine) # Import the Movie Data

# create a variable for the number of rows imported
rows_imported = 0
# get the start_time from time.time()
start_time = time.time()
for data in pd.read_csv(f'{file_dir}ratings.csv', chunksize=1000000):
    print(f'importing rows {rows_imported} to {rows_imported + len(data)}...', end='')
    data.to_sql(name='ratings', con=engine, if_exists='append')
    rows_imported += len(data)

    # add elapsed time to final print out
    print(f'Done. {time.time() - start_time} total seconds elapsed')
```  
Once the cell finishes running, confirm the table imported correctly using pgAdmin. Verify the columns have the correct data type, inspect the first 100 rows, and check the row count. The code above only works if you already have a database by the specified name already created in your PGAdmin server.
