# 9_ADA_holding_time

In the following sections, each cell in the corresponding 'code' (with a similar filename in the code directory of this repository) is documented with a brief description of its purpose and functionality, followed by the summarized cell code.

***

### Import Libraries and Set Constant Variables

This cell imports a comprehensive set of libraries and modules required for data analysis, visualization, multiprocessing, and Spark-based operations. It also initializes certain environment variables, constants, and performs some basic computations.

#### Explanation of the Code:
1. **Library Imports**:
   - **Core libraries**: `numpy`, `array`, `csv`, `datetime`, and `os`.
   - **Search and sorting**: `bisect`.
   - **Visualization**: `matplotlib.pyplot`.
   - **Parallel processing**: `multiprocessing` and `threading`.
   - **Big data processing**: `pyspark` and `pandas`.
   - **Graph processing**: `networkx` and `community`.
   - **Probability distributions**: `powerlaw`.
   - **Progress tracking**: `tqdm`.
   - **Serialization**: `pickle`.

2. **Environment Variable Setup**:
   - Disables timezone warnings for PyArrow with `os.environ`.

3. **Cardano Blockchain Date and Address Analysis**:
   - Sets constants for unique address counts and calculates the total time length of the Cardano blockchain in days.

4. **Outputs**:
   - Prints address count statistics and a success message.

#### Cell Code:
```python
# Import libraries and set constant variables:

import numpy as np
from array import *
import csv

# using datetime module
import datetime;

# Binary Search
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

# https://python-louvain.readthedocs.io/en/latest/api.html
# community.modularity(partition, graph, weight='weight')
from community import modularity

import pickle

import powerlaw

print('----------------------')
# unique_payment_addresses_len = len(unique_payment_addresses)
unique_raw_addresses_len        = 40330345
unique_payment_addresses_len    = 40324960
unique_delegation_addresses_len = 3868049
print('unique_raw_addresses_len        = ', unique_raw_addresses_len)
print('unique_payment_addresses_len    = ', unique_payment_addresses_len)
print('unique_delegation_addresses_len = ', unique_delegation_addresses_len)

INITIAL_DATE_CARDANO      = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
FINAL_DATE_CARDANO        = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds()/86400) + 1

print('----------------------')
print('done!')
```


***

### Define Base and Temporary Directory Paths

This cell sets up the base and temporary directory paths to organize and manage file storage for the Cardano project.

#### Explanation of the Code:
1. **`BASE_ADDRESS`**:
   - Defines the base directory for storing exported data related to the project.

2. **`TEMP_ADDRESS`**:
   - Defines a subdirectory within the base directory to store temporary files.

These paths centralize file storage management and ensure consistent directory usage throughout the project.

#### Cell Code:
```python
BASE_ADDRESS = '/local/scratch/exported/Cardano_MCH_2023_1/'
TEMP_ADDRESS = BASE_ADDRESS + '/temp_files/'
```



***

# ADA Velocity: Distribution of Holding ADA

This cell calculates the distribution of ADA holding durations for the period from January 2022 to February 2022. A sampling rate is applied to select a subset of transactions for estimation. The main steps include:

1. **Input Parameters**:
   - **SAMPLE_RATE**: Determines the proportion of transactions to sample for the analysis.
   - **FIRST_DAY** and **LAST_DAY**: Define the range of days relative to Cardano's operational start date (September 23, 2017).
2. **Data Initialization**:
   - Initializes an array (`hodling_day_array`) to store the aggregated ADA values for each holding day.
   - Defines the range of operational days from Cardano's launch to January 2023.
3. **File Processing**:
   - Loads transaction data from multiple CSV files. Each file contains information about transactions, including input values and creation times.
   - Filters transactions within the specified date range and calculates the holding duration for each sampled input.
   - Aggregates the ADA value held for the calculated holding duration.
4. **Output Storage**:
   - Stores the holding day distribution in a pickle file with a filename reflecting the sample rate and date range.
5. **Logging**:
   - Logs elapsed time for processing individual files and the overall execution.

### CSV File Details
- **Format**: `|` delimited
- **Columns**:
  - `TX_ID` (Transaction ID)
  - `BLOCK_TIME` (Transaction timestamp in `%Y-%m-%d %H:%M:%S` format)
  - `INPUTs` (Semicolon-separated list containing input details)
    - Example of input details: `...,value,creation_time,...`

### Output File Details
- **Filename Format**:
  - `YuZhang__HoldingDayArray__SampleRate_{SAMPLE_RATE}_From_{FIRST_DAY}_To_{LAST_DAY}__Cardano_TXs_All.txt`
  - Example: `YuZhang__HoldingDayArray__SampleRate_0.3_From_1575_To_1605__Cardano_TXs_All.txt`
- **File Format**: Pickle (`.txt` in binary format)
- **Number of Columns**: 1
- **Column Details**:
  - **Column Name**: ADA Value
  - **Column Type**: Integer
  - **Description**: Aggregated ADA value held for a specific holding duration (in days). Each row's index corresponds to the number of holding days.

### Code
```python
# ADA Velocity: Distribution of holding ADA from "Jan. 2022" to "Feb. 2022"  --->> [SAMPLE_RATE = 0.3]:

print('----------------------')
import random

def Find_ADA_Velocity(queue_):
    # read input queue arguments
    in_args = queue_.get()
    SAMPLE_RATE = in_args[0]
    FIRST_DAY = in_args[1]
    LAST_DAY = in_args[2]

    # ct stores current time
    ct = datetime.datetime.now()

    INITIAL_DATE_CARDANO = datetime.datetime.strptime('2017-09-23 21:44:51', '%Y-%m-%d %H:%M:%S').date()
    FINAL_DATE_CARDANO = datetime.datetime.strptime('2023-01-21 17:39:30', '%Y-%m-%d %H:%M:%S').date()
    total_time_length_CARDANO = int((FINAL_DATE_CARDANO - INITIAL_DATE_CARDANO).total_seconds() / 86400) + 1

    hodling_day_array = [0] * total_time_length_CARDANO

    CSV_FILES_NAME_FORMAT = BASE_ADDRESS + '/cardano_TXs_Velocity_'
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
            # Extract transaction details
            TX_ID = df.loc[index, 'TX_ID']
            BLOCK_TIME = datetime.datetime.strptime(str(df.loc[index, 'BLOCK_TIME']), '%Y-%m-%d %H:%M:%S').date()
            tx_delta_day = int((BLOCK_TIME - INITIAL_DATE_CARDANO).total_seconds() / 86400)
            if tx_delta_day < FIRST_DAY or tx_delta_day > LAST_DAY:
                continue

            EPOCH_NO = str(df.loc[index, 'EPOCH_NO'])
            inputs_list = list(df.loc[index, 'INPUTs'].split(';'))
            for tx_input in inputs_list:
                input_value = int(tx_input.split(',')[6])
                input_time_str = tx_input.split(',')[10]
                INPUT_TIME = datetime.datetime.strptime(input_time_str, '%Y-%m-%d %H:%M:%S').date()
                INPUT_HOLDING_DAY = int((BLOCK_TIME - INPUT_TIME).total_seconds() / 86400)

                # Random sampling
                random_float = random.random()
                if random_float <= SAMPLE_RATE:
                    hodling_day_array[INPUT_HOLDING_DAY] += input_value

        et_temp = datetime.datetime.now() - ct_temp
        print("elapsed time (ADA Velocity from CSV File " + file_name + "): ", et_temp)

    output_filename = BASE_ADDRESS + '/YuZhang_Holding_Days/YuZhang__HoldingDayArray__' + 'SampleRate_' + str(SAMPLE_RATE).zfill(4) + '_From_' + str(FIRST_DAY).zfill(4) + '_To_' + str(LAST_DAY).zfill(4) + '__Cardano_TXs_All.txt'
    print('output_filename = ', output_filename)
    pickle.dump(hodling_day_array, open(output_filename, 'wb'))

    print('----------------------')
    et = datetime.datetime.now() - ct
    print("Total elapsed time (ADA Velocity): ", et)

print('----------------------')
print('done!')
```



***


# Find Ground Truth for Velocity

This cell calculates the ground truth for ADA velocity (average ADA movement per day) for each month in 2022. The results will later be used to compare with velocities calculated using random sampling methods.

1. **Date Range**:
   - The `FIRST_DAY_of_each_month_2022` array defines the first day of each month in 2022 as day offsets from Cardano's operational start date (September 23, 2017).

2. **File Processing**:
   - Loads holding day distributions for each month from pickle files. These files contain daily ADA values derived from prior processing.
   - Converts values from lovelaces to ADA by dividing by 1,000,000.

3. **Velocity Calculation**:
   - Extracts the first 400 days of the holding distribution for analysis.
   - Calculates the ground truth velocity for the month as the sum of ADA held divided by the length of the month.

4. **Visualization Preparation**:
   - Defines `x` (holding days) and `y` (ADA held for each day) for potential plotting.
   - Prints the calculated ground truth velocities for each month.

### Output
The ground truth velocities are printed as a comma-separated list, which can later be used for comparison.

### File Details
- **Filename Format**:
  - `YuZhang__HoldingDayArray__SampleRate_0001_From_{START_DAY}_To_{END_DAY}__Cardano_TXs_All.txt`
  - Example: `YuZhang__HoldingDayArray__SampleRate_0001_From_1561_To_1592__Cardano_TXs_All.txt`

- **File Format**: Pickle
- **Content**: Array representing daily ADA holding values (in lovelaces) for the specified date range.

### Code
```python
# Find ground truth for velocity:

# Subsequently, this value will be compared to the velocity calculated based on random sampling of transactions.

FIRST_DAY_of_each_month_2022 = [1561, 1592, 1620, 1651, 1681, 1712, 1742, 1773, 1804, 1834, 1865, 1895, 1926]

# Create your individual plots and customize them
for i, ax in tqdm(enumerate(axes.flatten())):
    file_name = BASE_ADDRESS + '/YuZhang_Holding_Days/' + 'YuZhang__HoldingDayArray__SampleRate_0001_From_' + str(FIRST_DAY_of_each_month_2022[i]).zfill(4) + '_To_' + str(FIRST_DAY_of_each_month_2022[i+1]).zfill(4) + '__Cardano_TXs_All.txt'

    hodling_day_array = pickle.load(open(file_name, 'rb'))
    hodling_day_array_ADA = [0] * len(hodling_day_array)
    for j in range(len(hodling_day_array_ADA)):
        hodling_day_array_ADA[j] = float(hodling_day_array[j]) / 1000000

    # Pick up only the first 400 values:
    hodling_day_array_ADA = hodling_day_array_ADA[:400]

    month_length = FIRST_DAY_of_each_month_2022[i+1] - FIRST_DAY_of_each_month_2022[i] + 1
    ground_truth_velocity = sum(hodling_day_array_ADA) / month_length

    x = np.arange(1, len(hodling_day_array_ADA) + 1)
    y = np.array(hodling_day_array_ADA)

    print(str(ground_truth_velocity) + ',')

***






