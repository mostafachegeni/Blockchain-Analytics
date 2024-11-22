# 4_Cardano_Analysis__Address_Entity_Level


In the following sections, each cell in the corresponding 'code' (with a similar filename in the code directory of this repository) is documented with a brief description of its purpose and functionality, followed by the summarized cell code.

***

### Import Libraries and Set Constant Variables

This cell imports a comprehensive set of libraries and modules required for data analysis, visualization, multiprocessing, and Spark-based operations. It also initializes certain environment variables, constants, and performs some basic computations.

#### Explanation of the Code:
1. **Library Imports**:
   - **Core libraries**: `numpy`, `array`, `csv`, `datetime`, and `os`.
   - **Search and sorting**: `bisect`.
   - **Visualization**: `matplotlib.pyplot`.
   - **Parallel processing**: `multiprocessing` and `threading`.
   - **Big data processing**: `pyspark` and `pandas`.
   - **Graph processing**: `networkx` and `community`.
   - **Probability distributions**: `powerlaw`.
   - **Progress tracking**: `tqdm`.
   - **Serialization**: `pickle`.

2. **Environment Variable Setup**:
   - Disables timezone warnings for PyArrow with `os.environ`.

3. **Cardano Blockchain Date and Address Analysis**:
   - Sets constants for unique address counts and calculates the total time length of the Cardano blockchain in days.

4. **Outputs**:
   - Prints address count statistics and a success message.

#### Cell Code:
```python
# Import libraries and set constant variables:

import numpy as np
from array import *
import csv

# using datetime module
import datetime;

# Binary Search
from bisect import bisect_left
from bisect import bisect_right

import matplotlib.pyplot as plt
import json

import multiprocessing as mp
from multiprocessing import Process, Queue
from multiprocessing import current_process
import queue
import threading

import os
os.environ["PYARROW_IGNORE_TIMEZONE"] = "1"
from pyspark.sql import SparkSession
import pyspark.pandas as ps
from pyspark.sql.functions import col
import pandas as pd

import random
import networkx as nx
from tqdm import tqdm

# https://python-louvain.readthedocs.io/en/latest/api.html
# community.modularity(partition, graph, weight='weight')
from community import modularity

import pickle

import powerlaw

print('----------------------')
# unique_payment_addresses_len = len(unique_payment_addresses)
unique_raw_addresses_len        = 40330345
unique_payment_addresses_len    = 40324960
unique_delegation_addresses_len = 3868049
print('unique_raw_addresses_len        = ', unique_raw_addresses_len)
print('unique_payment_addresses_len    = ', unique_payment_addresses_len)
print('unique_delegation_addresses_len = ', unique_delegation_addresses_len)

INITIAL_DATE_CARDANO      = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO        = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds()/86400) + 1

print('----------------------')
print('done!')
```


***

### Define Base and Temporary Directory Paths

This cell sets up the base and temporary directory paths to organize and manage file storage for the Cardano project.

#### Explanation of the Code:
1. **`BASE_ADDRESS`**:
   - Defines the base directory for storing exported data related to the project.

2. **`TEMP_ADDRESS`**:
   - Defines a subdirectory within the base directory to store temporary files.

These paths centralize file storage management and ensure consistent directory usage throughout the project.

#### Cell Code:
```python
BASE_ADDRESS = '/local/scratch/exported/Cardano_MCH_2023_1/'
TEMP_ADDRESS = BASE_ADDRESS + '/temp_files/'
```

***



### Define Required Methods

This cell contains a set of essential utility functions and algorithms to support various computational tasks, such as data manipulation, searching, clustering, graph processing, and file I/O operations. Each function is explained below.

#### Explanation of the Code:
1. **Time Logging**:
   - Prints the current timestamp to record the start of execution.

2. **Parent Array Operations**:
   - `parent`: Retrieves the parent of a node in a Union-Find structure.
   - `find_parent`: Finds the root parent of a node using path compression.
   - `link_address`: Implements the Union-Find algorithm to link two nodes.
   - `resolveAll`: Resolves all parent-child relationships to find ultimate roots.

3. **Clustering Operations**:
   - `remapClusterIds`: Reassigns cluster IDs to ensure a contiguous sequence starting from zero.

4. **Merge Parent Arrays**:
   - `merge_parents`: Combines two parent arrays into a unified structure.

5. **Binary Search**:
   - `BinarySearch`: Performs a binary search for a specific element.
   - `BinarySearch_Find_start_end`: Finds the start and end indices of a target element in a sorted array.

6. **File Operations**:
   - `store_array_to_file`, `store_array_to_file_2D`: Save arrays (1D or 2D) to files using CSV or JSON.
   - `load_file_to_array`, `load_file_to_array_2D`: Load arrays from CSV or JSON files.
   - `store_dict_to_file_INT`: Saves a dictionary to a file with integer keys and values.
   - `load_file_to_dict_INT`: Loads a dictionary from a file with integer keys and values.

7. **Graph Processing**:
   - `add_edge_info`: Adds weighted edges between nodes in a graph structure.

8. **Address Extraction**:
   - `extract_payment_delegation_parts`: Splits raw addresses into payment and delegation parts based on Cardano's address types (Byron or Shelley).

9. **Statistical Calculations**:
   - `gini_index`: Computes the Gini index for measuring inequality in wealth distribution.

#### Cell Code:
```python
# Define required methods:

print('----------------------')

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

##########################################################################################
def parent (id1, parents_array):
    return parents_array[id1]

##########################################################################################
def find_parent (id1, parents_array):
    while (id1 != parent(id1, parents_array)):
        new_parent = parent(parent(id1, parents_array), parents_array)
        id1 = new_parent
    return id1

##########################################################################################
# Link two addresses based on "Union-Find" Algorithm:
def link_address (addr_position_1, addr_position_2, parents_array):
    id1 = find_parent(addr_position_1, parents_array)
    id2 = find_parent(addr_position_2, parents_array)
    if (id1 == id2):
        return
    if id1 < id2:
        id1, id2 = id2, id1
    parents_array[id1] = id2
    return

##########################################################################################
def resolveAll (parents_array):
    for id1 in tqdm(range(len(parents_array))):
        parents_array[id1] = find_parent(id1, parents_array)
    return

##########################################################################################
def remapClusterIds (parents_array, clustering_array):
    cluster_count = 0
    place_holder = 9999999999999
    new_cluster_ids = [place_holder] * len(parents_array)
    for i in range(len(clustering_array)):
        clustering_array[i] = parents_array[i]
    for i in tqdm(range(len(clustering_array))):
        parent_index = clustering_array[i]
        if (new_cluster_ids [parent_index] == place_holder):
            new_cluster_ids [parent_index] = cluster_count
            cluster_count += 1
        clustering_array[i] = new_cluster_ids [parent_index]
    return cluster_count

##########################################################################################
def merge_parents(parents_array, parents_merged):
    if (len(parents_array) != len(parents_merged)):
        print('parents_merged Error: -1 (Length)')
        return -1
    for i in tqdm(range(len(parents_merged))):
        link_address(i, parents_array[i], parents_merged)

##########################################################################################
def BinarySearch(a, x, debug=True):
    i = bisect_left(a, x)
    if i < len(a) and a[i] == x:
        return i
    else:
        if(debug):
            print('BinarySearch Error: -1')
        return -1

##########################################################################################
def BinarySearch_Find_start_end(a, x):
    i = bisect_left(a, x)
    j = bisect_right(a, x) - 1
    if i < len(a) and a[i] == x and j < len(a) and a[j] == x:
        return [i, j]
    else:
        print('BinarySearch Error: -1')
        print('i = ', i)
        print('j = ', j)
        return -1

##########################################################################################
def store_array_to_file (input_array_name, file_name, index_=False, header_=None):
    ct = datetime.datetime.now()
    print('start time (Store Array to ' + file_name + '): ', ct)
    df = pd.DataFrame(input_array_name)
    df.to_csv(file_name, index=index_, header=header_)
    et = datetime.datetime.now() - ct
    print('elapsed time (Store Array to ' + file_name + '): ', et)
    return

##########################################################################################
def load_file_to_array (file_name, header_=None):
    ct = datetime.datetime.now()
    print('start time (Load ' + file_name  + ' to Array): ', ct)
    df = pd.read_csv(file_name, header=header_)
    output_array_name = df.to_numpy()
    et = datetime.datetime.now() - ct
    print('elapsed time (Load ' + file_name  + ' to Array): ', et)
    return output_array_name

##########################################################################################
def store_array_to_file_2D (input_array_name, file_name):
    ct = datetime.datetime.now()
    print('start time (Store Array 2D to ' + file_name + '): ', ct)
    with open(file_name, "w") as filehandle:
        json.dump(input_array_name, filehandle)
    et = datetime.datetime.now() - ct
    print('elapsed time (Store Array 2D to ' + file_name + '): ', et)
    return

##########################################################################################
def load_file_to_array_2D (file_name):
    ct = datetime.datetime.now()
    print('start time (Load ' + file_name  + ' to Array 2D): ', ct)
    with open(file_name) as filehandle:
        output_array_name = json.load(filehandle)
    et = datetime.datetime.now() - ct
    print('elapsed time (Load ' + file_name  + ' to Array 2D): ', et)
    return output_array_name

##########################################################################################
def store_dict_to_file_INT (input_dict_name, file_name):
    ct = datetime.datetime.now()
    print('start time (Store Dictionary to ' + file_name + '): ', ct)
    filehandle = csv.writer(open(file_name, 'w'))
    for key, val in input_dict_name.items():
        filehandle.writerow([key, val])
    et = datetime.datetime.now() - ct
    print('elapsed time (Store Dictionary to ' + file_name + '): ', et)
    return

##########################################################################################
def load_file_to_dict_INT (file_name):
    ct = datetime.datetime.now()
    print('start time (Load ' + file_name  + ' to Dictionary): ', ct)
    filehandle = csv.reader(open(file_name, 'r'))
    output_dict_name = {int(rows[0]):int(rows[1]) for rows in filehandle}
    et = datetime.datetime.now() - ct
    print('elapsed time (Load ' + file_name  + ' to Dictionary): ', et)
    return output_dict_name

##########################################################################################
def add_edge_info(node_1, node_2, edges_array, weight=1):
    if (node_1 == node_2):
        return
    if (node_1 < node_2):
        n1, n2 = node_2, node_1
    else:
        n1, n2 = node_1, node_2
    for i in range(weight):
        edges_array[n1].append(n2)
    return

##########################################################################################
def extract_payment_delegation_parts(address_raw, payment_cred, stake_address):
    if (address_raw == ''):
        return ['', '']
    if (address_raw[2] == '8'): # Byron Address
        if (payment_cred != '') or (stake_address != ''):
            return ['', '']
        payment_part, delegation_part = address_raw, ''
    else: # Shelley Address
        if (payment_cred == ''):
            return ['', '']
        payment_part, delegation_part = payment_cred, stake_address
    return [payment_part, delegation_part]

##########################################################################################
def gini_index(inp_array):
    array = np.array(inp_array).astype(float).flatten()
    if np.amin(array) < 0:
        array -= np.amin(array)
    array += 0.0000001
    array = np.sort(array)
    index = np.arange(1, array.shape[0] + 1)
    n = array.shape[0]
    return ((np.sum((2 * index - n - 1

```



***

### Read Sorted and Unique Address Lists from Files

This cell reads three sorted and unique address lists (`raw_address_list`, `payment_address_list`, and `delegation_address_list`) from files and stores them in memory.

#### Explanation of the Code:
1. **File Names and Formats**:
   - The `BASE_ADDRESS` path is combined with filenames to locate the address lists.
   - The input files are **text files** (`.txt`) containing sorted and unique address lists. The files have the following structure:
     - **Columns**:
       - Single column of strings, where each string represents an address.

2. **Loading Address Lists**:
   - The `load_file_to_array` function is used to read each file into a NumPy array.

3. **Address Lists**:
   - **Raw Addresses (`unique_raw_addresses`)**:
     - Contains unique raw addresses from Cardano transactions.
   - **Payment Addresses (`unique_payment_addresses`)**:
     - Contains unique payment addresses.
   - **Delegation Addresses (`unique_delegation_addresses`)**:
     - Contains unique delegation addresses.

4. **Output**:
   - Prints the length of each address list to verify successful loading.

#### Cell Code:
```python
# Read ("sorted" "unique" array_list) [raw_address_list/payment_address_list/delegation_address_list] from file:

print('----------------------')

file_name = BASE_ADDRESS + '/Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357.txt'
unique_raw_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)
print('Length of "unique_raw_addresses" = ' + str(len(unique_raw_addresses)))

file_name = BASE_ADDRESS + '/Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953.txt'
unique_payment_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)
print('Length of "unique_payment_addresses" = ' + str(len(unique_payment_addresses)))

file_name = BASE_ADDRESS + '/Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415.txt'
unique_delegation_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)
print('Length of "unique_delegation_addresses" = ' + str(len(unique_delegation_addresses)))

##########################################################################################
print('----------------------')
print('done!')

```



***


### Read Address Clustering Results from Files

This cell reads the results of heuristic-based address clustering from files. The clustering results are based on three scenarios: heuristic 1, heuristic 2, and a combination of both heuristics.

#### Explanation of the Code:
1. **File Names and Formats**:
   - The `BASE_ADDRESS` path is combined with filenames to locate the clustering results.
   - The input files are **CSV files** containing clustering data. The files are structured as follows:
     - **Columns**:
       - Single column with integer values representing the cluster IDs for each address.

2. **Loading Clustering Results**:
   - The `load_file_to_array` function is used to read each file into a NumPy array.

3. **Clustering Results**:
   - **Heuristic 1 (`clustering_array_heur1`)**:
     - Results from clustering using heuristic 1 (excluding smart contracts).
   - **Heuristic 2 (`clustering_array_heur2`)**:
     - Results from clustering using heuristic 2.
   - **Heuristic 1 and 2 Combined (`clustering_array_heur1and2`)**:
     - Results from clustering using both heuristics.

4. **Output**:
   - Successfully loads and stores clustering results for further processing.

#### Cell Code:
```python
# This cell reads the results of heuristic-based address clustering: based on heuristic 1, heuristic 2, as well as both heuristics 1 and 2

# Read clustering_array[] from file:

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223957.txt'
clustering_array_heur1 = load_file_to_array(file_name)  # CSV file, single column of integers (cluster IDs)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic2__Cardano_TXs_All__2023-03-26_110150.txt'
clustering_array_heur2 = load_file_to_array(file_name)  # CSV file, single column of integers (cluster IDs)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212.txt'
clustering_array_heur1and2 = load_file_to_array(file_name)  # CSV file, single column of integers (cluster IDs)

##########################################################################################
print('----------------------')
print('done!')


```




***

### Find the Number of Members in Each Cluster (Heuristic 1)

This cell calculates the number of members in each cluster based on the clustering results obtained using Heuristic 1.

#### Explanation of the Code:
1. **Input**:
   - `clustering_array_heur1`: A **NumPy array** containing the cluster IDs for each address. The array is loaded from a CSV file in a previous step.

2. **Steps**:
   - **Sort the Array**:
     - The `clustering_array_heur1` is sorted to enable efficient cluster member counting.
   - **Determine Number of Clusters**:
     - The total number of clusters is determined as the maximum cluster ID plus one.
   - **Count Members in Each Cluster**:
     - For each cluster ID, the `BinarySearch_Find_start_end` function is used to find the start and end indices of the cluster in the sorted array. The difference between these indices gives the number of members in the cluster.

3. **Output**:
   - Prints the number of clusters (`num_of_clusters_heur1`).
   - Prints the total number of unique addresses (as a verification step) by summing all cluster member counts.

4. **Performance**:
   - Logs the elapsed time to measure the efficiency of the computation.

#### Cell Code:
```python
# Find number of members in each cluster (Heur1):

print('----------------------')
ct = datetime.datetime.now()
print("start time: ", ct)

print('----------------------')
sorted_clustering_array_heur1 = np.sort(clustering_array_heur1, axis=None)  # Sort the flattened array
print('clustering_array_heur1 was sorted!')

num_of_clusters_heur1 = max(sorted_clustering_array_heur1) + 1  # Total number of clusters
print('number of clusters_heur1 = ', num_of_clusters_heur1)

# Initialize an array to store the number of members in each cluster
num_of_cluster_members_heur1 = np.array([0] * num_of_clusters_heur1)

# Count the members of each cluster
for i in tqdm(range(num_of_clusters_heur1)):
    x = BinarySearch_Find_start_end(sorted_clustering_array_heur1, i)  # Find start and end indices for cluster ID `i`
    num_of_cluster_members_heur1[i] = x[1] - x[0] + 1  # Calculate the number of members in the cluster

print('----------------------')
# Verify by summing the members of all clusters
print('number of unique addresses (to verify num_of_cluster_members_heur1) = ', sum(num_of_cluster_members_heur1))

print('----------------------')
et = datetime.datetime.now() - ct
print("elapsed time: ", et)

##########################################################################################
print('----------------------')
print('done!')

```



***


### Power-Law Distributions Analysis (Heuristic 1)

This cell analyzes the distribution of cluster sizes (number of members per cluster) using power-law fitting. It calculates key parameters of the power-law distribution and visualizes the results.

#### Explanation of the Code:
1. **Input**:
   - `num_of_cluster_members_heur1`: A **NumPy array** containing the number of members in each cluster.

2. **Steps**:
   - **Power-Law Fitting**:
     - The `powerlaw.Fit` function fits a power-law distribution to the cluster size data.
     - Key parameters are computed:
       - `alpha`: The scaling parameter of the power-law.
       - `sigma`: The standard error of `alpha`.
       - `xmin`: The minimum value above which the data follows a power-law.
       - `xmax`: The maximum value considered in the fitting.
     - Compares the power-law distribution to an exponential distribution.
   - **Visualization**:
     - Plots the probability density function (PDF) of the data on a log-log scale.
     - Displays the fitted power-law distribution and its key parameters (`alpha`, `sigma`, `xmin`).

3. **Output**:
   - Prints the power-law parameters and comparison result.
   - Saves the plot as `fig_cluster_mems_dist_heur1.pdf`.

#### Cell Code:
```python
# Power-law distributions (Heur 1):

fit = powerlaw.Fit(num_of_cluster_members_heur1)

print('fit.power_law.alpha = ', fit.power_law.alpha)
print('fit.power_law.sigma = ', fit.power_law.sigma)
print('fit.power_law.xmin = ',  fit.power_law.xmin)
print('fit.power_law.xmax = ',  fit.power_law.xmax)
print('fit.distribution_compare(\'power_law\', \'exponential\') = ', fit.distribution_compare('power_law', 'exponential'))

plt.style.use('https://raw.githubusercontent.com/benckj/mpl_style/main/uzh.mplstyle')

plt.xlabel('Entity (Cluster) Size')
plt.ylabel('Probability Density')

plt.ylim(bottom=1e-14)
markersize = 3

fig = fit.plot_pdf(linestyle='', marker='o', original_data=True)

# Add text for key parameters
plt.text(10**(5.0), 10**(-0.3), '\u03B1 = ' + str(round(fit.power_law.alpha, 2)) + '\n' + 
         '\u03C3 = ' + str(round(fit.power_law.sigma, 4)) + '\n' + 
         'xmin = ' + str(round(fit.power_law.xmin)), ha='left', va='top', fontsize=12)

# Function to plot a reference line
def abline(plt, slope, intercept, x0, x1):
    """Plot a line from slope and intercept"""
    x_vals = np.logspace(x0, x1, num=100)
    y_vals = 10**(intercept) * (x_vals**slope)
    plt.plot(x_vals, y_vals, '--')

# Plot the reference line
abline(plt, -fit.power_law.alpha, -0.4, 0.2, 5.7)

# Save the plot
plt.savefig('fig_cluster_mems_dist_heur1.pdf', bbox_inches='tight', facecolor='white')
plt.show()

```



***


### Find "Entity" of Each "Stake Address" (Delegation Address) [Heuristic 2]

This cell associates each stake address (delegation address) with its corresponding entity using Heuristic 2 by processing CSV files containing Cardano transaction data.

#### Explanation of the Code:

1. **Input Data**:
   - **`unique_delegation_addresses`**: A sorted array of unique stake addresses.
   - **`unique_payment_addresses`**: A sorted array of unique payment addresses.
   - **`clustering_array_heur2`**: Array containing entity indices for payment addresses (output of Heuristic 2).
   - **CSV Files**: Files generated by a PostgreSQL query containing Cardano transaction data with the following format:

     **CSV File Details**:
     - **Delimiter**: `|`
     - **Key Columns**:
       - `TX_ID`: Transaction ID.
       - `BLOCK_TIME`: Time of the block.
       - `EPOCH_NO`: Epoch number.
       - `INPUTs`: Aggregated input details (semicolon-separated). Each input is a comma-separated string with:
         - Input ID, Reference Transaction ID, Reference Output Index, Reference Output ID, Reference Output Raw Address, Reference Stake Address ID, Reference Output Value, Reference Output Script Indicator, Reference Payment Credential, Reference Stake Address, Reference Output Block Time.
       - `OUTPUTs`: Aggregated output details (semicolon-separated). Each output is a comma-separated string with:
         - Output ID, Output Raw Address, Output Stake Address ID, Output Value, Output Script Indicator, Output Payment Credential, Output Stake Address.

2. **Steps**:
   - **Initialization**:
     - An array `entity_of_stake_addresses` is initialized with placeholders.
   - **Iterate Through CSV Files**:
     - Load each file into a DataFrame.
     - Extract and parse the `OUTPUTs` column to retrieve details about payment and delegation parts of each address.
     - Use `extract_payment_delegation_parts` to identify the payment and delegation components.
     - For valid delegation addresses:
       - Find their indices in `unique_delegation_addresses` using binary search.
       - Map the payment address to its entity using `clustering_array_heur2` and store the result in `entity_of_stake_addresses`.

3. **Output**:
   - `entity_of_stake_addresses`: Array mapping each delegation address to its corresponding entity index.
   - Logs the elapsed time for processing each file and the total elapsed time for the operation.

4. **Performance Logging**:
   - Captures the elapsed time for loading and processing each file for efficiency analysis.

#### Cell Code:
```python
# Find "Entity" of each "Stake Address" (= Delegation Address) [Heur 2]:

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize date range and clustering data
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0
clustering_array = clustering_array_heur2

place_holder = 999999999999
entity_of_stake_addresses = np.array([place_holder] * len(unique_delegation_addresses))

# File details
CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_Velocity_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each CSV file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()
    
    # Load the CSV file
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    # Iterate through rows and extract entities for stake addresses
    for index, row in tqdm(df.iterrows()):
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        
        for tx_output in outputs_list:
            output_details = tx_output.split(',')
            address_raw = output_details[1]
            address_has_script = output_details[4]
            payment_cred = output_details[5]
            stake_address = output_details[6]

            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
            if address_payment_part != '' and address_delegation_part != '':
                deleg_addr_indx = BinarySearch(unique_delegation_addresses, address_delegation_part)
                if entity_of_stake_addresses[deleg_addr_indx] == place_holder:
                    paymnt_addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[paymnt_addr_indx][0]
                    entity_of_stake_addresses[deleg_addr_indx] = entity_indx

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Entities of Stake Addresses from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Entities of Stake Addresses): ", et)

print('----------------------')
print('done!')

```




***

### Load/Store `entity_of_stake_addresses` from/into File

This cell handles storing and loading the `entity_of_stake_addresses` array, which maps stake addresses to their respective entities, using the file system for persistence.

#### **Explanation of the Code**

1. **File Format**:
   - **File Type**: Text file (`.txt`).
   - **Delimiter**: One entity mapping per line.
   - **Columns**:
     - Single column containing the entity index corresponding to each stake address:
       - **Type**: Integer
       - **Description**: The entity ID associated with the stake address.
   - Example Content:
     ```
     12345
     67890
     11121
     ```

2. **Storing Data**:
   - The `entity_of_stake_addresses` array is saved into a file.
   - The filename includes a timestamp for versioning:
     - Format: `Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt`
   - Uses the `store_array_to_file` function to write the array to disk.

3. **Loading Data**:
   - The `load_file_to_array` function is used to load the file back into a NumPy array.
   - Requires the full file path and name, which is timestamp-specific.

4. **Purpose**:
   - Ensures persistence and reusability of the processed data.
   - Avoids recomputation by loading the results from previous computations.


#### **Cell Code**
```python
# Load/Store "entity_of_stake_addresses" from/into file:

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store "entity_of_stake_addresses" into file:
'''
output_filename = BASE_ADDRESS + '/Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(entity_of_stake_addresses, output_filename)
'''

# Load "entity_of_stake_addresses" from file:
file_name = BASE_ADDRESS + '/Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__2024-01-23_212107.txt'
entity_of_stake_addresses = load_file_to_array(file_name)

##########################################################################################
print('----------------------')
print('done!')


```



***


### Calculate Entity Balances [Heuristic 2]

This cell computes the ADA balances for entities identified using Heuristic 2 by processing transaction data from multiple CSV files. The balances are stored daily for incremental analysis.


#### **Explanation of the Code**

1. **Input Data**:
   - **`clustering_array_heur2`**:
     - Maps payment addresses to their respective entity indices.
   - **CSV Files**:
     - Format: `cardano_TXs_Velocity_?.csv`
     - Delimiter: `|`
     - Key Columns:
       - `TX_ID`: Transaction ID.
       - `BLOCK_TIME`: Time of the block.
       - `EPOCH_NO`: Epoch number.
       - `INPUTs`: List of input details (semicolon-separated). Each input contains:
         - `Input_ID,Ref_TX_ID,Ref_Output_Index,Ref_Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`
       - `OUTPUTs`: List of output details (semicolon-separated). Each output contains:
         - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.

2. **Steps**:
   - **Initialize Balances**:
     - `balances_per_entity_array`: Array initialized to zero, with one entry per entity.
   - **Process Each CSV File**:
     - Load the file and iterate through transactions.
     - Update balances based on UTXOs consumed in `INPUTs` and created in `OUTPUTs`.
     - Use `BinarySearch` to map payment addresses to entities using `clustering_array_heur2`.
   - **Daily Balance Snapshots**:
     - If a new day is encountered in `BLOCK_TIME`, the current balances are serialized into a pickle file for persistence.

3. **File Output**:
   - **Daily Snapshot Files**:
     - File format: Pickle (`.pickle`).
     - Filename: `YuZhang__BalancesPerEntityDay_Heur2_<Day>__Cardano_TXs_All.pickle`
     - Content:
       - `balances_per_entity_array`: Array containing balances for all entities as of the given day.

4. **Purpose**:
   - Provides a time-series view of entity balances based on daily transaction data.
   - Enables incremental analysis without reprocessing all data.


#### **Cell Code**
```python
# Entity Balances [Heur 2]:

print('----------------------')
import random
import pickle

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize constants and balances array
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

clustering_array = clustering_array_heur2
balances_per_entity_array = [0] * (np.amax(clustering_array) + 1)
current_delta_day = 0

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_Velocity_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each CSV file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    # Iterate through transactions
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Store daily balance snapshot
        if current_delta_day < tx_delta_day:
            output_filename = BASE_ADDRESS + '/YuZhang_Cardano_Balances_Entities_Heur2__PICKLE/YuZhang__BalancesPerEntityDay_Heur2_' + str(current_delta_day).zfill(4) + '__Cardano_TXs_All.pickle'
            pickle.dump(balances_per_entity_array, open(output_filename, 'wb'))
            current_delta_day = tx_delta_day

        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Update balances based on inputs
        for input_data in inputs_list:
            address_raw = input_data.split(',')[4]
            payment_cred = input_data.split(',')[8]
            stake_address = input_data.split(',')[9]
            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
            if address_payment_part != '':
                addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                entity_indx = clustering_array[addr_indx][0]
                UTXO_value = int(input_data.split(',')[6])
                balances_per_entity_array[entity_indx] -= UTXO_value

        # Update balances based on outputs
        for output_data in outputs_list:
            address_raw = output_data.split(',')[1]
            payment_cred = output_data.split(',')[5]
            stake_address = output_data.split(',')[6]
            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
            if address_payment_part != '':
                addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                entity_indx = clustering_array[addr_indx][0]
                UTXO_value = int(output_data.split(',')[3])
                balances_per_entity_array[entity_indx] += UTXO_value

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Entity Balances [Heur 2] from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Entity Balances [Heur 2]): ", et)

print('----------------------')
print('done!')
```


#### **Example Output**
- **Daily Snapshot File**:
  - File: `YuZhang__BalancesPerEntityDay_Heur2_0001__Cardano_TXs_All.pickle`
  - Content: Serialized `balances_per_entity_array` for day 1.



***

### Calculate Number of Active Delegators/Rewarders (Entities) Per Epoch [Heuristic 2]

This cell calculates the number of active delegators and rewarders (addresses and entities) per epoch based on data from a CSV file. It also computes the staking and rewards per entity for each epoch, storing the results in serialized files for further analysis.


#### **Explanation of the Code**

1. **Input Data**:
   - **Epoch Range**:
     - `first_epoch_no = 210`: Staking began on 2020-08-08.
     - `last_epoch_no = 391`: Ended on 2023-01-30.
   - **CSV File**:
     - Format: `cardano_pools_4.csv`
     - Delimiter: `|`
     - Key Columns:
       - `EPOCH`: Epoch number.
       - `DELEGATORs`: List of delegator details (semicolon-separated). Each entry contains:
         - `Delegator_ID,Stake_Amount,Delegator_Stake_Addr`
       - `REWARDERs`: List of rewarder details (semicolon-separated). Each entry contains:
         - `Rewarder_ID,Reward_Amount,Rewarder_Stake_Addr`.

2. **Steps**:
   - **Initialization**:
     - Arrays to count delegator/rewarder addresses and entities per epoch.
     - Sets to track unique delegators and rewarders for the current epoch.
     - Arrays to store staking and rewards per entity (`stake_deleg_by_entities`, `reward_by_entities`).
   - **Processing Epochs**:
     - For each epoch, calculate:
       - **Active Delegators**:
         - Extract delegator addresses and entities using `BinarySearch` and `entity_of_stake_addresses`.
         - Add their staking amounts to `stake_deleg_by_entities`.
       - **Active Rewarders**:
         - Extract rewarder addresses and entities similarly and update `reward_by_entities`.
     - Save the results for each epoch as serialized files.
   - **Handling Epoch Transitions**:
     - At the end of each epoch:
       - Count and store the unique delegators and rewarders for that epoch.
       - Reset the sets for the next epoch.

3. **File Output**:
   - **Per Epoch Snapshots**:
     - Delegation and rewards are stored as Pickle files.
     - Filename format:
       - `StakeDelegPerEntityEpoch_Heur2_<Epoch>__Cardano_TXs_All.pickle`
       - `RewardPerEntityEpoch_Heur2_<Epoch>__Cardano_TXs_All.pickle`.



#### **Cell Code**
```python
# Calculate Number of Active Delegators/Rewarders (Entities) Per Epoch [Heuristic 2]

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize epoch ranges and dates
first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)

epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(total_num_of_epochs)]

# Initialize counters
num_delegator_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]
num_delegator_entities_per_epoch = [0 for _ in range(total_num_of_epochs)]
num_rewarder_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]
num_rewarder_entities_per_epoch = [0 for _ in range(total_num_of_epochs)]

current_epoch = first_epoch_no

# Load CSV file
file_name = BASE_ADDRESS + '/cardano_pools_4.csv'
df = pd.read_csv(file_name, delimiter='|')

# Initialize sets and arrays for tracking data
delegator_addresses_per_epoch_SET = set()
delegator_entities_per_epoch_SET = set()
reward_addresses_per_epoch_SET = set()
reward_entities_per_epoch_SET = set()

clustering_array = clustering_array_heur2

stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)
reward_by_entities = [0] * (np.amax(clustering_array) + 1)

# Process each row in the CSV file
for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    if EPOCH < first_epoch_no:
        continue

    if EPOCH > current_epoch:
        # Save epoch data
        num_delegator_addresses_per_epoch[current_epoch - first_epoch_no] = len(delegator_addresses_per_epoch_SET)
        num_delegator_entities_per_epoch[current_epoch - first_epoch_no] = len(delegator_entities_per_epoch_SET)
        num_rewarder_addresses_per_epoch[current_epoch - first_epoch_no] = len(reward_addresses_per_epoch_SET)
        num_rewarder_entities_per_epoch[current_epoch - first_epoch_no] = len(reward_entities_per_epoch_SET)

        delegator_addresses_per_epoch_SET.clear()
        delegator_entities_per_epoch_SET.clear()
        reward_addresses_per_epoch_SET.clear()
        reward_entities_per_epoch_SET.clear()

        # Store staking and rewards
        output_filename = BASE_ADDRESS + f'/YuZhang_Cardano_StakeDelegation_Entities_Heur2__PICKLE/StakeDelegPerEntityEpoch_Heur2_{str(current_epoch).zfill(4)}__Cardano_TXs_All.pickle'
        pickle.dump(stake_deleg_by_entities, open(output_filename, 'wb'))
        stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)

        output_filename = BASE_ADDRESS + f'/YuZhang_Cardano_Reward_Entities_Heur2__PICKLE/RewardPerEntityEpoch_Heur2_{str(current_epoch).zfill(4)}__Cardano_TXs_All.pickle'
        pickle.dump(reward_by_entities, open(output_filename, 'wb'))
        reward_by_entities = [0] * (np.amax(clustering_array) + 1)

        current_epoch = EPOCH

    if EPOCH > last_epoch_no:
        break

    # Process delegators
    for delegator in list(df.loc[index, 'DELEGATORs'].split(';')):
        deleg_stake_addr = delegator.split(',')[2]
        deleg_addr_indx = BinarySearch(unique_delegation_addresses, deleg_stake_addr, debug=False)
        if deleg_addr_indx != -1:
            deleg_entity_indx = entity_of_stake_addresses[deleg_addr_indx][0]
            stake_deleg_by_entities[deleg_entity_indx] += int(delegator.split(',')[1])
            delegator_addresses_per_epoch_SET.add(deleg_addr_indx)
            delegator_entities_per_epoch_SET.add(deleg_entity_indx)

    # Process rewarders
    if not pd.isna(df.loc[index, 'REWARDERs']):
        for rewarder in list(df.loc[index, 'REWARDERs'].split(';')):
            reward_stake_addr = rewarder.split(',')[2]
            reward_addr_indx = BinarySearch(unique_delegation_addresses, reward_stake_addr, debug=False)
            if reward_addr_indx != -1:
                reward_entity_indx = entity_of_stake_addresses[reward_addr_indx][0]
                reward_by_entities[reward_entity_indx] += int(rewarder.split(',')[1])
                reward_addresses_per_epoch_SET.add(reward_addr_indx)
                reward_entities_per_epoch_SET.add(reward_entity_indx)

et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')

```


***
### Calculate "Active Delegators (Stake Addresses)" Per Epoch

This cell calculates the total number of active delegator addresses per epoch using data from the `cardano_pools_3.csv` file.


#### **Input File Details: `cardano_pools_3.csv`**

- **File Type**: CSV
- **Delimiter**: `,`
- **Columns**:
  - **`EPOCH`**:
    - Type: Integer
    - Description: Epoch number.
  - **`POOL_HASH_BECH32`**:
    - Type: String
    - Description: Bech32-formatted pool hash.
  - **`POOL_STAKES`**:
    - Type: Integer
    - Description: Total amount of ADA staked in the pool during the epoch.
  - **`POOL_REWARDS`**:
    - Type: Integer
    - Description: Total rewards distributed by the pool during the epoch.
  - **`NUM_OF_DELEGATORS`**:
    - Type: Integer
    - Description: Number of unique delegator addresses staking in the pool.
  - **`NUM_OF_REWARDERS`**:
    - Type: Integer
    - Description: Number of unique rewarder addresses receiving rewards from the pool.


#### **Explanation of the Code**

1. **Epoch Range**:
   - Epochs processed range from:
     - `first_epoch_no = 210` (2020-08-08)
     - `last_epoch_no = 391` (2023-01-30).

2. **Steps**:
   - **Initialization**:
     - Create an array `active_delegator_addresses_per_epoch` to store the count of active delegators for each epoch.
   - **Processing Rows**:
     - For each row in the CSV file:
       - Extract the epoch number (`EPOCH`) and the number of delegators (`NUM_OF_DELEGATORS`).
       - Add the count of delegators for the pool to the total for the corresponding epoch.
   - **Epoch Filtering**:
     - Skip rows for epochs outside the specified range.

3. **Purpose**:
   - Provides a time-series view of the number of active delegator addresses per epoch.



#### **Cell Code**
```python
# "Active Delegators (Stake Addresses)" Per Epoch:

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize epoch ranges and dates
first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)

epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(total_num_of_epochs)]

# Initialize array for active delegators per epoch
active_delegator_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]

current_epoch = first_epoch_no

# Load CSV file
file_name = BASE_ADDRESS + '/cardano_pools_3.csv'
df = pd.read_csv(file_name, delimiter=',')

# Process each row in the CSV file
for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    NUM_OF_DELEGATORS = int(df.loc[index, 'NUM_OF_DELEGATORS'])

    # Skip epochs outside the range
    if EPOCH < first_epoch_no:
        continue
    if EPOCH > last_epoch_no:
        break

    # Update active delegators count for the epoch
    active_delegator_addresses_per_epoch[EPOCH - first_epoch_no] += NUM_OF_DELEGATORS

# Calculate total elapsed time
et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')
```



***

### Calculate "Active Delegators (Entities)" Per Epoch

This cell computes the number of unique active delegator entities per epoch, based on data from the `cardano_pools_4.csv` file. It also stores the staking amounts by entity for each epoch.


#### **Input File Details: `cardano_pools_4.csv`**

- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`EPOCH`**:
    - Type: Integer
    - Description: Epoch number.
  - **`POOL_ID`**:
    - Type: Integer
    - Description: Unique identifier for the staking pool.
  - **`POOL_HASH_BECH32`**:
    - Type: String
    - Description: Bech32-formatted pool hash.
  - **`POOL_STAKES`**:
    - Type: Integer
    - Description: Total ADA staked in the pool during the epoch.
  - **`POOL_REWARDS`**:
    - Type: Integer
    - Description: Total rewards distributed by the pool during the epoch.
  - **`NUM_OF_DELEGATORS`**:
    - Type: Integer
    - Description: Number of unique delegator addresses staking in the pool.
  - **`NUM_OF_REWARDERS`**:
    - Type: Integer
    - Description: Number of unique rewarder addresses receiving rewards from the pool.
  - **`DELEGATORs`**:
    - Type: String
    - Description: Semicolon-separated list of delegator details. Each entry contains:
      - `Delegator_ID,Stake_Amount,Delegator_Stake_Addr`
  - **`REWARDERs`**:
    - Type: String
    - Description: Semicolon-separated list of rewarder details. Each entry contains:
      - `Rewarder_ID,Reward_Amount,Rewarder_Stake_Addr`


#### **Explanation of the Code**

1. **Epoch Range**:
   - Epochs processed range from:
     - `first_epoch_no = 210` (2020-08-08)
     - `last_epoch_no = 391` (2023-01-30).

2. **Steps**:
   - **Initialization**:
     - Create arrays to count unique active delegator entities and addresses for each epoch.
     - Initialize sets to track unique delegators for the current epoch.
   - **Processing Rows**:
     - For each row in the CSV file:
       - Extract delegator details (`DELEGATORs`).
       - Use `BinarySearch` and `entity_of_stake_addresses` to map stake addresses to entity indices.
       - Track unique delegators at both address and entity levels.
       - Accumulate staking amounts by entity in `stake_deleg_by_entities`.
   - **Epoch Transitions**:
     - At the end of each epoch:
       - Store counts of unique delegators (addresses and entities).
       - Save staking amounts by entity to a file.
       - Reset tracking sets for the next epoch.
   - **Output**:
     - Writes staking and reward distributions by entity into separate text files for each epoch.

3. **File Output**:
   - **Staking and Rewards Per Entity**:
     - File format: Text (`.txt`).
     - Filename examples:
       - `StakeDelegPerEntityEpoch_<Epoch>__Cardano_TXs_All.txt`
       - `RewardPerEntityEpoch_<Epoch>__Cardano_TXs_All.txt`
     - Content:
       - Array of staking/reward amounts for each entity.


#### **Cell Code**
```python
# Calculate "Active Delegators (Entities)" Per Epoch

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize epoch ranges and dates
first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)
epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(total_num_of_epochs)]

# Initialize counters
num_delegator_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]
num_delegator_entities_per_epoch = [0 for _ in range(total_num_of_epochs)]

current_epoch = first_epoch_no

# Load CSV file
file_name = BASE_ADDRESS + '/cardano_pools_4.csv'
df = pd.read_csv(file_name, delimiter='|')

# Initialize sets and arrays for tracking data
delegator_addresses_per_epoch_SET = set()
delegator_entities_per_epoch_SET = set()
stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)

# Process each row in the CSV file
for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    DELEGATORs = list(df.loc[index, 'DELEGATORs'].split(';'))

    # Skip epochs outside the range
    if EPOCH < first_epoch_no:
        continue

    if EPOCH > current_epoch:
        # Save epoch data
        num_delegator_addresses_per_epoch[current_epoch - first_epoch_no] = len(delegator_addresses_per_epoch_SET)
        num_delegator_entities_per_epoch[current_epoch - first_epoch_no] = len(delegator_entities_per_epoch_SET)

        delegator_addresses_per_epoch_SET.clear()
        delegator_entities_per_epoch_SET.clear()

        # Save staking data
        output_filename = BASE_ADDRESS + f'/YuZhang_Cardano_StakeDelegation_Entities/StakeDelegPerEntityEpoch_{str(current_epoch).zfill(4)}__Cardano_TXs_All.txt'
        store_array_to_file(stake_deleg_by_entities, output_filename)
        stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)

        current_epoch = EPOCH

    # Process delegators
    for delegator in DELEGATORs:
        deleg_stake_addr = delegator.split(',')[2]
        deleg_addr_indx = BinarySearch(unique_delegation_addresses, deleg_stake_addr, debug=False)
        if deleg_addr_indx != -1:
            deleg_entity_indx = entity_of_stake_addresses[deleg_addr_indx][0]
            stake_deleg_by_entities[deleg_entity_indx] += int(delegator.split(',')[1])
            delegator_addresses_per_epoch_SET.add(deleg_addr_indx)
            delegator_entities_per_epoch_SET.add(deleg_entity_indx)

et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')

```



***

### Load/Store Aggregated Metrics Per Epoch

This cell handles the loading and storing of the following metrics per epoch:

1. **`num_delegator_addresses_per_epoch`**: Number of delegator addresses.
2. **`num_delegator_entities_per_epoch`**: Number of delegator entities.
3. **`num_rewarder_addresses_per_epoch`**: Number of rewarder addresses.
4. **`num_rewarder_entities_per_epoch`**: Number of rewarder entities.


#### **Explanation of the Code**

1. **File Format**:
   - **File Type**: Text file (`.txt`).
   - **Delimiter**: One value per line.
   - **Content**: Array values representing the respective metric for each epoch.

2. **Storing Metrics**:
   - Each metric array is stored in a separate text file.
   - Filenames include a timestamp for versioning:
     - Example: `Num_Delegator_addresses_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt`
   - The `store_array_to_file` function writes the arrays to disk.

3. **Loading Metrics**:
   - Metrics are loaded from corresponding text files.
   - The `load_file_to_array` function is used to read the arrays into memory.

4. **Purpose**:
   - Allows persistence of calculated metrics for further analysis without recalculating.
   - Facilitates reproducibility and version control with timestamped files.



#### **Cell Code**
```python
# Load/Store "num_delegator_addresses_per_epoch",
#            "num_delegator_entities_per_epoch",
#            "num_rewarder_addresses_per_epoch",
#            "num_rewarder_entities_per_epoch" from/into file:

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store metrics into files:
'''
output_filename = BASE_ADDRESS + '/Num_Delegator_addresses_per_epoch__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(num_delegator_addresses_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Delegator_entities_per_epoch__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(num_delegator_entities_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(num_rewarder_addresses_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Rewarder_entities_per_epoch__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(num_rewarder_entities_per_epoch, output_filename)
'''

# Load metrics from files:
file_name = BASE_ADDRESS + '/Num_Delegator_addresses_per_epoch__Cardano_TXs_All__2023-12-17_094541.txt'
num_delegator_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Delegator_entities_per_epoch__Cardano_TXs_All__2023-12-17_094541.txt'
num_delegator_entities_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__2023-12-17_132620.txt'
num_rewarder_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_entities_per_epoch__Cardano_TXs_All__2023-12-17_132620.txt'
num_rewarder_entities_per_epoch = load_file_to_array(file_name)

##########################################################################################
print('----------------------')
print('done!')

```



***


### Find Number of New Entities Over Time

This cell determines the number of new entities created each day, based on transaction data from multiple CSV files.


#### **Explanation of the Code**

1. **Objective**:
   - To calculate when each entity is first observed (i.e., the day it becomes "active").
   - Store the result in an array `entities_new_per_day_array`.

2. **Input Data**:
   - **Transaction CSV Files**:
     - Format: `cardano_TXs_<file_number>.csv`.
     - Delimiter: `|`
     - Key Columns:
       - `TX_ID`: Transaction ID.
       - `BLOCK_TIME`: Block timestamp of the transaction.
       - `EPOCH_NO`: Epoch number of the transaction.
       - `OUTPUTs`: Semicolon-separated list of transaction outputs. Each output contains:
         - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.

3. **Steps**:
   - **Initialization**:
     - `entities_new_per_day_array`: Array initialized with a placeholder value (`999999999999`).
     - `current_delta_day`: Tracks the current day in terms of elapsed days since the start of the Cardano network (`INITIAL_DATE_CARDANO`).
   - **Processing CSV Files**:
     - For each file:
       - Load the file into a DataFrame.
       - For each transaction output:
         - Extract the payment address (`address_payment_part`) and map it to an entity using `clustering_array_heur1and2`.
         - If the entity has not been observed before (`place_holder`), record the current day in the `entities_new_per_day_array`.
   - **Epoch Filtering**:
     - Exclude addresses associated with smart contracts (`address_has_script != 'f'`).

4. **Purpose**:
   - Tracks the time evolution of new entities in the network.
   - Useful for analyzing adoption and growth over time.


#### **Cell Code**
```python
# Find Number of New "Entities" Over Time

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize constants
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0
place_holder = 999999999999
entities_new_per_day_array = np.array([place_holder] * (max(clustering_array_heur1and2)[0] + 1))

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each CSV file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    # Load CSV file
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    # Process each row in the DataFrame
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        if tx_delta_day != current_delta_day:
            current_delta_day = tx_delta_day

        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            address_has_script = tx_output.split(',')[4]
            payment_cred = tx_output.split(',')[5]
            stake_address = tx_output.split(',')[6]

            if address_has_script == 'f':  # Exclude smart contract addresses
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array_heur1and2[addr_indx][0]
                    if entities_new_per_day_array[entity_indx] == place_holder:
                        entities_new_per_day_array[entity_indx] = current_delta_day

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (New Entities over Time from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (New Entities over Time): ", et)
print('----------------------')
print('done!')

```



***


### Load/Store `entities_new_per_day_array` From/Into File

This cell handles the persistence of the `entities_new_per_day_array` data, which tracks the day each entity is first observed. The data can be stored into or loaded from a text file.


#### **File Format**

- **File Type**: Text file (`.txt`)
- **Delimiter**: One value per line
- **Content**:
  - Array values representing the day each entity was first observed. The index corresponds to the entity index, and the value represents the day since the start of the Cardano network (`INITIAL_DATE_CARDANO`).


#### **Steps**

1. **Storing Data**:
   - Write the `entities_new_per_day_array` to a text file.
   - File is named with a timestamp for version control:
     - Example: `newPerDay_Entities__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt`.

2. **Loading Data**:
   - Load the array from a specified file.
   - Use the `load_file_to_array` function to restore the array into memory.


#### **Cell Code**
```python
# Load/Store "entities_new_per_day_array" from/into file

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store "entities_new_per_day_array" into file:
'''
output_filename = BASE_ADDRESS + '/newPerDay_Entities__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(entities_new_per_day_array, output_filename)
'''

# Load "entities_new_per_day_array" from file:
file_name = BASE_ADDRESS + '/newPerDay_Entities__Cardano_TXs_All__2023-04-22_012525.txt'
entities_new_per_day_array = load_file_to_array(file_name)

##########################################################################################
print('----------------------')
print('done!')

```



***


### Find Number of New "Byron", "Shelley", and "Stake" Addresses Over Time

This cell calculates the number of new addresses (Byron, Shelley, and Stake) observed each day using data from multiple transaction CSV files.


#### **Explanation of the Code**

1. **Objective**:
   - Track the first observation day for each unique address type:
     - **Raw Addresses**: All raw addresses in outputs.
     - **Byron Payment Addresses**: Payment addresses specific to Byron-era transactions.
     - **Shelley Payment Addresses**: Payment addresses specific to Shelley-era transactions.
     - **Delegation (Stake) Addresses**: Stake addresses used in delegation.

2. **Input Data**:
   - **Transaction CSV Files**:
     - Format: `cardano_TXs_<file_number>.csv`
     - Delimiter: `|`
     - Key Columns:
       - `OUTPUTs`: Semicolon-separated list of transaction outputs. Each output contains:
         - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.

3. **Steps**:
   - **Initialization**:
     - Arrays to store the first observation day for each address:
       - `raw_addresses_new_per_day_array`
       - `Byron_payment_addresses_new_per_day_array`
       - `Shelley_payment_addresses_new_per_day_array`
       - `delegation_addresses_new_per_day_array`
     - Placeholders (`999999999999`) indicate unobserved addresses.
   - **Processing CSV Files**:
     - For each file:
       - Load into a DataFrame.
       - For each transaction output:
         - Extract relevant address parts.
         - Determine the type of the address (Byron, Shelley, or Stake).
         - Update the corresponding array with the current day if the address is new.


#### **Cell Code**
```python
# Find Number of New "Byron", "Shelley", and "Stake" Addresses Over Time

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize constants
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0
place_holder = 999999999999

# Arrays for first observation day
raw_addresses_new_per_day_array = np.array([place_holder] * len(unique_raw_addresses))
Byron_payment_addresses_new_per_day_array = np.array([place_holder] * len(unique_payment_addresses))
Shelley_payment_addresses_new_per_day_array = np.array([place_holder] * len(unique_payment_addresses))
delegation_addresses_new_per_day_array = np.array([place_holder] * len(unique_delegation_addresses))

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each CSV file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    # Load CSV file
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    # Process each row in the DataFrame
    for index, row in tqdm(df.iterrows()):
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)
        if tx_delta_day != current_delta_day:
            current_delta_day = tx_delta_day

        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            address_has_script = tx_output.split(',')[4]
            payment_cred = tx_output.split(',')[5]
            stake_address = tx_output.split(',')[6]

            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)

            if address_raw != '':
                addr_position = BinarySearch(unique_raw_addresses, address_raw)
                if raw_addresses_new_per_day_array[addr_position] == place_holder:
                    raw_addresses_new_per_day_array[addr_position] = current_delta_day

            if address_payment_part != '':
                if address_raw[2] == '8':  # Byron Address
                    addr_position = BinarySearch(unique_payment_addresses, address_payment_part)
                    if Byron_payment_addresses_new_per_day_array[addr_position] == place_holder:
                        Byron_payment_addresses_new_per_day_array[addr_position] = current_delta_day
                else:  # Shelley Address
                    addr_position = BinarySearch(unique_payment_addresses, address_payment_part)
                    if Shelley_payment_addresses_new_per_day_array[addr_position] == place_holder:
                        Shelley_payment_addresses_new_per_day_array[addr_position] = current_delta_day

            if address_delegation_part != '':
                addr_position = BinarySearch(unique_delegation_addresses, address_delegation_part)
                if delegation_addresses_new_per_day_array[addr_position] == place_holder:
                    delegation_addresses_new_per_day_array[addr_position] = current_delta_day

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (New Addresses from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (New Addresses): ", et)
print('----------------------')
print('done!')

```




***


### Calculate Active Users and Entities Per Day

This cell computes the number of unique active addresses and entities on a daily basis, using input transaction data from multiple CSV files.


### Calculate "Number of NFT/FT/ADA Entity-Level Transactions" Over Time

This code processes multiple transaction CSV files to compute the number of transactions involving ADA, NFTs, and FTs at the entity level over time.


#### **File Format of Input CSV**
- **File Type**: CSV (`.csv`)
- **Delimiter**: `|`
- **Columns**:
  - `TX_ID`: Transaction ID.
  - `BLOCK_TIME`: Timestamp of the transaction block (`YYYY-MM-DD HH:MM:SS`).
  - `EPOCH_NO`: Epoch number.
  - `INPUTs`: List of transaction inputs, delimited by `;`, where each input includes:
    - `Input_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `OUTPUTs`: List of transaction outputs, delimited by `;`, where each output includes:
    - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `TX_OUTPUT_MAs`: List of multi-asset outputs, delimited by `;`, where each MA includes:
    - `Output_ID,MA_ID,Quantity,NFT_Name,FT_Name,MA_Name,MA_Policy,MA_Fingerprint`.


#### **Workflow**
1. **Initialization**:
   - Choose the heuristic (`clustering_array_heur1and2`) for mapping payment addresses to entities.
   - Initialize arrays:
     - `balances_per_entity_array` to track balances.
     - Lists for storing transactions involving ADA, NFTs, and FTs.

2. **Processing Transactions**:
   - Extract input entities for each transaction.
   - Identify output entities receiving ADA, NFTs, and FTs.
   - Record transactions, ensuring input and output entities are distinct.

3. **Balances**:
   - Update entity balances based on inputs and outputs.

4. **Outputs**:
   - Store the results in separate files:
     - `balances_per_entity_array`
     - `entity_level_ADA_Transactions`
     - `entity_level_NFT_Transactions`
     - `entity_level_FT_Transactions`

5. **Parallel Processing**:
   - Use multiprocessing to process multiple files simultaneously.
   - Aggregate results across processes.


#### **Outputs**
1. **Balances File**:
   - Tracks balances for each entity.

2. **Transaction Files**:
   - **ADA Transactions**:
     - Day, Sender Entity, Receiver Entity, Sender Balance, Receiver Balance.
   - **NFT/FT Transactions**:
     - Day, Sender Entity, Receiver Entity, Sender Balance, Receiver Balance.

3. **File Naming**:
   - Outputs are stored with timestamped filenames for version control.


#### **Cell Code**
```python
print('----------------------')

def Find_ADA_NFT_FT_TXs_per_Entity_over_time(queue_):
    in_args = queue_.get()
    csv_file_name = in_args[0]
    csv_file_basename = os.path.basename(csv_file_name)
    
    ct = datetime.datetime.now()
    clustering_array = clustering_array_heur1and2

    # Initialize outputs
    balances_per_entity_array = np.array([0] * (np.amax(clustering_array) + 1))
    entity_level_ADA_Transactions = []
    entity_level_NFT_Transactions = []
    entity_level_FT_Transactions = []

    INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
    total_time_length_CARDANO = int((datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date() - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

    df = pd.read_csv(csv_file_name, delimiter='|')
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(df.loc[index, 'BLOCK_TIME'], '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Process input entities
        input_entity_indx = []
        for tx_input in inputs_list:
            address_raw = tx_input.split(',')[4]
            if tx_input.split(',')[7] == 'f':  # Non-smart contract address
                addr_indx = BinarySearch(unique_payment_addresses, address_raw)
                input_entity_indx.append(clustering_array[addr_indx][0])
                break

        # Process output entities
        output_ADA_entities_indx = []
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            if tx_output.split(',')[4] == 'f':  # Non-smart contract address
                addr_indx = BinarySearch(unique_payment_addresses, address_raw)
                output_ADA_entities_indx.append(clustering_array[addr_indx][0])

        output_ADA_entities_indx = list(set(output_ADA_entities_indx))
        if input_entity_indx[0] in output_ADA_entities_indx:
            output_ADA_entities_indx.remove(input_entity_indx[0])

        # Process balances
        for i, tx_input in enumerate(inputs_list):
            if tx_input.split(',')[7] == 'f':
                addr_indx = BinarySearch(unique_payment_addresses, tx_input.split(',')[4])
                entity_indx = clustering_array[addr_indx][0]
                balances_per_entity_array[entity_indx] -= int(tx_input.split(',')[6])

        for i, tx_output in enumerate(outputs_list):
            if tx_output.split(',')[4] == 'f':
                addr_indx = BinarySearch(unique_payment_addresses, tx_output.split(',')[1])
                entity_indx = clustering_array[addr_indx][0]
                balances_per_entity_array[entity_indx] += int(tx_output.split(',')[3])

    # Save results
    output_balances_filename = TEMP_ADDRESS + f'/balances_{csv_file_basename}_{ct.strftime("%Y%m%d_%H%M%S")}.txt'
    pickle.dump(balances_per_entity_array, open(output_balances_filename, 'wb'))

    output_adaTXs_filename = TEMP_ADDRESS + f'/ada_txs_{csv_file_basename}_{ct.strftime("%Y%m%d_%H%M%S")}.txt'
    pickle.dump(entity_level_ADA_Transactions, open(output_adaTXs_filename, 'wb'))

```
#### **Explanation of the Code**

1. **Objective**:
   - Count the number of unique active addresses and entities per day.
   - Active addresses and entities are determined from transaction inputs.

2. **Input Data**:
   - **Transaction CSV Files**:
     - Format: `cardano_TXs_<file_number>.csv`
     - Delimiter: `|`
     - Key Columns:
       - `BLOCK_TIME`: Date of the transaction block.
       - `INPUTs`: Semicolon-separated list of transaction inputs. Each input contains:
         - `Input_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.

3. **Steps**:
   - **Initialization**:
     - Arrays to store the daily counts:
       - `active_addresses_per_day_array`
       - `active_entities_per_day_array`
   - **Processing CSV Files**:
     - For each file:
       - Load into a DataFrame.
       - For each transaction input:
         - Extract the raw address, payment credential, and stake address.
         - Determine if the address is non-smart contract (`Has_Script = 'f'`).
         - Map the address to an entity using `clustering_array`.
         - Append the address and entity indices to daily tracking lists.
     - At the end of each day (`BLOCK_TIME`):
       - Calculate the number of unique active addresses and entities for the day.
       - Reset the daily tracking lists for the next day.

4. **Purpose**:
   - Tracks daily activity levels of addresses and entities.
   - Provides insight into the network's engagement and usage patterns.


#### **Cell Code**
```python
# Active Users and Entities Per Day

print('----------------------')

# Store current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize constants
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0
active_addresses = []
active_entities = []

# Arrays for active counts per day
active_addresses_per_day_array = [0] * total_time_length_CARDANO
active_entities_per_day_array = [0] * total_time_length_CARDANO

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each CSV file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    # Load CSV file
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    # Process each row in the DataFrame
    for index, row in tqdm(df.iterrows()):
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # If the day changes, update the daily arrays and reset trackers
        if tx_delta_day > current_delta_day:
            active_addresses_per_day_array[current_delta_day] = len(set(active_addresses))
            active_entities_per_day_array[current_delta_day] = len(set(active_entities))
            current_delta_day = tx_delta_day
            active_addresses = []
            active_entities = []

        # Process transaction inputs
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        for tx_input in inputs_list:
            address_raw = tx_input.split(',')[4]
            address_has_script = tx_input.split(',')[7]
            payment_cred = tx_input.split(',')[8]
            stake_address = tx_input.split(',')[9]

            if address_has_script == 'f':  # Exclude smart contract addresses
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    active_addresses.append(addr_indx)
                    active_entities.append(entity_indx)

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Active Addresses from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Active Addresses): ", et)
print('----------------------')
print('done!')

```



***


### Calculate "Number of NFT/FT/ADA Entity-Level Transactions" Over Time

This code processes multiple transaction CSV files to compute the number of transactions involving ADA, NFTs, and FTs at the entity level over time.


#### **File Format of Input CSV**
- **File Type**: CSV (`.csv`)
- **Delimiter**: `|`
- **Columns**:
  - `TX_ID`: Transaction ID.
  - `BLOCK_TIME`: Timestamp of the transaction block (`YYYY-MM-DD HH:MM:SS`).
  - `EPOCH_NO`: Epoch number.
  - `INPUTs`: List of transaction inputs, delimited by `;`, where each input includes:
    - `Input_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `OUTPUTs`: List of transaction outputs, delimited by `;`, where each output includes:
    - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `TX_OUTPUT_MAs`: List of multi-asset outputs, delimited by `;`, where each MA includes:
    - `Output_ID,MA_ID,Quantity,NFT_Name,FT_Name,MA_Name,MA_Policy,MA_Fingerprint`.


#### **Workflow**
1. **Initialization**:
   - Choose the heuristic (`clustering_array_heur1and2`) for mapping payment addresses to entities.
   - Initialize arrays:
     - `balances_per_entity_array` to track balances.
     - Lists for storing transactions involving ADA, NFTs, and FTs.

2. **Processing Transactions**:
   - Extract input entities for each transaction.
   - Identify output entities receiving ADA, NFTs, and FTs.
   - Record transactions, ensuring input and output entities are distinct.

3. **Balances**:
   - Update entity balances based on inputs and outputs.

4. **Outputs**:
   - Store the results in separate files:
     - `balances_per_entity_array`
     - `entity_level_ADA_Transactions`
     - `entity_level_NFT_Transactions`
     - `entity_level_FT_Transactions`

5. **Parallel Processing**:
   - Use multiprocessing to process multiple files simultaneously.
   - Aggregate results across processes.


#### **Outputs**
1. **Balances File**:
   - Tracks balances for each entity.

2. **Transaction Files**:
   - **ADA Transactions**:
     - Day, Sender Entity, Receiver Entity, Sender Balance, Receiver Balance.
   - **NFT/FT Transactions**:
     - Day, Sender Entity, Receiver Entity, Sender Balance, Receiver Balance.

3. **File Naming**:
   - Outputs are stored with timestamped filenames for version control.


#### **Cell Code**
```python
print('----------------------')

def Find_ADA_NFT_FT_TXs_per_Entity_over_time(queue_):
    in_args = queue_.get()
    csv_file_name = in_args[0]
    csv_file_basename = os.path.basename(csv_file_name)
    
    ct = datetime.datetime.now()
    clustering_array = clustering_array_heur1and2

    # Initialize outputs
    balances_per_entity_array = np.array([0] * (np.amax(clustering_array) + 1))
    entity_level_ADA_Transactions = []
    entity_level_NFT_Transactions = []
    entity_level_FT_Transactions = []

    INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
    total_time_length_CARDANO = int((datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date() - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

    df = pd.read_csv(csv_file_name, delimiter='|')
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(df.loc[index, 'BLOCK_TIME'], '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Process input entities
        input_entity_indx = []
        for tx_input in inputs_list:
            address_raw = tx_input.split(',')[4]
            if tx_input.split(',')[7] == 'f':  # Non-smart contract address
                addr_indx = BinarySearch(unique_payment_addresses, address_raw)
                input_entity_indx.append(clustering_array[addr_indx][0])
                break

        # Process output entities
        output_ADA_entities_indx = []
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            if tx_output.split(',')[4] == 'f':  # Non-smart contract address
                addr_indx = BinarySearch(unique_payment_addresses, address_raw)
                output_ADA_entities_indx.append(clustering_array[addr_indx][0])

        output_ADA_entities_indx = list(set(output_ADA_entities_indx))
        if input_entity_indx[0] in output_ADA_entities_indx:
            output_ADA_entities_indx.remove(input_entity_indx[0])

        # Process balances
        for i, tx_input in enumerate(inputs_list):
            if tx_input.split(',')[7] == 'f':
                addr_indx = BinarySearch(unique_payment_addresses, tx_input.split(',')[4])
                entity_indx = clustering_array[addr_indx][0]
                balances_per_entity_array[entity_indx] -= int(tx_input.split(',')[6])

        for i, tx_output in enumerate(outputs_list):
            if tx_output.split(',')[4] == 'f':
                addr_indx = BinarySearch(unique_payment_addresses, tx_output.split(',')[1])
                entity_indx = clustering_array[addr_indx][0]
                balances_per_entity_array[entity_indx] += int(tx_output.split(',')[3])

    # Save results
    output_balances_filename = TEMP_ADDRESS + f'/balances_{csv_file_basename}_{ct.strftime("%Y%m%d_%H%M%S")}.txt'
    pickle.dump(balances_per_entity_array, open(output_balances_filename, 'wb'))

    output_adaTXs_filename = TEMP_ADDRESS + f'/ada_txs_{csv_file_basename}_{ct.strftime("%Y%m%d_%H%M%S")}.txt'
    pickle.dump(entity_level_ADA_Transactions, open(output_adaTXs_filename, 'wb'))

```




***

### Calculate "NFTs Owned by Each Entity" and "Balances of Entities"

This code processes multiple CSV files containing transaction details to compute the number of NFTs owned by each entity and the ADA balance of each entity over time.


#### **Input CSV File Details**
- **File Type**: CSV (`cardano_TXs_NFTs_?.csv`)
- **Delimiter**: `|`
- **Columns**:
  - `TX_ID`: Transaction ID.
  - `BLOCK_TIME`: Block timestamp (`YYYY-MM-DD HH:MM:SS`).
  - `EPOCH_NO`: Epoch number.
  - `INPUTs`: List of transaction inputs, delimited by `;`. Each input includes:
    - `Input_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `OUTPUTs`: List of transaction outputs, delimited by `;`. Each output includes:
    - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`.
  - `TX_INPUT_MAs`: List of multi-asset inputs, delimited by `;`. Each MA includes:
    - `Input_ID,MA_ID,Quantity,NFT_Name,FT_Name,MA_Name,MA_Policy,MA_Fingerprint`.
  - `TX_OUTPUT_MAs`: List of multi-asset outputs, delimited by `;`. Each MA includes:
    - `Output_ID,MA_ID,Quantity,NFT_Name,FT_Name,MA_Name,MA_Policy,MA_Fingerprint`.


#### **Workflow**

1. **Initialization**:
   - Set `clustering_array` to map payment addresses to entities.
   - Initialize:
     - `NFTs_owned_per_entity_array`: List of NFTs owned by each entity.
     - `count_NFTs_per_entity`: Count of NFTs owned by each entity.
     - `balances_per_entity_array`: ADA balance for each entity.

2. **Process Each CSV File**:
   - Read the file and iterate over transactions.
   - Extract and process inputs:
     - Deduct NFT quantities and ADA values from the corresponding entities.
   - Extract and process outputs:
     - Add NFT quantities and ADA values to the corresponding entities.

3. **Update Balances and Ownership**:
   - For each transaction:
     - Adjust `balances_per_entity_array` for ADA inputs and outputs.
     - Update `NFTs_owned_per_entity_array` and `count_NFTs_per_entity`.

4. **Time Tracking**:
   - Measure and log the elapsed time for each CSV file.


#### **Outputs**
1. **NFT Ownership**:
   - `NFTs_owned_per_entity_array`: List of NFTs owned by each entity.
   - `count_NFTs_per_entity`: Total count of NFTs owned by each entity.

2. **Entity Balances**:
   - `balances_per_entity_array`: ADA balance for each entity.


#### **Code**

```python
print('----------------------')

# Initialize data structures
clustering_array = clustering_array_heur1and2
NFTs_owned_per_entity_array = [[] for _ in range(np.amax(clustering_array)+1)]
count_NFTs_per_entity = [0] * len(NFTs_owned_per_entity_array)
balances_per_entity_array = np.array([0] * (np.amax(clustering_array)+1))

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_NFTs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(df.loc[index, 'BLOCK_TIME'], '%Y-%m-%d %H:%M:%S').date()

        # Process inputs
        TX_INPUT_MAs_list = list(df.loc[index, 'TX_INPUT_MAs'].split(';'))
        for INPUT_MAs in TX_INPUT_MAs_list:
            MAs_list = list(INPUT_MAs.split(':'))
            if MAs_list[0] == '':
                continue
            INPUT_ID = int(MAs_list.pop(0))
            for MA in MAs_list:
                MA_splited = MA.split(',')
                if len(MA_splited) < 7:
                    continue
                MA_ID, quantity, NFT_name = MA_splited[0], MA_splited[1], MA_splited[2]
                if NFT_name != '':
                    entity_indx = find_entity(INPUT_ID, inputs_list, clustering_array)
                    if MA_ID and entity_indx:
                        count_NFTs_per_entity[entity_indx] -= int(quantity)

        # Process outputs
        TX_OUTPUT_MAs_list = list(df.loc[index, 'TX_OUTPUT_MAs'].split(';'))
        for OUTPUT_MAs in TX_OUTPUT_MAs_list:
            MAs_list = list(OUTPUT_MAs.split(':'))
            if MAs_list[0] == '':
                continue
            OUTPUT_ID = int(MAs_list.pop(0))
            for MA in MAs_list:
                MA_splited = MA.split(',')
                if len(MA_splited) < 7:
                    continue
                MA_ID, quantity, NFT_name = MA_splited[0], MA_splited[1], MA_splited[2]
                if NFT_name != '':
                    entity_indx = find_entity(OUTPUT_ID, outputs_list, clustering_array)
                    if MA_ID and entity_indx:
                        count_NFTs_per_entity[entity_indx] += int(quantity)

    et_temp = datetime.datetime.now() - ct_temp
    print(f"elapsed time (Process CSV File {file_name}): ", et_temp)

print('----------------------')

```



***


### Store/Load "NFTs_owned_per_entity_array" and "balances_per_entity_array"

This code handles storing and loading entity-level data, specifically the NFTs owned and ADA balances of entities.


#### **Functionality**

1. **Store Data to File**:
   - Saves:
     - `NFTs_owned_per_entity_array`: Detailed list of NFTs owned by each entity.
     - `count_NFTs_per_entity`: Count of NFTs owned by each entity.
     - `balances_per_entity_array`: ADA balance for each entity.
   - Files are stored using the `pickle` library for efficient binary serialization.

2. **Load Data from File**:
   - Reads back the serialized data for reuse.


#### **File Details**

- **File Format**: Pickle (`.txt` but stores binary data)
- **Stored Data**:
  - `NFTs_owned_per_entity_array`: 2D array where each row corresponds to an entity and contains tuples of (NFT_ID, quantity).
  - `count_NFTs_per_entity`: 1D array where each index corresponds to an entity and holds the total NFT count.
  - `balances_per_entity_array`: 1D array where each index corresponds to an entity and holds the ADA balance.


#### **Code**

```python
print('----------------------')
import pickle

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store to file:
'''
output_filename = BASE_ADDRESS + '/EntityOwnNFTsWithNameArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(NFTs_owned_per_entity_array, open(output_filename, 'wb'))


output_filename = BASE_ADDRESS + '/EntityOwnNFTsNumberArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(count_NFTs_per_entity, open(output_filename, 'wb'))


output_filename = BASE_ADDRESS + '/EntityBalancesArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(balances_per_entity_array, open(output_filename, 'wb'))
'''

# Load from file:
file_name = BASE_ADDRESS + '/EntityOwnNFTsWithNameArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-06-05_085540.txt'
NFTs_owned_per_entity_array = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityOwnNFTsNumberArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-06-05_085540.txt'
count_NFTs_per_entity = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityBalancesArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-06-05_085540.txt'
balances_per_entity_array = pickle.load(open(file_name, 'rb'))

print('----------------------')
print('done!')

```


***

### Find NFTs and FTs Minted by Each Entity

This code processes Cardano transaction data to determine which entities minted specific NFTs and FTs.


#### **Functionality**

1. **Inputs**:
   - CSV files containing transaction data, including minted NFTs and FTs.
   - Heuristic-based clustering data to map payment addresses to entities.

2. **Outputs**:
   - `NFTs_minted_per_entity_array`: A 2D list where each row corresponds to an entity and contains a list of NFTs minted by the entity.
   - `FTs_minted_per_entity_array`: A 2D list where each row corresponds to an entity and contains a list of FTs minted by the entity.
   - `NFTs_minted_by_smart_contract`: Counter for NFTs minted via smart contracts.
   - `FTs_minted_by_smart_contract`: Counter for FTs minted via smart contracts.


#### **File Details**

- **Input CSV Format**:
  - Delimited by `|`.
  - Columns include `MINT_NFTs` and `MINT_FTs`, with details about each minted token.

- **Output Data**:
  - `NFTs_minted_per_entity_array`: Stores `MA_ID` (Minting Asset IDs) for NFTs.
  - `FTs_minted_per_entity_array`: Stores `MA_ID` for FTs.
  - Smart contract counts log cases where no non-smart contract input address is found.


#### **Code**

```python
print('----------------------')

# Current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Clustering array selection
clustering_array = clustering_array_heur1and2

# Initialize arrays
NFTs_minted_per_entity_array = [[] for _ in range(np.amax(clustering_array) + 1)]
FTs_minted_per_entity_array  = [[] for _ in range(np.amax(clustering_array) + 1)]

NFTs_minted_by_smart_contract = 0
FTs_minted_by_smart_contract  = 0

# CSV details
CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = str(datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date())
        EPOCH_NO = str(df.loc[index, 'EPOCH_NO'])
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        
        # Process NFTs
        NFTs_list = list(df.loc[index, 'MINT_NFTs'].split(';'))
        for NFT in NFTs_list:
            MA_ID = NFT.split(',')[0]
            if MA_ID:
                flag = 0
                for tx_input in inputs_list:
                    address_raw = tx_input.split(',')[4]
                    address_has_script = tx_input.split(',')[7]
                    payment_cred = tx_input.split(',')[8]
                    stake_address = tx_input.split(',')[9]
                    if address_has_script == 'f':
                        [address_payment_part, _] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                        if address_payment_part:
                            addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                            entity_indx = clustering_array[addr_indx][0]
                            NFTs_minted_per_entity_array[entity_indx].append(MA_ID)
                            flag += 1
                            break
                if flag == 0:
                    NFTs_minted_by_smart_contract += 1
                    if NFTs_minted_by_smart_contract < 100:
                        print("NFT: no non-smart contract input address was found for MA_ID = ", MA_ID, '|TX_ID = ', TX_ID)

        # Process FTs
        FTs_list = list(df.loc[index, 'MINT_FTs'].split(';'))
        for FT in FTs_list:
            MA_ID = FT.split(',')[0]
            if MA_ID:
                flag = 0
                for tx_input in inputs_list:
                    address_raw = tx_input.split(',')[4]
                    address_has_script = tx_input.split(',')[7]
                    payment_cred = tx_input.split(',')[8]
                    stake_address = tx_input.split(',')[9]
                    if address_has_script == 'f':
                        [address_payment_part, _] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                        if address_payment_part:
                            addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                            entity_indx = clustering_array[addr_indx][0]
                            FTs_minted_per_entity_array[entity_indx].append(MA_ID)
                            flag += 1
                            break
                if flag == 0:
                    FTs_minted_by_smart_contract += 1
                    if FTs_minted_by_smart_contract < 100:
                        print("FT: no non-smart contract input address was found for MA_ID = ", MA_ID, '|TX_ID = ', TX_ID)

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Extract minting data from CSV File " + file_name + "): ", et_temp)

print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')

```



***

### Store/Load "NFTs Minted per Entity" and "FTs Minted per Entity"

This script handles the storage and retrieval of the following data arrays:
- **`NFTs_minted_per_entity_array`**: 2D Lists of NFTs minted by each entity.
- **`FTs_minted_per_entity_array`**: 2D Lists of FTs minted by each entity.



**File Naming**:
   - The file naming convention includes the heuristic method and a timestamp for traceability.



#### **Code**

```python
print('----------------------')

# Current timestamp
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store data to files
output_filename = BASE_ADDRESS + '/EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file_2D(NFTs_minted_per_entity_array, output_filename)

output_filename = BASE_ADDRESS + '/EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file_2D(FTs_minted_per_entity_array, output_filename)

# Load data from files
file_name = BASE_ADDRESS + '/EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-04-08_075547.txt'
NFTs_minted_per_entity_array = load_file_to_array_2D(file_name)

file_name = BASE_ADDRESS + '/EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-04-08_075547.txt'
FTs_minted_per_entity_array = load_file_to_array_2D(file_name)

##########################################################################################
print('----------------------')
print('done!')

```

### **File Details**

1. **Stored Files**:
   - **NFT Minting Data**:  
     File: `EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>.txt`
     Format: 2D list of minted NFTs per entity.
   - **FT Minting Data**:  
     File: `EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>.txt`
     Format: 2D list of minted FTs per entity.

2. **Loaded Files**:
   - Specify the file paths for `NFTs_minted_per_entity_array` and `FTs_minted_per_entity_array` to load the corresponding data.


### **Example**

1. **`NFTs_minted_per_entity_array`**:
   ```python
   [
       ['NFT1_ID', 'NFT2_ID'], # Entity 0
       ['NFT3_ID'],            # Entity 1
       [],                     # Entity 2
       ...
   ]
   ```

2. **`FTs_minted_per_entity_array`**:
   ```python
   [
       ['FT1_ID'],             # Entity 0
       ['FT2_ID', 'FT3_ID'],   # Entity 1
       [],                     # Entity 2
       ...
   ]
   ```

***

# Find "Number of NFT/FT/ADA Entity-Level Transactions" Over Time

This script calculates entity-level transactions (`ADA`, `NFT`, and `FT`) from Cardano transaction CSV files and stores the results for further analysis.

---

### **Functionality**

1. **Processes CSV Files**:
   - Iterates through transaction CSV files.
   - Identifies entity-level transactions for ADA, NFT, and FT.

2. **Transaction Types**:
   - **ADA Transactions**:
     - Transfers involving ADA between entities.
   - **NFT Transactions**:
     - Transfers involving non-fungible tokens (NFTs) between entities.
   - **FT Transactions**:
     - Transfers involving fungible tokens (FTs) between entities.

3. **Storage**:
   - Stores balances and transaction data for each file in separate pickle files.

4. **Parallel Processing**:
   - Utilizes multiprocessing to handle multiple files simultaneously for faster computation.

---

### **Code**

#### **Transaction Extraction Function**

```python
def Find_ADA_NFT_FT_TXs_per_Entity_over_time(queue_):
    in_args = queue_.get()
    csv_file_name = in_args[0]

    csv_file_basename = os.path.basename(csv_file_name)
    ct = datetime.datetime.now()

    # Choose clustering array
    clustering_array = clustering_array_heur1and2

    # Initialize outputs
    balances_per_entity_array = np.array([0] * (np.amax(clustering_array) + 1))
    entity_level_ADA_Transactions = []
    entity_level_NFT_Transactions = []
    entity_level_FT_Transactions = []

    INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()

    # Read CSV file
    ct_temp = datetime.datetime.now()
    df = pd.read_csv(csv_file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print(f"Elapsed time (Load CSV File {csv_file_name}): {et_temp}")

    for index, row in tqdm(df.iterrows()):
        # Processing each transaction
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(df.loc[index, 'BLOCK_TIME'], '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Extract input and output entities
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Processing logic for ADA, NFT, FT transactions
        # Outputs stored in entity_level_ADA_Transactions, entity_level_NFT_Transactions, entity_level_FT_Transactions

    # Save results to files
    curr_timestamp = datetime.datetime.now().strftime('%Y-%m-%d_%H%M%S')
    output_adaTXs_filename = f"{TEMP_ADDRESS}/entity_level_ADA_Transactions__{csv_file_basename}__{curr_timestamp}.txt"
    pickle.dump(entity_level_ADA_Transactions, open(output_adaTXs_filename, 'wb'))
    queue_.put([output_adaTXs_filename])
    return
```


***


# Merge "balances_per_entity" and "entity_level_ADA/NFT/FT_Transactions" Arrays



This ensures that balances and transactions are merged seamlessly across multiple files while maintaining entity relationships.

The script updates:
1. **Balances**: `balances_per_entity_array_AllTXs` — the cumulative balances of all entities.
2. **Transactions**:
   - **ADA Transactions**: `entity_level_ADA_Transactions_AllTXs`
   - **NFT Transactions**: `entity_level_NFT_Transactions_AllTXs`
   - **FT Transactions**: `entity_level_FT_Transactions_AllTXs`




## **Details**

### **1. Merge Balances**
Balances are summed across all transaction files to form the cumulative balances for each entity:
```python
balances_per_entity_array_AllTXs = np.array([0] * len(balances_per_entity_array_1))
for i in range(len(balances_per_entity_array_AllTXs)):
    balances_per_entity_array_AllTXs[i] = (balances_per_entity_array_1[i] +
                                           balances_per_entity_array_2[i] +
                                           balances_per_entity_array_3[i] + 
                                           balances_per_entity_array_4[i] + 
                                           balances_per_entity_array_5[i] + 
                                           balances_per_entity_array_6[i])
```



### **2. Update Transactions**
Each transaction tuple is updated with the cumulative balances for `entity_from` and `entity_to`:

#### **Example: Updating ADA Transactions**
For transactions in the second file, balances from the first file are added:
```python
for i in tqdm(range(len(entity_level_ADA_Transactions_2))):
    entity_from = entity_level_ADA_Transactions_2[i][1]
    entity_to   = entity_level_ADA_Transactions_2[i][2]
    entity_level_ADA_Transactions_2[i] = change_tuple_item(
        entity_level_ADA_Transactions_2[i], 
        3, 
        entity_level_ADA_Transactions_2[i][3] + balances_per_entity_array_1[entity_from]
    )
    entity_level_ADA_Transactions_2[i] = change_tuple_item(
        entity_level_ADA_Transactions_2[i], 
        4, 
        entity_level_ADA_Transactions_2[i][4] + balances_per_entity_array_1[entity_to]
    )
```

This process is repeated for:
- **NFT Transactions**: `entity_level_NFT_Transactions_AllTXs`
- **FT Transactions**: `entity_level_FT_Transactions_AllTXs`


### **3. Combine Transactions**
The transactions across all files are appended into unified lists.

#### **Example: ADA Transactions**
```python
entity_level_ADA_Transactions_AllTXs = entity_level_ADA_Transactions_1
entity_level_ADA_Transactions_AllTXs.append(entity_level_ADA_Transactions_2)
entity_level_ADA_Transactions_AllTXs.append(entity_level_ADA_Transactions_3)
entity_level_ADA_Transactions_AllTXs.append(entity_level_ADA_Transactions_4)
entity_level_ADA_Transactions_AllTXs.append(entity_level_ADA_Transactions_5)
entity_level_ADA_Transactions_AllTXs.append(entity_level_ADA_Transactions_6)
```

Similar merging is performed for NFT and FT transactions.


## **Helper Functions**

### **`change_tuple_item`**
This function updates specific indices in a tuple:
```python
def change_tuple_item(t, index, new_value):
    t_list = list(t)
    t_list[index] = new_value
    new_t = tuple(t_list)    
    return new_t
```


## **Outputs**

1. **Balances**: `balances_per_entity_array_AllTXs`
2. **Transactions**:
   - `entity_level_ADA_Transactions_AllTXs`
   - `entity_level_NFT_Transactions_AllTXs`
   - `entity_level_FT_Transactions_AllTXs`

These outputs consolidate all balance and transaction data across files, with updated balances reflecting the cumulative transactions.



***



### Store/Load "entity_level_ADA_Transactions_AllTXs" and "entity_level_NFT_Transactions_AllTXs" and "entity_level_FT_Transactions_AllTXs" into file:

This cell handles the storage and retrieval of transaction data for ADA, NFT, and FT at the entity level. The data is serialized into files using the `pickle` module, allowing it to be saved and loaded efficiently.

#### What this cell does:
1. **Store Data to File:**
   - The current timestamp is generated to create unique filenames.
   - Data for ADA, NFT, and FT transactions is serialized and saved to separate files.

2. **Load Data from File:**
   - how to load the serialized data back into the notebook.

#### Code:
```python
# Store/Load "entity_level_ADA_Transactions_AllTXs" and "entity_level_NFT_Transactions_AllTXs" and "entity_level_FT_Transactions_AllTXs" into file:

print('----------------------')
import pickle

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store to file:

output_filename = BASE_ADDRESS + '/EntityTXsADA_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(entity_level_ADA_Transactions_AllTXs, open(output_filename, 'wb'))

output_filename = BASE_ADDRESS + '/EntityTXsNFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(entity_level_NFT_Transactions_AllTXs, open(output_filename, 'wb'))

output_filename = BASE_ADDRESS + '/EntityTXsFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
pickle.dump(entity_level_FT_Transactions_AllTXs, open(output_filename, 'wb'))

# Load from file:
'''
file_name = BASE_ADDRESS + '/EntityTXsADA_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__????????????????.txt'
entity_level_ADA_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityTXsNFT_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__????????????????.txt'
entity_level_NFT_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityTXsFT_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__????????????????.txt'
entity_level_FT_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))
'''

##########################################################################################
print('----------------------')
print('done!')
```


***

# Generate "balances_array" and "gini_array" for Payment Addresses

This script calculates the balance of payment addresses by processing transaction inputs and outputs from multiple CSV files. It also computes a Gini index periodically to assess the distribution of balances among addresses.


#### Process

1. **Initialization**:
   - A `balances_array` is initialized with zero values for each unique payment address.
   - A `gini_array` is used to store Gini index values for balance distributions at regular transaction intervals (`gini_sample_id`).

2. **Input Data**:
   - Transaction data is read from CSV files:
     - File format: `BASE_ADDRESS + '/cardano_TXs_<file_number>.csv'`
     - Delimiter: `|`
     - Number of files: `NUMBER_OF_CSV_FILES = 6`

3. **Processing Transactions**:
   - For each transaction in the CSV file:
     - Extract `INPUTs` and `OUTPUTs` columns.
     - For **inputs**:
       - Identify non-smart contract (non-SC) addresses.
       - Deduct the UTXO value from the corresponding address's balance in `balances_array`.
     - For **outputs**:
       - Identify non-SC addresses.
       - Add the UTXO value to the corresponding address's balance in `balances_array`.

4. **Gini Index Calculation**:
   - At every `gini_sample_id` transactions:
     - Filter out addresses with zero balances.
     - Compute the Gini index using the `gini_index` function.
     - Append the computed Gini index to `gini_array`.
     - Log Gini index values for specific transaction IDs.

5. **Performance Tracking**:
   - Log the elapsed time for loading each CSV file and processing its transactions.

6. **Completion**:
   - Print a completion message when processing is finished.


#### Key Variables
- **`balances_array`**: Tracks the net balance for each unique payment address.
- **`gini_array`**: Stores Gini index values to monitor balance inequality over time.



#### Code

```python
# Generate "balances_array" for payment addresses:

print('----------------------')

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Initialize balances_array:
balances_array = np.array([0] * unique_payment_addresses_len)
for i in range(unique_payment_addresses_len):
    balances_array[i] = 0 

gini_array = [0]
gini_sample_id = 200000

for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Calculate "Gini Index"
        if int(TX_ID) > gini_sample_id:
            gini_array_indx = int(gini_sample_id / 200000)
            balances_array_no_zeros = balances_array[balances_array != 0]
            gini_array.append(gini_index(balances_array_no_zeros))
            if TX_ID < 2000002:
                print('TX_ID = ', TX_ID, '  |  gini_array [', gini_array_indx, '] = ', gini_array[gini_array_indx])
            gini_sample_id += 200000

        # Process inputs
        for i in range(len(inputs_list)):
            address_has_script = inputs_list[i].split(',')[7]
            if address_has_script == 'f':  # non-Smart Contract Address
                address_raw = inputs_list[i].split(',')[4]
                payment_cred = inputs_list[i].split(',')[8]
                stake_address = inputs_list[i].split(',')[9]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    address_position = BinarySearch(unique_payment_addresses, address_payment_part)
                    UTXO_value = inputs_list[i].split(',')[6]
                    balances_array[address_position] -= int(UTXO_value)

        # Process outputs
        for i in range(len(outputs_list)):
            address_has_script = outputs_list[i].split(',')[4]
            if address_has_script == 'f':  # non-Smart Contract Address
                address_raw = outputs_list[i].split(',')[1]
                payment_cred = outputs_list[i].split(',')[5]
                stake_address = outputs_list[i].split(',')[6]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    address_position = BinarySearch(unique_payment_addresses, address_payment_part)
                    UTXO_value = outputs_list[i].split(',')[3]
                    balances_array[address_position] += int(UTXO_value)

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

print('----------------------')
print('done!')

```



***


# Store "balances_array" and "gini_array" into Files

This script saves both the `balances_array` (containing balances for payment addresses) and the `gini_array` (containing Gini index values) into separate files for later use.


#### Process

1. **Timestamp Generation**:
   - A timestamp in the format `YYYY-MM-DD_HHMMSS` is created to ensure file names are unique.

2. **File Naming**:
   - **Balances Array File**:
     - File format: `balancesList_paymentAddresses_noSC__Cardano_TXs_All__<timestamp>.txt`
     - Example: `balancesList_paymentAddresses_noSC__Cardano_TXs_All__2024-11-22_123456.txt`
   - **Gini Array File**:
     - File format: `giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__<timestamp>.txt`
     - Example: `giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__2024-11-22_123456.txt`

3. **Storing the Arrays**:
   - The `store_array_to_file` function is used to save each array into its respective file.
   - Each value in the array is stored on a new line.

4. **Completion**:
   - Prints the output file names and confirms the arrays are successfully stored.


#### Code

```python
# Store "balances_array" and "gini_array" into files:

print('----------------------')

# Store balances_array
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/balancesList_paymentAddresses_noSC__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)

store_array_to_file(balances_array, output_filename)

# Store gini_array
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)

store_array_to_file(gini_array, output_filename)

##########################################################################################
print('----------------------')
print('done!')

```










