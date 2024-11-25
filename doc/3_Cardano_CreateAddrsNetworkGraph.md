# 3. Cardano Create "Address Network" Graph


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


# Define Required Methods

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









