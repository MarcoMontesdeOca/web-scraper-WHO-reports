# web-scraper-WHO-reports
Simple python script designed to extract data from pdf situation reports from the WHO web. These reports include data such as the number of  confirmed cases with CoVid-19 per country.
```
import io
import requests
import PyPDF2 
import pandas as pd

def get_and_open(url):
    """
    Takes url as input, access and open a pdf from url.
    Returns a PyPDF2.pdf.PdfFileReader object.
    """
    request = requests.get(url)
    file = io.BytesIO(request.content)    
    pdfReader = PyPDF2.PdfFileReader(file)
    return pdfReader

def get_text(pdfReader, page):
    """
    Takes a PyPDF2.pdf.PdfFileReader object as input,
    returns text in a given page as string.
    """
    pageObj = pdfReader.getPage(page)
    page = pageObj.extractText()
    return page

def clean(page):
    """
    Takes raw page text as string,
    returns cleaner text as string.
    """
    page = page.replace('\n', ',')
    page = page.replace(', ', '')
    page = page.replace(' ,', ' ')
    return page

def find_cases(page, country):
    """
    Takes as input the text of a given page and a country,
    both as strings.
    Returns int number of cases from a country of interest.
    """
    index_start = page.find(country)
    if index_start == -1:
        return -1
    else:
        index_start = index_start + len(country) + 1
        index_temp = index_start
        index_end = index_start
        try:
            int(page[index_start])        
            while True:
                try:
                    int(page[index_temp + 1])
                    index_end = index_temp + 1
                    index_temp += 1
                except ValueError:
                    break
            cases = page[index_start:index_end + 1]
            return int(cases)
        except ValueError:
            return -1
```
This is an example of how to define countries of interest in order to extract their data from the report number 30 (19 February) up to the one released on 6 April.

```
# Define static elements of URLs of interest
str1 = 'https://www.who.int/docs/default-source/coronaviruse/situation-reports/'
str2 = '-sitrep-'
str3 = '-covid-19.pdf'



# Find cases, store in dataframe
total_cases = pd.DataFrame({'Republic of Korea':[], 'Singapore':[], 'Australia':[],
                      'Italy':[], 'Spain':[], 'United States of America':[],
                      'Chile':[], 'Ecuador':[], 'Paraguay':[]})

# Dates used in reports' urls
feb = list(range(20200219,20200230))
mar = list(range(20200301,20200332))
apr = list(range(20200401,20200407))
dates = feb + mar + apr

id2 = 30 #id2 from first report used

for d in range(len(dates)):#sequence of reports 
    id1 = dates[d]
    if id1 == 20200403:# this report is the only one with a different
                         # url pattern
       url = str1 + str(id1) + str2 + str(id2) + '-covid-19-mp.pdf'
    else:
        url = str1 + str(id1) + str2 + str(id2) + str3
    print(f'inside {id2} | {id1}')
    print(f'url: {url}')
    temp_dict = {'Republic of Korea':[], 'Singapore':[], 'Australia':[],
                          'Italy':[], 'Spain':[], 'United States of America':[],
                          'Chile':[], 'Ecuador':[], 'Paraguay':[], 'Day':[]}
    temp_dict['Day'].append(d)
    reader_obj = get_and_open(url)
    for i in range(1,20):#page "0" must be avoided
        try:
            page = get_text(reader_obj, i)
            page = clean(page)
            print("######################## NEW PAGE ###################")
            #print(page)
            for j in range(len(list(total_cases))):
                country = list(total_cases)[j]
                #print(country)
                try:
                    cases = find_cases(page, country)
                    if cases == -1:
                        continue
                    else:
                        #print(cases)
                        temp_dict[country].append(cases)
                except AttributeError:
                    continue
        except IndexError:
            break
    
    to_remove = []
    for key in temp_dict:
        if temp_dict[key] == []:
            to_remove.append(key)
    for i in range(len(to_remove)):
        key = to_remove[i]
        del temp_dict[key]
        
    temp_df = pd.DataFrame.from_dict(temp_dict)
    total_cases = total_cases.append(temp_df)    
    id2 = id2 + 1

total_cases = total_cases.set_index('Day')

# Visualization
import matplotlib.pyplot as plt
import os
cwd = os.getcwd() #find current working directory
os.chdir('/home/marco/Desktop/CoV')

plt.close('all')
plt.figure(figsize=(8,4.5))
plt.plot(total_cases)
plt.ylabel('Confirmed cases with CoVid-19', fontsize=14)
plt.xlabel('Days from 19 Feb', fontsize=14)
fig = plt.gcf()
fig.savefig('Covid19_countries.png', dpi = 100)
plt.close()
```
Finally, we obtain a plot like this one
![Covid19_countries](https://github.com/MarcoMontesdeOca/web-scraper-WHO-reports/blob/master/Covid19_countries.png)

