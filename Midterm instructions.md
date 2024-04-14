# Midterm exam repository
This file is a summary of all the main strings necessary for successfully passing the midterm exam

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
*Note:* Import satelite only, impacts are not relevant unless requested.

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

*Note:* So far you have imported the IOT of all 44 countries and 5 RoW regions per the 163 industries or 200 products. In the exam you'll be asked to focus on a specific region. These are the acronyms in the ISO. 

<img width="637" alt="Screenshot 2024-04-14 at 11 46 14" src="https://github.com/Ghirlo00/EEIOA---Midterm-repository/assets/166986311/03751485-a013-421d-abef-f2b5a9584c17">

## II. Calculate the matrices of the MRIO table
So far we imported A, Y, F and Fhh. With these we can calculate the other matrices as explained in the _PDF_ document.
I - identity of A 
L - Leontieff inverse - (I - A)<sup>-1</sup>
x - product output - L * Y --> Some x values are equal to 0. These cannot be inverted therefore we want to select only those that are >0.
f - extension coeffiecients - F * x_invense

--> _Why axis=1 in formula for x?_

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
