# To do:
## ~~Done~~
- ~~Acquire dataset~~
- ~~Create repo~~
- ~~Create a way to pull the datasets from Google Drive since they won't fit in the repo~~
- ~~Cleaning:~~
    - ~~Rename columns~~
    - ~~Fill null values~~
    - ~~Put strings in lower case~~
    - ~~Convert codes from numbers to strings~~
    - ~~Convert date/time strings to datetime format~~
    - ~~Remove missing lat/lons~~
- ~~Reduce the following columns down to a small number of categories:~~
    - ~~crimeCodeDescription~~
    - ~~weaponDescription~~
    - ~~siteDescription~~
- ~~EDA/ETL~~
- ~~Feature engineering~~
- ~~Encode the data~~
- ~~Balance the dataset~~
- ~~Split data into training and testing sets~~
- ~~Scale the data~~
## On deck
- Train models
    - Repeat training for various models, various hyperparameters, find best score

## Later
- Create a presentation
- (maybe) Write functions to make it all a pipeline
- (maybe) Create a mock deployment/frontend

# Ideas:
## Feature Engineering:
- reportDelay (number of days between the crime and the report)
- reportTimely (1 if reported within 2 days, otherwise 0)
- crimeCount (number of crimeCodes used across crimeCode1, 2, 3, and 4)
- victimSuspectRelationship (1 if victim and suspect know each other, otherwise 0)
    - this information can be parsed from the moCodes
- reduce victimAge to categories:
    - unknown (0)
    - minor (1-17)
    - adult (18-59)
    - senior (60+)
- reduce victimDescent to other/unknown, black, hispanic, white, asian
    - or omit this column entirely, especially if it has minimal impact on the model
- This is a shapefile for Los Angeles districts containing income/poverty data:
    - https://data.lacounty.gov/datasets/5455a5c504064c38b5ac9638d8580d92/explore?location=34.852000%2C-118.026853%2C7.93
    - We can match every lat/lon with its corresponding district and populate the income/poverty data for the surrounding area of each crime
    - We can also count the amount of crimes in each district and use that as a feature to indicate if the crime happened in a high- or low-crime district
## Misc
- A crime reported in 2014 that still has the 'investigation ongoing' status is probably never going to lead to an arrest. A crime that was reported in August 2024 might also have 'investigation ongoing' status, but it probably hasn't had enough time to lead to an arrest either. Because of this, we might find that there are way fewer 'arrested' statuses for the most recent records, skewing the data a bit. We might consider setting a cutoff date, where crimes before that date had plenty of time to lead to an arrest (if they haven't already), and crimes after that date can be left out. Just a thought.
- 'Adult Arrest' and 'Juvenile Arrest' are self-explanatory, but I still need to figure out what 'Adult Other' and 'Juvenile Other' mean for the statuses. The documentation with this dataset does not explain it. It seems to imply that they were not arrested, but if that's the case, what was the actual outcome? What kind of closure can we infer?
