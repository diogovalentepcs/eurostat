# Eurostat Python Package

Tools to read data from Eurostat API.


# Important!

From version 1.0.0, this package implements MAJOR CHANGES.

The previous official Eurostat API, used by the older releases of this Python package, is supposed to be decommissioned in January 2023.
It has been replaced with a new dissemination API.
This forced to rewrite this Python package, almost completely.

I have done my best to make the new Eurostat Python package compatible with the old releases.
Nevertheless, you may see some differences, also in the output format.

The main differences are in the SDMX functions.
In particular, OBS_STATUS is not more provided by the API.
It is replaced by a flag, that has different symbols and meanings.
From version 1.0.0, the SDMX functions of the Eurostat Python package are deprecated, but temporarily kept available.
These functions will be removed from version 2.0.0 of the Eurostat Python package.
They always print an alert message, that can be "muted" by setting the argument *noalert=True*.



# Features

* Read Eurostat data and metadata as list of tuples or as pandas dataframe.
* Use the new SDMX 2.1 Eurostat web services.
* Download data from Eurostat, COMEXT, DG COMP, DG ENV, DG GROW.
* Available from both pip and [conda][condapack]. 
* MIT license.


# Documentation


## Getting started:

Requires Python 3.5+

```bash
pip install eurostat
```


## Read the table of contents of the Eurostat database:

### As a list of tuples:

```python
eurostat.get_toc()
```

Read the table of contents and return a list of tuples. The first element of the list contains the header line. Dates are represented as strings.

#### Example:

```python
>>> import eurostat
>>> toc = eurostat.get_toc()
>>> toc[0]
('title', 'code', 'type', 'last update of data', 'last table structure change', 'data start', 'data end')
>>> toc[12:15]
[('Employment by NACE Rev. 2 activity and metropolitan typology', 'MET_10R_3EMP', 'dataset', '2022-05-06T23:00:00+0200', '2022-05-06T23:00:00+0200', '1995', '2020'),
 ('Gross domestic product (GDP) at current market prices by metropolitan regions', 'MET_10R_3GDP', 'dataset', '2022-04-21T23:00:00+0200', '2022-04-21T23:00:00+0200', '2000', '2020'),
 ('Gross value added at basic prices by metropolitan regions', 'MET_10R_3GVA', 'dataset', '2022-04-21T23:00:00+0200', '2022-04-21T23:00:00+0200', '1995', '2020')]
```

### As a pandas dataframe:

```python
eurostat.get_toc_df()
```

Read the table of contents of the main database and return a dataframe.

#### Example:

```python
>>> import eurostat
>>> toc_df = eurostat.get_toc_df()
>>> toc_df
                                                  title  ... data end
0     Road equipment: number of road vehicles by cat...  ...     2018
1        Road equipment: number of road vehicles by age  ...     2018
2              Road equipment: load capacity of lorries  ...     2015
3       Road equipment: new registrations by categories  ...     2015
4        Road traffic: road freight transport in volume  ...     2018
                                                ...  ...      ...
7225  Tropical wood imports to the EU from chapter 4...  ...  2020-12
7226  Candidate countries and potential candidates: ...  ...     2019
7227  Candidate countries and potential candidates: ...  ...     2014
7228  Candidate countries and potential candidates: ...  ...     2015
7229  Candidate countries and potential candidates: ...  ...     2014
```

You may also want to extract the datasets that pertain a topic. In that case, you can use:

```python
eurostat.subset_toc_df(toc_df, keyword)
```

Extract from toc_df the rows where the column *title* contains *keyword* (case-insensitive).

#### Example:

```python
>>> f = eurostat.subset_toc_df(toc_df, 'fleet')
>>> f
                                                  title  ... data end
4873                       Fishing fleet, total tonnage  ...     2021
4895                   Fishing Fleet, Number of Vessels  ...     2021
6169  Commercial aircraft fleet by age of aircraft a...  ...     2020
6172  Commercial aircraft fleet by age of aircraft a...  ...     2020
6175  Commercial aircraft fleet by aircraft category...  ...     2020
6178  Commercial aircraft fleet by aircraft category...  ...     2020
6576      Commercial aircraft fleet by type of aircraft  ...     2020
7120     Fishing fleet by age, length and gross tonnage  ...     2021
7121     Fishing fleet by type of gear and engine power  ...     2021
```


## Get the filter parameters to download a subset of a dataset:

```python
eurostat.get_pars(code)
```

Read the parameter names that can be filtered for a given dataset *code* and return them as a list.

#### Example:

```python
>>> import eurostat
>>> pars = eurostat.get_pars('demo_r_d2jan')
>>> pars
['freq', 'unit', 'sex', 'age', 'geo']
```

From the example, you can note that *code* is generally case-insensitive.

To get the parameter values for filtering, you can use:

```python
eurostat.get_par_values(code, par)
```

Read the values of a given parameter *par* that can be found in a given dataset *code*.

#### Example:

```python
>>> import eurostat
>>> par_values = eurostat.get_par_values('demo_r_d2jan', 'sex')
>>> par_values
['T', 'M', 'F']
```


## Get an Eurostat dictionary:

```python
eurostat.get_dic(code, par, [full=True], [frmt="list"], [lang="en"])
```

Read the dictionary of a given parameter *par* as a list of tuples or as a dictionary.
You must provide also the dataset *code*.

If you want the full list of possible values of *par*, set *full=True*, while *full=False* returns only the values that are in the dataset, output from *get_par_values()*.
Default is *full=True*.

*frmt="list"* makes the function return a list of tuples, where the first element of each tuple is the code value and the second one is its description.
If *frmt="dict"* it returns a dictionary.
Default is *frmt="list"*.

*lang* allows to download the dictionary in one of the following languages: *"en"*=English, *"fr"*=French, *"de"*=German, when provided by Eurostat. Default is English.

#### Example:

```python
>>> import eurostat
>>> dic = eurostat.get_dic('demo_r_d2jan', 'sex')
>>> dic
[('T', 'Total'),
 ('M', 'Males'),
 ('F', 'Females'),
 ('DIFF', 'Absolute difference between males and females'),
 ('NAP', 'Not applicable'),
 ('NRP', 'No response'),
 ('UNK', 'Unknown')]
```



## Read a dataset:

### As a list of tuples:

```python
eurostat.get_data(code, [flags=False], [filter_pars=dict()], [verbose=False], [reverse_time=False])
```

Read an Eurostat dataset and returns it as a list of tuples.
The first element of the list ("the first row") is the data header.

To get a subset, set *filter_pars* (a dictionary where keys are parameter names, values are the wanted items).

To see a rough progress status, set *verbose=True*.

*flag=True* downloads also the flags associated to the data.
Pay attention: the data format changes if *flags* is *True* or not.
Flag meanings can be found [here][abbr].

*reverse_time=True* reverses the order of the time columns. For compatibility with 0.x.x versions.

#### Example: Full download a dataset without flags as list of tuples
 
```python
>>> import eurostat
>>> data = eurostat.get_data('GOV_10DD_SLGD')
>>> data[0]
('freq', 'na_item', 'sector', 'maturity', 'unit', 'geo\\TIME_PERIOD', 2018, 2019, 2020, 2021)
>>> data[90:95]
[('A', 'F3', 'S11', 'TOTAL', 'MIO_EUR', 'BE', None, None, 23.8, 39.3),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_EUR', 'ES', None, None, 130.0, 122.2),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'AT', None, None, 0.0, 0.0),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'BE', None, None, 23.8, 39.3),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'ES', None, None, 130.0, 122.2)]
```

#### Example: Full download a dataset with flags as list of tuples
 
```python
>>> import eurostat
>>> data = eurostat.get_data('GOV_10DD_SLGD', True)
>>> data[0]
('freq', 'na_item', 'sector', 'maturity', 'unit', 'geo\\TIME_PERIOD', '2018_value', '2018_flag', '2019_value', '2019_flag', '2020_value', '2020_flag', '2021_value', '2021_flag')
>>> data[90:95]
[('A', 'F3', 'S11', 'TOTAL', 'MIO_EUR', 'BE', None, ':', None, ':', 23.8, '', 39.3, ''),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_EUR', 'ES', None, ':', None, ':', 130.0, '', 122.2, ''),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'AT', None, ':', None, ':', 0.0, '', 0.0, ''),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'BE', None, ':', None, ':', 23.8, '', 39.3, ''),
 ('A', 'F3', 'S11', 'TOTAL', 'MIO_NAC', 'ES', None, ':', None, ':', 130.0, '', 122.2, '')]
```


#### Example: Download a subset of data from a dataset without flags as list of tuples

To download a subset, you need to set the *filter_pars* dictionary.
Its keys can be: *'startPeriod'*, *'endPeriod'* and any parameter you get with *eurostat.get_pars()*.
Values can be number, string or list and generally are derived by *eurostat.get_par_values()*.

```python
>>> import eurostat
>>> code = 'GOV_10DD_SLGD'
>>> pars = eurostat.get_pars(code)
>>> pars
['freq', 'na_item', 'sector', 'maturity', 'unit', 'geo']
>>> par_values = eurostat.get_par_values(code, 'geo')
>>> par_values
['BE', 'DE', 'ES', 'AT']
>>> my_filter_pars = {'startPeriod': 2019, 'geo': ['AT','BE']}
>>> data = eurostat.get_data(code, filter_pars=my_filter_pars)
>>> data[0]
('freq', 'na_item', 'sector', 'maturity', 'unit', 'geo\\TIME_PERIOD', 2019, 2020, 2021)
>>> data[445:447]
[('A', 'GD', 'S1_S2', 'Y_LT1', 'PC_TOT', 'AT', None, 1.0, 0.9),
 ('A', 'F22', 'S1_S2', 'TOTAL', 'MIO_EUR', 'BE', None, 0.0, 0.0)]
```

#### Example: Download a subset of data from a dataset with flags as list of tuples

```python
>>> import eurostat
>>> code = 'GOV_10DD_SLGD'
>>> pars = eurostat.get_pars(code)
>>> pars
['freq', 'na_item', 'sector', 'maturity', 'unit', 'geo']
>>> par_values = eurostat.get_par_values(code, 'geo')
>>> par_values
['BE', 'DE', 'ES', 'AT']
>>> my_filter_pars = {'startPeriod': 2019, 'geo': ['AT','BE']}
>>> data = eurostat.get_data(code, True, filter_pars=my_filter_pars)
>>> data[0]
('freq', 'na_item', 'sector', 'maturity', 'unit', 'geo\\TIME_PERIOD', '2019_value', '2019_flag', '2020_value', '2020_flag', '2021_value', '2021_flag')
>>> data[446:448]
[('A', 'GD', 'S1_S2', 'Y_LT1', 'PC_TOT', 'AT', None, ':', 1.0, '', 0.9, ''),
 ('A', 'F22', 'S1_S2', 'TOTAL', 'MIO_EUR', 'BE', None, ':', 0.0, '', 0.0, '')]
```

### As a pandas dataframe:

```python
eurostat.get_data_df(code, [flags=False], [filter_pars=None], [verbose=False], [reverse_time=False])
```

Read an Eurostat dataset and returns it as pandas dataframe.
The first element of the list ("the first row") is the data header.

To get a subset, set *filter_pars* (a dictionary where keys are parameter names, values are the wanted items).

To see a rough progress status, set *verbose=True*.

*flag=True* downloads also the flags associated to the data.
Pay attention: the data format changes if *flags* is *True* or not.
Flag meanings can be found [here][abbr].

*reverse_time=True* reverses the order of the time columns. For compatibility with 0.x.x versions.

#### Example: Full download a dataset without flags as pandas dataframe
 
```python
>>> import eurostat
>>> data = eurostat.get_data_df('GOV_10DD_SLGD')
>>> data
     freq na_item sector maturity  ... 2018 2019    2020    2021
0       A     F22  S1_S2    TOTAL  ...  NaN  NaN     0.0     0.0
1       A     F22  S1_S2    TOTAL  ...  NaN  NaN     0.0     0.0
2       A     F22  S1_S2    TOTAL  ...  0.0  0.0     0.0     0.0
3       A     F22  S1_S2    TOTAL  ...  NaN  NaN     0.0     0.0
4       A     F22  S1_S2    TOTAL  ...  NaN  NaN     0.0     0.0
  ...     ...    ...      ...  ...  ...  ...     ...     ...
1181    A      GD  S1_S2    Y_LT1  ...  NaN  NaN  2849.0  2408.2
1182    A      GD  S1_S2    Y_LT1  ...  NaN  NaN     1.0     0.9
1183    A      GD  S1_S2    Y_LT1  ...  NaN  NaN     6.9     5.4
1184    A      GD  S1_S2    Y_LT1  ...  4.9  6.0     5.9     5.5
1185    A      GD  S1_S2    Y_LT1  ...  NaN  NaN     0.9     0.8
```

#### Example: Full download a dataset with flags as pandas dataframe
 
```python
>>> import eurostat
>>> data = eurostat.get_data_df('GOV_10DD_SLGD', True)
>>> data
     freq na_item sector maturity  ... 2020_value 2020_flag  2021_value 2021_flag
0       A     F22  S1_S2    TOTAL  ...        0.0         :         0.0          
1       A     F22  S1_S2    TOTAL  ...        0.0                   0.0          
2       A     F22  S1_S2    TOTAL  ...        0.0         :         0.0          
3       A     F22  S1_S2    TOTAL  ...        0.0                   0.0          
4       A     F22  S1_S2    TOTAL  ...        0.0                   0.0          
  ...     ...    ...      ...  ...        ...       ...         ...       ...
1182    A      GD  S1_S2    Y_LT1  ...        1.0                   0.9          
1183    A      GD  S1_S2    Y_LT1  ...        6.9                   5.4          
1184    A      GD  S1_S2    Y_LT1  ...        5.9                   5.5          
1185    A      GD  S1_S2    Y_LT1  ...        0.9         :         0.8          
1186    A      GD  S1_S2    Y_LT1  ...        0.9                   0.8          
```

#### Example: Download a subset of data from a dataset without flags as pandas dataframe

To download a subset, you need to set the *filter_pars* dictionary.
Its keys can be: *'startPeriod'*, *'endPeriod'* and any parameter you get with *eurostat.get_pars()*.
Values can be number, string or list and generally are derived by *eurostat.get_par_values()*.

```python
>>> import eurostat
>>> code = 'GOV_10DD_SLGD'
>>> pars = eurostat.get_pars(code)
>>> pars
['freq', 'na_item', 'sector', 'maturity', 'unit', 'geo']
>>> par_values = eurostat.get_par_values(code, 'geo')
>>> par_values
['BE', 'DE', 'ES', 'AT']
>>> my_filter_pars = {'endPeriod': 2020, 'geo': ['AT','BE']}
>>> data = eurostat.get_data_df(code, filter_pars=my_filter_pars)
>>> data
    freq na_item sector maturity     unit geo\TIME_PERIOD  2018  2019     2020
0      A     F22  S1_S2    TOTAL  MIO_EUR              AT   NaN   NaN      0.0
1      A     F22  S1_S2    TOTAL  MIO_NAC              AT   NaN   NaN      0.0
2      A     F22  S1_S2    Y_LT1  MIO_EUR              AT   NaN   NaN      0.0
3      A     F22  S1_S2    Y_LT1  MIO_NAC              AT   NaN   NaN      0.0
4      A     F29  S1_S2    TOTAL  MIO_EUR              AT   NaN   NaN      0.0
..   ...     ...    ...      ...      ...             ...   ...   ...      ...
633    A      GD  S1_S2    Y_GT1  MIO_NAC              BE   NaN   NaN  72660.0
634    A      GD  S1_S2    Y_GT1   PC_TOT              BE   NaN   NaN     93.1
635    A      GD  S1_S2    Y_LT1  MIO_EUR              BE   NaN   NaN   5387.0
636    A      GD  S1_S2    Y_LT1  MIO_NAC              BE   NaN   NaN   5387.0
637    A      GD  S1_S2    Y_LT1   PC_TOT              BE   NaN   NaN      6.9
```

#### Example: Download a subset of data from a dataset with flags as pandas dataframe

```python
>>> import eurostat
>>> code = 'GOV_10DD_SLGD'
>>> pars = eurostat.get_pars(code)
>>> pars
['freq', 'na_item', 'sector', 'maturity', 'unit', 'geo']
>>> par_values = eurostat.get_par_values(code, 'geo')
>>> par_values
['BE', 'DE', 'ES', 'AT']
>>> my_filter_pars = {'endPeriod': 2020, 'geo': ['AT','BE']}
>>> data = eurostat.get_data_df(code, True, filter_pars=my_filter_pars)
>>> data
    freq na_item sector maturity  ... 2019_value 2019_flag  2020_value 2020_flag
0      A     F22  S1_S2    TOTAL  ...        NaN         :         0.0          
1      A     F22  S1_S2    TOTAL  ...        NaN         :         0.0          
2      A     F22  S1_S2    Y_LT1  ...        NaN         :         0.0          
3      A     F22  S1_S2    Y_LT1  ...        NaN         :         0.0          
4      A     F29  S1_S2    TOTAL  ...        NaN         :         0.0          
..   ...     ...    ...      ...  ...        ...       ...         ...       ...
635    A      GD  S1_S2    Y_GT1  ...        NaN         :        93.1          
636    A      GD  S1_S2    Y_LT1  ...        NaN         :      5387.0          
637    A      GD  S1_S2    Y_LT1  ...        NaN         :      5387.0          
638    A      GD  S1_S2    Y_LT1  ...        NaN         :         6.9          
639    A      GD  S1_S2    Y_LT1  ...        NaN         :         6.9          
```

## In case you need to use a proxy:

Before doing anything else, you must configure the https proxy.

```python
eurostat.setproxy(proxyinfo)
```

It requires in input *proxyinfo*, a dictionary with one key (*'https'*) and value containing the connection parameters in a list.  
If authentication is not needed, set *username* and *password* to *None*.

Example:
```python
>>> import eurostat
>>> proxyinfo = {'https': ['myuser', 'mypassword', 'url:port']}
>>> eurostat.setproxy(proxyinfo)
```

It always returns *None*.


## Bug reports and feature requests:

Please [open an issue][issue] or send a message to noemi.cazzaniga [at] polimi.it .


## Disclaimer:

Download and usage of Eurostat data is subject to Eurostat's general copyright notice and licence policy (see [Policies][pol]). Please also be aware of the European Commission's [general conditions][cond].


## Data sources:

* Eurostat database: [online catalog][onlinecat].
* Eurostat nomenclatures: [RAMON][ram] metadata.
* Eurostat Interactive Data Browser: [Data Browser][databrow].
* Eurostat Interactive Tool for Comext Data: [Easy Comext][comext].
* Eurostat PRODCOM website: [PRODCOM][prodcom].
* Eurostat acronyms: [Symbols and abbreviations][abbr].
* Eurostat web services description: [Web services][webserv].


## References:

* R package [eurostat][es]: R Tools for Eurostat Open Data.
* Python package [pandas][pd]: Python Data Analysis Library.


## History:

### version 1.0.0 (08 Oct 2022):

* Adapted to the new Eurostat API (SDMX 2.1).
* pandasdmx not required anymore.
* Error messages improved.
* Compiled also for conda install.
* Multilingual dictionary.

### version 0.2.3 (06 Apr 2021):

* Internal bug fix.

### version 0.2.2 (31 Mar 2021):

* Bug fix (sdmx non-annual data). Deprecated.

### version 0.2.1 (10 Nov 2020):

* Bug fix (pandasdmx 0.9).

### version 0.2.0 (22 May 2020):

* Improved SDMX download capability in case of slow internet connections.

### version 0.1.5 (08 Jan 2020):

* Bug fix (proxy info).
* get_avail_sdmx, get_avail_sdmx_df, subset_avail_sdmx_df added.

### version 0.1.4 (20 Dec 2019):

* Added support to proxy.

### version 0.1.3 (17 Dec 2019):

* Bug fix (non-annual data headers).

### version 0.1.2 (25 Nov 2019):

* Added possibility to download flags.
* get_toc_df, subset_toc_df added.

### version 0.1.1 (21 Nov 2019):

* First official release.


[pol]: https://ec.europa.eu/eurostat/about/policies/copyright
[cond]: http://ec.europa.eu/geninfo/legal_notices_en.htm
[onlinecat]: https://ec.europa.eu/eurostat/data/database
[ram]: https://ec.europa.eu/eurostat/ramon/nomenclatures/index.cfm?TargetUrl=LST_NOM&StrGroupCode=SCL&StrLanguageCode=EN
[databrow]: https://ec.europa.eu/eurostat/databrowser/
[pd]: https://pandas.pydata.org/
[es]: http://ropengov.github.io/eurostat/
[issue]: https://bitbucket.org/noemicazzaniga/eurostat/issues/new
[abbr]: https://ec.europa.eu/eurostat/statistics-explained/index.php?title=Tutorial:Symbols_and_abbreviations#Statistical_symbols.2C_abbreviations_and_units_of_measurement
[prodcom]: https://ec.europa.eu/eurostat/web/prodcom
[comext]: http://epp.eurostat.ec.europa.eu/newxtweb/
[bulkcomext]: https://ec.europa.eu/eurostat/estat-navtree-portlet-prod/BulkDownloadListing?sort=1&dir=comext%2FCOMEXT_DATA%2FPRODUCTS
[webserv]: https://ec.europa.eu/eurostat/web/main/data/web-services
[condapack]: https://anaconda.org/noemicazzaniga/eurostat