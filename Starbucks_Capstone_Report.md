# What Offer Starbucks Should Send to You

![cover](/images/cover.jpeg)

## Project Overview

Once every few days, Starbucks sends out an offer to users of the mobile app. An offer can be merely an advertisement for a drink or an actual offer such as a discount or BOGO (buy one get one free). Some users might not receive any offer during certain weeks.

This project used the data set which contains simulated data that mimics customer behavior on the Starbucks rewards mobile app. This data set is a simplified version of the real Starbucks app because the underlying simulator only has one product whereas Starbucks actually sells dozens of products.

The goal is to combine transaction, demographic and offer data to determine which demographic groups respond best to which offer type. 

## Problem Statement

Based on the goal described in the overview section, this project will focus on:

1. What are the main features that affect someone on responding the offers?

2. Build model to predict whether or not someone will respond to an offer.

To achieve the goal, my strategy are following steps:

1. Combine the transaction, customer demographic and offer information datasets, by linking the offer id and customer id from portfolio and profile data onto the transcript data. Then each row represents a customer-offer pair.

2. Create the target variable "offer_effective" for each customer-offer pair based on the existing information.

3. Build the classification model to predict the target variable, evaluate the model performance, and refine the model if necessary.

## Metrics

To measure the model performance, I'll use the accuracy which is the most common metric for classification and it measures hhe fraction of samples predicted correctly (i.e. whether an offer is effective).

However, if the target variable classes is unbalanced, i.e. uneven percentages of effective and ineffective offer, then considering recall and precision scores may provide some insights. Recall score will tell us the fraction of positives events that predicted correctly, while precision tells the fraction of predicted positives events that are actually positive. In that case, F-1 score may be easier to interpret, because F-1 is the harmonic mean of recall and precision, with a higher score as a better model.

## Data Dictionary

**Portfolio Dataset**   
portfolio.json - containing offer ids and meta data about each offer
- id (string) - offer id
- offer_type (string) - type of offer ie BOGO, discount, informational
- difficulty (int) - minimum required spend to complete an offer
- reward (int) - reward given for completing an offer
- duration (int) - time for offer to be open, in days
- channels (list of strings)

Every offer has a validity period before the offer expires. As an example, a BOGO offer might be valid for only 5 days. The informational offers have a validity period even though these ads are merely providing information about a product; for example, if an informational offer has 7 days of validity, you can assume the customer is feeling the influence of the offer for 7 days after receiving the advertisement.

**Profile Dataset**     
profile.json - demographic data for each customer
- age (int) - age of the customer
- became_member_on (int) - date when customer created an app account
- gender (str) - gender of the customer (note some entries contain 'O' for - - other rather than M or F)
- id (str) - customer id
- income (float) - customer's income

**Transcript Dataset**      
transcript.json - records for transactions, offers received, offers viewed, and offers completed
- event (str) - record description (ie transaction, offer received, offer viewed, etc.)
- person (str) - customer id
- time (int) - time in hours since start of test. The data begins at time t=0
- value - (dict of strings) - either an offer id or transaction amount depending on the record

The transactional data showing user purchases made on the app including the timestamp of purchase and the amount of money spent on a purchase. This transactional data also has a record for each offer that a user receives as well as a record for when a user actually views the offer. There are also records for when a user completes an offer.


## Data Exploration

Before we start doing the analysis and solving the problems, first we need to have a better understanding of the data. To do that,  we'll explore the input datasets relevant to the problem, such as checking missing values, any features need modification, or the distribution of major variables. Sometimes I'll build data visualizations to further convey the information associated with the data exploration journey.

**Portfolio Dataset**  
![portfolio overview](/images/image_1.png)  

The protfolio data is pretty simple that only includes 10 individual offers in three types (bogo / discount / informational). The cleaning step I did at this stage is apply one hot encoding to the categorical column "channels".     

![cleaned portfolio](/images/image_2.png)       

**Profile Dataset**     
![profile overview](/images/image_3.png)    

From the first 5 rows of the profile dataset, we noticed that there are missing values in "gender" and "income" columns, as well as extreme age value (118). To further explore, we checked the missing value counts and plotted the histogram for variable "age".

![profile missing value](/images/image_4.png)       

![profile age](/images/image_5.png)     

From the above information, we noticed that only "gender" and "income" columns have missing values and the counts are the same (2175). The "age" column has the outlier value 118 and the count of that value looks the same as the count of missing values in "gender" and "income". This coincidence makes me think that those rows with missing value all have age as 118. To confirm, let's drop those rows with age 118 and check if there're still missing values in "gender" and "income"column.

![profile check](/images/image_6.png)       

From the result, we should be comfortable with dropping all rows with age 118 and doing that will also solve the missing value issue. Besides that, we'll apply label encoding for "gender" column, and calculate the number of days since the customer became the member.

![cleaned profile](/images/image_7.png)     

Finally, let's take a quick look at the data distributions.

![gender distribution](/images/image_8.png)     

![income distribuion](/images/image_9.png)      

![memeber days distribution](/images/image_10.png)      

Some quick observations on the customer profile:
- There are more male members than female
- The most common age range among all members is 50-60
- Most members have a income in range 50000 - 80000
- Most customers have the membership about 4-5 years

**Transcript Dataset**      
![transcript overview](/images/image_11.png)        

There's no missing value found in the transcript dataset.       
![transcript missing value](/images/image_12.png)

From the above overview, we noticed that the "value" column seems including the offer id, so let's take a look at all possible events and corresponding values.

![transcript events](/images/image_13.png)       
![transcript event values](/images/image_14.png)        

Based on the dataset description and exploration, we can extract "offer id" or "amount" from "value" column depending on the record event. Notice that for event "offer completed", the value column is "offer_id" instead of "offer id" for the others.

![cleaned transcript](/images/image_15.png)


## Data Preprocessing

After the data exploration step, the input datasets have been cleaned and we have a better understanding of the data. In order to do the further analysis and model implementation, we need to further preprocess the three datasats. 

First step is to combine the transaction, customer demographic and offer information datasets, by linking the offer id and customer id from portfolio and profile data onto the transcript data.

![merged data](/images/image_16.png)        

Let's take a quick view on the event distribution by offer type after combining the datasets.

![event distribution](/images/image_17.png)         

For informational offers, there is no 'offer complated' event. And given that all "transaction" events don't include the offer id, so it's relatively hard to figure out if an informational offer is completed by the information we have.

However, we can use the time sequence of the events to figure out if a BOGO or discount offer influenced a customer's purchase. More specifically, we'll define the offer effectiveness as following:

- Customer viewed and completed the offer - effective offer:        
    offer received -> offer viewed -> offer completed

- Customer did not complete the offer after viewed the offer -  ineffective offer:      
    offer received -> offer viewed

If the customer viewed the offer after receiving, and then completed the offer, it's highly possible that the customer was influenced by the offer, then we can say that the offer is effective. If the customer viewed the offer but did not complete the offer, it means the offer is not effective to make the customer purchase the product.

With the above definition, we'll focus on BOGO and discount offers for now, and reshape the data to define if the offer is effective or not.

![subset data](/images/image_18.png)

After we subset the dataset to BOGO and discount offers only, we'll need to transpose the dataset to a wide version where each customer-offer pair only has one row. Then we can use the time of each event to define the offer's effectiveness. Before transposing, we found that there are some duplicated cases where the same offer sent to the same customer multiple times. In that case, we used the earliest time of each event when doing the tranpose. 

![wide data](/images/image_19.png)

Finally, separate the data into 2 dataframes for BOGO & discount offer.

![separate](/images/image_20.png)

## Model Implementation & Evaluation

Now, we have the datasets and ready for the model implementation. Our goal is to predict whether or not someone will respond to an offer. In order to do that, we are going to use classification model to predict the target variable "offer_effective". And we'll build the model for BOGO and discount offer separately.

### **Preparation**
Before the model implementation, let's do some checks on our data. First, take a quick look at the correlation between the features.

![correlation](/images/image_21.png)

The above correlation map is for BOGO offer dataset. From the result, we could tell that "reward" and "difficulty" are perfect correlated for BOGO offer, which absolutely makes sense because that's why it calls buy 1 get 1 free. So when we implement the model, we'll combine those two as a single 0.5 discount rate. Also we noticed that there's no result for "email" and "mobile". By checking the offer dataset, we noticed that Starbucks always send the BOGO offers via email and mobile, so we can drop both features from BOGO model implementation.

Similarly, we'll drop "web" and "email" columns for Discount offer, and combine "reward" and "difficulty" into a single discount rate feature.

Next, let's check on the balance of the target variable.

![target balance](/images/image_22.png)

For Discount offer, the target classes are a bit uneven. In that case, I'll use Random Forest Classifier model to deal with the imbalance cases.

In data exploration, we already checked the distribution of some features, like gender, age, income and member_days. From the histograms, the data doesn't have the imbalance issue for the features.

### **Implementation**

Now we're ready for the model implementation. For both offer types, I'll use the Random Forest Classifier model. 

Because our dataset includes both scale and categorical features (like gender), we'll just standardize selected scale features when prepare the training and test data.

![prep model](/images/image_23.png)

Here is the model performance for the first implementation.

![bogo1](/images/image_24.png)          
![discount1](/images/image_25.png)

For the first try, I did not change any parameters in the models. We can see that the test accuracy scores for both offer models are not high, especially for BOGO (69.97%). The training accuracy scores are too high that could be overfitting issue. From the F1 score, we can see that the model performs better on class 1 (effective offer), especially for discount offer type, which could because of the imbalance distribution of our target variable.

Next, we used GridSearch to tune the model by finding the optimal parameters. Also, we checked the feature importance to see if removing features can improve our model performance.

### **Refinement**

First, using GridSearch to find the optimal parameters.

![grid search](/images/image_26.png)

BOGO and discount offer models have the same optimal parameter setting from GridSearch.
![bogo grid](/images/image_27.png)
![disc grid](/images/image_28.png)

Here is the model performance for the second implementation after changing parameters.

![bogo2](/images/image_29.png)
![discount2](/images/image_30.png)

For BOGO offer, updating the parameters improves the model performance by increasing the test accuracy from 69.97% to 72.43%, and the training accuracy score is about the same as test (no overfitting issue). However, the improvement is limited and both accuracy and F1 scores are still below 80%. Similiarly, using the parameters from GridSearch result improves the performance of discount offer model but not significantly.

Next, we checked the feature importance for both offer type models.

![feature importance](/images/image_31.png)

From the feature importance results, we can see that the most important feature for both offer types is member_days. For BOGO offer, income, age and gender are less important but still has some proportion, compared to the other features. So we removed some low important features from BOGO and discount offer data, and here is the model performance for the third implementation.

![bogo3](/images/image_32.png)
![discount3](/images/image_33.png)

From the above results, we can see that dropping some low importance features still only improved our model for a little bit but not significantly, for both offer types.

## Conclusion

### **Reflection**

At the beginning of this project, I brought up two questions to focus on:

1. what are the main features that affect someone on responding the offers?
2. Build model to predict whether or not someone will respond to an offer.

For question 1, based on the feature importance results for both BOGO and discount offer types, the top 3 main features are membership tenure, income, and age. The membership tenure is much more important than the other 2 features.

For question 2, I used seperate models to predict if the customer will respond to an BOGO or discount offer, in another word, if the offer is effective on the customer. I used Random Forest Classifier model for both offer types, and got accuracy score 72.66% for BOGO and 76.01% for discount. Although I was expecting to see the score over 80%, the current accuracy would be acceptable to send offers from business aspect.

### **Challenge**

The biggest challenge for this project is the structure of the transcript data.

In order to build the classification model and answer the two questions, I need to define if an offer is effective to an customer, using the transcript data. This is the most difficult and challenging part. Because all transaction events do not have offer id linked, so we cannot 100% sure which offer the customer applied during the transaction. So I decided to use the time sequence of the other events to identify if an offer is effective or not, because other events have the offer id linked.

However, during the data exploration, we noticed that there is no "offer completed" event for informational offers. In that case, I cannot tell if the customer was affected by the offer or not, because we cannot link the transaction. So I decided to only focus on BOGO and discount offers for this project.

We classified two groups for effective/ineffective offer as shown below. By doing that, we were actually dropping some pairs that potentially could be calssified into these two groups.

1. offer received -> offer viewed -> offer completed (effective)
2. offer received -> offer viewed (ineffective)

Another challenge is preparing data for model implementation. To build the model, we need to encoding some categorical variables, like channels and gender. Also, that means our training data includes both scale and categorical features, so we need to treat them differently when doing the feature standardization.

### **Improvement**

Because of the time constraint, I didn't get the model accuracy score over 80%. There are some aspects of the implementation could be improved in the future.

- The method to identify the offer effectiveness could be improved. Based on current information, we may consider to bring more cases into the ineffective group, like completed offer before viewing the offer or even did not view the offer. Also, we can study this case more deeply because in this case, it could indicate that this customer will purchase regardless of the offer, so in business perspective, not send an offer to them may be a better choice. Another idea is to further restrict the effective offer group by checking the time between "offer completed" and "offer viewed". For example, if a customer completed the offer after a week of viewing the offer, then this purchase may not be affected by the offer.

- More models could be tried and compared the performance with the current one. Also more model tuning experiments could be tried. Due to the time reason, I didn't put many parameter choices when doing the GridSearch, which could probably find better parameters if I gave more inputs. Also more feature engineering steps could be helpful to check if we can get any other new features that potentially could improve the model.