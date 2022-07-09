# Book-data-scrapper
This code is for scrapping web data of amazons best seller page
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import re
import time
from datetime import datetime
import matplotlib.dates as mdates
import matplotlib.ticker as ticker
from urllib.request import urlopen
from bs4 import BeautifulSoup
import requests

no_pages = 2

def get_data(pageNo):  
    headers = {"User-Agent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:66.0) Gecko/20100101 Firefox/66.0", "Accept-Encoding":"gzip, deflate", "Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8", "DNT":"1","Connection":"close", "Upgrade-Insecure-Requests":"1"}

    r = requests.get('https://www.amazon.in/gp/bestsellers/books/ref=zg_bs_pg_'+str(pageNo)+'?ie=UTF8&pg='+str(pageNo), headers=headers)#, proxies=proxies)
    content = r.content
    soup = BeautifulSoup(content,features="html.parser")
    #print(soup)

    alls = []
    for d in soup.findAll('div', attrs={'class':'zg-grid-general-faceout'}):
        #print(d)
        name = d.find('div', attrs={'class':'a-section a-spacing-mini _cDEzb_noop_3Xbw5'})
        n = name.find_all('img', alt=True)
        # print(n[0]['alt'])
        a = d.find('div', attrs={'class':'a-row a-size-small'})
        author = a.find('div', attrs={'class':'_cDEzb_p13n-sc-css-line-clamp-1_1Fn1y'})
        rating = d.find('span', attrs={'class':'a-icon-alt'})
        a = d.find('div', attrs={'class':'a-row'})
        users_rated = d.find('span', attrs={'class':'a-size-small'})
        price = d.find('span', attrs={'class':'_cDEzb_p13n-sc-price_3mJ9Z'})

        all1=[]

        if name is not None:
            #print(n[0]['alt'])
            all1.append(n[0]['alt'])
            # print(name.text)
        else:
            all1.append("unknown-product")

        if author is not None:
            # print(author.text)
            all1.append(author.text)
        elif author is None:
            author = d.find('span', attrs={'class':'a-size-small a-color-base'})
            if author is not None:
                all1.append(author.text)
            else:    
                all1.append('0')

        if rating is not None:
            # print(rating.text)
            all1.append(rating.text)
        else:
            all1.append('-1')

        if users_rated is not None:
            # print(users_rated.text)
            all1.append(users_rated.text)
        else:
            all1.append('0')     

        if price is not None:
            # print(price.text)
            all1.append(price.text)
        else:
            all1.append('0')
        alls.append(all1)    
    return alls

results = []
for i in range(1, no_pages+1):
    results.append(get_data(i))
flatten = lambda l: [item for sublist in l for item in sublist]
df = pd.DataFrame(flatten(results),columns=['Book Name','Author','Rating','Customers_Rated', 'Price'])
df.to_csv('amazon_products.csv', index=False, encoding='utf-8')
print("done")
df = pd.read_csv("amazon_products.csv")

df.shape

#modification of rating data

df['Rating'] = df['Rating'].apply(lambda x: x.split()[0])
df['Rating'] = pd.to_numeric(df['Rating'])

#modification of price data
df["Price"] = df["Price"].str.replace('₹', '')
df["Price"] = df["Price"].str.replace(',', '')
df['Price'] = df['Price'].apply(lambda x: x.split('.')[0])
df['Price'] = df['Price'].astype(int)

#modification of  data Customers_Rated
df["Customers_Rated"] = df["Customers_Rated"].str.replace(',', '')
df['Customers_Rated'] = pd.to_numeric(df['Customers_Rated'], errors='ignore')


df.replace(str(0), np.nan, inplace=True)
df.replace(0, np.nan, inplace=True)
df = df.dropna()

print("sorting according to price in ascending order")
data = df.sort_values(["Price"], axis=0, ascending=True )[:20]
data
