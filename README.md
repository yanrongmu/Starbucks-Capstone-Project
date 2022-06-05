# Udacity Data Scientist Nanodegree Capstone Project

## Table of Contents
1. [Installation](#installation)
2. [Project Motivation](#project-motivation)
3. [File Descriptions](#files)
4. [Results](#results)
5. [Licensing, Authors, Acknowledgements](#license)

### Installation <a name="installation"></a>

The code requires the following basic libraries:
1. pandas
2. numpy
3. math
4. json
5. matplotlib
6. seaborn
7. datetime
8. time
9. sklearn.(compose / preprocessing / model_selection / ensemble / tree / metrics)

The code should run with no issues using Python versions 3.*.
 
### Project Motivation <a name="project-motivation"></a>

For this project, I was interestested in using Starbucks transaction, demographic and offer data to answer 2 questions:

1. what are the main features that affect someone on responding the offers
2. build model to predict whether or not someone will respond to an offer

### File Descriptions <a name="files"></a>

The data folder in this repo contains 3 data files used in this project:
1. portfolio.json - containing offer ids and meta data about each offer (duration, type, etc.)
2. profile.json - demographic data for each customer
3. transcript.json - records for transactions, offers received, offers viewed, and offers completed

There is 1 notebook available here to show the data understanding, preparation, analysis and evaluation related to the above questions. There are also necessary markdown cells and comments to help understanding the whole process and individual steps.

There is a preject report to summarize the results of the analysis.

## Results<a name="results"></a>

The main findings can be found in conclusion section in the notebook. This repo also includes a complete project report "Starbucks_Capstone_Report.md". Below are the quick summaries:

For question 1, based on the feature importance results for both BOGO and discount offer types, the top 3 main features are membership tenure, income, and age. The membership tenure is much more important than the other 2 features.

For question 2, I used seperate models to predict if the customer will respond to an BOGO or discount offer, in another word, if the offer is effective on the customer. I used Random Forest Classifier model for both offer types, and got accuracy score 72.66% for BOGO and 76.01% for discount. Although I was expecting to see the score over 80%, the current accuracy would be acceptable to send offers from business aspect.

### Licensing, Authors, Acknowledgements, etc.<a name="license"></a>

Data was provided by Udacity.
