# Blockchain-Analytics

***

# List of `code` Files

Below is an overview of the `code` files included in the **code** folder. Detailed descriptions of the methods are provided in the relevant papers. The analysis steps are well-documented within the code through comments and self-descriptive variable names to facilitate understanding. Additional documentation for these `code` files is presented in the `doc` folder.

### File Descriptions

- **`1_Cardano_Analysis__Address_Entity_Level`**  
  Contains analysis at the address/entity level on the Cardano blockchain, including:
  - Wealth distribution
  - Entity size distribution
  - Active delegators/rewarders
  - Entity wealth, delegation, and size
  - Number of entity-level transactions

- **`2_Cardano_clustering_load_from_file`**  
  Performs analysis at the address/entity level, covering:
  - Distribution of fungible tokens (FT) and non-fungible tokens (NFT) mints
  - Balances of FT/NFTs
  - Entity balances
  - Count of new Byron/Shelley/Stake addresses
  - Number of new addresses/entities
  - Active addresses/entities

- **`3_Cardano_Clustering_Merge_Heuristics`**  
  Merges parent arrays from the algorithm to generate final clustering results.

- **`4_Cardano_CreateAddrsNetworkGraph`**  
  Creates an address network graph for Cardano and identifies the largest connected components (superclusters).

- **`5_Cardano_CommunityDetection`**  
  Analyzes the largest connected components (superclusters) in the address network of Cardano and performs community detection.

- **`6_pool_stake_reward_analysis`**  
  Analyzes:
  - Number of active/rewarded pools
  - Gini index of pool stakes
  - Gini index of entity wealth
  - Number of pools per address/entity

- **`7_heuristicbsed_Clustering_script`**  
  Clusters Cardano addresses to:
  - Determine parent arrays
  - Calculate balances
  - Obtain clustering arrays

- **`8_Extract_Cardano_Addresses`**  
  Extracts addresses from Cardano transactions.

- **`9_entity_analysis`**  
  Performs entity-level analysis on:
  - Entity size-wealth relationships
  - Entity size-delegation relationships
  - Entity wealth-delegation dynamics

- **`10_entity_balance_TxVolume`**  
  Analyzes:
  - Entities' balances (stored in `YuZhang_Cardano_Balances_Entities`)
  - Entities' transaction volume (stored in `YuZhang_Cardano_TX_Vols_Entities__PICKLE`)

- **`11_Untangling`**  
  Contains the code for the Cardano untangling paper.

- **`12_Find_Low_Reward_Pools`**  
  Conducts analysis to identify low-reward pools.

- **`13_staking_dynamics`**  
  Analyzes staking dynamics on Cardano, including:
  - Wealth distribution
  - Delegation and rewards distribution among entities and/or pools

- **`14_Velocity`**  
  Analyzes the distribution of holding ADA.


***

Each ‘code’ file consists of multiple cells. Almost for all cells, there is a comment at top, documenting what analysis/operation the cell does. Also, in each cell, whenever necessary, the sub-operations are commented to explain their activity and give enough information about the details of the steps taken in the code. Additionally, the variable names have been chosen in a way to be well-self-descriptive, helping to better understand the logic of the code. 


***

### Detailed Description of CSV File Columns for Cardano Transaction Data

Below is a comprehensive description of the columns present in the various CSV files (`cardano_TXs_All_MAs_?.csv`, `cardano_TXs_MAs_?.csv`, `cardano_TXs_NFTs_?.csv`, `cardano_TXs_?.csv`).


#### **1. Common Columns Across All Files**
- **`TX_ID`**:
  - Unique identifier of a transaction.
- **`BLOCK_TIME`**:
  - Timestamp of the block in which the transaction was included.
- **`EPOCH_NO`**:
  - The epoch number associated with the block containing the transaction.


#### **2. Specific Columns by File Type**

##### **`cardano_TXs_All_MAs_?.csv`**
- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`TX_FEE`**:
    - Transaction fee paid by the sender, in Lovelace.
  - **`TX_WITHDRAWAL`**:
    - Details of ADA withdrawals, if any, in the format:
      - `TX_ID:Withdrawal_Amount`
    - Semicolon-separated if multiple withdrawals exist.
  - **`TX_MINT_MAs`**:
    - Details of minted multi-assets (MA), including fungible tokens (FTs) and non-fungible tokens (NFTs).
      - Format: `TX_ID:Mint_Details`
      - Mint details can include attributes such as quantity, policy ID, and asset fingerprint.
    - Semicolon-separated for multiple minting events.
  - **`INPUT_MAs`**:
    - Multi-asset (MA) inputs, formatted as:
      - `Input_ID:Asset_Details`
      - Asset details can include name, policy, fingerprint, and quantity.
    - Semicolon-separated for multiple inputs.
  - **`OUTPUT_MAs`**:
    - Multi-asset (MA) outputs, formatted as:
      - `Output_ID:Asset_Details`
    - Semicolon-separated for multiple outputs.
  - **`INPUTs`**:
    - Detailed inputs of the transaction, formatted as:
      - `Input_ID,Ref_TX_ID,Ref_Output_Index,Ref_Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr,Block_Time`
    - Details:
      - `Input_ID`: Unique identifier for the input.
      - `Ref_TX_ID`: Transaction ID of the referenced output.
      - `Ref_Output_Index`: Output index in the referenced transaction.
      - `Ref_Output_ID`: Output ID in the referenced transaction.
      - `Raw_Address`: Raw address of the input.
      - `Stake_Addr_ID`: Stake address ID, if applicable.
      - `Value`: ADA value of the input.
      - `Has_Script`: Indicates if the address is a script (1 = true, 0 = false).
      - `Payment_Cred`: Payment credential hash.
      - `Stake_Addr`: Stake address hash, if applicable.
      - `Block_Time`: Time of the block containing the referenced output.
    - Semicolon-separated for multiple inputs.
  - **`OUTPUTs`**:
    - Detailed outputs of the transaction, formatted as:
      - `Output_ID,Raw_Address,Stake_Addr_ID,Value,Has_Script,Payment_Cred,Stake_Addr`
    - Details:
      - `Output_ID`: Unique identifier for the output.
      - `Raw_Address`: Raw address of the output.
      - `Stake_Addr_ID`: Stake address ID, if applicable.
      - `Value`: ADA value of the output.
      - `Has_Script`: Indicates if the address is a script (1 = true, 0 = false).
      - `Payment_Cred`: Payment credential hash.
      - `Stake_Addr`: Stake address hash, if applicable.
    - Semicolon-separated for multiple outputs.


##### **`cardano_TXs_MAs_?.csv`**
- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`TX_INPUT_MAs`**:
    - Multi-asset (MA) inputs for the transaction, formatted as:
      - `Input_ID:MA_Details`
      - MA details include attributes like name, policy, and fingerprint.
    - Semicolon-separated for multiple MA inputs.
  - **`TX_OUTPUT_MAs`**:
    - Multi-asset (MA) outputs for the transaction, formatted similarly to `TX_INPUT_MAs`.
  - **`MINT_NFTs`**:
    - Details of NFTs minted in the transaction, formatted as:
      - `MA_ID,Name,Fingerprint,Policy,Quantity,Mints_Count`
      - Details:
        - `MA_ID`: Multi-asset ID.
        - `Name`: Asset name.
        - `Fingerprint`: Unique asset fingerprint.
        - `Policy`: Policy ID governing the minting.
        - `Quantity`: Quantity of the asset minted.
        - `Mints_Count`: Total number of minting events for this asset.
    - Semicolon-separated for multiple NFTs.
  - **`MINT_FTs`**:
    - Details of FTs (fungible tokens) minted, formatted similarly to `MINT_NFTs`.


##### **`cardano_TXs_NFTs_?.csv`**
- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`NFTs`**:
    - Details of NFTs in the transaction, formatted as:
      - `MA_ID,Name,Fingerprint,Policy,Quantity,Mints_Count`
    - Semicolon-separated for multiple NFTs.
  - **`FTs`**:
    - Details of FTs in the transaction, formatted similarly to `NFTs`.


##### **`cardano_TXs_?.csv`**
- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`INPUTs`**:
    - Detailed inputs, formatted as in `cardano_TXs_All_MAs_?.csv`.
  - **`OUTPUTs`**:
    - Detailed outputs, formatted as in `cardano_TXs_All_MAs_?.csv`.



#### **Input File Details: `cardano_pools_4.csv`**
- **File Type**: CSV
- **Delimiter**: `|`
- **Columns**:
  - **`EPOCH`**:
    - Type: Integer
    - Description: Epoch number.
  - **`POOL_ID`**:
    - Type: Integer
    - Description: Unique identifier of the staking pool.
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
  - **`DELEGATORs`**:
    - Type: String
    - Description: Semicolon-separated list of delegator details. Each entry contains:
      - `Delegator_ID,Stake_Amount,Delegator_Stake_Addr`
  - **`REWARDERs`**:
    - Type: String
    - Description: Semicolon-separated list of rewarder details. Each entry contains:
      - `Rewarder_ID,Reward_Amount,Rewarder_Stake_Addr`



#### Notes on Delimiters and Encoding:
1. **Semicolon (`;`)**:
   - Used to separate multiple entries within a column.
2. **Comma (`,`)**:
   - Used to separate fields within a single entry.
3. **Colon (`:`)**:
   - Used in specific fields (e.g., `TX_MINT_MAs`, `TX_INPUT_MAs`, `TX_OUTPUT_MAs`) to separate transaction ID from associated details.

#### Use Cases:
- **`cardano_TXs_All_MAs_?.csv`**:
  - Comprehensive analysis of transactions, including ADA, NFTs, and FTs.
- **`cardano_TXs_MAs_?.csv`**:
  - Focused analysis of multi-asset transactions.
- **`cardano_TXs_NFTs_?.csv`**:
  - Specialized analysis of NFT transactions.
- **`cardano_TXs_?.csv`**:
  - General transaction data without multi-asset details.

***




