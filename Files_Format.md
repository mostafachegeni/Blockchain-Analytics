
***

#### File: `EntityTXsADA_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains a list of ADA-based transactions at the entity level in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. Each transaction includes details about the sender, recipient, transaction day, and entity balances before and after the transaction.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A list of tuples where each tuple represents a transaction.
  - Each tuple contains:
    - `tx_delta_day` (int): The day (relative to Cardano's initial date) when the transaction occurred.
    - `entity_from` (int): The index of the sending entity.
    - `entity_to` (int): The index of the receiving entity.
    - `balance_entity_from` (int): The sender's balance before the transaction.
    - `balance_entity_to` (int): The recipient's balance before the transaction.

### Example
```plaintext
[
  (100, 2, 5, 1000000, 500000), 
  (105, 3, 8, 200000, 700000), 
  ...
]
```
- **Example Interpretation**:
  - On day 100, entity 2 sent ADA to entity 5, starting with balances of 1,000,000 and 500,000 respectively.
  - On day 105, entity 3 sent ADA to entity 8, starting with balances of 200,000 and 700,000 respectively.

### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the transaction lists to files. This ensures efficient storage and retrieval of large datasets.
- **Loading**: The `pickle.load` method was used to deserialize and load the transaction lists back into memory for analysis.

- **Generation Method**:
  - ADA, NFT, and FT transactions were identified by analyzing inputs and outputs in the Cardano transaction data.
  - Entities were determined using a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.
  - The balance changes for each entity were tracked to calculate balances.
  - Transactions were filtered to exclude smart contract addresses.


***

#### File: `EntityTXsNFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains a list of NFT-based transactions at the entity level in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. Each transaction includes details about the sender, recipient, transaction day, and entity balances before and after the transaction.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A list of tuples where each tuple represents a transaction.
  - Each tuple contains:
    - `tx_delta_day` (int): The day (relative to Cardano's initial date) when the transaction occurred.
    - `entity_from` (int): The index of the sending entity.
    - `entity_to` (int): The index of the receiving entity.
    - `balance_entity_from` (int): The sender's balance before the transaction.
    - `balance_entity_to` (int): The recipient's balance before the transaction.

### Example
```plaintext
[
  (101, 4, 9, 500000, 300000), 
  (120, 6, 10, 800000, 450000), 
  ...
]
```
- **Example Interpretation**:
  - On day 101, entity 4 sent an NFT to entity 9, starting with balances of 500,000 and 300,000 respectively.
  - On day 120, entity 6 sent an NFT to entity 10, starting with balances of 800,000 and 450,000 respectively.

### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the transaction lists to files. This ensures efficient storage and retrieval of large datasets.
- **Loading**: The `pickle.load` method was used to deserialize and load the transaction lists back into memory for analysis.

- **Generation Method**:
  - ADA, NFT, and FT transactions were identified by analyzing inputs and outputs in the Cardano transaction data.
  - Entities were determined using a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.
  - The balance changes for each entity were tracked to calculate balances.
  - Transactions were filtered to exclude smart contract addresses.

***

#### File: `EntityTXsFT_AllTXs_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains a list of FT-based transactions at the entity level in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. Each transaction includes details about the sender, recipient, transaction day, and entity balances before and after the transaction.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A list of tuples where each tuple represents a transaction.
  - Each tuple contains:
    - `tx_delta_day` (int): The day (relative to Cardano's initial date) when the transaction occurred.
    - `entity_from` (int): The index of the sending entity.
    - `entity_to` (int): The index of the receiving entity.
    - `balance_entity_from` (int): The sender's balance before the transaction.
    - `balance_entity_to` (int): The recipient's balance before the transaction.

### Example
```plaintext
[
  (110, 7, 11, 600000, 550000), 
  (130, 12, 15, 300000, 800000), 
  ...
]
```
- **Example Interpretation**:
  - On day 110, entity 7 sent an FT to entity 11, starting with balances of 600,000 and 550,000 respectively.
  - On day 130, entity 12 sent an FT to entity 15, starting with balances of 300,000 and 800,000 respectively.


### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the transaction lists to files. This ensures efficient storage and retrieval of large datasets.
- **Loading**: The `pickle.load` method was used to deserialize and load the transaction lists back into memory for analysis.

- **Generation Method**:
  - ADA, NFT, and FT transactions were identified by analyzing inputs and outputs in the Cardano transaction data.
  - Entities were determined using a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.
  - The balance changes for each entity were tracked to calculate balances.
  - Transactions were filtered to exclude smart contract addresses.


***

#### File: `EntityBalancesArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains the balances of each entity in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. The balance represents the net total of UTXO values associated with each entity.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to an entity.
  - Each value represents the total balance (integer) of the entity.
- **Number of Elements**: Equal to the total number of unique entities.

### Example
```plaintext
[1000000, 200000, 0, -50000, ...]
```
- **Example Interpretation**:
  - Entity 0 has a balance of 1,000,000.
  - Entity 1 has a balance of 200,000.
  - Entity 2 has a balance of 0.
  - Entity 3 has a negative balance of -50,000 (indicating a debt), and so on.


### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the arrays to files. This ensures efficient storage of complex structures like two-dimensional arrays or lists of tuples.
- **Loading**: The `pickle.load` method was used to deserialize and load the arrays back into memory for further analysis.


- **Generation Method**:
  - Balances were calculated by summing the UTXO values in transaction inputs and outputs for each entity.
  - NFTs and FTs owned were determined by tracking `MA_ID`s in inputs and outputs, updating the corresponding entity's ownership.
  - Entity clustering was based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.


***

#### File: `EntityOwnNFTsNumberArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

### Description
This file contains the number of NFTs (Non-Fungible Tokens) owned by each entity in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to an entity.
  - Each value represents the number of NFTs owned by the entity.
- **Number of Elements**: Equal to the total number of unique entities.

### Example
```plaintext
[2, 0, 5, 10, ...]
```
- **Example Interpretation**:
  - Entity 0 owns 2 NFTs.
  - Entity 1 owns no NFTs.
  - Entity 2 owns 5 NFTs.
  - Entity 3 owns 10 NFTs, and so on.


### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the arrays to files. This ensures efficient storage of complex structures like two-dimensional arrays or lists of tuples.
- **Loading**: The `pickle.load` method was used to deserialize and load the arrays back into memory for further analysis.


- **Generation Method**:
  - Balances were calculated by summing the UTXO values in transaction inputs and outputs for each entity.
  - NFTs and FTs owned were determined by tracking `MA_ID`s in inputs and outputs, updating the corresponding entity's ownership.
  - Entity clustering was based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.


***

#### File: `EntityOwnNFTsWithNameArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains the detailed list of NFTs (Non-Fungible Tokens) owned by each entity in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. Each NFT is represented by its `MA_ID` and associated metadata.

### Format
- **Type**: Serialized Python object (Pickle file).
- **Structure**:
  - A two-dimensional array.
  - Each index corresponds to an entity.
  - Each element at the index is a list of tuples, where each tuple contains:
    - `MA_ID`: The unique identifier of the NFT.
    - Associated metadata (e.g., quantity, name, policy).
- **Number of Rows**: Equal to the total number of unique entities.

### Example
```plaintext
[
  [("NFT1", 1, "ArtToken", "Policy123"), ("NFT2", 1, "GameAsset", "Policy456")],
  [],
  [("NFT3", 1, "DigitalArt", "Policy789")],
  ...
]
```
- **Example Interpretation**:
  - Entity 0 owns NFTs `NFT1` and `NFT2` with metadata indicating their type and policy.
  - Entity 1 owns no NFTs.
  - Entity 2 owns NFT `NFT3` with the associated metadata, and so on.


### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the arrays to files. This ensures efficient storage of complex structures like two-dimensional arrays or lists of tuples.
- **Loading**: The `pickle.load` method was used to deserialize and load the arrays back into memory for further analysis.


- **Generation Method**:
  - Balances were calculated by summing the UTXO values in transaction inputs and outputs for each entity.
  - NFTs and FTs owned were determined by tracking `MA_ID`s in inputs and outputs, updating the corresponding entity's ownership.
  - Entity clustering was based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**.


***

#### File: `EntityNFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__2023-04-08_075547`

This file contains a list of NFTs (Non-Fungible Tokens) minted by each entity in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**, and NFTs are associated with entities through their minting activity.

### Format
- **Type**: Text file
- **Structure**:
  - A two-dimensional array stored in a line-by-line JSON-like format.
  - Each index corresponds to an entity.
  - Each element at the index is a list of `MA_ID`s (Mint Asset IDs) representing NFTs minted by the entity.
- **Number of Rows**: Equal to the total number of unique entities.

### Example
```plaintext
[
  ["NFT1", "NFT2"], 
  ["NFT3"], 
  [], 
  ["NFT4", "NFT5", "NFT6"], 
  ...
]
```
- **Example Interpretation**:
  - Entity 0 minted NFTs `NFT1` and `NFT2`.
  - Entity 1 minted NFT `NFT3`.
  - Entity 2 did not mint any NFTs.
  - Entity 3 minted NFTs `NFT4`, `NFT5`, and `NFT6`, and so on.

### Methods Used
- **Saving**: The `store_array_to_file_2D` method was used to save the NFT and FT arrays to text files. This method writes the two-dimensional arrays in a JSON-like format, line by line.
- **Loading**: The `load_file_to_array_2D` method was used to load the NFT and FT arrays back into memory. This method reads the files and converts them into two-dimensional arrays for further analysis.

- **Generation Method**:
  - **NFTs**:
    - NFT minting data (`MINT_NFTs`) was extracted from transactions.
    - For each NFT, the corresponding entity was identified based on the payment address in the transaction input.
    - If the minting address was a smart contract, the NFT was counted as minted by a smart contract.
  - **FTs**:
    - FT minting data (`MINT_FTs`) was extracted similarly, and entities were identified using the same method.
    - FTs minted by smart contracts were counted separately.
  - The entity clustering was based on the combination of **Heuristic 1 (noSC)** and **Heuristic 2**.

***

#### File: `EntityFTsArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains a list of FTs (Fungible Tokens) minted by each entity in the Cardano network. Entities are identified based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**, and FTs are associated with entities through their minting activity.

### Format
- **Type**: Text file
- **Structure**:
  - A two-dimensional array stored in a line-by-line JSON-like format.
  - Each index corresponds to an entity.
  - Each element at the index is a list of `MA_ID`s (Mint Asset IDs) representing FTs minted by the entity.
- **Number of Rows**: Equal to the total number of unique entities.

### Example
```plaintext
[
  ["FT1", "FT2", "FT3"], 
  [], 
  ["FT4"], 
  ["FT5", "FT6"], 
  ...
]
```
- **Example Interpretation**:
  - Entity 0 minted FTs `FT1`, `FT2`, and `FT3`.
  - Entity 1 did not mint any FTs.
  - Entity 2 minted FT `FT4`.
  - Entity 3 minted FTs `FT5` and `FT6`, and so on.


### Methods Used
- **Saving**: The `store_array_to_file_2D` method was used to save the NFT and FT arrays to text files. This method writes the two-dimensional arrays in a JSON-like format, line by line.
- **Loading**: The `load_file_to_array_2D` method was used to load the NFT and FT arrays back into memory. This method reads the files and converts them into two-dimensional arrays for further analysis.

- **Generation Method**:
  - **NFTs**:
    - NFT minting data (`MINT_NFTs`) was extracted from transactions.
    - For each NFT, the corresponding entity was identified based on the payment address in the transaction input.
    - If the minting address was a smart contract, the NFT was counted as minted by a smart contract.
  - **FTs**:
    - FT minting data (`MINT_FTs`) was extracted similarly, and entities were identified using the same method.
    - FTs minted by smart contracts were counted separately.
  - The entity clustering was based on the combination of **Heuristic 1 (noSC)** and **Heuristic 2**.

***

#### File: `Unique_AddressesListRaw__Cardano_TXs_All__<timestamp>`

This file contains the sorted and deduplicated list of all raw addresses extracted from the Cardano transaction data. Raw addresses are the complete address strings as they appear in the blockchain.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of unique raw addresses, sorted lexicographically.
  - Each line represents one unique raw address.
- **Number of Elements**: Equal to the total number of unique raw addresses.

### Example
```plaintext
addr1q57fghjsl...hxyzlmn
addr1qxabcuvw...defmnop
addr1qynopqrs...tuvwxyz
...
```

### Methods Used

#### Deduplication and Sorting
- Duplicate entries were removed, and the lists were sorted using the `sort` command with the `-k 1 -u` options:
  - `-k 1`: Specifies sorting by the first field (entire line for addresses).
  - `-u`: Ensures only unique lines are retained.

#### Loading
- The `load_file_to_array` method was used to load the deduplicated and sorted lists back into memory as arrays for further analysis.

- **Generation Method**:
  - Raw, payment, and delegation addresses were extracted from transaction outputs in Cardano data.
  - Deduplication and sorting were performed using the Unix `sort` command.
  - Resulting files represent unique address lists for each category.


***

#### File: `Unique_AddressesListPayment__Cardano_TXs_All__<timestamp>`

This file contains the sorted and deduplicated list of all payment addresses extracted from the Cardano transaction data. Payment addresses represent the payment credential portion of raw addresses.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of unique payment addresses, sorted lexicographically.
  - Each line represents one unique payment address.
- **Number of Elements**: Equal to the total number of unique payment addresses.

### Example
```plaintext
payment1qlmnopqr...yzabcd
payment1qrstuabc...efghij
payment1qvwxyzde...ghijkl
...
```

### Methods Used

#### Deduplication and Sorting
- Duplicate entries were removed, and the lists were sorted using the `sort` command with the `-k 1 -u` options:
  - `-k 1`: Specifies sorting by the first field (entire line for addresses).
  - `-u`: Ensures only unique lines are retained.

#### Loading
- The `load_file_to_array` method was used to load the deduplicated and sorted lists back into memory as arrays for further analysis.

- **Generation Method**:
  - Raw, payment, and delegation addresses were extracted from transaction outputs in Cardano data.
  - Deduplication and sorting were performed using the Unix `sort` command.
  - Resulting files represent unique address lists for each category.


***

#### File: `Unique_AddressesListDelegation__Cardano_TXs_All__<timestamp>`

This file contains the sorted and deduplicated list of all delegation (stake) addresses extracted from the Cardano transaction data. Delegation addresses represent the staking portion of raw addresses.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of unique delegation addresses, sorted lexicographically.
  - Each line represents one unique delegation address.
- **Number of Elements**: Equal to the total number of unique delegation addresses.

### Example
```plaintext
stake1ulmnopqr...xyzabcd
stake1umnopabc...efghijk
stake1uvwxdefg...ghijklm
...
```

### Methods Used

#### Deduplication and Sorting
- Duplicate entries were removed, and the lists were sorted using the `sort` command with the `-k 1 -u` options:
  - `-k 1`: Specifies sorting by the first field (entire line for addresses).
  - `-u`: Ensures only unique lines are retained.

#### Loading
- The `load_file_to_array` method was used to load the deduplicated and sorted lists back into memory as arrays for further analysis.

- **Generation Method**:
  - Raw, payment, and delegation addresses were extracted from transaction outputs in Cardano data.
  - Deduplication and sorting were performed using the Unix `sort` command.
  - Resulting files represent unique address lists for each category.

***

#### File: `AddressListRaw__Cardano_TXs_All__<timestamp>`

### Description
This file contains a list of all raw addresses extracted from the outputs of transactions in the Cardano network. Raw addresses are complete address strings as they appear in the blockchain, prior to separating into payment and delegation components.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of raw addresses.
  - Each line represents one unique raw address.
- **Number of Elements**: Equal to the total number of unique raw addresses extracted.

### Example
```plaintext
addr1q9xyvqldf3t...wex3npgz
addr1qx57cde6vkl...87shlm3k
addr1qqwxyg7xwfj...mlk78hrk
...
```

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each list of addresses to its respective text file. This method writes the array line by line.
- **Loading**: The `load_file_to_array` method can be used to load the lists back into memory as arrays for further analysis.


- **Generation Method**:
  - Addresses were extracted from the `OUTPUTs` column in multiple transaction CSV files.
  - For each transaction output:
    - Raw addresses were added directly.
    - Payment and delegation addresses were derived using the `extract_payment_delegation_parts` function and added to their respective lists.

***

#### File: `AddressListPayment__Cardano_TXs_All__<timestamp>`

### Description
This file contains a list of all payment addresses extracted from the raw addresses in the Cardano network. Payment addresses are derived by processing raw addresses and represent the payment credential part.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of payment addresses.
  - Each line represents one unique payment address.
- **Number of Elements**: Equal to the total number of unique payment addresses extracted.

### Example
```plaintext
payment1qvl0jxud3vl...asdef34j
payment1qwz57cjmvlk...67aqds21
payment1qqvtygkdxvj...as34lmnk
...
```

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each list of addresses to its respective text file. This method writes the array line by line.
- **Loading**: The `load_file_to_array` method can be used to load the lists back into memory as arrays for further analysis.


- **Generation Method**:
  - Addresses were extracted from the `OUTPUTs` column in multiple transaction CSV files.
  - For each transaction output:
    - Raw addresses were added directly.
    - Payment and delegation addresses were derived using the `extract_payment_delegation_parts` function and added to their respective lists.


***

#### File: `AddressListDelegation__Cardano_TXs_All__<timestamp>`

This file contains a list of all delegation (stake) addresses extracted from the raw addresses in the Cardano network. Delegation addresses represent the staking part of an address, derived from the raw address string.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional list of delegation addresses.
  - Each line represents one unique delegation address.
- **Number of Elements**: Equal to the total number of unique delegation addresses extracted.

### Example
```plaintext
stake1uyvqldf3t...9opqkfg
stake1u57cdexvl...afg123r
stake1uqwyvgdxj...asdfe3j
...
```

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each list of addresses to its respective text file. This method writes the array line by line.
- **Loading**: The `load_file_to_array` method can be used to load the lists back into memory as arrays for further analysis.


- **Generation Method**:
  - Addresses were extracted from the `OUTPUTs` column in multiple transaction CSV files.
  - For each transaction output:
    - Raw addresses were added directly.
    - Payment and delegation addresses were derived using the `extract_payment_delegation_parts` function and added to their respective lists.


***

#### File: `parentsList_Heuristic1noSC__Cardano_TXs_All__<timestamp>`

This file contains the parent mapping of addresses in the Cardano transaction network based on **Heuristic 1 without considering smart contracts** (`noSC`). The parent array represents the Union-Find data structure where each index corresponds to a unique address, and the value represents its parent in the clustering hierarchy.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique address.
  - Each value represents the parent of the address in the clustering hierarchy.
- **Number of Elements**: Equal to the total number of unique addresses.

### Example
```plaintext
0
0
2
3
3
0
...
```
- **Example Interpretation**:
  - Address 0 is its own parent (root of a cluster).
  - Address 1 belongs to the cluster rooted at Address 0.
  - Address 2 is its own parent (root of a cluster).
  - Address 3 belongs to the cluster rooted at Address 3, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each parent array to its corresponding text file. This method writes the array in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the parent arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - **Heuristic 1 (noSC)**: Clustering was performed using transactions excluding smart contract addresses, with multiple parent arrays merged into one.
  - **Heuristic 2**: Clustering was performed using relationships derived from Heuristic 2-specific analysis.
  - **Combined Heuristic**: The `merge_parents` function was used to combine the parent mappings from Heuristic 1 (noSC) and Heuristic 2.

***

#### File: `parentsList_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains the parent mapping of addresses in the Cardano transaction network based on **Heuristic 2**. The parent array represents the clustering of addresses into entities, determined by analyzing address relationships specific to Heuristic 2.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique address.
  - Each value represents the parent of the address in the clustering hierarchy.
- **Number of Elements**: Equal to the total number of unique addresses.

### Example
```plaintext
0
1
2
2
3
3
...
```
- **Example Interpretation**:
  - Address 0 is its own parent (root of a cluster).
  - Address 1 is its own parent.
  - Address 2 and Address 3 belong to the cluster rooted at Address 2, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each parent array to its corresponding text file. This method writes the array in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the parent arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - **Heuristic 1 (noSC)**: Clustering was performed using transactions excluding smart contract addresses, with multiple parent arrays merged into one.
  - **Heuristic 2**: Clustering was performed using relationships derived from Heuristic 2-specific analysis.
  - **Combined Heuristic**: The `merge_parents` function was used to combine the parent mappings from Heuristic 1 (noSC) and Heuristic 2.

***

#### File: `parentsList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

This file contains the parent mapping of addresses in the Cardano transaction network based on a combination of **Heuristic 1 (noSC)** and **Heuristic 2**. The parent array represents the clustering of addresses determined by merging the parent mappings from both heuristics.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique address.
  - Each value represents the parent of the address in the merged clustering hierarchy.
- **Number of Elements**: Equal to the total number of unique addresses.

### Example
```plaintext
0
1
1
3
3
1
...
```
- **Example Interpretation**:
  - Address 0 is its own parent (root of a cluster).
  - Address 1 is the root of a cluster that includes Address 2.
  - Address 3 is the root of a cluster that includes Address 4, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each parent array to its corresponding text file. This method writes the array in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the parent arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - **Heuristic 1 (noSC)**: Clustering was performed using transactions excluding smart contract addresses, with multiple parent arrays merged into one.
  - **Heuristic 2**: Clustering was performed using relationships derived from Heuristic 2-specific analysis.
  - **Combined Heuristic**: The `merge_parents` function was used to combine the parent mappings from Heuristic 1 (noSC) and Heuristic 2.



***

#### File: `newPerDay_rawAddresses__Cardano_TXs_All__<timestamp>`

### Description
This file contains the day number (relative to the Cardano blockchain's initial date) when each raw address first appeared in a transaction. If a raw address has not been observed in any transaction, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique raw address.
  - Each value indicates the day number when the address first appeared or `999999999999` if not observed.
- **Number of Elements**: Equal to the total number of unique raw addresses.

### Example
```plaintext
0
15
999999999999
42
...
```
- **Example Interpretation**:
  - Address 0 appeared on Day 0.
  - Address 1 appeared on Day 15.
  - Address 2 has not been observed, as indicated by the placeholder value.
  - Address 3 appeared on Day 42, and so on.


### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each array to its corresponding text file. This method writes the data in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - For each type of address (raw, Byron, Shelley, delegation), the day number was determined when the address first appeared in a transaction output.
  - A placeholder value (`999999999999`) was assigned to addresses that were not observed.


***

#### File: `newPerDay_ByronAddresses__Cardano_TXs_All__<timestamp>`

This file contains the day number when each Byron payment address first appeared in a transaction. Byron addresses are identified as non-smart contract addresses with a specific prefix. If a Byron address has not been observed in any transaction, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique Byron payment address.
  - Each value indicates the day number when the address first appeared or `999999999999` if not observed.
- **Number of Elements**: Equal to the total number of unique Byron payment addresses.

### Example
```plaintext
10
20
999999999999
35
...
```
- **Example Interpretation**:
  - Address 0 appeared on Day 10.
  - Address 1 appeared on Day 20.
  - Address 2 has not been observed, as indicated by the placeholder value.
  - Address 3 appeared on Day 35, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each array to its corresponding text file. This method writes the data in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - For each type of address (raw, Byron, Shelley, delegation), the day number was determined when the address first appeared in a transaction output.
  - A placeholder value (`999999999999`) was assigned to addresses that were not observed.


***

#### File: `newPerDay_ShelleyAddresses__Cardano_TXs_All__<timestamp>`

This file contains the day number when each Shelley payment address first appeared in a transaction. Shelley addresses are identified as non-Byron addresses. If a Shelley address has not been observed in any transaction, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique Shelley payment address.
  - Each value indicates the day number when the address first appeared or `999999999999` if not observed.
- **Number of Elements**: Equal to the total number of unique Shelley payment addresses.

### Example
```plaintext
5
18
999999999999
27
...
```
- **Example Interpretation**:
  - Address 0 appeared on Day 5.
  - Address 1 appeared on Day 18.
  - Address 2 has not been observed, as indicated by the placeholder value.
  - Address 3 appeared on Day 27, and so on.

### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each array to its corresponding text file. This method writes the data in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - For each type of address (raw, Byron, Shelley, delegation), the day number was determined when the address first appeared in a transaction output.
  - A placeholder value (`999999999999`) was assigned to addresses that were not observed.


***

#### File: `newPerDay_delegationAddresses__Cardano_TXs_All__<timestamp>`

This file contains the day number when each delegation (stake) address first appeared in a transaction. Delegation addresses are associated with staking operations. If a delegation address has not been observed in any transaction, a placeholder value of `999999999999` is used.

### Format
- **Type**: Text file
- **Structure**:
  - A one-dimensional array.
  - Each index corresponds to a unique delegation address.
  - Each value indicates the day number when the address first appeared or `999999999999` if not observed.
- **Number of Elements**: Equal to the total number of unique delegation addresses.

### Example
```plaintext
7
25
999999999999
38
...
```
- **Example Interpretation**:
  - Address 0 appeared on Day 7.
  - Address 1 appeared on Day 25.
  - Address 2 has not been observed, as indicated by the placeholder value.
  - Address 3 appeared on Day 38, and so on.


### Methods Used
- **Saving**: The `store_array_to_file` method was used to save each array to its corresponding text file. This method writes the data in a JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the arrays from the text files. This method reads the files and converts them into NumPy arrays for further analysis.

- **Generation Method**:
  - For each type of address (raw, Byron, Shelley, delegation), the day number was determined when the address first appeared in a transaction output.
  - A placeholder value (`999999999999`) was assigned to addresses that were not observed.


***

#### File: `Largest<number>_cc_subgraph_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__<timestamp>`

This file contains a serialized graph object representing the `<number>`th largest connected component (CC) of the Cardano address network, constructed using **Heuristic 1 without considering smart contracts** (`noSC`). The subgraph was extracted from the complete graph (`Graph G`) by isolating the `<number>`th largest set of connected nodes based on transaction co-occurrence data.

### Format
- **Type**: Pickle file (serialized Python object).
- **Structure**:
  - A graph object (`networkx.Graph`) where:
    - Nodes represent unique addresses in the `<number>`th largest connected component.
    - Edges represent connections between addresses, with weights indicating the frequency of interaction.
    - The subgraph is a copy of the isolated component.

### Example (Conceptual Representation)
```python
largest_cc_subgraph.nodes()
# [3, 7, 12, 45, ...]

largest_cc_subgraph.edges(data=True)
# [(3, 7, {'weight': 2}), (7, 12, {'weight': 1}), ...]
```

- **Example Interpretation**:
  - Node 3 is connected to Node 7 with a weight of 2.
  - Node 7 is connected to Node 12 with a weight of 1, and so on.

### Properties of the Subgraph
- **Connectivity**: The subgraph is fully connected, as verified using `nx.is_connected()`.
- **Number of Nodes**: `largest_cc_subgraph.number_of_nodes()`.
- **Number of Edges**: `largest_cc_subgraph.number_of_edges()`.

### Methods Used
- **Saving**: **The `pickle.dump` method was used to serialize and save the subgraph object to a file. This ensures efficient storage of the graph structure and associated metadata.
- **Loading**: **The `pickle.load` method was used to deserialize and load the subgraph object into memory for further analysis.

- **Generation Method**:
  - The `<number>`th largest connected component was identified by:
    1. Sorting all connected components of the complete graph (`Graph G`) by size in descending order.
    2. Extracting the `<number>`th component as a subgraph using `G.subgraph()`.


***

#### File: `Graph_G_AddrsNetwork_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__<timestamp>`

This file contains a serialized graph object (`Graph G`) representing the Cardano transaction network. The graph was constructed using **Heuristic 1 without considering smart contracts** (`noSC`) and is based on the weighted adjacency list stored in `graphWeightsArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_234559`. Nodes represent addresses, and edges represent co-occurrence relationships in transactions, with weights indicating the frequency of interactions.

### Format
- **Type**: Pickle file (serialized Python object).
- **Structure**:
  - A graph object (`networkx.Graph`) where:
    - Nodes correspond to unique addresses in the Cardano transaction network.
    - Weighted edges represent the strength of the connection between two addresses based on their frequency of co-occurrence.

### Example (Conceptual Representation)
```python
G.nodes()
# [0, 1, 2, 3, ...]

G.edges(data=True)
# [(0, 1, {'weight': 2}), (0, 2, {'weight': 3}), (1, 3, {'weight': 1}), ...]
```

- **Example Interpretation**:
  - Node 0 is connected to Node 1 with a weight of 2.
  - Node 0 is connected to Node 2 with a weight of 3.
  - Node 1 is connected to Node 3 with a weight of 1, and so on.


### Methods Used
- **Saving**: The `pickle.dump` method was used to serialize and save the graph object to a file. This method ensures efficient storage of the graph structure and metadata.
- **Loading**: The `pickle.load` method was used to deserialize and load the graph object into memory for analysis.


- **Generation Method**:
  - The graph was constructed using the weighted adjacency list (`graph_weights`) derived from Cardano transaction data.
  - Each address was added as a node, and weighted edges were created using the `add_weighted_edges_from` method of the `networkx` library.

***

#### File: `graphWeightsArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__<timestamp>`

### Description
This file contains the weighted adjacency list of a graph representing the Cardano transaction network. The weights of edges between addresses indicate the number of times the addresses were identified together (clustered) in a transaction based on **Heuristic 1 without considering smart contracts** (`noSC`). The file captures all weighted connections between addresses.

### Format
- **Type**: Text file
- **Structure**:
  - A two-dimensional array stored in a line-by-line JSON-like format.
  - Each index corresponds to a unique address.
  - Each element is a list of tuples, where each tuple contains:
    - The index of a connected address.
    - The weight of the edge (an integer representing the frequency of co-occurrence in transactions).
- **Number of Rows**: Equal to the total number of unique addresses (`unique_addresses_len`).

### Example
```plaintext
[(1, 2), (2, 3)], 
[(0, 2), (3, 1)], 
[(0, 3)], 
[(1, 1)]
```

- **Example Interpretation**:
  - Address 0 is connected to Address 1 with a weight of 2 and to Address 2 with a weight of 3.
  - Address 1 is connected to Address 0 with a weight of 2 and to Address 3 with a weight of 1.
  - Address 2 is connected to Address 0 with a weight of 3, and so on.

### Methods Used
- **Saving**: The weighted adjacency list was saved to a text file using a custom method that writes each list of tuples line by line.
- **Loading**: **The file was loaded back into memory by parsing each line as a list of tuples using Python's `ast.literal_eval` method.


- **Generation Method**:
  - The graph edges were first generated and stored in the file `graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__2023-02-25_224222`.
  - Weights were calculated using the `find_weights_graphEdges` function:
    - Each time two addresses were clustered together in a transaction, the weight of the edge between them was incremented by 1.
  - The resulting weighted adjacency list was stored in the current file.


***

#### File: `Entities_related_to_Stake_Addresses__Heuristic2__Cardano_TXs_All__<timestamp>`

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

#### File: `epochArray_rawAddresses__Cardano_TXs_All__<timestamp>`

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

## File: `epochArray_ByronAddresses__Cardano_TXs_All__<timestamp>`

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

#### File: `epochArray_ShelleyAddresses__Cardano_TXs_All__<timestamp>`

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

#### File: `epochArray_delegationAddresses__Cardano_TXs_All__<timestamp>`

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

#### File: `graphEdgesArrayList_Heuristic1noSC_LinkToALLAddressesInTX__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file_2D` method was used to save the adjacency list (2D array) to this text file. This method writes the array into a text file in JSON-like format.
- **Loading**: The `load_file_to_array_2D` method can be used to load the adjacency list back into memory for further analysis.

- **Generation Method**:
  - The edges were generated by linking each address in a transaction to all other addresses in the same transaction.
  - The `merge_graphEdges` function was used to combine multiple adjacency lists into the final merged graph.

***

#### File: `clusteringArrayList_Heuristic1noSC__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a text file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.


***


#### File: `clusteringArrayList_Heuristic2__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a text file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.

***

#### File: `clusteringArrayList_Heuristic1noSC_AND_Heuristic2__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file` method was used to save the clustering array to this text file. This method writes the array into a text file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for further analysis. This method reads the text file and converts it into a NumPy array.


- **Generation Method**:
  - The clustering array was generated by remapping parent cluster IDs from the `parents_merged` array using the `remapClusterIds` function.
  - Clustering IDs were assigned sequentially to ensure compact and unique cluster identification.



***

#### File: `activeAddressesPerDayList__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file` method was used to save the array to this text file. This method writes the array into a text file in JSON-like format.
- **Loading**: The `load_file_to_array` method was used to load the data back into an array for processing. This method reads the text file and converts it into a NumPy array.

***

#### File: `activeEntitiesPerDayList__Cardano_TXs_All__<timestamp>`

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
- **Saving**: The `store_array_to_file` method was used to save the array to this text file. This method writes the array into a text file in JSON-like format.
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


#### File: `Num_Delegator_addresses_per_epoch__Cardano_TXs_All__<timestamp>`

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

#### File: `Num_Delegator_entities_per_epoch__Cardano_TXs_All__<timestamp>`

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

#### File: `Num_Rewarder_addresses_per_epoch__Cardano_TXs_All__<timestamp>`

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

#### File: `Num_Rewarder_entities_per_epoch__Cardano_TXs_All__<timestamp>`

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
| `Num_Delegator_addresses_per_epoch__<timestamp>`     | Count of delegator addresses per epoch | 1 (Count)            | `int`     | `1000\n1200\n1100\n...`      |
| `Num_Delegator_entities_per_epoch__<timestamp>`      | Count of delegator entities per epoch | 1 (Count)            | `int`     | `950\n1150\n1050\n...`       |
| `Num_Rewarder_addresses_per_epoch__<timestamp>`      | Count of rewarder addresses per epoch | 1 (Count)            | `int`     | `800\n900\n850\n...`         |
| `Num_Rewarder_entities_per_epoch__<timestamp>`       | Count of rewarder entities per epoch  | 1 (Count)            | `int`     | `780\n870\n820\n...`         |


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




