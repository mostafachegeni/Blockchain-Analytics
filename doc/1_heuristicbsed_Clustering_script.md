# heuristicbsed_Clustering_script

This 'code' analyzes Cardano transactions, applies two heuristics to cluster the addresses into entities, and creates/stores the clustering results as an array.
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

# Extracts all cardano addresses (raw, payment, and delegation):

This cell extracts all addresses (raw, payment, and delegation) from Cardano transaction data stored in CSV files. The extracted addresses are categorized into three separate lists: `raw_address_list`, `payment_address_list`, and `delegation_address_list`. Additionally, it measures the time taken for loading and processing the data. Below is a detailed explanation of the code:


**File Details**:
    - The input files follow a naming convention: `BASE_ADDRESS + '/cardano_TXs_<number>.csv'`, where `<number>` ranges from 1 to `NUMBER_OF_CSV_FILES`.
    - Each file is a CSV file with a pipe (`|`) delimiter, and it contains columns relevant to transaction inputs and outputs.

**File Loading and Address Extraction**:
    - The script iterates through all specified CSV files.
    - For each file:
        - It measures the time taken to load the file into a DataFrame.
        - It processes each row of the DataFrame:
            - Extracts transaction output details from the `OUTPUTs` column, splitting it into components to retrieve the raw address, payment credentials, and delegation/stake address.
            - Uses the `extract_payment_delegation_parts` function to further split addresses into their payment and delegation components.
            - Appends these components to the corresponding lists if they are not empty.

### Code
```python
# Create extracts all addresses (raw, payment, and delegation) appeared on the cardano transactions and creates a List for each 
# [raw_address_list, payment_address_list, and delegation_address_list]:

# List of all addresses (from INPUTs and OUTPUTs)
raw_address_list = []
payment_address_list = []
delegation_address_list = []

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
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            address_has_script = tx_output.split(',')[4]
            payment_cred = tx_output.split(',')[5]
            stake_address = tx_output.split(',')[6]
            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
            if (address_raw != ''):
                raw_address_list.append(address_raw)
            if (address_payment_part != ''):
                payment_address_list.append(address_payment_part)
            if (address_delegation_part != ''):
                delegation_address_list.append(address_delegation_part)

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Extract Addresses from INs/OUTs of CSV File " + file_name + "): ", et_temp)

```


***


# Store "raw_address_list / payment_address_list / delegation_address_list" into a File:

This cell saves the extracted address lists (`raw_address_list`, `payment_address_list`, and `delegation_address_list`) into separate text files. Each file's name includes a timestamp to ensure unique identification. Below is a detailed explanation of the process:

**File Details**:
    - Each list is saved into a separate text file in the specified directory (`BASE_ADDRESS`).
    - File naming convention: `AddressList<Type>__Cardano_TXs_All__<timestamp>`, where `<Type>` corresponds to the list type (`Raw`, `Payment`, or `Delegation`).

**Storing Lists**:
    - The function `store_array_to_file` is used to write each list into its respective file. This function is assumed to handle the file writing process (e.g., storing each list item on a new line).



### Code
```python
# Store "raw_address_list / payment_address_list / delegation_address_list" into a File:

# write a list into a file:
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/AddressListRaw__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(raw_address_list, output_filename)

# write a list into a file:
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/AddressListPayment__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(payment_address_list, output_filename)

# write a list into a file:
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/AddressListDelegation__Cardano_TXs_All__' + curr_timestamp 
store_array_to_file(delegation_address_list, output_filename)

```

### Output File Format
- **File Type**: Plain text 
- **Content**: Each address is written on a new line.
    - `AddressListRaw__Cardano_TXs_All__<timestamp>`: Contains all raw addresses.
    - `AddressListPayment__Cardano_TXs_All__<timestamp>`: Contains all payment addresses.
    - `AddressListDelegation__Cardano_TXs_All__<timestamp>`: Contains all delegation addresses.
- **Timestamp**: The file name includes a timestamp in the format `YYYY-MM-DD_HHMMSS`.

### Example File Content
For `AddressListRaw__Cardano_TXs_All__<timestamp>`:
```
addr1qxyz...
addr1qabc...
addr1qlmn...
```

***



# Remove Duplicate Addresses and Sort raw_address_list / payment_address_list / delegation_address_list:

This cell removes duplicate addresses and sorts the `raw_address_list`, `payment_address_list`, and `delegation_address_list`. The operation is performed using the `sort` shell command, which is executed via Python's `os.system` method.

**Sorting and Deduplication**:
    - The `sort` command is used with the `-u` option to remove duplicates and sort the addresses lexicographically.
    - The `-k 1` option specifies sorting based on the first key (default behavior for text files).

**Input and Output Files**:
    - Input files: Address lists generated previously (`AddressListRaw`, `AddressListPayment`, and `AddressListDelegation`).
    - Output files: Deduplicated and sorted address lists, with filenames prefixed by `Unique_`.

### Code
```python
# Remove Duplicate Addresses and Sort raw_address_list / payment_address_list / delegation_address_list:

os.system('sort -k 1 -u /local/scratch/exported/blockchain_parsed/cardano_mostafa/AddressListRaw__Cardano_TXs_All__2023-02-28_143357        > /local/scratch/exported/blockchain_parsed/cardano_mostafa/Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357')
os.system('sort -k 1 -u /local/scratch/exported/blockchain_parsed/cardano_mostafa/AddressListPayment__Cardano_TXs_All__2023-02-28_143953    > /local/scratch/exported/blockchain_parsed/cardano_mostafa/Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953')
os.system('sort -k 1 -u /local/scratch/exported/blockchain_parsed/cardano_mostafa/AddressListDelegation__Cardano_TXs_All__2023-02-28_144415 > /local/scratch/exported/blockchain_parsed/cardano_mostafa/Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415')

```

### Input and Output File Details

- **Input Files**:
    - `AddressListRaw__Cardano_TXs_All__2023-02-28_143357`
    - `AddressListPayment__Cardano_TXs_All__2023-02-28_143953`
    - `AddressListDelegation__Cardano_TXs_All__2023-02-28_144415`
    - These files contain unsorted address lists, possibly with duplicates.

- **Output Files**:
    - `Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357`
    - `Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953`
    - `Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415`
    - These files contain sorted and deduplicated address lists.


***


# Read unique raw/payment/delegation array lists from file:


This cell reads unique address lists (`unique_raw_addresses`, `unique_payment_addresses`, and `unique_delegation_addresses`) from previously saved text files. Each file contains sorted and deduplicated addresses, one address per line. The script calculates and prints the length of each list for verification.

#### Process
**Input Files**:
   - The files contain sorted and unique addresses corresponding to raw, payment, and delegation categories.
   - File names:
     - `Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357`
     - `Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953`
     - `Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415`
   - These files are located in the directory defined by the `BASE_ADDRESS` variable.

**Loading Process**:
   - The `load_file_to_array` function reads each line of the file into an array.
   - Each address becomes an individual element of the array.

**Output**:
   - Arrays:
     - `unique_raw_addresses`: Contains unique raw addresses.
     - `unique_payment_addresses`: Contains unique payment addresses.
     - `unique_delegation_addresses`: Contains unique delegation addresses.
   - Lengths of each array are printed to summarize the number of unique addresses.


```python
# Read unique raw/payment/delegation array lists from file:

file_name = BASE_ADDRESS + '/Unique_AddressesListRaw__Cardano_TXs_All__2023-02-28_143357.txt'
unique_raw_addresses = load_file_to_array(file_name)
print('Length of \"unique_raw_addresses\" = ' + str(len(unique_raw_addresses)))

file_name = BASE_ADDRESS + '/Unique_AddressesListPayment__Cardano_TXs_All__2023-02-28_143953.txt'
unique_payment_addresses = load_file_to_array(file_name)
print('Length of \"unique_payment_addresses\" = ' + str(len(unique_payment_addresses)))

file_name = BASE_ADDRESS + '/Unique_AddressesListDelegation__Cardano_TXs_All__2023-02-28_144415.txt'
unique_delegation_addresses = load_file_to_array(file_name)
print('Length of \"unique_delegation_addresses\" = ' + str(len(unique_delegation_addresses)))

```



***


# Generate "Parents_Array" Based on Heuristic 1 and Union-Find Algorithm

This function generates a `parents_array` and `graph_edges_array` using **Heuristic 1** and the Union-Find algorithm. The focus is to cluster addresses and build a graph representation of the relationships between non-smart contract addresses.

#### Process

1. **Input Parameters**:
   - `queue_`: A multiprocessing queue for inter-process communication.
   - **Queue Input Data**:
     - `csv_file_name`: Path to the transaction CSV file.
     - `Heuristic_1`: Boolean flag to enable the heuristic for linking addresses.

2. **Spark Initialization**:
   - A Spark session is created to efficiently handle large transaction data files.
   - The CSV file is loaded into a Spark DataFrame using a pipe (`|`) delimiter.

3. **Array Initialization**:
   - `parents_array`: Tracks parent nodes for Union-Find operations. Initialized to map each address to itself.
   - `graph_edges_array`: Stores adjacency lists for graph representation.

4. **Heuristic 1**:
   - Links non-smart contract (non-SC) addresses within the transaction inputs.
   - **Steps**:
     1. Extract the `INPUTs` list from each transaction row.
     2. Check whether each address in the `INPUTs` is a non-SC address by inspecting the `address_has_script` field.
     3. Extract the `address_payment_part` using the `extract_payment_delegation_parts` function.
     4. Use `BinarySearch` to locate the position of the address in the unique payment addresses list.
     5. Apply the Union-Find algorithm (`link_address`) to link these addresses.
     6. Store graph edges between linked addresses in `graph_edges_array`.

5. **Output Files**:
   - Resolved `parents_array` and `graph_edges_array` are saved to temporary files.
   - File paths are returned via the multiprocessing queue.


#### Output Files
- **Parents Array File**:
  - File Name: `parentsList_temp__<csv_file_basename>__<timestamp>`
  - Contains the resolved `parents_array`.
- **Graph Edges File**:
  - File Name: `graphEdgesList_temp__<csv_file_basename>__<timestamp>`
  - Contains adjacency lists for graph edges.



```python
# Generate "Parents_Array" based on Heuristic 1 and UnionFind algorithm:

def generate_parents_array(queue_):
    # read input queue arguments
    in_args = queue_.get()

    csv_file_name = in_args[0]
    Heuristic_1   = in_args[1]

    csv_file_basename = os.path.basename(csv_file_name)

    # print current process identity
    str_current_proc = 'current_process()._identity[0] ' + '(' + csv_file_basename + ')' + ' = ' + str(current_process()._identity[0])
    print(str_current_proc)

    # Create SparkSession 
    spark = SparkSession.builder \
                     .master("local[1]") \
                     .appName("Cardano_Spark") \
                     .config('spark.driver.maxResultSize', '70g') \
                     .config('spark.executor.cores', 4) \
                     .config('spark.executor.memory', '30g') \
                     .config('spark.driver.memory', '30g') \
                     .config('spark.memory.offHeap.enabled', True) \
                     .config('spark.memory.offHeap.size', '40g') \
                     .getOrCreate() 

    df = spark.read.option("delimiter", "|").csv(csv_file_name, inferSchema=True, header=True)

    # Initialize parents_array:
    parents_array = np.array([0] * unique_payment_addresses_len)
    for i in range(unique_payment_addresses_len):
        parents_array[i] = i    

    # Initialize graph_edges_array:
    graph_edges_array = [[] for _ in range(unique_payment_addresses_len)]

    for row in df.collect():
        inputs_list  = list(row['INPUTs'].split(';'))
        outputs_list = list(row['OUTPUTs'].split(';'))

        # Heuristic 1
        if Heuristic_1:
            nonSC_addr_positions = []
            for i in range(len(inputs_list)):
                address_has_script = inputs_list[i].split(',')[7]
                if address_has_script == 'f':  # non-Smart Contract Address
                    address_raw   = inputs_list[i].split(',')[4]
                    payment_cred  = inputs_list[i].split(',')[8]
                    stake_address = inputs_list[i].split(',')[9]
                    [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                    if address_payment_part:
                        address_position = BinarySearch(unique_payment_addresses, address_payment_part)
                        nonSC_addr_positions.append(address_position) 

            for i in range(1, len(nonSC_addr_positions)):
                link_address(nonSC_addr_positions[0], nonSC_addr_positions[i], parents_array)                
                for j in range(i):
                    add_edge_info(node_1=nonSC_addr_positions[i], node_2=nonSC_addr_positions[j], edges_array=graph_edges_array, weight=1)

    spark.stop()

    # Resolve parents array
    resolveAll(parents_array)

    # Put file address of parents_array in queue
    ct_file = datetime.datetime.now()
    curr_timestamp = str(ct_file)[0:10] + '_' + str(ct_file)[11:13] + str(ct_file)[14:16] + str(ct_file)[17:26]

    output_parents_filename = TEMP_ADDRESS + '/parentsList_temp__' + csv_file_basename + '__' + curr_timestamp 
    store_array_to_file(parents_array, output_parents_filename)

    output_graghEdges_filename = TEMP_ADDRESS + '/graphEdgesList_temp__' + csv_file_basename + '__' + curr_timestamp 
    store_array_to_file_2D(graph_edges_array, output_graghEdges_filename)

    queue_.put([output_parents_filename, output_graghEdges_filename])

    return

```



***


# Create and Fill "Parents_" Arrays (Related to "Heuristic1"):

This cell processes transaction data from multiple CSV files in parallel using Heuristic1 to create and populate `parents_` arrays (accrding to a `UnionFind` algorithm) and corresponding graph edges for each file. It leverages multiprocessing for efficiency and stores the results for further analysis.



#### Process

1. **Initialization**:
   - Six queues (`q1` to `q6`) are initialized to handle the multiprocessing setup.
   - Each queue receives a set of arguments:
     - CSV file path (e.g., `BASE_ADDRESS + '/cardano_TXs_1.csv'`).
     - Flags for enabling or disabling heuristics (`True` for Heuristic1, `False` for others).

2. **Multiprocessing**:
   - Six processes (`p1` to `p6`) are created, each running the `generate_parents_array` function with the corresponding queue as input.
   - The processes are started concurrently and joined to ensure they finish before proceeding.

3. **Loading Results**:
   - The output filenames (for `parents_` arrays and graph edges) are retrieved from the queues.
   - The arrays are loaded into memory using `load_file_to_array` and `load_file_to_array_2D`.




#### Output
- **`parents_` Arrays**:
  - Represent hierarchical relationships between payment addresses.
  - Generated for each CSV file and loaded into `parents_1`, `parents_2`, ..., `parents_6`.
- **Graph Edges Arrays**:
  - Represent adjacency lists for graph relationships.
  - Loaded into `graghEdges_1`, `graghEdges_2`, ..., `graghEdges_6`.



#### Code

```python
# Create and Fill "Parents_" arrays (related to "Heuristic1"):

if __name__ == "__main__":  # confirms that the code is under main function
    q1 = Queue()
    q2 = Queue()
    q3 = Queue()
    q4 = Queue()
    q5 = Queue()
    q6 = Queue()

    q1.put([BASE_ADDRESS + '/cardano_TXs_1.csv', True, False, False])
    q2.put([BASE_ADDRESS + '/cardano_TXs_2.csv', True, False, False])
    q3.put([BASE_ADDRESS + '/cardano_TXs_3.csv', True, False, False])
    q4.put([BASE_ADDRESS + '/cardano_TXs_4.csv', True, False, False])
    q5.put([BASE_ADDRESS + '/cardano_TXs_5.csv', True, False, False])
    q6.put([BASE_ADDRESS + '/cardano_TXs_6.csv', True, False, False])

    # Create Processes:
    p1 = mp.Process(target=generate_parents_array, args=(q1,))
    p2 = mp.Process(target=generate_parents_array, args=(q2,))
    p3 = mp.Process(target=generate_parents_array, args=(q3,))
    p4 = mp.Process(target=generate_parents_array, args=(q4,))
    p5 = mp.Process(target=generate_parents_array, args=(q5,))
    p6 = mp.Process(target=generate_parents_array, args=(q6,))

    # Start Processes:
    p1.start()
    p2.start()
    p3.start()
    p4.start()
    p5.start()
    p6.start()

    # Wait for Processes to finish:
    p1.join()
    p2.join()
    p3.join()
    p4.join()
    p5.join()
    p6.join()

    print('----------------------')
    output_filename_1 = q1.get()
    parents_1    = load_file_to_array   (output_filename_1[0])
    graghEdges_1 = load_file_to_array_2D(output_filename_1[1])
    print('parents_1 and graghEdges_1 loaded!')

    output_filename_2 = q2.get()
    parents_2    = load_file_to_array   (output_filename_2[0])
    graghEdges_2 = load_file_to_array_2D(output_filename_2[1])
    print('parents_2 and graghEdges_2 loaded!')

    output_filename_3 = q3.get()
    parents_3    = load_file_to_array   (output_filename_3[0])
    graghEdges_3 = load_file_to_array_2D(output_filename_3[1])
    print('parents_3 and graghEdges_3 loaded!')

    output_filename_4 = q4.get()
    parents_4    = load_file_to_array   (output_filename_4[0])
    graghEdges_4 = load_file_to_array_2D(output_filename_4[1])
    print('parents_4 and graghEdges_4 loaded!')

    output_filename_5 = q5.get()
    parents_5    = load_file_to_array   (output_filename_5[0])
    graghEdges_5 = load_file_to_array_2D(output_filename_5[1])
    print('parents_5 and graghEdges_5 loaded!')

    output_filename_6 = q6.get()
    parents_6    = load_file_to_array   (output_filename_6[0])
    graghEdges_6 = load_file_to_array_2D(output_filename_6[1])
    print('parents_6 and graghEdges_6 loaded!')

```


***


# Heuristic 2: Link "Shelley Addresses" with the Same "address_delegation_part"

This cell implements Heuristic 2 to link Shelley addresses that share the same `address_delegation_part`. It achieves this by creating a `stake_delegation_array` and resolving a `parents_heur2_array` to group addresses with shared delegation parts.


#### Process

1. **Initialization**:
   - **`stake_delegation_array`**:
     - A list of lists where each index represents a unique delegation address.
     - Each list contains the indices of payment addresses delegated to that delegation address.
   - **`parents_heur2_array`**:
     - A Union-Find array initialized to map each payment address to itself.
     - Used to link addresses with the same delegation part.

2. **Input Data**:
   - Transaction data is read from multiple CSV files:
     - File format: `BASE_ADDRESS + '/cardano_TXs_<file_number>.csv'`
     - Delimiter: `|`
     - Number of files: `NUMBER_OF_CSV_FILES = 6`

3. **Building the `stake_delegation_array`**:
   - For each output in a transaction:
     - Extract `address_payment_part` and `address_delegation_part` using `extract_payment_delegation_parts`.
     - If both parts are non-empty:
       - Use `BinarySearch` to find the indices of the delegation part and payment part.
       - Append the payment part index to the corresponding delegation index in `stake_delegation_array`.
   - After processing all transactions, each list in `stake_delegation_array` is sorted and deduplicated.

4. **Creating and Filling `parents_heur2_array`**:
   - Using `stake_delegation_array`, link all payment addresses sharing the same delegation part in `parents_heur2_array`.
   - Apply Union-Find's `link_address` and resolve the parents array with `resolveAll`.


#### Output
- **`stake_delegation_array`**:
  - Contains sorted, deduplicated indices of payment addresses for each delegation address.
- **`parents_heur2_array`**:
  - Groups payment addresses sharing the same delegation part into linked components.




#### Code

```python
# Heuristic 2 (link "Shelley Addresses" with the same "address_delegation_part"):

# Initialize stake_delegation_array:
stake_delegation_array = [[] for _ in range(unique_delegation_addresses_len)]

CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_'
NUMBER_OF_CSV_FILES = 6
CSV_FILES_SUFFIX = '.csv'

for i in range(1, NUMBER_OF_CSV_FILES + 1):
    file_name = CSV_FILES_NAME_FORMAT + str(i) + CSV_FILES_SUFFIX
    df = pd.read_csv(file_name, delimiter='|')

    for index, row in tqdm(df.iterrows()):
        outputs_list = list(df.loc[index, 'OUTPUTs'].split(';'))
        for tx_output in outputs_list:
            address_raw = tx_output.split(',')[1]
            address_has_script = tx_output.split(',')[4]
            payment_cred = tx_output.split(',')[5]
            stake_address = tx_output.split(',')[6]
            [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
            if address_payment_part != '' and address_delegation_part != '':
                indx1 = BinarySearch(unique_delegation_addresses, address_delegation_part)
                indx2 = BinarySearch(unique_payment_addresses, address_payment_part)
                stake_delegation_array[indx1].append(indx2)

# Unique sort the "stake_delegation_array":
for i in tqdm(range(len(stake_delegation_array))):
    stake_delegation_array[i] = sorted(set(stake_delegation_array[i]))

# Initialize parents_heur2_array:
parents_heur2_array = np.array([0] * unique_payment_addresses_len)
for i in range(unique_payment_addresses_len):
    parents_heur2_array[i] = i

# Link "Shelley Addresses" with the same "address_delegation_part":
for i in tqdm(range(len(stake_delegation_array))):
    for j in range(1, len(stake_delegation_array[i])):
        link_address(stake_delegation_array[i][0], stake_delegation_array[i][j], parents_heur2_array)

# Resolve parents array:
resolveAll(parents_heur2_array)

```



***


# Load/Store "stake_delegation_array" and "parents_heur2_array" from/into File

This cell handles saving and loading the `stake_delegation_array` and `parents_heur2_array` to and from files for Heuristic 2. These files preserve the relationships between Shelley addresses and their respective delegation parts, allowing for reuse in further analysis.


#### Process

1. **Store `stake_delegation_array`**:
   - Saves the `stake_delegation_array` to a file with a timestamped name for unique identification.
   - Each entry in the array (a list of linked addresses) is stored in a 2D format using `store_array_to_file_2D`.

2. **Load `stake_delegation_array`**:
   - Reads the previously saved file and reconstructs the `stake_delegation_array` using `load_file_to_array_2D`.

3. **Store `parents_heur2_array`**:
   - Saves the `parents_heur2_array` to a separate file with a timestamped name.
   - Uses `store_array_to_file` to save the array, where each entry is stored on a new line.

4. **File Naming**:
   - **`stake_delegation_array` File**:
     - Format: `stakeDelegationArray__Heuristic2__Cardano_TXs_All__<timestamp>`
     - Example: `stakeDelegationArray__Heuristic2__Cardano_TXs_All__2024-11-22_123456`
   - **`parents_heur2_array` File**:
     - Format: `parentsList_Heuristic2__Cardano_TXs_All__<timestamp>`
     - Example: `parentsList_Heuristic2__Cardano_TXs_All__2024-11-22_123456`


#### Code

```python
# Load/Store "stake_delegation_array" and "parents_heur2_array" from/into file:

# Store "stake_delegation_array" into file:
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/stakeDelegationArray__Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file_2D(stake_delegation_array, output_filename)

# Load stake_delegation_array from file:
file_name = BASE_ADDRESS + '/stakeDelegationArray__Heuristic2__Cardano_TXs_All__2023-03-26_043620'
stake_delegation_array = load_file_to_array_2D(file_name)



# Store "parents_heur2_array" into file:
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/parentsList_Heuristic2__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)
store_array_to_file(parents_heur2_array, output_filename)


# Load parents_heur2_array from file:
file_name = BASE_ADDRESS + '/parentsList_Heuristic2__Cardano_TXs_All__2023-03-26_105842.txt'
parents_heur2_array = load_file_to_array (file_name)


```


***


# Create and Fill "Parents_Merged Arrays"

This cell combines multiple `parents` arrays generated from various heuristics into a unified `parents_merged` array. Depending on the requirements, the user can merge arrays from **Heuristic 1**, **Heuristic 2**, or both.


#### Process

1. **Initialization**:
   - A `parents_merged` array is initialized to track the merged parent relationships.
   - Users can select one of three configurations for merging:
     - **Heuristic 1 Only**: Merge `parents_1`, `parents_2`, ..., `parents_6`.
     - **Heuristic 2 Only**: Use `parents_heur2_array`.
     - **Heuristic 1 and 2**: Combine `parents_1`, `parents_2`, ..., `parents_6` with `parents_heur2_array`.

2. **Options for Merging**:
   - **Heuristic 1 Only**:
     - Resolve individual `parents` arrays (`parents_1` to `parents_6`).
     - Merge them into `parents_merged` using `merge_parents`.
   - **Heuristic 2 Only**:
     - Resolve `parents_heur2_array` and merge it into `parents_merged`.
   - **Heuristic 1 and 2**:
     - Load both `parents_heur1_array` and `parents_heur2_array` from files.
     - Resolve and merge them into `parents_merged`.

3. **Resolving Parents**:
   - After merging, resolve the `parents_merged` array using `resolveAll` to finalize the parent relationships.


#### Output
- **`parents_merged` Array**:
  - Contains the final resolved parent relationships combining the selected heuristics.


#### Code

```python
# Create and Fill "Parents_Merged Arrays":

##########################################################################################
# Initialize parents_merged array:
parents_merged = np.array([[0]] * unique_payment_addresses_len)
for i in range(unique_payment_addresses_len):
    parents_merged[i] = i

##########################################################################################
# parents_merged = "parents_1 + parents_2 + parents_3 + parents_4 + parents_5 + parents_6":

resolveAll (parents_1)
resolveAll (parents_2)
resolveAll (parents_3)
resolveAll (parents_4)
resolveAll (parents_5)
resolveAll (parents_6)

merge_parents(parents_1, parents_merged)
merge_parents(parents_2, parents_merged)
merge_parents(parents_3, parents_merged)
merge_parents(parents_4, parents_merged)
merge_parents(parents_5, parents_merged)
merge_parents(parents_6, parents_merged)

resolveAll (parents_merged)

##########################################################################################
# parents_merged = "parents_heur2_array":
'''
# Load parents_merged from file:
file_name = BASE_ADDRESS + '/parentsList_Heuristic2__Cardano_TXs_All__2023-03-26_105842.txt'
parents_heur2_array = load_file_to_array (file_name)

merge_parents(parents_heur2_array, parents_merged)
resolveAll (parents_merged)
'''

##########################################################################################
# parents_merged =   "parents_heur2_array":
#                  + "parents_1 + parents_2 + parents_3 + parents_4 + parents_5 + parents_6":
'''
# Load parents_merged from file:
file_name = BASE_ADDRESS + '/parentsList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223712.txt'
parents_heur1_array = load_file_to_array (file_name)

file_name = BASE_ADDRESS + '/parentsList_Heuristic2__Cardano_TXs_All__2023-03-26_105842.txt'
parents_heur2_array = load_file_to_array (file_name)

merge_parents(parents_heur1_array, parents_merged)
merge_parents(parents_heur2_array, parents_merged)

resolveAll (parents_merged)
'''

```



***


### Store `parents_merged` into File

This cell saves the `parents_merged` array, which combines parent relationships derived from Heuristic 1 and Heuristic 2, into a file for future use.


#### Process

**File Naming**:
   - The file is named based on the combined heuristics:
     - Format: `parentsList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`
     - Example: `parentsList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2024-11-22_123456`

**Saving the Array**:
   - The `store_array_to_file` function writes each entry of the `parents_merged` array to a new line in the output file.



#### Code

```python
# Store parents_merged into file:

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/parentsList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp + '.txt'
store_array_to_file(parents_merged, output_filename)

```


***


# Create and Fill "Clustering Array"

This cell generates a `clustering_array` to assign each payment address to a unique cluster based on the resolved `parents_merged` array. It also calculates the number of clusters formed.


#### Process

**Remap Cluster IDs**:
   - The `remapClusterIds` function:
     - Maps each address in `parents_merged` to a unique cluster ID.
     - Fills the `clustering_array` with these cluster IDs.
     - Returns the total number of clusters.


#### Output
- **`clustering_array`**:
  - An array mapping each payment address to its cluster ID.
- **Number of Clusters**:
  - The total count of unique clusters formed.



#### Code

```python
# Create and Fill "Clustering Array":

clustering_array = np.array([0] * unique_payment_addresses_len)
num_of_clusters = remapClusterIds(parents_merged, clustering_array)

##########################################################################################
print('Length of "clustering_array" = ', len(clustering_array))
print('Number of Clusters           = '  , len(np.unique(clustering_array)))
print('Number of Clusters           = '  , max(clustering_array) + 1)
print('Number of Clusters           = '  , num_of_clusters)

```



***


# Store/Load `clustering_array` into/from File

This cell saves the `clustering_array` to a file and provides functionality to reload it. The file contains the cluster ID for each unique payment address, representing its associated cluster.


#### File Details

1. **File Format**:
   - **Type**: Plain text file 
   - **Content**: Each line corresponds to a unique payment address, and the value on the line is the cluster ID assigned to that address.
   - **Structure**: 
     - **Single Column**: Contains integer cluster IDs.
     - Example:
       ```
       0
       1
       1
       2
       ```

2. **Number of Columns**:
   - **1 Column**: Represents the cluster ID for each payment address.

3. **Number of Rows**:
   - Equal to the number of unique payment addresses (`unique_payment_addresses_len`).

4. **Column Data Type**:
   - **Integer**: Cluster IDs are integers starting from 0.


#### Process

**Store `clustering_array`**:
   - A timestamp in the format `YYYY-MM-DD_HHMMSS` is generated to ensure file names are unique.
   - The array is saved using the `store_array_to_file` function, with each cluster ID written to a new line.
   - **File Naming Options**:
     - **Heuristic 1 Only**:
       - Format: `clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__<timestamp>`
     - **Heuristic 2 Only**:
       - Format: `clusteringArrayList_Heuristic2__Cardano_TXs_All__<timestamp>`
     - **Heuristic 1 and 2**:
       - Format: `clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

**Load `clustering_array`**:
   - The `load_file_to_array` function reads the file and reconstructs the `clustering_array` by converting each line into an integer value.



#### Code

```python
# Store/Load clustering_array into file:

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store clustering_array into file:
output_filename = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__' + curr_timestamp
print('output_filename = ', output_filename)
store_array_to_file(clustering_array, output_filename)

# Load clustering_array from file:
file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212'
clustering_array = load_file_to_array(file_name)

```



***


# Merge `graghEdges_` Arrays

This cell merges multiple graph edge arrays, each representing edges extracted from a subset of analyzed transactions, into a single unified `graphEdges_merged` array. The resulting merged array contains all identified edges in the address network graph.


#### Process

1. **Input Arrays**:
   - **`graghEdges_1` to `graghEdges_6`**:
     - Lists of adjacency lists, where each index corresponds to a subset of transactions.
     - Each entry contains a list of edges (e.g., connections to other addresses) related to the corresponding address network graph (generated accoring to a subset of trnsactions).

2. **Output Array**:
   - **`graphEdges_merged`**:
     - A unified adjacency list for all edges across all transactions.
     - Each index corresponds to a subset of transactions, containing all edges identified for that address from all input arrays.

3. **Merging Function**:
   - **`merge_graphEdges`**:
     - Extends the adjacency lists in `graphEdges_merged` with the data from each `graghEdges_array`.
     - Ensures that all edge data is retained without duplication.



#### File Format:
- **File Format**: 
  - **Type**: JSON-like or plain text structure.
  - **Structure**: Each index corresponds to a unique payment address, containing a list of edges (connections to other addresses).
  - Example:
    ```
    [
      [1, 2],       # Address 0 connected to Address 1 and Address 2
      [0, 3],       # Address 1 connected to Address 0 and Address 3
      []
    ]
    ```


#### Code

```python
# Merge graghEdges_ Arrays:

def merge_graphEdges(graghEdges_array, graghEdges_merged):
    if len(graghEdges_array) != len(graghEdges_merged):
        print('merge_graphEdges Error: -1 (Length)')
        return -1
    
    for i in tqdm(range(len(graghEdges_merged))):
        graghEdges_merged[i].extend(graghEdges_array[i])
    
    return

##########################################################################################

graphEdges_merged = [[] for _ in range(unique_payment_addresses_len)]

merge_graphEdges(graghEdges_1, graphEdges_merged)
merge_graphEdges(graghEdges_2, graphEdges_merged)
merge_graphEdges(graghEdges_3, graphEdges_merged)
merge_graphEdges(graghEdges_4, graphEdges_merged)
merge_graphEdges(graghEdges_5, graphEdges_merged)
merge_graphEdges(graghEdges_6, graphEdges_merged)

```


***



# Store `graphEdges_merged` into File

This script saves the merged graph edges array (`graphEdges_merged`) into a file for future analysis. The file contains adjacency lists representing the network of payment addresses.


#### File Format:
- **File Format**: 
  - **Type**: JSON-like or plain text structure.
  - **Structure**: Each index corresponds to a unique payment address, containing a list of edges (connections to other addresses).
  - Example:
    ```
    [
      [1, 2],       # Address 0 connected to Address 1 and Address 2
      [0, 3],       # Address 1 connected to Address 0 and Address 3
      []
    ]
    ```


#### Process

1. **File Naming**:
   - Generates a unique file name with a timestamp in the format `YYYY-MM-DD_HHMMSS`.
   - **File Example**:
     - `graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2024-11-22_123456`

2. **Saving the Array**:
   - The `store_array_to_file_2D` function writes the adjacency list to the file.
   - Each adjacency list is converted to a comma-separated string for storage.


#### Code

```python
# Store graphEdges_merged into file:

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

output_filename = BASE_ADDRESS + '/graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__' + curr_timestamp 
print('output_filename = ', output_filename)

store_array_to_file_2D(graphEdges_merged, output_filename)

##########################################################################################
print('----------------------')
print('done!')
```



