# Data Science Applied to Pay-Per-Bid Auctions
### Master Thesis - VI Master in Data Science - KSchool
##### Sergio Valmorisco Sierra
##### 4th June, 2018

# 1. Introduction
Pay-per-bid auctions are a special kind of auctions in which bidders incur a cost (bid fee) for placing each bid in addition to (or sometimes in lieu of) the winner's final purchase cost. Also, each bid that is placed increases the final purchased cost by a fixed amount. Pay-per-bid auctions begin at a reserved price (generally 0), and have an associated countdown clock. When a player places a bid, some additional time is added to the clock. When the clock expires, the last bidder has the chance to purchase the item at the final auction price.

*Example: users A and B have participated in a pay-per-bid auction. User A has placed 30 bids, user B has placed 60 bids, the bid fee is 1$ and the bid increment to the selling price is 0.10$. User B has won the auction because the countdown clock expired after user B placed the last bid.*

In this example, user A has paid 30$ (number of bids placed by user A multiplied by the bid fee) but has not won the auction, so user A does not get to purchase the item. User B has paid 60$ (number of bids placed by user B multiplied by the bid fee) and has won the auction, so user B gets to purchase the item by paying the final selling price of 9$ (number of total bids placed multiplied by the bid increment). In total, user B has spent 69$. The total money obtained by the auction site is 99$ (sum of the expenses of users A and B).

Swoopo was one of the most famous pay-per-bid auction commercial websites up to 2011. In order to participate in auctions, registered users had to first buy bids (called credits). Each bid extended the countdown clock by 10–20 seconds, so auctions could theoretically continue on indefinitely.

Appart from placing manual bids, Swoopo also provided the possibility of placing bids with BidButlers. These were automatic bidding agents provided by the Swoopo interface that bidded according to user-defined instructions. Users were able to define starting and ending price limits for which automated bids shall be placed, as well as the top number of bids that the BidButler was allowed to place. BidButlers placed automated bids when the auction timer dropped below 10 seconds.

Swoopo also offered special types of auctions:
  - Click-only auctions: also called "NailBiter" auctions. These auctions did not allow the use of BidButlers.
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
This file is based on information published directly by Swoopo and contains the following information for each auction:

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
| flg_click_only | a binary flag indicating a "NailBiter" auction (Swoopo auctions which do not permit the use of automated bids by a BidButler) |
| flg_beginnerauction | a binary flag indicating a beginner auction (auctions restricted to users who haven't previously won any auction in the site) |
| flg_fixedprice | a binary flag indicating a fixed-price auction (an auction in which the selling price is fixed from the beginning) |
| flg_endprice | a binary flag indicating a 100%-off auction (auctions in which the final selling price of the product is zero, and the revenue for Swoopo comes exclusively from the bids that are placed) |

## 2.2. traces.tsv
This file is based on traces of live auctions that the authors of the dataset recorded using their own recording infrastructure. It comprises 7,352 auctions conducted between October 1, 2009 and December 12, 2009. The traces include detailed bidding information for each auction.

The authors probed Swoopo at semi-regular time-intervals (more frequently near the end of the auction when bids are placed in rapid succession, less frequently near the beginning of the auction when bidding is sparse). When the countdown clock had less than 2 minutes left on it, they probed at the rate Swoopo suggested but at least once every second (every probe response was associated with a Swoopo defined update-interval that Swoopo uses to instruct the user’s browser when to update next).

The result of each probe was a list of up to ten tuples of the form (username,bidnumber), indicating the players that placed a bid since the previous probe and the order in which they did so. In the cases the list included more that one tuple, the authors ascribed the same timestamp to all of those bids.

A limitation from the authors' probing methodolody arises when more then ten players bid between successive probes. In particular, this happened when more than ten players were using BidButlers. In these cases, Swoopo responded with just the ten latest bids.

More details about the methodology used by the authors to create the dataset can be found at: *Byers, J. W., Mitzenmacher, M., & Zervas, G. (2010, June). Information asymmetries in pay-per-bid auctions. In Proceedings of the 11th ACM conference on Electronic commerce (pp. 1-12). ACM.*

The file contains the following columns:

| Column        | Description          
| ------------- |------------- | 
| auction_id | unique numerical id for the auction | 
| bid_time | the date and time of each bid | 
| bid_ct | the value of the countdown clock (seconds) at the time the bid was reported to the researchers that recorded the traces | 
| bid_number | the number of the bid placed in the auction in ascending order, starting with 1 | 
| bid_user | the username of the bidder | 
| bid_butler | 1 for BidButler bids, 0 otherwise | 
| bid_cp | the price of the item after the bid was placed | 
| bid_user_secs_added | the number of seconds added to the countdown clock as a result non-BidButler bidded in the bid group (*) | 
| bid_butler_secs_added | the number of seconds added to the countdown clock as a result BidButler bidded in the bid group (*) | 
| bid_infered | 0 for the final bid in a bid group, 1 otherwise (*) | 
| bid_group | bid group identifier (*). Bids that were reported as part of the same group will have the same group number | 
| bid_final | 1 for the winning bid (the last one), 0 otherwise | 

**(\*) bid groups**: Occassionaly, between successive probes, more than one bids would have occured and would be reported together. The authors refer to this as a bid group. All bids in a bid group have been ascribed the same timestamp

The other parameters of the auction can be looked up in outcomes.tsv using the auction_id field.

# 3. Methodology
The first step of the development process consists on the data acquisition. In this Master Thesis, the dataset that has been used is available to the public, and the authors of the dataset have already recorded traces of live auctions using their own recording infrastructure. Therefore, the data acquisition phase is minimal in this Master Thesis.

Nevertheless, as indicated in Section 2.1, the dataset contains a brief description about the product that is being auctioned. It could be interesting for the prediction model to have a product category to which the item being auctioned corresponds and not just a product description. This is because the final selling price of an item could greatly depend on its product category. The data acquisition process that has been followed to obtain the product category corresponding to each item is described in detail in Section 3.3.

After the data acquisition phase, a data cleaning process is necessary so that the results that are obtained in the end are of high quality. In Section 3.2, the process used to clean the data is explained. Afterwards, the data has to be analysed in order to obtain valuable insights. The analysis process is explained in Section 3.2.

After the data has been cleaned and analysed in Section 3.2, the product categories of the items have been obtained in Section 3.3. As an example, the product description for one of the items appearing in the dataset is "Sony Ericsson S500i Unlocked Mysterious Green". Although a human could identify by reading this description that the product is a mobile phone, a machine is not able to do this. Amazon.com is one of the largest Internet retailer in the world and sells or used to sell most of the products contained in the dataset. It also assigns a product category to each one of the products that it sells. For example, for the product previously indicated, the corresponding Amazon product category is "Cell Phones & Accessories › Cell Phones › Unlocked Cell Phones".

This category could be useful to complete the missing product category information for the products contained in the dataset, and web scraping methods can be used to extract these categories. This is explained in more detail in Section 3.3.

Once that these product categories have been obtained, it is necessary to find a way to introduce them as input information for the prediction model. Google's pre-trained Word2Vec model includes word vectors for a vocabulary of 3 million words and phrases that they trained on roughly 100 billion words from a Google News dataset.

A good way to group the product categories is to find clusters for the ones that have similar semantic meanings. A numerical representation of a product category semantic meaning can consist on a word embedding vector that encompasses the meaning of all of the words contained in the product category string. These word embedding vectors can later be used to define clusters according to the distances between them. This is explained in more detail in Section 3.4.

Finally, in Section 3.5, different prediction models to predict the final selling price of an auction before it starts have been built, and their results have been analysed to idenfity the best one.

## 3.1. Technology and requirements
Since the size of the model is 1.5GB, it is not included in the GitHub folder. Nevertheless, for the next piece of code to work, the model should be downloaded and put in the corresponding folder.

The link to download the model is: https://drive.google.com/file/d/0B7XkCwpI5KDYNlNUTTlSS21pQmM/edit?usp=sharing

## 3.2. Exploring and cleaning the dataset
## 3.3. Obtaining the product categories
## 3.4. Transforming the product categories to word embedding vectors
## 3.5. Building a prediction model for the final selling price of the auctions
# 4. Results

# 5. Front-end

# 6. Future Work




