# Los Angeles Crime Arrest Predictions

## Presentation Slides
https://docs.google.com/presentation/d/1lA3cuCAJBtnveRd9LGkl-GkJPqvFWRPSqvvNhsI4DLM/edit#slide=id.g2874446bc5d_0_618

## Data
- Los Angeles Crime 2020 - Present: https://data.lacity.org/Public-Safety/Crime-Data-from-2020-to-Present/2nrs-mtv8/about_data
- Los Angeles Crime 2010 - 2020: https://data.lacity.org/Public-Safety/Crime-Data-from-2010-to-2019/63jg-8b9z/about_data
- Los Angeles District and Income Data: https://data.lacounty.gov/datasets/5455a5c504064c38b5ac9638d8580d92/explore

## Python Packages/Libraries
- numpy
- pandas
- geopandas
- scikit-learn
- matplotlib
- PyPDF2

`pip install numpy pandas geopandas scikit-learn matplotlib PyPDF2`

## Objective
Given a large set of historical data about crimes recorded in Los Angeles, create a machine learning model to best predict whether a crime will be resolved (arrest or other) or remain under investigation.

## About the data
Los Angeles has a large collection of publicly available datasets on https://data.lacity.org/, including two crime datasets collectively spanning 14 years, with a little over 3 million records. The data, in the form of a CSV, includes the following information:
- Report ID
- Date the crime occurred
- Time the crime occurred
- Date the report was filed
- LAPD district
- Crime codes, crime descriptions
- Modus operandi codes
- Victim age, sex, descent
- Premise codes, premise descriptions
- Weapon codes, weapon descriptions
- Location (address, cross streets, latitude, longitude)
- Status (e.g. investigation continuing, adult arrested, juvenile arrested etc.)

Using the crime data's latitude/longitude data, we brought in a separate geospatial dataset detailing all of LA's district boundaries as polygons, as well as the number of households and median household income for those districts.

## Phase 1: Data Cleaning, EDA, ETL  
- Crimes, weapons, premises
    - The raw dataset included 143 unique kinds of crime, 80 unique kinds of weapons, and 321 unique kinds of premises
    - These were reduced to a few categories:
        - crimes: violent, property damage, theft, etc.
        - weapons: bodily, firearms, objects, etc.
        - premises: public, residential, business, etc.
    - A prominent method for sorting these into categories was to list out keywords related to each category, and running a script that puts them in that category if those words are found in their descriptions (e.g. ['stolen', 'theft', 'robbery', 'burglary'] -> theft category)

- Columns were renamed for better readability
- Null values were filled
- Null values in the target variable (status) were removed
- All string column were converted to lowercase for consistency
- Rows with zeroes in latitude and longitude were removed

- 19 unique ethnicities were reduced to 8
- Ages were reduced to minor, adult, senior, and unknown (the dataset used 0 for unknown/not applicable)

## Phase 2: Feature Engineering
- Report timeliness:
    - The number of days elapsed between the crime occuring and the crime being reported
- Victim/suspect relationship:
    - Many crimes included modus operandi codes, some of which corresponded to notes on the case indicating the victim and suspect know each other (e.g. family, friends, roommates, coworkers, etc.)
- Income
    - With most crimes falling into some city district's polygon, each crime could now include the income level of the surrounding area
- Crime total / density / arrest ratio
    - Using the same district polygons, each crime was matched with:
        - the total amount of crimes to have happened in that district since 2010
        - the total amount of crimes to have happened in that district 6 months prior
        - the ratio of crimes per household in that district since 2010
        - the ratio of crimes per household in that district 6 months prior
        - the percentage of crimes resulting in arrests in that area since 2010

## Phase 3: Data Preparation
### Encoding
Once we were satisfied with data cleaning and feature engineering efforts, we had to encode any remaining non-numeric columns.
- One-hot encoded: victim age, victim descent, victim sex
- Ordinal encoded: district income level, crime density level
### Balancing
As some already know, most crimes go unsolved. Unsurprisingly, the raw dataset comprises about 77.5% crimes that are in an 'investigation' status, and only about 22.5% crimes that had an arrest. In other words, unresolved crimes outnumber resolved ones about 3.5 to 1.
We decided it would be best if whatever model we train works with a balanced dataset, so we took equal samples from both classes in the dataset and combined them prior to training.
### Splitting
We used scikit-learn's train-test-split function, and used the 'stratify' argument to ensure that both training and testing sets include the same ratio of 1s and 0s as are present in the data (1:1).
### Scaling
We fit StandardScaler to our training features, and used that to transform the training and testing features

## Phase 4: Modeling
### Feature Selection
We used the feature importance attribute in RandomForestClassifier to get an idea of what features were the best candidates for training a model. From there, we selected the top 15 features, before importance values dropped off significantly. What stood out is that victim data (age, sex, descent) scored very low on importance, as did most weapon and premise categories. The top scoring features were relationship, latitude, longitude, crime density, thefts, arrest ratio, violent crimes, physical violence, armed crimes, and crimes in private homes.
### Training
We gathered a handful of machine learning models and used GridSearchCV to find an optimal set of hyperparameters for each model. We used accuracy as our primary metric, because our goal is to have as many correct predictions as possible overall, with no special importance placed on either class (1 or 0). Since we balanced the dataset prior to training, we mitigated the risks of getting a skewed model.  

Models tested:
- RandomForestClassifier
    - Validation accuracy: 0.7529
    - Entire dataset: 0.7624
- SVC (Support Vector Classification)
    - Validation accuracy: 0.7507
    - Entire dataset: 0.7629
- XGBoost
    - Validation accuracy: 0.7531
    - Entire dataset: 0.7628
- AdaBoostClassifier
    - Validation accuracy: 0.744
    - Entire dataset: 0.7614
- ExtraTreesClassifier 
    - Validation accuracy: 0.7516
    - Entire dataset: 0.7659  

More detailed scores (F1, precision, recall, etc.) are found in modeling.ipynb

### Ensembling
One method to increase model accuracy is to have multiple models predict on the same data, and use the average of their scores to see what they 'agree' on. This tends to work better when the models involved are all fundamentally different, which isn't really the case here. Nevertheless, we experimented with this method, and ensembled our 5 models to see the accuracy of their combined scores. In our case, this method of ensembling did not increase accuracy.

# Conclusions

Feature engineering and ETL was a major factor in improving model performance. The original crime dataset only had age, latitude, and longitude as features that were already numeric. The rest of it had to modified in some way (dates, times, categories, codes). Extracting information hidden in the dataset in the form of codes was necessary, as was getting creative and creating new features from existing features, and from outside datasets.

After testing thousands of hyperparameters, model accuracy hovers around 75-76% across multiple different models. This tells us that we’ve likely reached the limit of what hyperparameter tuning can accomplish with the data as far as accuracy, and that surpassing 76% would probably require more/better features, and some different approaches. Given the human-driven nature of criminology, the true limit of predictability might not be that much higher than 76%. Even with all the public data in the world, there are likely dozens of unseen, unmeasurable factors contributing to the outcome of a criminal investigation.
