
# What is the most sustainable way to generate electricity? A comparison of ecoinvent datasets.

<h1>Table of Contents<span class="tocSkip"></span></h1>
<div class="toc"><ul class="toc-item"><li><span><a href="#Introduction" data-toc-modified-id="Introduction-1"><span class="toc-item-num">1&nbsp;&nbsp;</span>Introduction</a></span></li><li><span><a href="#Calculation" data-toc-modified-id="Calculation-2"><span class="toc-item-num">2&nbsp;&nbsp;</span>Calculation</a></span><ul class="toc-item"><li><span><a href="#ILCD-scores" data-toc-modified-id="ILCD-scores-2.1"><span class="toc-item-num">2.1&nbsp;&nbsp;</span>ILCD scores</a></span></li><li><span><a href="#Sustainability-Index" data-toc-modified-id="Sustainability-Index-2.2"><span class="toc-item-num">2.2&nbsp;&nbsp;</span>Sustainability Index</a></span><ul class="toc-item"><li><span><a href="#Normalization" data-toc-modified-id="Normalization-2.2.1"><span class="toc-item-num">2.2.1&nbsp;&nbsp;</span>Normalization</a></span></li><li><span><a href="#Weighing-and-aggregation" data-toc-modified-id="Weighing-and-aggregation-2.2.2"><span class="toc-item-num">2.2.2&nbsp;&nbsp;</span>Weighing and aggregation</a></span></li></ul></li><li><span><a href="#Export" data-toc-modified-id="Export-2.3"><span class="toc-item-num">2.3&nbsp;&nbsp;</span>Export</a></span></li><li><span><a href="#Visualization" data-toc-modified-id="Visualization-2.4"><span class="toc-item-num">2.4&nbsp;&nbsp;</span>Visualization</a></span></li></ul></li><li><span><a href="#Discussion" data-toc-modified-id="Discussion-3"><span class="toc-item-num">3&nbsp;&nbsp;</span>Discussion</a></span></li></ul></div>

## Introduction

Which electricity generation technology is the most sustainable one? Is it photovoltaic panels on house roofs? Is it off-shore wind parks? Is it nuclear pressure water reactors?! The answer will depend on several aspects. 

First of all, it depends on the models used to describe the different generation technologies. Here, I will use models and parameters as supplied by ecoinvent 3.5, allocation at the point of substitution system model (https://www.ecoinvent.org/).

Secondly, it will depend on how we define sustainability. For example, wind energy seems very sustainable from a climate change point of view (low green house gas emissions). However, wind turbines need large amounts of minerals and metals in their construction, making them seem less sustainable from a resource point of view. This is one example of how different indicators will yield different answers. For this study, I will use the 19 midpoint indicators recommended by the International Reference Life Cycle Data System, version 2.0, as implemented in ecoinvent 3.5. I will present results for each indicator. Additionally, I will present normalized, equal-weighted aggregates. The later is just *one* example of how to aggregate multiple indicators to yield one sustainability index. There are infinitely many ways to aggregate different indicators and none of them is preferable over the other. In the end, sustainability measures will always be a subjective construct because different stakeholders give different emphasis to different impact categories.

## Calculation

### ILCD scores
I use the brightway2 package for python for impact calculations (https://brightwaylca.org/). Brightway allows to 
- read the database (ecoinvent 3.5 APOS)
- query the database for activities (all electricity production activities in the database)
- calculate the impact score of activities according to different methods (ILCD 2.0)

First, imports.


```python
import brightway2 as bw
import pandas as pd
import xlsxwriter
```

I'll skip the setup here. Please refer to the official brightway guide for details on how to import the ecoinvent 3.5 database etc.: https://nbviewer.jupyter.org/urls/bitbucket.org/cmutel/brightway2/raw/default/notebooks/Getting%20Started%20with%20Brightway2.ipynb

Let's start by getting all electricity production activities in the database.


```python
# setting the directory containing ecoinvent 3.5 APOS database
bw.projects.set_current("ecoinvent-import")

# querying 
lActivities = [a for a in bw.Database("ecoinvent 3.5 APOS") if "electricity production" in a["name"]]

len(lActivities)
```




    1488



ecoinvent knows 1,488 different activities that produce electricity! Let's go ahead and calculate their impacts. 

**Note: Computation of all values may take up to an hour! I reduced the number of activities to the first 5 in the list to make the notebook runable. Feel free to delete the corresponding line to calculate all impacts on your system.**


```python
###### delete line below to calculate ALL impacts ######
lActivities = lActivities[:5]
########################################################

# get ILCD 2.0 midpoint methods
ilcd = [m for m in bw.methods if "ILCD" in str(m) and "2018" in str(m) and "LT" not in str(m)]

# compute all ILCD scores for all activities
ldScores = []
for a in lActivities:
    oLCA = bw.LCA({a:1}, ilcd[0])
    oLCA.lci()
    oLCA.lcia()
    dScores = {ilcd[0]:oLCA.score}
    for oMethod in ilcd[1:]:
        oLCA.switch_method(oMethod)
        oLCA.lcia()
        dScores[oMethod] = oLCA.score
    ldScores.append(dScores)
    
# convert to dataframe
df = pd.DataFrame(ldScores)
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
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change biogenic)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change fossil)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change land use and land use change)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change total)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater and terrestrial acidification)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater ecotoxicity)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, marine eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, terrestrial eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ionising radiation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, non-carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ozone layer depletion)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, photochemical ozone creation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, respiratory effects, inorganics)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, dissipated water)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, fossils)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, land use)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, minerals and metals)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.000053</td>
      <td>0.588143</td>
      <td>0.000015</td>
      <td>0.588211</td>
      <td>0.000645</td>
      <td>0.217072</td>
      <td>0.000008</td>
      <td>0.000180</td>
      <td>0.001934</td>
      <td>1.142199e-09</td>
      <td>0.001369</td>
      <td>1.202924e-08</td>
      <td>5.099300e-08</td>
      <td>0.000718</td>
      <td>1.659245e-09</td>
      <td>0.044559</td>
      <td>9.281378</td>
      <td>0.170848</td>
      <td>1.040912e-07</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.000162</td>
      <td>1.053454</td>
      <td>0.000078</td>
      <td>1.053694</td>
      <td>0.010851</td>
      <td>0.074229</td>
      <td>0.000753</td>
      <td>0.001528</td>
      <td>0.015533</td>
      <td>1.561707e-09</td>
      <td>0.004844</td>
      <td>4.135861e-08</td>
      <td>6.808792e-09</td>
      <td>0.004120</td>
      <td>1.342487e-08</td>
      <td>0.077180</td>
      <td>15.350463</td>
      <td>2.357585</td>
      <td>1.142805e-07</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000026</td>
      <td>0.012671</td>
      <td>0.000030</td>
      <td>0.012726</td>
      <td>0.000074</td>
      <td>0.025228</td>
      <td>0.000009</td>
      <td>0.000060</td>
      <td>0.000187</td>
      <td>5.676048e-10</td>
      <td>1.184924</td>
      <td>3.716515e-09</td>
      <td>5.734082e-08</td>
      <td>0.000051</td>
      <td>2.871032e-09</td>
      <td>0.132819</td>
      <td>14.255169</td>
      <td>0.084286</td>
      <td>5.122173e-08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000363</td>
      <td>0.077957</td>
      <td>0.000177</td>
      <td>0.078496</td>
      <td>0.000570</td>
      <td>0.089036</td>
      <td>0.000072</td>
      <td>0.000101</td>
      <td>0.000926</td>
      <td>2.191182e-09</td>
      <td>0.009163</td>
      <td>2.707840e-08</td>
      <td>8.781046e-09</td>
      <td>0.000315</td>
      <td>4.861033e-09</td>
      <td>0.112100</td>
      <td>1.200793</td>
      <td>0.592213</td>
      <td>3.434466e-06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000287</td>
      <td>0.061599</td>
      <td>0.000140</td>
      <td>0.062025</td>
      <td>0.000450</td>
      <td>0.070353</td>
      <td>0.000057</td>
      <td>0.000080</td>
      <td>0.000732</td>
      <td>1.731384e-09</td>
      <td>0.007240</td>
      <td>2.139614e-08</td>
      <td>6.938799e-09</td>
      <td>0.000249</td>
      <td>3.841037e-09</td>
      <td>0.088576</td>
      <td>0.948821</td>
      <td>0.467936</td>
      <td>2.713751e-06</td>
    </tr>
  </tbody>
</table>
</div>



Numbered indices are not very readable. Let's use metadata about the activities as the index instead:


```python
# get names
names = [a["name"].split(",") for a in lActivities]
df_names = pd.DataFrame(names).fillna(" ")
# split names at the commas to make reading and manipulation easier
col_names = [("name_"+str(c), " ", " ") for c in df_names.columns]
df[col_names] = df_names

# add units and locations
df[("unit"," "," ")] = [a["unit"] for a in lActivities]
df[("location"," "," ")] = [a["location"] for a in lActivities]

# set index
meta_data_cols = col_names + [("unit", " ", " "), ("location", " ", " ")]
df.set_index(meta_data_cols, inplace=True)

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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change biogenic)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change fossil)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change land use and land use change)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change total)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater and terrestrial acidification)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater ecotoxicity)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, marine eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, terrestrial eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ionising radiation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, non-carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ozone layer depletion)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, photochemical ozone creation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, respiratory effects, inorganics)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, dissipated water)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, fossils)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, land use)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, minerals and metals)</th>
    </tr>
    <tr>
      <th>(name_0,  ,  )</th>
      <th>(name_1,  ,  )</th>
      <th>(name_2,  ,  )</th>
      <th>(name_3,  ,  )</th>
      <th>(name_4,  ,  )</th>
      <th>(name_5,  ,  )</th>
      <th>(unit,  ,  )</th>
      <th>(location,  ,  )</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">electricity production</th>
      <th>natural gas</th>
      <th>conventional power plant</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>CN-JS</th>
      <td>0.000053</td>
      <td>0.588143</td>
      <td>0.000015</td>
      <td>0.588211</td>
      <td>0.000645</td>
      <td>0.217072</td>
      <td>0.000008</td>
      <td>0.000180</td>
      <td>0.001934</td>
      <td>1.142199e-09</td>
      <td>0.001369</td>
      <td>1.202924e-08</td>
      <td>5.099300e-08</td>
      <td>0.000718</td>
      <td>1.659245e-09</td>
      <td>0.044559</td>
      <td>9.281378</td>
      <td>0.170848</td>
      <td>1.040912e-07</td>
    </tr>
    <tr>
      <th>hard coal</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>RoW</th>
      <td>0.000162</td>
      <td>1.053454</td>
      <td>0.000078</td>
      <td>1.053694</td>
      <td>0.010851</td>
      <td>0.074229</td>
      <td>0.000753</td>
      <td>0.001528</td>
      <td>0.015533</td>
      <td>1.561707e-09</td>
      <td>0.004844</td>
      <td>4.135861e-08</td>
      <td>6.808792e-09</td>
      <td>0.004120</td>
      <td>1.342487e-08</td>
      <td>0.077180</td>
      <td>15.350463</td>
      <td>2.357585</td>
      <td>1.142805e-07</td>
    </tr>
    <tr>
      <th>nuclear</th>
      <th>boiling water reactor</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>US-NPCC</th>
      <td>0.000026</td>
      <td>0.012671</td>
      <td>0.000030</td>
      <td>0.012726</td>
      <td>0.000074</td>
      <td>0.025228</td>
      <td>0.000009</td>
      <td>0.000060</td>
      <td>0.000187</td>
      <td>5.676048e-10</td>
      <td>1.184924</td>
      <td>3.716515e-09</td>
      <td>5.734082e-08</td>
      <td>0.000051</td>
      <td>2.871032e-09</td>
      <td>0.132819</td>
      <td>14.255169</td>
      <td>0.084286</td>
      <td>5.122173e-08</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">photovoltaic</th>
      <th rowspan="2" valign="top">3kWp slanted-roof installation</th>
      <th rowspan="2" valign="top">multi-Si</th>
      <th rowspan="2" valign="top">panel</th>
      <th rowspan="2" valign="top">mounted</th>
      <th rowspan="2" valign="top">kilowatt hour</th>
      <th>LV</th>
      <td>0.000363</td>
      <td>0.077957</td>
      <td>0.000177</td>
      <td>0.078496</td>
      <td>0.000570</td>
      <td>0.089036</td>
      <td>0.000072</td>
      <td>0.000101</td>
      <td>0.000926</td>
      <td>2.191182e-09</td>
      <td>0.009163</td>
      <td>2.707840e-08</td>
      <td>8.781046e-09</td>
      <td>0.000315</td>
      <td>4.861033e-09</td>
      <td>0.112100</td>
      <td>1.200793</td>
      <td>0.592213</td>
      <td>3.434466e-06</td>
    </tr>
    <tr>
      <th>IN-JH</th>
      <td>0.000287</td>
      <td>0.061599</td>
      <td>0.000140</td>
      <td>0.062025</td>
      <td>0.000450</td>
      <td>0.070353</td>
      <td>0.000057</td>
      <td>0.000080</td>
      <td>0.000732</td>
      <td>1.731384e-09</td>
      <td>0.007240</td>
      <td>2.139614e-08</td>
      <td>6.938799e-09</td>
      <td>0.000249</td>
      <td>3.841037e-09</td>
      <td>0.088576</td>
      <td>0.948821</td>
      <td>0.467936</td>
      <td>2.713751e-06</td>
    </tr>
  </tbody>
</table>
</div>



That's it! These are impact scores for all electricity production activities in ecoinvent 3.5. I can use these to answer indicator-specific questions like: Which electricity generation technology has the lowest total global warming potential (GWP 100)?


```python
df[("ILCD 2.0 2018 midpoint", "climate change", "climate change total")].idxmin()
```




    ('electricity production',
     ' nuclear',
     ' boiling water reactor',
     ' ',
     ' ',
     ' ',
     'kilowatt hour',
     'US-NPCC')



Or statistical evaluations, like what is the average and standard deviation for the GWP 100 indicator for all electricity generation activities?


```python
df[("ILCD 2.0 2018 midpoint", "climate change", "climate change total")].describe()
```




    count    5.000000
    mean     0.359031
    std      0.453299
    min      0.012726
    25%      0.062025
    50%      0.078496
    75%      0.588211
    max      1.053694
    Name: (ILCD 2.0 2018 midpoint, climate change, climate change total), dtype: float64



### Sustainability Index

Using the produced data we can rank the electricity generation datasets according to individual impact indicators. However, a single indicator does not give enough information to decide if a technology is sustainable or not. To get a bigger picture, I want to aggregate all indicators into one number. As mentioned in the introduction, there are infinitely many ways to do this. The one chosen here is not more right or wrong than any other way. Feel free to change this part according to your needs!

#### Normalization

For each indicator, I choose the minimum and the maximum value over all activities. I define the minimum as 0 and the maximum as 1. Then I use linear interpolation to project all other values into this [0, 1] range.


```python
df_normalized = df.copy()
for indicator in df.columns:
    max_value = df[indicator].max()
    min_value = df[indicator].min()
    df_normalized[indicator] = (df[indicator] - min_value) / (max_value - min_value)
    
df_normalized.head()
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change biogenic)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change fossil)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change land use and land use change)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change total)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater and terrestrial acidification)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater ecotoxicity)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, marine eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, terrestrial eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ionising radiation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, non-carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ozone layer depletion)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, photochemical ozone creation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, respiratory effects, inorganics)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, dissipated water)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, fossils)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, land use)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, minerals and metals)</th>
    </tr>
    <tr>
      <th>(name_0,  ,  )</th>
      <th>(name_1,  ,  )</th>
      <th>(name_2,  ,  )</th>
      <th>(name_3,  ,  )</th>
      <th>(name_4,  ,  )</th>
      <th>(name_5,  ,  )</th>
      <th>(unit,  ,  )</th>
      <th>(location,  ,  )</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">electricity production</th>
      <th>natural gas</th>
      <th>conventional power plant</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>CN-JS</th>
      <td>0.079120</td>
      <td>0.552922</td>
      <td>0.000000</td>
      <td>0.552836</td>
      <td>0.052945</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.081488</td>
      <td>0.113848</td>
      <td>0.353906</td>
      <td>0.000000</td>
      <td>0.220836</td>
      <td>0.874380</td>
      <td>0.163834</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.578584</td>
      <td>0.038077</td>
      <td>0.015627</td>
    </tr>
    <tr>
      <th>hard coal</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>RoW</th>
      <td>0.402550</td>
      <td>1.000000</td>
      <td>0.388987</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.255419</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.612291</td>
      <td>0.002936</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.369608</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.018639</td>
    </tr>
    <tr>
      <th>nuclear</th>
      <th>boiling water reactor</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>US-NPCC</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.087117</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.001290</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.102994</td>
      <td>1.000000</td>
      <td>0.923947</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">photovoltaic</th>
      <th rowspan="2" valign="top">3kWp slanted-roof installation</th>
      <th rowspan="2" valign="top">multi-Si</th>
      <th rowspan="2" valign="top">panel</th>
      <th rowspan="2" valign="top">mounted</th>
      <th rowspan="2" valign="top">kilowatt hour</th>
      <th>LV</th>
      <td>1.000000</td>
      <td>0.062728</td>
      <td>1.000000</td>
      <td>0.063182</td>
      <td>0.045976</td>
      <td>0.332602</td>
      <td>0.086409</td>
      <td>0.027954</td>
      <td>0.048144</td>
      <td>1.000000</td>
      <td>0.006586</td>
      <td>0.620632</td>
      <td>0.039030</td>
      <td>0.064746</td>
      <td>0.272131</td>
      <td>0.765254</td>
      <td>0.017496</td>
      <td>0.223432</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>IN-JH</th>
      <td>0.773791</td>
      <td>0.047011</td>
      <td>0.770071</td>
      <td>0.047359</td>
      <td>0.034886</td>
      <td>0.235214</td>
      <td>0.066049</td>
      <td>0.013513</td>
      <td>0.035482</td>
      <td>0.716799</td>
      <td>0.004961</td>
      <td>0.469677</td>
      <td>0.002573</td>
      <td>0.048516</td>
      <td>0.185438</td>
      <td>0.498725</td>
      <td>0.000000</td>
      <td>0.168763</td>
      <td>0.786975</td>
    </tr>
  </tbody>
</table>
</div>



The result is a table where all impact scores range between zero and one. Zero means lowest impact with reference to the benchmark (i.e. all ecoinvent 3.5 electricity generation activities). One means highest impact with reference to the benchmark.

#### Weighing and aggregation

We still have 19 numbers, each of which describes a small part of the big picture "sustainability". I will now boil them down to one number by simply adding them up. I call the resulting number "sustainability index". Let me stress this again: This index is not more right or wrong than any other one. It is *one* rather arbitrary way to aggregate the individual impact scores.

The lowest possible number for our index is zero. Zero indicates a technology which achieves the *lowest* possible (with reference to the benchmark) impact score in all nineteen impact categories. The highest possible index value is nineteen. It indicates a technology which has the *highest* possible (with reference to the benchmark) impact score in all nineteen impact categories.

Let's see how the ecoinvent activities score in this index:


```python
# sum
df_normalized[("SUM"," "," ")] = df_normalized.sum(axis=1)

# sort ascending
df_normalized.sort_values(by=("SUM"," "," "), ascending=True, inplace=True)

df_normalized.head()
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
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change biogenic)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change fossil)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change land use and land use change)</th>
      <th>(ILCD 2.0 2018 midpoint, climate change, climate change total)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater and terrestrial acidification)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater ecotoxicity)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, freshwater eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, marine eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, ecosystem quality, terrestrial eutrophication)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ionising radiation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, non-carcinogenic effects)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, ozone layer depletion)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, photochemical ozone creation)</th>
      <th>(ILCD 2.0 2018 midpoint, human health, respiratory effects, inorganics)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, dissipated water)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, fossils)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, land use)</th>
      <th>(ILCD 2.0 2018 midpoint, resources, minerals and metals)</th>
      <th>(SUM,  ,  )</th>
    </tr>
    <tr>
      <th>(name_0,  ,  )</th>
      <th>(name_1,  ,  )</th>
      <th>(name_2,  ,  )</th>
      <th>(name_3,  ,  )</th>
      <th>(name_4,  ,  )</th>
      <th>(name_5,  ,  )</th>
      <th>(unit,  ,  )</th>
      <th>(location,  ,  )</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="5" valign="top">electricity production</th>
      <th>nuclear</th>
      <th>boiling water reactor</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>US-NPCC</th>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.087117</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.001290</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.102994</td>
      <td>1.000000</td>
      <td>0.923947</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>4.115347</td>
    </tr>
    <tr>
      <th>natural gas</th>
      <th>conventional power plant</th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>CN-JS</th>
      <td>0.079120</td>
      <td>0.552922</td>
      <td>0.000000</td>
      <td>0.552836</td>
      <td>0.052945</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.081488</td>
      <td>0.113848</td>
      <td>0.353906</td>
      <td>0.000000</td>
      <td>0.220836</td>
      <td>0.874380</td>
      <td>0.163834</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.578584</td>
      <td>0.038077</td>
      <td>0.015627</td>
      <td>4.678403</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">photovoltaic</th>
      <th rowspan="2" valign="top">3kWp slanted-roof installation</th>
      <th rowspan="2" valign="top">multi-Si</th>
      <th rowspan="2" valign="top">panel</th>
      <th rowspan="2" valign="top">mounted</th>
      <th rowspan="2" valign="top">kilowatt hour</th>
      <th>IN-JH</th>
      <td>0.773791</td>
      <td>0.047011</td>
      <td>0.770071</td>
      <td>0.047359</td>
      <td>0.034886</td>
      <td>0.235214</td>
      <td>0.066049</td>
      <td>0.013513</td>
      <td>0.035482</td>
      <td>0.716799</td>
      <td>0.004961</td>
      <td>0.469677</td>
      <td>0.002573</td>
      <td>0.048516</td>
      <td>0.185438</td>
      <td>0.498725</td>
      <td>0.000000</td>
      <td>0.168763</td>
      <td>0.786975</td>
      <td>4.905804</td>
    </tr>
    <tr>
      <th>LV</th>
      <td>1.000000</td>
      <td>0.062728</td>
      <td>1.000000</td>
      <td>0.063182</td>
      <td>0.045976</td>
      <td>0.332602</td>
      <td>0.086409</td>
      <td>0.027954</td>
      <td>0.048144</td>
      <td>1.000000</td>
      <td>0.006586</td>
      <td>0.620632</td>
      <td>0.039030</td>
      <td>0.064746</td>
      <td>0.272131</td>
      <td>0.765254</td>
      <td>0.017496</td>
      <td>0.223432</td>
      <td>1.000000</td>
      <td>6.676299</td>
    </tr>
    <tr>
      <th>hard coal</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th>kilowatt hour</th>
      <th>RoW</th>
      <td>0.402550</td>
      <td>1.000000</td>
      <td>0.388987</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.255419</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.612291</td>
      <td>0.002936</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.369608</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.018639</td>
      <td>13.050429</td>
    </tr>
  </tbody>
</table>
</div>



### Export

Let's export the absolute and the normalized results to an excel file. 


```python
# transform index into individual columns for easier manipulation
df.reset_index(inplace=True)
df_normalized.reset_index(inplace=True)

# make multi-level headers for better readability
df.columns = pd.MultiIndex.from_tuples(df.columns)
df_normalized.columns = pd.MultiIndex.from_tuples(df_normalized.columns)

# export to xlsx
writer = pd.ExcelWriter("output/ecoinvent_electricity_comparison.xlsx", engine='xlsxwriter')
df.to_excel(writer, sheet_name='abs')
df_normalized.to_excel(writer, sheet_name='norm')
writer.save()
```

### Visualization

Let's draw a heat map showing all normalized impacts for all activities and coloring them according to their magnitude.


```python
import bokeh.io
import bokeh.models
import bokeh.plotting
from bokeh.palettes import Reds9
import re
import numpy as np

# construct list of activity names for display
names = df.loc[:,meta_data_cols].apply(lambda x: re.sub(' +', ' '," ".join(x[1:])).strip(), axis=1).to_list()

# construct list of method names for display
methods = [", ".join(m[1:]) for m in ilcd]

# define tooltips to be displayed
TOOLTIPS = [
    ("activity", "@act"),
    ("impact category", "@cat"),
    ("normalized score", "@score"),
]

# make figure
f = bokeh.plotting.figure(
    x_axis_label="ILCD 2.0 midpoint indicator", y_axis_label='ecoinvent activity',
    plot_width=900, plot_height=800,   
    tooltips = TOOLTIPS,
    y_range=bokeh.models.FactorRange(*names),
    x_range=bokeh.models.FactorRange(*methods)
)

# define plot data
data = {
    "score": [df_normalized.loc[i,m] for i in df_normalized.index for m in ilcd],
    "act": [names[i] for i in df_normalized.index for m in ilcd],
    "cat": [m for i in df_normalized.index for m in methods],
}

# define colormap
mapper = bokeh.models.LinearColorMapper(palette=Reds9, low=1, high=0)

# plot
f.rect(
    source=data, x="cat", y="act", width=1, height=1,
    fill_color={'field': 'score', 'transform': mapper},
    line_color=None,
)

# rotate x-axis ticks
f.xaxis.major_label_orientation = np.pi / 4
    
# show plot
bokeh.io.output_notebook()
bokeh.io.show(f)
```



    <div class="bk-root">
        <a href="https://bokeh.pydata.org" target="_blank" class="bk-logo bk-logo-small bk-logo-notebook"></a>
        <span id="1631">Loading BokehJS ...</span>
    </div>











  <div class="bk-root" id="8a597481-c264-464b-9e18-ae3bc871afe8" data-root-id="1594"></div>





Save the figure to disk.


```python
bokeh.io.output_file("output/heatmap.html")
path = bokeh.io.save(f)
```

## Discussion
