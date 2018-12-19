# Market-Basket-Analysis
## MBA study with the goal to provide strategic insights to E-Commerce Website.

![Figure 1-1](https://raw.github.com/clone95/Market-Basket-Analysis/master/mba.png "1")

For the Uni me and a mate we made a MBA study. I put here all our job in order to show what we did and also to help people who are new to data science!! 

This project involves SQL, Python and Orange Data Mining framework, which is required to get the things running. Install the package Orange3 on IDE or through 
  pip  --install orange3 

In the first phases we look at the dataset and import it from .CSV to PostgreSQL in order to manipulate it. Then we build a .basket file for the analysis, and in the end we provide some pratical aggregation rules (for products) to the E-commerce website.

I post here the whole report on the study, plus some screens and general consideration about this data mining process.

### Introduction

During the meeting with the “We-Commerce” company, their innovation group asked us for technical advises on how to generate knowledge (i.e., business value) from the large volume of data that “We-Commerce” has been storing throughout time. The “We-Commerce” captures and stores each event generated by visitors that navigates through their e-commerce Web sites.

### Attributes:

tracking_record_id is the ID of every action carried out on whatever page of the website, when a record is inserted this id is incremented. The type is a Serial. 

date_time is basically a date and a timestamp that represents when the user makes an action on the website (mentioned before). The type is a DateTime. 

user_gui is a unique identifier of users assigned the first time a visitor arrives on the website and register on it. The type of this attribute is a Varchar, it must respect this form: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX 

campaign_id identifies a promotional campaign that the consulted product belongs to, Varchar 15. Usually just the registered users have this campaign. 

product_gui is the identifier of a product visited by a visitor within a session, it could provide useless informations, such as Thome", "open","/costumer/account/loginr,"/costumer/account/forgotpassword". Varchar 100. 

company_id represents the name of the company providing the product_gui. Varchar 100. 

link is the URI of the visited page. Varchar 100.

tracking_id is the characters encoding the browser is using. Varchar 6. 

meio represents the way of entrance the site is reached. Varchar 6.

IP register every ip address of a visitor. Varchar 30.

browser is the type of browser used to enter the website. Varchar 100. 

session_id is created automatically when a user arrives to a page of the website. This session id expires when the browser is closed, or the visitor has been inactive for some time. Varchar 26. 

referer is the page where the user come from. Varchar 100.

cookie_id is a global unique way to identify users in the website (for each event), it contains the retrieved value of the stored cookie in the visitor's browser (assigned in the first session through the user_GUI). Varchar 36, it must respect this form: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX.

### Assumptions:
We decided to formulate the requests as a Market Basket Analysis problem because it would be interesting to understand which products are sticked together in the consumer mind, in order to maximize the probability of buying more products.
There are several ways to aggregate information about the products that often are together in the consumer purchase habits, so we analyzed some possibilities and come up with the most significant one. We considered the Product_GUI, the attribute that carries informations about which product is visited. Regarding the aggregator attribute (in order to create “baskets”to analyze) we can choose are cookie_id, session_id and IP adress. The adresses are probably useless because they are mostly dinamic and they can change over time.
The cookie_id is less useful than the session one, because we can add some noise to the analysis: someone can use the computer of a friend and cleaning programs often clean cookie from the browser. So, we came up with the session_id as the best aggregator in order to creation of the baskets.


To provide value to the company, we formulated the problem in this way: we want to aggregate each event (on the product_GUI attribute) during a session and consider that a basket. We want to know which items are bought together during the user’s session, and thus more likely to be related in some form in the consumer habits. 
Retailers can use the insights gained from MBA in several ways, including:
•	Grouping products that co-occur in the design of a store’s layout to increase the chance of cross-selling;
•	Driving online recommendation engines (“customers who purchased this product also viewed this product”)
•	Targeting marketing campaigns by sending out promotional coupons to customers for products related to items they recently purchased.



Apriori is an algorithm for frequent item set mining and association rule learning over transactional databases. It proceeds by identifying the frequent individual items in the database and extending them to larger and larger item sets if those item sets appear sufficiently often in the database.
We consider two metrics in order to classify how “strong” is a rule among products:
Support: the percentage of transactions that contain all the items in an itemset (e.g., pencil, paper and rubber). The higher the support the more frequently the itemset occurs. Rules with a high support are preferred since they are likely to be applicable to many future transactions.
Confidence: the probability that a transaction that contains the items on the left-hand side of the rule (in our example, pencil and paper) also contains the item on the right-hand side (a rubber). The higher the confidence, the greater the likelihood that the item on the right-hand side will be purchased or, in other words, the greater the return rate you can expect for a given rule.
We can have more than one product implying one product, for example if there is a strong rule involving more than 2 items, the algorithm will find it. 
(for example, A,B,C  --> D).



The dataset is given in a .csv format, and we imported it in PostgreSQL in order to manipulate it. 
The dataset is large and with noise (in terms of attributes and attributes value).
Let’s start gaining some insights on the dataset form, understanding dimensions, occurrences of values and so forth.
With the group by SQL Function,  we’ve found:
•	total number of events, which is the number of records
•	total number of visitors, number of different cookies
•	distribution on n.visitors vs n.session
•	distribution on n. of events per session vs n.visitors



Being the dataset large, we reduced the number of data to the most relevant ones. So, we decided to include only the events generated from the users with a consistent number of session, between 5 and 30. Only upon these events we will build the .basket file.



Using the psql command we extracted the file containing, for each line, the session_id and an event associated with that session. In the next step we aggregate for session_id and we obtain the .basket format.



Now our data are almost ready to transform them into a .basket file. But we still have noise given by un-normalized strings: we want to delete spaces, trails, accents, and uppercase all letters, and then generate the .basket file in the Orange Data Mining format. Now we can order to explore it through the Orange Desktop application and gain some rough insights about the distribution of item sets and rules, given minimum support and minimum confidence.



Being MBA an unsupervised learning algorithm, it can be expensive in finding solutions, both in terms of time and computational resource. 
Also, we are looking for a small amount of rules to apply, if not it would be dispersive for the company to reach those objectives. They don’t need 10000 weak rules, but 10 or 20 stron ones. So, we accept only rule_books with 8 < n.rules < 15.
In order to reduce the space where to look for “strong” rules, we tweaked Orange parameters in the “Assocation Rules” widget up to realize some important things:
•	There is no rule found further the 7% of support respect to the .basket file  
•	If we put the confidence to the maximum, the first “good” min support value falls at roughly 3%. For min supports below 3 % we have an “explosion” of rules, which are valueless for our purposes.
•	Between 0 and 60 %  of minimum confidence we find too much rules, fixed the the most stringent min support (7 %), then it doesn’t make sense to explore this range  (conf 0 -> 60%)

So, for our purposes, it makes sense to explore just between 3% and 7-8% of minimum support, and for each incrementale step in support, consider the minimum confidence between 60% and 100%. This allow us to reduce dramatically the computational time because the search space is a lot smaller thanks to these considerations.

To give expendable knowledge to We-Commerce, we decide to consider only three situations which are three blue points in the Excel Table.

![Figure 1-1](https://raw.github.com/clone95/Market-Basket-Analysis/master/Capture.PNG "2")


## Low supp with the best confidence:  12  RULES (3 or 2 items -> 1 item)

(['lon_4004', 'lon_2125', 'lon_4508'], ' ---> ', ['lon_4504'], 'supp', 5.226209048361934, 'conf', 0.9054054054054054)

(['lon_4004', 'lon_4504', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 5.226209048361934, 'conf', 0.9178082191780822)

(['lon_2119', 'lon_4504', 'lon_4508'], ' ---> ', ['lon_2125'], 'supp', 4.05616224648986, 'conf', 0.8813559322033898)

(['lon_2119', 'lon_4504', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 4.05616224648986, 'conf', 0.896551724137931)

(['lon_4504', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 6.5522620904836195, 'conf', 0.875)

(['lon_4004', 'lon_4504'], ' ---> ', ['lon_4508'], 'supp', 6.240249609984399, 'conf', 0.8888888888888888)

(['lon_4504', 'lon_4013'], ' ---> ', ['lon_2125'], 'supp', 4.446177847113884, 'conf', 0.890625)

(['lon_4504', 'lon_4013'], ' ---> ', ['lon_4508'], 'supp', 4.368174726989079, 'conf', 0.875)

(['lon_4108', 'lon_4504'], ' ---> ', ['lon_2125'], 'supp', 4.212168486739469, 'conf', 0.8709677419354839)

(['lon_4108', 'lon_4504'], ' ---> ', ['lon_4508'], 'supp', 4.212168486739469, 'conf', 0.8709677419354839)

(['lon_2119', 'lon_4004'], ' ---> ', ['lon_4504'], 'supp', 4.290171606864274, 'conf', 0.8870967741935484)

(['lon_2119', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 4.758190327613105, 'conf', 0.8840579710144928)


In this case we have those products that are more rarely visited but are often visited together. 

## High supp with the best confidence:  6 RULES (only 1 item -> 1 item)

(['lon_4508'], ' ---> ', ['lon_2125'], 'supp', 8.424336973478939, 'conf', 0.7012987012987013)

(['lon_2125'], ' ---> ', ['lon_4508'], 'supp', 8.424336973478939, 'conf', 0.7346938775510204)

(['lon_2125'], ' ---> ', ['lon_4504'], 'supp', 7.48829953198128, 'conf', 0.6530612244897959)

(['lon_4504'], ' ---> ', ['lon_2125'], 'supp', 7.48829953198128, 'conf', 0.732824427480916)

(['lon_4504'], ' ---> ', ['lon_4508'], 'supp', 7.800312012480499, 'conf', 0.7633587786259542)

(['lon_4004'], ' ---> ', ['lon_4508'], 'supp', 7.410296411856475, 'conf', 0.7480314960629921)


In this case we have group of products often bought but not so related. This is important because these products are a considerable part of the total things sold, so a small improve in this subset can provide a big value to the company.

## A good trade-off point: 11 RULES (mostly 2 item -> 1 item)

(['lon_2125', 'lon_4508'], ' ---> ', ['lon_4504'], 'supp', 6.5522620904836195, 'conf', 0.7777777777777778)

(['lon_4504', 'lon_4508'], ' ---> ', ['lon_2125'], 'supp', 6.5522620904836195, 'conf', 0.84)

(['lon_4504', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 6.5522620904836195, 'conf', 0.875)

(['lon_4504', 'lon_2125'], ' ---> ', ['lon_4004'], 'supp', 5.694227769110764, 'conf', 0.7604166666666666)

(['lon_4004', 'lon_2125'], ' ---> ', ['lon_4504'], 'supp', 5.694227769110764, 'conf', 0.8390804597701149)

(['lon_4004', 'lon_4504'], ' ---> ', ['lon_2125'], 'supp', 5.694227769110764, 'conf', 0.8111111111111111)

(['lon_4504', 'lon_4508'], ' ---> ', ['lon_4004'], 'supp', 6.240249609984399, 'conf', 0.8)

(['lon_4004', 'lon_4508'], ' ---> ', ['lon_4504'], 'supp', 6.240249609984399, 'conf', 0.8421052631578947)

(['lon_4004', 'lon_4504'], ' ---> ', ['lon_4508'], 'supp', 6.240249609984399, 'conf', 0.8888888888888888)

(['lon_4004', 'lon_4508'], ' ---> ', ['lon_2125'], 'supp', 5.77223088923557, 'conf', 0.7789473684210526)

(['lon_4004', 'lon_2125'], ' ---> ', ['lon_4508'], 'supp', 5.77223088923557, 'conf', 0.8505747126436781)

(['lon_4504'], ' ---> ', ['lon_4508'], 'supp', 7.800312012480499, 'conf', 0.7633587786259542)

(['lon_2119'], ' ---> ', ['lon_4508'], 'supp', 5.61622464898596, 'conf', 0.7741935483870968)


