# Megaline: Telecom operator (telecom_churn_prediction)

The main objective of this project is to carry out a preliminary analysis of the two prepaid plans offered by the telecom operator Megaline, Surf and Ultimate. 

Based on a relatively small selection of 500 Megaline customers and focusing on who the clients are, where they're from, which plan they use, and the number of calls they made and text messages they sent and the internet MB they consumed in 2018, this analysis is aimed to understand the clients' behavior and determine which prepaid plan brings in more revenue.

The following datasets containing info on the individual calls, messages and web sessions are to be stored in separate dataframes, as well as the table showing user's personal information and the one with each plan's features.

**1)Data preparation:** 

After loading and getting an overview of the data, some fixes and additions were made to each dataset to better prepare it for studying it:

**1) Plans: data on the plans:**

plan_name — calling plan name

usd_monthly_fee — monthly charge in US dollars

minutes_included — monthly minute allowance

messages_included — monthly text allowance

mb_per_month_included — data volume allowance (in megabytes)

usd_per_minute — price per minute after exceeding the package limits (e.g., if the package includes 100 minutes, the 101st minute will be charged)

usd_per_message — price per text after exceeding the package limits

usd_per_gb — price per extra gigabyte of data after exceeding the package limits (1 GB = 1024 megabytes)

***Description of the plans***

***Surf***

Monthly charge: $20

500 monthly minutes, 50 texts, and 15 GB of data

After exceeding the package limits:

1 minute: 3 cents

1 text message: 3 cents

1 GB of data: $10

***Ultimate***

Monthly charge: $70

3000 monthly minutes, 1000 text messages, and 30 GB of data

After exceeding the package limits:

1 minute: 1 cent

1 text message: 1 cent

1 GB of data: $7

***Data wrangling***

a) Since monthly megabytes consumption will be rounded up to Gigabytes, mb_per_month_included were converted into GB.

b) plan_name was set as the index of the dataframe, to more easily merge it with other datasets based on the plan each user has.

**2) Users:**

user_id — unique user identifier

first_name — user's name

last_name — user's last name

age — user's age (years)

reg_date — subscription date (dd, mm, yy)

churn_date — the date the user stopped using the service (if the value is missing, the calling plan was being used when this database was extracted)

city — user's city of residence

plan — calling plan name

***Data wrangling***

a) reg_date & churn date were converted to datetime.

b) The only column with missing values is: churn_date. The empty cells add up to 466.

As the Database contains values for the year 2018 and missing values mean the calling plan was being used when this database was extracted, they were replaced by the following year start date (1/1/2019). The idea of not using the year end date is to distinguish the ones that were still active at the end of the year from the ones whose churn date matches the last day of the year.

c) Registration month and churn month: since revenue from each user will be calculated on a monthly basis, two columns were added: reg_month and churn_month.

Based on their values, it's easy to calculate whether the user was active (was charged with the monthly pay) at a given month.

This will be estimated on the assumption that user's are charged both for the month they suscribed and the month they discontinued the service, regardless of the day of the month at which they became active/inactive and the calls, messages or GB they used.
For those users whose churn_date was missing and filled out with "1/1/2019" churn_month was set to 13 (so that it's always greater than the session month, and the user is deemed as active if it had already registered by that month).

**3) Calls:**

id — unique call identifier

call_date — call date

duration — call duration (in minutes)

user_id — the identifier of the user making the call

***Data wrangling***

a) call_date was converted to datetime, and duration was rounded up, since even if the call lasted just one second, it will be counted as one minute for invoicing.

b) Call month: Based on each call date, we'll add a column with the month at which they took place for adding up the total minutes cost per user each period. 

c) "duration" column name was changed by "call_duration".

**4) Messages**

id — unique text message identifier

message_date — text message date

user_id — the identifier of the user sending the text

***Data wrangling***

a) message_date was converted to datetime.

b) message month: Based on the date each message was sent, we added a column with the month for adding up the total messages sent per user each monthly period. 


**5) Internet**

id — unique session identifier

mb_used — the volume of data spent during the session (in megabytes)

session_date — web session date

user_id — user identifier

***Data wrangling***

a) session_date was converted to datetime.

b) For web traffic, individual web sessions are not rounded up. Instead, the total for the month is rounded up. If someone uses 1025 megabytes this month, they will be charged for 2 gigabytes. Hence, MB will be totalized per used_id & month, and rounded up later on.

c) session month: Based on the date of each session, we will add a column with the month for adding up the total MB used per user in each monthly period. 


**2) Aggregate monthly data per user**

For calls, messages and internet datasets, the total calls duration, messages sent and MB consumed was added up by user each month.

At this point, total MB consumed monthly were aggregated and rounded up to GB.


After that, a new dataframe indexed with month(from 1 to 12) and each unique user_id (repeated once each month) was created. Its total length was 500 users x 12 months = 6000 rows. The previously aggregated data was merged to this dataframe, as well as some relevant users' features:

'plan','reg_month', 'churn_month','reg_date','churn_date','city'

Finally, based on each user's plan, data on the GB, minutes and messages included, as well as on the monthly pay and the cost of extras was merged to the dataset as well.


**3) Create dataset with active users only** 

If at a given month the user had not registered yet, or had already discontinued the service (churn month < month), the monthly pay was not charged to her/him in that period (revenue=0). To separate only rows were the user was active that month (every user will be accounted for as active if the reg month = session month or churn_month = month, regardless of the day they registered/resigned, or their consumption). 

Once we got active users, we proceeded to calculate the monthly revenue provided by them as follows:

1) Set revenue = monthly pay if user_active = 1, or to 0 otherwise.

2) Subtract the free package limit from the total number of calls, text messages, and data; multiply the result by the calling plan value; add the monthly charge depending on the calling plan.

3) Separate active users in a new dataframe.

**4) Study user behavior**

Considering monthly calls duration, messages sent and GB consumed as a whole, the insight we get from our analysis is that users have a very similar behavior regarding the three services hired in their plan:

Average calls duration, messages sent and GB consumed share very similar values and follow the same pattern for every user (lower in month one, but gradually and consistently increasing throughout the year).

The distributions of calls duration, messages and web sessions by user per month are also the same.

The median and standard deviation of each variable under study for both plans are close to one another.
    
Variables dispersion does not present major differences between plans.

Taking into account that in the 75% of the cases, users from both plans did not exceed the monthly allowance of messages and calls included in the cheapest package (surf), and that the maximum paid by surf users for extra messages was USD 6 and for extra calls was USD 30, we can conclude that the main decision driver when choosing a plan are the GB included on it.

With that regard, we see that based on the maximum gb consumed monthly by a surf user, the extra internet purchase added up to USD 500. Furthermore, the median for both plans falls slightly below the gb included in surf plan (13 gb vs. 15 gb). However, the maximum internet monthly consumption for an ultimate user was less than half of the quantity available (3000 GB).

**5) Revenue**

At an individual level, it's easy to see that ultimate users provide a much more stable and higher revenue in every monthly period.

However, since the total suscriptors to surf plan (2242) double those to ultimate plan (1071), the aggregate revenue attributable to the former surpasses the latter's in April, steeply increasing towards the year end. Also, revenue provided by surf users was higher due to the cost of extra gb, minutes and messages consumed.

If the company decided to launch a new marketing campaign, it should aim to promote the ultimate plan to get more suscriptions to it.

**6) Probability of a user consuming more than 19 GB (Normal distribution assumption) (at 20 gb, the revenue is the same than that of the ultimate plan, provided that the minutes and the messages limits aren't exceeded):**

If users were highly likely to consume 20 GB or more, it would be more profitable that they were suscribed to the surf plan than the ultimate. If such probability is high, then we would prefer future customers to suscribe to the surf plan and pay the extra GB at USD 10 each. To proceed with this analysis:

Based on the sample composed by all active users from both plans:

1) The hypothesis that the mean of the total users population is higher than 12 was rejected (One tailed hypothesis - alpha = 0.01)

2) The hypothesis that the mean of the total users population is less than 11 was rejected (One tailed hypothesis - alpha = 0.01)

3) We checked that we could not reject the hypotheisis that the mean of the total population is = 11.5 GB (Two tailed hypothesis - alpha = 0.01)

4) We created a normal distribution with mu = 11.5 (mean) and sigma = standard deviation of the sample).

5) We calculated the cumulative distribution function of 19 GB (at 20 gb we prefer them to be surf users, as we could charge extra messages and minutes), getting as a result that users have an 77% of probability of consuming less than 20 Gb and a 23% to reach or exceed such value.

6) On top of that, the 90% of the users are likely to consume less than 25 GB, that's why maybe suscribing to a plan including 30 GB is not worth for them.


This confirms the idea in the previous bullet point: it's less risky and more profitable to get customers suscribed to the ultimate plan.

7) Since the 55% of the users are likely to use more than 10 gb monthly,one idea to get surf users upgraded to ultimate is lowering the gb included to 10 gb, increasing the cost of extra gb to USD 15, and offering them to switch plans if in a given month they reached or exceeded the USD 70 (because it's cheaper that month).


**6) Plan to upgrade surf users to ultimate**

Next, we studied how the revenue would have varied if the GB included by ultimate were raised to 35 GB, and the ones in surf plan were reduced to 10 GB, plus extra GB consumed being charged with a USD 15 fee instead of USD 10.

We performed the analysis on the supposition that every user who ever reached or exceeded the USD 69 monthly pay in any month was given the possibility to upgrade starting in the month of the excess, and the offer was accepted because the monthly pay resulted cheaper (USD 20 + 4GBx USD15= USD 80 > USD 70).


Since 280 surf users would have exceeded the USD 69 monthly pay under the new plans features, we assumed that they accepted the offer to upgrade to the ultimate plan.


Even though we could not reject the hypothesis that the mean monthly revenue per user before and after the change was the same (by applying the method the Equality of the Means of Paired Samples), it's clear that this change had a positive impact, since 280 surf users were upgraded to ultimate (many of them in the last months of the year, what is expected to improve performance the following year).



**7) Test statistical hypothesis**

***Hypothesis one: the average revenue from users of the Ultimate and Surf calling plans differs***

Null hypothesis: "The average monthly revenue from users of ultimate plan is equal to the one from users of Surf plan"

Statistical test: Hypothesis on the Equality of Two Population Mean.
        
alpha= 0.05

result: We reject the null hypothesis. The average revenue from both plans differ.
    
***Hypothesis two: the average revenue from users in the NY-NJ area is different from that of the users from the other regions***

Null hypothesis: "The average revenue from users in the NY-NJ area is equal to that of the users from the other regions"

Statistical test: Hypothesis on the Equality of Two Population Mean.

alpha=0.05

Result: We reject the null hypothesis. The average revenue of users from NYNJ differs from the rest of the cities


