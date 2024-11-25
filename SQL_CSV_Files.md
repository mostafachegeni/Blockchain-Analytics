
#### **`cardano_TXs_All_MAs_<filenumber>.csv`**


This CSV file contains detailed information on Cardano transactions along with metadata about withdrawals, minted assets, inputs, and outputs.


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: The file includes:
  - Block and transaction details (e.g., epoch number, transaction fee)
  - Aggregated metadata for withdrawals, minted assets, and transaction inputs/outputs
  - Details about metadata assets (MAs) involved in inputs, outputs, and minting events
  - Detailed transaction inputs and outputs


### Column Details

#### 1. **EPOCH_NO**
   - **Type**: Integer
   - **Description**: The epoch number when the transaction occurred.

#### 2. **BLOCK_TIME**
   - **Type**: Timestamp
   - **Description**: The timestamp of the block containing the transaction.

#### 3. **TX_ID**
   - **Type**: Integer
   - **Description**: Unique identifier for the transaction.

#### 4. **TX_FEE**
   - **Type**: Integer
   - **Description**: Fee paid for the transaction in Lovelaces (smallest unit of ADA).

#### 5. **TX_WITHDRAWAL**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of withdrawals in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `tx_id`: Transaction ID
     - `TX_ALL_WITHDRAWALs`: Metadata for all withdrawals in the transaction (comma-separated fields):
       - `withdrawal.id`: Withdrawal ID
       - `withdrawal.addr_id`: Address ID
       - `withdrawal.amount`: Withdrawal amount

#### 6. **TX_MINT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of metadata assets (MAs) minted in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `tx_id`: Transaction ID
     - `TX_ALL_MINT_MAs`: Metadata for all minted assets in the transaction (comma-separated fields):
       - `ma_tx_mint.id`: Minting ID
       - `ma_tx_mint.ident`: Unique identifier of the minted asset
       - `ma_tx_mint.quantity`: Quantity of the asset minted

#### 7. **INPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Metadata assets (MAs) associated with transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `tx_in.id`: Input ID
     - `TXOUT_ALL_MAs`: Metadata for the output referenced by this input (comma-separated fields):
       - `ma_tx_out.id`: Output MA ID
       - `ma_tx_out.ident`: MA identifier
       - `ma_tx_out.quantity`: MA quantity

#### 8. **OUTPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Metadata assets (MAs) associated with transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `tx_out.id`: Output ID
     - `TXOUT_ALL_MAs`: Metadata for this output (comma-separated fields):
       - `ma_tx_out.id`: Output MA ID
       - `ma_tx_out.ident`: MA identifier
       - `ma_tx_out.quantity`: MA quantity

#### 9. **INPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each input contains:
     - `tx_in.id`: Input ID
     - `tx_in.tx_out_id`: Referenced transaction output ID
     - `tx_in.tx_out_index`: Output index in the referenced transaction
     - `REF_OUT.id`: Referenced output ID
     - `REF_OUT.address_raw`: Raw address of the referenced output
     - `REF_OUT.stake_address_id`: Stake address ID of the referenced output
     - `REF_OUT.value`: Value of the referenced output
     - `REF_OUT.address_has_script`: Indicates if the address has a script
     - `REF_OUT.payment_cred`: Payment credential of the referenced output
     - `stake_address_REF_OUT.hash_raw`: Stake address hash of the referenced output
     - `block_REF_OUT.time`: Block time of the referenced transaction

#### 10. **OUTPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each output contains:
     - `tx_out.id`: Output ID
     - `tx_out.address_raw`: Raw address of the output
     - `tx_out.stake_address_id`: Stake address ID of the output
     - `tx_out.value`: Value of the output
     - `tx_out.address_has_script`: Indicates if the address has a script
     - `tx_out.payment_cred`: Payment credential of the output
     - `stake_address_main.hash_raw`: Stake address hash of the output


### Notes

- **Grouping**: Data is grouped by `TX_ID`, `EPOCH_NO`, `BLOCK_TIME`, and `TX_FEE`.
- **Joins**: Combines data from tables:
  - `tx`, `block`, `withdrawal`, `ma_tx_mint`, `ma_tx_out`, `tx_in`, `tx_out`, and `stake_address`.
- **Order**: Results are ordered by `TX_ID`, `EPOCH_NO`, `BLOCK_TIME`, and `TX_FEE`.

For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).


### Query:
```sql
# Create table Withdrawals:
cexplorer=# CREATE TABLE TBL_WITHDRAWALs AS 
                SELECT  withdrawal.tx_id as tx_id, 
                        STRING_AGG(distinct concat( withdrawal.id,        ',', 
                                                    withdrawal.addr_id,   ',', 
                                                    withdrawal.amount,    ',' 
                        ), E':') as "TX_ALL_WITHDRAWALs" 
                    FROM withdrawal 
                    GROUP BY withdrawal.tx_id;


# Create table MA mints:
cexplorer=# CREATE TABLE TBL_TX_MINT_MAs AS 
                SELECT  ma_tx_mint.tx_id as tx_id, 
                        STRING_AGG(distinct concat( ma_tx_mint.id,        ',', 
                                                    ma_tx_mint.ident,     ',', 
                                                    ma_tx_mint.quantity,  ','            
                        ), E':') as "TX_ALL_MINT_MAs" 
                    FROM ma_tx_mint 
                    GROUP BY ma_tx_mint.tx_id;


# Create table MA outputs:
cexplorer=# CREATE TABLE TBL_TXOUT_MAs AS 
                SELECT  ma_tx_out.tx_out_id as tx_out_id, 
                        STRING_AGG(distinct concat( ma_tx_out.id,         ',', 
                                                    ma_tx_out.ident,      ',', 
                                                    ma_tx_out.quantity,   ','     
                        ), E':') as "TXOUT_ALL_MAs" 
                    FROM ma_tx_out 
                    GROUP BY ma_tx_out.tx_out_id;



# MAs transacted in the network:
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer  
\copy ( 
        SELECT  block_main.epoch_no                                                                                            as "EPOCH_NO", 
                block_main.time                                                                                                as "BLOCK_TIME", 
                tx_main.id                                                                                                     as "TX_ID", 
                tx_main.fee                                                                                                    as "TX_FEE", 
                STRING_AGG(distinct concat( tx_main.id,      ':', TBL_WITHDRAWALs_main."TX_ALL_WITHDRAWALs", ''       ), E';') as "TX_WITHDRAWAL", 
                STRING_AGG(distinct concat( tx_main.id,      ':', TBL_TX_MINT_MAs_main."TX_ALL_MINT_MAs",    ''       ), E';') as "TX_MINT_MAs", 
                STRING_AGG(distinct concat( tx_in_main.id,   ':', TBL_TXOUT_MAs_REF_OUT."TXOUT_ALL_MAs",     ''       ), E';') as "INPUT_MAs", 
                STRING_AGG(distinct concat( tx_out_main.id,  ':', TBL_TXOUT_MAs_main."TXOUT_ALL_MAs",        ''       ), E';') as "OUTPUT_MAs", 
                STRING_AGG(distinct concat( tx_in_main.id,                                                   ',',  
                                            tx_in_main.tx_out_id,                                            ',',  
                                            tx_in_main.tx_out_index,                                         ',',  
                                            REF_OUT.id,                                                      ',',  
                                            REF_OUT.address_raw,                                             ',',  
                                            REF_OUT.stake_address_id,                                        ',',  
                                            REF_OUT.value,                                                   ',',  
                                            REF_OUT.address_has_script,                                      ',',  
                                            REF_OUT.payment_cred,                                            ',',  
                                            stake_address_REF_OUT.hash_raw,                                  ',',  
                                            block_REF_OUT.time,                                              ','      ), E';') as "INPUTs", 
                STRING_AGG(distinct concat( tx_out_main.id,                                                  ',',  
                                            tx_out_main.address_raw,                                         ',',  
                                            tx_out_main.stake_address_id,                                    ',',  
                                            tx_out_main.value,                                               ',',  
                                            tx_out_main.address_has_script,                                  ',',  
                                            tx_out_main.payment_cred,                                        ',',  
                                            stake_address_main.hash_raw,                                     ','      ), E';') as "OUTPUTs" 
            FROM    tx                           as tx_main 
                       LEFT JOIN block           as block_main              ON tx_main.block_id              = block_main.id 
                       LEFT JOIN TBL_WITHDRAWALs as TBL_WITHDRAWALs_main    ON tx_main.id                    = TBL_WITHDRAWALs_main.tx_id 
                       LEFT JOIN TBL_TX_MINT_MAs as TBL_TX_MINT_MAs_main    ON tx_main.id                    = TBL_TX_MINT_MAs_main.tx_id 
                       LEFT JOIN tx_out          as tx_out_main             ON tx_main.id                    = tx_out_main.tx_id 
                       LEFT JOIN stake_address   as stake_address_main      ON tx_out_main.stake_address_id  = stake_address_main.id 
                       LEFT JOIN TBL_TXOUT_MAs   as TBL_TXOUT_MAs_main      ON tx_out_main.id                = TBL_TXOUT_MAs_main.tx_out_id 
                       LEFT JOIN tx_in           as tx_in_main              ON tx_main.id                    = tx_in_main.tx_in_id 
                       LEFT JOIN tx_out          as REF_OUT                 ON tx_in_main.tx_out_id          = REF_OUT.tx_id 
                                                                           AND tx_in_main.tx_out_index       = REF_OUT.index 
                       LEFT JOIN stake_address   as stake_address_REF_OUT   ON REF_OUT.stake_address_id      = stake_address_REF_OUT.id 
                       LEFT JOIN TBL_TXOUT_MAs   as TBL_TXOUT_MAs_REF_OUT   ON REF_OUT.id                    = TBL_TXOUT_MAs_REF_OUT.tx_out_id 
                       LEFT JOIN tx              as tx_REF_OUT              ON REF_OUT.tx_id                 = tx_REF_OUT.id 
                       LEFT JOIN block           as block_REF_OUT           ON tx_REF_OUT.block_id           = block_REF_OUT.id 
            WHERE   tx_main.id BETWEEN (80000001) AND (90000000) 
            GROUP BY "TX_ID", "EPOCH_NO", "BLOCK_TIME", "TX_FEE" 
            ORDER BY "TX_ID", "EPOCH_NO", "BLOCK_TIME", "TX_FEE" 
) TO '/cardano_TXs_All_MAs_<file_number>.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_

```

***


#### **`cardano_TXs_<filenumber>.csv`**

This CSV file contains transaction data from the Cardano blockchain database, joining multiple tables to provide comprehensive details about transactions, inputs, outputs, and associated metadata. It aggregates information about NFTs, fungible tokens (FTs), inputs, and outputs for each transaction.

- **File Type:** CSV (Comma-Separated Values)
- **Delimiter:** `|`
- **Header:** The file includes a header row.


## **Columns**

### **1. TX_ID**
- **Type:** Integer
- **Description:** Unique identifier of the transaction.

### **2. BLOCK_TIME**
- **Type:** Timestamp
- **Description:** The timestamp of the block containing the transaction.

### **3. EPOCH_NO**
- **Type:** Integer
- **Description:** Epoch number in which the transaction was included.

### **4. NFTs**
- **Type:** String
- **Description:** Aggregated details of non-fungible tokens (NFTs) associated with the transaction, separated by `;`. Each NFT is represented by:
  - `MA_ID`: Unique ID of the NFT.
  - `MA_NAME`: Name of the NFT.
  - `MA_FINGERPRINT`: Unique fingerprint of the NFT.
  - `MA_POLICY`: Policy ID under which the NFT was minted.
  - `MA_TOTAL_QUANTITY`: Total quantity of the NFT.
  - `MA_TOTAL_MINTS_COUNT`: Total mints count of the NFT.

### **5. FTs**
- **Type:** String
- **Description:** Aggregated details of fungible tokens (FTs) associated with the transaction, separated by `;`. Each FT is represented by:
  - `MA_ID`: Unique ID of the FT.
  - `MA_NAME`: Name of the FT.
  - `MA_FINGERPRINT`: Unique fingerprint of the FT.
  - `MA_POLICY`: Policy ID under which the FT was minted.
  - `MA_TOTAL_QUANTITY`: Total quantity of the FT.
  - `MA_TOTAL_MINTS_COUNT`: Total mints count of the FT.

### **6. INPUTs**
- **Type:** String
- **Description:** Aggregated details of transaction inputs, separated by `;`. Each input is represented by:
  - `INPUT_ID`: ID of the input.
  - `INPUT_REFTX_ID`: Reference transaction ID for the input.
  - `INPUT_REFTX_OUTINDX`: Reference transaction output index.
  - `INPUT_REFOUT_ID`: ID of the referenced output.
  - `INPUT_REFOUT_RAWADDR`: Raw address of the referenced output.
  - `INPUT_REFOUT_STAKE_ADDR_ID`: Stake address ID of the referenced output.
  - `INPUT_REFOUT_VALUE`: Value of the referenced output.
  - `INPUT_REFOUT_ADDR_HAS_SCRIPT`: Boolean indicating if the referenced address has a script.
  - `INPUT_REFOUT_PAYMENT_CRED`: Payment credential of the referenced output.
  - `INPUT_REFOUT_STAKE_ADDR`: Raw stake address of the referenced output.

### **7. OUTPUTs**
- **Type:** String
- **Description:** Aggregated details of transaction outputs, separated by `;`. Each output is represented by:
  - `OUTPUT_ID`: ID of the output.
  - `OUTPUT_RAWADDR`: Raw address of the output.
  - `OUTPUT_STAKE_ADDR_ID`: Stake address ID associated with the output.
  - `OUTPUT_VALUE`: Value of the output.
  - `OUTPUT_ADDR_HAS_SCRIPT`: Boolean indicating if the output address has a script.
  - `OUTPUT_PAYMENT_CRED`: Payment credential of the output.
  - `OUTPUT_STAKE_ADDR`: Raw stake address associated with the output.



### Notes
- All multi-row relationships (NFTs, FTs, inputs, and outputs) are aggregated using `STRING_AGG` with a semicolon (`;`) delimiter.
- **Grouping:** Data is grouped by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.
- **Ordering:** The results are ordered by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.


For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).


### Query
```sql
# Create view VIEW_NFTs:
cexplorer=# CREATE OR REPLACE VIEW VIEW_NFTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE  tx_metadata.key IN (721) 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID";



# Create view VIEW_FTs:
cexplorer=# CREATE OR REPLACE VIEW VIEW_FTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE  tx_metadata.key NOT IN (721) 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID"; 




# Dump all transactions in the blockchain:
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as ( 
        WITH TXs_INs_OUTs_StakeAddr as ( 
            WITH TXs_INs_OUTs as ( 
                SELECT  tx.id                       as "TX_ID", 
                        block.time                  as "BLOCK_TIME", 
                        block.epoch_no              as "EPOCH_NO", 
                        tx_in.tx_in_id              as "INPUT_TXID", 
                        tx_out.tx_id                as "OUTPUT_TXID", 
                        tx_in.id                    as "INPUT_ID",  
                        tx_out.id                   as "OUTPUT_ID", 
                        tx_in.tx_out_id             as "INPUT_REFTX_ID", 
                        tx_in.tx_out_index          as "INPUT_REFTX_OUTINDX", 
                        tx_out.address_raw          as "OUTPUT_RAWADDR", 
                        tx_out.value                as "OUTPUT_VALUE", 
                        tx_out.address_has_script   as "OUTPUT_ADDR_HAS_SCRIPT", 
                        tx_out.payment_cred         as "OUTPUT_PAYMENT_CRED", 
                        tx_out.stake_address_id     as "OUTPUT_STAKE_ADDR_ID" 
                    FROM    tx LEFT JOIN tx_in   ON tx.id       = tx_in.tx_in_id 
                               LEFT JOIN tx_out  ON tx.id       = tx_out.tx_id 
                               LEFT JOIN block   ON tx.block_id = block.id 
                    WHERE   tx.id BETWEEN (50000001) and (60000000) 
            ) 
            SELECT  TXs_INs_OUTs.*, 
                    stake_address.hash_raw       as "OUTPUT_STAKE_ADDR",
                    REF_OUT.tx_id                as "INPUT_REFOUT_TXID", 
                    REF_OUT.index                as "INPUT_REFOUT_INDEX", 
                    REF_OUT.id                   as "INPUT_REFOUT_ID", 
                    REF_OUT.address_raw          as "INPUT_REFOUT_RAWADDR", 
                    REF_OUT.value                as "INPUT_REFOUT_VALUE", 
                    REF_OUT.address_has_script   as "INPUT_REFOUT_ADDR_HAS_SCRIPT", 
                    REF_OUT.payment_cred         as "INPUT_REFOUT_PAYMENT_CRED", 
                    REF_OUT.stake_address_id     as "INPUT_REFOUT_STAKE_ADDR_ID" 
                FROM    TXs_INs_OUTs LEFT JOIN stake_address   ON     TXs_INs_OUTs."OUTPUT_STAKE_ADDR_ID" = stake_address.id 
                                     LEFT JOIN tx_out REF_OUT  ON     TXs_INs_OUTs."INPUT_REFTX_ID"       = REF_OUT.tx_id 
                                                                  AND TXs_INs_OUTs."INPUT_REFTX_OUTINDX"  = REF_OUT.index 
        ) 
        SELECT  TXs_INs_OUTs_StakeAddr.*, 
                 stake_address.hash_raw      as "INPUT_REFOUT_STAKE_ADDR" 
            FROM  TXs_INs_OUTs_StakeAddr LEFT JOIN stake_address  ON   TXs_INs_OUTs_StakeAddr."INPUT_REFOUT_STAKE_ADDR_ID" = stake_address.id 
    ) 
    SELECT  things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO", 
            STRING_AGG(distinct concat(VIEW_NFTs."MA_ID",                  ',', VIEW_NFTs."MA_NAME",                 ',', VIEW_NFTs."MA_FINGERPRINT",      ',', VIEW_NFTs."MA_POLICY",                  ',', 
                                       VIEW_NFTs."MA_TOTAL_QUANTITY",      ',', VIEW_NFTs."MA_TOTAL_MINTS_COUNT",                                                                                       ','     ), E';') as "NFTs", 
            STRING_AGG(distinct concat(VIEW_FTs."MA_ID",                   ',', VIEW_FTs."MA_NAME",                  ',', VIEW_FTs."MA_FINGERPRINT",       ',', VIEW_FTs."MA_POLICY",                   ',', 
                                       VIEW_FTs."MA_TOTAL_QUANTITY",       ',', VIEW_FTs."MA_TOTAL_MINTS_COUNT",                                                                                        ','     ), E';') as "FTs", 
            STRING_AGG(distinct concat(things."INPUT_ID",                  ',', things."INPUT_REFTX_ID",             ',', things."INPUT_REFTX_OUTINDX",    ',', things."INPUT_REFOUT_ID",               ',', 
                                       things."INPUT_REFOUT_RAWADDR",      ',', things."INPUT_REFOUT_STAKE_ADDR_ID", ',', things."INPUT_REFOUT_VALUE",     ',', things."INPUT_REFOUT_ADDR_HAS_SCRIPT",  ',', 
                                       things."INPUT_REFOUT_PAYMENT_CRED", ',', things."INPUT_REFOUT_STAKE_ADDR",                                                                                       ','     ), E';') as "INPUTs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ',', things."OUTPUT_RAWADDR",             ',', things."OUTPUT_STAKE_ADDR_ID",   ',', things."OUTPUT_VALUE",                  ',', 
                                       things."OUTPUT_ADDR_HAS_SCRIPT",    ',', things."OUTPUT_PAYMENT_CRED",        ',', things."OUTPUT_STAKE_ADDR",                                                   ','     ), E';') as "OUTPUTs" 
        FROM things 
            LEFT JOIN VIEW_NFTs ON things."TX_ID" = VIEW_NFTs."MA_FIRST_TX_ID" 
            LEFT JOIN VIEW_FTs  ON things."TX_ID" = VIEW_FTs."MA_FIRST_TX_ID" 
        GROUP BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO" 
        ORDER BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO" 
) TO '/cardano_TXs_<filenumber>.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_

```


***

#### **`cardano_TXs_NFTs_<filenumber>.csv`**

This CSV file contains data about Cardano transactions and metadata, including associated fungible and non-fungible tokens (FTs and NFTs), transaction inputs, outputs, and minting events.


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: Provides detailed information on Cardano transactions (`tx`), their inputs and outputs, associated minting data, and metadata for fungible and non-fungible tokens.
- **Scope**: Includes transactions with IDs in the range `50000001` to `60000000`.


### Column Details

#### 1. **TX_ID**
   - **Type**: Integer
   - **Description**: Unique identifier of the transaction.

#### 2. **BLOCK_TIME**
   - **Type**: Timestamp
   - **Description**: Timestamp of the block containing the transaction.

#### 3. **EPOCH_NO**
   - **Type**: Integer
   - **Description**: Epoch number when the transaction occurred.

#### 4. **TX_INPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Details of metadata assets (MAs) associated with transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `INPUT_ID`: Input identifier
     - `INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT`: Metadata associated with the input.

#### 5. **TX_OUTPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Details of metadata assets (MAs) associated with transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `OUTPUT_ID`: Output identifier
     - `OUTPUT_MAs__NAME__POLICY__FINGERPRINT`: Metadata associated with the output.

#### 6. **MINT_NFTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of minted non-fungible tokens (NFTs) in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `MA_ID`: Unique identifier of the token.
     - `MA_NAME`: Name of the NFT.
     - `MA_FINGERPRINT`: Unique fingerprint of the token.
     - `MA_POLICY`: Policy ID of the token.
     - `MA_TOTAL_QUANTITY`: Total quantity of the NFT minted.
     - `MA_TOTAL_MINTS_COUNT`: Total minting events for this NFT.

#### 7. **MINT_FTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of minted fungible tokens (FTs) in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `MA_ID`: Unique identifier of the token.
     - `MA_NAME`: Name of the FT.
     - `MA_FINGERPRINT`: Unique fingerprint of the token.
     - `MA_POLICY`: Policy ID of the token.
     - `MA_TOTAL_QUANTITY`: Total quantity of the FT minted.
     - `MA_TOTAL_MINTS_COUNT`: Total minting events for this FT.

#### 8. **INPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each input contains:
     - `INPUT_ID`: Unique identifier of the input.
     - `INPUT_REFTX_ID`: Referenced transaction ID.
     - `INPUT_REFTX_OUTINDX`: Output index in the referenced transaction.
     - `INPUT_REFOUT_ID`: Referenced output ID.
     - `INPUT_REFOUT_RAWADDR`: Raw address of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR_ID`: Stake address ID of the referenced output.
     - `INPUT_REFOUT_VALUE`: Value of the referenced output.
     - `INPUT_REFOUT_ADDR_HAS_SCRIPT`: Indicates if the address has a script.
     - `INPUT_REFOUT_PAYMENT_CRED`: Payment credential of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR`: Stake address of the referenced output.

#### 9. **OUTPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each output contains:
     - `OUTPUT_ID`: Unique identifier of the output.
     - `OUTPUT_RAWADDR`: Raw address of the output.
     - `OUTPUT_STAKE_ADDR_ID`: Stake address ID of the output.
     - `OUTPUT_VALUE`: Value of the output.
     - `OUTPUT_ADDR_HAS_SCRIPT`: Indicates if the address has a script.
     - `OUTPUT_PAYMENT_CRED`: Payment credential of the output.
     - `OUTPUT_STAKE_ADDR`: Stake address of the output.


### Notes

- **Grouping**: Data is grouped by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.
- **Joins**: The query uses multiple joins to fetch data from:
  - `tx`, `tx_in`, `tx_out`, `block`, `stake_address`, `TBL_TXOUT_NFTs_FTs_MAs`, `TBL_MINT_NFTs`, and `TBL_MINT_FTs`.
- **Order**: The output is ordered by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.

For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).


### Query
```sql
# create table mint of nfts:
cexplorer=# CREATE TABLE TBL_MINT_NFTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE   tx_metadata.key IN (721) 
                AND MINT_TABLE."MA_TOTAL_QUANTITY" = 1 
                AND MINT_TABLE."MA_TOTAL_MINTS_COUNT" = 1 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID";


# create table mint of fts:
cexplorer=# CREATE TABLE TBL_MINT_FTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE  tx_metadata.key NOT IN (721) 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID"; 


# create table MA outputs:
cexplorer=# CREATE TABLE TBL_TXOUT_NFTs_FTs_MAs AS
        WITH things as (
            SELECT  ma_tx_out.tx_out_id              as "OUTPUT_ID", 
                    ma_tx_out.ident                  as "OUTPUT_MULTIASSET_TXOUT_IDENT", 
                    ma_tx_out.quantity               as "OUTPUT_MULTIASSET_TXOUT_QUANTITY", 
                    TBL_MINT_NFTs."MA_NAME"          as "OUTPUT_NFT_NAME", 
                    TBL_MINT_FTs."MA_NAME"           as "OUTPUT_FT_NAME", 
                    multi_asset.name                 as "OUTPUT_MA_NAME", 
                    multi_asset.policy               as "OUTPUT_MA_POLICY", 
                    multi_asset.fingerprint          as "OUTPUT_MA_FINGERPRINT" 
                FROM ma_tx_out 
                    LEFT JOIN TBL_MINT_NFTs  ON  ma_tx_out.ident = TBL_MINT_NFTs."MA_ID" 
                    LEFT JOIN TBL_MINT_FTs   ON  ma_tx_out.ident = TBL_MINT_FTs."MA_ID" 
                    LEFT JOIN multi_asset    ON  ma_tx_out.ident = multi_asset.id 
        )
        SELECT  things."OUTPUT_ID", 
                STRING_AGG(distinct concat(things."OUTPUT_MULTIASSET_TXOUT_IDENT", ',', things."OUTPUT_MULTIASSET_TXOUT_QUANTITY", ',', things."OUTPUT_NFT_NAME",       ',', things."OUTPUT_FT_NAME",  ',', 
                                           things."OUTPUT_MA_NAME",                ',', things."OUTPUT_MA_POLICY",                 ',', things."OUTPUT_MA_FINGERPRINT",                                ','   ), E':') AS "MAs__NAME__POLICY__FINGERPRINT" 
            FROM things 
            GROUP BY things."OUTPUT_ID";



# Dump all TXs: include all NFTs/FTs transacted in the transaction:
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as ( 
        WITH TXs_INs_OUTs as ( 
            SELECT  tx.id                                                      as "TX_ID", 
                    block.time                                                 as "BLOCK_TIME", 
                    block.epoch_no                                             as "EPOCH_NO", 
                    tx_in.tx_in_id                                             as "INPUT_TXID", 
                    tx_out.tx_id                                               as "OUTPUT_TXID", 
                    tx_in.id                                                   as "INPUT_ID",  
                    tx_out.id                                                  as "OUTPUT_ID", 
                    tx_in.tx_out_id                                            as "INPUT_REFTX_ID", 
                    tx_in.tx_out_index                                         as "INPUT_REFTX_OUTINDX", 
                    tx_out.address_raw                                         as "OUTPUT_RAWADDR", 
                    tx_out.value                                               as "OUTPUT_VALUE", 
                    tx_out.address_has_script                                  as "OUTPUT_ADDR_HAS_SCRIPT", 
                    tx_out.payment_cred                                        as "OUTPUT_PAYMENT_CRED", 
                    tx_out.stake_address_id                                    as "OUTPUT_STAKE_ADDR_ID", 
                    stake_address.hash_raw                                     as "OUTPUT_STAKE_ADDR", 
                    TBL_TXOUT_NFTs_FTs_MAs."MAs__NAME__POLICY__FINGERPRINT"    as "OUTPUT_MAs__NAME__POLICY__FINGERPRINT" 
                FROM    tx LEFT JOIN block                   ON tx.block_id             = block.id 
                           LEFT JOIN tx_in                   ON tx.id                   = tx_in.tx_in_id 
                           LEFT JOIN tx_out                  ON tx.id                   = tx_out.tx_id 
                           LEFT JOIN stake_address           ON tx_out.stake_address_id = stake_address.id 
                           LEFT JOIN TBL_TXOUT_NFTs_FTs_MAs  ON tx_out.id               = TBL_TXOUT_NFTs_FTs_MAs."OUTPUT_ID" 
                WHERE   tx.id BETWEEN (50000001) AND (60000000) 
        ) 
        SELECT  TXs_INs_OUTs.*, 
                REF_OUT.tx_id                                              as "INPUT_REFOUT_TXID", 
                REF_OUT.index                                              as "INPUT_REFOUT_INDEX", 
                REF_OUT.id                                                 as "INPUT_REFOUT_ID", 
                REF_OUT.address_raw                                        as "INPUT_REFOUT_RAWADDR", 
                REF_OUT.value                                              as "INPUT_REFOUT_VALUE", 
                REF_OUT.address_has_script                                 as "INPUT_REFOUT_ADDR_HAS_SCRIPT", 
                REF_OUT.payment_cred                                       as "INPUT_REFOUT_PAYMENT_CRED", 
                REF_OUT.stake_address_id                                   as "INPUT_REFOUT_STAKE_ADDR_ID", 
                stake_address.hash_raw                                     as "INPUT_REFOUT_STAKE_ADDR", 
                TBL_TXOUT_NFTs_FTs_MAs."MAs__NAME__POLICY__FINGERPRINT"    as "INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT" 
            FROM    TXs_INs_OUTs LEFT JOIN tx_out REF_OUT                  ON     TXs_INs_OUTs."INPUT_REFTX_ID"       = REF_OUT.tx_id 
                                                                              AND TXs_INs_OUTs."INPUT_REFTX_OUTINDX"  = REF_OUT.index 
                                 LEFT JOIN stake_address                   ON     REF_OUT.stake_address_id            = stake_address.id 
                                 LEFT JOIN TBL_TXOUT_NFTs_FTs_MAs          ON     REF_OUT.id                          = TBL_TXOUT_NFTs_FTs_MAs."OUTPUT_ID" 
    ) 
    SELECT  things."TX_ID", 
            things."BLOCK_TIME", 
            things."EPOCH_NO", 

            STRING_AGG(distinct concat(things."INPUT_ID",                  ':', things."INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT",                                                                                            ''      ), E';') as "TX_INPUT_MAs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ':', things."OUTPUT_MAs__NAME__POLICY__FINGERPRINT",                                                                                                  ''      ), E';') as "TX_OUTPUT_MAs", 

            STRING_AGG(distinct concat(TBL_MINT_NFTs."MA_ID",              ',', TBL_MINT_NFTs."MA_NAME",                      ',', TBL_MINT_NFTs."MA_FINGERPRINT",                  ',', TBL_MINT_NFTs."MA_POLICY",              ',', 
                                       TBL_MINT_NFTs."MA_TOTAL_QUANTITY",  ',', TBL_MINT_NFTs."MA_TOTAL_MINTS_COUNT",                                                                                                            ','     ), E';') as "MINT_NFTs", 
            STRING_AGG(distinct concat(TBL_MINT_FTs."MA_ID",               ',', TBL_MINT_FTs."MA_NAME",                       ',', TBL_MINT_FTs."MA_FINGERPRINT",                   ',', TBL_MINT_FTs."MA_POLICY",               ',', 
                                       TBL_MINT_FTs."MA_TOTAL_QUANTITY",   ',', TBL_MINT_FTs."MA_TOTAL_MINTS_COUNT",                                                                                                             ','     ), E';') as "MINT_FTs", 

            STRING_AGG(distinct concat(things."INPUT_ID",                  ',', things."INPUT_REFTX_ID",                      ',', things."INPUT_REFTX_OUTINDX",                    ',', things."INPUT_REFOUT_ID",               ',', 
                                       things."INPUT_REFOUT_RAWADDR",      ',', things."INPUT_REFOUT_STAKE_ADDR_ID",          ',', things."INPUT_REFOUT_VALUE",                     ',', things."INPUT_REFOUT_ADDR_HAS_SCRIPT",  ',', 
                                       things."INPUT_REFOUT_PAYMENT_CRED", ',', things."INPUT_REFOUT_STAKE_ADDR",                                                                                                                ','     ), E';') as "INPUTs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ',', things."OUTPUT_RAWADDR",                      ',', things."OUTPUT_STAKE_ADDR_ID",                   ',', things."OUTPUT_VALUE",                  ',', 
                                       things."OUTPUT_ADDR_HAS_SCRIPT",    ',', things."OUTPUT_PAYMENT_CRED",                 ',', things."OUTPUT_STAKE_ADDR",                                                                   ','     ), E';') as "OUTPUTs" 
        FROM things 
            LEFT JOIN TBL_MINT_NFTs ON things."TX_ID" = TBL_MINT_NFTs."MA_FIRST_TX_ID" 
            LEFT JOIN TBL_MINT_FTs  ON things."TX_ID" = TBL_MINT_FTs."MA_FIRST_TX_ID" 
        GROUP BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO" 
        ORDER BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO" 
) TO '/cardano_TXs_NFTs_<filenumber>.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_

```


***


#### **`cardano_TXs_Velocity_<filenumber>.csv`**

This CSV file contains information on Cardano transactions, their inputs, outputs, and metadata, focusing on transactions within a specified range (`tx.id` between `50000001` and `60000000`).


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: Contains aggregated details about transaction inputs and outputs, including reference data for input transactions and metadata about stake addresses.
- **Scope**: The data includes:
  - Transaction details (ID, block time, epoch number)
  - Metadata for transaction inputs and outputs
  - Reference information about the blocks and outputs linked to transaction inputs


### Column Details

#### 1. **TX_ID**
   - **Type**: Integer
   - **Description**: Unique identifier of the transaction.

#### 2. **BLOCK_TIME**
   - **Type**: Timestamp
   - **Description**: The timestamp of the block containing the transaction.

#### 3. **EPOCH_NO**
   - **Type**: Integer
   - **Description**: The epoch number during which the transaction occurred.

#### 4. **INPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all inputs for the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `INPUT_ID`: Unique identifier of the input.
     - `INPUT_REFTX_ID`: Referenced transaction ID.
     - `INPUT_REFTX_OUTINDX`: Output index in the referenced transaction.
     - `INPUT_REFOUT_ID`: Referenced output ID.
     - `INPUT_REFOUT_RAWADDR`: Raw address of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR_ID`: Stake address ID of the referenced output.
     - `INPUT_REFOUT_VALUE`: Value of the referenced output.
     - `INPUT_REFOUT_ADDR_HAS_SCRIPT`: Indicates if the referenced address has a script.
     - `INPUT_REFOUT_PAYMENT_CRED`: Payment credential of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR`: Stake address of the referenced output.
     - `INPUT_REFOUT_BLOCK_TIME`: Block timestamp of the referenced transaction.

#### 5. **OUTPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all outputs for the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `OUTPUT_ID`: Unique identifier of the output.
     - `OUTPUT_RAWADDR`: Raw address of the output.
     - `OUTPUT_STAKE_ADDR_ID`: Stake address ID of the output.
     - `OUTPUT_VALUE`: Value of the output.
     - `OUTPUT_ADDR_HAS_SCRIPT`: Indicates if the address has a script.
     - `OUTPUT_PAYMENT_CRED`: Payment credential of the output.
     - `OUTPUT_STAKE_ADDR`: Stake address of the output.


### Notes

- **Grouping**: Data is grouped by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.
- **Joins**:
  - Combines data from `tx`, `tx_in`, `tx_out`, `block`, and `stake_address` tables.
  - Reference output details (`REF_OUT`) and their corresponding block times are included.
- **Order**: Results are ordered by `TX_ID`, `BLOCK_TIME`, and `EPOCH_NO`.


For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).

### Query
```sql
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h 172.23.38.242 -p 5432 -U postgres cexplorer 
\copy ( 
    WITH things as ( 
        WITH TXs_INs_OUTs as ( 
            SELECT  tx.id                                                      as "TX_ID", 
                    block.time                                                 as "BLOCK_TIME", 
                    block.epoch_no                                             as "EPOCH_NO", 
                    tx_in.tx_in_id                                             as "INPUT_TXID", 
                    tx_out.tx_id                                               as "OUTPUT_TXID", 
                    tx_in.id                                                   as "INPUT_ID",  
                    tx_out.id                                                  as "OUTPUT_ID", 
                    tx_in.tx_out_id                                            as "INPUT_REFTX_ID", 
                    tx_in.tx_out_index                                         as "INPUT_REFTX_OUTINDX", 
                    tx_out.address_raw                                         as "OUTPUT_RAWADDR", 
                    tx_out.value                                               as "OUTPUT_VALUE", 
                    tx_out.address_has_script                                  as "OUTPUT_ADDR_HAS_SCRIPT", 
                    tx_out.payment_cred                                        as "OUTPUT_PAYMENT_CRED", 
                    tx_out.stake_address_id                                    as "OUTPUT_STAKE_ADDR_ID", 
                    stake_address.hash_raw                                     as "OUTPUT_STAKE_ADDR" 
                FROM    tx LEFT JOIN block                   ON tx.block_id             = block.id 
                           LEFT JOIN tx_in                   ON tx.id                   = tx_in.tx_in_id 
                           LEFT JOIN tx_out                  ON tx.id                   = tx_out.tx_id 
                           LEFT JOIN stake_address           ON tx_out.stake_address_id = stake_address.id 
                WHERE   tx.id BETWEEN (50000001) AND (60000000) 
        ) 
        SELECT  TXs_INs_OUTs.*, 
                block.time                                                 as "INPUT_REFOUT_BLOCK_TIME", 
                REF_OUT.tx_id                                              as "INPUT_REFOUT_TXID", 
                REF_OUT.index                                              as "INPUT_REFOUT_INDEX", 
                REF_OUT.id                                                 as "INPUT_REFOUT_ID", 
                REF_OUT.address_raw                                        as "INPUT_REFOUT_RAWADDR", 
                REF_OUT.value                                              as "INPUT_REFOUT_VALUE", 
                REF_OUT.address_has_script                                 as "INPUT_REFOUT_ADDR_HAS_SCRIPT", 
                REF_OUT.payment_cred                                       as "INPUT_REFOUT_PAYMENT_CRED", 
                REF_OUT.stake_address_id                                   as "INPUT_REFOUT_STAKE_ADDR_ID", 
                stake_address.hash_raw                                     as "INPUT_REFOUT_STAKE_ADDR" 
            FROM    TXs_INs_OUTs LEFT JOIN tx_out REF_OUT   ON   TXs_INs_OUTs."INPUT_REFTX_ID"       = REF_OUT.tx_id 
                                                             AND TXs_INs_OUTs."INPUT_REFTX_OUTINDX"  = REF_OUT.index 
                                 LEFT JOIN stake_address    ON   REF_OUT.stake_address_id            = stake_address.id 
                                 LEFT JOIN tx               ON   REF_OUT.tx_id                       = tx.id 
                                 LEFT JOIN block            ON   tx.block_id                         = block.id 
    ) 
    SELECT  things."TX_ID", 
            things."BLOCK_TIME", 
            things."EPOCH_NO", 
            STRING_AGG(distinct concat(things."INPUT_ID",                  ',', things."INPUT_REFTX_ID",                      ',', things."INPUT_REFTX_OUTINDX",                    ',', things."INPUT_REFOUT_ID",               ',', 
                                       things."INPUT_REFOUT_RAWADDR",      ',', things."INPUT_REFOUT_STAKE_ADDR_ID",          ',', things."INPUT_REFOUT_VALUE",                     ',', things."INPUT_REFOUT_ADDR_HAS_SCRIPT",  ',', 
                                       things."INPUT_REFOUT_PAYMENT_CRED", ',', things."INPUT_REFOUT_STAKE_ADDR",             ',', things."INPUT_REFOUT_BLOCK_TIME",                                                             ','     ), E';') as "INPUTs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ',', things."OUTPUT_RAWADDR",                      ',', things."OUTPUT_STAKE_ADDR_ID",                   ',', things."OUTPUT_VALUE",                  ',', 
                                       things."OUTPUT_ADDR_HAS_SCRIPT",    ',', things."OUTPUT_PAYMENT_CRED",                 ',', things."OUTPUT_STAKE_ADDR",                                                                   ','     ), E';') as "OUTPUTs" 
        FROM things 
        GROUP BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO" 
        ORDER BY things."TX_ID", things."BLOCK_TIME", things."EPOCH_NO"
) TO '/cardano_TXs_Velocity_<filenumber>.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_

```



***

#### **`cardano_TXs_MAs_<filenumber>.csv`**


This CSV file contains comprehensive information about Cardano transactions, including transaction fees, inputs, outputs, and metadata for fungible and non-fungible tokens (MAs - Metadata Assets). 


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: Includes transaction details (`tx`), their fees, inputs, outputs, minting events, and metadata for fungible and non-fungible tokens.
- **Scope**: Focuses on transactions with IDs in the range `50000001` to `60000000`.


### Column Details

#### 1. **TX_ID**
   - **Type**: Integer
   - **Description**: Unique identifier of the transaction.

#### 2. **TX_FEE**
   - **Type**: Integer
   - **Description**: The fee paid for the transaction, measured in Lovelaces (smallest unit of ADA).

#### 3. **BLOCK_TIME**
   - **Type**: Timestamp
   - **Description**: The timestamp of the block containing the transaction.

#### 4. **EPOCH_NO**
   - **Type**: Integer
   - **Description**: The epoch number when the transaction was processed.

#### 5. **TX_INPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Metadata assets (MAs) associated with transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `INPUT_ID`: Input identifier.
     - `INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT`: Metadata (name, policy, fingerprint) associated with the input.

#### 6. **TX_OUTPUT_MAs**
   - **Type**: String (Aggregated)
   - **Description**: Metadata assets (MAs) associated with transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `OUTPUT_ID`: Output identifier.
     - `OUTPUT_MAs__NAME__POLICY__FINGERPRINT`: Metadata (name, policy, fingerprint) associated with the output.

#### 7. **MINT_NFTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of minted non-fungible tokens (NFTs) in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `MA_ID`: Unique identifier of the NFT.
     - `MA_NAME`: Name of the NFT.
     - `MA_FINGERPRINT`: Unique fingerprint of the NFT.
     - `MA_POLICY`: Policy ID governing the NFT.
     - `MA_TOTAL_QUANTITY`: Total quantity minted.
     - `MA_TOTAL_MINTS_COUNT`: Total number of minting events.

#### 8. **MINT_FTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of minted fungible tokens (FTs) in the transaction.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `MA_ID`: Unique identifier of the FT.
     - `MA_NAME`: Name of the FT.
     - `MA_FINGERPRINT`: Unique fingerprint of the FT.
     - `MA_POLICY`: Policy ID governing the FT.
     - `MA_TOTAL_QUANTITY`: Total quantity minted.
     - `MA_TOTAL_MINTS_COUNT`: Total number of minting events.

#### 9. **INPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction inputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `INPUT_ID`: Unique identifier of the input.
     - `INPUT_REFTX_ID`: Referenced transaction ID.
     - `INPUT_REFTX_OUTINDX`: Output index in the referenced transaction.
     - `INPUT_REFOUT_ID`: Referenced output ID.
     - `INPUT_REFOUT_RAWADDR`: Raw address of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR_ID`: Stake address ID of the referenced output.
     - `INPUT_REFOUT_VALUE`: Value of the referenced output.
     - `INPUT_REFOUT_ADDR_HAS_SCRIPT`: Indicates if the address has a script.
     - `INPUT_REFOUT_PAYMENT_CRED`: Payment credential of the referenced output.
     - `INPUT_REFOUT_STAKE_ADDR`: Stake address of the referenced output.

#### 10. **OUTPUTs**
   - **Type**: String (Aggregated)
   - **Description**: Details of transaction outputs.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `OUTPUT_ID`: Unique identifier of the output.
     - `OUTPUT_RAWADDR`: Raw address of the output.
     - `OUTPUT_STAKE_ADDR_ID`: Stake address ID of the output.
     - `OUTPUT_VALUE`: Value of the output.
     - `OUTPUT_ADDR_HAS_SCRIPT`: Indicates if the address has a script.
     - `OUTPUT_PAYMENT_CRED`: Payment credential of the output.
     - `OUTPUT_STAKE_ADDR`: Stake address of the output.


### Notes

- **Grouping**: Data is grouped by `TX_ID`, `TX_FEE`, `BLOCK_TIME`, and `EPOCH_NO`.
- **Joins**: Utilizes multiple joins with tables like:
  - `tx`, `tx_in`, `tx_out`, `block`, `stake_address`, `TBL_TXOUT_NFTs_FTs_MAs`, `TBL_MINT_NFTs`, and `TBL_MINT_FTs`.
- **Order**: The output is ordered by `TX_ID`, `TX_FEE`, `BLOCK_TIME`, and `EPOCH_NO`.

For additional schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).



### Query
```sql
# Table of NFT Mintings:
cexplorer=# CREATE TABLE TBL_MINT_NFTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE   tx_metadata.key IN (721) 
                AND MINT_TABLE."MA_TOTAL_QUANTITY" = 1 
                AND MINT_TABLE."MA_TOTAL_MINTS_COUNT" = 1 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID";



# Table of FT Mintings:
cexplorer=# CREATE TABLE TBL_MINT_FTs AS
        WITH MINT_TABLE as ( 
            SELECT  ma_tx_mint.ident         AS "MA_ID", 
                    sum(ma_tx_mint.quantity) AS "MA_TOTAL_QUANTITY", 
                    count(*)                 AS "MA_TOTAL_MINTS_COUNT", 
                    min(ma_tx_mint.tx_id)    AS "MA_FIRST_TX_ID" 
                FROM ma_tx_mint 
                GROUP BY ma_tx_mint.ident 
        ) 
        SELECT  MINT_TABLE.*, 
                multi_asset.fingerprint     AS "MA_FINGERPRINT", 
                multi_asset.policy          AS "MA_POLICY", 
                multi_asset.name            AS "MA_NAME", 
                ARRAY_AGG(tx_metadata.key)  AS "MA_FIRST_TX_METADATA_KEY" 
            FROM MINT_TABLE 
                LEFT JOIN multi_asset ON multi_asset.id    = MINT_TABLE."MA_ID" 
                LEFT JOIN tx_metadata ON tx_metadata.tx_id = MINT_TABLE."MA_FIRST_TX_ID" 
            WHERE  tx_metadata.key NOT IN (721) 
            GROUP BY "MA_FINGERPRINT", "MA_POLICY", "MA_NAME", MINT_TABLE."MA_ID", MINT_TABLE."MA_TOTAL_QUANTITY", MINT_TABLE."MA_TOTAL_MINTS_COUNT", MINT_TABLE."MA_FIRST_TX_ID"; 



cexplorer=# CREATE TABLE TBL_TXOUT_NFTs_FTs_MAs AS
        WITH things as (
            SELECT  ma_tx_out.tx_out_id              as "OUTPUT_ID", 
                    ma_tx_out.ident                  as "OUTPUT_MULTIASSET_TXOUT_IDENT", 
                    ma_tx_out.quantity               as "OUTPUT_MULTIASSET_TXOUT_QUANTITY", 
                    TBL_MINT_NFTs."MA_NAME"          as "OUTPUT_NFT_NAME", 
                    TBL_MINT_FTs."MA_NAME"           as "OUTPUT_FT_NAME", 
                    multi_asset.name                 as "OUTPUT_MA_NAME", 
                    multi_asset.policy               as "OUTPUT_MA_POLICY", 
                    multi_asset.fingerprint          as "OUTPUT_MA_FINGERPRINT" 
                FROM ma_tx_out 
                    LEFT JOIN TBL_MINT_NFTs  ON  ma_tx_out.ident = TBL_MINT_NFTs."MA_ID" 
                    LEFT JOIN TBL_MINT_FTs   ON  ma_tx_out.ident = TBL_MINT_FTs."MA_ID" 
                    LEFT JOIN multi_asset    ON  ma_tx_out.ident = multi_asset.id 
        )
        SELECT  things."OUTPUT_ID", 
                STRING_AGG(distinct concat(things."OUTPUT_MULTIASSET_TXOUT_IDENT", ',', things."OUTPUT_MULTIASSET_TXOUT_QUANTITY", ',', things."OUTPUT_NFT_NAME",       ',', things."OUTPUT_FT_NAME",  ',', 
                                           things."OUTPUT_MA_NAME",                ',', things."OUTPUT_MA_POLICY",                 ',', things."OUTPUT_MA_FINGERPRINT",                                ','   ), E':') AS "MAs__NAME__POLICY__FINGERPRINT" 
            FROM things 
            GROUP BY things."OUTPUT_ID";



# All transactions in the network with MA data:
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as ( 
        WITH TXs_INs_OUTs as ( 
            SELECT  tx.id                                                      as "TX_ID", 
                    tx.fee                                                     as "TX_FEE", 
                    block.time                                                 as "BLOCK_TIME", 
                    block.epoch_no                                             as "EPOCH_NO", 
                    tx_in.tx_in_id                                             as "INPUT_TXID", 
                    tx_out.tx_id                                               as "OUTPUT_TXID", 
                    tx_in.id                                                   as "INPUT_ID",  
                    tx_out.id                                                  as "OUTPUT_ID", 
                    tx_in.tx_out_id                                            as "INPUT_REFTX_ID", 
                    tx_in.tx_out_index                                         as "INPUT_REFTX_OUTINDX", 
                    tx_out.address_raw                                         as "OUTPUT_RAWADDR", 
                    tx_out.value                                               as "OUTPUT_VALUE", 
                    tx_out.address_has_script                                  as "OUTPUT_ADDR_HAS_SCRIPT", 
                    tx_out.payment_cred                                        as "OUTPUT_PAYMENT_CRED", 
                    tx_out.stake_address_id                                    as "OUTPUT_STAKE_ADDR_ID", 
                    stake_address.hash_raw                                     as "OUTPUT_STAKE_ADDR", 
                    TBL_TXOUT_NFTs_FTs_MAs."MAs__NAME__POLICY__FINGERPRINT"    as "OUTPUT_MAs__NAME__POLICY__FINGERPRINT" 
                FROM    tx LEFT JOIN block                   ON tx.block_id             = block.id 
                           LEFT JOIN tx_in                   ON tx.id                   = tx_in.tx_in_id 
                           LEFT JOIN tx_out                  ON tx.id                   = tx_out.tx_id 
                           LEFT JOIN stake_address           ON tx_out.stake_address_id = stake_address.id 
                           LEFT JOIN TBL_TXOUT_NFTs_FTs_MAs  ON tx_out.id               = TBL_TXOUT_NFTs_FTs_MAs."OUTPUT_ID" 
                WHERE   tx.id BETWEEN (50000001) AND (60000000) 
        ) 
        SELECT  TXs_INs_OUTs.*, 
                REF_OUT.tx_id                                              as "INPUT_REFOUT_TXID", 
                REF_OUT.index                                              as "INPUT_REFOUT_INDEX", 
                REF_OUT.id                                                 as "INPUT_REFOUT_ID", 
                REF_OUT.address_raw                                        as "INPUT_REFOUT_RAWADDR", 
                REF_OUT.value                                              as "INPUT_REFOUT_VALUE", 
                REF_OUT.address_has_script                                 as "INPUT_REFOUT_ADDR_HAS_SCRIPT", 
                REF_OUT.payment_cred                                       as "INPUT_REFOUT_PAYMENT_CRED", 
                REF_OUT.stake_address_id                                   as "INPUT_REFOUT_STAKE_ADDR_ID", 
                stake_address.hash_raw                                     as "INPUT_REFOUT_STAKE_ADDR", 
                TBL_TXOUT_NFTs_FTs_MAs."MAs__NAME__POLICY__FINGERPRINT"    as "INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT" 
            FROM    TXs_INs_OUTs LEFT JOIN tx_out REF_OUT                  ON     TXs_INs_OUTs."INPUT_REFTX_ID"       = REF_OUT.tx_id 
                                                                              AND TXs_INs_OUTs."INPUT_REFTX_OUTINDX"  = REF_OUT.index 
                                 LEFT JOIN stake_address                   ON     REF_OUT.stake_address_id            = stake_address.id 
                                 LEFT JOIN TBL_TXOUT_NFTs_FTs_MAs          ON     REF_OUT.id                          = TBL_TXOUT_NFTs_FTs_MAs."OUTPUT_ID" 
    ) 
    SELECT  things."TX_ID", 
            things."TX_FEE", 
            things."BLOCK_TIME", 
            things."EPOCH_NO", 

            STRING_AGG(distinct concat(things."INPUT_ID",                  ':', things."INPUT_REFOUT_MAs__NAME__POLICY__FINGERPRINT",                                                                                            ''      ), E';') as "TX_INPUT_MAs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ':', things."OUTPUT_MAs__NAME__POLICY__FINGERPRINT",                                                                                                  ''      ), E';') as "TX_OUTPUT_MAs", 

            STRING_AGG(distinct concat(TBL_MINT_NFTs."MA_ID",              ',', TBL_MINT_NFTs."MA_NAME",                      ',', TBL_MINT_NFTs."MA_FINGERPRINT",                  ',', TBL_MINT_NFTs."MA_POLICY",              ',', 
                                       TBL_MINT_NFTs."MA_TOTAL_QUANTITY",  ',', TBL_MINT_NFTs."MA_TOTAL_MINTS_COUNT",                                                                                                            ','     ), E';') as "MINT_NFTs", 
            STRING_AGG(distinct concat(TBL_MINT_FTs."MA_ID",               ',', TBL_MINT_FTs."MA_NAME",                       ',', TBL_MINT_FTs."MA_FINGERPRINT",                   ',', TBL_MINT_FTs."MA_POLICY",               ',', 
                                       TBL_MINT_FTs."MA_TOTAL_QUANTITY",   ',', TBL_MINT_FTs."MA_TOTAL_MINTS_COUNT",                                                                                                             ','     ), E';') as "MINT_FTs", 

            STRING_AGG(distinct concat(things."INPUT_ID",                  ',', things."INPUT_REFTX_ID",                      ',', things."INPUT_REFTX_OUTINDX",                    ',', things."INPUT_REFOUT_ID",               ',', 
                                       things."INPUT_REFOUT_RAWADDR",      ',', things."INPUT_REFOUT_STAKE_ADDR_ID",          ',', things."INPUT_REFOUT_VALUE",                     ',', things."INPUT_REFOUT_ADDR_HAS_SCRIPT",  ',', 
                                       things."INPUT_REFOUT_PAYMENT_CRED", ',', things."INPUT_REFOUT_STAKE_ADDR",                                                                                                                ','     ), E';') as "INPUTs", 
            STRING_AGG(distinct concat(things."OUTPUT_ID",                 ',', things."OUTPUT_RAWADDR",                      ',', things."OUTPUT_STAKE_ADDR_ID",                   ',', things."OUTPUT_VALUE",                  ',', 
                                       things."OUTPUT_ADDR_HAS_SCRIPT",    ',', things."OUTPUT_PAYMENT_CRED",                 ',', things."OUTPUT_STAKE_ADDR",                                                                   ','     ), E';') as "OUTPUTs" 
        FROM things 
            LEFT JOIN TBL_MINT_NFTs ON things."TX_ID" = TBL_MINT_NFTs."MA_FIRST_TX_ID" 
            LEFT JOIN TBL_MINT_FTs  ON things."TX_ID" = TBL_MINT_FTs."MA_FIRST_TX_ID" 
        GROUP BY things."TX_ID", things."TX_FEE", things."BLOCK_TIME", things."EPOCH_NO" 
        ORDER BY things."TX_ID", things."TX_FEE", things."BLOCK_TIME", things."EPOCH_NO" 
) TO '/cardano_TXs_MAs_<filenumber>.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_

```


***


#### **`cardano_pools.csv`**

This query generates a CSV file containing details of staking pools in the Cardano network, including their stake amounts and rewards for each epoch.

- **File Format**: CSV with `,` as the delimiter and a header row.
- **File Content**: The file includes aggregated information about active staking pools and their rewards.
- **Scope**:
  - Stake amounts (`POOL_STAKES`) by epoch and pool
  - Rewards earned (`POOL_REWARDS`) by epoch and pool


### Column Details

#### 1. **EPOCH**
   - **Type**: Integer
   - **Description**: The epoch number during which the stakes and rewards were recorded.

#### 2. **POOL_ID**
   - **Type**: String
   - **Description**: Unique identifier of the staking pool.

#### 3. **POOL_STAKES**
   - **Type**: Numeric
   - **Description**: Total amount of ADA staked in the pool for the corresponding epoch.

#### 4. **POOL_REWARDS**
   - **Type**: Numeric
   - **Description**: Total rewards distributed to the pool for the corresponding epoch. If no rewards were distributed, this value is `0`.

- **Joins**:
  - Combines data from:
    - `epoch_stake`: To calculate total stakes per epoch and pool.
    - `reward`: To calculate total rewards distributed per epoch and pool.
  - Pools with stakes but no rewards are included using a `LEFT JOIN` with `COALESCE` to handle null rewards.
- **Grouping**: Data is grouped by `EPOCH` and `POOL_ID`.
- **Order**: Results are ordered by `EPOCH` and descending `POOL_REWARDS`.


For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).

```sql
# Epoch | pool_id | stake amount | rewards amount
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as (
        WITH    active_pools    as (select epoch_no     as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_STAKES"  from epoch_stake group by epoch_no, pool_id ), 
                rewarded_pools  as (select earned_epoch as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_REWARDS" from reward      group by earned_epoch, pool_id) 
        SELECT  a."EPOCH" as "EPOCH", a."POOL_ID" as "POOL_ID", a."POOL_STAKES", COALESCE(r."POOL_REWARDS",0) as "POOL_REWARDS" 
            FROM active_pools a LEFT JOIN rewarded_pools r ON a."EPOCH"=r."EPOCH" AND a."POOL_ID"=r."POOL_ID") 
    SELECT * FROM things t ORDER BY t."EPOCH", t."POOL_REWARDS" desc 
) TO '/cardano_pools.csv' WITH CSV DELIMITER ',' HEADER 
_EOF_
```

***

#### **`cardano_pools_2.csv`**

This query generates a CSV file containing information about staking pools in the Cardano network, including their stake amounts, rewards, and pool hash in Bech32 format for each epoch.

- **File Format**: CSV with `,` as the delimiter and a header row.
- **File Content**: Aggregates data about staking pools, including:
  - Stake amounts (`POOL_STAKES`) for each pool in an epoch.
  - Rewards distributed (`POOL_REWARDS`) to each pool.
  - Staking pool hash in Bech32 format (`POOL_HASH_BECH32`).
- **Scope**: Covers all pools, including those with stakes but no rewards (reward amount set to `0`).


### Column Details

#### 1. **EPOCH**
   - **Type**: Integer
   - **Description**: The epoch number during which the stakes and rewards were recorded.

#### 2. **POOL_HASH_BECH32**
   - **Type**: String
   - **Description**: The Bech32 format of the staking pool's hash, representing the pool's unique identifier.

#### 3. **POOL_STAKES**
   - **Type**: Numeric
   - **Description**: Total amount of ADA staked in the pool for the corresponding epoch.

#### 4. **POOL_REWARDS**
   - **Type**: Numeric
   - **Description**: Total rewards distributed to the pool for the corresponding epoch. If no rewards were distributed, this value is `0`.


- **Joins**:
  - Combines data from:
    - `epoch_stake`: To calculate total stakes per epoch and pool.
    - `reward`: To calculate total rewards distributed per epoch and pool.
    - `pool_hash`: To fetch the Bech32 hash representation of the pool.
  - Pools with stakes but no rewards are included using a `LEFT JOIN` with `COALESCE` to handle null rewards.

- **Grouping**: Data is grouped by `EPOCH` and `POOL_ID`.

- **Order**: Results are ordered by `EPOCH` and descending `POOL_REWARDS`.



For further details on the database schema, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).

```sql
# Epoch | pool_hash | stake amount | rewards amount
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD=postgres \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as (
        WITH    active_pools    as (select epoch_no     as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_STAKES"  from epoch_stake group by epoch_no, pool_id), 
                rewarded_pools  as (select earned_epoch as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_REWARDS" from reward      group by earned_epoch, pool_id) 
        SELECT  a."EPOCH" as "EPOCH", p.view as "POOL_HASH_BECH32", a."POOL_STAKES", COALESCE(r."POOL_REWARDS",0) as "POOL_REWARDS" 
            FROM active_pools a LEFT JOIN pool_hash p      ON a."POOL_ID" = p.id 
                                LEFT JOIN rewarded_pools r ON a."EPOCH"=r."EPOCH" AND a."POOL_ID"=r."POOL_ID") 
    SELECT * FROM things t ORDER BY t."EPOCH", t."POOL_REWARDS" desc 
) TO '/cardano_pools_2.csv' WITH CSV DELIMITER ',' HEADER 
_EOF_
```


***

#### **`cardano_pools_3.csv`**


This query generates a CSV file with aggregated information about staking pools in the Cardano network, including details on stakes, rewards, and the number of delegators and rewarders for each epoch.

- **File Format**: CSV with `,` as the delimiter and a header row.
- **File Content**: Contains aggregated data on staking pools:
  - Total stake amounts (`POOL_STAKES`) for each pool in an epoch.
  - Rewards distributed (`POOL_REWARDS`) to each pool.
  - Number of unique delegators contributing to the pool (`NUM_OF_DELEGATORS`).
  - Number of unique reward recipients in the pool (`NUM_OF_REWARDERS`).
  - Staking pool hash (`POOL_HASH_BECH32`) for pool identification.


### Column Details

#### 1. **EPOCH**
   - **Type**: Integer
   - **Description**: The epoch number during which the stakes and rewards were recorded.

#### 2. **POOL_HASH_BECH32**
   - **Type**: String
   - **Description**: Bech32-encoded hash representing the unique identifier of the staking pool.

#### 3. **POOL_STAKES**
   - **Type**: Numeric
   - **Description**: Total amount of ADA staked in the pool during the epoch.

#### 4. **POOL_REWARDS**
   - **Type**: Numeric
   - **Description**: Total rewards distributed to the pool for the epoch. If no rewards were distributed, this value is `0`.

#### 5. **NUM_OF_DELEGATORS**
   - **Type**: Integer
   - **Description**: Number of unique delegators staking ADA in the pool during the epoch.

#### 6. **NUM_OF_REWARDERS**
   - **Type**: Integer
   - **Description**: Number of unique reward recipients from the pool during the epoch. If no rewards were distributed, this value is `0`.



### Query Design

1. **Data Sources**:
   - `epoch_stake`: Provides stake amounts and unique delegator counts (`NUM_OF_DELEGATORS`) for each pool in an epoch.
   - `reward`: Provides reward amounts and unique reward recipient counts (`NUM_OF_REWARDERS`) for each pool in an epoch.
   - `pool_hash`: Maps pool IDs to their Bech32 hash representations.

2. **Joins**:
   - `LEFT JOIN` with `rewarded_pools` ensures that pools with stakes but no rewards are included, setting `POOL_REWARDS` and `NUM_OF_REWARDERS` to `0` when necessary.
   - `LEFT JOIN` with `pool_hash` fetches the Bech32 pool hash for each pool ID.

3. **Grouping**:
   - Aggregates data by `EPOCH` and `POOL_ID`.

4. **Ordering**:
   - Results are ordered by `EPOCH` (ascending) and `POOL_REWARDS` (descending).



For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).

```sql
# Epoch | pool_hash | stake amount | rewards amount | num of delegators | num of rewarders 
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD=postgres \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as (
        WITH    active_pools    as (select epoch_no     as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_STAKES",  count(distinct addr_id) as "NUM_OF_DELEGATORS"  from epoch_stake group by epoch_no, pool_id), 
                rewarded_pools  as (select earned_epoch as "EPOCH", pool_id as "POOL_ID", sum(amount) as "POOL_REWARDS", count(distinct addr_id) as "NUM_OF_REWARDERS"   from reward      group by earned_epoch, pool_id) 
        SELECT  a."EPOCH"                        as "EPOCH", 
                p.view                           as "POOL_HASH_BECH32", 
                a."POOL_STAKES"                  as "POOL_STAKES", 
                COALESCE(r."POOL_REWARDS",0)     as "POOL_REWARDS", 
                a."NUM_OF_DELEGATORS"            as "NUM_OF_DELEGATORS", 
                COALESCE(r."NUM_OF_REWARDERS",0) as "NUM_OF_REWARDERS" 
            FROM active_pools a LEFT JOIN pool_hash p      ON a."POOL_ID" = p.id 
                                LEFT JOIN rewarded_pools r ON a."EPOCH"=r."EPOCH" AND a."POOL_ID"=r."POOL_ID"
    ) 
    SELECT * FROM things t ORDER BY t."EPOCH", t."POOL_REWARDS" desc 
) TO '/cardano_pools_3.csv' WITH CSV DELIMITER ',' HEADER 
_EOF_
```


***

#### **`cardano_pools_4.csv`**


This query generates a CSV file containing detailed information about Cardano staking pools, including their stake and rewards data, the number of delegators and rewarders, and detailed lists of delegators and rewarders.


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: Includes aggregated and detailed information about staking pools:
  - Total stake and rewards (`POOL_STAKES`, `POOL_REWARDS`) for each pool in an epoch.
  - The number of delegators (`NUM_OF_DELEGATORS`) and rewarders (`NUM_OF_REWARDERS`) for each pool.
  - Detailed lists of delegators and rewarders, including their stake address IDs, delegated amounts, and raw stake addresses.


### Column Details

#### 1. **EPOCH**
   - **Type**: Integer
   - **Description**: The epoch number during which the stakes and rewards were recorded.

#### 2. **POOL_ID**
   - **Type**: Integer
   - **Description**: Unique identifier of the staking pool.

#### 3. **POOL_HASH_BECH32**
   - **Type**: String
   - **Description**: Bech32-encoded hash representing the unique identifier of the staking pool.

#### 4. **POOL_STAKES**
   - **Type**: Numeric
   - **Description**: Total amount of ADA staked in the pool during the epoch.

#### 5. **POOL_REWARDS**
   - **Type**: Numeric
   - **Description**: Total rewards distributed to the pool for the epoch. If no rewards were distributed, this value is `0`.

#### 6. **NUM_OF_DELEGATORS**
   - **Type**: Integer
   - **Description**: Number of unique delegators staking ADA in the pool during the epoch.

#### 7. **NUM_OF_REWARDERS**
   - **Type**: Integer
   - **Description**: Number of unique reward recipients from the pool during the epoch. If no rewards were distributed, this value is `0`.

#### 8. **DELEGATORs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all delegators staking ADA in the pool during the epoch.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `addr_id`: Stake address ID of the delegator.
     - `amount`: Amount of ADA delegated.
     - `hash_raw`: Raw stake address of the delegator.

#### 9. **REWARDERs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all reward recipients from the pool during the epoch.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `addr_id`: Stake address ID of the reward recipient.
     - `amount`: Amount of ADA rewarded.
     - `hash_raw`: Raw stake address of the reward recipient.


### Query Design Notes

1. **Data Sources**:
   - `epoch_stake`: Provides stake amounts and unique delegator details (`NUM_OF_DELEGATORS`, `DELEGATORs`) for each pool in an epoch.
   - `reward`: Provides reward amounts and unique reward recipient details (`NUM_OF_REWARDERS`, `REWARDERs`) for each pool in an epoch.
   - `pool_hash`: Maps pool IDs to their Bech32 hash representations.

2. **Joins**:
   - Combines data from the `epoch_stake` and `reward` tables.
   - `LEFT JOIN` with `pool_hash` fetches the Bech32 pool hash.
   - `LEFT JOIN` with `stake_address` adds detailed delegator and rewarder information.

3. **Grouping**:
   - Aggregates data by `EPOCH` and `POOL_ID`.

4. **Ordering**:
   - Results are ordered by `EPOCH` (ascending) and `POOL_REWARDS` (descending).


- **Breakdown Delegator and Rewarder Details**: Generate separate tables for delegators and rewarders with individual rows for each record.
- **Reward Analysis**: Add a reward-to-stake ratio for each pool.


For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).

```sql
# Epoch | pool_id | pool_hash | stake amount | rewards amount | num of delegators | num of rewarders |
# | delegators (stake address id, delegated amount, stake address) |
# | rewarders  (stake address id, delegated amount, stake address) |

cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h <IP> -p <PORT> -U postgres cexplorer 
\copy ( 
    WITH things as (
        WITH    active_pools    as (select  e.epoch_no     as "EPOCH", e.pool_id as "POOL_ID", sum(e.amount) as "POOL_STAKES",  count(distinct e.addr_id) as "NUM_OF_DELEGATORS", 
                                            STRING_AGG(distinct concat(e.addr_id, ',', e.amount, ',', s.hash_raw, ','), E';') as "DELEGATORs" 
                                        from epoch_stake e 
                                            LEFT JOIN stake_address s ON e.addr_id = s.id 
                                        group by e.epoch_no, e.pool_id 
                ), 
                rewarded_pools  as (select  r.earned_epoch as "EPOCH", r.pool_id as "POOL_ID", sum(r.amount) as "POOL_REWARDS", count(distinct r.addr_id) as "NUM_OF_REWARDERS", 
                                            STRING_AGG(distinct concat(r.addr_id, ',', r.amount, ',', s.hash_raw, ','), E';') as "REWARDERs" 
                                        from reward r 
                                            LEFT JOIN stake_address s ON r.addr_id = s.id 
                                        group by r.earned_epoch, r.pool_id 
                ) 
        SELECT  a."EPOCH"                        as "EPOCH", 
                a."POOL_ID"                      as "POOL_ID", 
                p.view                           as "POOL_HASH_BECH32", 
                a."POOL_STAKES"                  as "POOL_STAKES", 
                COALESCE(r."POOL_REWARDS",0)     as "POOL_REWARDS", 
                a."NUM_OF_DELEGATORS"            as "NUM_OF_DELEGATORS", 
                COALESCE(r."NUM_OF_REWARDERS",0) as "NUM_OF_REWARDERS", 
                a."DELEGATORs"                   as "DELEGATORs", 
                r."REWARDERs"                    as "REWARDERs" 
            FROM active_pools a LEFT JOIN pool_hash p      ON a."POOL_ID" = p.id 
                                LEFT JOIN rewarded_pools r ON a."EPOCH"=r."EPOCH" AND a."POOL_ID"=r."POOL_ID" 
    ) 
    SELECT * FROM things t ORDER BY t."EPOCH", t."POOL_REWARDS" desc 
) TO '/cardano_pools_4.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_
```


***

#### **`cardano_pools_5.csv`**

This query generates a CSV file containing detailed information about Cardano staking pools for each epoch, including stakes, rewards, delegators, and rewarders. Additionally, it provides a detailed breakdown of delegator and rewarder data.


- **File Format**: CSV with `|` as the delimiter and a header row.
- **File Content**: Includes aggregated and detailed information for each pool, including:
  - Epoch and pool identifiers (`EPOCH`, `POOL_ID`, `POOL_HASH_BECH32`).
  - Total stake amounts (`POOL_STAKES`) and rewards distributed (`POOL_REWARDS`).
  - Number of delegators and rewarders.
  - Detailed lists of delegators and rewarders with their stake addresses, amounts, and reward types.


### Column Details

#### 1. **EPOCH**
   - **Type**: Integer
   - **Description**: The epoch number during which the stakes and rewards were recorded.

#### 2. **POOL_ID**
   - **Type**: Integer
   - **Description**: Unique identifier for the staking pool.

#### 3. **POOL_HASH_BECH32**
   - **Type**: String
   - **Description**: Bech32-encoded hash representing the staking pool.

#### 4. **POOL_STAKES**
   - **Type**: Numeric
   - **Description**: Total amount of ADA staked in the pool during the epoch.

#### 5. **POOL_REWARDS**
   - **Type**: Numeric
   - **Description**: Total rewards distributed to the pool during the epoch. If no rewards were distributed, this value is `0`.

#### 6. **NUM_OF_DELEGATORS**
   - **Type**: Integer
   - **Description**: Number of unique delegators staking ADA in the pool during the epoch.

#### 7. **NUM_OF_REWARDERS**
   - **Type**: Integer
   - **Description**: Number of unique reward recipients from the pool during the epoch. If no rewards were distributed, this value is `0`.

#### 8. **DELEGATORs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all delegators staking ADA in the pool during the epoch.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `addr_id`: Stake address ID of the delegator.
     - `amount`: Amount of ADA delegated.
     - `hash_raw`: Raw stake address of the delegator.

#### 9. **REWARDERs**
   - **Type**: String (Aggregated)
   - **Description**: Aggregated details of all reward recipients from the pool during the epoch.
   - **Format**: Aggregated string separated by `;`, where each entry contains:
     - `addr_id`: Stake address ID of the reward recipient.
     - `amount`: Amount of ADA rewarded.
     - `hash_raw`: Raw stake address of the reward recipient.
     - `type`: Type of reward (e.g., "leader", "member").


### Query Design Notes

1. **Data Sources**:
   - `epoch_stake`: Provides stake amounts and unique delegator details for each pool in an epoch.
   - `reward`: Provides reward amounts, reward recipient details, and reward types for each pool in an epoch.
   - `pool_hash`: Maps pool IDs to their Bech32 hash representations.
   - `stake_address`: Adds detailed address information for delegators and rewarders.

2. **Joins**:
   - Combines data from the `epoch_stake` and `reward` tables.
   - `LEFT JOIN` with `pool_hash` to fetch Bech32 pool hashes.
   - `LEFT JOIN` with `stake_address` for address details.

3. **Grouping**:
   - Aggregates data by `EPOCH` and `POOL_ID`.

4. **Ordering**:
   - Results are ordered by `EPOCH` (ascending) and `POOL_REWARDS` (descending).


For schema details, refer to the [Cardano DB Schema Documentation](https://github.com/IntersectMBO/cardano-db-sync/blob/13.3.0.0/doc/schema.md).


```sql
# Epoch | pool_id | pool_hash | stake amount | rewards amount | num of delegators | num of rewarders |
# | delegators (stake address id, delegated amount, stake address) |
# | rewarders  (stake address id, delegated amount, stake address) |


#cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD=postgres \psql -h 172.23.38.242 -p 5432 -U postgres cexplorer 
cat <<_EOF_ | tr '\n' ' ' | PGPASSWORD='???' \psql -h -h <IP> -p <PORT> -U utxoexplorer utxoexplorer 
\copy ( 
    WITH things as (
        WITH    active_pools    as (select  e.epoch_no     as "EPOCH", e.pool_id as "POOL_ID", sum(e.amount) as "POOL_STAKES",  count(distinct e.addr_id) as "NUM_OF_DELEGATORS", 
                                            STRING_AGG(distinct concat(e.addr_id, ',', e.amount, ',', s.hash_raw, ','), E';') as "DELEGATORs" 
                                        from epoch_stake e 
                                            LEFT JOIN stake_address s ON e.addr_id = s.id 
                                        group by e.epoch_no, e.pool_id 
                ), 
                rewarded_pools  as (select  r.earned_epoch as "EPOCH", r.pool_id as "POOL_ID", sum(r.amount) as "POOL_REWARDS", count(distinct r.addr_id) as "NUM_OF_REWARDERS", 
                                            STRING_AGG(distinct concat(r.addr_id, ',', r.amount, ',', s.hash_raw, ',', r.type, ','), E';') as "REWARDERs" 
                                        from reward r 
                                            LEFT JOIN stake_address s ON r.addr_id = s.id 
                                        group by r.earned_epoch, r.pool_id 
                ) 
        SELECT  a."EPOCH"                        as "EPOCH", 
                a."POOL_ID"                      as "POOL_ID", 
                p.view                           as "POOL_HASH_BECH32", 
                a."POOL_STAKES"                  as "POOL_STAKES", 
                COALESCE(r."POOL_REWARDS",0)     as "POOL_REWARDS", 
                a."NUM_OF_DELEGATORS"            as "NUM_OF_DELEGATORS", 
                COALESCE(r."NUM_OF_REWARDERS",0) as "NUM_OF_REWARDERS", 
                a."DELEGATORs"                   as "DELEGATORs", 
                r."REWARDERs"                    as "REWARDERs" 
            FROM active_pools a LEFT JOIN pool_hash p      ON a."POOL_ID" = p.id 
                                LEFT JOIN rewarded_pools r ON a."EPOCH"=r."EPOCH" AND a."POOL_ID"=r."POOL_ID" 
    ) 
    SELECT * FROM things t ORDER BY t."EPOCH", t."POOL_REWARDS" desc 
) TO './cardano_pools_5.csv' WITH CSV DELIMITER '|' HEADER 
_EOF_
```


***

