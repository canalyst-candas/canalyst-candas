***

# Canalyst Candas Data Science Library

## Name
Canalyst Candas 

## Description
A data science library using Canalyst's API, developed for securities analysis using Python.  
- Search KPI
- Company data Dataframes (one company or many)
- Charts
- Model update (scenario analysis)
- Visualize formula builds

## Installation
Installation instructions can be found on our [PyPI page](https://pypi.org/project/canalyst-candas/)

## Usage

<b>Search Guidance:</b>

Candas is built to facilitate easy discovery of guidance in our Modelverse.  You can search guidance for key items, either filtered by a ticker / ticker list or just across the entire Modelverse.

Example:

```
canalyst_search.search_guidance_time_series(ticker = "", #any ticker or list of tickers 
                sector="Consumer", #path in our nomenclature is a hierarchy of sectors
                file_name="", #file name is a proxy for company name
                time_series_name="", #our range name
                time_series_description="china", #human readable row header
                most_recent=True) #most recent item or all items 
```

<b>Search KPI:</b>

Candas is built to facilitate easy discovery of KPI names in our Modelverse.  You can search guidance for key items, either filtered by a ticker / ticker list or just across the entire Modelverse.

Example:

```
canalyst_search.search_time_series(ticker = "",
                 sector="Thrifts",
                 category="",
                 unit_type="percentage",
                 mo_only=True,
                 period_duration_type='fiscal_quarter',
                 time_series_name='',
                 time_series_description='total revenue growth', #guessing on the time series name
                 query = 'value > 5')
```

<b>ModelSet:</b>

The core objects in Candas are Models.
Models can be also arranged in a set by instantiating a ModelFrame.
In both cases, it's model_set = cd.ModelSet(ticker_list=[ticker_list],config=config)
Or model = cd.Model(ticker=ticker,config=config)

With either a model or modelset, model_frame returns Pandas dataframes.
It is the workhorse of the Model or ModelSet objects, the default parameters are shown below:
- Send in a partial string as time series name, model_frame will regex search for it
- Pivot allows for excel-model style wide data (good for comp screens)
- mrq filters to ONLY the most recent quarter
- period_duration type is fiscal quarter or fiscal year or blank for both
- is_historical is True will filter to only historical, False only forecasts, or blank for both
- n_periods defaults to 12 but most of our models go back to 2013
- mrq_notation applies to pivot, and will filter to historical data and apply MRQ-n notation on the columns (a way to handle off fiscal reporters in comp screens)

Example:

```
model_set.model_frame(time_series_name="MO_RIS_REV",
                  is_driver="",
                  pivot=False,
                  mrq=False,
                  period_duration_type='fiscal_quarter', #or fiscal_year
                  is_historical="",
                  n_periods=12,
                  mrq_notation=False)
`

```

<b>Charting:</b>

Candas has a Canalyst standard charting library which allows for easy visualizations.

Chart Example:
![Chart](c1.png)

```
df_plot = df[df['ticker'].isin(['AZUL US','MESA US'])][['ticker','period_name','value']].pivot_table(values="value", index=["period_name"],columns=["ticker"]).reset_index()
p = cd.Chart(df_plot['period_name'],df_plot[["AZUL US", "MESA US"]],["AZUL US", "MESA US"], [["Periods", "Actual"]], title="MO_MA_Fuel")
p.show()
```

<b>Scenario Analysis:</b>

Candas can arrange a forecast and send it to our scenario engine via the fit() function, and get changed outputs vs the default.

Example:

`
return_series = "MO_RIS_EPS_WAD_Adj"
list_output = []
for ts in time_series_names:
    df_params = model_set.forecast_frame(ts,
                             n_periods=-1,
                             function_name='multiply',
                             function_value=(1.1))
    dicts_output=model_set.fit(df_params,return_series)
    for key in dicts_output.keys():
        list_output.append(dicts_output[key].head(1))
`

<b>ModelMap:</b>

Candas can show a node tree at any level of the PNL

Example:

```
model_set.create_model_map(ticker=ticker,time_series_name="MO_RIS_REV",col_for_labels = "time_series_description").show() #launches in a separate browser window
```

ModelMap example: Node Chart for Fuel Margin
![Fuel Margin](c2.png)

KPI Importance: Use the same node tree to extract key drivers, then use our scenario engine to rank order 1% changes in KPI driver vs subsequent revenue change

```
#use the same node tree to extract key drivers (red nodes in the map)
df = model_set.models[ticker].key_driver_map("MO_RIS_REV")
return_series = 'MO_RIS_REV'
driver_list_df = []
for i, row in df.iterrows():

    time_series_name = row['time_series_name']
    print(f"scenario: move {time_series_name} 1% and get resultant change in {return_series}")

    #create a param dataframe for each time series name in our list
    df_1_param = model_set.forecast_frame(time_series_name,
                         n_periods=-1,
                         function_name='multiply',
                         function_value=1.01)


    d_output=model_set.fit(df_1_param,return_series) #our fit function will return a link to scenario engine JSON for audit

    df_output = model_set.filter_summary(d_output,period_type='Q')

    df_merge = pd.merge(df_output,df_1_param,how='inner',left_on=['ticker','period_name'],right_on=['ticker','period_name'])

    driver_list_df.append(df_merge) #append to a list for concatenating at the end
df = pd.concat(driver_list_df).sort_values('diff',ascending=False)[['ticker','time_series_name_y','diff']]
df = df.rename(columns={'time_series_name_y':'time_series_name'})
df['diff'] = df['diff']-1
df = df.sort_values('diff')
df.plot(x='time_series_name',y='diff',kind='barh',title=ticker+" Key Drivers Revenue Sensitivity")
```
![KPI Rank](c3.png)


## Support
support@canalyst.com

## Contributing
Project is currently only open to contributors through discussion with the maintainer.

## Authors and acknowledgment
jed.gore@canalyst.com

## License
APL 2.0 

## Project status
Ongoing

