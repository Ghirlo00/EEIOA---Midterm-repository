# Midterm exam repository
This file is a summary of all the main strings necessary for successfully passing the midterm exam.
A different version of this repository can be found here [explainations](https://colab.research.google.com/drive/1Z0jPk2rJgV6SCm6RD1QUDaOAOYYPNT3D?usp=drive_link#scrollTo=uV0ApU9UNmDh) 

__Note:__ In case the following code snippets were not to work properly you can resort to use the following original files
1. [Land Use](https://github.com/Ghirlo00/EEIOA---Midterm-repository/blob/main/Land%20use.ipynb)
2. [UK - Non-metallic Minerals vs. Metal ores](https://github.com/Ghirlo00/EEIOA---Midterm-repository/blob/main/UK%20-%20Non-metallic%20Minerals%20vs.%20Metal%20Ores..ipynb)
3. [US_CN - Carbon Footprint]([https://github.com/Ghirlo00/EEIOA---Midterm-repository/blob/main/Land%20use.ipynb](https://github.com/Ghirlo00/EEIOA---Midterm-repository/blob/main/US_CN%20-%20Carbon%20footprint.ipynb))


## General Instructions
1. Please ensure that you run each cell (ctrl+enter) so that your inputs are saved
2. All questions have either coding cell or a text cell, or a combination of the two.
3. You are allowed to create additional cells for each answer to facilitate your work
4. For every uncertainty related to a specific type of code you can search on Google `df.name of the formula`

## I. Download and import Exiobase data
The instructions of the exam will give the year and the type of table (IxI or PxP) that you will need to import. 
Go to https://zenodo.org/record/5589597 and save the .zip file into the folder connected to your Jupyter.

```python
# Import modules
import pandas as pd
import numpy as np
```
### Import exiobase
Beware: exiobase is composed by large datasets so it may take some time to load and process
**Note:** Import satelite only, impacts are not relevant unless requested.

Change the destination of the `path = ` in order to get the files from the correct folder.

```python
# Import here your data
path = "data/IOT_2016_ixi/"
A = pd.read_csv(f'{path}A.txt', sep='\t', index_col=[0, 1], header=[0, 1])
Y = pd.read_csv(f'{path}Y.txt' , sep='\t', index_col=[0, 1], header=[0, 1])
```

```python
# Import satellite accounts
F_sat = pd.read_csv(f'{path}satellite/F.txt' , sep='\t', index_col=[0], header=[0, 1])
F_sat_hh = pd.read_csv(f'{path}satellite/F_y.txt' , sep='\t', index_col=[0], header=[0, 1])
```
In alternative, if requested a specific extension:

```python
# Import satellite accounts
F_sat_CO2 = F_sat[F_sat.index.str.contains("CO2")].sum(axis=0)
F_sat_hh_CO2 = F_sat_hh[F_sat_hh.index.str.contains("CO2")].sum(axis=0)
```

**Note:** So far you have imported the IOT of all 44 countries and 5 RoW regions per the 163 industries or 200 products. In the exam you'll be asked to focus on a specific region. These are the acronyms in the ISO. 

<img width="637" alt="Screenshot 2024-04-14 at 11 46 14" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/03751485-a013-421d-abef-f2b5a9584c17">

## II. Calculate the matrices of the MRIO table
So far we imported A, Y, F and Fhh. With these we can calculate the other matrices as explained in the _PDF_ document.
I - identity of A 
L - Leontieff inverse - (I - A)<sup>-1</sup>
x - product output - L * Y --> Some x values are equal to 0. These cannot be inverted therefore we want to select only those that are >0.
f - extension coeffiecients - F * x_invense

**Note:** Axis 1 calculates along the rows, while axis 0 calculates along the columns.

```python
I = np.identity(A.shape[0])
L = np.linalg.inv(I-A)
x = L @ Y.sum(axis=1)
```
```python
x_ = x.copy()
x_[x_!=0] = 1/x_[x_!=0]
inv_diag_x_ = np.diag(x_)
```
```python
f = F * x_inv

f
```

## III. Ecological footprint
Footprints are calculated differently depending on the impacts connected. Carbon footprint has a global impact (GWP) therefore its calculation doesn't need to account for specific exports of the impact in other countries. Whereas, Water footprint is a local impact that must account for the implications of exporting production.
In this course we have discussed two types of comprehensive footprints: 

--> **Dashboard** = Carbon, Water, Land and Employement

--> **Material** = Biomass, Fossil fuels, Metal ores, Non-Metallic minerals

The images below are summaries of the main formulas that are applied in the code.

<img width="610" alt="Screenshot 2024-04-14 at 12 27 37" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/cd6e5339-e1cf-4297-ae6e-09457942834c">

<img width="613" alt="Screenshot 2024-04-14 at 12 27 43" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/c265d19a-44df-49b9-a8ec-79deba7200b3">
  
### 3.1 Territorial Footprint
A territorial footprint only accounts for the impacts produced directly in a territory. These can be:
1. __Carbon footprint__ - In this example it uses only f_sat_CO2 but could use the extended version with other indicators.

```python
e_CO2_pba = np.diag(f_sat_CO2) @ x
e_CO2_pba = e_CO2_pba.reshape(r,s).sum(1) + F_sat_hh_CO2_reg

e_CO2_pba_pp = e_CO2_pba/pop2015.values/1000 #convert unit from kg to tonne/capita

e_CO2_pba_pp.loc[["US","CN"]]
```

2. __Material footprint__ - In this example the material is the Non-metalli minerals in the UK.
```python
F_minerals_ = F_sat.loc[F_sat.index.str.contains("Domestic Extraction Used - Non-Metallic Minerals")]
F_minerals_hh = F_sat_hh.loc[F_sat_hh.index.str.contains("Domestic Extraction Used - Non-Metallic Minerals")]
F_minerals_tot_ter = pd.concat([F_minerals_.loc[:,"GB"], F_minerals_hh.loc[:,"GB"]], axis=1).sum().sum()
F_minerals_tot_ter
```

### 3.2 National Footprint
This calculation accounts also for the indirect emissions caused by the consumption demand of the nation.
We start by creating a modified finald demand matrix (Y_mod) that accounts for only those categories we account in the calculation in the specific nation (example Netherlands)

Sometime it might be necessary to group all the Y for doing the calculation:
```python
Y_reg = Y.groupby(level=0, axis=1, sort=False).sum()
Y_reg
```
Then always:
```python
Y_mod = Y.loc[:,"NL"]
Y_mod
```

Then we isolate the extantion in which we are interested
```python
indicator = "GHG emissions (GWP100) | Problem oriented approach: baseline (CML, 2001) | GWP100 (IPCC, 2007)"
```
```python
# the intensity vector in which we are interested
f_ =  f.loc[indicator]
f_
```
```python
# the final demand CO2 emissions
e_hh_ = F_hh.loc[indicator, "NL"]
```
```python
# Calculate the total global footprint
e_total_reg = f_ @ L @ Y_mod.sum(axis=1) + e_hh_.sum()
e_total_reg
```

### 3.3 Emissions Embodied in Trade
These calculations are necessary to understand if a country is a net importer or exporter of an impact.

__Example 1:__ was the UK a net importer or net exporter of non-metallic minerals in 2016? 
```python
net_import = e_minerals - F_minerals_tot_ter
net_import
```
Top three regions from which the UK imported non-metallic minerals in 2016:
```python
e_minerals = np.diag(f_minerals) @ L @ Y_reg

e_minerals.index = Y.index

region_labels = Y.index.to_frame(index=False).region.unique()
sector_labels = Y.index.to_frame(index=False).sector.unique()

CBA_minerals = pd.DataFrame(e_minerals.loc[:, "GB"].values.reshape(49,163), columns=sector_labels, index=region_labels).sum(axis=1)

CBA_minerals[CBA_minerals.index!="GB"].sort_values(ascending=False).head(3)
```

__Example 2:__ Were the US and China net importers or exporters of CO2 emissions in 2015?
```python
net_import = (e_CO2 - e_CO2_pba)*1e-9 # convert from kg to million metric ton
net_import.loc[["US","CN"]]
```
top three regional contributers of US' and China's carbon footprints:
```python
e_CO2_cont = np.diag(f_sat_CO2) @ L @ Y_reg
e_CO2_US = e_CO2_cont.loc[:, "US"]
e_CO2_CN = e_CO2_cont.loc[:, "CN"]

e_CO2_US.index = Y.index
e_CO2_CN.index = Y.index

# print(e_CO2_US.groupby(level=0).sum().nlargest(n=3))
# print(e_CO2_CN.groupby(level=0).sum().nlargest(n=3))

print(e_CO2_US.groupby(level=0).sum().sort_values(ascending=False)[:3])
print(e_CO2_CN.groupby(level=0).sum().sort_values(ascending=False)[:3])
```
how much CO2 emissions did the US outsourced to China in 2015
```python
net_import = (e_CO2 - e_CO2_pba)*1e-9 # convert from kg to million metric ton
net_import.loc[["US","CN"]]
```
top three regional contributers of US' and China's carbon footprints:
```python
e_CO2_US_to_CN = e_CO2_US.loc["CN"].sum() - e_CO2_CN.loc["US"].sum()
e_CO2_US_to_CN * 1e-9
```

### 3.4 Pro capita footprint
Download the excel file. Rename accurately. Drop it in the right folder. Call pop2015 with the requested year. If asked to select a row use for example `pop2015 = population.loc["AT","y2015"]`

```python
# Import population data
population = pd.read_excel('data/exiobase_PopulationGDP_1995_2019.xlsx',sheet_name='Population', index_col=[0, 1, 2])
pop2015 = population.loc[:,"y2015"]
```
```python
# Aggregate Y by region
Y_reg = Y.groupby(level=0, axis=1, sort=False).sum()
```
```python
# Aggregate F_sat_hh by region
F_sat_hh_CO2_reg = F_sat_hh_CO2.groupby(level=0, sort=False).sum()
```
```python
# Footprint calculation
e_CO2 = f_sat_CO2 @ L @ Y_reg + F_sat_hh_CO2_reg
e_CO2_pp = e_CO2/pop2015.values/1000 #convert the unit from kg to metric ton/capita

e_CO2_pp
```

### 3.5 Extended footprint
Relating to the eventuality of calculations that account for other extensions. Ex_ not only CO2 but also CH4 and N2O

```python
F_sat_CO2 = F_sat[F_sat.index.str.contains("CO2")].sum(axis=0)
F_sat_CH4 = F_sat[F_sat.index.str.contains("CH4")].sum(axis=0)*29.8
F_sat_N2O = F_sat[F_sat.index.str.contains("N2O")].sum(axis=0)*273

F_GHG_ = pd.concat([F_sat_CO2, F_sat_CH4, F_sat_N2O],axis=1).T
F_GHG_.index = ["CO2", "CH4", "N2O"]
F_GHG_
```
```python
# Intensities satellite
f_GHG = F_GHG_ @ inv_diag_x_ 
f_GHG
```
```python
# Households satellites
F_sat_hh_CO2 = F_sat_hh[F_sat_hh.index.str.contains("CO2")].sum(axis=0)
F_sat_hh_CH4 = F_sat_hh[F_sat_hh.index.str.contains("CH4")].sum(axis=0)*29.8
F_sat_hh_N2O = F_sat_hh[F_sat_hh.index.str.contains("N2O")].sum(axis=0)*273

F_hh_GHG_ = pd.concat([F_sat_hh_CO2, F_sat_hh_CH4, F_sat_hh_N2O],axis=1).T
F_hh_GHG_.index = ["CO2", "CH4", "N2O"]

# Aggregate F_sat_hh by region
F_hh_GHG_reg = F_hh_GHG_.groupby(level=0, axis=1, sort=False).sum()

F_hh_GHG_reg
```

```python
# Footprint calculation
e_CO2 = f_GHG.loc["CO2"] @ L @ Y_reg + F_hh_GHG_reg.loc["CO2"]
e_CH4 = f_GHG.loc["CH4"] @ L @ Y_reg + F_hh_GHG_reg.loc["CH4"]
e_N2O = f_GHG.loc["N2O"] @ L @ Y_reg + F_hh_GHG_reg.loc["N2O"]

e_CO2eq = e_CO2 + e_CH4 + e_N2O

# CO2_eq per capita
e_CO2eq_pp = e_CO2eq/pop2015.values/1000 #convert the unit from kg to metric ton/capita

e_CO2eq_pp
```

__Other request:__ What is the proportion of CO2 emissions in CO2e in each region's carbon footprint measured in CO2e?
```python
# Proportion of CO2 emissions to GWP by country
e_GWP = e_CO2 + e_CH4 + e_N2O
CO2_to_CO2eq = e_CO2/e_GWP * 100

CO2_to_CO2eq, CO2_to_CO2eq.max(), CO2_to_CO2eq.min() 
```

### 3.6 Environmental vs Economic footprint analysis
Using the "impact" accounts, what were the value added footprints of the US and China, respectively, in 2015?
```python
# Import impact accounts
F_imp = pd.read_csv(f'{path}impacts/F.txt' , sep='\t', index_col=[0], header=[0, 1])
F_imp_hh = pd.read_csv(f'{path}impacts/F_y.txt' , sep='\t', index_col=[0], header=[0, 1])
```
```python
# VA coefficients
VA = F_imp[F_imp.index.str.contains("Value Added")]
f_VA = VA.values @ inv_diag_x_
e_VA = f_VA @ L @ Y_reg

e_VA_pp = e_VA/pop2015.values*1e6 

e_VA_pp.loc[:, ["US", "CN"]]
```

### 3.7 Dashboard analysis
Quantify the reliance of the UK on each of the two global regions, EU27 and Non-EU27, in 2016 concerning Non-Metallic minerals and Metal Ores.
```python
F_metals_ = F_sat.loc[F_sat.index.str.contains("Domestic Extraction Used - Metal Ores")]
F_metals_hh = F_sat_hh.loc[F_sat_hh.index.str.contains("Domestic Extraction Used - Metal Ores")]

# Total territorial emissions
F_metals_tot_ter = F_metals_.loc[:,"GB"].sum().sum() + F_metals_hh.loc[:,"GB"].sum().sum()
F_metals_tot_ter
```
```python
geo_e_minerals = e_minerals.groupby(level=0, axis=0, sort=False).sum()
```
```python
geo_e_minerals.GB.iloc[:27].sum()
```
```python
minerals_UK_CBA = geo_e_minerals.loc[:, "GB"] 
minerals_UK_PBA = geo_e_minerals.loc["GB", :]

minerals_UK_trade_rel = minerals_UK_CBA - minerals_UK_PBA

minerals_UK_trade_rel_EU = minerals_UK_trade_rel.iloc[:27].sum()
minerals_UK_trade_rel_ROW = minerals_UK_trade_rel.iloc[-21:].sum()

UK_dependency_minerals = pd.DataFrame([minerals_UK_trade_rel_EU, minerals_UK_trade_rel_ROW], index=["UK_dependency_on_EU","UK_dependency_on_ROW"], columns=["non_metallic_minerals, kt"]) 

UK_dependency_minerals
```
```python
f_metals = F_metals_.sum() @ inv_diag_x_ 
```
```python
e_metals = np.diag(f_metals) @ L @ Y.groupby(level=0, axis=1, sort=False).sum()
e_metals.index = Y.index
```
```python
geo_e_metals = e_metals.groupby(level=0, axis=0, sort=False).sum()
```
```python
metals_UK_CBA = geo_e_metals.loc[:, "GB"] 
metals_UK_PBA = geo_e_metals.loc["GB", :]

metals_UK_trade_rel = metals_UK_CBA - metals_UK_PBA

metals_UK_trade_rel_EU = metals_UK_trade_rel.iloc[:27].sum()
metals_UK_trade_rel_ROW = metals_UK_trade_rel.iloc[-21:].sum()


UK_dependency_metals = pd.DataFrame([metals_UK_trade_rel_EU, metals_UK_trade_rel_ROW], index=["UK_dependency_on_EU","UK_dependency_on_ROW"], columns=["metal_ores, kt"]) 

UK_dependency_metals
```
```python
dashboard = pd.concat([UK_dependency_minerals, UK_dependency_metals],axis=1)
dashboard
```
```python
dashboard_relative = round(dashboard / np.array([minerals_UK_CBA.sum(), metals_UK_CBA.sum()]) *100, 2)
dashboard_relative.columns = ["non_metallic_minerals, %", "metal_ores, %"]
dashboard_relative
```
