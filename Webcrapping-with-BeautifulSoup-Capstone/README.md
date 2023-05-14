# Capstone Webscrapping using BeautifulSoup

This notebook contains guidances & tasks on the data processing for the application
Coursebook: Python for Data Analysts

- Prepared by : Mohamad Gandhi Ardi Wibawanto
- Batch       : Vulcan DA Online

## Background

### US Dollar (USD) To Indonesian Rupiah (IDR) Exchange Rate History
- We will the US Dollar (USD) to Indonesian Rupiah (IDR) exchange rate history summary page with 180 days of USD to IDR rate 

## Requirements Library

- Install package requirements
    ```
    pip install -r requirements.txt
    ```

## Requesting the Data and Creating a BeautifulSoup

Let's begin with requesting the web from the site with `get` method.


```python
import requests

url_get = requests.get('https://www.exchange-rates.org/exchange-rate-history/usd-idr')
```

To visualize what exactly you get from the `request.get`, we can use .content to see what we exactly get, in here i slice it so it won't make our screen full of the html we get from the page. You can delete the slicing if you want to see what we fully get.


```python
url_get.content[1:500]
```




    b'!DOCTYPE html>\r\n<!--[if lt IE 9]>\r\n<html class="no-js ie8 oldie" lang="en" xml:lang=\'en\'>\r\n<![endif]-->\r\n<!--[if gt IE 8]><!--><html class="no-js" lang="en" xml:lang=\'en\'><!--<![endif]-->\r\n<head>\r\n<title>USD to IDR exchange rate history</title>\r\n<meta http-equiv="X-UA-Compatible" content="IE=edge">\r\n<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=5">\r\n<meta charset="utf-8" /><meta http-equiv="Content-Type" content="text/html; charset=utf-8" />\r\n<base href="http'



As we can see we get a very unstructured and complex html, which actually contains the codes needed to show the webpages on your web browser. But we as human still confused what and where we can use that piece of code, so here where we use the beautifulsoup. Beautiful soup class will result a beautifulsoup object. Beautiful Soup transforms a complex HTML document into a complex tree of Python objects. 

Let's make Beautiful soup object and feel free to explore the object here.


```python
from bs4 import BeautifulSoup 

soup = BeautifulSoup(url_get.content,"html.parser")
print(type(soup))
```

    <class 'bs4.BeautifulSoup'>
    


```python
print(soup.prettify()[:500])
```

    <!DOCTYPE html>
    <!--[if lt IE 9]>
    <html class="no-js ie8 oldie" lang="en" xml:lang='en'>
    <![endif]-->
    <!--[if gt IE 8]><!-->
    <html class="no-js" lang="en" xml:lang="en">
     <!--<![endif]-->
     <head>
      <title>
       USD to IDR exchange rate history
      </title>
      <meta content="IE=edge" http-equiv="X-UA-Compatible"/>
      <meta content="width=device-width, initial-scale=1, maximum-scale=5" name="viewport"/>
      <meta charset="utf-8">
       <meta content="text/html; charset=utf-8" http-equiv="Content-Type">
       
    

## Finding the right key to scrap the data & Extracting the right information

<img src="asset/image1.jpg">


In the HTML address used, it is identified that the exchange rate data information code key to be retrieved is in the `tbody` section. Then it is used in the `find()` function to get all the information contained in the HTML `tbody`. Then use `find.all()` to explore other data for the repeating keyword.


```python
table = soup.find('tbody')
print(table.prettify()[1:500])
```

    tbody>
     <tr>
      <td>
       <a class="w" href="/exchange-rate-history/usd-idr-2023-05-12">
        May 12, 2023
       </a>
       <a class="n" href="/exchange-rate-history/usd-idr-2023-05-12">
        2023-5-12
       </a>
      </td>
      <td>
       <span class="w">
        <span class="nowrap">
         <span class="currencySymbol">
          $
         </span>
         1 =
        </span>
        <span class="nowrap">
         <span class="currencySymbol">
          Rp
         </span>
         14,844
        </span>
       </span>
       <span class="n">
        <span class="nowrap"
    


```python
table.find_all('a', attrs={'class':'n'})[:5]
```




    [<a class="n" href="/exchange-rate-history/usd-idr-2023-05-12">2023-5-12</a>,
     <a class="n" href="/exchange-rate-history/usd-idr-2023-05-11">2023-5-11</a>,
     <a class="n" href="/exchange-rate-history/usd-idr-2023-05-10">2023-5-10</a>,
     <a class="n" href="/exchange-rate-history/usd-idr-2023-05-09">2023-5-9</a>,
     <a class="n" href="/exchange-rate-history/usd-idr-2023-05-08">2023-5-8</a>]




```python
table.find_all('a', attrs={'class':'n'})[3].text
```




    '2023-5-9'




```python
table.find_all('span', attrs={'class':'nowrap'})[:6]
```




    [<span class="nowrap"><span class="currencySymbol">$</span>1 =</span>,
     <span class="nowrap"><span class="currencySymbol">Rp</span>14,844</span>,
     <span class="nowrap"><span class="currencySymbol">$</span>1 =</span>,
     <span class="nowrap"><span class="currencySymbol">Rp</span>14,844</span>,
     <span class="nowrap"><span class="currencySymbol">$</span>1 =</span>,
     <span class="nowrap"><span class="currencySymbol">Rp</span>14,776</span>]




```python
table.find_all('span', attrs={'class':'nowrap'})[0].text
```




    '$1 ='




```python
table.find_all('span', attrs={'class':'nowrap'})[1].text
```




    'Rp14,844'




```python
row = table.find_all('a', attrs={'class':'n'})
row_length = len(row)
row_length
```




    130




```python
row_kurs = table.find_all('span', attrs={'class':'nowrap'})
row_length_kurs = len(row_kurs)
row_length_kurs
```




    520



In the scrapping process, a loop function is used to retrieve the necessary data according to the keywords used, namely ('a', attrs={'class':'n'}) for date data and ('span', attrs={'class' :'nowrap'}) for exchange rate data. As can be seen in the previous code, the information structure of the exchange rate data, we will only retrieve the rupiah exchange rate data every 4 data jumps (index : 1, 5, 9, 13, etc., up to data 516). Use the lopping function to retrieve the required data as follows:


```python
TransDate = [] #initiating a tuple
#scrapping process
#get date 
for i in range(0,row_length):  
    date = table.find_all('a', attrs={'class':'n'})[i].text
    
    TransDate.append((date))
TransDate
```




    ['2023-5-12',
     '2023-5-11',
     '2023-5-10',
     '2023-5-9',
     '2023-5-8',
     '2023-5-5',
     '2023-5-4',
     '2023-5-3',
     '2023-5-2',
     '2023-5-1',
     '2023-4-28',
     '2023-4-27',
     '2023-4-26',
     '2023-4-25',
     '2023-4-24',
     '2023-4-21',
     '2023-4-20',
     '2023-4-19',
     '2023-4-18',
     '2023-4-17',
     '2023-4-14',
     '2023-4-13',
     '2023-4-12',
     '2023-4-11',
     '2023-4-10',
     '2023-4-7',
     '2023-4-6',
     '2023-4-5',
     '2023-4-4',
     '2023-4-3',
     '2023-3-31',
     '2023-3-30',
     '2023-3-29',
     '2023-3-28',
     '2023-3-27',
     '2023-3-24',
     '2023-3-23',
     '2023-3-22',
     '2023-3-21',
     '2023-3-20',
     '2023-3-17',
     '2023-3-16',
     '2023-3-15',
     '2023-3-14',
     '2023-3-13',
     '2023-3-10',
     '2023-3-9',
     '2023-3-8',
     '2023-3-7',
     '2023-3-6',
     '2023-3-3',
     '2023-3-2',
     '2023-3-1',
     '2023-2-28',
     '2023-2-27',
     '2023-2-24',
     '2023-2-23',
     '2023-2-22',
     '2023-2-21',
     '2023-2-20',
     '2023-2-17',
     '2023-2-16',
     '2023-2-15',
     '2023-2-14',
     '2023-2-13',
     '2023-2-10',
     '2023-2-9',
     '2023-2-8',
     '2023-2-7',
     '2023-2-6',
     '2023-2-3',
     '2023-2-2',
     '2023-2-1',
     '2023-1-31',
     '2023-1-30',
     '2023-1-27',
     '2023-1-26',
     '2023-1-25',
     '2023-1-24',
     '2023-1-23',
     '2023-1-20',
     '2023-1-19',
     '2023-1-18',
     '2023-1-17',
     '2023-1-16',
     '2023-1-13',
     '2023-1-12',
     '2023-1-11',
     '2023-1-10',
     '2023-1-9',
     '2023-1-6',
     '2023-1-5',
     '2023-1-4',
     '2023-1-3',
     '2023-1-2',
     '2022-12-30',
     '2022-12-29',
     '2022-12-28',
     '2022-12-27',
     '2022-12-26',
     '2022-12-23',
     '2022-12-22',
     '2022-12-21',
     '2022-12-20',
     '2022-12-19',
     '2022-12-16',
     '2022-12-15',
     '2022-12-14',
     '2022-12-13',
     '2022-12-12',
     '2022-12-9',
     '2022-12-8',
     '2022-12-7',
     '2022-12-6',
     '2022-12-5',
     '2022-12-2',
     '2022-12-1',
     '2022-11-30',
     '2022-11-29',
     '2022-11-28',
     '2022-11-25',
     '2022-11-24',
     '2022-11-23',
     '2022-11-22',
     '2022-11-21',
     '2022-11-18',
     '2022-11-17',
     '2022-11-16',
     '2022-11-15',
     '2022-11-14']




```python
USDtoIDR = []

for i in range(1,row_length_kurs,4):
    kurs_Rp = table.find_all('span', attrs={'class':'nowrap'})[i].text

    USDtoIDR.append((kurs_Rp))
USDtoIDR     
```




    ['Rp14,844',
     'Rp14,776',
     'Rp14,698',
     'Rp14,776',
     'Rp14,744',
     'Rp14,675',
     'Rp14,699',
     'Rp14,680',
     'Rp14,747',
     'Rp14,677',
     'Rp14,674',
     'Rp14,691',
     'Rp14,841',
     'Rp14,940',
     'Rp14,934',
     'Rp14,936',
     'Rp14,954',
     'Rp14,995',
     'Rp14,889',
     'Rp14,850',
     'Rp14,782',
     'Rp14,722',
     'Rp14,835',
     'Rp14,915',
     'Rp14,950',
     'Rp14,941',
     'Rp14,931',
     'Rp14,960',
     'Rp14,957',
     'Rp14,922',
     'Rp14,969',
     'Rp15,024',
     'Rp15,034',
     'Rp15,060',
     'Rp15,107',
     'Rp15,165',
     'Rp15,085',
     'Rp15,253',
     'Rp15,301',
     'Rp15,343',
     'Rp15,375',
     'Rp15,429',
     'Rp15,459',
     'Rp15,380',
     'Rp15,416',
     'Rp15,503',
     'Rp15,495',
     'Rp15,449',
     'Rp15,429',
     'Rp15,354',
     'Rp15,278',
     'Rp15,322',
     'Rp15,249',
     'Rp15,241',
     'Rp15,216',
     'Rp15,265',
     'Rp15,211',
     'Rp15,197',
     'Rp15,235',
     'Rp15,168',
     'Rp15,166',
     'Rp15,144',
     'Rp15,205',
     'Rp15,186',
     'Rp15,216',
     'Rp15,188',
     'Rp15,147',
     'Rp15,133',
     'Rp15,154',
     'Rp15,198',
     'Rp15,095',
     'Rp14,907',
     'Rp14,896',
     'Rp15,002',
     'Rp15,013',
     'Rp14,973',
     'Rp14,955',
     'Rp14,940',
     'Rp14,956',
     'Rp15,026',
     'Rp15,063',
     'Rp15,154',
     'Rp15,138',
     'Rp15,180',
     'Rp15,126',
     'Rp15,115',
     'Rp15,206',
     'Rp15,432',
     'Rp15,536',
     'Rp15,596',
     'Rp15,607',
     'Rp15,635',
     'Rp15,582',
     'Rp15,594',
     'Rp15,554',
     'Rp15,534',
     'Rp15,627',
     'Rp15,789',
     'Rp15,620',
     'Rp15,621',
     'Rp15,582',
     'Rp15,574',
     'Rp15,540',
     'Rp15,564',
     'Rp15,568',
     'Rp15,616',
     'Rp15,629',
     'Rp15,543',
     'Rp15,560',
     'Rp15,676',
     'Rp15,604',
     'Rp15,594',
     'Rp15,612',
     'Rp15,625',
     'Rp15,520',
     'Rp15,376',
     'Rp15,389',
     'Rp15,633',
     'Rp15,734',
     'Rp15,743',
     'Rp15,693',
     'Rp15,647',
     'Rp15,626',
     'Rp15,664',
     'Rp15,741',
     'Rp15,641',
     'Rp15,714',
     'Rp15,639',
     'Rp15,557',
     'Rp15,554']




Then we combine the `TransDate` and `USDtoIDR` data to form a list of data using the `list(zip(__,__))` function so that they can be converted to one dataframe.


```python
Exchange = list(zip(TransDate, USDtoIDR))
Exchange
```




    [('2023-5-12', 'Rp14,844'),
     ('2023-5-11', 'Rp14,776'),
     ('2023-5-10', 'Rp14,698'),
     ('2023-5-9', 'Rp14,776'),
     ('2023-5-8', 'Rp14,744'),
     ('2023-5-5', 'Rp14,675'),
     ('2023-5-4', 'Rp14,699'),
     ('2023-5-3', 'Rp14,680'),
     ('2023-5-2', 'Rp14,747'),
     ('2023-5-1', 'Rp14,677'),
     ('2023-4-28', 'Rp14,674'),
     ('2023-4-27', 'Rp14,691'),
     ('2023-4-26', 'Rp14,841'),
     ('2023-4-25', 'Rp14,940'),
     ('2023-4-24', 'Rp14,934'),
     ('2023-4-21', 'Rp14,936'),
     ('2023-4-20', 'Rp14,954'),
     ('2023-4-19', 'Rp14,995'),
     ('2023-4-18', 'Rp14,889'),
     ('2023-4-17', 'Rp14,850'),
     ('2023-4-14', 'Rp14,782'),
     ('2023-4-13', 'Rp14,722'),
     ('2023-4-12', 'Rp14,835'),
     ('2023-4-11', 'Rp14,915'),
     ('2023-4-10', 'Rp14,950'),
     ('2023-4-7', 'Rp14,941'),
     ('2023-4-6', 'Rp14,931'),
     ('2023-4-5', 'Rp14,960'),
     ('2023-4-4', 'Rp14,957'),
     ('2023-4-3', 'Rp14,922'),
     ('2023-3-31', 'Rp14,969'),
     ('2023-3-30', 'Rp15,024'),
     ('2023-3-29', 'Rp15,034'),
     ('2023-3-28', 'Rp15,060'),
     ('2023-3-27', 'Rp15,107'),
     ('2023-3-24', 'Rp15,165'),
     ('2023-3-23', 'Rp15,085'),
     ('2023-3-22', 'Rp15,253'),
     ('2023-3-21', 'Rp15,301'),
     ('2023-3-20', 'Rp15,343'),
     ('2023-3-17', 'Rp15,375'),
     ('2023-3-16', 'Rp15,429'),
     ('2023-3-15', 'Rp15,459'),
     ('2023-3-14', 'Rp15,380'),
     ('2023-3-13', 'Rp15,416'),
     ('2023-3-10', 'Rp15,503'),
     ('2023-3-9', 'Rp15,495'),
     ('2023-3-8', 'Rp15,449'),
     ('2023-3-7', 'Rp15,429'),
     ('2023-3-6', 'Rp15,354'),
     ('2023-3-3', 'Rp15,278'),
     ('2023-3-2', 'Rp15,322'),
     ('2023-3-1', 'Rp15,249'),
     ('2023-2-28', 'Rp15,241'),
     ('2023-2-27', 'Rp15,216'),
     ('2023-2-24', 'Rp15,265'),
     ('2023-2-23', 'Rp15,211'),
     ('2023-2-22', 'Rp15,197'),
     ('2023-2-21', 'Rp15,235'),
     ('2023-2-20', 'Rp15,168'),
     ('2023-2-17', 'Rp15,166'),
     ('2023-2-16', 'Rp15,144'),
     ('2023-2-15', 'Rp15,205'),
     ('2023-2-14', 'Rp15,186'),
     ('2023-2-13', 'Rp15,216'),
     ('2023-2-10', 'Rp15,188'),
     ('2023-2-9', 'Rp15,147'),
     ('2023-2-8', 'Rp15,133'),
     ('2023-2-7', 'Rp15,154'),
     ('2023-2-6', 'Rp15,198'),
     ('2023-2-3', 'Rp15,095'),
     ('2023-2-2', 'Rp14,907'),
     ('2023-2-1', 'Rp14,896'),
     ('2023-1-31', 'Rp15,002'),
     ('2023-1-30', 'Rp15,013'),
     ('2023-1-27', 'Rp14,973'),
     ('2023-1-26', 'Rp14,955'),
     ('2023-1-25', 'Rp14,940'),
     ('2023-1-24', 'Rp14,956'),
     ('2023-1-23', 'Rp15,026'),
     ('2023-1-20', 'Rp15,063'),
     ('2023-1-19', 'Rp15,154'),
     ('2023-1-18', 'Rp15,138'),
     ('2023-1-17', 'Rp15,180'),
     ('2023-1-16', 'Rp15,126'),
     ('2023-1-13', 'Rp15,115'),
     ('2023-1-12', 'Rp15,206'),
     ('2023-1-11', 'Rp15,432'),
     ('2023-1-10', 'Rp15,536'),
     ('2023-1-9', 'Rp15,596'),
     ('2023-1-6', 'Rp15,607'),
     ('2023-1-5', 'Rp15,635'),
     ('2023-1-4', 'Rp15,582'),
     ('2023-1-3', 'Rp15,594'),
     ('2023-1-2', 'Rp15,554'),
     ('2022-12-30', 'Rp15,534'),
     ('2022-12-29', 'Rp15,627'),
     ('2022-12-28', 'Rp15,789'),
     ('2022-12-27', 'Rp15,620'),
     ('2022-12-26', 'Rp15,621'),
     ('2022-12-23', 'Rp15,582'),
     ('2022-12-22', 'Rp15,574'),
     ('2022-12-21', 'Rp15,540'),
     ('2022-12-20', 'Rp15,564'),
     ('2022-12-19', 'Rp15,568'),
     ('2022-12-16', 'Rp15,616'),
     ('2022-12-15', 'Rp15,629'),
     ('2022-12-14', 'Rp15,543'),
     ('2022-12-13', 'Rp15,560'),
     ('2022-12-12', 'Rp15,676'),
     ('2022-12-9', 'Rp15,604'),
     ('2022-12-8', 'Rp15,594'),
     ('2022-12-7', 'Rp15,612'),
     ('2022-12-6', 'Rp15,625'),
     ('2022-12-5', 'Rp15,520'),
     ('2022-12-2', 'Rp15,376'),
     ('2022-12-1', 'Rp15,389'),
     ('2022-11-30', 'Rp15,633'),
     ('2022-11-29', 'Rp15,734'),
     ('2022-11-28', 'Rp15,743'),
     ('2022-11-25', 'Rp15,693'),
     ('2022-11-24', 'Rp15,647'),
     ('2022-11-23', 'Rp15,626'),
     ('2022-11-22', 'Rp15,664'),
     ('2022-11-21', 'Rp15,741'),
     ('2022-11-18', 'Rp15,641'),
     ('2022-11-17', 'Rp15,714'),
     ('2022-11-16', 'Rp15,639'),
     ('2022-11-15', 'Rp15,557'),
     ('2022-11-14', 'Rp15,554')]



To do a further analysis let's reverse our list we can use `::-1` to do that, because the original webpage give us reversed information.


```python
Exchange=Exchange[::-1]
Exchange[:10]
```




    [('2022-11-14', 'Rp15,554'),
     ('2022-11-15', 'Rp15,557'),
     ('2022-11-16', 'Rp15,639'),
     ('2022-11-17', 'Rp15,714'),
     ('2022-11-18', 'Rp15,641'),
     ('2022-11-21', 'Rp15,741'),
     ('2022-11-22', 'Rp15,664'),
     ('2022-11-23', 'Rp15,626'),
     ('2022-11-24', 'Rp15,647'),
     ('2022-11-25', 'Rp15,693')]



## Creating data frame & Data wrangling

Put the array into dataframe


```python
import pandas as pd
pd.options.display.float_format = '{:,.2f}'.format
df = pd.DataFrame(Exchange,columns=('Date','USDtoIDR'))
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>USDtoIDR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-11-14</td>
      <td>Rp15,554</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-11-15</td>
      <td>Rp15,557</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-11-16</td>
      <td>Rp15,639</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-11-17</td>
      <td>Rp15,714</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-11-18</td>
      <td>Rp15,641</td>
    </tr>
  </tbody>
</table>
</div>




The data that is formed from the conversion of the list to the dataframe has a data type object, so it is necessary to change the appropriate data type and clean up unused strings.


```python
df.dtypes
```




    Date        object
    USDtoIDR    object
    dtype: object




```python
df['USDtoIDR']=df['USDtoIDR'].str.replace('Rp','')
```


```python
df['USDtoIDR']=df['USDtoIDR'].str.replace(',','')
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>USDtoIDR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-11-14</td>
      <td>15554</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-11-15</td>
      <td>15557</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-11-16</td>
      <td>15639</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-11-17</td>
      <td>15714</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-11-18</td>
      <td>15641</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>125</th>
      <td>2023-5-8</td>
      <td>14744</td>
    </tr>
    <tr>
      <th>126</th>
      <td>2023-5-9</td>
      <td>14776</td>
    </tr>
    <tr>
      <th>127</th>
      <td>2023-5-10</td>
      <td>14698</td>
    </tr>
    <tr>
      <th>128</th>
      <td>2023-5-11</td>
      <td>14776</td>
    </tr>
    <tr>
      <th>129</th>
      <td>2023-5-12</td>
      <td>14844</td>
    </tr>
  </tbody>
</table>
<p>130 rows × 2 columns</p>
</div>




```python
df['USDtoIDR'] = df['USDtoIDR'].astype('int64')
df['Date'] = df['Date'].astype('datetime64[ns]')

df.dtypes
```




    Date        datetime64[ns]
    USDtoIDR             int64
    dtype: object




In the visualization process, it is necessary to set `Date` to be an index so that it is not read as data for graphing.


```python
df = df.set_index('Date')
```


```python
df.plot(title='US Dollar (USD) To Indonesian Rupiah (IDR) Exchange Rate History',
               xlabel='Date',
               ylabel='Rate (Rp)',
               figsize=(12,8)
       )
```




    <Axes: title={'center': 'US Dollar (USD) To Indonesian Rupiah (IDR) Exchange Rate History'}, xlabel='Date', ylabel='Rate (Rp)'>




    
![png](output_34_1.png)
    


### Implementing your webscrapping to the flask dashboard

- Copy paste all of your web scrapping process to the desired position on the `app.py`
- Changing the title of the dasboard at `index.html`

## Analysis and Conclusion


```python
df.describe(include='number')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>USDtoIDR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>130.00</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>15,240.88</td>
    </tr>
    <tr>
      <th>std</th>
      <td>315.06</td>
    </tr>
    <tr>
      <th>min</th>
      <td>14,674.00</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>14,957.75</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>15,208.50</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>15,556.25</td>
    </tr>
    <tr>
      <th>max</th>
      <td>15,789.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
df['USDtoIDR'].idxmax()
```




    Timestamp('2022-12-28 00:00:00')




```python
df['USDtoIDR'].idxmin()
```




    Timestamp('2023-04-28 00:00:00')



This chart shows data from 2022-11-14 to 2023-5-12.
- The USD/IDR rate is down **-5.00% in the six months**. This means the US Dollar has decreased in value compared to the Indonesian Rupiah.
- Highest:	15,789 IDR on December 28, 2022
- Lowest :	14,674 IDR on April 28, 2023

### Implement it at the webapps

- You can create additional analysis from the data.
- Implement it to the dashboard with at `app.py` dan `index.html`.
