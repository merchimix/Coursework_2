# Курсовая работа № 2
## Аналитика. Начальный уровень

Курсовая работа состоит из двух частей – обязательной и дополнительной. **Для зачета необходимо выполнение только первой части.** Выполнение второй части может потребовать дополнительные знания Python.

- [Часть первая](#Часть-первая)
- [Часть вторая](#Часть-вторая)

## Часть первая


Перед вами стоит задача – подготовить аналитический отчет для HR-отдела. На основании проведенной аналитики предполагается составить рекомендации для отдела кадров по стратегии набора персонала, а также по взаимодействию с уже имеющимися сотрудниками.
<br><br> В базе данных лежит набор таблиц, которые содержат данные о сотрудниках вымышленной компании.
Сделайте обзор штата сотрудников компании. Составьте набор предметов исследования, а затем проверьте их на данных. Вся аналитика должна быть выполена с помощью SQL. Впоследствии данные можно визуализировать, однако финальные датафреймы для графиков также должны быть подготовлены с помощью SQL. <br><br>

Примеры гипотез:
1. Есть зависимость между `perfomance score` и тем, под чьим руководством работает сотрудник.
2. Есть зависимость между продолжительностью работы в компании и семейным положением сотрудника.
2. Есть зависимость между продолжительностью работы в компании и возрастом сотрудника.

<br><br>
Параметры для подключения следующие: хост – `dsstudents.skillbox.ru`, порт – `5432`, имя базы данных – `human_resources`, пользователь – `readonly`, пароль – `6hajV34RTQfmxhS`. Таблицы, доступные для анализа, – `hr_dataset`, `production_staff`, `recruiting_costs`, `salary_grid`.


```python
# vk_token = "709d6bee4295919acf67d050343196f7d56e03c3b5ca9dd13dbfd701e3481d56e5cef785f80b68579e764"


# url = 'https://api.vk.com/method/wall.get?access_token=' + str(vk_token) + '&owner_id=-66669811&count=1&v=5.92'
# response = getjson(url)

```


```python
#Импортируем все нужные библиотеки
import psycopg2
import sqlalchemy
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

```


```python
#Соединение с базой данной
connection = 'postgresql+psycopg2://'\
+'readonly'\
+':6hajV34RTQfmxhS'\
+'@dsstudents'\
+'.skillbox.ru:5432'\
+'/human_resources'

#Движок базы данной 
engine = sqlalchemy.create_engine(connection)

#Объект подключения к БД
conn = engine.connect()

#Создаём инспектора и проверям отображение таблиц
inspector =  sqlalchemy.inspect(engine)
inspector.get_table_names()

```




    ['hr_dataset', 'production_staff', 'recruiting_costs', 'salary_grid']




```python
#Настройка отображения для того чтобы было видно все колонки
pd.set_option('display.max_columns', None)

#Вывод таблиц для понимания содержания
pd.read_sql(
    'SELECT * FROM hr_dataset', 
    conn)

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
      <th>id</th>
      <th>Employee Name</th>
      <th>Employee Number</th>
      <th>marriedid</th>
      <th>maritalstatusid</th>
      <th>genderid</th>
      <th>empstatus_id</th>
      <th>deptid</th>
      <th>perf_scoreid</th>
      <th>age</th>
      <th>Pay Rate</th>
      <th>state</th>
      <th>zip</th>
      <th>dob</th>
      <th>sex</th>
      <th>maritaldesc</th>
      <th>citizendesc</th>
      <th>Hispanic/Latino</th>
      <th>racedesc</th>
      <th>Date of Hire</th>
      <th>Days Employed</th>
      <th>Date of Termination</th>
      <th>Reason For Term</th>
      <th>Employment Status</th>
      <th>department</th>
      <th>position</th>
      <th>Manager Name</th>
      <th>Employee Source</th>
      <th>Performance Score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Brown, Mia</td>
      <td>1103024456</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>30</td>
      <td>28.50</td>
      <td>MA</td>
      <td>1450</td>
      <td>1987-11-24</td>
      <td>Female</td>
      <td>Married</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>Black or African American</td>
      <td>2008-10-27</td>
      <td>3317</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Admin Offices</td>
      <td>Accountant I</td>
      <td>Brandon R. LeBlanc</td>
      <td>Diversity Job Fair</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>LaRotonda, William</td>
      <td>1106026572</td>
      <td>0</td>
      <td>2</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>34</td>
      <td>23.00</td>
      <td>MA</td>
      <td>1460</td>
      <td>1984-04-26</td>
      <td>Male</td>
      <td>Divorced</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>Black or African American</td>
      <td>2014-01-06</td>
      <td>1420</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Admin Offices</td>
      <td>Accountant I</td>
      <td>Brandon R. LeBlanc</td>
      <td>Website Banner Ads</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Steans, Tyrone</td>
      <td>1302053333</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>31</td>
      <td>29.00</td>
      <td>MA</td>
      <td>2703</td>
      <td>1986-09-01</td>
      <td>Male</td>
      <td>Single</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>White</td>
      <td>2014-09-29</td>
      <td>1154</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Admin Offices</td>
      <td>Accountant I</td>
      <td>Brandon R. LeBlanc</td>
      <td>Internet Search</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Howard, Estelle</td>
      <td>1211050782</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>9</td>
      <td>32</td>
      <td>21.50</td>
      <td>MA</td>
      <td>2170</td>
      <td>1985-09-16</td>
      <td>Female</td>
      <td>Married</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>White</td>
      <td>2015-02-16</td>
      <td>58</td>
      <td>2015-04-15</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Admin Offices</td>
      <td>Administrative Assistant</td>
      <td>Brandon R. LeBlanc</td>
      <td>Pay Per Click - Google</td>
      <td>N/A- too early to review</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Singh, Nan</td>
      <td>1307059817</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>9</td>
      <td>30</td>
      <td>16.56</td>
      <td>MA</td>
      <td>2330</td>
      <td>1988-05-19</td>
      <td>Female</td>
      <td>Single</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>White</td>
      <td>2015-05-01</td>
      <td>940</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Admin Offices</td>
      <td>Administrative Assistant</td>
      <td>Brandon R. LeBlanc</td>
      <td>Website Banner Ads</td>
      <td>N/A- too early to review</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>305</th>
      <td>306</td>
      <td>Navathe, Kurt</td>
      <td>1009919960</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>48</td>
      <td>52.25</td>
      <td>MA</td>
      <td>2056</td>
      <td>1970-04-25</td>
      <td>Male</td>
      <td>Single</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>Asian</td>
      <td>2017-02-10</td>
      <td>289</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>IT/IS</td>
      <td>Senior BI Developer</td>
      <td>Brian Champaigne</td>
      <td>Indeed</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>306</th>
      <td>307</td>
      <td>Wang, Charlie</td>
      <td>1009919970</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>36</td>
      <td>51.00</td>
      <td>MA</td>
      <td>1887</td>
      <td>1981-07-08</td>
      <td>Male</td>
      <td>Single</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>Asian</td>
      <td>2017-02-15</td>
      <td>284</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>IT/IS</td>
      <td>Senior BI Developer</td>
      <td>Brian Champaigne</td>
      <td>Indeed</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>307</th>
      <td>308</td>
      <td>Smith, Jason</td>
      <td>1009919980</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>34</td>
      <td>46.00</td>
      <td>MA</td>
      <td>2045</td>
      <td>1983-09-04</td>
      <td>Male</td>
      <td>Single</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>White</td>
      <td>2017-02-15</td>
      <td>284</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>IT/IS</td>
      <td>BI Developer</td>
      <td>Brian Champaigne</td>
      <td>Indeed</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>308</th>
      <td>309</td>
      <td>Westinghouse, Matthew</td>
      <td>1009919990</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>30</td>
      <td>45.00</td>
      <td>MA</td>
      <td>2134</td>
      <td>1987-10-24</td>
      <td>Male</td>
      <td>Married</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>White</td>
      <td>2017-04-20</td>
      <td>220</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>IT/IS</td>
      <td>BI Developer</td>
      <td>Brian Champaigne</td>
      <td>Indeed</td>
      <td>Fully Meets</td>
    </tr>
    <tr>
      <th>309</th>
      <td>310</td>
      <td>Hubert, Robert</td>
      <td>1009920000</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>28</td>
      <td>45.00</td>
      <td>MA</td>
      <td>2134</td>
      <td>1989-06-30</td>
      <td>Male</td>
      <td>Married</td>
      <td>US Citizen</td>
      <td>No</td>
      <td>Black or African American</td>
      <td>2017-04-20</td>
      <td>220</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>IT/IS</td>
      <td>BI Developer</td>
      <td>Brian Champaigne</td>
      <td>Indeed</td>
      <td>Fully Meets</td>
    </tr>
  </tbody>
</table>
<p>310 rows × 29 columns</p>
</div>




```python
df99 = pd.read_sql(
    'SELECT citizendesc, sex FROM hr_dataset', 
    conn)
df99.groupby('citizendesc').count()
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
      <th>sex</th>
    </tr>
    <tr>
      <th>citizendesc</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Eligible NonCitizen</th>
      <td>12</td>
    </tr>
    <tr>
      <th>Non-Citizen</th>
      <td>4</td>
    </tr>
    <tr>
      <th>US Citizen</th>
      <td>294</td>
    </tr>
  </tbody>
</table>
</div>




```python
pd.read_sql(
    'SELECT * FROM production_staff',
    conn)
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
      <th>id</th>
      <th>Employee Name</th>
      <th>Race Desc</th>
      <th>Date of Hire</th>
      <th>TermDate</th>
      <th>Reason for Term</th>
      <th>Employment Status</th>
      <th>Department</th>
      <th>Position</th>
      <th>Pay</th>
      <th>Manager Name</th>
      <th>Performance Score</th>
      <th>Abutments/Hour Wk 1</th>
      <th>Abutments/Hour Wk 2</th>
      <th>Daily Error Rate</th>
      <th>90-day Complaints</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Albert, Michael</td>
      <td>White</td>
      <td>2011-08-01</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Production</td>
      <td>Production Manager</td>
      <td>$54.50</td>
      <td>Elisa Bramante</td>
      <td>Fully Meets</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Bozzi, Charles</td>
      <td>Asian</td>
      <td>2013-09-30</td>
      <td>2014-08-07</td>
      <td>retiring</td>
      <td>Voluntarily Terminated</td>
      <td>Production</td>
      <td>Production Manager</td>
      <td>$50.50</td>
      <td>Elisa Bramante</td>
      <td>Fully Meets</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Butler, Webster  L</td>
      <td>White</td>
      <td>2016-01-28</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Production</td>
      <td>Production Manager</td>
      <td>$55.00</td>
      <td>Elisa Bramante</td>
      <td>Exceeds</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Dunn, Amy</td>
      <td>White</td>
      <td>2014-09-18</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Production</td>
      <td>Production Manager</td>
      <td>$51.00</td>
      <td>Elisa Bramante</td>
      <td>Fully Meets</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Gray, Elijiah</td>
      <td>White</td>
      <td>2015-06-02</td>
      <td>None</td>
      <td>N/A - still employed</td>
      <td>Active</td>
      <td>Production</td>
      <td>Production Manager</td>
      <td>$54.00</td>
      <td>Elisa Bramante</td>
      <td>Fully Meets</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>251</th>
      <td>252</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>252</th>
      <td>253</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>253</th>
      <td>254</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>254</th>
      <td>255</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>255</th>
      <td>256</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>256 rows × 16 columns</p>
</div>




```python
pd.read_sql(
    'SELECT * from recruiting_costs', 
    conn)
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
      <th>id</th>
      <th>Employment Source</th>
      <th>January</th>
      <th>February</th>
      <th>March</th>
      <th>April</th>
      <th>May</th>
      <th>June</th>
      <th>July</th>
      <th>August</th>
      <th>September</th>
      <th>October</th>
      <th>November</th>
      <th>December</th>
      <th>Total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Billboard</td>
      <td>520</td>
      <td>520</td>
      <td>520</td>
      <td>520</td>
      <td>0</td>
      <td>0</td>
      <td>612</td>
      <td>612</td>
      <td>729</td>
      <td>749</td>
      <td>910</td>
      <td>500</td>
      <td>6192</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Careerbuilder</td>
      <td>410</td>
      <td>410</td>
      <td>410</td>
      <td>820</td>
      <td>820</td>
      <td>410</td>
      <td>410</td>
      <td>820</td>
      <td>820</td>
      <td>1230</td>
      <td>820</td>
      <td>410</td>
      <td>7790</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Company Intranet - Partner</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Diversity Job Fair</td>
      <td>0</td>
      <td>5129</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>4892</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>10021</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Employee Referral</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Glassdoor</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Information Session</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Internet Search</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>MBTA ads</td>
      <td>640</td>
      <td>640</td>
      <td>640</td>
      <td>640</td>
      <td>640</td>
      <td>640</td>
      <td>640</td>
      <td>1300</td>
      <td>1300</td>
      <td>1300</td>
      <td>1300</td>
      <td>1300</td>
      <td>10980</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Monster.com</td>
      <td>500</td>
      <td>500</td>
      <td>500</td>
      <td>440</td>
      <td>500</td>
      <td>500</td>
      <td>440</td>
      <td>500</td>
      <td>440</td>
      <td>440</td>
      <td>500</td>
      <td>500</td>
      <td>5760</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Newspager/Magazine</td>
      <td>629</td>
      <td>510</td>
      <td>293</td>
      <td>810</td>
      <td>642</td>
      <td>675</td>
      <td>707</td>
      <td>740</td>
      <td>772</td>
      <td>805</td>
      <td>838</td>
      <td>870</td>
      <td>8291</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>On-campus Recruiting</td>
      <td>0</td>
      <td>0</td>
      <td>2500</td>
      <td>0</td>
      <td>0</td>
      <td>2500</td>
      <td>0</td>
      <td>0</td>
      <td>2500</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>7500</td>
    </tr>
    <tr>
      <th>12</th>
      <td>13</td>
      <td>On-line Web application</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>14</td>
      <td>Other</td>
      <td>0</td>
      <td>492</td>
      <td>0</td>
      <td>829</td>
      <td>744</td>
      <td>0</td>
      <td>610</td>
      <td>0</td>
      <td>0</td>
      <td>510</td>
      <td>0</td>
      <td>810</td>
      <td>3995</td>
    </tr>
    <tr>
      <th>14</th>
      <td>15</td>
      <td>Pay Per Click</td>
      <td>110</td>
      <td>110</td>
      <td>60</td>
      <td>121</td>
      <td>110</td>
      <td>109</td>
      <td>130</td>
      <td>146</td>
      <td>105</td>
      <td>109</td>
      <td>105</td>
      <td>110</td>
      <td>1323</td>
    </tr>
    <tr>
      <th>15</th>
      <td>16</td>
      <td>Pay Per Click - Google</td>
      <td>330</td>
      <td>330</td>
      <td>180</td>
      <td>362</td>
      <td>197</td>
      <td>152</td>
      <td>389</td>
      <td>437</td>
      <td>315</td>
      <td>327</td>
      <td>315</td>
      <td>176</td>
      <td>3509</td>
    </tr>
    <tr>
      <th>16</th>
      <td>17</td>
      <td>Professional Society</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>100</td>
      <td>1200</td>
    </tr>
    <tr>
      <th>17</th>
      <td>18</td>
      <td>Search Engine - Google Bing Yahoo</td>
      <td>330</td>
      <td>410</td>
      <td>388</td>
      <td>372</td>
      <td>472</td>
      <td>412</td>
      <td>416</td>
      <td>495</td>
      <td>619</td>
      <td>502</td>
      <td>389</td>
      <td>378</td>
      <td>5183</td>
    </tr>
    <tr>
      <th>18</th>
      <td>19</td>
      <td>Social Networks - Facebook Twitter etc</td>
      <td>420</td>
      <td>481</td>
      <td>452</td>
      <td>479</td>
      <td>392</td>
      <td>508</td>
      <td>578</td>
      <td>466</td>
      <td>389</td>
      <td>439</td>
      <td>491</td>
      <td>478</td>
      <td>5573</td>
    </tr>
    <tr>
      <th>19</th>
      <td>20</td>
      <td>Vendor Referral</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>21</td>
      <td>Website Banner Ads</td>
      <td>400</td>
      <td>400</td>
      <td>300</td>
      <td>388</td>
      <td>592</td>
      <td>610</td>
      <td>620</td>
      <td>669</td>
      <td>718</td>
      <td>767</td>
      <td>816</td>
      <td>865</td>
      <td>7143</td>
    </tr>
    <tr>
      <th>21</th>
      <td>22</td>
      <td>Word of Mouth</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
pd.read_sql(
    'SELECT * FROM salary_grid', 
    conn)
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
      <th>id</th>
      <th>Position</th>
      <th>Salary Min</th>
      <th>Salary Mid</th>
      <th>Salary Max</th>
      <th>Hourly Min</th>
      <th>Hourly Mid</th>
      <th>Hourly Max</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Administrative Assistant</td>
      <td>30000</td>
      <td>40000</td>
      <td>50000</td>
      <td>14.42</td>
      <td>19.23</td>
      <td>24.04</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Sr. Administrative Assistant</td>
      <td>35000</td>
      <td>45000</td>
      <td>55000</td>
      <td>16.83</td>
      <td>21.63</td>
      <td>26.44</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Accountant I</td>
      <td>42274</td>
      <td>51425</td>
      <td>62299</td>
      <td>20.32</td>
      <td>24.72</td>
      <td>29.95</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Accountant II</td>
      <td>50490</td>
      <td>62158</td>
      <td>74658</td>
      <td>24.27</td>
      <td>29.88</td>
      <td>35.89</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Sr. Accountant</td>
      <td>63264</td>
      <td>76988</td>
      <td>92454</td>
      <td>30.42</td>
      <td>37.01</td>
      <td>44.45</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>Network Engineer</td>
      <td>50845</td>
      <td>66850</td>
      <td>88279</td>
      <td>24.44</td>
      <td>32.14</td>
      <td>42.44</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Sr. Network Engineer</td>
      <td>79428</td>
      <td>99458</td>
      <td>120451</td>
      <td>38.19</td>
      <td>47.82</td>
      <td>57.91</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>Database Administrator</td>
      <td>50569</td>
      <td>68306</td>
      <td>93312</td>
      <td>24.31</td>
      <td>32.84</td>
      <td>44.86</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>Sr. DBA</td>
      <td>92863</td>
      <td>116007</td>
      <td>139170</td>
      <td>44.65</td>
      <td>55.77</td>
      <td>66.91</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>Production Technician I</td>
      <td>30000</td>
      <td>40000</td>
      <td>50000</td>
      <td>14.42</td>
      <td>19.23</td>
      <td>24.04</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>Production Technician II</td>
      <td>38000</td>
      <td>48000</td>
      <td>58000</td>
      <td>18.27</td>
      <td>23.08</td>
      <td>27.88</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>Lead Production Technician</td>
      <td>45000</td>
      <td>55000</td>
      <td>65000</td>
      <td>21.63</td>
      <td>26.44</td>
      <td>31.25</td>
    </tr>
  </tbody>
</table>
</div>



### Гипотеза 1. Чаще уходят из компании работники определённого возраста


```python
#Создаём датасет с количеством работников и количеством уволившихся, 
#сгруппированный по годам
df = pd.read_sql('''
    SELECT age, COUNT('age') AS age_count, 
    COUNT("Date of Termination") AS Termination_count 
    FROM hr_dataset 
    GROUP BY age 
    ORDER BY age
    ''', 
conn)

#Вычисляем соотношение уволившихся к общему числу работников
df['ratio'] = df.termination_count / df.age_count 
```


```python
#Настраиваем отображение графика и размечаем его
fig, ax = plt.subplots(figsize=(10, 6))
sns.barplot(data=df, x='age', y='ratio', ax=ax)

#Проводим линию - общую медиану соотношения по всем возрастам
ax.hlines(df['ratio'].median(), 0, 39, color ="red", linestyles= 'dashed', 
                                      label=round(df['ratio'].median(),2))
ax.legend()

# Выводим подписи осей координат
ax.set_xlabel('Возраст')
ax.set_ylabel('Соотношение уволившихся к общему числу работников')

#Чуток подровняем
plt.tight_layout()
```


    
![png](output_12_0.png)
    


#### Резких выбросов, помимо "предпенсионной" группы не наблюдается
#### Наименьшую текучку показывают группы от 27 до 29 и от 38 до 42 
#### Наибольшую - от 30 до 37 и от 44 до 47

### Гипотеза 2. Лучшие кадры приходят с определенного ресурса



```python
df2 = pd.read_sql('''
                  SELECT "Employee Source", COUNT("Days Employed") AS count, 
                  AVG("Days Employed") AS Average_days
                  FROM hr_dataset hr
                  JOIN recruiting_costs rc ON  
                  hr."Employee Source" = rc."Employment Source"   
                  GROUP BY "Employee Source" 
                  ''', 
                  conn)
df2
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
      <th>Employee Source</th>
      <th>count</th>
      <th>average_days</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Careerbuilder</td>
      <td>1</td>
      <td>2428.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Other</td>
      <td>9</td>
      <td>1910.333333</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Billboard</td>
      <td>16</td>
      <td>1680.750000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MBTA ads</td>
      <td>17</td>
      <td>1580.352941</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Information Session</td>
      <td>4</td>
      <td>1551.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Website Banner Ads</td>
      <td>13</td>
      <td>1471.307692</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Professional Society</td>
      <td>20</td>
      <td>1416.600000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Monster.com</td>
      <td>24</td>
      <td>1403.000000</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Diversity Job Fair</td>
      <td>29</td>
      <td>1340.448276</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Search Engine - Google Bing Yahoo</td>
      <td>25</td>
      <td>1324.560000</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Pay Per Click - Google</td>
      <td>21</td>
      <td>1287.285714</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Internet Search</td>
      <td>6</td>
      <td>1274.166667</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Employee Referral</td>
      <td>31</td>
      <td>1252.161290</td>
    </tr>
    <tr>
      <th>13</th>
      <td>On-campus Recruiting</td>
      <td>12</td>
      <td>1214.416667</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Newspager/Magazine</td>
      <td>18</td>
      <td>1193.777778</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Glassdoor</td>
      <td>14</td>
      <td>1134.785714</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Social Networks - Facebook Twitter etc</td>
      <td>11</td>
      <td>1082.727273</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Word of Mouth</td>
      <td>13</td>
      <td>1022.923077</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Vendor Referral</td>
      <td>15</td>
      <td>1022.066667</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Company Intranet - Partner</td>
      <td>1</td>
      <td>444.000000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>On-line Web application</td>
      <td>1</td>
      <td>194.000000</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Pay Per Click</td>
      <td>1</td>
      <td>2.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Настраиваем отображение графика и размечаем его
fig, ax = plt.subplots(figsize=(10, 10))
sns.barplot(data=df2, x='average_days', y='Employee Source', ax=ax,
                                        palette='flare', orient='h')

#Проводим линию - общую медиану соотношения по всем возрастам
ax.vlines(df2['average_days'].median(), -1, 21, color ="blue", 
          linestyles= 'dashed',label=round(df2['average_days'].median()))
ax.legend()

# Выводим подписи осей координат
ax.set_xlabel('Источник')
ax.set_ylabel('Среднее количество проработанных дней')

#Чуток подровняем
plt.tight_layout()

```


    
![png](output_16_0.png)
    



```python
#Дополнительно создадим списки для более наглядной аналитики в будущем 
good_by_days = list(df2[df2['average_days'] >=\
                        df2['average_days'].median()]['Employee Source'])

bad_by_days = list(df2[df2['average_days'] <=\
                        df2['average_days'].median()]['Employee Source'])
```

### Гипотеза 3. Некоторые источники найма лучше других


```python
df3 = pd.read_sql('''
    SELECT "Employment Source",  "Total"
    FROM recruiting_costs
    ORDER BY "Total" DESC
    ''', 
conn)

```


```python
#Настраиваем отображение графика и размечаем его
fig, ax = plt.subplots(figsize=(10, 10))
sns.barplot(data=df3, x='Total', y='Employment Source',
                    ax=ax, palette='crest', orient='h')

#Проводим линию - общую медиану соотношения по всем возрастам
ax.vlines(df3['Total'].median(), -1, 21, color ="blue", 
          linestyles= 'dashed',label=round(df3['Total'].median()))
ax.legend()

# Выводим подписи осей координат
ax.set_xlabel('Источник')
ax.set_ylabel('Общая стоимость')

#Чуток подровняем
plt.tight_layout()



```


    
![png](output_20_0.png)
    



```python
#Аналогично создаём списки для анализа
expensive_by_total = list(df3[df3['Total'] >=\ 
                              df3['Total'].median()]['Employment Source'])

cheaply_by_total = list(df3[df3['Total'] <=\ 
                              df3['Total'].median()]['Employment Source'])
```


```python
#В цикле высчитаем два списка
absolutely_good, absolutely_bad = [], []

#Один содержит лучшие источники найма, с которыми, возможно, стоит улучшить взаимодействие
for i in cheaply_by_total:
    if i in good_by_days:
        absolutely_good.append(i)

#Второй худшие, с источниками, взаимодействие с которыми, возможно, стоит пересмотреть       
for i in expensive_by_total:
    if i in bad_by_days:
        absolutely_bad.append(i)   

#Выводим получившиеся списки 
print('Лучшие источники найма - ', absolutely_good, '\n' 'Худшие - ', absolutely_bad)
```

    Лучшие источники найма -  ['Pay Per Click - Google', 'Professional Society', 'Information Session'] 
    Худшие -  ['Newspager/Magazine', 'On-campus Recruiting', 'Social Networks - Facebook Twitter etc']
    

### Гипотеза 4. В Компании женщины получают за ту же работу меньшую зарплату


```python
df4_2 = pd.read_sql('''
    SELECT "position", "sex", AVG("Pay Rate") 
    FROM hr_dataset
    GROUP BY "position", "sex"
    ORDER BY position
    ''', 
conn)
df4_2
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
      <th>position</th>
      <th>sex</th>
      <th>avg</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Accountant I</td>
      <td>Male</td>
      <td>26.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Accountant I</td>
      <td>Female</td>
      <td>28.500000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Administrative Assistant</td>
      <td>Female</td>
      <td>19.520000</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Area Sales Manager</td>
      <td>Female</td>
      <td>55.083333</td>
      <td>12</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Area Sales Manager</td>
      <td>Male</td>
      <td>55.333333</td>
      <td>15</td>
    </tr>
    <tr>
      <th>5</th>
      <td>BI Developer</td>
      <td>Male</td>
      <td>45.333333</td>
      <td>3</td>
    </tr>
    <tr>
      <th>6</th>
      <td>BI Developer</td>
      <td>Female</td>
      <td>45.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>BI Director</td>
      <td>Male</td>
      <td>63.500000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>CIO</td>
      <td>Female</td>
      <td>65.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Data Architect</td>
      <td>Female</td>
      <td>55.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Database Administrator</td>
      <td>Male</td>
      <td>39.000000</td>
      <td>6</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Database Administrator</td>
      <td>Female</td>
      <td>39.885714</td>
      <td>7</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Director of Operations</td>
      <td>Female</td>
      <td>60.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Director of Sales</td>
      <td>Female</td>
      <td>60.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>IT Director</td>
      <td>Male</td>
      <td>65.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>IT Manager - DB</td>
      <td>Male</td>
      <td>41.500000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>16</th>
      <td>IT Manager - Infra</td>
      <td>Male</td>
      <td>63.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>IT Manager - Support</td>
      <td>Male</td>
      <td>64.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>IT Support</td>
      <td>Male</td>
      <td>28.990000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>IT Support</td>
      <td>Female</td>
      <td>28.296666</td>
      <td>3</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Network Engineer</td>
      <td>Male</td>
      <td>41.420000</td>
      <td>5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Network Engineer</td>
      <td>Female</td>
      <td>37.500000</td>
      <td>4</td>
    </tr>
    <tr>
      <th>22</th>
      <td>President &amp; CEO</td>
      <td>Female</td>
      <td>80.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Production Manager</td>
      <td>Female</td>
      <td>47.500000</td>
      <td>6</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Production Manager</td>
      <td>Male</td>
      <td>51.312500</td>
      <td>8</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Production Technician I</td>
      <td>Male</td>
      <td>19.061509</td>
      <td>53</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Production Technician I</td>
      <td>Female</td>
      <td>19.131928</td>
      <td>83</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Production Technician II</td>
      <td>Female</td>
      <td>25.378108</td>
      <td>37</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Production Technician II</td>
      <td>Male</td>
      <td>25.462500</td>
      <td>20</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Sales Manager</td>
      <td>Female</td>
      <td>57.125000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Sales Manager</td>
      <td>Male</td>
      <td>56.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Senior BI Developer</td>
      <td>Female</td>
      <td>50.250000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Senior BI Developer</td>
      <td>Male</td>
      <td>51.625000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Shared Services Manager</td>
      <td>Male</td>
      <td>55.000000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Software Engineer</td>
      <td>Male</td>
      <td>48.556666</td>
      <td>3</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Software Engineer</td>
      <td>Female</td>
      <td>52.329999</td>
      <td>6</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Software Engineering Manager</td>
      <td>Male</td>
      <td>27.000000</td>
      <td>1</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Sr. Accountant</td>
      <td>Female</td>
      <td>34.950001</td>
      <td>2</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Sr. DBA</td>
      <td>Male</td>
      <td>60.100000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Sr. DBA</td>
      <td>Female</td>
      <td>59.900000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Sr. Network Engineer</td>
      <td>Female</td>
      <td>54.650000</td>
      <td>2</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Sr. Network Engineer</td>
      <td>Male</td>
      <td>54.333333</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
fig, ax = plt.subplots(figsize=(15, 10))


sns.barplot(data=df4_2, x='avg', y='position', hue='sex', orient='h', ax=ax)

```




    <AxesSubplot:xlabel='avg', ylabel='position'>




    
![png](output_25_1.png)
    


### Общие выводы

-Мужчины и женщины получают примерно одинаковую зарплату  
-Наиболее привлекательный возраст нового сотрудника от 27 до 29 и от 38 до 42  
-Пересмотреть бюджетирование следующих источников:   
    'Pay Per Click - Google', 'Professional Society', 'Information Session' - в большую сторону  
    'Newspager/Magazine', 'On-campus Recruiting', 'Social Networks - Facebook Twitter etc' - в меньшую сторону

## Часть вторая

Перед вами стоит задача – подготовить аналитический ответ для SMM-отдела компании Skillbox. <br> Объектом анализа является  [паблик Skillbox Вконтакте](https://vk.com/skillbox_education). <br> <br> 
Подключитесь к  API VK и выгрузите посты со стены паблика Skillbox за интересующий период (определите самостоятельно и обоснуйте). Проанализируйте влияние различных факторов (например, времени публикации) на вовлеченность пользователей (количество лайков, комментариев, голосов в опросах). Сделайте аналитику по рубрикам (примеры рубрик: дизайн-битва, игра по управлению), которые есть в паблике. Выбрать нужные посты можно с помощью регулярных выражений. Составьте перечень рекомандаций для SMM-отдела по итогам анализа. <br> <br> 

Дополнительные инструкции по работе с API VK расположены [здесь](https://colab.research.google.com/drive/1rRaTay-OSPLAOX8V9UaFvTiAciVtp2s3).

# At work.........


```python
# vk_token = "709d6bee4295919acf67d050343196f7d56e03c3b5ca9dd13dbfd701e3481d56e5cef785f80b68579e764"


# url = 'https://api.vk.com/method/wall.get?access_token=' + str(vk_token) + '&owner_id=-66669811&count=1&v=5.92'
# response = getjson(url)

```
