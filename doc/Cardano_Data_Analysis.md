# Cardano_Data_Analysis


In the following sections, each cell in the corresponding 'code' (with a similar filename in the code directory of this repository) is documented with a brief description of its purpose and functionality, followed by the summarized cell code.


***

### Import Libraries and Set Constant Variables

This cell imports a comprehensive set of libraries and modules required for data analysis, visualization, multiprocessing, and Spark-based operations. It also initializes certain environment variables, constants, and performs some basic computations.

#### Explanation of the Code:
**Library Imports**:
   - **Core libraries**: `numpy`, `array`, `csv`, `datetime`, and `os`.
   - **Search and sorting**: `bisect`.
   - **Visualization**: `matplotlib.pyplot`.
   - **Parallel processing**: `multiprocessing` and `threading`.
   - **Big data processing**: `pyspark` and `pandas`.
   - **Graph processing**: `networkx` and `community`.
   - **Probability distributions**: `powerlaw`.
   - **Progress tracking**: `tqdm`.
   - **Serialization**: `pickle`.


#### Cell Code:
```python
import numpy as np
from array import *
import csv
import datetime;
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
from community import modularity
import pickle
import powerlaw

print('----------------------')
print('done!')
```


***

### Define Base and Temporary Directory Paths

This cell sets up the base and temporary directory paths to organize and manage file storage for the Cardano project.

**`BASE_ADDRESS`**:
   - Defines the base directory for storing exported data related to the project.

**`TEMP_ADDRESS`**:
   - Defines a subdirectory within the base directory to store temporary files.


#### Cell Code:
```python
BASE_ADDRESS = '/local/scratch/exported/Cardano_MCH_2023_1/'
TEMP_ADDRESS = BASE_ADDRESS + '/temp_files/'
```

***



### Define Required Methods

This cell contains a set of essential utility functions and algorithms to support various computational tasks, such as data manipulation, searching, clustering, graph processing, and file I/O operations. Each function is explained below.

#### Explanation of the Code:
**UnionFind Operations**:
   - `parent`: Retrieves the parent of a node in a Union-Find structure.
   - `find_parent`: Finds the root parent of a node using path compression.
   - `link_address`: Implements the Union-Find algorithm to link two nodes.
   - `resolveAll`: Resolves all parent-child relationships to find ultimate roots.
   - `remapClusterIds`: Reassigns cluster IDs to ensure a contiguous sequence starting from zero.
   - `merge_parents`: Combines two parent arrays into a unified structure.

**Binary Search**:
   - `BinarySearch`: Performs a binary search for a specific element.
   - `BinarySearch_Find_start_end`: Finds the start and end indices of a target element in a sorted array.

**File Operations**:
   - `store_array_to_file`, `store_array_to_file_2D`: Save arrays (1D or 2D) to files using CSV or JSON.
   - `load_file_to_array`, `load_file_to_array_2D`: Load arrays from CSV or JSON files.
   - `store_dict_to_file_INT`: Saves a dictionary to a file with integer keys and values.
   - `load_file_to_dict_INT`: Loads a dictionary from a file with integer keys and values.

**Graph Processing**:
   - `add_edge_info`: Adds weighted edges between nodes in a graph structure.

**Cardano Address Extraction**:
   - `extract_payment_delegation_parts`: Splits raw addresses into payment and delegation parts based on Cardano's address types (Byron or Shelley).

**Statistical Calculations**:
   - `gini_index`: Computes the Gini index for measuring inequality in wealth distribution.

#### Cell Code:
```python
##########################################################################################
def parent (id1, parents_array):
    return parents_array[id1]

def find_parent (id1, parents_array):
    while (id1 != parent(id1, parents_array)):
        new_parent = parent(parent(id1, parents_array), parents_array)
        id1 = new_parent
    return id1

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

def resolveAll (parents_array):
    for id1 in tqdm(range(len(parents_array))):
        parents_array[id1] = find_parent(id1, parents_array)
    return

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

def load_file_to_array (file_name, header_=None):
    ct = datetime.datetime.now()
    print('start time (Load ' + file_name  + ' to Array): ', ct)
    df = pd.read_csv(file_name, header=header_)
    output_array_name = df.to_numpy()
    et = datetime.datetime.now() - ct
    print('elapsed time (Load ' + file_name  + ' to Array): ', et)
    return output_array_name

def store_array_to_file_2D (input_array_name, file_name):
    ct = datetime.datetime.now()
    print('start time (Store Array 2D to ' + file_name + '): ', ct)
    with open(file_name, "w") as filehandle:
        json.dump(input_array_name, filehandle)
    et = datetime.datetime.now() - ct
    print('elapsed time (Store Array 2D to ' + file_name + '): ', et)
    return

def load_file_to_array_2D (file_name):
    ct = datetime.datetime.now()
    print('start time (Load ' + file_name  + ' to Array 2D): ', ct)
    with open(file_name) as filehandle:
        output_array_name = json.load(filehandle)
    et = datetime.datetime.now() - ct
    print('elapsed time (Load ' + file_name  + ' to Array 2D): ', et)
    return output_array_name

def store_dict_to_file_INT (input_dict_name, file_name):
    ct = datetime.datetime.now()
    print('start time (Store Dictionary to ' + file_name + '): ', ct)
    filehandle = csv.writer(open(file_name, 'w'))
    for key, val in input_dict_name.items():
        filehandle.writerow([key, val])
    et = datetime.datetime.now() - ct
    print('elapsed time (Store Dictionary to ' + file_name + '): ', et)
    return

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
# Read Sorted and Unique Address Lists from Files

This cell reads three sorted and unique address lists (`raw_address_list`, `payment_address_list`, and `delegation_address_list`) from files and stores them in memory.


**File Names and Formats**:
   - The input files contain sorted and unique address lists. The files have the following structure:
     - **Columns**:
       - Single column of strings, where each string represents an address.

**Loading Address Lists**:
   - The `load_file_to_array` function is used to read each file into a NumPy array.

**Address Lists**:
   - **Raw Addresses (`unique_raw_addresses`)**:
     - Contains unique raw addresses from Cardano transactions.
   - **Payment Addresses (`unique_payment_addresses`)**:
     - Contains unique payment addresses.
   - **Delegation Addresses (`unique_delegation_addresses`)**:
     - Contains unique delegation addresses.


#### Cell Code:
```python
# Read ("sorted" "unique" array_list) [raw_address_list/payment_address_list/delegation_address_list] from file:

file_name = BASE_ADDRESS + '/Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357.txt'
unique_raw_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)

file_name = BASE_ADDRESS + '/Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953.txt'
unique_payment_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)

file_name = BASE_ADDRESS + '/Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415.txt'
unique_delegation_addresses = load_file_to_array(file_name)  # Text file, single column of strings (addresses)

##########################################################################################
INITIAL_DATE_CARDANO      = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO        = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds()/86400) + 1

unique_raw_addresses_len        = len(unique_raw_addresses)
unique_payment_addresses_len    = len(unique_payment_addresses)
unique_delegation_addresses_len = len(unique_delegation_addresses)


```



***


# Read Address Clustering Results from Files

This cell reads the results of heuristic-based address clustering from files. The clustering results are based on three scenarios: heuristic 1, heuristic 2, and a combination of both heuristics.

**File Formats**:
   - The input files are **CSV files** containing clustering data. The files are structured as follows:
     - **Columns**:
       - Single column with integer values representing the cluster IDs for each address.

**Loading Clustering Results**:
   - The `load_file_to_array` function is used to read each file into a NumPy array.

**Clustering Results**:
   - **Heuristic 1 (`clustering_array_heur1`)**:
     - Results from clustering using heuristic 1 (excluding smart contracts).
   - **Heuristic 2 (`clustering_array_heur2`)**:
     - Results from clustering using heuristic 2.
   - **Heuristic 1 and 2 Combined (`clustering_array_heur1and2`)**:
     - Results from clustering using both heuristics.

#### Cell Code:
```python
# Read Address Clustering Results from Files

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223957'
clustering_array_heur1 = load_file_to_array(file_name)  # single column of integers (cluster IDs)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic2__Cardano_TXs_All__2023-03-26_110150'
clustering_array_heur2 = load_file_to_array(file_name)  # single column of integers (cluster IDs)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212'
clustering_array_heur1and2 = load_file_to_array(file_name)  # single column of integers (cluster IDs)

```




***
# Find the Number of Members in Each Cluster (Heuristic 1)

This cell calculates the number of members in each cluster based on the clustering results obtained using Heuristic 1.

#### Explanation of the Code:
**Input**:
   - `clustering_array_heur1`: A **NumPy array** containing the cluster IDs for each address.

**Steps**:
   - **Sort the Array**:
     - The `clustering_array_heur1` is sorted to enable efficient cluster member counting.
   - **Determine Number of Clusters**:
     - The total number of clusters is determined as the maximum cluster ID plus one.
   - **Count Members in Each Cluster**:
     - For each cluster ID, the `BinarySearch_Find_start_end` function is used to find the start and end indices of the cluster in the sorted array. The difference between these indices gives the number of members in the cluster.

#### Cell Code:
```python
# Find number of members in each cluster (Heur1):


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

# Verify by summing the members of all clusters
print('number of unique addresses (to verify num_of_cluster_members_heur1) = ', sum(num_of_cluster_members_heur1))

```



***


# Power-Law Distributions Analysis (Heuristic 1)

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

# Find "Entity" of Each "Stake Address" (Delegation Address) [Heuristic 2]

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

#### Cell Code:
```python
# Find "Entity" of Each "Stake Address" (Delegation Address) [Heuristic 2]:

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
    # Load the CSV file
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

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

```




***

### Load/Store `entity_of_stake_addresses` from/into File

This cell handles storing and loading the `entity_of_stake_addresses` array, which maps stake addresses to their respective entities, using the file system for persistence.

#### **Explanation of the Code**

1. **File Format**:
   - **File Type**: Plain text
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


#### **Cell Code**
```python
# Load/Store "entity_of_stake_addresses" from/into file:

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store "entity_of_stake_addresses" into file:
output_filename = BASE_ADDRESS + '/Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
store_array_to_file(entity_of_stake_addresses, output_filename)

# Load "entity_of_stake_addresses" from file:
file_name = BASE_ADDRESS + '/Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__2024-01-23_212107.txt'
entity_of_stake_addresses = load_file_to_array(file_name)

```



***


# Calculate Entity Balances [Heuristic 2]

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
   - **Daily Balance Snapshots**:
     - If a new day is encountered in `BLOCK_TIME`, the current balances are serialized into a pickle file for persistence.

3. **File Output**:
   - **Daily Snapshot Files**:
     - File format: Pickle (`.pickle`).
     - Filename: `BalancesPerEntityDay_Heur2_<Day>__Cardano_TXs_All.pickle`
     - Content:
       - `balances_per_entity_array`: Array containing balances for all entities as of the given day.

4. **Purpose**:
   - Provides a time-series view of entity balances based on daily transaction data.
   - Enables incremental analysis without reprocessing all data.


#### **Cell Code**
```python
# Entity Balances [Heur 2]:

import random
import pickle

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
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

    # Iterate through transactions
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Store daily balance snapshot
        if current_delta_day < tx_delta_day:
            output_filename = BASE_ADDRESS + '/Cardano_Balances_Entities_Heur2__PICKLE/YuZhang__BalancesPerEntityDay_Heur2_' + str(current_delta_day).zfill(4) + '__Cardano_TXs_All.pickle'
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

```


#### **Example Output**
- **Daily Snapshot File**:
  - File: `BalancesPerEntityDay_Heur2_0001__Cardano_TXs_All.pickle`
  - Content: Serialized `balances_per_entity_array` for day 1.



***

# Calculate Number of Active Delegators/Rewarders (Entities) Per Epoch [Heuristic 2]

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

```


***
# Calculate "Active Delegators (Stake Addresses)" Per Epoch

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
   - **Processing Rows**:
     - For each row in the CSV file:
       - Extract delegator details (`DELEGATORs`).
       - Use `entity_of_stake_addresses` to map stake addresses to entity indices.
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
     - File format: Plain Text
     - Filename examples:
       - `StakeDelegPerEntityEpoch_<Epoch>__Cardano_TXs_All`
       - `RewardPerEntityEpoch_<Epoch>__Cardano_TXs_All`
     - Content:
       - Array of staking/reward amounts for each entity.


#### **Cell Code**
```python
# Calculate "Active Delegators (Entities)" Per Epoch

# Initialize epoch ranges and dates
first_epoch_no = 210
last_epoch_no  = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date() # epoch_no = 210
LAST_DATE_POOLS_STAKING  = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date() # epoch_no = 391



#total_num_of_epochs = int(int((LAST_DATE_POOLS_STAKING - FIRST_DATE_POOLS_STAKING).total_seconds()/86400)/5) + 1
total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)

epochs_array      = list(range(first_epoch_no, last_epoch_no+1))
epochs_date_array = [0]*len(epochs_array)
for i in range(len(epochs_date_array)):
    epochs_date_array[i] = FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i*5))


num_delegator_addresses_per_epoch  = [0 for _ in range(total_num_of_epochs)]
num_delegator_entities_per_epoch   = [0 for _ in range(total_num_of_epochs)]
num_rewarder_addresses_per_epoch   = [0 for _ in range(total_num_of_epochs)]
num_rewarder_entities_per_epoch    = [0 for _ in range(total_num_of_epochs)]



current_epoch = first_epoch_no


file_name = BASE_ADDRESS + '/cardano_pools_4.csv'
df = pd.read_csv(file_name, delimiter='|')


# entity_size_array_heur1and2
# entity_of_stake_addresses
delegator_addresses_per_epoch_SET = set()
delegator_entities_per_epoch_SET  = set()
reward_addresses_per_epoch_SET    = set()
reward_entities_per_epoch_SET     = set()


clustering_array = clustering_array_heur1and2


stake_deleg_by_entities = [0] * ( np.amax(clustering_array)+1 )
reward_by_entities      = [0] * ( np.amax(clustering_array)+1 )


stake_deleg_by_addresses = [0] * len(unique_delegation_addresses)
reward_by_addresses      = [0] * len(unique_delegation_addresses)



for index, row in tqdm(df.iterrows()):
    ##########################################################################################
    EPOCH             = int(  df.loc[index ,  'EPOCH']             )
    POOL_ID           = int(  df.loc[index ,  'POOL_ID']           )
    POOL_HASH_BECH32  =       df.loc[index ,  'POOL_HASH_BECH32']
    POOL_STAKES       = int(  df.loc[index ,  'POOL_STAKES']       )
    POOL_REWARDS      = int(  df.loc[index ,  'POOL_REWARDS']      )
    NUM_OF_DELEGATORS = int(  df.loc[index ,  'NUM_OF_DELEGATORS'] )
    NUM_OF_REWARDERS  = int(  df.loc[index ,  'NUM_OF_REWARDERS']  )
    DELEGATORs        = list( df.loc[index , 'DELEGATORs'].split(';') )
    if(not pd.isna(df.loc[index, 'REWARDERs'])):
        REWARDERs     = list( df.loc[index , 'REWARDERs'].split(';') )
    else:
        REWARDERs     = list()

    ##########################################################################################
    if(EPOCH < first_epoch_no):
        continue;

    ##########################################################################################
    if(EPOCH > current_epoch):
        num_delegator_addresses_per_epoch [current_epoch - first_epoch_no] = len(delegator_addresses_per_epoch_SET)
        num_delegator_entities_per_epoch  [current_epoch - first_epoch_no] = len(delegator_entities_per_epoch_SET)

        num_rewarder_addresses_per_epoch  [current_epoch - first_epoch_no] = len(reward_addresses_per_epoch_SET)
        num_rewarder_entities_per_epoch   [current_epoch - first_epoch_no] = len(reward_entities_per_epoch_SET)

        delegator_addresses_per_epoch_SET.clear()
        delegator_entities_per_epoch_SET.clear()

        reward_addresses_per_epoch_SET.clear()
        reward_entities_per_epoch_SET.clear()
        
        output_filename = BASE_ADDRESS + '/YuZhang_Cardano_StakeDelegation_Entities/StakeDelegPerEntityEpoch_' + str(current_epoch).zfill(4) + '__Cardano_TXs_All.txt'
        store_array_to_file(stake_deleg_by_entities, output_filename)
        stake_deleg_by_entities = [0] * ( np.amax(clustering_array)+1 )
        
        output_filename = BASE_ADDRESS + '/YuZhang_Cardano_Reward_Entities/RewardPerEntityEpoch_' + str(current_epoch).zfill(4) + '__Cardano_TXs_All.txt'
        store_array_to_file(reward_by_entities, output_filename)
        reward_by_entities      = [0] * ( np.amax(clustering_array)+1 )

        current_epoch      = EPOCH

    ##########################################################################################
    if(EPOCH > last_epoch_no):
        break;

    ##########################################################################################
    for delegator in DELEGATORs:
        #temp_str          = delegator.split(',')
        deleg_addr_id     =      delegator.split(',')[0]
        deleg_amount      = int( delegator.split(',')[1] )
        deleg_stake_addr  =      delegator.split(',')[2]
        
        deleg_addr_indx   = BinarySearch(unique_delegation_addresses, deleg_stake_addr, debug=False)
        if(deleg_addr_indx != -1):
            deleg_entity_indx = entity_of_stake_addresses[deleg_addr_indx][0]        
    
            if (deleg_entity_indx < len(stake_deleg_by_entities)):
                stake_deleg_by_entities[deleg_entity_indx] = stake_deleg_by_entities[deleg_entity_indx] + deleg_amount
    
            delegator_addresses_per_epoch_SET.add(deleg_addr_indx)
            delegator_entities_per_epoch_SET.add(deleg_entity_indx)
        else:
            delegator_addresses_per_epoch_SET.add(deleg_stake_addr)
            delegator_entities_per_epoch_SET.add(deleg_stake_addr)  # consider it as a separate entity

    ##########################################################################################
    for rewarder in REWARDERs:
        #temp_str          = rewarder.split(',')
        reward_addr_id    =      rewarder.split(',')[0]
        reward_amount     = int( rewarder.split(',')[1] )
        reward_stake_addr =      rewarder.split(',')[2]
     
        reward_addr_indx   = BinarySearch(unique_delegation_addresses, reward_stake_addr, debug=False)
        if(reward_addr_indx != -1):
            reward_entity_indx = entity_of_stake_addresses[reward_addr_indx][0]
    
            if (reward_entity_indx < len(reward_by_entities)):
                reward_by_entities[reward_entity_indx] = reward_by_entities[reward_entity_indx] + reward_amount
    
            reward_addresses_per_epoch_SET.add(reward_addr_indx)
            reward_entities_per_epoch_SET.add(reward_entity_indx)
        else:
            reward_addresses_per_epoch_SET.add(reward_stake_addr)
            reward_entities_per_epoch_SET.add(reward_stake_addr) # consider it as a separate entity

```



***

# Load/Store Num of Address/Entity Delegators/Rewarders Per Epoch:

This cell handles the loading and storing of the following metrics per epoch:

1. **`num_delegator_addresses_per_epoch`**: Number of delegator addresses.
2. **`num_delegator_entities_per_epoch`**: Number of delegator entities.
3. **`num_rewarder_addresses_per_epoch`**: Number of rewarder addresses.
4. **`num_rewarder_entities_per_epoch`**: Number of rewarder entities.


#### **Explanation of the Code**

1. **File Format**:
   - **File Type**: Plain Text 
   - **Delimiter**: One value per line.
   - **Content**: Array values representing the respective metric for each epoch.

2. **Storing Metrics**:
   - Each metric array is stored in a separate text file.
   - Filenames include a timestamp for versioning:
     - Example: `Num_Delegator_addresses_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS`
   - The `store_array_to_file` function writes the arrays to disk.

3. **Loading Metrics**:
   - Metrics are loaded from corresponding text files.
   - The `load_file_to_array` function is used to read the arrays into memory.




#### **Cell Code**
```python
# Load/Store Num of Address/Entity Delegators/Rewarders Per Epoch:


# Load/Store "num_delegator_addresses_per_epoch",
#            "num_delegator_entities_per_epoch",
#            "num_rewarder_addresses_per_epoch",
#            "num_rewarder_entities_per_epoch" from/into file:

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store metrics into files:
output_filename = BASE_ADDRESS + '/Num_Delegator_addresses_per_epoch__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file(num_delegator_addresses_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Delegator_entities_per_epoch__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file(num_delegator_entities_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file(num_rewarder_addresses_per_epoch, output_filename)

output_filename = BASE_ADDRESS + '/Num_Rewarder_entities_per_epoch__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file(num_rewarder_entities_per_epoch, output_filename)


# Load metrics from files:
file_name = BASE_ADDRESS + '/Num_Delegator_addresses_per_epoch__Cardano_TXs_All__2023-12-17_094541'
num_delegator_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Delegator_entities_per_epoch__Cardano_TXs_All__2023-12-17_094541'
num_delegator_entities_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__2023-12-17_132620'
num_rewarder_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_entities_per_epoch__Cardano_TXs_All__2023-12-17_132620'
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


#### **Cell Code**
```python
# Find Number of New "Entities" Over Time


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
```



***


### Load/Store `entities_new_per_day_array` From/Into File

This cell handles the persistence of the `entities_new_per_day_array` data, which tracks the day each entity is first observed. The data can be stored into or loaded from a text file.


#### **File Format**

- **File Type**: Plain Text
- **Delimiter**: One value per line
- **Content**:
  - Array values representing the day each entity was first observed. The index corresponds to the entity index, and the value represents the day since the start of the Cardano network (`INITIAL_DATE_CARDANO`).


#### **Steps**

1. **Storing Data**:
   - Write the `entities_new_per_day_array` to a text file.
   - File is named with a timestamp for version control:
     - Example: `newPerDay_Entities__Cardano_TXs_All__YYYY-MM-DD_HHMMSS`.

2. **Loading Data**:
   - Use the `load_file_to_array` function to restore the array into memory.



#### **Cell Code**
```python
# Load/Store "entities_new_per_day_array" from/into file

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store "entities_new_per_day_array" into file:
output_filename = BASE_ADDRESS + '/newPerDay_Entities__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(entities_new_per_day_array, output_filename)

# Load "entities_new_per_day_array" from file:
file_name = BASE_ADDRESS + '/newPerDay_Entities__Cardano_TXs_All__2023-04-22_012525'
entities_new_per_day_array = load_file_to_array(file_name)

```



***


## Find Number of New "Byron", "Shelley", and "Stake" Addresses vs Time

This cell calculates the number of new Cardano addresses (Byron, Shelley, and Stake delegation addresses) created per day over the entire operational period of Cardano. It processes transaction data from CSV files and determines the day each unique address first appears in the blockchain data.

1. **Datetime Initialization**:
   - Calculates the total operational days of Cardano between its initial launch date (September 23, 2017) and the final date considered (January 21, 2023).
2. **Address Initialization**:
   - Uses placeholder values (`999999999999`) to initialize arrays for tracking the first appearance day of:
     - **Raw Addresses**: All unique raw addresses.
     - **Byron Payment Addresses**: Addresses starting with the prefix `'8'`.
     - **Shelley Payment Addresses**: Shelley-specific addresses.
     - **Delegation Addresses**: Stake delegation addresses.
3. **File Processing**:
   - Reads transaction data from CSV files containing outputs.
   - Extracts output details to determine if each unique address appears for the first time.
   - Updates the respective arrays with the day offset of the first occurrence.
4. **Address Type Classification**:
   - **Byron Addresses**: Identified by the prefix `'8'`.
   - **Shelley Addresses**: Non-Byron payment addresses.
   - **Delegation Addresses**: Derived from the delegation parts of the Shelley addresses.
5. **Output Data**:
   - Each array (`raw_addresses_new_per_day_array`, `Byron_payment_addresses_new_per_day_array`, etc.) contains the first appearance day of addresses relative to the Cardano start date.

### CSV File Details
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID` (Transaction ID)
  - `BLOCK_TIME` (Timestamp in `%Y-%m-%d %H:%M:%S` format)
  - `EPOCH_NO` (Epoch number)
  - `OUTPUTs` (Semicolon-separated list containing address details)

### Output Data Details
- **Format**: Arrays
- **Number of Columns**: 1 per array
- **Array Descriptions**:
  - `raw_addresses_new_per_day_array`: Tracks the first appearance of raw addresses.
  - `Byron_payment_addresses_new_per_day_array`: Tracks Byron payment addresses.
  - `Shelley_payment_addresses_new_per_day_array`: Tracks Shelley payment addresses.
  - `delegation_addresses_new_per_day_array`: Tracks delegation addresses.
- **Values**:
  - Day offsets (integer) representing the day the address first appeared.
  - Placeholder values for addresses that do not appear in the data.

### Code
```python
# Find Number of new "Byron", "Shelley", and "Stake" addresses VS "Time":

# Current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Initialize date range
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0

# Initialize arrays for new addresses
place_holder = 999999999999
raw_addresses_new_per_day_array = np.array([place_holder] * len(unique_raw_addresses))
Byron_payment_addresses_new_per_day_array = np.array([place_holder] * len(unique_payment_addresses))
Shelley_payment_addresses_new_per_day_array = np.array([place_holder] * len(unique_payment_addresses))
delegation_addresses_new_per_day_array = np.array([place_holder] * len(unique_delegation_addresses))

# File details
CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
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

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)
        if tx_delta_day != current_delta_day:
            current_delta_day = tx_delta_day

        EPOCH_NO = str(df.loc[index, 'EPOCH_NO'])
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
```

***

# Load/Store Address Data per Day:

This cell provides functionality to save and load arrays tracking the first appearance day of various types of Cardano addresses (Byron, Shelley, Stake delegation, and raw addresses) to and from files.

#### Store Functionality
1. **Timestamped Filenames**:
   - Files are named using the current timestamp for versioning and identification.
   - Arrays saved:
     - `raw_addresses_new_per_day_array`
     - `Byron_payment_addresses_new_per_day_array`
     - `Shelley_payment_addresses_new_per_day_array`
     - `delegation_addresses_new_per_day_array`
2. **File Format**:
   - Data is stored as Plain Text files using a custom array storage function.

#### Load Functionality
1. **File Names**:
   - Arrays are loaded back into memory from specific `.txt` files.
   - The filenames must match the saved timestamped names.
2. **File Format**:
   - Data is loaded using the custom function, which retrieves the array values.

### File Details
- **Filename Format for Saving**:
  - `newPerDay_{AddressType}__Cardano_TXs_All__{Timestamp}.txt`
  - Example: `newPerDay_rawAddresses__Cardano_TXs_All__2023-04-20_025121.txt`
- **Data Format**: CSV-like `.txt` files
- **Columns**: 
  - Single column containing integers representing the first appearance day offset for each address.
  - Placeholder values (`999999999999`) indicate that the address did not appear.

### Code
```python
# Load/Store Address Data per Day:


# Load/Store "raw_addresses_new_per_day_array" from/into file:
#            "Byron_payment_addresses_new_per_day_array",
#            "Shelley_payment_addresses_new_per_day_array",
#            "delegation_addresses_new_per_day_array"

print('----------------------')

# Store arrays into files
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

output_filename = BASE_ADDRESS + '/newPerDay_rawAddresses__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(raw_addresses_new_per_day_array, output_filename)
output_filename = BASE_ADDRESS + '/newPerDay_ByronAddresses__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(Byron_payment_addresses_new_per_day_array, output_filename)
output_filename = BASE_ADDRESS + '/newPerDay_ShelleyAddresses__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(Shelley_payment_addresses_new_per_day_array, output_filename)
output_filename = BASE_ADDRESS + '/newPerDay_delegationAddresses__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(delegation_addresses_new_per_day_array, output_filename)

# Load arrays from files
file_name = BASE_ADDRESS + '/newPerDay_rawAddresses__Cardano_TXs_All__2023-04-20_025121'
raw_addresses_new_per_day_array = load_file_to_array(file_name)
file_name = BASE_ADDRESS + '/newPerDay_ByronAddresses__Cardano_TXs_All__2023-04-20_025121'
Byron_payment_addresses_new_per_day_array = load_file_to_array(file_name)
file_name = BASE_ADDRESS + '/newPerDay_ShelleyAddresses__Cardano_TXs_All__2023-04-20_025121'
Shelley_payment_addresses_new_per_day_array = load_file_to_array(file_name)
file_name = BASE_ADDRESS + '/newPerDay_delegationAddresses__Cardano_TXs_All__2023-04-20_025121'
delegation_addresses_new_per_day_array = load_file_to_array(file_name)

```


***


# Find Number of "New Addresses" Per Day

This cell calculates the daily count of new Cardano addresses, categorized as raw addresses, Byron payment addresses, Shelley payment addresses, and stake delegation addresses. The process involves loading precomputed arrays of address first-appearance days, sorting these arrays, and aggregating the daily counts.

### Key Steps
**Load Data**:
   - Load arrays tracking the first appearance of each address type (`raw`, `Byron`, `Shelley`, `delegation`) from saved files.
   - Remove placeholder values (`999999999999`), which indicate addresses that did not appear.

**Aggregate New Addresses per Day**:
   - Count the number of addresses for each day and store the results in arrays:
     - `num_of_raw_addresses_per_day`
     - `num_of_Byron_addresses_per_day`
     - `num_of_Shelley_addresses_per_day`
     - `num_of_delegation_addresses_per_day`


### Code
```python
# Find number of "new addresses" in each "day":

# Remove placeholder values
place_holder = 999999999999

raw_addresses_new_per_day_array = [value for value in raw_addresses_new_per_day_array if value != place_holder]
Byron_payment_addresses_new_per_day_array = [value for value in Byron_payment_addresses_new_per_day_array if value != place_holder]
Shelley_payment_addresses_new_per_day_array = [value for value in Shelley_payment_addresses_new_per_day_array if value != place_holder]
delegation_addresses_new_per_day_array = [value for value in delegation_addresses_new_per_day_array if value != place_holder]

# Sort arrays
sorted_raw_addresses_new_per_day_array = np.sort(raw_addresses_new_per_day_array, axis=None)
sorted_Byron_payment_addresses_new_per_day_array = np.sort(Byron_payment_addresses_new_per_day_array, axis=None)
sorted_Shelley_payment_addresses_new_per_day_array = np.sort(Shelley_payment_addresses_new_per_day_array, axis=None)
sorted_delegation_addresses_new_per_day_array = np.sort(delegation_addresses_new_per_day_array, axis=None)

# Calculate number of days
num_of_days_raw = max(sorted_raw_addresses_new_per_day_array) + 1
num_of_days_byron = max(sorted_Byron_payment_addresses_new_per_day_array) + 1
num_of_days_shelley = max(sorted_Shelley_payment_addresses_new_per_day_array) + 1
num_of_days_delegation = max(sorted_delegation_addresses_new_per_day_array) + 1
print('num_of_days_raw = ', num_of_days_raw)
print('num_of_days_byron = ', num_of_days_byron)
print('num_of_days_shelley = ', num_of_days_shelley)
print('num_of_days_delegation = ', num_of_days_delegation)
MAX_num_of_days = max(num_of_days_raw, num_of_days_byron, num_of_days_shelley, num_of_days_delegation)

# Calculate new addresses per day
num_of_raw_addresses_per_day = np.array([0] * MAX_num_of_days)
for i in tqdm(range(MAX_num_of_days)):
    x = BinarySearch_Find_start_end(sorted_raw_addresses_new_per_day_array, i)
    if x != -1:
        num_of_raw_addresses_per_day[i] = x[1] - x[0] + 1

num_of_Byron_addresses_per_day = np.array([0] * MAX_num_of_days)
for i in tqdm(range(MAX_num_of_days)):
    x = BinarySearch_Find_start_end(sorted_Byron_payment_addresses_new_per_day_array, i)
    if x != -1:
        num_of_Byron_addresses_per_day[i] = x[1] - x[0] + 1

num_of_Shelley_addresses_per_day = np.array([0] * MAX_num_of_days)
for i in tqdm(range(MAX_num_of_days)):
    x = BinarySearch_Find_start_end(sorted_Shelley_payment_addresses_new_per_day_array, i)
    if x != -1:
        num_of_Shelley_addresses_per_day[i] = x[1] - x[0] + 1

num_of_delegation_addresses_per_day = np.array([0] * MAX_num_of_days)
for i in tqdm(range(MAX_num_of_days)):
    x = BinarySearch_Find_start_end(sorted_delegation_addresses_new_per_day_array, i)
    if x != -1:
        num_of_delegation_addresses_per_day[i] = x[1] - x[0] + 1
```


*** 

## Calculate Active Users and Entities Per Day

This cell calculates the number of active users (addresses) and active entities (clusters of addresses) per day based on transaction data. It processes transaction inputs to identify active participants and groups them by day.

1. **Datetime Initialization**:
   - The analysis covers the period from Cardano's launch (September 23, 2017) to January 21, 2023.
   - Computes the total operational days.

2. **Active User and Entity Tracking**:
   - Initializes arrays to store the daily count of active addresses and entities:
     - `active_addresses_per_day_array`
     - `active_entities_per_day_array`

3. **File Processing**:
   - Reads transaction data from multiple CSV files.
   - For each transaction:
     - Extracts inputs.
     - Identifies the address and its corresponding entity using a clustering array.
     - Adds the address and entity to daily active sets.

4. **Daily Aggregation**:
   - At the end of each day, counts the unique active addresses and entities.
   - Resets the daily tracking for the next day's processing.


### CSV File Details
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID` (Transaction ID)
  - `BLOCK_TIME` (Transaction timestamp in `%Y-%m-%d %H:%M:%S` format)
  - `EPOCH_NO` (Epoch number)
  - `INPUTs` (Semicolon-separated list containing input details)

### Code
```python
# Calculate Active Users and Entities Per Day:

# Initialize date range
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

current_delta_day = 0
active_addresses = []
active_entities = []

# Initialize daily arrays
active_addresses_per_day_array = [0] * total_time_length_CARDANO
active_entities_per_day_array = [0] * total_time_length_CARDANO

# File details
CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process each file
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()

    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        if tx_delta_day > current_delta_day:
            # End of the current day, update counts
            active_addresses_per_day_array[current_delta_day] = len(set(active_addresses))
            active_entities_per_day_array[current_delta_day] = len(set(active_entities))
            current_delta_day = tx_delta_day
            active_addresses = []
            active_entities = []

        EPOCH_NO = str(df.loc[index, 'EPOCH_NO'])

        # Process transaction inputs
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        for tx_input in inputs_list:
            address_raw = tx_input.split(',')[4]
            address_has_script = tx_input.split(',')[7]
            payment_cred = tx_input.split(',')[8]
            stake_address = tx_input.split(',')[9]

            if address_has_script == 'f':  # Non-smart contract address
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    active_addresses.append(addr_indx)
                    active_entities.append(entity_indx)
```

*** 

# Store/Load Active Users and Entities Per Day

This cell provides functionality to save and load daily counts of active users (addresses) and entities (clusters) to and from files.

#### Store Functionality
1. **Timestamped Filenames**:
   - Files are named with a timestamp to allow versioning and unique identification.
   - Arrays saved:
     - `active_addresses_per_day_array`
     - `active_entities_per_day_array`
2. **File Format**:
   - Data is stored as `.txt` files in CSV format.

#### Load Functionality
1. **Input File Names**:
   - Arrays are loaded from specified `.txt` files.
   - The filenames must match the timestamped naming convention used during saving.
2. **File Format**:
   - Data is loaded as arrays using a custom file loading function.

### File Details
- **Filename Format for Saving**:
  - `activeAddressesPerDayList__Cardano_TXs_All__{Timestamp}.txt`
  - `activeEntitiesPerDayList__Cardano_TXs_All__{Timestamp}.txt`
  - Example: `activeAddressesPerDayList__Cardano_TXs_All__2023-04-09_224357.txt`
- **Data Format**: CSV-like `.txt` files
- **Columns**: 
  - Single column containing integers representing daily counts of active addresses or entities.

### Code
```python
# Store/Load "active_addresses_per_day_array" and "active_entities_per_day_array" into/from file:

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store "active_addresses_per_day_array" and "active_entities_per_day_array" to file:
output_filename = BASE_ADDRESS + '/activeAddressesPerDayList__Cardano_TXs_All__' + curr_timestamp + '.txt'
store_array_to_file(active_addresses_per_day_array, output_filename)

output_filename = BASE_ADDRESS + '/activeEntitiesPerDayList__Cardano_TXs_All__' + curr_timestamp + '.txt'
store_array_to_file(active_entities_per_day_array, output_filename)

# Load "active_addresses_per_day_array" and "active_entities_per_day_array" from File:
file_name = BASE_ADDRESS + '/activeAddressesPerDayList__Cardano_TXs_All__2023-04-09_224357.txt'
active_addresses_per_day_array = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/activeEntitiesPerDayList__Cardano_TXs_All__2023-04-09_224357.txt'
active_entities_per_day_array = load_file_to_array(file_name)

```

***


## Calculate "Number of NFT/FT/ADA Entity-Level Transactions" Over Time

This cell processes Cardano blockchain data to calculate the number of NFT (Non-Fungible Token), FT (Fungible Token), and ADA transactions at the entity level over time. It also updates the balances for each entity.

1. **Initialization**:
   - Defines the clustering array to group addresses into entities.
   - Initializes arrays for tracking balances and transactions:
     - `balances_per_entity_array`: Current balance for each entity.
     - `entity_level_ADA_Transactions`: Tracks ADA transactions between entities.
     - `entity_level_NFT_Transactions`: Tracks NFT transactions between entities.
     - `entity_level_FT_Transactions`: Tracks FT transactions between entities.

2. **Date Range**:
   - Operates over Cardano's active period (September 23, 2017 - January 21, 2023).
   - Calculates the total operational days.

3. **File Processing**:
   - Reads transaction data from a CSV file.
   - Extracts inputs and outputs for each transaction:
     - Identifies input entities based on clustering.
     - Identifies entities receiving ADA, NFTs, and FTs.

4. **Transaction Tracking**:
   - For each transaction:
     - ADA transactions are recorded with input and output entity IDs and their balances.
     - NFT/FT transactions are recorded similarly, filtering based on multi-asset details.

5. **Balance Updates**:
   - Updates entity balances based on transaction inputs (deduction) and outputs (addition).

6. **File Output**:
   - Stores the resulting arrays in timestamped files:
     - `balances_per_entity_array`
     - `entity_level_ADA_Transactions`
     - `entity_level_NFT_Transactions`
     - `entity_level_FT_Transactions`
   - Outputs the file paths to a queue for further processing.

7. **Logging**:
   - Logs elapsed time for processing.

### Input File Details
- **Filename**: `cardano_TXs_NFTs_1.csv`
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID`: Transaction ID
  - `BLOCK_TIME`: Timestamp in `%Y-%m-%d %H:%M:%S` format
  - `EPOCH_NO`: Epoch number
  - `INPUTs`: Semicolon-separated list of inputs
  - `OUTPUTs`: Semicolon-separated list of outputs
  - `TX_OUTPUT_MAs`: Semicolon-separated list of multi-asset outputs

### Output File Details
- **Filename Format**:
  - `balances_per_entity_array__TEMP__{csv_file_basename}__{timestamp}.txt`
  - `entity_level_ADA_Transactions__TEMP__{csv_file_basename}__{timestamp}.txt`
  - `entity_level_NFT_Transactions__TEMP__{csv_file_basename}__{timestamp}.txt`
  - `entity_level_FT_Transactions__TEMP__{csv_file_basename}__{timestamp}.txt`
- **Format**: Pickle files
- **Content**:
  - `balances_per_entity_array`: Entity balances.
  - `entity_level_ADA_Transactions`: Tuple of `(day_delta, entity_from, entity_to, balance_from, balance_to)`.
  - `entity_level_NFT_Transactions`: Similar format, but for NFTs.
  - `entity_level_FT_Transactions`: Similar format, but for FTs.

### Code
```python
print('----------------------')
def Find_ADA_NFT_FT_TXs_per_Entity_over_time(queue_):
    # Read input queue arguments
    in_args = queue_.get()
    csv_file_name = 'cardano_TXs_NFTs_1.csv'
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
    FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
    total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

    # Load file
    ct_temp = datetime.datetime.now()
    file_name = csv_file_name
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()
    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Extract inputs and outputs
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))

        # Input entities
        input_entity_indx = []
        for tx_input in inputs_list:
            address_raw = tx_input.split(',')[4]
            address_has_script = tx_input.split(',')[7]
            payment_cred = tx_input.split(',')[8]
            stake_address = tx_input.split(',')[9]
            if address_has_script == 'f':  # Non-Smart Contract Address
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    input_entity_indx.append(clustering_array[addr_indx][0])
                    break

        if not input_entity_indx:
            continue

        # Output entities
        output_ADA_entities_indx = []
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            address_has_script = tx_output.split(',')[4]
            payment_cred = tx_output.split(',')[5]
            stake_address = tx_output.split(',')[6]
            if address_has_script == 'f':  # Non-Smart Contract Address
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    output_ADA_entities_indx.append(clustering_array[addr_indx][0])

        output_ADA_entities_indx = list(set(output_ADA_entities_indx))
        if input_entity_indx[0] in output_ADA_entities_indx:
            output_ADA_entities_indx.remove(input_entity_indx[0])

        # Process NFT/FT transactions
        TX_OUTPUT_MAs_list = list(df.loc[index, 'TX_OUTPUT_MAs'].split(';'))
        # (NFT and FT logic similar to above)

        #################################################################
        # Store outputs
        ct_file = datetime.datetime.now()
        curr_timestamp = str(ct_file)[0:10] + '_' + str(ct_file)[11:13] + str(ct_file)[14:16] + str(ct_file)[17:26]

        output_balances_filename = TEMP_ADDRESS + '/balances_per_entity_array__TEMP__' + csv_file_basename + '__' + curr_timestamp + '.txt'
        pickle.dump(balances_per_entity_array, open(output_balances_filename, 'wb'))

        output_adaTXs_filename = TEMP_ADDRESS + '/entity_level_ADA_Transactions__TEMP__' + csv_file_basename + '__' + curr_timestamp + '.txt'
        pickle.dump(entity_level_ADA_Transactions, open(output_adaTXs_filename, 'wb'))

        # Add more saving logic for NFT and FT transactions
    print("done!")
```

***

# Create and Fill "balances_per_entity_array" and "entity_level_  ADA/NFT/FT/  _Transactions" arrays ("Heuristic 1 and 2"):

This script uses multiprocessing to calculate the number of NFT, FT, and ADA transactions over time at the entity level. The main steps include creating queues and processes, distributing the workload, and collecting the results after parallel execution.

1. **Queue Setup**:
   - Each queue (`q1` to `q6`) stores the file path for one CSV file containing transaction data.

2. **Process Creation**:
   - Each process runs the `Find_ADA_NFT_FT_TXs_per_Entity_over_time` function, using one queue as input.
   - Six processes are created to handle six CSV files.

3. **Parallel Execution**:
   - Processes are started and joined in batches to optimize resource usage and avoid overloading the system.

4. **Result Retrieval**:
   - After processing, results are retrieved from the queues:
     - Entity balances.
     - ADA transactions.
     - NFT transactions.
     - FT transactions.
   - Results are deserialized using `pickle` and stored in separate arrays.


### Code
```python
if __name__ == "__main__":  # confirms that the code is under main function
    q1 = Queue()
    q2 = Queue()
    q3 = Queue()
    q4 = Queue()
    q5 = Queue()
    q6 = Queue()

    # Input files
    q1.put([BASE_ADDRESS + '/cardano_TXs_NFTs_1.csv'])
    q2.put([BASE_ADDRESS + '/cardano_TXs_NFTs_2.csv'])
    q3.put([BASE_ADDRESS + '/cardano_TXs_NFTs_3.csv'])
    q4.put([BASE_ADDRESS + '/cardano_TXs_NFTs_4.csv'])
    q5.put([BASE_ADDRESS + '/cardano_TXs_NFTs_5.csv'])
    q6.put([BASE_ADDRESS + '/cardano_TXs_NFTs_6.csv'])

    # Create Processes
    p1 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q1,))
    p2 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q2,))
    p3 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q3,))
    p4 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q4,))
    p5 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q5,))
    p6 = mp.Process(target=Find_ADA_NFT_FT_TXs_per_Entity_over_time, args=(q6,))

    # Start and Join Processes in Batches
    p1.start()
    p2.start()
    p3.start()
    p1.join()
    p2.join()
    p3.join()

    p4.start()
    p5.start()
    p4.join()
    p5.join()

    p6.start()
    p6.join()

    # Retrieve and Deserialize Results
    print('----------------------')
    output_filename_1 = q1.get()
    balances_per_entity_array_1 = pickle.load(open(output_filename_1[0], 'rb'))
    entity_level_ADA_Transactions_1 = pickle.load(open(output_filename_1[1], 'rb'))
    entity_level_NFT_Transactions_1 = pickle.load(open(output_filename_1[2], 'rb'))
    entity_level_FT_Transactions_1 = pickle.load(open(output_filename_1[3], 'rb'))
    print('arrays_1 loaded!')

    output_filename_2 = q2.get()
    balances_per_entity_array_2 = pickle.load(open(output_filename_2[0], 'rb'))
    entity_level_ADA_Transactions_2 = pickle.load(open(output_filename_2[1], 'rb'))
    entity_level_NFT_Transactions_2 = pickle.load(open(output_filename_2[2], 'rb'))
    entity_level_FT_Transactions_2 = pickle.load(open(output_filename_2[3], 'rb'))
    print('arrays_2 loaded!')

    output_filename_3 = q3.get()
    balances_per_entity_array_3 = pickle.load(open(output_filename_3[0], 'rb'))
    entity_level_ADA_Transactions_3 = pickle.load(open(output_filename_3[1], 'rb'))
    entity_level_NFT_Transactions_3 = pickle.load(open(output_filename_3[2], 'rb'))
    entity_level_FT_Transactions_3 = pickle.load(open(output_filename_3[3], 'rb'))
    print('arrays_3 loaded!')

    output_filename_4 = q4.get()
    balances_per_entity_array_4 = pickle.load(open(output_filename_4[0], 'rb'))
    entity_level_ADA_Transactions_4 = pickle.load(open(output_filename_4[1], 'rb'))
    entity_level_NFT_Transactions_4 = pickle.load(open(output_filename_4[2], 'rb'))
    entity_level_FT_Transactions_4 = pickle.load(open(output_filename_4[3], 'rb'))
    print('arrays_4 loaded!')

    output_filename_5 = q5.get()
    balances_per_entity_array_5 = pickle.load(open(output_filename_5[0], 'rb'))
    entity_level_ADA_Transactions_5 = pickle.load(open(output_filename_5[1], 'rb'))
    entity_level_NFT_Transactions_5 = pickle.load(open(output_filename_5[2], 'rb'))
    entity_level_FT_Transactions_5 = pickle.load(open(output_filename_5[3], 'rb'))
    print('arrays_5 loaded!')

    output_filename_6 = q6.get()
    balances_per_entity_array_6 = pickle.load(open(output_filename_6[0], 'rb'))
    entity_level_ADA_Transactions_6 = pickle.load(open(output_filename_6[1], 'rb'))
    entity_level_NFT_Transactions_6 = pickle.load(open(output_filename_6[2], 'rb'))
    entity_level_FT_Transactions_6 = pickle.load(open(output_filename_6[3], 'rb'))
    print('arrays_6 loaded!')
```




***

# Merge `balances_per_entity` and `entity_level_ADA/NFT/FT_Transactions` Arrays

This code merges data from multiple transaction files (for multiple subsets of transactions), combining entity balances and transaction records (ADA, NFT, and FT) into consolidated arrays for "all" transactions.


### Key Steps

#### 1. **Combine Entity Balances**
- Combines entity balances from six separate arrays into a unified array (`balances_per_entity_array_AllTXs`).
- The balance for each entity is the sum of balances across all files.

#### 2. **Update ADA Transactions**
- For each transaction in the individual ADA transaction arrays (`entity_level_ADA_Transactions_*`):
  - Updates the `balance_entity_from` and `balance_entity_to` values to include balances from preceding files.
- Merges all ADA transactions into a single array, `entity_level_ADA_Transactions_AllTXs`.

#### 3. **Update NFT Transactions**
- Similar to ADA transactions:
  - Updates `balance_entity_from` and `balance_entity_to` values for NFT transactions.
  - Merges all NFT transactions into `entity_level_NFT_Transactions_AllTXs`.

#### 4. **Update FT Transactions**
- Similar to ADA and NFT transactions:
  - Updates `balance_entity_from` and `balance_entity_to` values for FT transactions.
  - Merges all FT transactions into `entity_level_FT_Transactions_AllTXs`.

#### 5. **Utility Function**
- `change_tuple_item`: A helper function to modify specific fields in transaction tuples.

### Code
```python
print('----------------------')

def change_tuple_item(t, index, new_value):
    t_list = list(t)
    t_list[index] = new_value
    new_t = tuple(t_list)    
    return new_t

# Combine Entity Balances
balances_per_entity_array_AllTXs = np.array([0] * len(balances_per_entity_array_1))
for i in range(len(balances_per_entity_array_AllTXs)):
    balances_per_entity_array_AllTXs[i] = (balances_per_entity_array_1[i] +
                                           balances_per_entity_array_2[i] +
                                           balances_per_entity_array_3[i] + 
                                           balances_per_entity_array_4[i] + 
                                           balances_per_entity_array_5[i] + 
                                           balances_per_entity_array_6[i])

# Update and Merge ADA Transactions
for i in tqdm(range(len(entity_level_ADA_Transactions_2))):
    entity_from = entity_level_ADA_Transactions_2[i][1]
    entity_to   = entity_level_ADA_Transactions_2[i][2]
    entity_level_ADA_Transactions_2[i] = change_tuple_item(entity_level_ADA_Transactions_2[i], 3, entity_level_ADA_Transactions_2[i][3] + balances_per_entity_array_1[entity_from])
    entity_level_ADA_Transactions_2[i] = change_tuple_item(entity_level_ADA_Transactions_2[i], 4, entity_level_ADA_Transactions_2[i][4] + balances_per_entity_array_1[entity_to])

# (Repeat for entity_level_ADA_Transactions_3 through entity_level_ADA_Transactions_6)

entity_level_ADA_Transactions_AllTXs = entity_level_ADA_Transactions_1
entity_level_ADA_Transactions_AllTXs.extend(entity_level_ADA_Transactions_2)
entity_level_ADA_Transactions_AllTXs.extend(entity_level_ADA_Transactions_3)
entity_level_ADA_Transactions_AllTXs.extend(entity_level_ADA_Transactions_4)
entity_level_ADA_Transactions_AllTXs.extend(entity_level_ADA_Transactions_5)
entity_level_ADA_Transactions_AllTXs.extend(entity_level_ADA_Transactions_6)

# Update and Merge NFT Transactions
for i in tqdm(range(len(entity_level_NFT_Transactions_2))):
    entity_from = entity_level_NFT_Transactions_2[i][1]
    entity_to   = entity_level_NFT_Transactions_2[i][2]
    entity_level_NFT_Transactions_2[i] = change_tuple_item(entity_level_NFT_Transactions_2[i], 3, entity_level_NFT_Transactions_2[i][3] + balances_per_entity_array_1[entity_from])
    entity_level_NFT_Transactions_2[i] = change_tuple_item(entity_level_NFT_Transactions_2[i], 4, entity_level_NFT_Transactions_2[i][4] + balances_per_entity_array_1[entity_to])

# (Repeat for entity_level_NFT_Transactions_3 through entity_level_NFT_Transactions_6)

entity_level_NFT_Transactions_AllTXs = entity_level_NFT_Transactions_1
entity_level_NFT_Transactions_AllTXs.extend(entity_level_NFT_Transactions_2)
entity_level_NFT_Transactions_AllTXs.extend(entity_level_NFT_Transactions_3)
entity_level_NFT_Transactions_AllTXs.extend(entity_level_NFT_Transactions_4)
entity_level_NFT_Transactions_AllTXs.extend(entity_level_NFT_Transactions_5)
entity_level_NFT_Transactions_AllTXs.extend(entity_level_NFT_Transactions_6)

# Update and Merge FT Transactions
for i in tqdm(range(len(entity_level_FT_Transactions_2))):
    entity_from = entity_level_FT_Transactions_2[i][1]
    entity_to   = entity_level_FT_Transactions_2[i][2]
    entity_level_FT_Transactions_2[i] = change_tuple_item(entity_level_FT_Transactions_2[i], 3, entity_level_FT_Transactions_2[i][3] + balances_per_entity_array_1[entity_from])
    entity_level_FT_Transactions_2[i] = change_tuple_item(entity_level_FT_Transactions_2[i], 4, entity_level_FT_Transactions_2[i][4] + balances_per_entity_array_1[entity_to])

# (Repeat for entity_level_FT_Transactions_3 through entity_level_FT_Transactions_6)

entity_level_FT_Transactions_AllTXs = entity_level_FT_Transactions_1
entity_level_FT_Transactions_AllTXs.extend(entity_level_FT_Transactions_2)
entity_level_FT_Transactions_AllTXs.extend(entity_level_FT_Transactions_3)
entity_level_FT_Transactions_AllTXs.extend(entity_level_FT_Transactions_4)
entity_level_FT_Transactions_AllTXs.extend(entity_level_FT_Transactions_5)
entity_level_FT_Transactions_AllTXs.extend(entity_level_FT_Transactions_6)

print('----------------------')
print('done!')

```



***


## Store/Load "Entity-Level Transactions" for ADA, NFT, and FT

### Explanation
This script provides functionality to store and load entity-level transaction data (ADA, NFT, FT) into/from files. 

#### Store Functionality
1. **Timestamped Filenames**:
   - Files are named with the current timestamp for unique identification.
   - Arrays stored:
     - `entity_level_ADA_Transactions_AllTXs`
     - `entity_level_NFT_Transactions_AllTXs`
     - `entity_level_FT_Transactions_AllTXs`
2. **File Format**:
   - Data is serialized and saved using Python's `pickle`.

#### Load Functionality
1. **Input File Names**:
   - Arrays are loaded from specified binary files.
   - The filenames should match the saved files with correct timestamps.
2. **File Format**:
   - Data is deserialized using `pickle` and loaded into arrays.

### File Details
- **Filename Format**:
  - `EntityTXsADA_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__{timestamp}`
  - `EntityTXsNFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__{timestamp}`
  - `EntityTXsFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__{timestamp}`
  - Example: `EntityTXsADA_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-11-22_103423`
- **Format**: Pickle
- **Content**:
  - Each file contains a list of tuples representing transactions:
    - `(day_delta, entity_from, entity_to, balance_entity_from, balance_entity_to)`

### Code
```python
print('----------------------')
import pickle

# Current timestamp
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store to file
output_filename = BASE_ADDRESS + '/EntityTXsADA_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
pickle.dump(entity_level_ADA_Transactions_AllTXs, open(output_filename, 'wb'))

output_filename = BASE_ADDRESS + '/EntityTXsNFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
pickle.dump(entity_level_NFT_Transactions_AllTXs, open(output_filename, 'wb'))

output_filename = BASE_ADDRESS + '/EntityTXsFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
pickle.dump(entity_level_FT_Transactions_AllTXs, open(output_filename, 'wb'))

# Load from file
file_name = BASE_ADDRESS + '/EntityTXsADA_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>
entity_level_ADA_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityTXsNFT_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>'
entity_level_NFT_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))

file_name = BASE_ADDRESS + '/EntityTXsFT_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>'
entity_level_FT_Transactions_AllTXs = pickle.load(open(file_name, 'rb'))

print('----------------------')
print('done!')
```

***

# Calculate "NFTs Owned by Each Entity" and "Balances of Entities"

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

- **File Format**: Pickle 
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
output_filename = BASE_ADDRESS + '/EntityOwnNFTsWithNameArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
pickle.dump(NFTs_owned_per_entity_array, open(output_filename, 'wb'))


output_filename = BASE_ADDRESS + '/EntityOwnNFTsNumberArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
pickle.dump(count_NFTs_per_entity, open(output_filename, 'wb'))


output_filename = BASE_ADDRESS + '/EntityBalancesArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
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

# Find NFTs and FTs Minted by Each Entity:

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
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

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

```



***

# Store/Load "NFTs Minted per Entity" and "FTs Minted per Entity"

This script handles the storage and retrieval of the following data arrays:
- **`NFTs_minted_per_entity_array`**: 2D Lists of NFTs minted by each entity.
- **`FTs_minted_per_entity_array`**: 2D Lists of FTs minted by each entity.



**File Naming**:
   - The file naming convention includes the heuristic method and a timestamp for traceability.



#### **Code**

```python

# Current timestamp
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store data to files
output_filename = BASE_ADDRESS + '/EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file_2D(NFTs_minted_per_entity_array, output_filename)

output_filename = BASE_ADDRESS + '/EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file_2D(FTs_minted_per_entity_array, output_filename)

# Load data from files
file_name = BASE_ADDRESS + '/EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-04-08_075547'
NFTs_minted_per_entity_array = load_file_to_array_2D(file_name)

file_name = BASE_ADDRESS + '/EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-04-08_075547'
FTs_minted_per_entity_array = load_file_to_array_2D(file_name)

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


#### Key Variables
- **`balances_array`**: Tracks the net balance for each unique payment address.
- **`gini_array`**: Stores Gini index values to monitor balance inequality over time.



#### Code

```python
# Generate "balances_array" for payment addresses:


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

```



***


# Store "balances_array" and "gini_array" into Files

This script saves both the `balances_array` (containing balances for payment addresses) and the `gini_array` (containing Gini index values) into separate files for later use.


#### Process

1. **Timestamp Generation**:
   - A timestamp in the format `YYYY-MM-DD_HHMMSS` is created to ensure file names are unique.

2. **File Naming**:
   - **Balances Array File**:
     - File format: `balancesList_paymentAddresses_noSC__Cardano_TXs_All__<timestamp>`
     - Example: `balancesList_paymentAddresses_noSC__Cardano_TXs_All__2024-11-22_123456`
   - **Gini Array File**:
     - File format: `giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__<timestamp>`
     - Example: `giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__2024-11-22_123456`

3. **Storing the Arrays**:
   - The `store_array_to_file` function is used to save each array into its respective file.
   - Each value in the array is stored on a new line.

#### Code

```python
# Store "balances_array" and "gini_array" into files:

# Store balances_array
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/balancesList_paymentAddresses_noSC__Cardano_TXs_All__' + curr_timestamp + '.txt'
store_array_to_file(balances_array, output_filename)


# Store gini_array
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/giniArray_noZeros__paymentAddresses_noSC__Cardano_TXs_All__' + curr_timestamp + '.txt'
store_array_to_file(gini_array, output_filename)

```


***

# Calculate Entities' Daily Balances:

This script calculates the daily balances for Cardano entities by aggregating transactions from multiple CSV files. It processes inputs and outputs to update the balances of entities for each day.

#### Key Steps

1. **Initialization**:
   - `balances_per_entity_array`: Stores balances for each entity, initialized to zero.
   - `current_delta_day`: Tracks the current day being processed, starting from the initial blockchain date.

2. **Process CSV Files**:
   - Reads multiple transaction CSV files.
   - For each transaction:
     - **Inputs**: Subtracts UTXO values from the corresponding entity's balance.
     - **Outputs**: Adds UTXO values to the corresponding entity's balance.

3. **Daily Balances**:
   - At the end of each day (`current_delta_day`), the balances are saved to a file using `store_array_to_file`.
   - The process continues to the next day, resetting the daily balance calculations.

4. **Logging**:
   - Logs the elapsed time for processing each file.

### Input File Details
- **Filename**: `cardano_TXs_Velocity_{index}.csv`
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID`: Transaction ID
  - `BLOCK_TIME`: Timestamp in `%Y-%m-%d %H:%M:%S` format
  - `INPUTs`: Semicolon-separated list of inputs
  - `OUTPUTs`: Semicolon-separated list of outputs

### Output File Details
- **Filename Format**:
  - `BalancesPerEntityDay_{day}`
- **Content**:
  - A list of balances for all entities at the end of each day.
- **Format**: Pickle file with serialized arrays.

### Code
```python
print('----------------------')
import random
import pickle

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Define date range
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

# Choose clustering array
clustering_array = clustering_array_heur1and2

# Initialize balances array
balances_per_entity_array = [0] * (np.amax(clustering_array) + 1)
current_delta_day = 0

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_Velocity_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process files
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Save daily balances and move to the next day
        if current_delta_day < tx_delta_day:
            output_filename = BASE_ADDRESS + '/Cardano_Balances_Entities/BalancesPerEntityDay_' + str(current_delta_day).zfill(4) + '__Cardano_TXs_All'
            store_array_to_file(balances_per_entity_array, output_filename)
            current_delta_day = tx_delta_day

        # Process inputs
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        for i in range(len(inputs_list)):
            address_has_script = inputs_list[i].split(',')[7]
            if address_has_script == 'f':  # Non-Smart Contract Address
                address_raw = inputs_list[i].split(',')[4]
                payment_cred = inputs_list[i].split(',')[8]
                stake_address = inputs_list[i].split(',')[9]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    UTXO_value = int(inputs_list[i].split(',')[6])
                    balances_per_entity_array[entity_indx] -= int(UTXO_value)

        # Process outputs
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for i in range(len(outputs_list)):
            address_has_script = outputs_list[i].split(',')[4]
            if address_has_script == 'f':  # Non-Smart Contract Address
                address_raw = outputs_list[i].split(',')[1]
                payment_cred = outputs_list[i].split(',')[5]
                stake_address = outputs_list[i].split(',')[6]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    UTXO_value = int(outputs_list[i].split(',')[3])
                    balances_per_entity_array[entity_indx] += int(UTXO_value)
```



***


# Calculate Entities' Daily Transaction Volume:

### Explanation
This script calculates the daily transaction volume for blockchain entities by processing transaction inputs and outputs. The volumes are aggregated for each entity on a daily basis.

#### Key Steps

1. **Initialization**:
   - `TX_vol_per_entity_array`: An array to store transaction volumes for each entity, initialized to zero.
   - `current_delta_day`: Tracks the current day being processed, starting from the initial blockchain date.

2. **Process CSV Files**:
   - Reads multiple transaction CSV files.
   - For each transaction:
     - **Inputs**: Subtracts UTXO values from the transaction volume of the corresponding entity.
     - **Outputs**: Adds UTXO values to the transaction volume of the corresponding entity.

3. **Daily Transaction Volumes**:
   - At the end of each day (`current_delta_day`), transaction volumes are saved to a file using `pickle.dump`.
   - Resets the daily transaction volume array for the next day.

4. **Logging**:
   - Logs the elapsed time for processing each file.

### Input File Details
- **Filename**: `cardano_TXs_Velocity_{index}.csv`
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID`: Transaction ID
  - `BLOCK_TIME`: Timestamp in `%Y-%m-%d %H:%M:%S` format
  - `INPUTs`: Semicolon-separated list of inputs
  - `OUTPUTs`: Semicolon-separated list of outputs

### Output File Details
- **Filename Format**:
  - `TX_Vol_PerEntityDay_{day}.pickle`
- **Content**:
  - A list containing the transaction volumes for each entity at the end of the day.
- **Format**: `.pickle` serialized files.

### Code
```python
print('----------------------')
import random
import pickle

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

# Define date range
INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

# Choose clustering array
clustering_array = clustering_array_heur1and2

# Initialize transaction volume array
TX_vol_per_entity_array = [0] * (np.amax(clustering_array) + 1)
current_delta_day = 0

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_Velocity_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

# Process files
for i in range(1, NUMBER_OF_CSV_FILES + 1):
    ct_temp = datetime.datetime.now()
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')
    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Load CSV File " + file_name + "): ", et_temp)

    ct_temp = datetime.datetime.now()

    for index, row in tqdm(df.iterrows()):
        TX_ID = df.loc[index, 'TX_ID']
        BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
        tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)

        # Save daily transaction volume and move to the next day
        if current_delta_day < tx_delta_day:
            output_filename = BASE_ADDRESS + '/Cardano_TX_Vols_Entities__PICKLE/TX_Vol_PerEntityDay_' + str(current_delta_day).zfill(4) + '__Cardano_TXs_All.pickle'
            pickle.dump(TX_vol_per_entity_array, open(output_filename, 'wb'))
            TX_vol_per_entity_array = [0] * (np.amax(clustering_array) + 1)
            current_delta_day = tx_delta_day

        # Process inputs
        inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
        TX_vol_dict = {}
        for tx_input in inputs_list:
            tx_input_split = tx_input.split(',')
            address_has_script = tx_input_split[7]
            if address_has_script == 'f':  # Non-Smart Contract Address
                address_raw = tx_input_split[4]
                payment_cred = tx_input_split[8]
                stake_address = tx_input_split[9]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    UTXO_value = int(tx_input_split[6])
                    TX_vol_dict[entity_indx] = TX_vol_dict.get(entity_indx, 0) - UTXO_value

        # Process outputs
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for tx_output in outputs_list:
            tx_output_split = tx_output.split(',')
            address_has_script = tx_output_split[4]
            if address_has_script == 'f':  # Non-Smart Contract Address
                address_raw = tx_output_split[1]
                payment_cred = tx_output_split[5]
                stake_address = tx_output_split[6]
                [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                if address_payment_part != '':
                    addr_indx = BinarySearch(unique_payment_addresses, address_payment_part)
                    entity_indx = clustering_array[addr_indx][0]
                    UTXO_value = int(tx_output_split[3])
                    TX_vol_dict[entity_indx] = TX_vol_dict.get(entity_indx, 0) + UTXO_value

        for key in TX_vol_dict:
            TX_vol_per_entity_array[key] += abs(TX_vol_dict[key])

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Process CSV File " + file_name + "): ", et_temp)

print('----------------------')
print('done!')
```


***






