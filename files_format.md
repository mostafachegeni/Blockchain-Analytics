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
