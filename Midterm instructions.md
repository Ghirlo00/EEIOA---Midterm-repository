# Midterm exam repository
This file is a summary of all the main strings necessary for successfully passing the midterm exam.
A different version of this repository can be found here [explainations](https://colab.research.google.com/drive/1Z0jPk2rJgV6SCm6RD1QUDaOAOYYPNT3D?usp=drive_link#scrollTo=uV0ApU9UNmDh) 

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

## III. Calculate the footprint of a nation
Footprints are calculated differently depending on the impacts connected. Carbon footprint has a global impact (GWP) therefore its calculation doesn't need to account for specific exports of the impact in other countries. Whereas, Water footprint is a local impact that must account for the implications of exporting production.
In this course we have discussed two types of comprehensive footprints: 

--> **Dashboard** = Carbon, Water, Land and Employement

--> **Material** = Biomass, Fossil fuels, Metal ores, Non-Metallic minerals

The images below are summaries of the main formulas that are applied in the code.

<img width="610" alt="Screenshot 2024-04-14 at 12 27 37" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/cd6e5339-e1cf-4297-ae6e-09457942834c">

<img width="613" alt="Screenshot 2024-04-14 at 12 27 43" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/c265d19a-44df-49b9-a8ec-79deba7200b3">

### Import population values for per capita calculations
Download the excel file. Rename accurately. Drop it in the right folder. Call pop2015 with the requested year. If asked to select a row use for example `pop2015 = population.loc["AT","y2015"]`

```python
# Import population data
population = pd.read_excel('data/exiobase_PopulationGDP_1995_2019.xlsx',sheet_name='Population', index_col=[0, 1, 2])
pop2015 = population.loc[:,"y2015"]
```

### Carbon footprint of a nation
We start by creating a modified finald demand matrix (Y_mod) that accounts for only those categories we account in the calculation in the specific nation (example Netherlands)

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

--> _Somansh: Explain when you diagonalize f and when Y. Is it because the f is used to analyse how an impact is exported up the supply chain while Y...?_
