#### **`DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__<timestamp>.txt`**

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


#### **`DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__<timestamp>.txt`**

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

#### **`DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__<timestamp>.txt`**

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
| `DelegEvents__entityWealth_poolStakes_pairs__AllEpochs__.txt`  | 1. Entity Wealth<br>2. Pool Stakes | `float`, `int`          | `[(5000.0, 10000), (10000.0, 25000), ...]`       |
| `DelegEvents__entityWealth_numOfPools_pairs__AllEpochs__.txt`  | 1. Entity Wealth<br>2. Num Pools | `float`, `int`          | `[(5000.0, 2), (10000.0, 3), ...]`               |
| `DelegEvents__entityWealth_rewardAmount_pairs__AllEpochs__.txt`| 1. Entity Wealth<br>2. Rewards  | `float`, `int`          | `[(5000.0, 1000), (10000.0, 2000), ...]`         |


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
  /YuZhang_Cardano_StakeDelegation_Entities/StakeDelegPerEntityEpoch_XXXX__Cardano_TXs_All.txt
  ```
  - `XXXX`: Zero-padded epoch number.
  - Example: `StakeDelegPerEntityEpoch_0210__Cardano_TXs_All.txt`.

- **File Format**:
  - **Type**: Plain text file (`.txt`).
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
| **Active Delegators (Entities)**            | Text file (`.txt`) | Lines correspond to delegation amounts for entities in a specific epoch.                              | `0\n100000\n500000\n...`              |
| **Active Delegators (Stake Addresses)**     | Python arrays      | Two arrays tracking the number of active delegator addresses and entities per epoch.                  | `[1000, 1200, 1100, ...]`             |

***


#### **File: `Num_Delegator_addresses_per_epoch__Cardano_TXs_All__<timestamp>.txt`**

- **Purpose**: 
  Tracks the number of unique delegator addresses active during each epoch.

- **File Format**:
  - **Type**: Plain text file (`.txt`).
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

#### **File: `Num_Delegator_entities_per_epoch__Cardano_TXs_All__<timestamp>.txt`**

- **Purpose**:
  Tracks the number of unique delegator entities active during each epoch.

- **File Format**:
  - **Type**: Plain text file (`.txt`).
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

#### **File: `Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__<timestamp>.txt`**

- **Purpose**:
  Tracks the number of unique rewarder addresses active during each epoch.

- **File Format**:
  - **Type**: Plain text file (`.txt`).
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

#### **File: `Num_Rewarder_entities_per_epoch__Cardano_TXs_All__<timestamp>.txt`**

- **Purpose**:
  Tracks the number of unique rewarder entities active during each epoch.

- **File Format**:
  - **Type**: Plain text file (`.txt`).
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
  - `YuZhang__BalancesPerEntityDay_{day_offset}__Cardano_TXs_All.txt`
  - Example: `YuZhang__BalancesPerEntityDay_0001__Cardano_TXs_All.txt`

- **File Format**: CSV (`.txt` files saved as comma-separated values)

- **Number of Columns**: 1

- **Column Details**:
  - **Column Name**: Balance
  - **Column Type**: Integer
  - **Description**: Represents the ADA balance for each entity on the given day. The index of each row corresponds to the entity ID in the `clustering_array`.

- **Rows**: One row per entity in the clustering array.

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
   - Balances: Stored in `YuZhang_Cardano_Balances_Entities` directory.
   - Transaction Volumes: Stored in `YuZhang_Cardano_TX_Vols_Entities__PICKLE` directory.
4. **File Size**: As both formats iterate over entities daily, the file sizes scale with the number of entities and daily transactions processed.

***




