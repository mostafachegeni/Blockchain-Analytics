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
