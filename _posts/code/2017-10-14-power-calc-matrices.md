---
layout: post
title: Generating power calculation matrices in Python
author: joanne_yeh
excerpt: "Jupyter Notebook Tutorial"
categories: code
comments: true
share: true
---


# Need a quick way to see how your minimum detectable effect change your sample size? 
Plus in your relevant information and instantly get a matrix of sample sizes you need for various effect sizes


### Are RetailerName shoppers in GA fundamentally different from RetailerName shoppers in the 4 other states (SC, VA, TN, MD)?

Date: September 21th, 2017 / Joanne Yeh


# Query and Data In Use
**Here is the massive pull I used to get total_august.csv**

# Environment Setup
**Import and set up data file**

Should return a dataframe called dat_spend which has all the relevant columns.

~~~python
    # MODIFY THIS: Write new name of data file here

    data_file_to_use = 'total_august.csv'

    #georgia_file = 'georgia_august.csv'
~~~

~~~python
    import pandas as pd
    import numpy as np
    from scipy import stats

    import matplotlib.pyplot as plt
    from matplotlib import figure
    import seaborn as sns
    from __future__ import division

    import statsmodels.stats.api as sms

    %matplotlib inline
    %config InlineBackend.figure_formats=['svg']

~~~

Create function to do individual comparisons with less writing


~~~python
def compare_states(state1, state2, metric):
    
    tstat = 0
    p_val = 0
    
    if metric is 'num_visits':
        tstat, p_val = stats.ttest_ind(fl_shoppers[(fl_shoppers.retailer == 'FOOD LION') & (fl_shoppers.state == state1)].groupby('uid').uid.count().values, \
                fl_shoppers[(dat_spend.retailer == 'FOOD LION') & (fl_shoppers.state == state2)].groupby('uid').uid.count().values)
        
        print "Mean of ", state1, "is", dat_spend[(dat_spend.retailer == 'FOOD LION') & (dat_spend.state == state1)].groupby('uid').uid.count().mean() 
        print "Mean of ", state2, "is", dat_spend[(dat_spend.retailer == 'FOOD LION') & (dat_spend.state == state2)].groupby('uid').uid.count().mean() 
    
    elif metric is 'total_spend_fl':
        tstat, p_val = stats.ttest_ind(fl_shoppers[(fl_shoppers.retailer == 'FOOD LION') & (fl_shoppers.state == state1)].groupby('uid').amount_cents.sum().values, \
                fl_shoppers[(fl_shoppers.retailer == 'FOOD LION') & (fl_shoppers.state == state2)].groupby('uid').amount_cents.sum().values)

        print "Mean of ", state1, "is", fl_shoppers[(fl_shoppers.retailer == 'FOOD LION') & (fl_shoppers.state == state1)].groupby('uid').amount_cents.sum().mean()
        print "Mean of ", state2, "is", fl_shoppers[(fl_shoppers.retailer == 'FOOD LION') & (fl_shoppers.state == state2)].groupby('uid').amount_cents.sum().mean()
    
    elif metric is 'total_spend_all':
        tstat, p_val = stats.ttest_ind(fl_shoppers[fl_shoppers.state == state1].groupby('uid').amount_cents.sum().values, \
                fl_shoppers[fl_shoppers.state == state2].groupby('uid').amount_cents.sum().values)
        
        print "Mean of ", state1, "is", fl_shoppers[fl_shoppers.state == state1].groupby('uid').amount_cents.sum().mean()
        print "Mean of ", state2, "is", fl_shoppers[fl_shoppers.state == state2].groupby('uid').amount_cents.sum().mean()
    
    elif metric is 'small_dollar_pct':
        tstat, p_val = stats.ttest_ind(percentage_small_transactions[percentage_small_transactions.state == state1].pct.values, \
                percentage_small_transactions[percentage_small_transactions.state == state2].pct.values)
        
        print "Mean of ", state1, "is", percentage_small_transactions[percentage_small_transactions.state == state1].pct.mean()
        print "Mean of ", state2, "is", percentage_small_transactions[percentage_small_transactions.state == state2].pct.values.mean()

    else:
        print "error, metric not correct. Use num_vists, total_spend_fl, total_spend_all, or small_dollar_pct"
    
    if p_val <= 0.05:
        print "Statistically significant, pval:", p_val
    else:
        print "Not statistically significant, pval:", p_val 
        
    return    
~~~

Run the blocks below to clean and create necessary dataframes


~~~python
dat = pd.read_csv(data_file_to_use, sep = ';')
dat.columns = ['uid', 'tid', 'date', 'retailer', 'amount_cents', 'transaction_type', 'state']

# clean the retailer names
dat.retailer = map(lambda x: x.decode('utf-8').upper(), dat.retailer)
dat.retailer = dat.retailer.replace(to_replace=u"\xa0", value=" ", regex=True)
dat.retailer = dat.retailer.str.strip()
dat.retailer = dat.retailer.replace(to_replace='-|#|\.', value=' ', regex=True)
dat.retailer = dat.retailer.replace(to_replace='[0-9]*$', value='', regex=True)
dat.retailer = dat.retailer.str.strip()
dat.retailer = dat.retailer.replace(to_replace='WAL MART', value ='WALMART', regex=True)
dat.retailer = dat.retailer.replace(to_replace='WALMART SUPER CENTER', value='WM SUPERCENTER', regex=True)

# exclude food benefit authorization from calculations and blank retailer 
# names because usually adminstrative type fees
dat_spend = dat[~dat['retailer'].isin(['FOOD  BENEFIT AUTHORIZATION',''])]

# remove all transactions which are not spending
dat_spend = dat_spend[dat_spend.amount_cents < 0]
~~~


~~~python
# Get all transactions from users who have shopped at Food Lion

fl_shoppers = dat_spend[dat_spend['uid'].isin(dat_spend[dat_spend.retailer == 'FOOD LION'].uid.unique())]
~~~


~~~python
# pre-filter some dataframes for use in the apply portion later

small_transactions = fl_shoppers[fl_shoppers.amount_cents > -600]
food_lion_transactions = fl_shoppers[fl_shoppers.retailer == 'FOOD LION']


# create dataframe that has counts for total transactions, small dollar where each row is one person

percentage_small_transactions = pd.DataFrame(fl_shoppers.uid.unique(), columns = ['fl_shopper_id'])

percentage_small_transactions['total_transactions'] = percentage_small_transactions['fl_shopper_id'].apply\
(lambda x: fl_shoppers[fl_shoppers.uid == x].tid.count())

percentage_small_transactions['small_dollar_transctions'] = percentage_small_transactions['fl_shopper_id'].apply\
(lambda x: small_transactions[small_transactions.uid == x].tid.count())


# calculate percentage based off these two values and place in same column
percentage_small_transactions['pct'] = percentage_small_transactions['small_dollar_transctions']/percentage_small_transactions['total_transactions'] 


# get first entry of series in column 6, to get user's state

percentage_small_transactions['state'] = percentage_small_transactions['fl_shopper_id'].apply\
(lambda x: fl_shoppers[fl_shoppers.uid == x].iloc[0,6])


~~~

Preview the first 5 lines of the dataframes

Dataframe of all spending transactions from those states during that time period

Dataframe of all spending transactions with people who have shopped at Food Lion at least once during this time period

Dataframe that contains percentage of small dollar transactions by Food Lion shopper

# Analysis Section

### Main question to answer: Are RetailerName shoppers in GA fundamentally different from RetailerName shoppers in the 4 other states (SC, VA, TN, MD)?

In a period of 30 days in September. // **August 1 to before Sept 1**
* Total number of times each person went to Food Lion
* Total spend per person at Food Lion
* Total spend per person
* Percentage of transactions under \$6

Added:
* Average spend per trip

## Between GA and SC


~~~python
compare_states('TN', 'SC', 'num_visits')
~~~

    /usr/local/lib/python2.7/site-packages/ipykernel_launcher.py:7: UserWarning: Boolean Series key will be reindexed to match DataFrame index.
      import sys


    Mean of  TN is 2.27788461538
    Mean of  SC is 3.05244483867
    Statistically significant, pval: 3.09797045076e-16



~~~python
compare_states('TN', 'SC', 'total_spend_fl')
~~~

    Mean of  TN is -9356.99230769
    Mean of  SC is -11349.7171527
    Statistically significant, pval: 2.63088834821e-07



~~~python
compare_states('GA', 'SC', 'total_spend_all')
~~~

    Mean of  GA is -46167.4709302
    Mean of  SC is -42576.192926
    Statistically significant, pval: 0.000103216028067



~~~python
compare_states('TN', 'SC', 'small_dollar_pct')
~~~

    Mean of  TN is 0.25880529809
    Mean of  SC is 0.242008259508
    Statistically significant, pval: 0.00635070323556


## Number of Visits
How do RetailerName shoppers in Georgia compare to RetailerMa,e shoppers in the four other states, regarding the total number of times each person visited Retailer Name? 


~~~python
# takes all RetailerName transactions, sums visits for each person, then takes average

dat_spend[dat_spend.retailer == 'FOOD LION'].groupby(['state','uid']).uid.count().mean(level = 0)
~~~




    state
    GA    2.475291
    MD    2.695441
    SC    3.052445
    TN    2.277885
    VA    3.493663
    Name: uid, dtype: float64



We can run a t-test on this:

## Total Spend Per Person at Food Lion
Of those who have shopped at Food Lion during this time period, total spend at FL during that time period, by person


## Total Spend Per Person (All Transactions)
Of those who have shopped at Food Lion during this time period, total spend during that time period, by person


~~~
python
# takes all Food Lion transactions, sums visits for each person, then takes average

fl_shoppers.groupby(['state','uid']).amount_cents.sum().mean(level = 0)
~~~




    state
    GA   -46167.470930
    MD   -46530.272551
    SC   -42576.192926
    TN   -43508.835577
    VA   -41474.117455
    Name: amount_cents, dtype: float64



## Percentage of Transactions under \$6

How does the percentage of transactions under \$6 compare across Food Lion shoppers between these states?


~~~python
percentage_small_transactions.groupby('state').pct.mean().mean(level = 0)
~~~




    state
    GA    0.255696
    MD    0.245640
    SC    0.242008
    TN    0.258805
    VA    0.294904
    Name: pct, dtype: float64



Run t-test if desired
