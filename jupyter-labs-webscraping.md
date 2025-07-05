<p style="text-align:center">
    <a href="https://skills.network" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo">
    </a>
</p>


# **Space X  Falcon 9 First Stage Landing Prediction**


## Web scraping Falcon 9 and Falcon Heavy Launches Records from Wikipedia


Estimated time needed: **40** minutes


In this lab, you will be performing web scraping to collect Falcon 9 historical launch records from a Wikipedia page titled `List of Falcon 9 and Falcon Heavy launches`

https://en.wikipedia.org/wiki/List_of_Falcon_9_and_Falcon_Heavy_launches


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_1_L2/images/Falcon9_rocket_family.svg)


Falcon 9 first stage will land successfully


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0701EN-SkillsNetwork/api/Images/landing_1.gif)


Several examples of an unsuccessful landing are shown here:


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DS0701EN-SkillsNetwork/api/Images/crash.gif)


More specifically, the launch records are stored in a HTML table shown below:


![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_1_L2/images/falcon9-launches-wiki.png)


  ## Objectives
Web scrap Falcon 9 launch records with `BeautifulSoup`: 
- Extract a Falcon 9 launch records HTML table from Wikipedia
- Parse the table and convert it into a Pandas data frame


First let's import required packages for this lab



```python
!pip3 install beautifulsoup4
!pip3 install requests
```

    Requirement already satisfied: beautifulsoup4 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (4.11.1)
    Requirement already satisfied: soupsieve>1.2 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from beautifulsoup4) (2.3.2.post1)
    Requirement already satisfied: requests in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (2.29.0)
    Requirement already satisfied: charset-normalizer<4,>=2 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests) (3.1.0)
    Requirement already satisfied: idna<4,>=2.5 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests) (3.4)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests) (1.26.15)
    Requirement already satisfied: certifi>=2017.4.17 in /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages (from requests) (2023.5.7)



```python
import sys

import requests
from bs4 import BeautifulSoup
import re
import unicodedata
import pandas as pd
```

and we will provide some helper functions for you to process web scraped HTML table



```python
def date_time(table_cells):
    """
    This function returns the data and time from the HTML  table cell
    Input: the  element of a table data cell extracts extra row
    """
    return [data_time.strip() for data_time in list(table_cells.strings)][0:2]

def booster_version(table_cells):
    """
    This function returns the booster version from the HTML  table cell 
    Input: the  element of a table data cell extracts extra row
    """
    out=''.join([booster_version for i,booster_version in enumerate( table_cells.strings) if i%2==0][0:-1])
    return out

def landing_status(table_cells):
    """
    This function returns the landing status from the HTML table cell 
    Input: the  element of a table data cell extracts extra row
    """
    out=[i for i in table_cells.strings][0]
    return out


def get_mass(table_cells):
    mass=unicodedata.normalize("NFKD", table_cells.text).strip()
    if mass:
        mass.find("kg")
        new_mass=mass[0:mass.find("kg")+2]
    else:
        new_mass=0
    return new_mass


def extract_column_from_header(row):
    """
    This function returns the landing status from the HTML table cell 
    Input: the  element of a table data cell extracts extra row
    """
    if (row.br):
        row.br.extract()
    if row.a:
        row.a.extract()
    if row.sup:
        row.sup.extract()
        
    colunm_name = ' '.join(row.contents)
    
    # Filter the digit and empty names
    if not(colunm_name.strip().isdigit()):
        colunm_name = colunm_name.strip()
        return colunm_name    

```

To keep the lab tasks consistent, you will be asked to scrape the data from a snapshot of the  `List of Falcon 9 and Falcon Heavy launches` Wikipage updated on
`9th June 2021`



```python
static_url = "https://en.wikipedia.org/w/index.php?title=List_of_Falcon_9_and_Falcon_Heavy_launches&oldid=1027686922"
```

Next, request the HTML page from the above URL and get a `response` object


### TASK 1: Request the Falcon9 Launch Wiki page from its URL


First, let's perform an HTTP GET method to request the Falcon9 Launch HTML page, as an HTTP response.



```python
# use requests.get() method with the provided static_url
html_data = requests.get(static_url)
# assign the response to a object
html_data.status_code
```




    200



Create a `BeautifulSoup` object from the HTML `response`



```python
# Use BeautifulSoup() to create a BeautifulSoup object from a response text content
soup = BeautifulSoup(html_data.text, 'html.parser')
```

Print the page title to verify if the `BeautifulSoup` object was created properly 



```python
# Use soup.title attribute
soup.title
```




    <title>List of Falcon 9 and Falcon Heavy launches - Wikipedia</title>



### TASK 2: Extract all column/variable names from the HTML table header


Next, we want to collect all relevant column names from the HTML table header


Let's try to find all tables on the wiki page first. If you need to refresh your memory about `BeautifulSoup`, please check the external reference link towards the end of this lab



```python
# Use the find_all function in the BeautifulSoup object, with element type `table`
html_tables = soup.find_all('table')
# Assign the result to a list called `html_tables`
first_launch_table = html_tables[2]
print(first_launch_table)
```

    <table class="wikitable plainrowheaders collapsible" style="width: 100%;">
    <tbody><tr>
    <th scope="col">Flight No.
    </th>
    <th scope="col">Date and<br/>time (<a href="/wiki/Coordinated_Universal_Time" title="Coordinated Universal Time">UTC</a>)
    </th>
    <th scope="col"><a href="/wiki/List_of_Falcon_9_first-stage_boosters" title="List of Falcon 9 first-stage boosters">Version,<br/>Booster</a> <sup class="reference" id="cite_ref-booster_11-0"><a href="#cite_note-booster-11"><span class="cite-bracket">[</span>b<span class="cite-bracket">]</span></a></sup>
    </th>
    <th scope="col">Launch site
    </th>
    <th scope="col">Payload<sup class="reference" id="cite_ref-Dragon_12-0"><a href="#cite_note-Dragon-12"><span class="cite-bracket">[</span>c<span class="cite-bracket">]</span></a></sup>
    </th>
    <th scope="col">Payload mass
    </th>
    <th scope="col">Orbit
    </th>
    <th scope="col">Customer
    </th>
    <th scope="col">Launch<br/>outcome
    </th>
    <th scope="col"><a href="/wiki/Falcon_9_first-stage_landing_tests" title="Falcon 9 first-stage landing tests">Booster<br/>landing</a>
    </th></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">1
    </th>
    <td>4 June 2010,<br/>18:45
    </td>
    <td><a href="/wiki/Falcon_9_v1.0" title="Falcon 9 v1.0">F9 v1.0</a><sup class="reference" id="cite_ref-MuskMay2012_13-0"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B0003.1<sup class="reference" id="cite_ref-block_numbers_14-0"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/Dragon_Spacecraft_Qualification_Unit" title="Dragon Spacecraft Qualification Unit">Dragon Spacecraft Qualification Unit</a>
    </td>
    <td>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a>
    </td>
    <td><a href="/wiki/SpaceX" title="SpaceX">SpaceX</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success
    </td>
    <td class="table-failure" style="background: #FFC7C7; color:black; vertical-align: middle; text-align: center;">Failure<sup class="reference" id="cite_ref-ns20110930_15-0"><a href="#cite_note-ns20110930-15"><span class="cite-bracket">[</span>9<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-16"><a href="#cite_note-16"><span class="cite-bracket">[</span>10<span class="cite-bracket">]</span></a></sup><br/><small>(parachute)</small>
    </td></tr>
    <tr>
    <td colspan="9">First flight of Falcon 9 v1.0.<sup class="reference" id="cite_ref-sfn20100604_17-0"><a href="#cite_note-sfn20100604-17"><span class="cite-bracket">[</span>11<span class="cite-bracket">]</span></a></sup> Used a boilerplate version of Dragon capsule which was not designed to separate from the second stage.<small>(<a href="#First_flight_of_Falcon_9">more details below</a>)</small> Attempted to recover the first stage by parachuting it into the ocean, but it burned up on reentry, before the parachutes even deployed.<sup class="reference" id="cite_ref-parachute_18-0"><a href="#cite_note-parachute-18"><span class="cite-bracket">[</span>12<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">2
    </th>
    <td>8 December 2010,<br/>15:43<sup class="reference" id="cite_ref-spaceflightnow_Clark_Launch_Report_19-0"><a href="#cite_note-spaceflightnow_Clark_Launch_Report-19"><span class="cite-bracket">[</span>13<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.0" title="Falcon 9 v1.0">F9 v1.0</a><sup class="reference" id="cite_ref-MuskMay2012_13-1"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B0004.1<sup class="reference" id="cite_ref-block_numbers_14-1"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_Dragon" title="SpaceX Dragon">Dragon</a> <a class="mw-redirect" href="/wiki/COTS_Demo_Flight_1" title="COTS Demo Flight 1">demo flight C1</a><br/>(Dragon C101)
    </td>
    <td>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a href="/wiki/International_Space_Station" title="International Space Station">ISS</a>)
    </td>
    <td><style data-mw-deduplicate="TemplateStyles:r1126788409">.mw-parser-output .plainlist ol,.mw-parser-output .plainlist ul{line-height:inherit;list-style:none;margin:0;padding:0}.mw-parser-output .plainlist ol li,.mw-parser-output .plainlist ul li{margin-bottom:0}</style><div class="plainlist">
    <ul><li><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Orbital_Transportation_Services" title="Commercial Orbital Transportation Services">COTS</a>)</li>
    <li><a href="/wiki/National_Reconnaissance_Office" title="National Reconnaissance Office">NRO</a></li></ul>
    </div>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-ns20110930_15-1"><a href="#cite_note-ns20110930-15"><span class="cite-bracket">[</span>9<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-failure" style="background: #FFC7C7; color:black; vertical-align: middle; text-align: center;">Failure<sup class="reference" id="cite_ref-ns20110930_15-2"><a href="#cite_note-ns20110930-15"><span class="cite-bracket">[</span>9<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-20"><a href="#cite_note-20"><span class="cite-bracket">[</span>14<span class="cite-bracket">]</span></a></sup><br/><small>(parachute)</small>
    </td></tr>
    <tr>
    <td colspan="9">Maiden flight of <a class="mw-redirect" href="/wiki/Dragon_capsule" title="Dragon capsule">Dragon capsule</a>, consisting of over 3 hours of testing thruster maneuvering and reentry.<sup class="reference" id="cite_ref-spaceflightnow_Clark_unleashing_Dragon_21-0"><a href="#cite_note-spaceflightnow_Clark_unleashing_Dragon-21"><span class="cite-bracket">[</span>15<span class="cite-bracket">]</span></a></sup> Attempted to recover the first stage by parachuting it into the ocean, but it disintegrated upon reentry, before the parachutes were deployed.<sup class="reference" id="cite_ref-parachute_18-1"><a href="#cite_note-parachute-18"><span class="cite-bracket">[</span>12<span class="cite-bracket">]</span></a></sup> <small>(<a href="#COTS_demo_missions">more details below</a>)</small> It also included two <a href="/wiki/CubeSat" title="CubeSat">CubeSats</a>,<sup class="reference" id="cite_ref-NRO_Taps_Boeing_for_Next_Batch_of_CubeSats_22-0"><a href="#cite_note-NRO_Taps_Boeing_for_Next_Batch_of_CubeSats-22"><span class="cite-bracket">[</span>16<span class="cite-bracket">]</span></a></sup> and a wheel of <a href="/wiki/Brou%C3%A8re" title="Brouère">Brouère</a> cheese.
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">3
    </th>
    <td>22 May 2012,<br/>07:44<sup class="reference" id="cite_ref-BBC_new_era_23-0"><a href="#cite_note-BBC_new_era-23"><span class="cite-bracket">[</span>17<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.0" title="Falcon 9 v1.0">F9 v1.0</a><sup class="reference" id="cite_ref-MuskMay2012_13-2"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B0005.1<sup class="reference" id="cite_ref-block_numbers_14-2"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_Dragon" title="SpaceX Dragon">Dragon</a> <a class="mw-redirect" href="/wiki/Dragon_C2%2B" title="Dragon C2+">demo flight C2+</a><sup class="reference" id="cite_ref-C2_24-0"><a href="#cite_note-C2-24"><span class="cite-bracket">[</span>18<span class="cite-bracket">]</span></a></sup><br/>(Dragon C102)
    </td>
    <td>525 kg (1,157 lb)<sup class="reference" id="cite_ref-25"><a href="#cite_note-25"><span class="cite-bracket">[</span>19<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a href="/wiki/International_Space_Station" title="International Space Station">ISS</a>)
    </td>
    <td><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Orbital_Transportation_Services" title="Commercial Orbital Transportation Services">COTS</a>)
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-26"><a href="#cite_note-26"><span class="cite-bracket">[</span>20<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt
    </td></tr>
    <tr>
    <td colspan="9">Dragon spacecraft demonstrated a series of tests before it was allowed to approach the <a href="/wiki/International_Space_Station" title="International Space Station">International Space Station</a>. Two days later, it became the first commercial spacecraft to board the ISS.<sup class="reference" id="cite_ref-BBC_new_era_23-1"><a href="#cite_note-BBC_new_era-23"><span class="cite-bracket">[</span>17<span class="cite-bracket">]</span></a></sup> <small>(<a href="#COTS_demo_missions">more details below</a>)</small>
    </td></tr>
    <tr>
    <th rowspan="3" scope="row" style="text-align:center;">4
    </th>
    <td rowspan="2">8 October 2012,<br/>00:35<sup class="reference" id="cite_ref-SFN_LLog_27-0"><a href="#cite_note-SFN_LLog-27"><span class="cite-bracket">[</span>21<span class="cite-bracket">]</span></a></sup>
    </td>
    <td rowspan="2"><a href="/wiki/Falcon_9_v1.0" title="Falcon 9 v1.0">F9 v1.0</a><sup class="reference" id="cite_ref-MuskMay2012_13-3"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B0006.1<sup class="reference" id="cite_ref-block_numbers_14-3"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td rowspan="2"><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_CRS-1" title="SpaceX CRS-1">SpaceX CRS-1</a><sup class="reference" id="cite_ref-sxManifest20120925_28-0"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><br/>(Dragon C103)
    </td>
    <td>4,700 kg (10,400 lb)
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a href="/wiki/International_Space_Station" title="International Space Station">ISS</a>)
    </td>
    <td><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Resupply_Services" title="Commercial Resupply Services">CRS</a>)
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success
    </td>
    <td rowspan="2" style="background:#ececec; text-align:center;"><span class="nowrap">No attempt</span>
    </td></tr>
    <tr>
    <td><a href="/wiki/Orbcomm_(satellite)" title="Orbcomm (satellite)">Orbcomm-OG2</a><sup class="reference" id="cite_ref-Orbcomm_29-0"><a href="#cite_note-Orbcomm-29"><span class="cite-bracket">[</span>23<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>172 kg (379 lb)<sup class="reference" id="cite_ref-gunter-og2_30-0"><a href="#cite_note-gunter-og2-30"><span class="cite-bracket">[</span>24<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a>
    </td>
    <td><a href="/wiki/Orbcomm" title="Orbcomm">Orbcomm</a>
    </td>
    <td class="table-partial" style="background: #FFB; color:black; vertical-align: middle; text-align: center;">Partial failure<sup class="reference" id="cite_ref-nyt-20121030_31-0"><a href="#cite_note-nyt-20121030-31"><span class="cite-bracket">[</span>25<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">CRS-1 was successful, but the <a href="/wiki/Secondary_payload" title="Secondary payload">secondary payload</a> was inserted into an abnormally low orbit and subsequently lost. This was due to one of the nine <a href="/wiki/SpaceX_Merlin" title="SpaceX Merlin">Merlin engines</a> shutting down during the launch, and NASA declining a second reignition, as per <a href="/wiki/International_Space_Station" title="International Space Station">ISS</a> visiting vehicle safety rules, the primary payload owner is contractually allowed to decline a second reignition. NASA stated that this was because SpaceX could not guarantee a high enough likelihood of the second stage completing the second burn successfully which was required to avoid any risk of secondary payload's collision with the ISS.<sup class="reference" id="cite_ref-OrbcommTotalLoss_32-0"><a href="#cite_note-OrbcommTotalLoss-32"><span class="cite-bracket">[</span>26<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-sn20121011_33-0"><a href="#cite_note-sn20121011-33"><span class="cite-bracket">[</span>27<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-34"><a href="#cite_note-34"><span class="cite-bracket">[</span>28<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">5
    </th>
    <td>1 March 2013,<br/>15:10
    </td>
    <td><a href="/wiki/Falcon_9_v1.0" title="Falcon 9 v1.0">F9 v1.0</a><sup class="reference" id="cite_ref-MuskMay2012_13-4"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B0007.1<sup class="reference" id="cite_ref-block_numbers_14-4"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_CRS-2" title="SpaceX CRS-2">SpaceX CRS-2</a><sup class="reference" id="cite_ref-sxManifest20120925_28-1"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><br/>(Dragon C104)
    </td>
    <td>4,877 kg (10,752 lb)
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a class="mw-redirect" href="/wiki/ISS" title="ISS">ISS</a>)
    </td>
    <td><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Resupply_Services" title="Commercial Resupply Services">CRS</a>)
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt
    </td></tr>
    <tr>
    <td colspan="9">Last launch of the original Falcon 9 v1.0 <a href="/wiki/Launch_vehicle" title="Launch vehicle">launch vehicle</a>, first use of the unpressurized trunk section of Dragon.<sup class="reference" id="cite_ref-sxf9_20110321_35-0"><a href="#cite_note-sxf9_20110321-35"><span class="cite-bracket">[</span>29<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">6
    </th>
    <td>29 September 2013,<br/>16:00<sup class="reference" id="cite_ref-pa20130930_36-0"><a href="#cite_note-pa20130930-36"><span class="cite-bracket">[</span>30<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a><sup class="reference" id="cite_ref-MuskMay2012_13-5"><a href="#cite_note-MuskMay2012-13"><span class="cite-bracket">[</span>7<span class="cite-bracket">]</span></a></sup><br/>B1003<sup class="reference" id="cite_ref-block_numbers_14-5"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a class="mw-redirect" href="/wiki/Vandenberg_Air_Force_Base" title="Vandenberg Air Force Base">VAFB</a>,<br/><a href="/wiki/Vandenberg_Space_Launch_Complex_4" title="Vandenberg Space Launch Complex 4">SLC-4E</a>
    </td>
    <td><a href="/wiki/CASSIOPE" title="CASSIOPE">CASSIOPE</a><sup class="reference" id="cite_ref-sxManifest20120925_28-2"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-CASSIOPE_MDA_37-0"><a href="#cite_note-CASSIOPE_MDA-37"><span class="cite-bracket">[</span>31<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>500 kg (1,100 lb)
    </td>
    <td><a href="/wiki/Polar_orbit" title="Polar orbit">Polar orbit</a> <a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a>
    </td>
    <td><a href="/wiki/Maxar_Technologies" title="Maxar Technologies">MDA</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-pa20130930_36-1"><a href="#cite_note-pa20130930-36"><span class="cite-bracket">[</span>30<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-no2" style="background: #FFE3E3; color: black; vertical-align: middle; text-align: center;">Uncontrolled<br/><small>(ocean)</small><sup class="reference" id="cite_ref-ocean_landing_38-0"><a href="#cite_note-ocean_landing-38"><span class="cite-bracket">[</span>d<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">First commercial mission with a private customer, first launch from Vandenberg, and demonstration flight of Falcon 9 v1.1 with an improved 13-tonne to LEO capacity.<sup class="reference" id="cite_ref-sxf9_20110321_35-1"><a href="#cite_note-sxf9_20110321-35"><span class="cite-bracket">[</span>29<span class="cite-bracket">]</span></a></sup> After separation from the second stage carrying Canadian commercial and scientific satellites, the first stage booster performed a controlled reentry,<sup class="reference" id="cite_ref-39"><a href="#cite_note-39"><span class="cite-bracket">[</span>32<span class="cite-bracket">]</span></a></sup> and an <a href="/wiki/Falcon_9_first-stage_landing_tests" title="Falcon 9 first-stage landing tests">ocean touchdown test</a> for the first time. This provided good test data, even though the booster started rolling as it neared the ocean, leading to the shutdown of the central engine as the roll depleted it of fuel, resulting in a hard impact with the ocean.<sup class="reference" id="cite_ref-pa20130930_36-2"><a href="#cite_note-pa20130930-36"><span class="cite-bracket">[</span>30<span class="cite-bracket">]</span></a></sup> This was the first known attempt of a rocket engine being lit to perform a supersonic retro propulsion, and allowed SpaceX to enter a public-private partnership with <a href="/wiki/NASA" title="NASA">NASA</a> and its Mars entry, descent, and landing technologies research projects.<sup class="reference" id="cite_ref-40"><a href="#cite_note-40"><span class="cite-bracket">[</span>33<span class="cite-bracket">]</span></a></sup> <small>(<a href="#Maiden_flight_of_v1.1">more details below</a>)</small>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">7
    </th>
    <td>3 December 2013,<br/>22:41<sup class="reference" id="cite_ref-sfn_wwls20130624_41-0"><a href="#cite_note-sfn_wwls20130624-41"><span class="cite-bracket">[</span>34<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a><br/>B1004
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/SES-8" title="SES-8">SES-8</a><sup class="reference" id="cite_ref-sxManifest20120925_28-3"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-spx-pr_42-0"><a href="#cite_note-spx-pr-42"><span class="cite-bracket">[</span>35<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-aw20110323_43-0"><a href="#cite_note-aw20110323-43"><span class="cite-bracket">[</span>36<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>3,170 kg (6,990 lb)
    </td>
    <td><a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">GTO</a>
    </td>
    <td><a class="mw-redirect" href="/wiki/SES_S.A." title="SES S.A.">SES</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-SNMissionStatus7_44-0"><a href="#cite_note-SNMissionStatus7-44"><span class="cite-bracket">[</span>37<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt<br/><sup class="reference" id="cite_ref-sf10120131203_45-0"><a href="#cite_note-sf10120131203-45"><span class="cite-bracket">[</span>38<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">First <a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">Geostationary transfer orbit</a> (GTO) launch for Falcon 9,<sup class="reference" id="cite_ref-spx-pr_42-1"><a href="#cite_note-spx-pr-42"><span class="cite-bracket">[</span>35<span class="cite-bracket">]</span></a></sup> and first successful reignition of the second stage.<sup class="reference" id="cite_ref-46"><a href="#cite_note-46"><span class="cite-bracket">[</span>39<span class="cite-bracket">]</span></a></sup> SES-8 was inserted into a <a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">Super-Synchronous Transfer Orbit</a> of 79,341 km (49,300 mi) in apogee with an <a href="/wiki/Orbital_inclination" title="Orbital inclination">inclination</a> of 20.55° to the <a href="/wiki/Equator" title="Equator">equator</a>.
    </td></tr></tbody></table>


Starting from the third table is our target table contains the actual launch records.



```python
# Let's print the third table and check its content
first_launch_table = html_tables[3]
print(first_launch_table)
```

    <table class="wikitable plainrowheaders collapsible" style="width: 100%;">
    <tbody><tr>
    <th scope="col">Flight No.
    </th>
    <th scope="col">Date and<br/>time (<a href="/wiki/Coordinated_Universal_Time" title="Coordinated Universal Time">UTC</a>)
    </th>
    <th scope="col"><a href="/wiki/List_of_Falcon_9_first-stage_boosters" title="List of Falcon 9 first-stage boosters">Version,<br/>Booster</a><sup class="reference" id="cite_ref-booster_11-1"><a href="#cite_note-booster-11"><span class="cite-bracket">[</span>b<span class="cite-bracket">]</span></a></sup>
    </th>
    <th scope="col">Launch site
    </th>
    <th scope="col">Payload<sup class="reference" id="cite_ref-Dragon_12-1"><a href="#cite_note-Dragon-12"><span class="cite-bracket">[</span>c<span class="cite-bracket">]</span></a></sup>
    </th>
    <th scope="col">Payload mass
    </th>
    <th scope="col">Orbit
    </th>
    <th scope="col">Customer
    </th>
    <th scope="col">Launch<br/>outcome
    </th>
    <th scope="col"><a href="/wiki/Falcon_9_first-stage_landing_tests" title="Falcon 9 first-stage landing tests">Booster<br/>landing</a>
    </th></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">8
    </th>
    <td>6 January 2014,<br/>22:06<sup class="reference" id="cite_ref-NASA_Spaceflight_48-0"><a href="#cite_note-NASA_Spaceflight-48"><span class="cite-bracket">[</span>41<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a>
    </td>
    <td><a href="/wiki/Cape_Canaveral_Space_Force_Station" title="Cape Canaveral Space Force Station">CCAFS</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">SLC-40</a>
    </td>
    <td><a href="/wiki/Thaicom_6" title="Thaicom 6">Thaicom 6</a><sup class="reference" id="cite_ref-sxManifest20120925_28-4"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>3,325 kg (7,330 lb)
    </td>
    <td><a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">GTO</a>
    </td>
    <td><a href="/wiki/Thaicom" title="Thaicom">Thaicom</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-sn20140106_49-0"><a href="#cite_note-sn20140106-49"><span class="cite-bracket">[</span>42<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt<br/><sup class="reference" id="cite_ref-50"><a href="#cite_note-50"><span class="cite-bracket">[</span>43<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">The Thai communication satellite was the second <a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">GTO</a> launch for Falcon 9. The <a href="/wiki/United_States_Air_Force" title="United States Air Force">USAF</a> evaluated launch data from this flight as part of a separate certification program for SpaceX to qualify to fly military payloads, but found that the launch had "unacceptable fuel reserves at engine cutoff of the stage 2 second burnoff".<sup class="reference" id="cite_ref-bloomberg20140722_51-0"><a href="#cite_note-bloomberg20140722-51"><span class="cite-bracket">[</span>44<span class="cite-bracket">]</span></a></sup> Thaicom-6 was inserted into a <a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">Super-Synchronous Transfer Orbit</a> of 90,039 km (55,948 mi) in <a href="/wiki/Apsis" title="Apsis">apogee</a> with an <a href="/wiki/Orbital_inclination" title="Orbital inclination">inclination</a> of 22.46° to the <a href="/wiki/Equator" title="Equator">equator</a>.
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">9
    </th>
    <td>18 April 2014,<br/>19:25<sup class="reference" id="cite_ref-SFN_LLog_27-1"><a href="#cite_note-SFN_LLog-27"><span class="cite-bracket">[</span>21<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a>
    </td>
    <td><a class="mw-redirect" href="/wiki/Cape_Canaveral_Air_Force_Station" title="Cape Canaveral Air Force Station">Cape Canaveral</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">LC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_CRS-3" title="SpaceX CRS-3">SpaceX CRS-3</a><sup class="reference" id="cite_ref-sxManifest20120925_28-5"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><br/>(Dragon C105)
    </td>
    <td>2,296 kg (5,062 lb)<sup class="reference" id="cite_ref-52"><a href="#cite_note-52"><span class="cite-bracket">[</span>45<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a class="mw-redirect" href="/wiki/ISS" title="ISS">ISS</a>)
    </td>
    <td><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Resupply_Services" title="Commercial Resupply Services">CRS</a>)
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success
    </td>
    <td class="partial table-partial" style="background: #BFE; color:black; vertical-align: middle; text-align: center;">Controlled<br/><small>(ocean)</small> <sup class="reference" id="cite_ref-ocean_landing_38-1"><a href="#cite_note-ocean_landing-38"><span class="cite-bracket">[</span>d<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-auto_53-0"><a href="#cite_note-auto-53"><span class="cite-bracket">[</span>46<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">Following second-stage separation, SpaceX conducted a second <a href="/wiki/Falcon_9_first-stage_landing_tests" title="Falcon 9 first-stage landing tests">controlled-descent test</a> of the discarded booster vehicle and achieved the first successful controlled ocean touchdown of a liquid-rocket-engine orbital booster.<sup class="reference" id="cite_ref-mit20140422_54-0"><a href="#cite_note-mit20140422-54"><span class="cite-bracket">[</span>47<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-aw20140428_55-0"><a href="#cite_note-aw20140428-55"><span class="cite-bracket">[</span>48<span class="cite-bracket">]</span></a></sup> Following the soft touchdown, the first stage tipped over as expected and was destroyed. This was the first Falcon 9 booster to fly with extensible landing legs and the first Dragon mission with the <a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">Falcon 9 v1.1</a> launch vehicle. This flight also launched the <a href="/wiki/Educational_Launch_of_Nanosatellites" title="Educational Launch of Nanosatellites">ELaNa 5</a> mission for <a href="/wiki/NASA" title="NASA">NASA</a> as a secondary payload.<sup class="reference" id="cite_ref-auto2_56-0"><a href="#cite_note-auto2-56"><span class="cite-bracket">[</span>49<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-57"><a href="#cite_note-57"><span class="cite-bracket">[</span>50<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">10
    </th>
    <td>14 July 2014,<br/>15:15
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a>
    </td>
    <td><a class="mw-redirect" href="/wiki/Cape_Canaveral_Air_Force_Station" title="Cape Canaveral Air Force Station">Cape Canaveral</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">LC-40</a>
    </td>
    <td><a class="mw-redirect" href="/wiki/Orbcomm-OG2" title="Orbcomm-OG2">Orbcomm-OG2</a>-1<br/>(6 satellites)<sup class="reference" id="cite_ref-sxManifest20120925_28-6"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>1,316 kg (2,901 lb)
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a>
    </td>
    <td><a href="/wiki/Orbcomm" title="Orbcomm">Orbcomm</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-og2-01_20140714_58-0"><a href="#cite_note-og2-01_20140714-58"><span class="cite-bracket">[</span>51<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="partial table-partial" style="background: #BFE; color:black; vertical-align: middle; text-align: center;">Controlled<br/><small>(ocean)</small><sup class="reference" id="cite_ref-ocean_landing_38-2"><a href="#cite_note-ocean_landing-38"><span class="cite-bracket">[</span>d<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-auto_53-1"><a href="#cite_note-auto-53"><span class="cite-bracket">[</span>46<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">Payload included six satellites weighing 172 kg (379 lb) each and two 142 kg (313 lb) mass simulators.<sup class="reference" id="cite_ref-gunter-og2_30-1"><a href="#cite_note-gunter-og2-30"><span class="cite-bracket">[</span>24<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-gunter-og2-sim_59-0"><a href="#cite_note-gunter-og2-sim-59"><span class="cite-bracket">[</span>52<span class="cite-bracket">]</span></a></sup> Equipped for the second time with <a class="mw-redirect" href="/wiki/Launch_vehicle_landing_gear" title="Launch vehicle landing gear">landing legs</a>, the first-stage booster successfully conducted a <a class="mw-redirect" href="/wiki/SpaceX_Falcon_9_booster_post-mission,_controlled-descent,_test_program" title="SpaceX Falcon 9 booster post-mission, controlled-descent, test program">controlled-descent</a> test consisting of a burn for deceleration from <a class="mw-redirect" href="/wiki/Hypersonic" title="Hypersonic">hypersonic</a> velocity in the upper atmosphere, a <a href="/wiki/Atmospheric_entry" title="Atmospheric entry">reentry</a> burn, and a final landing burn before soft-landing on the ocean surface.<sup class="reference" id="cite_ref-SpaceX22072014_60-0"><a href="#cite_note-SpaceX22072014-60"><span class="cite-bracket">[</span>53<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">11
    </th>
    <td>5 August 2014,<br/>08:00
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a>
    </td>
    <td><a class="mw-redirect" href="/wiki/Cape_Canaveral_Air_Force_Station" title="Cape Canaveral Air Force Station">Cape Canaveral</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">LC-40</a>
    </td>
    <td><a href="/wiki/AsiaSat_8" title="AsiaSat 8">AsiaSat 8</a><sup class="reference" id="cite_ref-sxManifest20120925_28-7"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-AsiaSat_SpaceX_61-0"><a href="#cite_note-AsiaSat_SpaceX-61"><span class="cite-bracket">[</span>54<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-62"><a href="#cite_note-62"><span class="cite-bracket">[</span>55<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>4,535 kg (9,998 lb)
    </td>
    <td><a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">GTO</a>
    </td>
    <td><a href="/wiki/AsiaSat" title="AsiaSat">AsiaSat</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-as8_20140805_63-0"><a href="#cite_note-as8_20140805-63"><span class="cite-bracket">[</span>56<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt<br/><sup class="reference" id="cite_ref-amspace-20140803_64-0"><a href="#cite_note-amspace-20140803-64"><span class="cite-bracket">[</span>57<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">First time SpaceX managed a launch site turnaround between two flights of under a month (22 days). GTO launch of the large communication satellite from Hong Kong did not allow for propulsive return-over-water and controlled splashdown of the first stage.<sup class="reference" id="cite_ref-amspace-20140803_64-1"><a href="#cite_note-amspace-20140803-64"><span class="cite-bracket">[</span>57<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">12
    </th>
    <td>7 September 2014,<br/>05:00
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a><br/>B1011<sup class="reference" id="cite_ref-block_numbers_14-6"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a class="mw-redirect" href="/wiki/Cape_Canaveral_Air_Force_Station" title="Cape Canaveral Air Force Station">Cape Canaveral</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">LC-40</a>
    </td>
    <td><a href="/wiki/AsiaSat_6" title="AsiaSat 6">AsiaSat 6</a><sup class="reference" id="cite_ref-sxManifest20120925_28-8"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-AsiaSat_SpaceX_61-1"><a href="#cite_note-AsiaSat_SpaceX-61"><span class="cite-bracket">[</span>54<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-65"><a href="#cite_note-65"><span class="cite-bracket">[</span>58<span class="cite-bracket">]</span></a></sup>
    </td>
    <td>4,428 kg (9,762 lb)
    </td>
    <td><a href="/wiki/Geostationary_transfer_orbit" title="Geostationary transfer orbit">GTO</a>
    </td>
    <td><a href="/wiki/AsiaSat" title="AsiaSat">AsiaSat</a>
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-sdc20140907_66-0"><a href="#cite_note-sdc20140907-66"><span class="cite-bracket">[</span>59<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-noAttempt" style="background: #EEE; color:black; vertical-align: middle; white-space: nowrap; text-align: center;">No attempt
    </td></tr>
    <tr>
    <td colspan="9">Launch was delayed for two weeks for additional verifications after a malfunction observed in the development of the <a class="mw-redirect" href="/wiki/F9R_Dev1" title="F9R Dev1">F9R Dev1</a> prototype.<sup class="reference" id="cite_ref-67"><a href="#cite_note-67"><span class="cite-bracket">[</span>60<span class="cite-bracket">]</span></a></sup> GTO launch of the heavy payload did not allow for controlled splashdown.<sup class="reference" id="cite_ref-68"><a href="#cite_note-68"><span class="cite-bracket">[</span>61<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <th rowspan="2" scope="row" style="text-align:center;">13
    </th>
    <td>21 September 2014,<br/>05:52<sup class="reference" id="cite_ref-SFN_LLog_27-2"><a href="#cite_note-SFN_LLog-27"><span class="cite-bracket">[</span>21<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Falcon_9_v1.1" title="Falcon 9 v1.1">F9 v1.1</a><br/>B1010<sup class="reference" id="cite_ref-block_numbers_14-7"><a href="#cite_note-block_numbers-14"><span class="cite-bracket">[</span>8<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a class="mw-redirect" href="/wiki/Cape_Canaveral_Air_Force_Station" title="Cape Canaveral Air Force Station">Cape Canaveral</a>,<br/><a href="/wiki/Cape_Canaveral_Space_Launch_Complex_40" title="Cape Canaveral Space Launch Complex 40">LC-40</a>
    </td>
    <td><a href="/wiki/SpaceX_CRS-4" title="SpaceX CRS-4">SpaceX CRS-4</a><sup class="reference" id="cite_ref-sxManifest20120925_28-9"><a href="#cite_note-sxManifest20120925-28"><span class="cite-bracket">[</span>22<span class="cite-bracket">]</span></a></sup><br/>(Dragon <a href="/wiki/Dragon_C106" title="Dragon C106">C106</a>.1)
    </td>
    <td>2,216 kg (4,885 lb)<sup class="reference" id="cite_ref-69"><a href="#cite_note-69"><span class="cite-bracket">[</span>62<span class="cite-bracket">]</span></a></sup>
    </td>
    <td><a href="/wiki/Low_Earth_orbit" title="Low Earth orbit">LEO</a> (<a class="mw-redirect" href="/wiki/ISS" title="ISS">ISS</a>)
    </td>
    <td><a href="/wiki/NASA" title="NASA">NASA</a> (<a href="/wiki/Commercial_Resupply_Services" title="Commercial Resupply Services">CRS</a>)
    </td>
    <td class="table-success" style="background: #9EFF9E; color:black; vertical-align: middle; text-align: center;">Success<sup class="reference" id="cite_ref-nasacrs420140921_70-0"><a href="#cite_note-nasacrs420140921-70"><span class="cite-bracket">[</span>63<span class="cite-bracket">]</span></a></sup>
    </td>
    <td class="table-no2" style="background: #FFE3E3; color: black; vertical-align: middle; text-align: center;">Uncontrolled<br/><small>(ocean)</small><sup class="reference" id="cite_ref-ocean_landing_38-3"><a href="#cite_note-ocean_landing-38"><span class="cite-bracket">[</span>d<span class="cite-bracket">]</span></a></sup><sup class="reference" id="cite_ref-fail-13_71-0"><a href="#cite_note-fail-13-71"><span class="cite-bracket">[</span>64<span class="cite-bracket">]</span></a></sup>
    </td></tr>
    <tr>
    <td colspan="9">Fourth attempt of a soft ocean touchdown,<sup class="reference" id="cite_ref-aw20141016_72-0"><a href="#cite_note-aw20141016-72"><span class="cite-bracket">[</span>65<span class="cite-bracket">]</span></a></sup> but the booster ran out of liquid oxygen.<sup class="reference" id="cite_ref-fail-13_71-1"><a href="#cite_note-fail-13-71"><span class="cite-bracket">[</span>64<span class="cite-bracket">]</span></a></sup> Detailed <a class="mw-redirect" href="/wiki/Thermal_imaging" title="Thermal imaging">thermal imaging</a> infrared sensor data was collected however by NASA, as part of a joint arrangement with SpaceX as part of research on <a class="mw-redirect" href="/wiki/Supersonic_retropropulsion" title="Supersonic retropropulsion">retropropulsive deceleration technologies</a> for developing new approaches to Martian <a href="/wiki/Atmospheric_entry" title="Atmospheric entry">atmospheric entry</a>.<sup class="reference" id="cite_ref-aw20141016_72-1"><a href="#cite_note-aw20141016-72"><span class="cite-bracket">[</span>65<span class="cite-bracket">]</span></a></sup>
    </td></tr></tbody></table>


You should able to see the columns names embedded in the table header elements `<th>` as follows:


```
<tr>
<th scope="col">Flight No.
</th>
<th scope="col">Date and<br/>time (<a href="/wiki/Coordinated_Universal_Time" title="Coordinated Universal Time">UTC</a>)
</th>
<th scope="col"><a href="/wiki/List_of_Falcon_9_first-stage_boosters" title="List of Falcon 9 first-stage boosters">Version,<br/>Booster</a> <sup class="reference" id="cite_ref-booster_11-0"><a href="#cite_note-booster-11">[b]</a></sup>
</th>
<th scope="col">Launch site
</th>
<th scope="col">Payload<sup class="reference" id="cite_ref-Dragon_12-0"><a href="#cite_note-Dragon-12">[c]</a></sup>
</th>
<th scope="col">Payload mass
</th>
<th scope="col">Orbit
</th>
<th scope="col">Customer
</th>
<th scope="col">Launch<br/>outcome
</th>
<th scope="col"><a href="/wiki/Falcon_9_first-stage_landing_tests" title="Falcon 9 first-stage landing tests">Booster<br/>landing</a>
</th></tr>
```


Next, we just need to iterate through the `<th>` elements and apply the provided `extract_column_from_header()` to extract column name one by one



```python
column_names = []

# Apply find_all() function with `th` element on first_launch_table
# Iterate each th element and apply the provided extract_column_from_header() to get a column name
# Append the Non-empty column name (`if name is not None and len(name) > 0`) into a list called column_names
element = soup.find_all('th')
for row in range(len(element)):
    try:
        name = extract_column_from_header(element[row])
        if (name is not None and len(name) > 0):
            column_names.append(name)
    except:
        pass
```

Check the extracted column names



```python
print(column_names)
```

    ['Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'N/A', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'FH 2', 'FH 3', 'Flight No.', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Date and time ( )', 'Launch site', 'Payload', 'Payload mass', 'Orbit', 'Customer', 'Launch outcome', 'Date and time ( )', 'Launch site', 'Payload', 'Orbit', 'Customer', 'Date and time ( )', 'Launch site', 'Payload', 'Orbit', 'Customer', 'Date and time ( )', 'Launch site', 'Payload', 'Orbit', 'Customer', 'Date and time ( )', 'Launch site', 'Payload', 'Orbit', 'Customer', 'Demonstrations', 'logistics', 'Crewed', 'Commercial satellites', 'Scientific satellites', 'Military satellites', 'Rideshares', 'Transporter', 'Bandwagon', 'Flight tests', 'Crewed', 'Commercial satellites', 'Current', 'In development', 'Retired', 'Cancelled', 'Spacecraft', 'Cargo', 'Crewed', 'Test vehicles', 'Current', 'Retired', 'Landing sites', 'Other facilities', 'Support', 'Contracts', 'R&D programs', 'Key people', 'Related', 'General', 'General', 'People', 'Vehicles', 'Launches by rocket type', 'Launches by spaceport', 'Agencies, companies and facilities', 'Other mission lists and timelines']


## TASK 3: Create a data frame by parsing the launch HTML tables


We will create an empty dictionary with keys from the extracted column names in the previous task. Later, this dictionary will be converted into a Pandas dataframe



```python
launch_dict= dict.fromkeys(column_names)

# Remove an irrelvant column
del launch_dict['Date and time ( )']

# Let's initial the launch_dict with each value to be an empty list
launch_dict['Flight No.'] = []
launch_dict['Launch site'] = []
launch_dict['Payload'] = []
launch_dict['Payload mass'] = []
launch_dict['Orbit'] = []
launch_dict['Customer'] = []
launch_dict['Launch outcome'] = []
# Added some new columns
launch_dict['Version Booster']=[]
launch_dict['Booster landing']=[]
launch_dict['Date']=[]
launch_dict['Time']=[]
```

Next, we just need to fill up the `launch_dict` with launch records extracted from table rows.


Usually, HTML tables in Wiki pages are likely to contain unexpected annotations and other types of noises, such as reference links `B0004.1[8]`, missing values `N/A [e]`, inconsistent formatting, etc.


To simplify the parsing process, we have provided an incomplete code snippet below to help you to fill up the `launch_dict`. Please complete the following code snippet with TODOs or you can choose to write your own logic to parse all launch tables:



```python
extracted_row = 0
#Extract each table 
for table_number,table in enumerate(soup.find_all('table',"wikitable plainrowheaders collapsible")):
   # get table row 
    for rows in table.find_all("tr"):
        #check to see if first table heading is as number corresponding to launch a number 
        if rows.th:
            if rows.th.string:
                flight_number=rows.th.string.strip()
                flag=flight_number.isdigit()
        else:
            flag=False
        #get table element 
        row=rows.find_all('td')
        #if it is number save cells in a dictonary 
        if flag:
            extracted_row += 1
            # Flight Number value
            # TODO: Append the flight_number into launch_dict with key `Flight No.`
            launch_dict['Flight No.'].append(flight_number)
            #print(flight_number)
            datatimelist=date_time(row[0])
            
            # Date value
            # TODO: Append the date into launch_dict with key `Date`
            date = datatimelist[0].strip(',')
            launch_dict['Date'].append(date)
            #print(date)
            
            # Time value
            # TODO: Append the time into launch_dict with key `Time`
            time = datatimelist[1]
            launch_dict['Time'].append(time)
            #print(time)
              
            # Booster version
            # TODO: Append the bv into launch_dict with key `Version Booster`
            bv=booster_version(row[1])
            if not(bv):
                bv=row[1].a.string
            print(bv)
            launch_dict['Version Booster'].append(bv)
            
            # Launch Site
            # TODO: Append the bv into launch_dict with key `Launch Site`
            launch_site = row[2].a.string
            launch_dict['Launch site'].append(launch_site)
            #print(launch_site)
            
            # Payload
            # TODO: Append the payload into launch_dict with key `Payload`
            payload = row[3].a.string
            #print(payload)
            launch_dict['Payload'].append(payload)
            
            # Payload Mass
            # TODO: Append the payload_mass into launch_dict with key `Payload mass`
            payload_mass = get_mass(row[4])
            launch_dict['Payload mass'].append(payload_mass)
            #print(payload)
            
            # Orbit
            # TODO: Append the orbit into launch_dict with key `Orbit`
            orbit = row[5].a.string
            launch_dict['Orbit'].append(orbit)
            #print(orbit)
            
            # Customer
            # TODO: Append the customer into launch_dict with key `Customer`
            try:
                customer = row[6].a.string
            except:
                customer = 'Various'
            launch_dict['Customer'].append(customer)
            #print(customer)
            
            # Launch outcome
            # TODO: Append the launch_outcome into launch_dict with key `Launch outcome`
            launch_outcome = list(row[7].strings)[0]
            launch_dict['Launch outcome'].append(launch_outcome)
            #print(launch_outcome)
            
            # Booster landing
            # TODO: Append the launch_outcome into launch_dict with key `Booster landing`
            booster_landing = landing_status(row[8])
            launch_dict['Booster landing'].append(booster_landing)
            #print(booster_landing)
            
```

    F9 v1.07B0003.18
    F9 v1.07B0004.18
    F9 v1.07B0005.18
    F9 v1.07B0006.18
    F9 v1.07B0007.18
    F9 v1.17B10038
    F9 v1.1
    F9 v1.1
    F9 v1.1
    F9 v1.1
    F9 v1.1
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 v1.1[
    F9 FT[
    F9 v1.1[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FT♺[
    F9 FT[
    F9 FT[
    F9 FT[
    F9 FTB1029.2195
    F9 FT[
    F9 FT[
    F9 B4[
    F9 FT[
    F9 B4[
    F9 B4[
    F9 FTB1031.2220
    F9 B4[
    F9 FTB1035.2227
    F9 FTB1036.2227
    F9 B4[
    F9 FTB1032.2245
    F9 FTB1038.2268
    F9 B4[
    F9 B4B1041.2268
    F9 B4B1039.2292
    F9 B4[
    F9 B5311B1046.1268
    F9 B4B1043.2322
    F9 B4B1040.2268
    F9 B4B1045.2336
    F9 B5
    F9 B5349B1048[
    F9 B5B1046.2354
    F9 B5[
    F9 B5B1048.2364
    F9 B5B1047.2268
    F9 B5B1046.3268
    F9 B5[
    F9 B5[
    F9 B5B1049.2397
    F9 B5B1048.3399
    F9 B5[]413
    F9 B5[
    F9 B5B1049.3434
    F9 B5B1051.2420
    F9 B5B1056.2465
    F9 B5B1047.3472
    F9 B5
    F9 B5[
    F9 B5B1056.3482
    F9 B5
    F9 B5
    F9 B5
    F9 B5
    F9 B5
    F9 B5
    F9 B5
    F9 B5[
    F9 B5
    F9 B5
    F9 B5
    F9 B5B1058.2544
    F9 B5
    F9 B5B1049.6544
    F9 B5
    F9 B5B1060.2563
    F9 B5B1058.3565
    F9 B5B1051.6568
    F9 B5
    F9 B5
    F9 B5[
    F9 B5
    F9 B5 ♺[
    F9 B5 ♺[
    F9 B5 ♺
    F9 B5 ♺
    F9 B5
    F9 B5B1051.8609
    F9 B5B1058.5613
    F9 B5 ♺[
    F9 B5 ♺
    F9 B5 ♺[
    F9 B5 ♺[
    F9 B5 ♺
    F9 B5B1060.6643
    F9 B5 ♺
    F9 B5B1061.2647
    F9 B5B1060.7652
    F9 B5B1049.9655
    F9 B5B1051.10657
    F9 B5B1058.8660
    F9 B5B1063.2665
    F9 B5B1067.1668
    F9 B5


After you have fill in the parsed launch record values into `launch_dict`, you can create a dataframe from it.



```python
df=pd.DataFrame(launch_dict)
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
      <th>Flight No.</th>
      <th>Launch site</th>
      <th>Payload</th>
      <th>Payload mass</th>
      <th>Orbit</th>
      <th>Customer</th>
      <th>Launch outcome</th>
      <th>N/A</th>
      <th>FH 2</th>
      <th>FH 3</th>
      <th>...</th>
      <th>People</th>
      <th>Vehicles</th>
      <th>Launches by rocket type</th>
      <th>Launches by spaceport</th>
      <th>Agencies, companies and facilities</th>
      <th>Other mission lists and timelines</th>
      <th>Version Booster</th>
      <th>Booster landing</th>
      <th>Date</th>
      <th>Time</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>CCAFS</td>
      <td>Dragon Spacecraft Qualification Unit</td>
      <td>0</td>
      <td>LEO</td>
      <td>SpaceX</td>
      <td>Success\n</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>F9 v1.07B0003.18</td>
      <td>Failure</td>
      <td>4 June 2010</td>
      <td>18:45</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>CCAFS</td>
      <td>Dragon</td>
      <td>0</td>
      <td>LEO</td>
      <td>NASA</td>
      <td>Success</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>F9 v1.07B0004.18</td>
      <td>Failure</td>
      <td>8 December 2010</td>
      <td>15:43</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>CCAFS</td>
      <td>Dragon</td>
      <td>525 kg</td>
      <td>LEO</td>
      <td>NASA</td>
      <td>Success</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>F9 v1.07B0005.18</td>
      <td>No attempt\n</td>
      <td>22 May 2012</td>
      <td>07:44</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>CCAFS</td>
      <td>SpaceX CRS-1</td>
      <td>4,700 kg</td>
      <td>LEO</td>
      <td>NASA</td>
      <td>Success\n</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>F9 v1.07B0006.18</td>
      <td>No attempt</td>
      <td>8 October 2012</td>
      <td>00:35</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>CCAFS</td>
      <td>SpaceX CRS-2</td>
      <td>4,877 kg</td>
      <td>LEO</td>
      <td>NASA</td>
      <td>Success\n</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>...</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>F9 v1.07B0007.18</td>
      <td>No attempt\n</td>
      <td>1 March 2013</td>
      <td>15:10</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 45 columns</p>
</div>



We can now export it to a <b>CSV</b> for the next section, but to make the answers consistent and in case you have difficulties finishing this lab. 

Following labs will be using a provided dataset to make each lab independent. 


<code>df.to_csv('spacex_web_scraped.csv', index=False)</code>


## Authors


<a href="https://www.linkedin.com/in/yan-luo-96288783/">Yan Luo</a>


<a href="https://www.linkedin.com/in/nayefaboutayoun/">Nayef Abou Tayoun</a>


<!--
## Change Log
-->


<!--
| Date (YYYY-MM-DD) | Version | Changed By | Change Description      |
| ----------------- | ------- | ---------- | ----------------------- |
| 2021-06-09        | 1.0     | Yan Luo    | Tasks updates           |
| 2020-11-10        | 1.0     | Nayef      | Created the initial version |
-->


Copyright © 2021 IBM Corporation. All rights reserved.

