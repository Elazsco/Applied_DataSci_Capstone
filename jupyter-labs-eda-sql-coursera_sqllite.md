<p style="text-align:center">
    <a href="https://skills.network" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo">
    </a>
</p>

<h1 align=center><font size = 5>Assignment: SQL Notebook for Peer Assignment</font></h1>

Estimated time needed: **60** minutes.

## Introduction
Using this Python notebook you will:

1.  Understand the Spacex DataSet
2.  Load the dataset  into the corresponding table in a Db2 database
3.  Execute SQL queries to answer assignment questions 


## Overview of the DataSet

SpaceX has gained worldwide attention for a series of historic milestones. 

It is the only private company ever to return a spacecraft from low-earth orbit, which it first accomplished in December 2010.
SpaceX advertises Falcon 9 rocket launches on its website with a cost of 62 million dollars wheras other providers cost upward of 165 million dollars each, much of the savings is because Space X can reuse the first stage. 


Therefore if we can determine if the first stage will land, we can determine the cost of a launch. 

This information can be used if an alternate company wants to bid against SpaceX for a rocket launch.

This dataset includes a record for each payload carried during a SpaceX mission into outer space.


### Download the datasets

This assignment requires you to load the spacex dataset.

In many cases the dataset to be analyzed is available as a .CSV (comma separated values) file, perhaps on the internet. Click on the link below to download and save the dataset (.CSV file):

 <a href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/data/Spacex.csv" target="_blank">Spacex DataSet</a>




```python
!pip install --force-reinstall ibm_db==3.1.0 ibm_db_sa==0.3.3
!pip install sqlalchemy==1.3.24
!pip uninstall ipython-sql -y
!pip install ipython-sql==0.4.1
```

    Collecting ibm_db==3.1.0
      Using cached ibm_db-3.1.0-cp312-cp312-linux_x86_64.whl
    Collecting ibm_db_sa==0.3.3
      Using cached ibm_db_sa-0.3.3-py3-none-any.whl
    Collecting sqlalchemy>=0.7.3 (from ibm_db_sa==0.3.3)
      Using cached sqlalchemy-2.0.41-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (9.6 kB)
    Collecting greenlet>=1 (from sqlalchemy>=0.7.3->ibm_db_sa==0.3.3)
      Using cached greenlet-3.2.3-cp312-cp312-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl.metadata (4.1 kB)
    Collecting typing-extensions>=4.6.0 (from sqlalchemy>=0.7.3->ibm_db_sa==0.3.3)
      Using cached typing_extensions-4.14.1-py3-none-any.whl.metadata (3.0 kB)
    Using cached sqlalchemy-2.0.41-cp312-cp312-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (3.3 MB)
    Using cached greenlet-3.2.3-cp312-cp312-manylinux_2_24_x86_64.manylinux_2_28_x86_64.whl (605 kB)
    Using cached typing_extensions-4.14.1-py3-none-any.whl (43 kB)
    Installing collected packages: ibm_db, typing-extensions, greenlet, sqlalchemy, ibm_db_sa
      Attempting uninstall: ibm_db
        Found existing installation: ibm_db 3.1.0
        Uninstalling ibm_db-3.1.0:
          Successfully uninstalled ibm_db-3.1.0
      Attempting uninstall: typing-extensions
        Found existing installation: typing_extensions 4.14.1
        Uninstalling typing_extensions-4.14.1:
          Successfully uninstalled typing_extensions-4.14.1
      Attempting uninstall: greenlet
        Found existing installation: greenlet 3.2.3
        Uninstalling greenlet-3.2.3:
          Successfully uninstalled greenlet-3.2.3
      Attempting uninstall: sqlalchemy
        Found existing installation: SQLAlchemy 1.3.24
        Uninstalling SQLAlchemy-1.3.24:
          Successfully uninstalled SQLAlchemy-1.3.24
      Attempting uninstall: ibm_db_sa
        Found existing installation: ibm_db_sa 0.3.3
        Uninstalling ibm_db_sa-0.3.3:
          Successfully uninstalled ibm_db_sa-0.3.3
    Successfully installed greenlet-3.2.3 ibm_db-3.1.0 ibm_db_sa-0.3.3 sqlalchemy-2.0.41 typing-extensions-4.14.1
    Collecting sqlalchemy==1.3.24
      Using cached SQLAlchemy-1.3.24-cp312-cp312-linux_x86_64.whl
    Installing collected packages: sqlalchemy
      Attempting uninstall: sqlalchemy
        Found existing installation: SQLAlchemy 2.0.41
        Uninstalling SQLAlchemy-2.0.41:
          Successfully uninstalled SQLAlchemy-2.0.41
    [31mERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
    jupyterhub 5.2.1 requires SQLAlchemy>=1.4.1, but you have sqlalchemy 1.3.24 which is incompatible.[0m[31m
    [0mSuccessfully installed sqlalchemy-1.3.24
    Found existing installation: ipython-sql 0.4.1
    Uninstalling ipython-sql-0.4.1:
      Successfully uninstalled ipython-sql-0.4.1
    Collecting ipython-sql==0.4.1
      Using cached ipython_sql-0.4.1-py3-none-any.whl.metadata (17 kB)
    Requirement already satisfied: prettytable<1 in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (0.7.2)
    Requirement already satisfied: ipython>=1.0 in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (8.31.0)
    Requirement already satisfied: sqlalchemy>=0.6.7 in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (1.3.24)
    Requirement already satisfied: sqlparse in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (0.5.3)
    Requirement already satisfied: six in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (1.17.0)
    Requirement already satisfied: ipython-genutils>=0.1.0 in /opt/conda/lib/python3.12/site-packages (from ipython-sql==0.4.1) (0.2.0)
    Requirement already satisfied: decorator in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (5.1.1)
    Requirement already satisfied: jedi>=0.16 in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (0.19.2)
    Requirement already satisfied: matplotlib-inline in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (0.1.7)
    Requirement already satisfied: pexpect>4.3 in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (4.9.0)
    Requirement already satisfied: prompt_toolkit<3.1.0,>=3.0.41 in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (3.0.50)
    Requirement already satisfied: pygments>=2.4.0 in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (2.19.1)
    Requirement already satisfied: stack_data in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (0.6.3)
    Requirement already satisfied: traitlets>=5.13.0 in /opt/conda/lib/python3.12/site-packages (from ipython>=1.0->ipython-sql==0.4.1) (5.14.3)
    Requirement already satisfied: parso<0.9.0,>=0.8.4 in /opt/conda/lib/python3.12/site-packages (from jedi>=0.16->ipython>=1.0->ipython-sql==0.4.1) (0.8.4)
    Requirement already satisfied: ptyprocess>=0.5 in /opt/conda/lib/python3.12/site-packages (from pexpect>4.3->ipython>=1.0->ipython-sql==0.4.1) (0.7.0)
    Requirement already satisfied: wcwidth in /opt/conda/lib/python3.12/site-packages (from prompt_toolkit<3.1.0,>=3.0.41->ipython>=1.0->ipython-sql==0.4.1) (0.2.13)
    Requirement already satisfied: executing>=1.2.0 in /opt/conda/lib/python3.12/site-packages (from stack_data->ipython>=1.0->ipython-sql==0.4.1) (2.1.0)
    Requirement already satisfied: asttokens>=2.1.0 in /opt/conda/lib/python3.12/site-packages (from stack_data->ipython>=1.0->ipython-sql==0.4.1) (3.0.0)
    Requirement already satisfied: pure_eval in /opt/conda/lib/python3.12/site-packages (from stack_data->ipython>=1.0->ipython-sql==0.4.1) (0.2.3)
    Using cached ipython_sql-0.4.1-py3-none-any.whl (21 kB)
    Installing collected packages: ipython-sql
    Successfully installed ipython-sql-0.4.1


### Connect to the database

Let us first load the SQL extension and establish a connection with the database



```python
%load_ext sql
```

    The sql extension is already loaded. To reload it, use:
      %reload_ext sql



```python
import csv, sqlite3

con = sqlite3.connect("my_data1.db")
cur = con.cursor()
```


```python
%sql sqlite:///my_data1.db
```


```python
import pandas as pd
df = pd.read_csv("https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/data/Spacex.csv")
df.to_sql('SPACEXTBL", con, if_exists='replace', index=False, method="multi")
```


      Cell In[71], line 3
        df.to_sql('SPACEXTBL", con, if_exists='replace', index=False, method="multi")
                                                      ^
    SyntaxError: unterminated string literal (detected at line 3)




```python

```


```python

```




    101



**Note:This below code is added to remove blank rows from table**



```python

```

     * sqlite:///my_data1.db
    Done.





    []




```python

```

     * sqlite:///my_data1.db
    Done.





    []



## Tasks

Now write and execute SQL queries to solve the assignment tasks.

**Note: If the column names are in mixed case enclose it in double quotes
   For Example "Landing_Outcome"**

### Task 1




##### Display the names of the unique launch sites  in the space mission



```python
%sql SELECT DISTINCT LAUNCH_SITE as "Launch_Sites" FROM SPACEX;
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT DISTINCT LAUNCH_SITE as "Launch_Sites" FROM SPACEX;]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)



### Task 2


#####  Display 5 records where launch sites begin with the string 'CCA' 



```python
%sql SELECT * FROM SPACEX WHERE LAUNCH_STIE LIKE 'CCA%' LIMIT 20;
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT * FROM SPACEX WHERE LAUNCH_STIE LIKE 'CCA%' LIMIT 20;]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 3




##### Display the total payload mass carried by boosters launched by NASA (CRS)



```python
%sql SELECT SUM(payload_mass_kg_) FROM SPACEX WHERE customer = 'NASA (CRS)' ;
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT SUM(payload_mass_kg_) FROM SPACEX WHERE customer = 'NASA (CRS)' ;]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 4




##### Display average payload mass carried by booster version F9 v1.1



```python
%sql SELECT AVG(payload_mass_kg_) FROM SPACEX WHERE booster_version = 'F9 v1.1';
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT AVG(payload_mass_kg_) FROM SPACEX WHERE booster_version = 'F9 v1.1' ;]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 5

##### List the date when the first succesful landing outcome in ground pad was acheived.


_Hint:Use min function_ 



```python
%sql SELECT MIN(DATE) AS "First Successful Landing" FROM SPACEX WHERE landing_outcome = 'Success (ground pad)';
```

### Task 6

##### List the names of the boosters which have success in drone ship and have payload mass greater than 4000 but less than 6000



```python
%sql SELECT booster_version FROM SPACEX WHERE landing_outcome = 'Success (drone ship)' AND payload_mass_kg_ > 4000 AND payload_mass_kg_ < 6000
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT booster_version FROM SPACEX WHERE landing_outcome = 'Success (drone ship)' AND payload_mass_kg_ > 4000 AND payload_mass_kg_ < 6000]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 7




##### List the total number of successful and failure mission outcomes



```python
%sql SELECT COUNT(mission_outcome) FROM SPACEX WHERE mission_outcome LIKE 'Success%'
%sql SELECT COUNT(mission_outcome) FROM SPACEX WHERE mission_outcome LIKE 'Fail%'
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT COUNT(mission_outcome) FROM SPACEX WHERE mission_outcome LIKE 'Success%']
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 8



##### List all the booster_versions that have carried the maximum payload mass, using a subquery with a suitable aggregate function.



```python
%sql SELECT DISTINCT booster version AS "Booster Versions which carried the Maximum Payload Mass" FROM SPACEX \
WHERE payload_mass_kg_ = (SELECT MAX(payload_mass_kg_) FROM SPACEX);
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) near "AS": syntax error
    [SQL: SELECT DISTINCT booster version AS "Booster Versions which carried the Maximum Payload Mass" FROM SPACEX WHERE payload_mass_kg_ = (SELECT MAX(payload_mass_kg_) FROM SPACEX);]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 9


##### List the records which will display the month names, failure landing_outcomes in drone ship ,booster versions, launch_site for the months in year 2015.

**Note: SQLLite does not support monthnames. So you need to use  substr(Date, 6,2) as month to get the months and substr(Date,0,5)='2015' for year.**



```python
%sql SELECT booster_version, launch_stie FROM SPACEX WHERE DATE LIKE '2015-%' AND \
landing_outcome = 'Failure (drone ship)';
```

     * sqlite:///my_data1.db
    (sqlite3.OperationalError) no such table: SPACEX
    [SQL: SELECT booster_version, launch_stie FROM SPACEX WHERE DATE LIKE '2015-%' AND landing_outcome = 'Failure (drone ship)' ;]
    (Background on this error at: https://sqlalche.me/e/20/e3q8)


### Task 10




##### Rank the count of landing outcomes (such as Failure (drone ship) or Success (ground pad)) between the date 2010-06-04 and 2017-03-20, in descending order.



```python
%sql SELECT landing_outcome as "Landing Outcome", COUNT(land_outcome) AS "Total Count" FROM SPACEX \ 
WHERE DATE BETWEEN '2010-06-04' AND '2017-03-20' \
GROUP BY landing_outcome \
ORDER BY COUNT(landing_outcome) DESC ; 
```


      Cell In[82], line 2
        WHERE DATE BETWEEN '2010-06-04' AND '2017-03-20' \
              ^
    SyntaxError: invalid syntax



### Reference Links

* <a href ="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20String%20Patterns%20-%20Sorting%20-%20Grouping/instructional-labs.md.html?origin=www.coursera.org">Hands-on Lab : String Patterns, Sorting and Grouping</a>  

*  <a  href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20Built-in%20functions%20/Hands-on_Lab__Built-in_Functions.md.html?origin=www.coursera.org">Hands-on Lab: Built-in functions</a>

*  <a  href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20Sub-queries%20and%20Nested%20SELECTs%20/instructional-labs.md.html?origin=www.coursera.org">Hands-on Lab : Sub-queries and Nested SELECT Statements</a>

*   <a href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Module%205/DB0201EN-Week3-1-3-SQLmagic.ipynb">Hands-on Tutorial: Accessing Databases with SQL magic</a>

*  <a href= "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Module%205/DB0201EN-Week3-1-4-Analyzing.ipynb">Hands-on Lab: Analyzing a real World Data Set</a>




## Author(s)

<h4> Lakshmi Holla </h4>


## Other Contributors

<h4> Rav Ahuja </h4>


<!--
## Change log
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2024-07-10 | 1.1 |Anita Verma | Changed Version|
| 2021-07-09 | 0.2 |Lakshmi Holla | Changes made in magic sql|
| 2021-05-20 | 0.1 |Lakshmi Holla | Created Initial Version |
-->


## <h3 align="center"> © IBM Corporation 2021. All rights reserved. <h3/>

