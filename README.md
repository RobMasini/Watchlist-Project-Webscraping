# Watchlist-Project
import pandas_datareader.data as web
import datetime
import matplotlib.pyplot as plt
import numpy as np
from bs4 import BeautifulSoup
from urllib.request import urlopen
import requests


def program():

    menuchoice = input('Stock List(1), Stock Analyzer(2), Specific Stock Search(3): ')
    stocklist = []
    if menuchoice == '1':
        print(stocklist)
        stocklistadd = input('Enter a stock ticker to add to your watchlist: ')
        stocklist.append(stocklistadd)
        print(stocklist)
        program()

    elif menuchoice == '2':
        
        analyze = input('Top Gainers(1), Top Losers(2), Most Active(3), Most volatile(4): ')

        if analyze == '1':
            url_tvgainers = "https://www.tradingview.com/markets/stocks-usa/market-movers-gainers/"
            page_tvgainers = urlopen(url_tvgainers)
            html_bytes = page_tvgainers.read()
            html = html_bytes.decode('utf-8')
            urlcontent_tvgainers = requests.get(url_tvgainers)
            data = urlcontent_tvgainers.text
            soup_tvgainers = BeautifulSoup(urlcontent_tvgainers.content, 'lxml')
            #print(soup_tvgainers.prettify())
            #results_tvgainers = soup_tvgainers.find(class_='tv-data-table__cell tv-screener-table__cell tv-screener-table__cell--up tv-screener-table__cell--big tv-screener-table__cell--with-marker')
            #print(results_tvgainers)
            print(soup_tvgainers.title.text)
            table_tvgainers = soup_tvgainers.find("div", attrs={"id": "js-category-content"})
            row_tvgainers = table_tvgainers.tbody.find_all("tr")
            data = {
                'name' : [],
                'close' : [],
                'change' : [],
            }
            for row in row_tvgainers:
                td_tvgainers = row.find_all("td")
                data['name'].append(td_tvgainers[0].find('name',)
                data['close'].append(td_tvgainers[1].get_text())
                data['change'].append(td_tvgainers[2].get_text())
            #gainers = []
            #for td in row_tvgainers[0].find_all("td"):
            #    gainers.append("data-symbol")
            #print(str(row_tvgainers))

        elif analyze == '2':
            url_tvlosers = "https://www.tradingview.com/markets/stocks-usa/market-movers-losers/"
            page_tvlosers = urlopen(url_tvlosers)
            html_bytes = page_tvlosers.read()
            html = html_bytes.decode('utf-8')
            urlcontent_tvlosers = requests.get(url_tvlosers)
            soup_tvlosers = BeautifulSoup(urlcontent_tvlosers.content, 'html5lib')
            print(soup_tvlosers.prettify())

        elif analyze == '3':
            url_tvactive = "https://www.tradingview.com/markets/stocks-usa/market-movers-active/"
            page_tvactive = urlopen(url_tvactive)
            html_bytes = page_tvactive.read()
            html = html_bytes.decode('utf-8')
            urlcontent_tvactive = requests.get(url_tvactive)
            soup_tvactive = BeautifulSoup(urlcontent_tvactive.content, 'html5lib')
            print(soup_tvactive.prettify())

        elif analyze == '4':
            url_tvvolatile = "https://www.tradingview.com/markets/stocks-usa/market-movers-most-volatile/"
            page_tvvolatile = urlopen(url_tvvolatile)
            html_bytes = page_tvvolatile.read()
            html = html_bytes.decode('utf-8')
            urlcontent_tvvolatile = requests.get(url_tvvolatile)
            soup_tvvolatile = BeautifulSoup(urlcontent_tvvolatile.content, 'html5lib')
            print(soup_tvvolatile.prettify())

        else:
            print('Error: Enter 1, 2, 3, or 4')

    elif menuchoice == '3':
        search_ticker = input(str('Search Ticker: '))
        year = input(str('From Year: '))
        day = input(str('From Month: '))
        month = input(str('From Day: '))
        start = datetime.datetime(year,day,month)
        from datetime import date
        end = date.today()
        search_stock = web.DataReader(search_ticker,'yahoo',start,end)
        search_stock.to_csv('Search_Stock.csv')
        search_stock.head()
        search_stock['Open'].plot()

    else:
        print('Error: Enter 1, 2, or 3')

program()
