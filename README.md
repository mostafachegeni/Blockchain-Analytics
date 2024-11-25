# Blockchain-Analytics

This repository contains code for blockchain analysis of Cardano transactions. The Jupyter notebooks are stored in the `code` directory, with corresponding `.md` files in the `doc` directory that explain the functionality of each notebook. Detailed descriptions of the methods can be found in the relevant papers.

The analysis steps are thoroughly documented within the code using comments and self-explanatory variable names to enhance understanding. Each `code` file is composed of multiple cells, with most cells preceded by comments that describe their purpose and operations. Where necessary, sub-operations within the cells are also commented to provide detailed explanations of the steps taken. Variable names are thoughtfully chosen to be self-descriptive, helping users easily follow the logic of the code.

Most of the analyses are performed on CSV files generated by querying the Cardano PostgreSQL database. For reference, the schema of this database is detailed in the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md). 

The format of these CSV files and the SQL queries used to generate them are provided in the `SQL_CSV_Files.md` file. Additionally, the format of the intermediate and final data files produced by these codes is explained in `Files_Format.md`.

Below, you will find a list of notebooks along with a brief overview of each.

# List of `code` Files

- **`1_Cardano_HeuristicBased_Clustering.ipynb`**: The code processes Cardano transactions, applies two heuristics to cluster addresses into entities, and stores the clustering results in an array. Additionally, it generates and saves the edge list for the address network graph.

- **`2_Cardano_Address_Entity_Analysis.ipynb`**: The code performs the following analyses:
    - Calculate the number of members in each cluster (based on heuristic 1 and/or 2)
    - Fitting power-law distributions to the distribution of cluster sizes
    - Finding the corresponding "Entity ID" for Each "Stake Address" (Delegation Address) based on Heuristic 2
    - Calculate balances of entities
    - Calculate number of active delegators/rewarders (entities) per epoch
    - Calculate number of active delegators (Stake Addresses/Entities) per epoch
    - Calculate number of new entities over time
    - Calculate number of new "Byron", "Shelley", and "Stake" addresses over time
    - Calculate number of new addresses per day
    - Calculate numbe rof active Users/Entities per day
    - Calculate list of NFT/FT/ADA transactions" over time at an entity-level
    - Calculate distribution of NFTs/ADAs owned by entities
    - Calculate distribution of NFTs/FTs minted by entities
    - Calculate entities' daily balances
    - Calculate entities' daily transaction volume
    - Calculate distribution of Ada holding time
    - PLOT: Entity Stake Delegation (ADA) vs. Entity Wealth (ADA)
    - PLOT: Entity Wealth (ADA) vs. Entity Size
    - PLOT: Entity Stake Delegation (ADA) vs. Entity Size
    - PLOT: Entity Stake Delegation (ADA) vs. Entity Wealth (ADA)
    - PLOT: Number of new entities per day
    - PLOT: Number of new addresses per day
    - PLOT: Result of active addresses/entities
    - PLOT: ADA hodling time distribution
    - PLOT: compare ADA velocity time calculated with random sampling rates 100% and 30%
    - PLOT: Number of owned NFTs/ADAs per entity
    - PLOT: Distribution of minted NFTs/FTs per Entity
    - PLOT: fitting a power-law distributions to the distribution of minted NFTs/FTs

- **`3_Cardano_CreateAddrsNetworkGraph.ipynb`**: The code creates an address network graph for Cardano addresses and identifies the largest connected components (superclusters). It also analyzes the largest connected components (superclusters) in the address network of Cardano and performs community detection (label propagation).


- **`4_Cardano_Staking_Dynamics.ipynb`**; The code performs the following analyses:
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



