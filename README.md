# Data Science Applied to Pay-Per-Bid Auctions
## Master Thesis - VI Master in Data Science - KSchool
#### Sergio Valmorisco Sierra
#### 4th June, 2018
LinkedIn: www.linkedin.com/in/sergio-valmorisco-sierra
E-mail: s.valmorisco@gmail.com

# 1. Introduction
Pay-per-bid auctions are a special kind of auctions in which bidders incur a cost (bid fee) for placing each bid in addition to (or sometimes in lieu of) the winner's final purchase cost. Also, each bid that is placed increases the final purchased cost by a fixed amount. Pay-per-bid auctions begin at a reserved price (generally 0), and have an associated countdown clock. When a player places a bid, some additional time is added to the clock. When the clock expires, the last bidder has the chance to purchase the item at the final auction price.

*Example: users A and B have participated in a pay-per-bid auction. User A has placed 30 bids, user B has placed 60 bids, the bid fee is 1$ and the bid increment to the selling price is 0.10$. User B has won the auction because the countdown clock expired after user B placed the last bid.*

In this example, user A has paid 30$ (number of bids placed by user A multiplied by the bid fee) but has not won the auction, so user A does not get to purchase the item. User B has paid 60$ (number of bids placed by user B multiplied by the bid fee) and has won the auction, so user B gets to purchase the item by paying the final selling price of 9$ (number of total bids placed multiplied by the bid increment). In total, user B has spent 69$. The total money obtained by the auction site is 99$ (sum of the expenses of users A and B).

Swoopo was one of the most famous pay-per-bid auction commercial websites up to 2011. In order to participate in auctions, registered users had to first buy bids (called credits). Each bid extended the countdown clock by 10–20 seconds, so auctions could theoretically continue on indefinitely.

Appart from placing manual bids, Swoopo also provided the possibility of placing bids with Bid Butlers. These were automatic bidding agents provided by the Swoopo interface that bidded according to user-defined instructions. Users were able to define starting and ending price limits for which automated bids shall be placed, as well as the top number of bids that the Bid Butler was allowed to place. Bid Butlers placed automated bids when the auction timer dropped below 10 seconds.

Swoopo also offered special types of auctions:
  - Click-only auctions: also called "NailBiter" auctions. These auctions did not allow the use of Bid Butlers.
  - Beginner auctions: auctions restricted to users who had not previously won any auction in the site.
  - Fixed-price auctions: auctions in which the selling price was fixed from the beginning.
  - End-price auctions: also called 100%-off auction. In these auctions the final selling price of the product was zero, and the revenue for Swoopo came exclusively from the bids that were placed.

The main topic of this Master Thesis is the development of a prediction model that allows predicting the final selling price of an auction in Swoopo (or other auction sites that work in a similar way) before it starts. Having an estimation of the final selling price of an auction could be very helpful for auction site owners.

Auctions, as compared to other sales, are associated to a big uncertainty. The final selling price of an auction depends on a lot of factors and can vary greatly between two different auctions, even when the item that is auctioned is the same and the two auctions take place at a similar point of time.

If relatively accurate estimations of the selling prices of the items can be made, this can help auction site owners to take important business decisions. Consequently, this is a relevant problem.

The dataset used in this Master Thesis contains information about Swoopo's auctions and can be found in the following link: http://people.bu.edu/zg/swoopo.html

The dataset consists on two files:
  - **outcomes.tsv**: this file is based on information published directly by Swoopo, which contains limited information about an auction: the description of the product, its retail price and selling price, the winnner of the auction, the number of placed bids, etc.
  - **traces.tsv**: this file is based on traces of live auctions that the authors of the dataset recorded using their own recording infrastructure. It contains the record of the bids that were placed for several auctions, including the user who placed each bid, the bidding time, etc.

[Reference] *Byers, J. W., Mitzenmacher, M., & Zervas, G. (2010, June). Information asymmetries in pay-per-bid auctions. In Proceedings of the 11th ACM conference on Electronic commerce (pp. 1-12). ACM.*

# 2. Dataset Description
As it has been previously mentioned, the dataset used in this Master Thesis contains information about Swoopo's auctions and can be found in the following link: http://people.bu.edu/zg/swoopo.html

The dataset consists on two files. The detailed description of these two files can be found in Sections 2.1 and 2.2. Both are tab-separated values (TSV) files.

During the different steps of the development, other files are generated and required. These files are explained in more detail in Section 3.1.

## 2.1. outcomes.tsv
This file is based on information published directly by Swoopo and contains information about 121419 auctions that took place between 2008-08-20 and 2009-12-12. The file contains the following columns:

| Column        | Description          
| ------------- |------------- | 
| auction_id      | unique numerical id for the auction | 
| product_id     | unique product id      |
| item | text string describing the product |
| desc | more information about the product |
| retail | the stated retail value of the item, in dollars |
| price | the price the auction reached, in dollars |
| finalprice | the price charged to the winner in dollars. For certain types of auctions, the value in this column may differ from the value specified in column "price". For example, this may happen in fixed-price auctions or 100%-off auctions |
| bidincrement | the price increment of a bid, in cents |
| bidfee | the cost incurred to make a bid, in cents |
| winner | the winner's username |
| placedbids | the number of paid bids placed by the winner |
| freebids | the number of free bids placed by the winner. Free bids are offered by Swoopo in certain deal packages, and users placing free bids do not incur in any bid fee |
| endtime_str | the auction's end time |
| flg_click_only | a binary flag indicating a "NailBiter" auction (Swoopo auctions which do not permit the use of automated bids by a Bid Butler) |
| flg_beginnerauction | a binary flag indicating a beginner auction (auctions restricted to users who haven't previously won any auction in the site) |
| flg_fixedprice | a binary flag indicating a fixed-price auction (an auction in which the selling price is fixed from the beginning) |
| flg_endprice | a binary flag indicating a 100%-off auction (auctions in which the final selling price of the product is zero, and the revenue for Swoopo comes exclusively from the bids that are placed) |

## 2.2. traces.tsv
This file is based on traces of live auctions that the authors of the dataset recorded using their own recording infrastructure. It comprises 7,352 auctions conducted between October 1, 2009 and December 12, 2009. The traces include detailed bidding information for each auction.

The authors probed Swoopo at semi-regular time-intervals (more frequently near the end of the auction when bids are placed in rapid succession, less frequently near the beginning of the auction when bidding is sparse). When the countdown clock had less than 2 minutes left on it, they probed at the rate Swoopo suggested but at least once every second (every probe response was associated with a Swoopo defined update-interval that Swoopo uses to instruct the user’s browser when to update next).

The result of each probe was a list of up to ten tuples of the form (username,bidnumber), indicating the players that placed a bid since the previous probe and the order in which they did so. In the cases the list included more that one tuple, the authors ascribed the same timestamp to all of those bids.

A limitation from the authors' probing methodolody arises when more then ten players bid between successive probes. In particular, this happened when more than ten players were using Bid Butlers. In these cases, Swoopo responded with just the ten latest bids.

More details about the methodology used by the authors to create the dataset can be found at: *Byers, J. W., Mitzenmacher, M., & Zervas, G. (2010, June). Information asymmetries in pay-per-bid auctions. In Proceedings of the 11th ACM conference on Electronic commerce (pp. 1-12). ACM.*

The file contains the following columns:

| Column        | Description          
| ------------- |------------- | 
| auction_id | unique numerical id for the auction | 
| bid_time | the date and time of each bid | 
| bid_ct | the value of the countdown clock (seconds) at the time the bid was reported to the researchers that recorded the traces | 
| bid_number | the number of the bid placed in the auction in ascending order, starting with 1 | 
| bid_user | the username of the bidder | 
| bid_butler | 1 for Bid Butler bids, 0 otherwise | 
| bid_cp | the price of the item after the bid was placed | 
| bid_user_secs_added | the number of seconds added to the countdown clock as a result non-Bid Butler bidded in the bid group (*) | 
| bid_butler_secs_added | the number of seconds added to the countdown clock as a result Bid Butler bidded in the bid group (*) | 
| bid_infered | 0 for the final bid in a bid group, 1 otherwise (*) | 
| bid_group | bid group identifier (*). Bids that were reported as part of the same group will have the same group number | 
| bid_final | 1 for the winning bid (the last one), 0 otherwise | 

**(\*) bid groups**: Occassionaly, between successive probes, more than one bids would have occured and would be reported together. The authors refer to this as a bid group. All bids in a bid group have been ascribed the same timestamp

The other parameters of the auction can be looked up in outcomes.tsv using the auction_id field.

# 3. Methodology
The first step of the development process consists on the data acquisition. In this Master Thesis, the dataset that has been used is available to the public, and the authors of the dataset have already recorded traces of live auctions using their own recording infrastructure. Therefore, the data acquisition phase is minimal in this Master Thesis.

Nevertheless, as indicated in Section 2.1, the dataset contains a brief description about the product that is being auctioned. It could be interesting for the prediction model to have a product category to which the item being auctioned corresponds and not just a product description. This is because the final selling price of an item could greatly depend on its product category. The data acquisition process that has been followed to obtain the product category corresponding to each item is described in detail in Section 3.3.

After the data acquisition phase, a data cleaning process is necessary so that the results that are obtained in the end are of high quality. In Section 3.2, the process used to clean the data is explained. Afterwards, the data has to be analysed in order to obtain valuable insights. The analysis process is explained in Section 3.2.

After the data has been cleaned and analysed in Section 3.2, the product categories of the items have been obtained in Section 3.3. As an example, the product description for one of the items appearing in the dataset is "Sony Ericsson S500i Unlocked Mysterious Green". Although a human could identify by reading this description that the product is a mobile phone, a machine is not able to do this. Amazon.com assigns a product category to each one of the products that it sells. For example, for the product previously indicated, the corresponding Amazon product category is "Cell Phones & Accessories › Cell Phones › Unlocked Cell Phones". This category could be useful to complete the missing product category information for the products contained in the dataset, and web scraping methods can be used to extract these categories. This is explained in more detail in Section 3.3.

Once that these product categories have been obtained, it is necessary to find a way to introduce them as input information for the prediction model. A numerical representation of a product category semantic meaning can consist on a word embedding vector that encompasses the meaning of all of the words contained in the product category string. This is explained in more detail in Section 3.4.

Finally, in Section 3.5, different prediction models to predict the final selling price of an auction before it starts have been built, and their results have been analysed to idenfity the best one.

## 3.1. Requirements and steps to run the project
As mentioned in [Section 2](#2-dataset-description), the dataset is composed of two files. The size of the file "outcomes.tsv" is 17.9 MB and the size of "traces.tsv" is 128 MB. Therefore, cluster-computing frameworks, such as Apache Spark, are not needed to analyse these files since their size is relatively small. The Master Thesis has been entirely done with Python, except for the visualization dashboard that has been created with Tableau.

The packages that are necessary to run this project can be installed with the conda environment file **"environment.yml"** available in this GitHub repository.

The project has been divided into four IPython notebooks that **should be run in order**. Also, the cells contained in each of these IPython notebooks **should be run in order too**:
1. **"01_exploring_and_cleaning_the_dataset.ipynb"**: this file contains the initial exploration of the data, as well as the cleaning process that has been applied to the data. For details, read [Section 3.2](#32-exploring-and-cleaning-the-dataset).
2. **"02_obtaining_the_product_categories.ipynb"**: this file contains the code that has to be executed to extract the product categories of the items contained in the dataset. For details, read [Section 3.3](#33-obtaining-the-product-categories).
3. **"03_product_categories_to_word_embedding_vectors.ipynb"**: this file contains the code that calculates the word embedding vectors associated to the product categories. For details, read [Section 3.4](#34-transforming-the-product-categories-to-word-embedding-vectors)
4. **"04_prediction_model_auction_selling_price.ipynb"**: this file contains the different prediction models that have been developed to predict the final selling price of the auctions. For details, read [Section 3.5](#35-building-a-prediction-model-for-the-final-selling-price-of-the-auctions).

Also, as detailed in [Section 5](#5-front-end), a visualization dashboard has been created with Tableau. The dashboard is available in this GitHub project, and requires the clean version of the dataset to run. The IPython notebooks also need certain files as input and create certain outputs used by other IPython notebooks. In the following tables, the file dependencies are summarized:

| IPython notebook        | Inputs | Outputs |          
| ------------- |------------- | ------------- | 
| 01_exploring_and_cleaning_the_dataset.ipynb | F1, F2 | F3, F4 |
| 02_obtaining_the_product_categories.ipynb | F3 | F5 | 
| 03_product_categories_to_word_embedding_vectors.ipynb | F5, F6 | F7 |
| 04_prediction_model_auction_selling_price.ipynb | F3, F7 | - |

| Visualization dashboard        | Inputs | Outputs |  
| ------------- |------------- | ------------- | 
| Dashboard | F3, F4 | - |

| File        | Name | Description |          
| ------------- |------------- | ------------- | 
| F1 | "outcomes.tsv" | Swoopo dataset ([Section 2.1](#21-outcomestsv)). Publicly available: http://people.bu.edu/zg/swoopo.html |
| F2 | "traces.tsv" | Swoopo dataset ([Section 2.2](#22-tracestsv)). Publicly available: http://people.bu.edu/zg/swoopo.html |
| F3 | "outcomes_clean.tsv" | Clean version of F1. The file "01_exploring_and_cleaning_the_dataset.ipynb" has to be run to obtain this file.  |
| F4 | "traces_clean.tsv" | Clean version of F2. The file "01_exploring_and_cleaning_the_dataset.ipynb" has to be run to obtain this file. |
| F5 | "product_categories.xlsx" | File that contains the item product categories obtained from Amazon. Available in this GitHub repository. |
| F6 | "GoogleNews-vectors-negative300.bin.gz" | Google's pre-trained Word2Vec model (1.5GB). Publicly available: https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit |
| F7 | "productDescriptionToVector.xlsx" | File that contains the word embedding vectors for the product categories contained in F5. Available in this GitHub repository.  |





## 3.2. Exploring and cleaning the dataset
Sections 3.2.1 and 3.2.2 describe both the cleaning and the analysis process that has been followed for the two files contained in the dataset.

### 3.2.1. Exploring and cleaning "outcomes.tsv"

The file does not contain any null values. The columns "retail", "price" and "finalprice" are all expressed in dollars, which the columns "bidincrement" and "bidfee" are both expressed in cents. These last two columns have been converted to dollars. 

The price increment of the final price of the item after each bid depends on the auction. In the dataset, it ranges from 1 cent to 24 cents. The bid fee also depends on the auction. In the dataset, it ranges from 60 cents to 75 cents.

Free bids are offered by Swoopo in certain deal packages. Of all of the bids placed in the dataset by all winners, 414321 bids were free bids and 9098807 were paid bids. The ratio between free bids and paid bids is then 4.55%. Taking account into account this approximation, the total money that Swoopo obtains for an auction is the number of non-free bids placed by all participants (approximated as a 95.45% of the total number of bids placed) multiplied by the bid fee, plus the final selling price of the item paid by the winner.

In order to identify the different items sold in the auctions, the columns "product_id", "item" and "desc" can be used. An analysis of the unique values contained in these columns revealed that each one of them contains a different number of unique values. The "product_id" column contains different values for rows of the dataset in which the same item is sold. Therefore, the "product_id" value is not a good option to identify unique products. Also, in some cases, some items do not contain a description in the column "desc", and the "item" column is the name of an HTML page. In most of the cases in which there are different values in the column "desc" for the same value of the column "item", it is due to small grammar differences in the text contained in the column "desc". The dataset has been cleaned in order to fix all of these problems.

Sometimes, Swoopo obtaines a negative profit (i.e., Swoopo sells an item below its retail price). This is the case for approximately 47.50% of the auctions. Nevertheless, the average profit obtained by Swoopo over the retail price of an item is around 200$. Naturally, users are motivated to place bids because if they win the auction they will most likely buy the item for a price that is much lower than its retail price. In average, the benefit that the winner obtains over the retail price of the item is around 160$.

The most profitable auctions are, in descending order: fixed-price auctions, "normal" auctions, end-price auctions, click-only auctions and beginner auctions (where "normal" auctions are the ones that are not included in the other categories).

The majority of the items that are auctioned are electronics. A lot of the products that are auctioned contain the words "voucher" or "bids" in their product description. That is because certain items come along (or directly are in some cases) bids or cash that can be later used by the winnner in other Swoopo auctions.

The dataset contains auctions within 2008-08-20 and 2009-12-12. 

In general, November, December and January are the months in which more bids are placed. During the last two weeks of a year (2008/51 - 2008/52) plus the three first weeks of the next year (2009/01 - 2008/03) the profit obtained is higher than average, with its peak in the first week of the new year. Week 2009/14 (corresponding to the period from March 30, 2009 to April 5, 2009) is the week in which the highest profit is obtained with a very noticiable difference. The period between weeks 2009/29 - 2009/45 (July 13, 2009 - November 8, 2009) the profit obtained is lower than average. 

The profit ratio for special dates has also been analyzed. These include the federal holidays in the USA, Black Friday, the week before Valentine's Day, and Christmas' season (December).

Considering only the day of the week (Monday-Sunday), the highest profit ratio is obtained on Friday, while the lowest positive ratio is obtained on Tuesday. Nevertheless, the values are pretty similar throughout all days of the week.

### 3.2.2. Exploring and cleaning "traces.tsv"
This file is based on traces of live auctions that the authors of the dataset recorded using their own recording infrastructure. It comprises 7,352 auctions conducted between October 1, 2009 and December 12, 2009. The traces include detailed bidding information for each auction.

Generally, it can be observed that when an auction is about to end, a few users place bids consecutively until one of them decides to keep bidding while the others do not.

The average auction duration (calculated as the time that passed between the first bid and the last bid placed for that auction) is 7 hours and 10 minutes:

Some users changed their username during the time that the auction traces were recorded, while others used their phone to place bids, and their phone number was the username recorded in the traces. This caused a mismatch between the usernames appearing in this file "traces.csv" and the other file contained in the dataset "outcomes.csv", where the username of the winner of each auction is specified. Every case in which this happens has been identified and the usernames appearing in the traces have been modified to match the ones publicly shared by Swoopo.

The mean time difference between the bids placed by the winners is around 5 minutes, while the mean time difference between the bids placed by the losers is around 13 minutes.

The number of bids placed is significantly different for different values of the countdown clock. This is because, when a bid is placed and the countdown clock is about to end, that bid increments the value of the countdown clock a few seconds, so that users do not place bids only a few seconds before the auction finishes. Most of the bids make the countdown clock restart its value to either 10 seconds (for bids placed when the countdown clock is displaying a value between 1 and 9 seconds), 15 seconds (for bids placed when the countdown clock is displaying a value between 11 and 14 seconds) or 20 seconds (for bids placed when the countdown clock is displaying a value between 16 and 19 seconds). This means that the majority of the bids are placed when the auction is supposedly (but not really) about to end, and the countdown clock has a very low value for a great part of the auction duration. This is a common practice in most pay-per-bid auction sites.

In average, for the auctions recorded in the traces, the winner places about 20% of all of the bids placed in the auction. Winners place around 35 bids per auction, while losers place around 6 bids per auction. Consequently, winning an auction normally implies placing a large number of bids as well as placing more bids than the other participants.

In average, around 25% of all of the bids placed with Bid Butlers during a single auction are placed by the winner. Also, around 50% of all of the bids placed by winners are placed using Bid Butlers. Moreover, there is a chance of around 33% that the last bid placed in an auction (i.e., the winning bid) was placed by a Bid Butler. Consequently, using Bid Butlers seems to be a good choice to win auctions.

Nevertheless, for the auctions contained in the traces, the mean number of bids placed by the winners in the auctions that were won by placing the final bid with a Bid Butler is around 79 bids. The mean number of bids placed by the winners in the auctions that were not won by placing the final bid with a Bid Butler is much lower, around 28 bids. Therefore, although using Bid Butlers seems to be a good choice to win auctions, it generally implies placing a much larger number of bids.

For the auctions recorded in the traces, the mean profit obtained by Swoopo over the retail price of the items in the auctions that were won by placing the final bid with a Bid Butler is around 130$. The mean profit obtained by Swoopo for the auctions that were not won by placing the final bid with a Bid Butler is much lower, around 64$. Therefore, the fact that users choose to use Bid Butlers to win auctions is benefitial for Swoopo in terms of profit.

## 3.3. Obtaining the product categories
The column "desc" in the dataset contains a description about the product that is auctioned. Most of the products that are auctioned in the dataset are electronics (mobile phones, video games, laptops, televisions, etc.), but the dataset does not specify any category to which each product belongs to. This could be useful information, because it may ocurr that the auction results are different depending on the category of the product that is being offered.

Amazon.com is one of the largest Internet retailer in the world and sells or used to sell most of the products contained in the dataset. It also assigns a product category to each one of the products that it sells. This category could be useful to complete the missing product category information for the products contained in the dataset, and web scraping methods can be used to extract those categories.

The process that has been implemented to extract the product categories first obtains Amazon links related to the product name given as input. The links that are extracted are the first ones appearing in Google when a search is performed for that product name with the restriction that the search results should correspond to the Amazon website. Once that the Amazon link corresponding to the specific product has been obtained, the product category is extracted from the Amazon product site by searching for the corresponding HTML tag.

This process is repeated for each unique item contained in the dataset. The fields contained in the original dataset to describe the product (columns "item" and "desc") are then saved in a text file along with the extracted product category and the Amazon link from where the category has been extracted (in case that it is needed to extract more information from the same link in the future, such as product characteristics, product reviews, etc).

## 3.4. Transforming the product categories to word embedding vectors
As an example, the product with the description "PSP Slim & Lite Sony Piano Black" contained in the dataset is associated to the following product category "Video Games > Sony PSP> Consoles". However, it is necessary to find a way to introduce this information as input for the prediction model. 

Google's pre-trained Word2Vec model includes word vectors for a vocabulary of 3 million words and phrases that they trained on roughly 100 billion words from a Google News dataset.

A good way to group the product categories is to find clusters for the ones that have similar semantic meanings. A numerical representation of a product category semantic meaning can consist on a word embedding vector that encompasses the meaning of all of the words contained in the product category string. These word embedding vectors can later be used to define clusters according to the distances between them.

Given the product category string, each single word contained in the string is analyzed separately and its Google's pre-trained Word2Vec vector is obtained. Then, the average of the word embedding vectors of the words that compose the product category is calculated. This final vector represents the global semantic meaning of the product category.

After having performed this process for all of the items in the dataset, the final vectors have been saved in a new file, so that it is easy to have access to them in the future.

## 3.5. Building a prediction model for the final selling price of the auctions
In this Section, different sets of prediction models to predict the final selling price of an auction before it starts have been built.

Each set is compossed of the following types of predictors: a random forest regressor, a k-neighbors regressor, a decision tree regressor, a linear regression and a RANSAC regressor.

The final selling price of an auction depends on a lot of things: the users that are monitoring the auction, the amount of money that they have and their interests at the moment of the auction, etc. Even when the item that is auctioned is the same one, the final price that is reached for each one of the auctions can be very different. The most extreme values could be considered outliers and be removed from the dataset so that they do not negatively influence the model. However, in this case, the extreme values are legitimate observations and they are most likely not incorrectly entered data. Therefore, instead of removing them, it has been decided to include some prediction models that are robust to outliers. For example, random forests and decision trees isolate atypical observations into small leaves, and the RANSAC algorithm provides a robust linear model estimation.

The dataset has been divided into different parts for the training and the testing phases, and 5-fold cross-validation has been applied so that the resulting quality metrics are more reliable. The metrics that have been calculated are the mean absolute error and the median absolute error. The median absolute error is not highly influenced by outliers.

### 3.5.1. Single model - Without having into account the product categories
The first set of models that has been built does not take into account the product categories. The input variables for the model are: the retail price of the item, the bid increment, the bid fee, and the flags that indicate whether the auction is a click-only auction, a beginner auction, a fixed-price auction or and end-price auction.

For this set of models, the one with the best results in terms of the median absolute error is the RANSAC regressor. The decision tree regresor and the random forest regressor both present good results for the two metrics. The k-neighbors regressor performs alright, but worse than the other two. The worst results are obtained with the linear regression, probably because it is more sensitive to outliers than the others.

### 3.5.2. Single Model - Clustering by product categories
A second set of models that have into account the product categories have been built. As specified in previous sections, with the use of the product description available for each product in the column "desc" of the dataset, the Amazon product category has been obtained. A word embedding vector representing the product category has been calculated based on Google's pre-trained Word2Vec model. The distances defined by these vectors can be used to perform a distance-based clustering. For these models, a K-Means clustering has been used to group the products belonging to semantically similar categories.

The idea behind categorizing the products is that, if the final selling price of the items have similar patterns for similar products, a prediction model will perform better if a grouping based on the product categories is given as input. In this case, the input is the cluster identifier that each product belongs to. Since this is a categorical integer feature, a one-hot encoder has been used.

Therefore, for this set of models, the input variables are: the retail price of the item, the bid increment, the bid fee, the flags that indicate the type of auction, and the one-hot encoded cluster identifier. The prediction models and the metrics are the same as in the previous part.

The model with the best results is the random forest regressor, both in terms of the median absolute error and the mean absolute error. It is closely followed by the decision tree regressor. Although the random forest regressor performs better in terms of the chosen metrics, the decision tree regressor is easier to interpret. Nevertheless, the decision tree regressor is also more likely to overfit the data. The k-neighbors regressor is the next one with the better results, followed by the RANSAC regressor. Ultimatelly, the worst results are obtained with the linear regression.

In general, the results are better with this set of models that takes into account the product categories as compared with the results obtained with the set of models that does not take them into account (explained in Section 3.5.1).

Nevertheless, this set of models requires a bigger effort, since a preprocessing step is necessary to make predictions. This preprocessing step consists on, given the product description of an item in the column "desc", its Amazon category must be obtained, and afterwards, the corresponding word embedding vector.

Furthermore, during the training part of the model, the product category clusters are obtained, and these clusters are the ones that are used each time that a new prediction is made. After some time, more and more new products will begin to be auctioned, which will not have been considered to form the clusters. Because of this, the clusters may eventually become outdated, and the models will perform worse. Therefore, appart from the preprocessing step, using the product categories also involves having to retrain the prediction models from time to time so that clusters are updated.

### 3.5.3. Multiple models - Clustering by product categories
A third set of models that have into account the product categories has been built. The difference between this set of models and the one explained in Section 3.5.2 is that instead of using a single model for all items, a different prediction model is used for each different product category (i.e., each cluster).

For this set of models, the model with the best results in terms of the median absolute error is the RANSAC regressor. Nevertheless, the random forest regressor performs similarly in terms of the median absolute error, and performs way better in terms of the mean absolute error. It is closely followed by the decision tree regresor, which is easier to interpret but is also more likely to overfit the data. The k-neighbors regressor is the next one with the better results, followed by the linear regression with the worst results by far.

In general, the results are slightly better with this set of models as compared with the set of models explained in Section 3.5.2.

In terms of effort, this model requires the same preprocessing step to make predictions as the previous set of models: given the product description of an item in the column "desc", its Amazon category must be obtained, and afterwards, the corresponding word embedding vector.

A different prediction model is used for each different product category (cluster) that is obtained during the training part of the model. In the same way as with the set of models mentioned in Section 3.5.2, more and more new products will begin to be auctioned after some time, and the clusters (and therefore, the model assigned to each cluster) will become outdated and the models will perform worse. Therefore, appart from the preprocessing step, the prediction models have to be retrained after some time (and this time, it is not necessary to retrain a single model, but to retrain as many prediction models as existing clusters).

The decision between using a single model to make the predictions (as explained in Section 3.5.2), or using a different model for each product category should also be highly influenced by the amount of training data available. The data that is used to train the prediction model corresponding to a single cluster is only the one that contains items associated to the product category corresponding to that cluster. If the amount of training data is low, the model will perform badly when making predictions for that cluster.

In this case, the metrics have been calculated as the average results between the models corresponding to the different clusters. Therefore, it may happen that the prediction model associated to a certain cluster performs much worse than the ones associated to the other clusters (because the amount of training data for that cluster is low).

In conclusion, using a different prediction model for each cluster may be interesting when the amount of training data is big, but if this is not the case, it is better to use a single prediction model for all clusters.

### 3.5.4. Single model - Clustering by product categories and retail prices
A fourth set of models that have into account the product categories have been built. In this case, a single prediction model is used to make the predictions. The difference is that the product category clusters have not only been built with the word embedding vector representing the product category, but also with the retail price of the item.

This has been done to divide the products not only by their category, but also by their retail price. For example, mobile phone auctions can consist on high-end and mid-range mobile phones. The product category in both cases is mobile phones, but the selling price of high-end mobile phones will likely be higher than for mid-range mobile phones. Therefore, if different clusters are created for these two cases, the prediction model may perform better.

During the training part, the K-Means algorithm is used to obtain the clusters. The input given to the clustering algorithm is the word embedding vector and the retail price of each different product. The values of the retail prices are much higher than the values of the word embedding vectors, and therefore, they have to be scaled accordingly.

Since there are outliers in the retail prices distribution, a robust scaler has been used. This scaler is similar to the Min-Max scaler, but instead of the maximum and minimum values, it uses percentiles.

For this set of models, the model with the best results is the decision tree regressor. The random forest regressor and the the k-neighbors regressor perform slightly worse. Ultimately, the RANSAC regressor and the linear regression perform significantly worse.

In general, the results are worse with this set of models as compared with the set of models explained in Section 3.5.3.

In terms of effort, this model requires several preprocessing steps: given the product description of an item in the column "desc", its Amazon category must be obtained, and afterwards, the corresponding word embedding vector. Moreover, the retail price of the item has to be scaled before making the prediction.

Appart from the preprocessing steps, during the training part of the model, the clusters and the scaler instance are obtained, and they are later used everytime that a new prediction is made. After some time, more and more new products will begin to be auctioned, which will not have been considered to form the clusters and to calculate the scaling values. Because of this, the clusters and the scaler instance may eventually become outdated, and the models will perform worse.

For the given dataset, this set of models is not interesting, since it is more complex and performs worse than some of the set of models explained in other sections. As the amount of data increases, it could be interesting to analyse the performance of this set of models again.

### 3.5.5. Multiple models - Clustering by product categories and retail prices
A fith set of models that calculate clusters based on the product categories and retail prices has been built. The difference between this set of models and the previous one is that instead of using a single model for all items, a different prediction model is used for each different cluster.

For this set of models, the model with the best results in terms of the median absolute error is the RANSAC regressor, while the model with the best results in terms of the mean absolute error is the random forest regressor. The results for the decision tree regressor are very similar to the ones obtained with the random forest regressor. The k-neighbors regressor results are, although worse, very similar too. The worst results are obtained with the linear regression.

As compared with Section 3.5.4 in which a single model is used, the results obtained with multiple models in this section are better, but in general, the results are worse than the ones obtained with the set of models in other sections. For the given dataset, this set of models is not interesting, since it is more complex and performs worse than some of the set of models explained in other sections. As the amount of data increases, it could be interesting to analyze the performance of this set of models again.

### 3.5.6. Choosing and optimizing the final model
In general, the random forest regressor and the decision tree regressor are the algorithms that have provided the best results. These two models are relatively robust to outliers, since they isolate atypical observations into small leaves. The random forest regressor generally performs a bit better than the decision tree regressor and is less likely to overfit the data. The disadvantage is that random forest regressor are more difficult to interpret than decision tree regressors.

Two different clustering options have been analyzed: one that uses the word embedding vectors to cluster the items according to their product categories, and another one that, appart from the word embedding vectors, it also uses the retail prices of the items to obtain the clusters. With the available data, the model that obtains the clusters by only using as input the word embedding vectors performs better. Moreover, it also requires less preprocessing steps and having to retrain the model less often.

The results obtained with the models that take into account the product categories are better than the ones obtained with the models that do not take them into account. Nevertheless, taking into account the product categories requires a bigger effort, since a preprocessing step is necessary to make a new prediction: given the product description of an item in the column "desc", its Amazon category must be obtained, and afterwards, the corresponding word embedding vector.

Furthermore, the clusters product category clusters obtained during the trianing part of the model are the ones that are used each time that a new prediction is made. After some time, more and more new products will begin to be auctioned and the clusters may eventually become outdated, causing the model to perform worse. Therefore, appart from the preprocessing step, using the product categories also involves having to retrain the prediction model from time to time.

For the models that take only take into account the word embedding vectors to form the clusters, two different versions have been analyzed: one consisting on a single prediction model and another one that consists on a set of different models, where the model chosen to make every prediction depends on the cluster that each row in the test data belongs to.

The results obtained with the set of multiple models were slightly better than the ones obtained when using a single model. Nevertheless, one of the prediction models corresponding to a certain cluster might perform much worse than the others.

Also, in both versions, the clusters will eventually become outdated, and when a retraining is necessary, a single prediction model will have to be retrained in one of the versions, while in the other one, as many prediction models as existing clusters will have to be retrained. Considering this disadvantage, as well as the bigger complexity that using a set of multiple models implies, and the fact that the results are fairly similar for both versions, it has been decided to choose the version consisting on a single prediction model as the final one.

Therefore, the final chosen set of models is the one explained in [Section 3.5.2](#352-single-model---clustering-by-product-categories). Of the different prediction algorithms used in [Section 3.5.2](#352-single-model---clustering-by-product-categories), the one with the best results is the random forest regressor. This is the one that has been implemented as the final model.

The variables that have a clear impact on the results for this model are: the number of clusters chosen for the K-Means clustering algorithm, as well as the input parameters for the random forest regressor. A grid of different values for the number of clusters and the number of trees used in the random forest algorithm has been created.

The prediction model metrics results for the best combination of variables found are:
```sh
Median absolute error = 10.37 $
Mean absolute error = 28.29 $
```
# 4. Conclusions
As indicated in [Section 3.2](#32-exploring-and-cleaning-the-dataset), Swoopo obtains a negative profit (i.e., Swoopo obtains less money for the auction than the retail price of the item) for 47.53% of the auctions. The number of bids placed in an auction can be calculated as the final price of the auction divided by the bid price increment. The total money that Swoopo obtains for an auction is calculated as the number of bids placed multiplied by the bid fee, plus the final selling price of the item that is paid by the winner. Since both the bid fee and the bid price increment are known variables, the fact whether Swoopo losses or wins money for a certain auction can be directly derived if the final selling price of the auction is known. The prediction model developed in [Section 3.5.5](#355-choosing-and-optimizing-the-final-model) can be used to predict the selling price of an auction. These predictions could be used to decide whether to offer certain auctions or not depending on whether the profit would be positive or negative based on the prediction results. This could dramatically lower the percentage of auctions for which Swoopo losses money.

As mentioned in [Section 3.2](#32-exploring-and-cleaning-the-dataset), in average, the benefit that the winner obtains over the retail price of the item is around 160$. This is a very positive piece of news for the participants. Perhaps the data about the benefit that the each winner obtained over the retail price of the item for every auction could be made publicly available to encourage users to participate.

In the analysis performed for [Section 3.2](#32-exploring-and-cleaning-the-dataset), a ranking of winner users was made. This information could be used for considering the possibility of adding a reward system. For example, a ranking of the winners could be made publicly available in the website, and the top winners could be rewarded from time to time to encourage general participation and competitiveness. The rewards could for example consist on pack of free bids, which would also motivate them to keep participating in future auctions.

In the analysis performed for [Section 3.2](#32-exploring-and-cleaning-the-dataset), it was observed that November, December and January are the months in which more bids are placed. It could be interesting to increase the number of auctions during these months. The information about the profit ratio for each week of the year and special days (fixed holidays) has also been obtained and could be used to manage the moments in which it is preferable to offer more or less auctions.

The average auction duration (calculated as the time that passed between the first bid and the last bid placed for that auction) is 7 hours and 10 minutes. Each bid extends the countdown clock by 10–20 seconds, so auctions can theoretically continue on indefinitely. As observed in the the analysis performed for [Section 3.2](#32-exploring-and-cleaning-the-dataset), the majority of the bids are placed when the auction is supposedly (but not really) about to end, and the countdown clock has a very low value for a great part of the auction duration. Therefore, the countdown clock should be set so that the auction theoretical ending time is a time in which a lot of users are able to participate in the auctions: for example, a couple of hours after the average ending work schedule time, and some hours before people go to bed.

Swoopo provides the user with the possibility of using Bid Butlers (automatic bidding agents that place automated bids according to user-defined restrictions when the auction timer drops below 10 seconds). There is a chance of around 33% that the last bid placed in an auction (i.e., the winning bid) is placed by a Bid Butler. Consequently, using Bid Butlers seems to be a good choice for the participants to win auctions. 

For the auctions contained in the traces, the mean number of bids placed by the winners in the auctions that were won by placing the final bid with a Bid Butler is around 79 bids. The mean number of bids placed by the winners in the auctions that were not won by placing the final bid with a Bid Butler is much lower, around 28 bids. Therefore, although using Bid Butlers seems to be a good choice to win auctions, it generally implies placing a much larger number of bids. This is good for Swoopo's business. For the auctions recorded in the traces, the mean profit obtained by Swoopo over the retail price of the items in the auctions that were won by placing the final bid with a Bid Butler is around 130$. The mean profit obtained by Swoopo for the auctions that were not won by placing the final bid with a Bid Butler is much lower, around 64$.

In conclusion, offering and using BidButlers is a win-win strategy both for Swoopo and the auction participants.

As explained in [Section 3.5.2](#352-single-model---clustering-by-product-categories), including the word embedding vectors for the product categories improved the performance of the selling price prediction model. This suggests that the prediction model could be further improved by completing the dataset with extra information. During the product category extraction proccess detailed in [Section 3.3](#33-obtaining-the-product-categories), the Amazon product links where the product categories have been obtained from have been saved for future uses. When people are deciding whether to purchase or not an item online, a lot of them first look for user reviews and average rating scores. All of this information can also be extracted from the Amazon product link. Extracting the average rating is straightforward, and a sentiment analysis could be done for user reviews to find out whether the comments are mostly positive or negative. Having all of this information can help to improve the prediction model: it is very likely that, on average, products with good ratings and good user reviews will most likely reach higher selling prices since more participants will be eager to purchase them based on their online research.

As it can be observed by having a look at the clustering result examples contained in the code that has been developed for [Section 3.5](#35-building-a-prediction-model-for-the-final-selling-price-of-the-auctions), the word embedding vectors obtained for the product categories in [Section 3.4](#34-transforming-the-product-categories-to-word-embedding-vectors) are very useful to group similar items. These word embedding vectors can have more applications appart from improving the prediction model results. For example, they could be used within a recommendation system that recommends users similar products to the ones that they have placed bids for in the past (especially if they did not win those auctions).

The final prediction model obtained in [Section 3.5.6](#355-choosing-and-optimizing-the-final-model) allows making final selling price predictions with a median absolute error of 10.37$. This can help Swoopo to forecast its benefits, as well as deciding whether to offer certain auctions or not depending on whether the profit would be positive or negative based on the prediction results.

In conclusion, insights that can help Swoopo to improve its business in terms of profit and management have been found in this Master Thesis.

# 5. Front-end
![](https://github.com/sergio-valmorisco-sierra/tfm_data_science_sergio_valmorisco_sierra/tree/master/images/dashboard_01.jpg)
![](https://github.com/sergio-valmorisco-sierra/tfm_data_science_sergio_valmorisco_sierra/tree/master/images/dashboard_02.jpg)
![](https://github.com/sergio-valmorisco-sierra/tfm_data_science_sergio_valmorisco_sierra/tree/master/images/dashboard_03.jpg)
![](https://github.com/sergio-valmorisco-sierra/tfm_data_science_sergio_valmorisco_sierra/tree/master/images/dashboard_04.jpg)

# 6. Future Work






  - Import a HTML file and watch it magically convert to Markdown
  - Drag and drop images (requires your Dropbox account be linked)


You can also:
  - Import and save files from GitHub, Dropbox, Google Drive and One Drive
  - Drag and drop markdown and HTML files into Dillinger
  - Export documents as Markdown, HTML and PDF

Markdown is a lightweight markup language based on the formatting conventions that people naturally use in email.  As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually* written in Markdown! To get a feel for Markdown's syntax, type some text into the left window and watch the results in the right.

### Tech

Dillinger uses a number of open source projects to work properly:

* [AngularJS] - HTML enhanced for web apps!
* [Ace Editor] - awesome web-based text editor
* [markdown-it] - Markdown parser done right. Fast and easy to extend.
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [node.js] - evented I/O for the backend
* [Express] - fast node.js network app framework [@tjholowaychuk]
* [Gulp] - the streaming build system
* [Breakdance](http://breakdance.io) - HTML to Markdown converter
* [jQuery] - duh

And of course Dillinger itself is open source with a [public repository][dill]
 on GitHub.

### Installation

Dillinger requires [Node.js](https://nodejs.org/) v4+ to run.

Install the dependencies and devDependencies and start the server.

```sh
$ cd dillinger
$ npm install -d
$ node app
```

For production environments...

```sh
$ npm install --production
$ NODE_ENV=production node app
```

### Plugins

Dillinger is currently extended with the following plugins. Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| Github | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |


### Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantanously see your updates!

Open your favorite Terminal and run these commands.

First Tab:
```sh
$ node app
```

Second Tab:
```sh
$ gulp watch
```

(optional) Third:
```sh
$ karma test
```
#### Building for source
For production release:
```sh
$ gulp build --prod
```
Generating pre-built zip archives for distribution:
```sh
$ gulp build dist --prod
```
### Docker
Dillinger is very easy to install and deploy in a Docker container.

By default, the Docker will expose port 8080, so change this within the Dockerfile if necessary. When ready, simply use the Dockerfile to build the image.

```sh
cd dillinger
docker build -t joemccann/dillinger:${package.json.version}
```
This will create the dillinger image and pull in the necessary dependencies. Be sure to swap out `${package.json.version}` with the actual version of Dillinger.

Once done, run the Docker image and map the port to whatever you wish on your host. In this example, we simply map port 8000 of the host to port 8080 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 8000:8080 --restart="always" <youruser>/dillinger:${package.json.version}
```

Verify the deployment by navigating to your server address in your preferred browser.

```sh
127.0.0.1:8000
```

#### Kubernetes + Google Cloud

See [KUBERNETES.md](https://github.com/joemccann/dillinger/blob/master/KUBERNETES.md)


### Todos

 - Write MORE Tests
 - Add Night Mode

License
----

MIT


**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
