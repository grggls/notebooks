# Exchange Rate Conversion

## Problem Definition

## Solution Contents
---
```shell
> cd <this directory>
> ls -l
-rw-r--r--   1 gregory.damiani  admins      171 Feb 26 18:57 Pipfile
-rw-r--r--   1 gregory.damiani  admins    19493 Feb 26 18:57 Pipfile.lock
-rw-r--r--   1 gregory.damiani  admins     1186 Mar  2 16:00 README.md
-rw-r--r--   1 gregory.damiani  admins   506680 Mar  2 16:01 exchangerate.ipynb
-rw-r--r--@  1 gregory.damiani  admins  4442679 Mar  2 14:05 transactions.csv
-rw-r--r--   1 gregory.damiani  admins  4447404 Mar  2 16:01 transactions_amended.csv
```
---
 
 - `Pipfile` - explained in the section below
 - `Pipfile.lock` - compiled when we use the `Pipfile`
 - `README.md` - you're reading it
 - `exchangerate.ipynb` - an IPython notebook containing code solving the problem described above 
 - `transactions.csv` - distributed with the problem definition
 - `transactions_amended.csv` - *should be created by copying `transactions.csv` before running the notebook.*   Contains the data of the solution. 

## How To Use this IPython Notebook

I built this notebook in Jupyter Lab. There's one out-of-band dependency: [`forex-python`](https://pypi.org/project/forex-python/)([documentation](http://forex-python.readthedocs.io/en/latest/usage.html), [github](https://github.com/MicroPyramid/forex-python).

This dependency is listed in the Pipfile. I recommend you use `pipenv`:

---
```shell
> cd <this directory>
> pipenv install
> pipenv shell
> jupyter lab
```
---

`forex-python` uses https://ratesapi.io - a free API for current and historical foreign exchange rates published by European Central Bank. The rates are updated daily 3PM CET. No rate limiting is in place for this API. Users are encouraged to cache results to avoid excessive API calls.

## Notes On The Solution

We're using the transactions_amended.csv as a cache and a restore point. All fetched and generated values are stored here. There are two methods for loading and saving the data to/from disk:

---
```
load_transactions_csv('transactions_amended.csv')
save_transactions_csv('transactions_amended.csv')
```
---

By far, the most costly operation in this solution is the call to the forex API. Several optimizations were made around this.

1. Loop through the transactions and check if they've got an `amount` field > 0.0. If not, simply remove the transaction from the list and do no more work.

1. Pre-process and set the 'currency' field in each transaction to the ISO standard currency codes like EUR, GBP, and USD. 

1. Sort the list of transactions by transaction_date

1. Once an `exchange_rate` was fetched from the API for a `(currency, transaction_date)` pair, we looked forward in the transaction list to see if any other transactions matched and took the opportunity to update their `exchange_rate` field. We obviated the need for a large number of expensive API calls at the cost of a nested for-loop.

## Notes on Improvements and Moving This Into Production

To prevent repeated fetching of the same currency conversion rates from the external API, these values should be cached in a database like redis. Future 