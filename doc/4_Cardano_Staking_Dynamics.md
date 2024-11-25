# 4. Cardano Staking Dynamics

This `.md` file provides an explanation of the implementation in the corresponding notebook, **"4_Cardano_Staking_Dynamics.ipynb"**. The code performs the following analyses:

- Calculate "Gini Index" of entities' balances per epoch
- Calculate "Gini Index" of entities' stake delegations per epoch
- Calculate "Gini Index" of entities rewards per epoch
- Calculate number of active delegators (Stake Addresses) per epoch
- PLOT: Entities(Heur1 and Heur2) "Wealth" distributions
- PLOT: Entities(Heur1 and Heur2) "Stake Delegation" distributions
- PLOT: Entities(Heur1 and Heur2) "Reward" distributions
- PLOT: Gini Index of entities' wealth, stake delegations, rewards in each epoch
- PLOT: Number of "Delegator/Rewardee" "Addresses/Entities" in each epoch
- PLOT: Scatter Plot "Pool Rewards" vs "Pool Delegations" for all delegation events in all epochs
- PLOT: Scatter Plot "Pool Delegation" vs "Entity Wealth" for all delegation events in all epochs
- PLOT: Scatter Plot "Number of Pools received Delegation from an Entity per epoch" vs "Entity Wealth" for all delegation events in all epochs
- PLOT: Scatter Plot "Reward per epoch" vs "Entity Wealth" for all delegation events in all epochs
- PLOT: "Entity Wealth (ADA)" vs. "Entity Size"
- PLOT: "Entity Stake Delegation (ADA)" vs. "Entity Size"
- PLOT: "Entity Stake Delegation (ADA)" vs. "Entity Wealth (ADA)"

In the following sections, each cell in the corresponding 'code' is documented with a brief description of its purpose and functionality, followed by the summarized cell code.

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

***

# Load `graphEdges_merged` from File

This script loads the `graphEdges_merged` array, which contains all edges of the address network graph. Each entry represents the adjacency list for a payment address.



#### File Details

1. **File Format**:
   - **Type**: JSON file.
   - **Content**: Each entry in the file corresponds to a unique payment address and contains a list of integers representing connected addresses (edges).
   - **Structure**:
     - Each line contains a JSON array representing the adjacency list.
     - Example:
       ```json
       [
         [1, 2, 3],   // Address 0 is connected to addresses 1, 2, and 3
         [0, 4],      // Address 1 is connected to addresses 0 and 4
         [0, 5, 6]       // Address 2 is connected to addresses 0, 5, and 6
       ]
       ```

2. **Number of Columns**:
   - **Variable Columns**: Each entry contains a list of integers, so the number of columns varies.

3. **Number of Rows**:
   - Equal to the number of unique payment addresses (`unique_payment_addresses_len`).

4. **Column Data Type**:
   - **Integers**: Represent indices of connected payment addresses.



#### Process

1. **File Path**:
   - File Example: 
     `BASE_ADDRESS + '/graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_224222.txt'`

2. **Loading the Array**:
   - The `load_file_to_array_2D` function reads the JSON file and reconstructs the adjacency lists into a Python list of lists.

3. **Completion**:
   - Logs the length of the loaded array and confirms successful loading.



#### Code

```python
# Load graphEdges_merged from file:

print('----------------------')
# "graphEdges_merged" contains all identified edges in all transactions.

graphEdges_merged = load_file_to_array_2D(BASE_ADDRESS + '/graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_224222.txt')

print('Length of "graphEdges_merged" = ', len(graphEdges_merged))

##########################################################################################
print('----------------------')
print('done!')
```


***


# Calculate `graph_weights`

This script calculates the weights of edges in the address network graph based on the frequency of clustering between two addresses. Each time two addresses are identified to be clustered together in a transaction, the weight of the edge between them is incremented by 1.



#### Process

1. **Initialization**:
   - **`graph_weights`**:
     - A list of lists initialized to store the weighted edges for each address.
     - Each entry corresponds to an address and contains tuples in the format `(node, connected_node, weight)`.

2. **Edge Weight Calculation**:
   - The `find_weights_graphEdges` function iterates through the adjacency lists in `graphEdges_merged`.
   - For each address:
     - Identifies unique connected nodes.
     - Counts the occurrences of each connection to determine the weight.
   - Appends the weighted edges to the corresponding entry in `graph_weights`.

3. **Performance Tracking**:
   - Logs the start time, tracks progress using `tqdm`, and calculates the total elapsed time for the weight calculation.

4. **Completion**:
   - Confirms successful calculation of `graph_weights`.



#### Example File Details (if stored later):
- **File Format**: JSON or CSV
- **Content**: List of weighted edges.
- **Structure**:
  - Each entry contains `(node, connected_node, weight)`:
    ```
    [
      [(0, 1, 2), (0, 2, 1)],   # Address 0 connects to 1 with weight 2, and 2 with weight 1
      [(1, 0, 2)],              # Address 1 connects back to 0 with weight 2
      [(2, 0, 1)]               # Address 2 connects back to 0 with weight 1
    ]
    ```


#### Code

```python
# Calculate graph_weights:

print('----------------------')

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

##########################################################################################
graph_weights = [[] for _ in range(unique_addresses_len)]
find_weights_graphEdges(graphEdges_merged, graph_weights)

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Calculate graph_weights): ", et)

##########################################################################################
print('----------------------')
print('done!')

```



***


# Store/Load `graph_weights` into/from File

This script stores the calculated `graph_weights` into a file and provides functionality to reload it. Each entry represents weighted edges for a payment address in the graph.



#### File Details

1. **File Format**:
   - **Type**: Plain text file (`.txt`).
   - **Content**: Each line corresponds to a unique address and contains a list of weighted edges.
   - **Structure**:
     - Each entry contains tuples in the format `(node, connected_node, weight)`:
       ```
       [(0, 1, 2), (0, 2, 1)]
       [(1, 0, 2)]
       [(2, 0, 1)]
       ```
     - Each line represents the weighted edges of the corresponding address.

2. **Number of Columns**:
   - **Variable Columns**: Each entry contains a list of tuples, so the number of columns per line varies.

3. **Number of Rows**:
   - Equal to the number of unique payment addresses (`unique_addresses_len`).

4. **Column Data Type**:
   - **Tuples**: Each tuple contains:
     - **Node ID (int)**: The current node index.
     - **Connected Node ID (int)**: The index of the connected node.
     - **Weight (int)**: The frequency of the connection.



#### Process

1. **Store `graph_weights`**:
   - Generates a timestamp for unique file naming.
   - Iterates through `graph_weights` and writes each list to a new line in the file.

2. **Load `graph_weights`**:
   - Reads the file line by line and reconstructs `graph_weights` using `ast.literal_eval`.
   - Each line is parsed to a Python list of tuples representing weighted edges.

3. **Performance Tracking**:
   - Logs the elapsed time for both storing and loading operations.

4. **Completion**:
   - Confirms the successful storage and loading of the `graph_weights`.




#### Code

```python
# Store graph_weights into file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/graphWeightsArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)

with open(output_filename, 'w') as filehandle:
    for element in graph_weights:
        filehandle.write(f'{element}\n')

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Store graph_weights into file): ", et)

##########################################################################################
print('----------------------')
print('done!')

# Load graph_weights from file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

input_filename = BASE_ADDRESS + '/graphWeightsArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_234559.txt'
print('input_filename = ', input_filename)

graph_weights = [[] for _ in range(unique_addresses_len)]

i = 0
with open(input_filename) as filehandle:
    for row in filehandle:
        graph_weights[i] = ast.literal_eval(row[:-1])
        i += 1
        if i % 1000000 == 0:
            print('One million records done!', i)

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Load graph_weights from file): ", et)

##########################################################################################
print('----------------------')
print('done!')

```



***

# Generate "Graph G Addrs Network" Using `graph_weights`

This script constructs a graph `G` using `graph_weights`, where nodes represent unique payment addresses, and weighted edges represent the frequency of clustering between two addresses.



#### Process

1. **Initialization**:
   - Creates an empty undirected graph `G` using the `networkx` library.

2. **Node and Edge Addition**:
   - Iterates through the `graph_weights` list:
     - Adds each node (`i`) to the graph.
     - Uses `add_weighted_edges_from` to add weighted edges to the graph.

3. **Graph Properties**:
   - Checks and prints key properties of the generated graph:
     - Connectivity (`nx.is_connected`).
     - Number of nodes (`G.number_of_nodes()`).
     - Number of edges (`G.number_of_edges()`).



#### Code

```python
# Generate "Graph G Addrs network" using graph_weights:

print('----------------------')

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

G = nx.Graph()

for i in tqdm(range(len(graph_weights))):
    G.add_node(i)
    G.add_weighted_edges_from(graph_weights[i])

print('----------------------')
print('Is Connected    (G) = ', nx.is_connected(G))
print('Number of Nodes (G) = ', G.number_of_nodes())
print('Number of Edges (G) = ', G.number_of_edges())

print('----------------------')
test_results = [39057080, 454231, 397965]
for i in test_results:
    print('----------------------')
    print('Degree of node ' + str(i) + ' = ', G.degree()[i])
    print('graph_weights[' + str(i) + '] = ', graph_weights[i])

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Generate Graph G Addrs Network): ", et)

##########################################################################################
print('----------------------')
print('done!')
```



***

# Store/Load Graph Object `Graph G Addrs Network` to/from File

This script provides functionality to save and load the `Graph G Addrs Network` object, which represents the address network graph, using Python's `pickle` module.


#### File Details

1. **File Format**:
   - **Type**: Binary file (`.pickle`).
   - **Content**: Serialized graph object created using the `networkx` library.

2. **Graph Object Details**:
   - **Nodes**: Represent unique payment addresses.
   - **Edges**: Represent connections between addresses with weights indicating clustering frequency.

3. **Number of Nodes and Edges**:
   - Stored as part of the `networkx` graph object.


#### Process

1. **Store Graph Object**:
   - Generates a timestamp in the format `YYYY-MM-DD_HHMMSS` for unique file naming.
   - Uses the `pickle.dump` function to serialize and save the graph object to a file.
   - Example File Name:
     - `Graph_G_AddrsNetwork_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2024-11-22_123456.pickle`

2. **Load Graph Object**:
   - Reads the file using `pickle.load` to reconstruct the graph object.
   - The graph object retains all its properties, including nodes, edges, and weights.



#### Code

```python
# Store graph object to file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

import pickle
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/Graph_G_AddrsNetwork_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__' + curr_timestamp + '.pickle'
print('output_filename = ', output_filename)
pickle.dump(G, open(output_filename, 'wb'))

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Store G into file): ", et)

##########################################################################################
print('----------------------')
print('done!')

# Load graph object from file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

import pickle
input_filename = BASE_ADDRESS + '/Graph_G_AddrsNetwork_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-26_000507.pickle'
print('input_filename = ', input_filename)
G = pickle.load(open(input_filename, 'rb'))

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Load G from file): ", et)

##########################################################################################
print('----------------------')
print('done!')

```



***


# Extract the Largest Connected Component

This script identifies and analyzes the largest connected component in the address network graph (`G`). It calculates key properties such as connectivity, node count, edge count, and weighted degree statistics.



#### Process

1. **Identify the Largest Connected Component**:
   - Extracts all connected components of the graph `G` using `nx.connected_components`.
   - Sorts the components by size in descending order and selects the largest one.
   - Creates a subgraph (`largest_cc_subgraph`) from the largest component.

2. **Connectivity and Properties**:
   - Checks if the largest connected component is fully connected using `nx.is_connected`.
   - Logs the number of nodes and edges in the subgraph.

3. **Weighted Degree Statistics**:
   - Calculates the weighted degree of each node in the subgraph using `G.degree(weight='weight')`.
   - Sorts the degrees and computes:
     - **Sum of Weighted Degrees**: Total edge weights for the subgraph.
     - **Maximum Weighted Degree**: Node with the highest degree weight.

4. **Degree Distribution**:
   - Uses `BinarySearch_Find_start_end` to calculate the count of nodes with specific weighted degrees (1 to 14).
   - Logs the counts for each weighted degree.



#### Code

```python
# Extract the largest connected component:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

# Identify the largest connected component
largest_cc = list(sorted(nx.connected_components(G), key=len, reverse=True))[0]
largest_cc_subgraph = G.subgraph(largest_cc).copy()

# Log connectivity and properties
print('Is Connected    (largest_cc_subgraph) = ', nx.is_connected(largest_cc_subgraph))
print('Number of Nodes (largest_cc_subgraph) = ', largest_cc_subgraph.number_of_nodes())
print('Number of Edges (largest_cc_subgraph) = ', largest_cc_subgraph.number_of_edges())

# Weighted degree statistics
print('----------------------')
degree_sequence = sorted((d for n, d in largest_cc_subgraph.degree(weight='weight')), reverse=False)
print('Sum Weighted Degrees (largest_cc_subgraph) = ', sum(degree_sequence))
print('Max Weighted Degrees (largest_cc_subgraph) = ', max(degree_sequence))

# Degree distribution
print('----------------------')
for i in range(1, 15):
    x = BinarySearch_Find_start_end(degree_sequence, i)
    print('Number of nodes with Weighted Degree "' + str(i) + '" = ', x[1] - x[0] + 1)

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Extract largest connected component): ", et)

##########################################################################################
print('----------------------')
print('done!')
```


***


# Store/Load Graph Object `largest_cc_subgraph` to/from File

This script provides functionality to save and load the `largest_cc_subgraph`, which represents the largest connected component of the address network graph, using Python's `pickle` module.


#### File Details

1. **File Format**:
   - **Type**: Binary file (`.pickle`).
   - **Content**: Serialized graph object created using the `networkx` library.
   - **Graph Properties**:
     - **Nodes**: Nodes in the largest connected component.
     - **Edges**: Connections between these nodes with associated weights.

2. **Graph Size**:
   - Stored as part of the `networkx` graph object, including node and edge counts.


#### Process

1. **Store Graph Object**:
   - Generates a timestamp in the format `YYYY-MM-DD_HHMMSS` for unique file naming.
   - Uses `pickle.dump` to serialize and save the `largest_cc_subgraph` to a file.
   - Example File Name:
     - `Largest1_cc_subgraph_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2024-11-22_123456.pickle`

2. **Load Graph Object**:
   - Reads the file using `pickle.load` to reconstruct the `largest_cc_subgraph`.
   - The graph retains all properties, including nodes, edges, and weights.




#### Code

```python
# Store graph object to file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

import pickle
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]
output_filename = BASE_ADDRESS + '/Largest1_cc_subgraph_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__' + curr_timestamp + '.pickle'
print('output_filename = ', output_filename)
pickle.dump(largest_cc_subgraph, open(output_filename, 'wb'))

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Store largest_cc_subgraph into file): ", et)

##########################################################################################
print('----------------------')
print('done!')

# Load graph object from file:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

import pickle
input_filename = BASE_ADDRESS + '/Largest2_cc_subgraph_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-01-10_181953.pickle'
print('input_filename = ', input_filename)
largest_cc_subgraph = pickle.load(open(input_filename, 'rb'))

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Load largest_cc_subgraph into file): ", et)

##########################################################################################
print('----------------------')
print('done!')

```


***

# Perform "Label Propagation" on `largest_cc_subgraph`

This script applies a Label Propagation algorithm on the largest connected component's copy graph (`largest_cc_subgraph_COPY`). Label Propagation is an iterative algorithm for community detection in graphs.


#### Process

1. **Initialization**:
   - Sets up arguments for the Label Propagation algorithm:
     - **`rounds`**: Number of iterations (default: `8`).
     - **`seed`**: Random seed for reproducibility (default: `42`).

2. **Label Propagation**:
   - Initializes a `LabelPropagator` model using the `largest_cc_subgraph_COPY` graph and the arguments.
   - Executes a series of label propagation iterations using `do_a_series_of_propagations`.
   - Produces a dictionary (`NodeLabelsDict`) mapping each node to its detected label.

3. **Sorting Labels**:
   - Optionally sorts the `NodeLabelsDict`:
     - By **key**: Node IDs (default).
     - By **value**: Label assignments.

4. **Performance Tracking**:
   - Logs the start time, elapsed time, and confirms the completion of label propagation.




#### Code

```python
# Perform "Label Propagation" on the largest_cc_subgraph_COPY:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)

if __name__ == "__main__":
    """
    --rounds      INT    Number of iterations    Default is 8.
    --seed        INT    Initial seed            Default is 42.
    """
    args = {'rounds': 8, 'seed': 42}
    model = LabelPropagator(largest_cc_subgraph_COPY, args)
    NodeLabelsDict = model.do_a_series_of_propagations()
    # NodeLabelsDict = dict(sorted(NodeLabelsDict.items()))                          # sorted by key
    NodeLabelsDict = dict(sorted(NodeLabelsDict.items(), key=lambda item: item[1])) # sorted by value

##########################################################################################
print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Label Propagation): ", et)

##########################################################################################
print('----------------------')
print('done!')
```



***



# Read Address Clustering Results from Files

This script reads clustering results obtained from different heuristics and combinations of heuristics, stored in files, and loads them into corresponding arrays.


**Clustering Results**:
   - **Heuristic 1**: Clustering results based on Heuristic 1 (excluding smart contracts).
   - **Heuristic 2**: Clustering results based on Heuristic 2.
   - **Heuristic 1 + 2**: Combined clustering results based on both Heuristics 1 and 2.


#### Process

1. **File Paths**:
   - **Heuristic 1**:
     - `BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223957.txt'`
   - **Heuristic 2**:
     - `BASE_ADDRESS + '/clusteringArrayList_Heuristic2__Cardano_TXs_All__2023-03-26_110150.txt'`
   - **Heuristic 1 + 2**:
     - `BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212.txt'`

2. **Loading the Files**:
   - Uses `load_file_to_array` to read each file into an array.



#### Code

```python
# Read Address Clustering Results from Files:

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223957.txt'
clustering_array_heur1 = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic2__Cardano_TXs_All__2023-03-26_110150.txt'
clustering_array_heur2 = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212.txt'
clustering_array_heur1and2 = load_file_to_array(file_name)

##########################################################################################
print('----------------------')
print('done!')
```




***

# Calculate "Gini Index" of Entities (Balances / Stake Delegation / Reward)

This script calculates the Gini Index for entities' balances over a range of epochs. The Gini Index is a measure of inequality, where a value of `0` represents perfect equality and `1` represents maximum inequality.



#### Process

1. **Initialization**:
   - **Epoch Range**:
     - `first_epoch_no = 0`
     - `last_epoch_no = 391`
   - **Gini Index Storage**:
     - `gini_index_entity_Balances`: A list initialized to store the Gini Index for each epoch.

2. **Load Balances**:
   - Iterates through files containing entity balances in 5-day intervals (`range(0, 1946, 5)`).
   - Loads balances for entities from the file corresponding to the current day using `load_file_to_array`.

3. **Preprocessing Balances**:
   - Filters out entities with zero balances.
   - Converts balances to floating-point numbers and normalizes them by dividing by \(10^6\) [Lovelace to ADA conversion].



**File Names**:
   ```
   /YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_0000__Cardano_TXs_All.txt
   /YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_0005__Cardano_TXs_All.txt
   ...
   ```


#### Code

```python
# Calculate "Gini Index" of Entities (Balances):

print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

first_epoch_no = 0
last_epoch_no = 391
total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)

gini_index_entity_Balances = [0] * total_num_of_epochs

for i in tqdm(range(0, 1945 + 1, 5)):
    # Load "entity_balances" from file:
    file_name = BASE_ADDRESS + '/YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_' + str(i).zfill(4) + '__Cardano_TXs_All.txt'
    entity_balances = load_file_to_array(file_name)
    entity_balances = entity_balances[entity_balances != 0]
    entity_balances = np.array(entity_balances, dtype=np.float64)
    entity_balances = entity_balances / (10**6)

    epoch_no = int(i / 5)
    if epoch_no < len(gini_index_entity_Balances):
        gini_index_entity_Balances[epoch_no] = Gini_rank(entity_balances)

##########################################################################################
print('----------------------')
print('done!')
```



***


# Store `gini_index_entity_Balances` into File

This script stores the calculated Gini Index values for entity balances over epochs into a text file for future analysis.

1. **File Format**:
   - **Type**: Plain text file (`.txt`).
   - **Content**: Each line corresponds to the Gini Index value for a specific epoch.
 

2. **File Name**:
   - Generated dynamically with a timestamp to ensure uniqueness:
     ```
     Entities_GiniIndex_Balances_perEpoch__YYYY-MM-DD_HHMMSS__Cardano_TXs_All.txt
     ```


#### Process

1. **File Name Generation**:
   - Combines the base address, timestamp (`curr_timestamp`), and file name structure.

2. **Write to File**:
   - Uses `store_array_to_file` to save the `gini_index_entity_Balances` list to the specified file.

3. **Completion**:
   - Confirms the file's creation and logs the file name.



#### Code

```python
# Store "gini_index_entity_Balances" into file:

output_filename = BASE_ADDRESS + '/Entities_GiniIndex_Balances_perEpoch__' + curr_timestamp + '__Cardano_TXs_All.txt'
print('output_filename = ', output_filename)
store_array_to_file(gini_index_entity_Balances, output_filename)
```


***

# Calculate "Pool Delegation vs Entity Wealth" and "Number of Pools vs Entity Wealth" for All Delegation Events Across All Epochs

This script calculates the relationship between **entity wealth** and:
1. **Stake Delegation**: Pool stake delegated by entities during delegation events.
2. **Number of Pools**: The number of pools an entity delegated to per epoch.



#### Process

1. **Initialization**:
   - Defines the epoch range for pool staking:
     - Start: Epoch `210` (2020-08-08).
     - End: Epoch `391` (2023-01-30).
   - Initializes arrays and dictionaries to store relationships and data for entities, pools, and rewards.

2. **Epoch Loop**:
   - For each epoch:
     - Loads **entity balances** for the current day.
     - Tracks delegation events:
       - Maps **entity wealth** to **pool stake**.
       - Tracks the **number of pools** delegated by entities.
     - Tracks reward events:
       - Maps **entity wealth** to **reward amounts** received.
   - Updates the current epoch and resets intermediate variables for the next epoch.

3. **Delegation and Reward Mapping**:
   - **Delegators**:
     - Extracts entity indices for delegators.
     - Maps entity wealth to the pool stakes they contribute to.
     - Tracks pools an entity delegates to during the epoch.
   - **Rewarders**:
     - Maps entity wealth to rewards received.

4. **Results**:
   - Stores relationships between:
     - Entity wealth and pool stake delegation.
     - Entity wealth and number of pools delegated to.
     - Entity wealth and rewards received.


#### Output

1. **Delegation Events**:
   - `entityWealth_to_poolStake_delegationEvents_ALL`: Combined wealth-to-stake mappings across epochs.
   - `entityWealth_to_poolStake_delegationEvents_perEpoch`: Wealth-to-stake mappings per epoch.
   - `entity_to_numOf_pool_deleg_AllEvents`: Number of pools an entity delegated to and their wealth.

2. **Reward Events**:
   - `rewarded_entities_AllEvents`: Wealth-to-reward mappings across epochs.
   - `rewarded_entities_perEpoch`: Wealth-to-reward mappings per epoch.

3. **Performance**:
   - Logs the total elapsed time for computation.


#### Notes
- **Input Files**:
  - Pool data: `/cardano_pools_4.csv`
  - Entity balances: `/YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_XXXX__Cardano_TXs_All.txt` (where `XXXX` is zero-padded to represent days).
- Ensure that functions like `BinarySearch` and file loaders are implemented and optimized.
- The script assumes unique mappings between delegation/reward addresses and entities.




#### Code

```python
from collections import defaultdict

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)
epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(len(epochs_array))]

file_name = BASE_ADDRESS + '/cardano_pools_4.csv'
df = pd.read_csv(file_name, delimiter='|')

clustering_array = clustering_array_heur1and2
current_epoch = first_epoch_no
current_day = current_epoch * 5

file_name = BASE_ADDRESS + '/YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_' + str(current_day).zfill(4) + '__Cardano_TXs_All.txt'
entities_wealth_per_epoch = load_file_to_array(file_name)

entityWealth_to_poolStake_delegationEvents_ALL = []
entityWealth_to_poolStake_delegationEvents_perEpoch = []

entity_to_numOf_pool_deleg_AllEvents = []
entity_to_pool_deleg_perEpoch = []

rewarded_entities_perEpoch = []
rewarded_entities_AllEvents = []

for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    POOL_ID = int(df.loc[index, 'POOL_ID'])
    POOL_STAKES = int(df.loc[index, 'POOL_STAKES'])
    DELEGATORs = list(df.loc[index, 'DELEGATORs'].split(';'))
    REWARDERs = list(df.loc[index, 'REWARDERs'].split(';')) if not pd.isna(df.loc[index, 'REWARDERs']) else []

    if EPOCH < first_epoch_no:
        continue

    if EPOCH > current_epoch:
        entityWealth_to_poolStake_delegationEvents_ALL.extend(entityWealth_to_poolStake_delegationEvents_perEpoch)
        entityWealth_to_poolStake_delegationEvents_perEpoch = []

        entity_pools = defaultdict(set)
        for entity_indx, pool_id in entity_to_pool_deleg_perEpoch:
            entity_pools[entity_indx].add(pool_id)
        entity_to_numOf_pool_deleg_AllEvents.extend([(entities_wealth_per_epoch[entity_indx][0], len(pools)) for entity_indx, pools in entity_pools.items()])
        entity_to_pool_deleg_perEpoch = []

        entity_rewards = defaultdict(set)
        for entity_indx, reward_amount in rewarded_entities_perEpoch:
            entity_rewards[entity_indx].add(reward_amount)
        rewarded_entities_AllEvents.extend([(entities_wealth_per_epoch[entity_indx][0], sum(reward_amounts)) for entity_indx, reward_amounts in entity_rewards.items()])
        rewarded_entities_perEpoch = []

        current_epoch = EPOCH
        current_day = current_epoch * 5
        if current_day <= 1945:
            file_name = BASE_ADDRESS + '/YuZhang_Cardano_Balances_Entities/BalancesPerEntityDay_' + str(current_day).zfill(4) + '__Cardano_TXs_All.txt'
            entities_wealth_per_epoch = load_file_to_array(file_name)

    delegator_entities_to_this_pool_per_epoch_SET = set()
    for delegator in DELEGATORs:
        deleg_stake_addr = delegator.split(',')[2]
        deleg_addr_indx = BinarySearch(unique_delegation_addresses, deleg_stake_addr, debug=False)
        if deleg_addr_indx != -1:
            deleg_entity_indx = entity_of_stake_addresses[deleg_addr_indx][0]
            delegator_entities_to_this_pool_per_epoch_SET.add(deleg_entity_indx)

    for entity_indx in delegator_entities_to_this_pool_per_epoch_SET:
        if entity_indx < len(entities_wealth_per_epoch):
            entityWealth_to_poolStake_delegationEvents_perEpoch.append((entities_wealth_per_epoch[entity_indx][0], POOL_STAKES))
            entity_to_pool_deleg_perEpoch.append((entity_indx, POOL_ID))

    for rewarder in REWARDERs:
        reward_stake_addr = rewarder.split(',')[2]
        reward_addr_indx = BinarySearch(unique_delegation_addresses, reward_stake_addr, debug=False)
        if reward_addr_indx != -1:
            reward_entity_indx = entity_of_stake_addresses[reward_addr_indx][0]
            if reward_entity_indx < len(entities_wealth_per_epoch):
                rewarded_entities_perEpoch.append((reward_entity_indx, int(rewarder.split(',')[1])))

et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')
```




***



# Store/Load "entityWealth_to_poolStake_delegationEvents_ALL" and "entity_to_numOf_pool_deleg_AllEvents"

This script handles the storage and loading of three datasets:
1. **`entityWealth_to_poolStake_delegationEvents_ALL`**: Mapping of entity wealth to pool stakes across all epochs.
2. **`entity_to_numOf_pool_deleg_AllEvents`**: Number of pools an entity delegated to and their wealth.
3. **`rewarded_entities_AllEvents`**: Mapping of entity wealth to rewards received across all epochs.


**File Format**:
   - **Type**: Serialized binary file (`.txt`).
   - **Content**: Pickled Python objects.

**File Naming Convention**:
   - Includes a timestamp for uniqueness:
     ```
     DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__YYYY-MM-DD_HHMMSS.txt
     DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__YYYY-MM-DD_HHMMSS.txt
     DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__YYYY-MM-DD_HHMMSS.txt
     ```


#### Process

1. **Store Files**:
   - Serializes and saves the data structures using Python's `pickle.dump`.
   - Logs the file paths for confirmation.

2. **Load Files**:
   - Reads the serialized files using `pickle.load`.
   - Reconstructs the data structures for further analysis.

3. **Performance Tracking**:
   - Logs the start and completion of the operations.

**Stored Files**:
   ```
   /DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__2024-11-22_163024.txt
   /DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__2024-11-22_163024.txt
   /DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__2024-11-22_163024.txt
   ```


#### Code

```python
import pickle
print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store file:
output_filename = BASE_ADDRESS + '/DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
with open(output_filename, 'wb') as file:
    pickle.dump(entityWealth_to_poolStake_delegationEvents_ALL, file)

output_filename = BASE_ADDRESS + '/DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
with open(output_filename, 'wb') as file:
    pickle.dump(entity_to_numOf_pool_deleg_AllEvents, file)

output_filename = BASE_ADDRESS + '/DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__' + curr_timestamp + '.txt'
print('output_filename = ', output_filename)
with open(output_filename, 'wb') as file:
    pickle.dump(rewarded_entities_AllEvents, file)

# Load file:
filename = BASE_ADDRESS + '/DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__2023-12-18_043529.txt'
with open(filename, 'rb') as file:
    entityWealth_to_poolStake_delegationEvents_ALL = pickle.load(file)

filename = BASE_ADDRESS + '/DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__2023-12-18_043529.txt'
with open(filename, 'rb') as file:
    entity_to_numOf_pool_deleg_AllEvents = pickle.load(file)

filename = BASE_ADDRESS + '/DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__2023-12-18_142414.txt'
with open(filename, 'rb') as file:
    rewarded_entities_AllEvents = pickle.load(file)

##########################################################################################
print('----------------------')
print('done!')
```


***

# Calculate "Active Delegators (Stake Addresses)" per Epoch

This script calculates the total number of active delegators (stake addresses) per epoch based on delegation data.


#### Process

1. **Initialization**:
   - Defines the epoch range for pool staking:
     - Start: Epoch `210` (2020-08-08).
     - End: Epoch `391` (2023-01-30).
   - Creates arrays for:
     - Epoch numbers (`epochs_array`).
     - Epoch start dates (`epochs_date_array`).
     - Active delegator count per epoch (`active_delegator_addresses_per_epoch`).

2. **Read Delegation Data**:
   - Reads delegation data from `cardano_pools_3.csv`.
   - Iterates through rows of the dataset to extract delegation information.

3. **Aggregate Delegators**:
   - For each epoch:
     - Accumulates the number of delegators (`NUM_OF_DELEGATORS`) for all pools.



#### Input File

- **File Path**:
  ```
  BASE_ADDRESS + '/cardano_pools_3.csv'
  ```
- **File Format**:
  - **Type**: CSV.
  - **Columns**:
    1. **EPOCH** (int): Epoch number.
    2. **POOL_HASH_BECH32** (string): Unique identifier for the pool.
    3. **POOL_STAKES** (int): Total stake delegated to the pool.
    4. **POOL_REWARDS** (int): Rewards distributed by the pool.
    5. **NUM_OF_DELEGATORS** (int): Number of delegators in the epoch.
    6. **NUM_OF_REWARDERS** (int): Number of reward recipients in the epoch.



#### Code

```python
# Calculate "Active Delegators (Stake Addresses)" per epoch:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)
epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(len(epochs_array))]

active_delegator_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]

current_epoch = first_epoch_no

file_name = BASE_ADDRESS + '/cardano_pools_3.csv'
df = pd.read_csv(file_name, delimiter=',')

for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    NUM_OF_DELEGATORS = int(df.loc[index, 'NUM_OF_DELEGATORS'])

    if EPOCH < first_epoch_no:
        continue
    if EPOCH > last_epoch_no:
        break

    active_delegator_addresses_per_epoch[EPOCH - first_epoch_no] += NUM_OF_DELEGATORS

et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')
```


***


# Calculate "Active Delegators (Entities)" per Epoch

This script calculates the number of active delegators (stake entities) and active rewarders (reward entities) per epoch.


#### Process

1. **Initialization**:
   - Defines the epoch range for pool staking:
     - Start: Epoch `210` (2020-08-08).
     - End: Epoch `391` (2023-01-30).
   - Initializes arrays for:
     - Number of unique delegator/rewarder addresses and entities per epoch.

2. **Iterate Through Pool Data**:
   - Reads data from `cardano_pools_4.csv`.
   - For each row in the data:
     - Tracks delegator and rewarder addresses and maps them to entities using clustering information.
     - Accumulates delegation amounts for each entity.

3. **Per-Epoch Data Handling**:
   - When transitioning to a new epoch:
     - Saves the current counts of delegator/rewarder addresses and entities.
     - Stores entity-level delegation amounts into a file for the completed epoch.
     - Resets tracking variables for the next epoch.

4. **Entity Mapping**:
   - Uses clustering information (`clustering_array`) to map stake addresses to entities.
   - Delegations from unknown addresses are treated as individual entities.

5. **Output**:
   - Stores delegation amounts for each entity in a file named:
     ```
     StakeDelegPerEntityEpoch_XXXX__Cardano_TXs_All.txt
     ```

6. **Performance Tracking**:
   - Logs the total elapsed time for computation.


#### Input File

- **File Path**:
  ```
  BASE_ADDRESS + '/cardano_pools_4.csv'
  ```
- **File Format**:
  - **Type**: CSV with `|` as a delimiter.
  - **Columns**:
    1. **EPOCH** (int): Epoch number.
    2. **POOL_ID** (int): Pool identifier.
    3. **POOL_HASH_BECH32** (string): Unique identifier for the pool.
    4. **POOL_STAKES** (int): Total stake delegated to the pool.
    5. **POOL_REWARDS** (int): Rewards distributed by the pool.
    6. **NUM_OF_DELEGATORS** (int): Number of delegators in the epoch.
    7. **NUM_OF_REWARDERS** (int): Number of reward recipients in the epoch.
    8. **DELEGATORs** (string): Semicolon-separated list of delegators.
    9. **REWARDERs** (string): Semicolon-separated list of rewarders.





#### Code

```python
# "Active Delegators (Entities)" per epoch:

print('----------------------')
# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')

first_epoch_no = 210
last_epoch_no = 391
FIRST_DATE_POOLS_STAKING = datetime.datetime.strptime('2020-08-08 21:44:51', '%Y-%m-%d %H:%M:%S').date()
LAST_DATE_POOLS_STAKING = datetime.datetime.strptime('2023-01-30 21:46:16', '%Y-%m-%d %H:%M:%S').date()

total_num_of_epochs = int(last_epoch_no - first_epoch_no + 1)
epochs_array = list(range(first_epoch_no, last_epoch_no + 1))
epochs_date_array = [FIRST_DATE_POOLS_STAKING + datetime.timedelta(days=(i * 5)) for i in range(len(epochs_array))]

num_delegator_addresses_per_epoch = [0 for _ in range(total_num_of_epochs)]
num_delegator_entities_per_epoch = [0 for _ in range(total_num_of_epochs)]

delegator_addresses_per_epoch_SET = set()
delegator_entities_per_epoch_SET = set()

clustering_array = clustering_array_heur1and2
stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)

file_name = BASE_ADDRESS + '/cardano_pools_4.csv'
df = pd.read_csv(file_name, delimiter='|')

current_epoch = first_epoch_no

for index, row in tqdm(df.iterrows()):
    EPOCH = int(df.loc[index, 'EPOCH'])
    DELEGATORs = list(df.loc[index, 'DELEGATORs'].split(';'))

    if EPOCH < first_epoch_no:
        continue
    if EPOCH > current_epoch:
        num_delegator_addresses_per_epoch[current_epoch - first_epoch_no] = len(delegator_addresses_per_epoch_SET)
        num_delegator_entities_per_epoch[current_epoch - first_epoch_no] = len(delegator_entities_per_epoch_SET)

        delegator_addresses_per_epoch_SET.clear()
        delegator_entities_per_epoch_SET.clear()

        output_filename = BASE_ADDRESS + '/YuZhang_Cardano_StakeDelegation_Entities/StakeDelegPerEntityEpoch_' + str(current_epoch).zfill(4) + '__Cardano_TXs_All.txt'
        store_array_to_file(stake_deleg_by_entities, output_filename)
        stake_deleg_by_entities = [0] * (np.amax(clustering_array) + 1)

        current_epoch = EPOCH
    if EPOCH > last_epoch_no:
        break

    for delegator in DELEGATORs:
        deleg_stake_addr = delegator.split(',')[2]
        deleg_addr_indx = BinarySearch(unique_delegation_addresses, deleg_stake_addr, debug=False)
        if deleg_addr_indx != -1:
            deleg_entity_indx = entity_of_stake_addresses[deleg_addr_indx]
            stake_deleg_by_entities[deleg_entity_indx] += int(delegator.split(',')[1])
            delegator_addresses_per_epoch_SET.add(deleg_addr_indx)
            delegator_entities_per_epoch_SET.add(deleg_entity_indx)
        else:
            delegator_addresses_per_epoch_SET.add(deleg_stake_addr)
            delegator_entities_per_epoch_SET.add(deleg_stake_addr)

et = datetime.datetime.now() - ct
print("Total elapsed time: ", et)
print('----------------------')
print('done!')
```


***


# Load/Store "Active Delegators and Rewarders" Counts per Epoch

This script handles the storage and retrieval of four arrays tracking the number of active delegators and rewarders (both addresses and entities) for each epoch.


#### Files and Formats

1. **File Names**:
   - The file names include a timestamp to ensure uniqueness.
   - Examples:
     ```
     /Num_Delegator_addresses_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt
     /Num_Delegator_entities_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt
     /Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt
     /Num_Rewarder_entities_per_epoch__Cardano_TXs_All__YYYY-MM-DD_HHMMSS.txt
     ```

2. **File Format**:
   - **Type**: Plain text file (`.txt`).
   - **Structure**: One integer per line, representing the count for each epoch.
   - **Length**: Equal to the number of epochs (`last_epoch_no - first_epoch_no + 1`).

3. **Details**:
   - **Number of Columns**: 1.
   - **Column Type**: Integer.
   - **Example** (for a single file):
     ```
     1000
     1200
     1100
     ...
     ```
     - Line 1: Count for Epoch 210.
     - Line 2: Count for Epoch 211.
     - Line 3: Count for Epoch 212.


#### Arrays

- **Arrays to Store/Load**:
  1. `num_delegator_addresses_per_epoch`: Number of unique delegator addresses per epoch.
  2. `num_delegator_entities_per_epoch`: Number of unique delegator entities per epoch.
  3. `num_rewarder_addresses_per_epoch`: Number of unique rewarder addresses per epoch.
  4. `num_rewarder_entities_per_epoch`: Number of unique rewarder entities per epoch.




#### Code

```python
print('----------------------')
ct = datetime.datetime.now()
curr_timestamp = str(ct)[0:10] + '_' + str(ct)[11:13] + str(ct)[14:16] + str(ct)[17:19]

# Store arrays into files:
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

# Load arrays from files:
file_name = BASE_ADDRESS + '/Num_Delegator_addresses_per_epoch__Cardano_TXs_All__2024-11-22_123456.txt'
num_delegator_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Delegator_entities_per_epoch__Cardano_TXs_All__2024-11-22_123456.txt'
num_delegator_entities_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__2024-11-22_123456.txt'
num_rewarder_addresses_per_epoch = load_file_to_array(file_name)

file_name = BASE_ADDRESS + '/Num_Rewarder_entities_per_epoch__Cardano_TXs_All__2024-11-22_123456.txt'
num_rewarder_entities_per_epoch = load_file_to_array(file_name)

print('----------------------')
print('done!')
```



***


# NFTs and FTs Minted by each entity:


This cell extracts the NFTs and FTs minted by each entity based on clustering results. Each entity is identified using a clustering algorithm applied to Cardano transactions data. The code processes input data from multiple CSV files and associates NFTs and FTs to entities based on the inputs of the minting transactions.


#### Arrays:
1. **NFTs_minted_per_entity_array**:
   - Tracks NFTs minted by each entity.
   - Format: 2D List where each row represents an entity, and the values in the row are the `MA_ID`s of the NFTs minted by that entity.

2. **FTs_minted_per_entity_array**:
   - Tracks FTs minted by each entity.
   - Format: 2D List where each row represents an entity, and the values in the row are the `MA_ID`s of the FTs minted by that entity.



### Code

```python
print('----------------------')

# ct stores current time
ct = datetime.datetime.now()
print("current time: ", ct)
print('----------------------')


# = "clustering_array_heur1"  OR  "clustering_array_heur2"  OR  "clustering_array_heur1and2"
clustering_array = clustering_array_heur1and2


# Initialize "NFTs_minted_per_entity_array" and "FTs_minted_per_entity_array" (For every "payment_address", this array shows which "MA_ID"s are minted by that address):
NFTs_minted_per_entity_array = [[] for _ in range( np.amax(clustering_array)+1 )]
FTs_minted_per_entity_array  = [[] for _ in range( np.amax(clustering_array)+1 )]

NFTs_minted_by_smart_contract = 0
FTs_minted_by_smart_contract  = 0




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
        ##########################################################################################
        TX_ID      = df.loc[index , 'TX_ID']
        BLOCK_TIME = str(datetime.datetime.strptime(str(df.loc[index , 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date())
        EPOCH_NO   = str(  df.loc[index , 'EPOCH_NO'] )
        ##########################################################################################
        inputs_list = list( df.loc[index , 'INPUTs'].split(';') )
        ##########################################################################################
        NFTs_list  = list( df.loc[index , 'MINT_NFTs'].split(';') )
        for NFT in NFTs_list:
            MA_ID = NFT.split(',')[0]
            if(MA_ID != ''):
                flag = 0
                for tx_input in inputs_list:
                    address_raw           = tx_input.split(',')[4]
                    address_has_script    = tx_input.split(',')[7]
                    payment_cred          = tx_input.split(',')[8]
                    stake_address         = tx_input.split(',')[9]
                    if(address_has_script == 'f'):  # non-Smart Contract Address 
                        [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                        if (address_payment_part != ''):
                            addr_indx   = BinarySearch(unique_payment_addresses, address_payment_part)
                            entity_indx = clustering_array[addr_indx][0]
                            NFTs_minted_per_entity_array[entity_indx].append(MA_ID)
                            flag = flag + 1
                            break; # we need to find only one payment address, so break after finding it
                if(flag == 0):
                    NFTs_minted_by_smart_contract = NFTs_minted_by_smart_contract + 1
                    if(NFTs_minted_by_smart_contract < 100):
                        print("NFT: no non-smart contract input address was found for MA_ID = ", MA_ID, '|TX_ID = ', TX_ID)
        ##########################################################################################
        FTs_list   = list( df.loc[index , 'MINT_FTs'].split(';') )
        for FT in FTs_list:
            MA_ID = FT.split(',')[0]
            if(MA_ID != ''):
                flag = 0
                for tx_input in inputs_list:
                    address_raw           = tx_input.split(',')[4]
                    address_has_script    = tx_input.split(',')[7]
                    payment_cred          = tx_input.split(',')[8]
                    stake_address         = tx_input.split(',')[9]
                    if(address_has_script == 'f'):  # non-Smart Contract Address 
                        [address_payment_part, address_delegation_part] = extract_payment_delegation_parts(address_raw, payment_cred, stake_address)
                        if (address_payment_part != ''):
                            addr_indx   = BinarySearch(unique_payment_addresses, address_payment_part)
                            entity_indx = clustering_array[addr_indx][0]
                            FTs_minted_per_entity_array[entity_indx].append(MA_ID)
                            flag = flag + 1
                            break; # we need to find only one payment address, so break after finding it
                if(flag == 0):
                    FTs_minted_by_smart_contract = FTs_minted_by_smart_contract + 1
                    if(FTs_minted_by_smart_contract < 100):
                        print(" FT: no non-smart contract input address was found for MA_ID = ", MA_ID, '|TX_ID = ', TX_ID)
        ##########################################################################################

    et_temp = datetime.datetime.now() - ct_temp
    print("elapsed time (Extract stake delegations from CSV File " + file_name + "): ", et_temp)



print('----------------------')
et = datetime.datetime.now() - ct
print("Total elapsed time (Heuristic2: find \"Shelley Addresses\" with the same \"address_delegation_part\"): ", et)


print('----------------------')
print('done!')
```



***




