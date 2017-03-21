# Workaround For Part 22 of Sentdex's Python for ML & Investing Tuotrials
---

Hi there,

So, as you probably experienced for yourselves when running through the last few minutes of [Part 22](https://www.youtube.com/watch?v=5igqQ5V--tE&index=22&list=PLQVvvaa0QuDd0flgGphKCej-9jp-QdzZ3) of the great [Machine Learning for Investing With Python](https://www.youtube.com/playlist?list=PLQVvvaa0QuDd0flgGphKCej-9jp-QdzZ3) series, the Yahoo! Finance parsing seems to spit out a blank CSV file (specifically the *NO_NA* version).

This is due to a change (which, as it turns out, [happens quite often at Yahoo!](https://www.elitetrader.com/et/threads/old-yahoo-finance-page-any-way-to-access-it.301296/#post-4306183)) to the "Key Statistics" page. 

When you inspect a certain element (Say, Enterprise Value/EVITDA) with FireBug (Firefox) /Developer Tools (Chrome) you can clearly see the value in the inspector's panel.
However, when right-clicking and then pressing "View Source", that number is gone.

Fortunately for us, there is a solution: Yahoo! Finance has a JSON (and also CSV, but we're gonna stick with JSON for now) API that lets us get all the data we need (and so much more ....).

Alas, this requires a few changes to **21.py** (the file we use to gather the data) and **22.py** (the file we use to create the Dataset). 

First of all, 21.py. 

Notice that I've changed the URL to fit the API's one, and added a few so-called "modules". These are basically names for different subsets of company data.

[I've added all of the ones I know about](http://stackoverflow.com/questions/38567661/how-to-get-key-statistics-for-yahoo-finance-web-search-api), just to make sure I get as much data as possible with each API call (to avoid potentially going over the Yahoo! Finance limit - although we're way below the [the 2,000 requests/hour rate](http://stackoverflow.com/questions/9346582/what-is-the-query-limit-on-yahoos-finance-api)).

## 21.py

```python
	import urllib.request
	import os
	import time

	path = "MY/PATH/TO/THE/TUTORIAL/FOLDER"

	def Check_Yahoo():
	    statspath = path+'/_KeyStats'
	    stock_list = [x[0] for x in os.walk(statspath)]
	    
	    ## Added a counter to call out how many files we've already added
	    counter = 0
	    for e in stock_list[1:]:
	
		try:
		    e = e.replace("MY/PATH/TO/THE/TUTORIAL/FOLDER/_KeyStats","")
		    ## Changed the URL & added the modules
		    link = "https://query2.finance.yahoo.com/v10/finance/quoteSummary/"+e.upper()+"?modules=assetProfile,financialData,defaultKeyStatistics,calendarEvents,incomeStatementHistory,cashflowStatementHistory,balanceSheetHistory"
		    resp = urllib.request.urlopen(link).read()
		    ## We go by Bond. JSON Bond
		    save = "forward_json/"+str(e)+".json"
		    store = open(save,"w")
		    store.write(str(resp))
		    store.close()
		    ## Print some stuff while working. Communication is key
		    counter +=1
		    print("Stored "+ e +".json")
		    print("We now have "+str(counter)+" JSON files in the directory.")
		    
		    
		except Exception as e:
		    print(str(e))
		    time.sleep(2)
	Check_Yahoo()
```	
	
Moving over to 22.py. Remember that JSON is text - just like HTML. 

The main changes here have to do with the names for each variable - I've added the new names to the `gather` variable inside the `Forward()` function.

Also, the regex has to change slightly. The resulting code looks like this:

*Note: I could just parse the JSON and forget the regex, but for the sake of continuity I'm going to continue using Harrison's original method.*

## 22.py

```python
	import pandas as pd
	import os
	import time
	import re

	def Forward(gather=['debtToEquity',
		                'Trailing P/E', ## Couldn't find correlating value
		                'Price/Sales', ## Couldn't find correlating value
		                'priceToBook',
		                'profitMargins',
		                'operatingMargins',
		                'returnOnAssets',
		                'returnOnEquity',
		                'revenuePerShare',
		                'Market Cap', ## Couldn't find correlating value
		                'enterpriseValue',
		                'forwardPE',
		                'pegRatio',
		                'enterpriseToRevenue',
		                'enterpriseToEbitda',
		                'totalRevenue',
		                'grossProfit',
		                'ebitda',
		                'netIncomeToCommon',
		                'trailingEps',
		                'earningsGrowth',
		                'revenueGrowth',
		                'totalCash',
		                'totalCashPerShare',
		                'totalDebt',
		                'currentRatio',
		                'bookValue',
		                'operatingCashflow',
		                'beta',
		                'heldPercentInsiders',
		                'heldPercentInstitutions',
		                'sharesShort',
		                'shortRatio',
		                'shortPercentOfFloat',
		                'sharesShortPriorMonth']):


	    df = pd.DataFrame(columns = ['Date',
		                         'Unix',
		                         'Ticker',
		                         'Price',
		                         'stock_p_change',
		                         'SP500',
		                         'sp500_p_change',
		                         'Difference',
		                         ##############
		                         'DE Ratio',
		                         'Trailing P/E',
		                         'Price/Sales',
		                         'Price/Book',
		                         'Profit Margin',
		                         'Operating Margin',
		                         'Return on Assets',
		                         'Return on Equity',
		                         'Revenue Per Share',
		                         'Market Cap',
		                         'Enterprise Value',
		                         'Forward P/E',
		                         'PEG Ratio',
		                         'Enterprise Value/Revenue',
		                         'Enterprise Value/EBITDA',
		                         'Revenue',
		                         'Gross Profit',
		                         'EBITDA',
		                         'Net Income Avl to Common ',
		                         'Diluted EPS',
		                         'Earnings Growth',
		                         'Revenue Growth',
		                         'Total Cash',
		                         'Total Cash Per Share',
		                         'Total Debt',
		                         'Current Ratio',
		                         'Book Value Per Share',
		                         'Cash Flow',
		                         'Beta',
		                         'Held by Insiders',
		                         'Held by Institutions',
		                         'Shares Short (as of',
		                         'Short Ratio',
		                         'Short % of Float',
		                         'Shares Short (prior ',                                
		                         ##############
		                         'Status'])

	    ## Change to JSON Folder   
	    file_list = os.listdir("forward_json")    

	    ## Split file before JSON
	    for each_file in file_list:
		ticker = each_file.split(".json"[0])
		
		
		## Change to JSON folder
		full_file_path = "forward_json/"+each_file
		source = open(full_file_path, "r").read()

	       
		try:
		    value_list = []

		    for each_data in gather:
		        try:    
		            
		            regex = re.escape(each_data) + r'.*?"(\d{1,8}\.\d{1,8}M?B?K?|N/A)%?'
		            value = re.search(regex, source)
		            value = value.group(1)
		            
		            if "B" in value:
		                value = float(value.replace("B",'')) * 1000000000
		            
		            elif "M" in value:
		                value = float(value.replace("M",'')) * 1000000
		            
		            elif "K" in value:
		                value = float(value.replace("K",'')) * 1000
		                
		            value_list.append(value)
		            
		        except Exception as e:
		            value = "N/A"
		            value_list.append(value)
		            
	       
		    if value_list.count("N/A") > 15:
		        pass
		    
		    else:
		    
		        df = df.append({'Date':"N/A",
		                        'Unix':"N/A",
		                        'Ticker':ticker[0], ## Getting Only The Stock Name, not 'json'
		                        'Price':"N/A",
		                        'stock_p_change':"N/A",
		                        'SP500':"N/A",
		                        'sp500_p_change':"N/A",
		                        'Difference':"N/A",
		                        'DE Ratio':value_list[0],
		                        #'Market Cap':value_list[1],
		                        'Trailing P/E':value_list[1],
		                        'Price/Sales':value_list[2],
		                        'Price/Book':value_list[3],
		                        'Profit Margin':value_list[4],
		                        'Operating Margin':value_list[5],
		                        'Return on Assets':value_list[6],
		                        'Return on Equity':value_list[7],
		                        'Revenue Per Share':value_list[8],
		                        'Market Cap':value_list[9],
		                        'Enterprise Value':value_list[10],
		                        'Forward P/E':value_list[11],
		                        'PEG Ratio':value_list[12],
		                        'Enterprise Value/Revenue':value_list[13],
		                        'Enterprise Value/EBITDA':value_list[14],
		                        'Revenue':value_list[15],
		                        'Gross Profit':value_list[16],
		                        'EBITDA':value_list[17],
		                        'Net Income Avl to Common ':value_list[18],
		                        'Diluted EPS':value_list[19],
		                        'Earnings Growth':value_list[20],
		                        'Revenue Growth':value_list[21],
		                        'Total Cash':value_list[22],
		                        'Total Cash Per Share':value_list[23],
		                        'Total Debt':value_list[24],
		                        'Current Ratio':value_list[25],
		                        'Book Value Per Share':value_list[26],
		                        'Cash Flow':value_list[27],
		                        'Beta':value_list[28],
		                        'Held by Insiders':value_list[29],
		                        'Held by Institutions':value_list[30],
		                        'Shares Short (as of':value_list[31],
		                        'Short Ratio':value_list[32],
		                        'Short % of Float':value_list[33],
		                        'Shares Short (prior ':value_list[34],
		                        'Status':"N/A"}, ignore_index = True)
		except Exception as e:
		    pass

	    df.to_csv("forward_sample_WITH_NA.csv")       

	Forward()
```	
	
Note that this leaves us with the ability to **only create the *WITH_NA* file**.  The **NO_NA** file will still remain blank because we will **always** have at least 3 blanks - the ones I couldn't match. I can play around with the numbers to make it work, but this kind of beats the purpose - it will no longer be a file without any N/As.

Hope this helps:)

-T

