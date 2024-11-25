

***

#### File: `Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__2024-01-23_212107`

This file contains the mapping of stake (delegation) addresses to the entities they are associated with, based on **Heuristic 2**. The mapping identifies the entity (cluster) of payment addresses linked to each stake address through transaction outputs. If a stake address does not have an associated entity, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique stake (delegation) address.
  - Each value indicates the entity ID associated with the address or the placeholder `999999999999` if no entity is associated.
- **Number of Elements**: Equal to the total number of unique delegation addresses.

### Example
```plaintext
5
12
999999999999
7
18
...
```
- **Example Interpretation**:
  - Stake address 0 is linked to entity 5.
  - Stake address 1 is linked to entity 12.
  - Stake address 2 has no associated entity, as indicated by the placeholder value.
  - Stake address 3 is linked to entity 7, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the `entity_of_stake_addresses` array to this text file. This method writes the array in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into memory. This method reads the text file and converts it into a NumPy array for further analysis.


- **Generation Method**:
  - The mapping was constructed by iterating over transaction outputs across multiple CSV files (`cardano_TXs_Velocity_*`) and linking stake addresses to entities of payment addresses when both address parts were present.
  - The first observed link between a stake address and an entity was recorded.
  - A placeholder value (`999999999999`) was assigned to stake addresses with no associated entities.

***

#### File: `epochArray_rawAddresses__Cardano_TXs_All__2023-03-27_085508`

This file contains the epoch numbers when each raw address first appeared on the Cardano blockchain. Each entry corresponds to a unique raw address, and the value indicates the epoch during which the address was first recorded in a transaction output. If an address does not belong to this category, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**: 
  - A one-dimensional array.
  - Each index corresponds to a unique raw address.
  - Each value indicates the epoch number when the address was first observed, or `999999999999` if the address belongs to another type.
- **Number of Elements**: Equal to the total number of unique raw addresses.

### Example
```plaintext
0
12
999999999999
7
18
...
```
- **Example Interpretation**:
  - Address 0 appeared in epoch 0.
  - Address 1 appeared in epoch 12.
  - Address 2 is of another type, indicated by the placeholder value.
  - Address 3 appeared in epoch 7, and so on.


### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each epoch array to a text file. This method writes the one-dimensional array in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the epoch arrays back into memory for further analysis. This method reads the text file and converts it into a NumPy array.


***

## File: `epochArray_ByronAddresses__Cardano_TXs_All__2023-03-27_085508`

### Description
This file contains the epoch numbers when each Byron payment address first appeared on the Cardano blockchain. Byron addresses are identified as non-smart contract addresses starting with a specific prefix. If an address does not belong to this category, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique Byron payment address.
  - Each value indicates the epoch number when the address was first observed, or `999999999999` if the address belongs to another type.
- **Number of Elements**: Equal to the total number of unique Byron payment addresses.

### Example
```plaintext
2
10
999999999999
14
5
...
```
- **Example Interpretation**:
  - Address 0 appeared in epoch 2.
  - Address 1 appeared in epoch 10.
  - Address 2 is of another type, indicated by the placeholder value.
  - Address 3 appeared in epoch 14, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each epoch array to a text file. This method writes the one-dimensional array in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the epoch arrays back into memory for further analysis. This method reads the text file and converts it into a NumPy array.


***

#### File: `epochArray_ShelleyAddresses__Cardano_TXs_All__2023-03-27_085508`

This file contains the epoch numbers when each Shelley payment address first appeared on the Cardano blockchain. Shelley addresses are identified as non-Byron addresses used in Cardano transactions. If an address does not belong to this category, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique Shelley payment address.
  - Each value indicates the epoch number when the address was first observed, or `999999999999` if the address belongs to another type.
- **Number of Elements**: Equal to the total number of unique Shelley payment addresses.

### Example
```plaintext
3
7
10
999999999999
11
...
```
- **Example Interpretation**:
  - Address 0 appeared in epoch 3.
  - Address 1 appeared in epoch 7.
  - Address 3 is of another type, indicated by the placeholder value.
  - Address 4 appeared in epoch 11, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each epoch array to a text file. This method writes the one-dimensional array in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the epoch arrays back into memory for further analysis. This method reads the text file and converts it into a NumPy array.

***

#### File: `epochArray_delegationAddresses__Cardano_TXs_All__2023-03-27_085508`

This file contains the epoch numbers when each delegation address (stake address) first appeared on the Cardano blockchain. Delegation addresses are associated with stake key identifiers used for staking purposes. If an address does not belong to this category, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique delegation address.
  - Each value indicates the epoch number when the address was first observed, or `999999999999` if the address belongs to another type.
- **Number of Elements**: Equal to the total number of unique delegation addresses.

### Example
```plaintext
5
999999999999
14
12
17
...
```
- **Example Interpretation**:
  - Address 0 appeared in epoch 5.
  - Address 1 is of another type, indicated by the placeholder value.
  - Address 2 appeared in epoch 14, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each epoch array to a text file. This method writes the one-dimensional array in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the epoch arrays back into memory for further analysis. This method reads the text file and converts it into a NumPy array.

- **Generation Method**:
  - Epoch numbers were calculated using transaction IDs and their corresponding epochs.
  - The first occurrence of each address in transaction outputs determined its associated epoch.
  - A placeholder value (`999999999999`) was assigned when an address did not belong to the relevant category.

***

#### File: `graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_224222`

This file represents the adjacency list of a graph constructed from Cardano transaction data. The graph edges were generated using **Heuristic 1** without considering smart contracts (`noSC`), and links were created between each payment address and all other addresses involved in the same transaction. Each list in the adjacency array corresponds to a node (payment address) and contains the indices of all other addresses it is linked to.

### Format
- **Type**: Text file
- **Structure**:
  - A two-dimensional array stored as a JSON-like text file.
  - Each index corresponds to a unique payment address.
  - Each element at the index is a list of indices representing linked payment addresses.

### Example
```plaintext
[
  [1, 2, 3], 
  [0, 3], 
  [0], 
  [0, 1]
]
```
- **Example Interpretation**:
  - Address 0 is linked to addresses 1, 2, and 3.
  - Address 1 is linked to addresses 0 and 3.
  - Address 2 is linked only to address 0.
  - Address 3 is linked to addresses 0 and 1, and so on.

### Methods Used
- **Saving**: The `store_array_to_file_2D` method was used to save the adjacency list (2D array) to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array_2D` method can be used to load the adjacency list back into memory for further analysis.

- **Generation Method**:
  - The edges were generated by linking each address in a transaction to all other addresses in the same transaction.
  - The `merge_graphEdges` function was used to combine multiple adjacency lists into the final merged graph.

***

#### File: `clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__2023-02-25_223957`

This file contains the clustering results for payment addresses on the Cardano network, derived using `Heuristic 1 without considering smart contracts` (`noSC`). Each element in the array corresponds to a payment address, and the value represents the cluster ID to which the address belongs. Clusters are identified through a Union-Find algorithm-based heuristic clustering method.

### Format
- **Type**: Text file
- **Structure**: 
  - A one-dimensional array.
  - Each index corresponds to a unique payment address.
  - Each value indicates the cluster ID assigned to the address.
- **Number of Elements**: Equal to the total number of unique payment addresses (`unique_payment_addresses_len`).

### Example
```plaintext
0
1
1
2
3
0
...
```
- **Example Interpretation**: 
  - Address 1 belongs to cluster 0.
  - Address 2 and Address 3 belong to cluster 1.
  - Address 4 belongs to cluster 2, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.


***


#### File: `clusteringArrayList_Heuristic2__Cardano_TXs_All__2023-03-26_110150`

This file contains the clustering results for payment addresses on the Cardano network, derived using `Heuristic 2`. Each element in the array corresponds to a payment address, and the value represents the cluster ID to which the address belongs. Clusters are identified through a Union-Find algorithm-based heuristic clustering method.

### Format
- **Type**: Text file
- **Structure**: 
  - A one-dimensional array.
  - Each index corresponds to a unique payment address.
  - Each value indicates the cluster ID assigned to the address.
- **Number of Elements**: Equal to the total number of unique payment addresses (`unique_payment_addresses_len`).

### Example
```plaintext
0
1
1
2
3
0
...
```
- **Example Interpretation**: 
  - Address 1 belongs to cluster 0.
  - Address 2 and Address 3 belong to cluster 1.
  - Address 4 belongs to cluster 2, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.

***

#### File: `clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-03-26_141212`

This file contains the clustering results for payment addresses on the Cardano network, derived using `Heuristic 1 without considering smart contracts` (`noSC`) and `Heuristic 2`. Each element in the array corresponds to a payment address, and the value represents the cluster ID to which the address belongs. Clusters are identified through a Union-Find algorithm-based heuristic clustering method.

### Format
- **Type**: Text file
- **Structure**: 
  - A one-dimensional array.
  - Each index corresponds to a unique payment address.
  - Each value indicates the cluster ID assigned to the address.
- **Number of Elements**: Equal to the total number of unique payment addresses (`unique_payment_addresses_len`).

### Example
```plaintext
0
1
1
2
3
0
...
```
- **Example Interpretation**: 
  - Address 1 belongs to cluster 0.
  - Address 2 and Address 3 belong to cluster 1.
  - Address 4 belongs to cluster 2, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.



***

#### File: **`activeAddressesPerDayList__Cardano_TXs_All__2023-04-09_224357`**

This file contains the count of unique active payment addresses on the Cardano network for each day within the dataset's date range (from 2017-09-23 to 2023-01-21). Each value corresponds to the number of distinct payment addresses observed in transactions on that day.

### Format
- **Type**: Text file
- **Structure**: A single column, where each row represents the count of active addresses for one day.
- **Number of Rows**: 1,947 (total days from 2017-09-23 to 2023-01-21).

### Example
```plaintext
5
7
10
8
15
...
```
- **Example Interpretation**: On Day 1 (2017-09-23), there were 5 active addresses; on Day 2, 7 active addresses; and so on.

- **Active Addresses**: Extracted directly from non-smart contract payment addresses in transaction inputs.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the array to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for processing. This method reads the text file and converts it into a NumPy array.

***

#### File: `activeEntitiesPerDayList__Cardano_TXs_All__2023-04-09_224357`

This file contains the count of unique active entities on the Cardano network for each day within the dataset's date range. An entity represents a group of payment addresses clustered based on their interaction patterns (e.g., input-output relationships).

### Format
- **Type**: Text file
- **Structure**: A single column, where each row represents the count of active entities for one day.
- **Number of Rows**: 1,947 (total days from 2017-09-23 to 2023-01-21).

### Example
```plaintext
3
5
6
4
9
...
```
- **Example Interpretation**: On Day 1 (2017-09-23), there were 3 active entities; on Day 2, 5 active entities; and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save the array to this text file. This method writes the array into a `.txt` file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for processing. This method reads the text file and converts it into a NumPy array.

- **Active Entities**: Derived from clustering payment addresses using the Union-Find algorithm.

***

#### **`DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__<timestamp>`**

- **File Format**:
  - Binary file using Python's `pickle` serialization.

- **Structure**:
  - A list of tuples.
  - Each tuple contains:
    1. **Entity Wealth**: (float) The wealth of the entity.
    2. **Pool Stakes**: (int) The stake delegated by the entity to the pool.

- **Number of Columns**: 2 (per tuple).

- **Column Types**:
  - Column 1: `float` (Entity Wealth).
  - Column 2: `int` (Pool Stakes).

- **Example**:
  ```python
  [
      (5000.0, 10000),
      (10000.0, 25000),
      (7500.0, 15000)
  ]
  ```

***


#### **`DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__<timestamp>`**

- **File Format**:
  - Binary file using Python's `pickle` serialization.

- **Structure**:
  - A list of tuples.
  - Each tuple contains:
    1. **Entity Wealth**: (float) The wealth of the entity.
    2. **Number of Pools**: (int) The number of pools the entity delegated to.

- **Number of Columns**: 2 (per tuple).

- **Column Types**:
  - Column 1: `float` (Entity Wealth).
  - Column 2: `int` (Number of Pools).

- **Example**:
  ```python
  [
      (5000.0, 2),
      (10000.0, 3),
      (7500.0, 1)
  ]
  ```

***

#### **`DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__<timestamp>`**

- **File Format**:
  - Binary file using Python's `pickle` serialization.

- **Structure**:
  - A list of tuples.
  - Each tuple contains:
    1. **Entity Wealth**: (float) The wealth of the entity.
    2. **Reward Amount**: (int) The total rewards received by the entity.

- **Number of Columns**: 2 (per tuple).

- **Column Types**:
  - Column 1: `float` (Entity Wealth).
  - Column 2: `int` (Reward Amount).

- **Example**:
  ```python
  [
      (5000.0, 1000),
      (10000.0, 2000),
      (7500.0, 1500)
  ]
  ```

### Summary Table

| **File Name**                                                  | **Columns**                    | **Types**               | **Example**                                       |
|----------------------------------------------------------------|--------------------------------|-------------------------|---------------------------------------------------|
| `DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__`      | 1. Entity Wealth<br>2. Pool Stakes | `float`, `int`          | `[(5000.0, 10000), (10000.0, 25000), ...]`       |
| `DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__`      | 1. Entity Wealth<br>2. Num Pools | `float`, `int`          | `[(5000.0, 2), (10000.0, 3), ...]`               |
| `DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__`    | 1. Entity Wealth<br>2. Rewards  | `float`, `int`          | `[(5000.0, 1000), (10000.0, 2000), ...]`         |


### File Access
To read these files, use Python's `pickle` module with:
```python
with open(filename, 'rb') as file:
    data = pickle.load(file)
```


***


#### **Active Delegators (Entities) per Epoch**

Each file stores the delegation amounts for entities per epoch. These files track how much stake each entity delegated during a specific epoch.

- **File Name**:
  ```
  /Cardano_StakeDelegation_Entities/StakeDelegPerEntityEpoch_XXXX__Cardano_TXs_All
  ```
  - `XXXX`: Zero-padded epoch number.
  - Example: `StakeDelegPerEntityEpoch_0210__Cardano_TXs_All`.

- **File Format**:
  - **Type**: Plain text 
  - **Structure**: Each line represents the total delegation amount for an entity in the epoch.
  - **Length**: The number of lines corresponds to the total number of entities (indexed by clustering array).

- **Details**:
  - **Number of Columns**: 1.
  - **Column Type**: Integer (delegation amount).
  - **Example**:
    ```
    0
    100000
    500000
    0
    ...
    ```
    - Line 1: Entity 0 delegated `0`.
    - Line 2: Entity 1 delegated `100,000`.
    - Line 3: Entity 2 delegated `500,000`.

***

#### **Active Delegators (Stake Addresses) per Epoch**

This data is aggregated for each epoch and stored in arrays, not individual files. The data tracks the number of unique delegator addresses and entities per epoch.

- **Output Format**:
  - Stored as two arrays:
    1. **`num_delegator_addresses_per_epoch`**:
       - Number of unique delegator addresses active during each epoch.
    2. **`num_delegator_entities_per_epoch`**:
       - Number of unique delegator entities active during each epoch.

- **Example Arrays**:
  ```python
  num_delegator_addresses_per_epoch = [1000, 1200, 1100, ...]
  num_delegator_entities_per_epoch = [950, 1100, 1050, ...]
  ```

- **Details**:
  - **Array Length**: Equal to the number of epochs (`last_epoch_no - first_epoch_no + 1`).
  - **Array Values**:
    - Each element represents the count of unique delegators (addresses/entities) for a specific epoch.

- **Example Explanation**:
  - Epoch 210:
    - `1000` delegator addresses.
    - `950` delegator entities.
  - Epoch 211:
    - `1200` delegator addresses.
    - `1100` delegator entities.


### Summary Table

| **Output**                                  | **Format**        | **Details**                                                                                           | **Example**                           |
|---------------------------------------------|-------------------|-------------------------------------------------------------------------------------------------------|---------------------------------------|
| **Active Delegators (Entities)**            | Text file          | Lines correspond to delegation amounts for entities in a specific epoch.                              | `0\n100000\n500000\n...`              |
| **Active Delegators (Stake Addresses)**     | Python arrays      | Two arrays tracking the number of active delegator addresses and entities per epoch.                  | `[1000, 1200, 1100, ...]`             |

***


#### **File: `Num_Delegator_addresses_per_epoch__Cardano_TXs_All__<timestamp>`**

- **Purpose**: 
  Tracks the number of unique delegator addresses active during each epoch.

- **File Format**:
  - **Type**: Plain text
  - **Structure**: 
    - Each line contains the count of unique delegator addresses for one epoch.
  - **Number of Columns**: 1 (Count of delegator addresses per epoch).
  - **Data Type**: Integer.

- **Example Content**:
  ```
  1000
  1200
  1100
  ...
  ```
  - Line 1: Epoch 210 had 1,000 delegator addresses.
  - Line 2: Epoch 211 had 1,200 delegator addresses.
  - Line 3: Epoch 212 had 1,100 delegator addresses.

***

#### **File: `Num_Delegator_entities_per_epoch__Cardano_TXs_All__<timestamp>`**

- **Purpose**:
  Tracks the number of unique delegator entities active during each epoch.

- **File Format**:
  - **Type**: Plain text 
  - **Structure**:
    - Each line contains the count of unique delegator entities for one epoch.
  - **Number of Columns**: 1 (Count of delegator entities per epoch).
  - **Data Type**: Integer.

- **Example Content**:
  ```
  950
  1150
  1050
  ...
  ```
  - Line 1: Epoch 210 had 950 delegator entities.
  - Line 2: Epoch 211 had 1,150 delegator entities.
  - Line 3: Epoch 212 had 1,050 delegator entities.

***

#### **File: `Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__<timestamp>`**

- **Purpose**:
  Tracks the number of unique rewarder addresses active during each epoch.

- **File Format**:
  - **Type**: Plain text 
  - **Structure**:
    - Each line contains the count of unique rewarder addresses for one epoch.
  - **Number of Columns**: 1 (Count of rewarder addresses per epoch).
  - **Data Type**: Integer.

- **Example Content**:
  ```
  800
  900
  850
  ...
  ```
  - Line 1: Epoch 210 had 800 rewarder addresses.
  - Line 2: Epoch 211 had 900 rewarder addresses.
  - Line 3: Epoch 212 had 850 rewarder addresses.

***

#### **File: `Num_Rewarder_entities_per_epoch__Cardano_TXs_All__<timestamp>`**

- **Purpose**:
  Tracks the number of unique rewarder entities active during each epoch.

- **File Format**:
  - **Type**: Plain text 
  - **Structure**:
    - Each line contains the count of unique rewarder entities for one epoch.
  - **Number of Columns**: 1 (Count of rewarder entities per epoch).
  - **Data Type**: Integer.

- **Example Content**:
  ```
  780
  870
  820
  ...
  ```
  - Line 1: Epoch 210 had 780 rewarder entities.
  - Line 2: Epoch 211 had 870 rewarder entities.
  - Line 3: Epoch 212 had 820 rewarder entities.


### Summary Table

| **File Name**                                        | **Purpose**                          | **Columns**           | **Type**  | **Example**                  |
|-----------------------------------------------------|--------------------------------------|-----------------------|-----------|------------------------------|
| `Num_Delegator_addresses_per_epoch__<timestamp>.txt` | Count of delegator addresses per epoch | 1 (Count)            | `int`     | `1000\n1200\n1100\n...`      |
| `Num_Delegator_entities_per_epoch__<timestamp>.txt`  | Count of delegator entities per epoch | 1 (Count)            | `int`     | `950\n1150\n1050\n...`       |
| `Num_Rewarder_addresses_per_epoch__<timestamp>.txt`  | Count of rewarder addresses per epoch | 1 (Count)            | `int`     | `800\n900\n850\n...`         |
| `Num_Rewarder_entities_per_epoch__<timestamp>.txt`   | Count of rewarder entities per epoch  | 1 (Count)            | `int`     | `780\n870\n820\n...`         |


### File Access

To load the files:
```python
num_delegator_addresses_per_epoch = load_file_to_array(file_name)
```

***

### **Output Files for "Calculate Entities Balances"**

- **Filename Format**:
  - `BalancesPerEntityDay_{day_offset}__Cardano_TXs_All`
  - Example: `BalancesPerEntityDay_0001__Cardano_TXs_All`

- **File Format**: CSV (plain text files saved as comma-separated values)

- **Number of Columns**: 1

- **Column Details**:
  - **Column Name**: Balance
  - **Column Type**: Integer
  - **Description**: Represents the ADA balance for each entity on the given day. The index of each row corresponds to the entity ID in the `clustering_array`.

- **Rows**: One row per entity in the clustering array.


***


### **Output Files for "Calculate Entities TX_vol"**

- **Filename Format**:
  - `TX_Vol_PerEntityDay_{day_offset}__Cardano_TXs_All.pickle`
  - Example: `TX_Vol_PerEntityDay_0001__Cardano_TXs_All.pickle`

- **File Format**: Pickle (`.pickle` binary files)

- **Number of Columns**: 1 (represented as a Python list)

- **Column Details**:
  - **Column Name**: Transaction Volume
  - **Column Type**: Integer
  - **Description**: Represents the absolute transaction volume (sum of input and output amounts) for each entity on the given day. The index of each element corresponds to the entity ID in the `clustering_array`.

- **Rows**: One row per entity in the clustering array.

### General Notes on File Structure
1. **Daily Output**: Each file represents data for a single day, identified by the `day_offset` starting from the initial date (`2017-09-23`).
2. **Consistency**: The index or order of rows in both file formats aligns with the entity IDs defined in the clustering array (`clustering_array`).
3. **Storage Location**:
   - Balances: Stored in `Cardano_Balances_Entities` directory.
   - Transaction Volumes: Stored in `Cardano_TX_Vols_Entities__PICKLE` directory.
4. **File Size**: As both formats iterate over entities daily, the file sizes scale with the number of entities and daily transactions processed.

***




