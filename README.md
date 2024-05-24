# Top 'n' Similarity Recommendation

## Objective
The primary objective of this project is to develop a similarity comparison algorithm capable of listing similar Type#1s and Type#2s for each Type#1/Type#2. The project emphasizes performance and production readiness to accommodate the substantial amount of data to be processed multiple times a day.

## Introduction
The project revolves around a vast database housing over 800k records of web-scraped Type#1s and Type#2s. Each Type#1/Type#2 entry contains a plethora of information, including textual details like Type#1 text/title, categorical attributes such as Type#1 type or buyer, and continuous variables like price or execution deadline days. Additionally, Type#1s and Type#2s are categorized based on European procurement Category#1 codes and geographical location.

## Goal
The performance goal is twofold:
- Process 20 Type#1s in under 10 minutes.
- Process 30 Type#2s in under 10 minutes.
- Process all historical data (800k+ Type#1s, 80k+ Type#2s) within a maximum of 48 hours.

The algorithm is expected to preferably run on a non-GPU enabled computer (not mandatory).

## Challenge
The challenge was to develop a similarity comparison algorithm that, for each Type#1/Type#2, can list similar Type#1s and Type#2s. The project should be focused on performance and production readiness, considering the current and future amount of data that needs to be processed multiple times a day.

## Where Did I Start?
With my two best friends - DB and SQL. I spent hours understanding the data and making it mine...

## Data Collection
Data gathering involved meticulous analysis of various tables in the relational database management system (RDBMS). This analysis included understanding the database design, column specifics, and their interconnections. Optimized SQL queries were employed to fetch the data modularly, ensuring both modularity and minimal runtime due to the sheer volume of production data.

### Data Fetched
**Type#1s Master Table:**
- Categorical variables: Type#1 procedure type, Type#1 types, Category#1s
- Numerical variables: Execution deadline days, effective price
- Text variables: Object brief description, country, county, buyer

**Type#2s Master Table:**
- Categorical variables: Type#2 types, Category#1s
- Text variables: Object brief description, country, county, buyer
- Numerical variable: Effective price

## Data Cleaning
Data cleaning was a crucial step involving:
- Removing duplicate values resulting from merged dataframes.
- Addressing missing values, largely handled during SQL data fetching by replacing null values with "No Info".
- Reformatting data types to ensure consistency and readiness for further processing.

## Data Transformations
Extensive data transformations were applied while fetching:
1. **Category#1s:** 
   - The Category#1 establishes a single classification system for public procurement aimed at standardizing the references used by contracting authorities and entities to describe the subject of procurement contracts. Some contracts/ads may be classified under more than one Category#1 conjoined into a single row. These Category#1s were separated such that there were multiple lines per contract/ad when fetched.

2. **Effective Price:** 
   - Although a numerical variable, it made more sense to transform this data into effective price brackets for similarity matching. The transformations were as follows:
     - Less than 0: 0
     - 0-100000: 1
     - 100001-500000: 2
     - 500001-1000000: 3
     - 1000001-3000000: 4
     - 3000001-9000000: 5
     - 9000000-20000000: 6
     - 20000001-50000000: 7
     - 50000001-100000000: 8
     - 100000001-200000000: 9
     - 200000001-400000000: 10
     - 400000001-1000000000: 11
     - Greater than 1000000000: 12

3. **Execution Deadline Days:** 
   - Similar to effective price, the execution deadline days were also transformed into brackets:
     - Less than 0: 0
     - 0-15: 1
     - 15-30: 2
     - 31-45: 3
     - 46-60: 4
     - 61-90: 5
     - 91-120: 6
     - 121-150: 7
     - 151-180: 8
     - 181-270: 9
     - 271-365: 10
     - 366-730: 11
     - 731-1095: 12
     - 1096-1460: 13
     - 1461-1825: 14
     - Greater than 1825: 15

## Sampling
Due to resource constraints, sampling was employed to manage the volume of data effectively.

## Exploratory Data Analysis (EDA)
EDA encompassed a thorough examination of the dataset, including:
- Type#1s and Type#2s counts per procedure type.
- Type#2s/Buyers counts per price bracket.
- Type#1s counts per price bracket and execution deadline days bracket.
- Distribution of Type#1s and Type#2s across various categories.

## Feature Engineering
### Feature Construction
- **Type#1s/Type#2s Text:** Converted to text embeddings using the Sentence Transformers multi-lingual algorithm.
- **Category#1 Text:** Lemmatized and converted to embeddings using the same algorithm.
- **Type#1 Type:** Categorical feature encoded using the Dummy technique.
- **Procedure Type:** Another categorical feature encoded using the Dummy technique.

### Feature Selection
Feature selection was a pivotal aspect of the project, aiming to identify the most relevant attributes to optimize performance and accuracy.

## Feature Selection Strategies
Feature selection involved a systematic exploration of various feature combinations and weighting strategies to determine the optimal set of attributes for similarity comparison.

### Trials and Results
**First Trial:**
- **Description Text:** Used "all-MiniLM-L6-v2".
- **Description:** The "all-MiniLM-L6-v2" is a variant of the MiniLM model, which is a compact version of BERT (Bidirectional Encoder Representations from Transformers). It is specifically trained for language understanding tasks and is capable of capturing the contextual information of textual data. The "all" variant indicates that it has been trained on a large corpus of text data encompassing various domains and languages, making it suitable for multilingual applications.
- **Result:** Poor match due to missing Portuguese support.

**Second Trial:**
- **Description Text:** Used "distiluse-base-multilingual-cased-v1".
- **Result:** Not-so-good match due to lemmatized text containing numerals and single characters.

**Third Trial:**
- **Clean Lemmatized Description Text + Category#1 ID:** Used "distiluse-base-multilingual-cased-v1".
- **Result:** Better for some, worse for others. Needed more info to segregate and match.

**Fourth Trial:**
- **Clean Lemmatized Description Text + All Extracted Features:** Used "distiluse-base-multilingual-cased-v1".
- **Result:** Better for some, worse for others. Description was given equal importance as other features.

**Fifth Trial:**
- **Weighted Clean Lemmatized Description Text + Category#1 ID + Price Bracket:** Used "distiluse-base-multilingual-cased-v1".
- **Result:** Better for some, margin to improve. Category#1 ID being ordinal was putting two completely non-related records closer.

**Final Trial:**
- **Weighted Clean Lemmatized Description Text + Category#1 Text Vector + Price Bracket:** Used "distiluse-base-multilingual-cased-v1".
- **Result:** Acceptable results.

## Modeling
The project employed Approximate Nearest Neighbor (ANN) search models, specifically ANNOY by Spotify, to find top matches for a given query. ANNOY efficiently preprocesses data into an index, speeding up the search process.

### ANNOY
- Creates a forest (many trees).
- Each tree is constructed by picking two points at random and splitting the space into two by their hyperplane, recursively splitting into subspaces until the points associated with a node are small enough.
- The forest is traversed to obtain a set of candidate points from which the closest to the query point is returned.

## Results
Results were manually analyzed based on reference test data provided. The project aimed to ensure that the top matches for Type#1s/Type#2s were acceptable and intuitive.

## Conclusion
The similarity comparison algorithm successfully identifies top matches for any given Type#1/Type#2. The project achieved its performance and production readiness goals, processing a significant amount of data efficiently. Further enhancements could include additional features or optimizations for even better results.

**Note:** Due to legal constraints, the complete details of the project cannot be disclosed, and certain proprietary information may be omitted from this report.
