## Quickstart

### Introduction

Hi! Glad to see you made it to the Quickstart guide! In this guide you are introduced to how CARTOframes can be used by data scientists in spatial analysis workflows. Using fake Starbucks revenue data, this guide walks through some common steps a data scientist takes to answer the following question: which stores are performing better than others?

Before you get started, we encourage you to have CARTOframes installed so you can get a feel for the library by using it:

```
pip install --pre cartoframes
```

If you want to know other ways to install it, check out the [installation guide](/developers/cartoframes/guides/Installation) first.

#### Spatial analysis scenario

Let's say you are a data scientist working for Starbucks and you want to better understand why some stores in Brooklyn, New York, perform better than others.

To begin, let's outline a workflow: 

- Get and explore your company's data
- Create areas of influence for your stores
- Enrich your data with demographic data
- And finally, share the results of your analysis with your team

Let's get started!

### Get and explore your company's data

[This dataset](https://github.com/CartoDB/cartoframes/blob/develop/examples/files/starbucks_brooklyn.csv) is the one you have to start your exploration. It contains information about the location of Starbucks and each store's annual revenue. As a first exploratory step, you read it into a Jupyter Notebook using pandas.


```python
import pandas as pd

starbucks_df = pd.read_csv('../files/starbucks_brooklyn.csv')
starbucks_df.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Franklin Ave &amp; Eastern Pkwy</td>
      <td>341 Eastern Pkwy,Brooklyn, NY 11238</td>
      <td>1321040.772</td>
    </tr>
    <tr>
      <th>1</th>
      <td>607 Brighton Beach Ave</td>
      <td>607 Brighton Beach Avenue,Brooklyn, NY 11235</td>
      <td>1268080.418</td>
    </tr>
    <tr>
      <th>2</th>
      <td>65th St &amp; 18th Ave</td>
      <td>6423 18th Avenue,Brooklyn, NY 11204</td>
      <td>1248133.699</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bay Ridge Pkwy &amp; 3rd Ave</td>
      <td>7419 3rd Avenue,Brooklyn, NY 11209</td>
      <td>1185702.676</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Caesar's Bay Shopping Center</td>
      <td>8973 Bay Parkway,Brooklyn, NY 11214</td>
      <td>1148427.411</td>
    </tr>
  </tbody>
</table>
</div>

To be able to display your stores as points on a map, you first have to convert the `address` column into geometries. This process is called geocoding and CARTO provides a straightforward way to do it (you can learn more about it in the [location data services guide](/developers/cartoframes/guides/Location-Data-Services)).

In order to geocode, you have to set your CARTO credentials. If you don't know your API key yet, check the [authentication guide](/developers/cartoframes/guides/Authentication/) to learn how to get it. In case you want to see the result of the geocoding without being logged in, here it is the [geocoded dataset](https://github.com/CartoDB/cartoframes/blob/develop/examples/files/starbucks_brooklyn_iso_enriched.csv).

> Note: If you don't have an account yet, you can get a trial, or a free account if you are a student, by [signing up here](https://carto.com/signup/).


```python
from cartoframes.auth import set_default_credentials

set_default_credentials('creds.json')
```

Now, we are ready to geocode the dataframe:


```python
from cartoframes.data.services import Geocoding

starbucks_df, _ = Geocoding().geocode(starbucks_df, street='address')
starbucks_df.head()
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>address</th>
      <th>revenue</th>
      <th>gc_status_rel</th>
      <th>carto_geocode_hash</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Franklin Ave &amp; Eastern Pkwy</td>
      <td>341 Eastern Pkwy,Brooklyn, NY 11238</td>
      <td>1321040.772</td>
      <td>0.91</td>
      <td>9212e0e908d8c64d07c6a94827322397</td>
      <td>POINT (-73.95901 40.67109)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>607 Brighton Beach Ave</td>
      <td>607 Brighton Beach Avenue,Brooklyn, NY 11235</td>
      <td>1268080.418</td>
      <td>0.97</td>
      <td>b1bbfe2893914a350193969a682dc1f5</td>
      <td>POINT (-73.96122 40.57796)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>65th St &amp; 18th Ave</td>
      <td>6423 18th Avenue,Brooklyn, NY 11204</td>
      <td>1248133.699</td>
      <td>0.95</td>
      <td>e47cf7b16d6c9b53c63e86a0418add1d</td>
      <td>POINT (-73.98976 40.61912)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bay Ridge Pkwy &amp; 3rd Ave</td>
      <td>7419 3rd Avenue,Brooklyn, NY 11209</td>
      <td>1185702.676</td>
      <td>0.95</td>
      <td>2f21749c02f73116892eb3b6fd5d5738</td>
      <td>POINT (-74.02744 40.63152)</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Caesar's Bay Shopping Center</td>
      <td>8973 Bay Parkway,Brooklyn, NY 11214</td>
      <td>1148427.411</td>
      <td>0.95</td>
      <td>134c23973313802448365db6235783f9</td>
      <td>POINT (-74.00098 40.59321)</td>
    </tr>
  </tbody>
</table>
</div>



Done! Now that the stores are geocoded, you will notice a new column named `geometry` has been added. This column stores the geographic location of each store and it's used to plot each location on the map.

You can quickly visualize your geocoded dataframe using the Map and Layer classes. Check out the [visualization guide](/developers/cartoframes/guides/Visualization) to learn more about the visualization capabilities inside of CARTOframes.


```python
from cartoframes.viz import Map, Layer

Map(Layer(starbucks_df))
```

<div class="example-map">
    <iframe
        id="quickstart_guide_starbucks_brooklyn"
        src="https://cartoframes.carto.com/kuviz/f442503a-f612-403d-b171-95e9138d0188"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>


Great! We have a map!

Now, you have a better sense about where the stores are. To continue with your exploration, you want to know where are the stores with more revenue. To do so, you can use the `size_continuous_layer` visualization layer:


```python
from cartoframes.viz.helpers import size_continuous_layer

Map(size_continuous_layer(starbucks_df, 'revenue', 'Revenue in $'))
```

<div class="example-map">
    <iframe
        id="quickstart_guide_starbucks_brooklyn_by_revenue"
        src="https://cartoframes.carto.com/kuviz/2e27384e-0cb4-4c02-809f-efa618ee5a6c"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>

Good job! By using the [size continuous visualization layer](/developers/cartoframes/examples/#example-size-continuous-layer) you can see right away where the stores with higher revenue are. By default, visualization layers also provide a popup with the mapped value and an appropriate legend.

### Create your areas of influence

Similar to geocoding, there is a straightforward method for creating isochrones to define your areas of influence. Isochrones are concentric polygons that display equally calculated levels over a given surface area measured by time.

For our analysis, let's create isochrones for each store that cover the area within a 15 minute walk.

To do this we will use the Isolines data service:


```python
from cartoframes.data.services import Isolines

isochrones_df, _ = Isolines().isochrones(starbucks_df, [15*60], mode='walk')
Map([
    Layer(isochrones_df),
    Layer(starbucks_df)]
)
```

<div class="example-map">
    <iframe
        id="quickstart_guide_starbucks_brooklyn_iso"
        src="https://cartoframes.carto.com/kuviz/17291073-7c93-431e-9dad-07e5da307a6f"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>

There they are! To learn more about creating isochrones and isodistances check out the [location data services guide](/developers/cartoframes/guides/Location-Data-Services).


### Enrich your data with demographic data

Now that you have the area of influence calculated for each store, let's augment the result with population information to help better understand a store's average revenue per person.

First, let's find the demographic variable we need. We will use the `Catalog` class that can be filter by country and category. In our case, we have to look for USA demographics datasets. Let's check which geographies (spatial resolution) are available.

```python
from cartoframes.data.observatory import Catalog

Catalog().country('usa').category('demographics').geographies
```


<pre class="u-topbottom-Margin"><code>[<Geography('ags_blockgroup_1c63771c')>,
<Geography('ags_q17_4739be4f')>,
<Geography('mbi_blockgroups_1ab060a')>,
<Geography('mbi_counties_141b61cd')>,
<Geography('mbi_county_subd_e8e6ea23')>,
<Geography('mbi_pc_5_digit_4b1682a6')>,
<Geography('usct_blockgroup_f45b6b49')>,
<Geography('usct_cbsa_6c8b51ef')>,
<Geography('usct_censustract_bc698c5a')>,
<Geography('usct_congression_b6336b2c')>,
<Geography('usct_county_ec40c962')>,
<Geography('usct_place_12d6699f')>,
<Geography('usct_puma_b859f0fa')>,
<Geography('usct_schooldistr_515af763')>,
<Geography('usct_schooldistr_da72a4cb')>,
<Geography('usct_schooldistr_287be4f7')>,
<Geography('usct_state_4c8090b5')>,
<Geography('usct_zcta5_75071016')>]
</code></pre>

This time, let's choose the block groups from AGS and check which datasets are available.

```python
Catalog().country('usa').category('demographics').geography('ags_blockgroup_1c63771c').datasets
```

<pre class="u-topbottom-Margin"><code>[<Dataset('ags_sociodemogr_e92b1637')>,
<Dataset('ags_consumerspe_fe5d060a')>,
<Dataset('ags_retailpoten_ddf56a1a')>,
<Dataset('ags_consumerpro_e8344e2e')>,
<Dataset('ags_businesscou_a8310a11')>,
<Dataset('ags_crimerisk_9ec89442')>]
</code></pre>

Nice! The population variables are inside of the sociodemographic category, let take a look at what options are available and the associated descriptions.


```python
from cartoframes.data.observatory import Dataset

dataset = Dataset.get('ags_sociodemogr_e92b1637')
variables_df = dataset.variables.to_dataframe()
variables_df[variables_df['description'].str.contains('population', case=False)]
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>agg_method</th>
      <th>column_name</th>
      <th>dataset_id</th>
      <th>db_type</th>
      <th>description</th>
      <th>id</th>
      <th>name</th>
      <th>slug</th>
      <th>starred</th>
      <th>summary_json</th>
      <th>variable_group_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>31</th>
      <td>SUM</td>
      <td>AGECYGT85</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 85+ (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECYGT85</td>
      <td>AGECYGT85_b9d8a94d</td>
      <td>False</td>
      <td>{'head': [1, 0, 0, 2, 2, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>32</th>
      <td>SUM</td>
      <td>AGECYGT25</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population Age 25+ (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECYGT25</td>
      <td>AGECYGT25_433741c7</td>
      <td>False</td>
      <td>{'head': [6, 3, 0, 18, 41, 0, 0, 0, 0, 0], 'ta...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>33</th>
      <td>SUM</td>
      <td>AGECYGT15</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population Age 15+ (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECYGT15</td>
      <td>AGECYGT15_681a1204</td>
      <td>False</td>
      <td>{'head': [6, 3, 0, 20, 959, 0, 0, 0, 0, 0], 't...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>34</th>
      <td>SUM</td>
      <td>AGECY8084</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 80-84 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY8084</td>
      <td>AGECY8084_b25d4aed</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>35</th>
      <td>SUM</td>
      <td>AGECY7579</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 75-79 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY7579</td>
      <td>AGECY7579_15dcf822</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>36</th>
      <td>SUM</td>
      <td>AGECY7074</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 70-74 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY7074</td>
      <td>AGECY7074_6da64674</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>37</th>
      <td>SUM</td>
      <td>AGECY6064</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 60-64 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY6064</td>
      <td>AGECY6064_cc011050</td>
      <td>False</td>
      <td>{'head': [1, 2, 0, 0, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>38</th>
      <td>SUM</td>
      <td>AGECY5559</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 55-59 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY5559</td>
      <td>AGECY5559_8de3522b</td>
      <td>False</td>
      <td>{'head': [1, 0, 0, 2, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>39</th>
      <td>SUM</td>
      <td>AGECY5054</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 50-54 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY5054</td>
      <td>AGECY5054_f599ec7d</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>40</th>
      <td>SUM</td>
      <td>AGECY4549</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 45-49 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY4549</td>
      <td>AGECY4549_2c44040f</td>
      <td>False</td>
      <td>{'head': [1, 0, 0, 3, 3, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>41</th>
      <td>SUM</td>
      <td>AGECY4044</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 40-44 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY4044</td>
      <td>AGECY4044_543eba59</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>42</th>
      <td>SUM</td>
      <td>AGECY3034</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 30-34 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY3034</td>
      <td>AGECY3034_86a81427</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 5, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>43</th>
      <td>SUM</td>
      <td>AGECY2529</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 25-29 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY2529</td>
      <td>AGECY2529_5f75fc55</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 31, 0, 0, 0, 0, 0], 'tai...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>44</th>
      <td>SUM</td>
      <td>AGECY1519</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 15-19 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY1519</td>
      <td>AGECY1519_66ed0078</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 488, 0, 0, 0, 0, 0], 'ta...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>45</th>
      <td>SUM</td>
      <td>AGECY0509</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 5-9 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY0509</td>
      <td>AGECY0509_c74a565c</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>46</th>
      <td>SUM</td>
      <td>AGECY0004</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 0-4 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY0004</td>
      <td>AGECY0004_bf30e80a</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>49</th>
      <td>SUM</td>
      <td>AGECY2024</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 20-24 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY2024</td>
      <td>AGECY2024_270f4203</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 1, 430, 0, 0, 0, 0, 0], 'ta...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>50</th>
      <td>SUM</td>
      <td>AGECY1014</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 10-14 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY1014</td>
      <td>AGECY1014_1e97be2e</td>
      <td>False</td>
      <td>{'head': [0, 2, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>51</th>
      <td>SUM</td>
      <td>AGECY3539</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 35-39 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY3539</td>
      <td>AGECY3539_fed2aa71</td>
      <td>False</td>
      <td>{'head': [0, 1, 0, 1, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>55</th>
      <td>SUM</td>
      <td>POPPY</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>FLOAT</td>
      <td>Population (2024A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>POPPY</td>
      <td>POPPY_946f4ed6</td>
      <td>False</td>
      <td>{'head': [0, 0, 8, 0, 0, 0, 4, 0, 2, 59], 'tai...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>66</th>
      <td>SUM</td>
      <td>SEXCYMAL</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population male (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>SEXCYMAL</td>
      <td>SEXCYMAL_ca14d4b8</td>
      <td>False</td>
      <td>{'head': [1, 2, 0, 13, 374, 0, 0, 0, 0, 0], 't...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>67</th>
      <td>SUM</td>
      <td>SEXCYFEM</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population female (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>SEXCYFEM</td>
      <td>SEXCYFEM_d52acecb</td>
      <td>False</td>
      <td>{'head': [5, 3, 0, 9, 585, 0, 0, 0, 0, 0], 'ta...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>75</th>
      <td>SUM</td>
      <td>POPCYGRPI</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Institutional Group Quarters Population (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>POPCYGRPI</td>
      <td>POPCYGRPI_147af7a9</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>76</th>
      <td>SUM</td>
      <td>POPCYGRP</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population in Group Quarters (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>POPCYGRP</td>
      <td>POPCYGRP_74c19673</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 959, 0, 0, 0, 0, 0], 'ta...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>77</th>
      <td>SUM</td>
      <td>POPCY</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>POPCY</td>
      <td>POPCY_f5800f44</td>
      <td>False</td>
      <td>{'head': [6, 5, 0, 22, 959, 0, 0, 0, 0, 0], 't...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>92</th>
      <td>SUM</td>
      <td>HISCYHISP</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population Hispanic (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>HISCYHISP</td>
      <td>HISCYHISP_f3b3a31e</td>
      <td>False</td>
      <td>{'head': [0, 0, 0, 0, 36, 0, 0, 0, 0, 0], 'tai...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>96</th>
      <td>SUM</td>
      <td>LBFCYLBF</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population In Labor Force (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>LBFCYLBF</td>
      <td>LBFCYLBF_59ce7ab0</td>
      <td>False</td>
      <td>{'head': [0, 2, 0, 10, 378, 0, 0, 0, 0, 0], 't...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>99</th>
      <td>SUM</td>
      <td>LBFCYPOP16</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population Age 16+ (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>LBFCYPOP16</td>
      <td>LBFCYPOP16_53fa921c</td>
      <td>False</td>
      <td>{'head': [6, 3, 0, 20, 959, 0, 0, 0, 0, 0], 't...</td>
      <td>None</td>
    </tr>
    <tr>
      <th>107</th>
      <td>SUM</td>
      <td>AGECY6569</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>INTEGER</td>
      <td>Population age 65-69 (2019A)</td>
      <td>carto-do.ags.demographics_sociodemographic_usa...</td>
      <td>AGECY6569</td>
      <td>AGECY6569_b47bae06</td>
      <td>False</td>
      <td>{'head': [2, 0, 0, 7, 0, 0, 0, 0, 0, 0], 'tail...</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>


We can see the variable that contains the population for 2019 is the one with the slug `POPCY_f5800f44`. Now we are ready to enrich our areas of influence with that variable.

```python
from cartoframes.data.observatory import Variable
from cartoframes.data.observatory import Enrichment

variable = Variable.get('POPCY_f5800f44')

isochrones_df = Enrichment().enrich_polygons(isochrones_df, [variable])
isochrones_df.head()
```

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>data_range</th>
      <th>geometry</th>
      <th>POPCY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>900</td>
      <td>MULTIPOLYGON (((-73.95934 40.68011, -73.96074 ...</td>
      <td>1221.455789</td>
    </tr>
    <tr>
      <th>1</th>
      <td>900</td>
      <td>MULTIPOLYGON (((-73.96187 40.58632, -73.96289 ...</td>
      <td>1534.248133</td>
    </tr>
    <tr>
      <th>2</th>
      <td>900</td>
      <td>MULTIPOLYGON (((-73.99082 40.62694, -73.99170 ...</td>
      <td>1311.667005</td>
    </tr>
    <tr>
      <th>3</th>
      <td>900</td>
      <td>MULTIPOLYGON (((-74.02851 40.64062, -74.02882 ...</td>
      <td>1179.999175</td>
    </tr>
    <tr>
      <th>4</th>
      <td>900</td>
      <td>MULTIPOLYGON (((-74.00110 40.60186, -74.00249 ...</td>
      <td>1334.454790</td>
    </tr>
  </tbody>
</table>
</div>

Great! Let's see the result on a map:


```python
from cartoframes.viz.helpers import color_continuous_layer

Map(color_continuous_layer(isochrones_df, 'POPCY', 'Population'))
```

<div class="example-map">
    <iframe
        id="quickstart_guide_starbucks_brooklyn_pop"
        src="https://cartoframes.carto.com/kuviz/2f8f2c92-f8ed-4d1d-8e09-04ef85ad80dd"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>

We can see that the area of influence of the store on the right, is the one with the highest population. Let's go a bit further and calculate and visualize the average revenue per person.

```python
starbucks_df = starbucks_df.reset_index(drop=True)
starbucks_df['rev_pop'] = starbucks_df['revenue']/isochrones_df['POPCY']
Map(size_continuous_layer(starbucks_df, 'rev_pop', 'Revenue per person'))
```

<div class="example-map">
    <iframe
        id="quickstart_guide_starbucks_brooklyn_rev_pop"
        src="https://cartoframes.carto.com/kuviz/6b3e7bc2-cca2-4172-9f6e-73deb71cde03"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>

As we can see, there are clearly 3 stores that have lower revenue per person. This insight will help us to focus on them in further analyses.

To learn more about discovering the data you want, check out the [data discovery guide](/developers/cartoframes/guides/Data-discovery). To learn more about enriching your data check out the [data enrichment guide]().

### Publish and share your results

The final step in the workflow is to share this interactive map with your colleagues so they can explore the information on their own. Let's do it!

First, let's add widgets so people are able to see some graphs of the information and filter it. To do this, we only have to add `widget=True` to the visualization layers. Remember to check the [visualization guide](/developers/cartoframes/guides/Visualization) to learn more.

```python
result_map = Map([
    color_continuous_layer(isochrones_df, 'POPCY', 'Population', stroke_width=0, opacity=0.7),
    size_continuous_layer(starbucks_df, 'rev_pop', 'Rev/Pop', stroke_color='white', widget=True)
])
result_map
```

<div class="example-map">
    <iframe
        id="quickstart_guide_startbucks_analysis"
        src="https://cartoframes.carto.com/kuviz/3c900d1f-d3ef-472f-9bc0-a2005a08df27"
        width="100%"
        height="500"
        style="margin: 20px auto !important"
        frameBorder="0">
    </iframe>
</div>

Cool! Now that you have a small dashboard to play with, let's publish it on CARTO so you are able to share it with anyone. To do this, you just need to call the publish method from the Map class:


```python
result_map.publish('startbucks_analysis')
```

<pre class="u-topbottom-Margin"><code>{'id': '3c900d1f-d3ef-472f-9bc0-a2005a08df27',
 'url': 'https://cartoframes.carto.com/kuviz/3c900d1f-d3ef-472f-9bc0-a2005a08df27',
 'name': 'startbucks_analysis',
 'privacy': 'public'}
</code></pre>

### Conclusion

Congratulations! You have finished this guide and have a sense about how CARTOframes can speed up your workflow. To continue learning, you can check the specific [guides](/developers/cartoframes/guides), check the [reference](/developers/cartoframes/reference) to know everything about a class or a method or check the [examples](/developers/cartoframes/examples).
