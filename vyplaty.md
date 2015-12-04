
# Zadanie:


## Zistiť z dát, ktoré platby predstavujú mzdy a nájsť ďalšie hlavné kategórie pre platby 


```python

# Importovanie knižníc

import datetime
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings('ignore')


```


```python

# Načítanie dát a vytvorenie názvov pre jednotlivé stĺpce

data = pd.read_table('data_for_applicants.txt', header = None, 
                     names = ['date', 'client', 'amount', 'sender'])

```


```python

# pohľad na dáta

data.head()

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2014-09-12</td>
      <td>651959088</td>
      <td>6686</td>
      <td>627986531</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014-09-17</td>
      <td>725866593</td>
      <td>14462</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2014-09-12</td>
      <td>313567829</td>
      <td>9965</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2014-09-12</td>
      <td>855015364</td>
      <td>10835</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2014-09-03</td>
      <td>414606140</td>
      <td>18320</td>
      <td>222382928</td>
    </tr>
  </tbody>
</table>
</div>



* **date** --  dátum

* **client**  -- id klienta

* **amount**  -- výška platby

* **sender**  -- id odosielateľa

## 1. Krok 

* Stanovenie minimálnej mzdy. V roku 2014 bola minimálna mzda v ČR 8500 korún

* Vyselektovanie platieb vyšších ako min. mzda do premennej **high_income**


```python

min_mzda = 8500

# podmnožina high_income obsahuje všetky platby vyššie ako minimálna mzda + ostatné stĺpce

high_income = data[ data['amount'] > min_mzda ]

```

## 2. Krok

* Konvertovanie dátumov z formátu rok-mesiac-deň na rok-mesiac. Stĺpec **date** bude obsahovať iba 6 kategórii (mesiacov), čo uľahčí hľadanie výplat (výplatu hodnotím ako pravidelnú platbu, ktora bola klientovi pripísaná na účet každý mesiac aspoň od jedného odosielateľa) <br/> <br/>

* Pridanie nového stĺpca **join_id** do podmnožiny **high_income**, ktorý bude obsahovať spárované id klienta a id odosieteľa. Vytvorím tak jedninečné id páry medzi klientom a odosielateľom, ktoré mi pomôžu pri selekcii výplat <br/> <br/> 

* Následné vytvorenie nových premenných (**months, id**) s jedinečnými hodnotami pre mesiace a id páry. Obidve premenné vložím do **for loop** cyklu. Cieľ cyklu je získanie iba takých id párov, ktoré sa nachádzajú v každom mesiaci. Použijem na to funkciu intersection. Výsledné id páry mi pomôžu vyselektovať len takých klientov, ktorí dostávajú platby vyššie ako min. mzda každý mesiac 


```python

# formátovanie dátumov

date = pd.to_datetime( high_income['date'] )
high_income['date'] = date.dt.strftime('%Y-%m')

# vytvorenie nového stĺpca ('join_id'), ktorý bude obsahovať id páry medzi klientom a odosielateľom

high_income['join_id'] = high_income.client.map(str) + high_income.sender.map(str)

# vytvorene premenných, ktoré obsahujú unikátne dátumy a spojené id páry

months = set(high_income['date'])
id = set(high_income['join_id'])

# cyklus, ktorého cieľom je nájsť všetky id páry nachádzajúce sa v každom mesiaci

for m in months:
    month = high_income[ high_income['date'] == m ]
    id = id.intersection(month['join_id']) 
    
    
# pohľad na id páry

print(list(id)[0:5])

```

    ['814910129838639938', '821357916674455418', '132302670356081381', '893860660559111653', '592384784308204383']


## 3. Krok

* Použitie id párov, ktoré sa nachádzajú v premennej **id** na získanie novej podmnožiny **stable_income**. Nová podmnožina bude obsahovať všetkých klientov, ktorí dostávajú každý mesiac platby minimálne od jedného odosielateľov. Každý odosielateľ platí klientovi každý mesiac minimálne jednu čiastku, ktorá prevyšuje minimálnu mzdu <br/> <br/>

* Následne sa vytvorí nová premenná **client_senders**, ktorá obsahuje počty jedinečných odosielateľov pre kažďeho klienta. Táto premenná sa použije na vyselektovanie klientov, ktorí majú len jedného odosielateľa (**one_sender**) alebo viacerých odosielateľov (**multi_sender**)


```python

# vyselektovanie dát z high_income podľa id párov z premennej 'id', ktorá pochádza z for loop cyklu

stable_income = high_income[ high_income['join_id'].isin(id) ]

# vytvorenie premennej, ktorá obsahuje počty unikátnych odosielateľov pre každého klienta

client_senders = stable_income.groupby('client')['sender'].nunique()

# premenná, ktorá obsahuje id klientov len s jedným odosielateľom  

one_sender = client_senders[ client_senders == 1 ].index

# v premennej multi_sender sa nachádzajú klienti, ktorí dostávajú platby od viacerých odosielateľov každý  mesiac

multi_sender = client_senders[ client_senders > 1 ].index

```

### Tabuľka početnosti klientov s unikátnymi odosielateľmi 


```python

# v tabuľke na ľavej strane sú skupiny vyjadrujúce množstvo jedinečných odosielateľov pre klientov 

# v pravej časti je celkový počet klientov, ktorí majú dané množstvo unikátnych odosielateľov prislúchajúce na ľavej strane

# napr. 3. riadok z výstupu vyjadruje celkové množstvo klientov, ktorí majú troch unikátnych odosielateľov

client_senders.value_counts()
```




    1     15033
    2      1808
    3       103
    4         2
    11        1
    Name: sender, dtype: int64



## 4. Krok 

* Vytvorenie dvoch nových podmnožín zo **stable_income** na základe id klientov z premenných **one_sender** a **multi_sender** <br/> <br/>

* Dostanem podmnožinu **income_one_sender**, ktorá obsahuje klientov len s jedným odosielateľom platiacim každý mesiac klientovi čiastku prevyšujúcu 8500 korún. <br/> <br/>

* Druhá podmnožina **income_multi_sender** bude obsahovať klientov s viacerými odosielateľmi, ktorí posielajú klientovi každý mesiac peniaze v hodnote prevyšujúcej minimálnu mzdu 



```python

# prvá podmnožina obsahuje klientov len z jedným odosielateľom platiacim každý mesiac

income_one_sender = stable_income[ stable_income['client'].isin(one_sender) ]

# druhá podmnožina obsahuje klientov, ktorí dostávajú platby od viacerých odosielateľov každý mesiac

income_multi_sender = stable_income[ stable_income['client'].isin(multi_sender) ]

```

### Pohľad na klientov s jedným alebo s viacerými odosielateľmi, ktorí posielajú klientom každý mesiac čiastku vyššiu ako min. mzda


```python

# prvá podmnožina je zoradená podľa id klientov a dátumov, obsahuje klienov s jedným odosielateľom

income_one_sender.sort(['client', 'date']).head(12)

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
      <th>join_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>105145</th>
      <td>2014-04</td>
      <td>56847</td>
      <td>11432</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>102734</th>
      <td>2014-05</td>
      <td>56847</td>
      <td>11437</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>99788</th>
      <td>2014-06</td>
      <td>56847</td>
      <td>11434</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>96986</th>
      <td>2014-07</td>
      <td>56847</td>
      <td>11434</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>94791</th>
      <td>2014-08</td>
      <td>56847</td>
      <td>11339</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>91341</th>
      <td>2014-09</td>
      <td>56847</td>
      <td>11371</td>
      <td>222382928</td>
      <td>56847222382928</td>
    </tr>
    <tr>
      <th>69484</th>
      <td>2014-04</td>
      <td>61441</td>
      <td>17032</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
    <tr>
      <th>68814</th>
      <td>2014-05</td>
      <td>61441</td>
      <td>45773</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
    <tr>
      <th>65786</th>
      <td>2014-06</td>
      <td>61441</td>
      <td>17821</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
    <tr>
      <th>62836</th>
      <td>2014-07</td>
      <td>61441</td>
      <td>17445</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
    <tr>
      <th>57598</th>
      <td>2014-08</td>
      <td>61441</td>
      <td>25562</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
    <tr>
      <th>56965</th>
      <td>2014-09</td>
      <td>61441</td>
      <td>17459</td>
      <td>547166056</td>
      <td>61441547166056</td>
    </tr>
  </tbody>
</table>
</div>




```python

# druhá podmnožina obsahuje klientov s viacerými odosielateľmi

income_multi_sender.sort(['client', 'date']).head(12)

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
      <th>join_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>70642</th>
      <td>2014-04</td>
      <td>450431</td>
      <td>12704</td>
      <td>219047249</td>
      <td>450431219047249</td>
    </tr>
    <tr>
      <th>327953</th>
      <td>2014-04</td>
      <td>450431</td>
      <td>14748</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>327954</th>
      <td>2014-04</td>
      <td>450431</td>
      <td>16117</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>68106</th>
      <td>2014-05</td>
      <td>450431</td>
      <td>12704</td>
      <td>219047249</td>
      <td>450431219047249</td>
    </tr>
    <tr>
      <th>326953</th>
      <td>2014-05</td>
      <td>450431</td>
      <td>15095</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>326954</th>
      <td>2014-05</td>
      <td>450431</td>
      <td>16359</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>65081</th>
      <td>2014-06</td>
      <td>450431</td>
      <td>12706</td>
      <td>219047249</td>
      <td>450431219047249</td>
    </tr>
    <tr>
      <th>326077</th>
      <td>2014-06</td>
      <td>450431</td>
      <td>12842</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>326078</th>
      <td>2014-06</td>
      <td>450431</td>
      <td>16303</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>62153</th>
      <td>2014-07</td>
      <td>450431</td>
      <td>12779</td>
      <td>219047249</td>
      <td>450431219047249</td>
    </tr>
    <tr>
      <th>325128</th>
      <td>2014-07</td>
      <td>450431</td>
      <td>12214</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
    <tr>
      <th>325129</th>
      <td>2014-07</td>
      <td>450431</td>
      <td>16009</td>
      <td>633236581</td>
      <td>450431633236581</td>
    </tr>
  </tbody>
</table>
</div>



## 5. Krok

* Vytvorenie zvyšnej podmnožiny (**unstable_income**) kde sa nachádzajú klienti, ktorým chýba odosielateľ platiaci každý mesiac čiastku prevyšujúcu minimálnu mzdu. V danej podmnožine klienti môžu prijímať každý mesiac sumu vyššiu ako je minimálna mzda, ale musia ju prijímať minimálne od dvoch odosielateľov. <br/> <br/>

* Podmnožinu **unstable_income** dostanem z **high_income** kde sa nachádzajú všetky platby vyššie ako minimálna mzda. Budem na to potrebovať spoločné indexy z podmnožín **income_one_sender** a **income_multi_sender**, ktoré odčítam od indexov z **high_income**. Dostanem klientov, ktorí nemajú pravidelné mesačné platby od jedného alebo viacerých odosielateľov <br/> <br/>



```python

# spoločne indexy z income_ine_sender a income_multi_sender pridelené do premennej income_all

income_all = pd.concat([income_one_sender, income_multi_sender], axis=0)

# podmnožina unstable_income bude obsahovať indexy, ktoré sa nachádazjú v high_income, ale nenachádzajú sa v income_all
# dostanem klientov s odosielateľmi, ktorí im neposialajú každý  mesiac čiastku prevyšujúcu min. mzdu

unstable_income = high_income[~ high_income.index.isin(income_all.index)]

```

### Pohľad na dáta v unstable_income podmnožine


```python
unstable_income.sort(['client', 'date']).head(14)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
      <th>join_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>285509</th>
      <td>2014-04</td>
      <td>61441</td>
      <td>15346</td>
      <td>45437871</td>
      <td>6144145437871</td>
    </tr>
    <tr>
      <th>243911</th>
      <td>2014-05</td>
      <td>61441</td>
      <td>9703</td>
      <td>762490320</td>
      <td>61441762490320</td>
    </tr>
    <tr>
      <th>243104</th>
      <td>2014-06</td>
      <td>61441</td>
      <td>8875</td>
      <td>762490320</td>
      <td>61441762490320</td>
    </tr>
    <tr>
      <th>240922</th>
      <td>2014-08</td>
      <td>61441</td>
      <td>11008</td>
      <td>762490320</td>
      <td>61441762490320</td>
    </tr>
    <tr>
      <th>397989</th>
      <td>2014-04</td>
      <td>72755</td>
      <td>9981</td>
      <td>566495527</td>
      <td>72755566495527</td>
    </tr>
    <tr>
      <th>54286</th>
      <td>2014-04</td>
      <td>78274</td>
      <td>68655</td>
      <td>482592832</td>
      <td>78274482592832</td>
    </tr>
    <tr>
      <th>104502</th>
      <td>2014-04</td>
      <td>87884</td>
      <td>21397</td>
      <td>222382928</td>
      <td>87884222382928</td>
    </tr>
    <tr>
      <th>48037</th>
      <td>2014-06</td>
      <td>106137</td>
      <td>10301</td>
      <td>627986531</td>
      <td>106137627986531</td>
    </tr>
    <tr>
      <th>45180</th>
      <td>2014-07</td>
      <td>106137</td>
      <td>11864</td>
      <td>627986531</td>
      <td>106137627986531</td>
    </tr>
    <tr>
      <th>40155</th>
      <td>2014-08</td>
      <td>106137</td>
      <td>25278</td>
      <td>345882789</td>
      <td>106137345882789</td>
    </tr>
    <tr>
      <th>40156</th>
      <td>2014-08</td>
      <td>106137</td>
      <td>12263</td>
      <td>627986531</td>
      <td>106137627986531</td>
    </tr>
    <tr>
      <th>39536</th>
      <td>2014-09</td>
      <td>106137</td>
      <td>12161</td>
      <td>627986531</td>
      <td>106137627986531</td>
    </tr>
    <tr>
      <th>248309</th>
      <td>2014-05</td>
      <td>155840</td>
      <td>14066</td>
      <td>745087094</td>
      <td>155840745087094</td>
    </tr>
    <tr>
      <th>180008</th>
      <td>2014-09</td>
      <td>184477</td>
      <td>219466</td>
      <td>329355591</td>
      <td>184477329355591</td>
    </tr>
  </tbody>
</table>
</div>



## 6. Krok

* Vytvorenie premennej **client_month_incomes** z **unstable_incomes**, ktorá bude obsahovať klientov a počet mesiacov, v ktorých klienti dostali platbu prevyšujúcu min. mzdu. Ak sa bude id klienta nachádzať vo všetkých šiestich mesiacoch, tak daný klient bude príjimať každý mesiac platbu vyššiu ako je minimálna mzda. Jeho odosielatelia budú minimálne dvaja a ich platby nebude klient dostávať každý mesiac. Napr. jeden odosielateľ bude klientovi posielať peniaze iba 4 mesiace a zvyšné 2 mesiace bude posielať druhý odosielateľ <br/> <br/>

* Pohľad na počet klientov, ktorí su zaradení do skupín mesiacov, v ktorých dostali platbu prevyšujúcu min. mzdu. 


```python

# premenná, ktorá obsahuje id klientov a ich výskyt v danom počte mesiacov kedy dostali platbu  

client_month_incomes = unstable_income.groupby('client')['date'].nunique()

# pohľad na počet klientov, ktorí dostali platby len v určitom počte mesiacov

client_month_incomes.value_counts()

```




    1    7765
    6    3016
    2    2575
    5    2049
    4    1696
    3    1549
    Name: date, dtype: int64



V tabuľke hore, napr. v 4. riadku je 2049 klientov, ktorí dostali platbu v piatich mesiacoch, ale v jednom mesiaci platbu nedostali

## 7. Krok

* Selekcia id klientov z **client_month_incomes**, ktorí dostali každý mesiac platbu prevyšujúcu min. mzdu do premennej **client_id** <br/> <br/>

* ID klientov použijem na vytvorenie novej podmnožiny **six_month_clients** z *unstable_income*. To bude zároveň posledná podmnožina s klientami, ktorí dostali každý mesiac príjem vyšší ako minimálna mzda. Ostatní klienti nedostali každý mesiac čiastku, ktorá by prevyšovala hodnotu minimálnu mzdy. Vytvorím pre nich podmnožinu **other_incomes**


```python

# id klientov, ktorí každý mesiac prijali platbu vyššiu ako min. mzda

client_id = client_month_incomes[ client_month_incomes == 6].index

# vytvorenie podmnoziny six_month_clients cez spoločných id klientov medzi unstable_income a client_id 

six_month_clients = unstable_income[unstable_income['client'].isin(client_id)]

# zvyšná skupina klientov, ktorí nedostali platby vyššie ako min. mzda každy mesiac 

other_incomes = unstable_income[~ unstable_income['client'].isin(client_id)]

```

### Pohľad na podmnožinu six_month_clients


```python
six_month_clients.sort(['client', 'date']).head(12)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
      <th>join_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>168804</th>
      <td>2014-04</td>
      <td>1177887</td>
      <td>14853</td>
      <td>784440758</td>
      <td>1177887784440758</td>
    </tr>
    <tr>
      <th>167892</th>
      <td>2014-05</td>
      <td>1177887</td>
      <td>13774</td>
      <td>136454232</td>
      <td>1177887136454232</td>
    </tr>
    <tr>
      <th>167115</th>
      <td>2014-06</td>
      <td>1177887</td>
      <td>14985</td>
      <td>136454232</td>
      <td>1177887136454232</td>
    </tr>
    <tr>
      <th>166230</th>
      <td>2014-07</td>
      <td>1177887</td>
      <td>10649</td>
      <td>136454232</td>
      <td>1177887136454232</td>
    </tr>
    <tr>
      <th>165594</th>
      <td>2014-08</td>
      <td>1177887</td>
      <td>13903</td>
      <td>136454232</td>
      <td>1177887136454232</td>
    </tr>
    <tr>
      <th>164540</th>
      <td>2014-09</td>
      <td>1177887</td>
      <td>14394</td>
      <td>136454232</td>
      <td>1177887136454232</td>
    </tr>
    <tr>
      <th>421948</th>
      <td>2014-04</td>
      <td>2088495</td>
      <td>12377</td>
      <td>773266812</td>
      <td>2088495773266812</td>
    </tr>
    <tr>
      <th>421605</th>
      <td>2014-05</td>
      <td>2088495</td>
      <td>11401</td>
      <td>773266812</td>
      <td>2088495773266812</td>
    </tr>
    <tr>
      <th>218683</th>
      <td>2014-06</td>
      <td>2088495</td>
      <td>13985</td>
      <td>536955217</td>
      <td>2088495536955217</td>
    </tr>
    <tr>
      <th>217753</th>
      <td>2014-07</td>
      <td>2088495</td>
      <td>24421</td>
      <td>536955217</td>
      <td>2088495536955217</td>
    </tr>
    <tr>
      <th>216207</th>
      <td>2014-08</td>
      <td>2088495</td>
      <td>24556</td>
      <td>536955217</td>
      <td>2088495536955217</td>
    </tr>
    <tr>
      <th>215966</th>
      <td>2014-09</td>
      <td>2088495</td>
      <td>26815</td>
      <td>536955217</td>
      <td>2088495536955217</td>
    </tr>
  </tbody>
</table>
</div>



## Zahrnutie klientov do segmentov <br/>

* a) Prvý segment obsahuje klientov, ktorí prijali každý mesiac platby prevyšujúce min. mzdu len od jedného odosielateľa. Tvorí ho **income_one_sender** <br/> <br/>

* b) Druhý segment obsahuje klientov s viacerými odosielateľmi, ktorí im posielali platby každý  mesiac. Ak klient prijímal peniaze od dvoch odosielateľov, tak obidvaja odosielatelia posielali klientovi každý mesiac na účet čiastku prevyšujúcu min. mzdu. Tento segment tvorí **income_multi_sender**<br/> <br/>

* c) Tretí segment obsahuje klientov, ktorí síce dostali každý mesiac príjem vyšši ako min. mzda, ale ich odosielatelia im neposielali peniaze každý mesiac. Klienti v tomto segment prijímali platby najmenej od dvoch odosielateľov. Tretí segment tvorí **six_month_clients**  <br/> <br/>

* d) Posledný segment obsahuje klientov, ktorým chýbal minimálne jeden príjem za určitý mesiac prevyšujúci min. mzdu. V ostatných mesiacoch mohli prijať čiastky prevyšujúce min. mzdu. Prehľad aké množstvo klientov prijalo platby v danom počte mesiacov je znázornení v tabuľka v 6. kroku. Posledný segment tvorí **other_incomes** <br/> <br/>

* Klientov z posledných dvoch segmentov využijem aj na zaradenie do nižších platových segmentov 

## Ďalší segment - dôchodky, soc. dávky, brigády, part-time a nízke príjmy

V ďalších segmentoch sa budú nachádzať klienti s nižšími pravidelnými príjmami ako je min. mzda. Klienti nebudú patriť do prvých dvoch segmentov **a)** a **d)** spomenutých vyššie, ale môzu patriť do segmentov **c)** a **d)**. Platby v nových segmentoch môžu byť dôchodky, sociálne dávky, platby z brigád a podobne. Pri selekcii platieb stanovím minimálnu hranicu 3100 korún, čo predstavovalo v roku 2014 neoficiálne minimálny dôchodok v ČR. V ďalšom kroku nájdem id klientov, ktorí prijímali každý mesiac platby vyššie ako min. dôchodok a zároveň nižšie ako min. mzda. Postupujem podobne ako v 2. kroku. Platby označujem ako dôchodky len pre zjednodušenie.


```python

# prvé dva segmenty spomenuté vyššie boli spojené do premennej income_all v 5. kroku. Z tejto premennej použijem 
# id klientov na selekciu nových klientov, ktorí nepatria do income_all, ale nachádzajú sa v pôvodných dátach

client_selection = data[~ data['client'].isin(income_all.client)]

# premenná pension obsahuje platby vyššie ako min. dôchodok, ale nižšie ako min. mzda

min_penzia = 3100
pension = client_selection[ (client_selection['amount'] > min_penzia) & (client_selection['amount'] <= min_mzda) ] 

# formátujem dátumi na rok-mesiac 

date = pd.to_datetime( pension['date'] )
pension['date'] = date.dt.strftime('%Y-%m')

# premmené, ktoré vložím do for loop cyklu

months = set(pension['date'])
id = set(pension['client'])

# for loop cyklus na selekciu id klientov, ktorí prijímali "penziu" každý  mesiac

for m in months:
    month = pension[ pension['date'] == m ]
    id = id.intersection(month['client']) 

# vytvorenie segmentu s klientami, ktorí každý mesiac dostali "penziu" od 1 a viac odsielateľov

stable_pension_income = pension[ pension['client'].isin(id) ]

```

### Pohľad na dáta klientov s pravidelnými mesačnými "penziami" 


```python
stable_pension_income.sort(['client', 'date']).head(18)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>52493</th>
      <td>2014-04</td>
      <td>3306</td>
      <td>5404</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>48841</th>
      <td>2014-05</td>
      <td>3306</td>
      <td>5404</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>321072</th>
      <td>2014-05</td>
      <td>3306</td>
      <td>4216</td>
      <td>550277640</td>
    </tr>
    <tr>
      <th>45872</th>
      <td>2014-06</td>
      <td>3306</td>
      <td>5348</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>320127</th>
      <td>2014-06</td>
      <td>3306</td>
      <td>5503</td>
      <td>550277640</td>
    </tr>
    <tr>
      <th>43008</th>
      <td>2014-07</td>
      <td>3306</td>
      <td>5404</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>319208</th>
      <td>2014-07</td>
      <td>3306</td>
      <td>5591</td>
      <td>550277640</td>
    </tr>
    <tr>
      <th>40843</th>
      <td>2014-08</td>
      <td>3306</td>
      <td>5403</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>318568</th>
      <td>2014-08</td>
      <td>3306</td>
      <td>6957</td>
      <td>550277640</td>
    </tr>
    <tr>
      <th>37508</th>
      <td>2014-09</td>
      <td>3306</td>
      <td>5400</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>317449</th>
      <td>2014-09</td>
      <td>3306</td>
      <td>5588</td>
      <td>550277640</td>
    </tr>
    <tr>
      <th>106681</th>
      <td>2014-04</td>
      <td>87884</td>
      <td>5179</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>103385</th>
      <td>2014-05</td>
      <td>87884</td>
      <td>5178</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>100425</th>
      <td>2014-06</td>
      <td>87884</td>
      <td>5170</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>97615</th>
      <td>2014-07</td>
      <td>87884</td>
      <td>5170</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>95377</th>
      <td>2014-08</td>
      <td>87884</td>
      <td>5174</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>91942</th>
      <td>2014-09</td>
      <td>87884</td>
      <td>5173</td>
      <td>222382928</td>
    </tr>
    <tr>
      <th>353968</th>
      <td>2014-04</td>
      <td>268247</td>
      <td>5557</td>
      <td>136793160</td>
    </tr>
  </tbody>
</table>
</div>



Ostatných klientov, ktorí neprijímali "penziu" pravidelne každý mesiac prenesiem do segmentu z označením **unstable_pension**. Budú to klienti, ktorí sa nenechádzajú v **stable_pension_income**, ale sú  prítomní v **pension** 


```python
unstable_pension = pension[~ pension['client'].isin(stable_pension_income['client'])]
```

### Pohľad na klientov s nepravidelnými mesačnými platbami vo výške 3100-8500 korún



```python
unstable_pension.sort(['client', 'date']).head(8)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>53350</th>
      <td>2014-04</td>
      <td>106137</td>
      <td>6326</td>
      <td>345882789</td>
    </tr>
    <tr>
      <th>51034</th>
      <td>2014-05</td>
      <td>106137</td>
      <td>6226</td>
      <td>345882789</td>
    </tr>
    <tr>
      <th>48036</th>
      <td>2014-06</td>
      <td>106137</td>
      <td>6321</td>
      <td>345882789</td>
    </tr>
    <tr>
      <th>87643</th>
      <td>2014-04</td>
      <td>503312</td>
      <td>4164</td>
      <td>31873657</td>
    </tr>
    <tr>
      <th>85758</th>
      <td>2014-05</td>
      <td>516929</td>
      <td>4181</td>
      <td>439913156</td>
    </tr>
    <tr>
      <th>408544</th>
      <td>2014-09</td>
      <td>516929</td>
      <td>4460</td>
      <td>813580157</td>
    </tr>
    <tr>
      <th>405448</th>
      <td>2014-07</td>
      <td>842767</td>
      <td>5193</td>
      <td>635276731</td>
    </tr>
    <tr>
      <th>118480</th>
      <td>2014-06</td>
      <td>958936</td>
      <td>5160</td>
      <td>963808981</td>
    </tr>
  </tbody>
</table>
</div>



V poslednom segmente sú klienti, ktorých platby nepresahujú výšku minimálneho dôchodku. Vyselektujem ich z **high_incomes** a **pension** kde sa nachádzajú klienti, ktorí dostali aspoň jednu platbu vyššiu ako min. dôchodok. Posledný segment označím ako **other_low_incomes**


```python

# spojenie id klientov z high_income (z 1. kroku) a z pension

higher_incomes = pd.concat([high_income.client, pension.client], axis=0)

# posledný segment s klientami, ktorí nedostali ani jednu platbu vyššiu ako min. dôchodok

other_low_incomes = data[~ data['client'].isin(higher_incomes)]

```

### Pohľad na posledný segment


```python
other_low_incomes.sort(['client', 'date']).head(10)

```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>client</th>
      <th>amount</th>
      <th>sender</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>66340</th>
      <td>2014-06-11</td>
      <td>120424</td>
      <td>195</td>
      <td>544975132</td>
    </tr>
    <tr>
      <th>63308</th>
      <td>2014-07-11</td>
      <td>120424</td>
      <td>194</td>
      <td>544975132</td>
    </tr>
    <tr>
      <th>58071</th>
      <td>2014-08-11</td>
      <td>120424</td>
      <td>201</td>
      <td>544975132</td>
    </tr>
    <tr>
      <th>57436</th>
      <td>2014-09-11</td>
      <td>120424</td>
      <td>200</td>
      <td>544975132</td>
    </tr>
    <tr>
      <th>357754</th>
      <td>2014-06-12</td>
      <td>295940</td>
      <td>281</td>
      <td>233423418</td>
    </tr>
    <tr>
      <th>229886</th>
      <td>2014-04-13</td>
      <td>484522</td>
      <td>414</td>
      <td>849713090</td>
    </tr>
    <tr>
      <th>229491</th>
      <td>2014-05-11</td>
      <td>484522</td>
      <td>428</td>
      <td>849713090</td>
    </tr>
    <tr>
      <th>228780</th>
      <td>2014-06-17</td>
      <td>484522</td>
      <td>431</td>
      <td>849713090</td>
    </tr>
    <tr>
      <th>228010</th>
      <td>2014-07-10</td>
      <td>484522</td>
      <td>419</td>
      <td>849713090</td>
    </tr>
    <tr>
      <th>226733</th>
      <td>2014-08-17</td>
      <td>484522</td>
      <td>420</td>
      <td>849713090</td>
    </tr>
  </tbody>
</table>
</div>



## Zhrnutie segmentov <br/> 

* Podľa výšky platieb: | **0-3100** | **3100-8500** | **8500+** | <br/> <br/>



Segmenty **a), b), c), d)** už boli vysvetlené vyššie a patria do kategórie **8500+** <br/> <br/>

Ďalšie segmenty pre kategóriu **3100-8500** <br/> 

e) Segment **stable_pension_income** obsahuje klientov, ktorí každý mesiac dostali platbu ("penziu") vo výške 3100-8500  korún od jedného alebo viacerých odosielateľov. V tomto segmente môžu byť zahrnutý klienti zo segmentov **c)** a **d)**

f) Segment **unstable_pension** obsahuje klientov, ktorí nedostali "penziu" každý mesiac <br/> <br/>


Posledný  segment pre kategóriu **0-3100** <br/> 

g) Segment **other_low_incomes** obsahuje klientov, ktorí majú všetky platby nižšie ako min. dôchodok. 


