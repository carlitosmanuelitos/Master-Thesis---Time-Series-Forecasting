I have the need to run some analytics on a couple of DB. 
Main goal is to track the DB schema performance is regards to size on specific dates. 

There is a need to provide a current data assessment + a combined data assessment (How the database size in tables has evolved through time). The files regarding the data are stored in a local env with multiple csv files. 
Data is stored under "Data" -> "Schema" -> files
files are stored with the following convention:

Schema_DB_dd_mm_yyyy.csv where dd represents the day, mm represents the month and yyyy represents the year of the extraction. 
Currently we have 3 files available on the directory. 

Schema_DB_23_04_2024.csv,
Schema_DB_30_04_2024.csv,
Schema_DB_07_05_2024.csv

The csv files contains the following columns: 

Table Name;	Rows; Used Space (MB); Total Space (MB); TypeCode; Extension Name;	Item Type; Details

I have some old code that new refactoring to works with the new requirements and data sources. The goal is to returned a current_df with the most recent dates of extraction signified with date_str & a combined_df signified of the append of all files. There is a logic with the old code to remove duplicates based off the "UPDATE DATE" column that should be removed as this logic is not needed anymore. 

There is also a logic of an output folder that should be maintained. 

Can you adapt the following code?

import os
import warnings
import logging
import pandas as pd
import numpy as np
from datetime import datetime
import seaborn as sns
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
from IPython.display import display, HTML


warnings.simplefilter(action="ignore")
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def setup_paths(date_str, specific_dates):
    data_folder = "Visualization/Data/Orders/"
    output_folder_base = "Visualization/Graphs Schema/Schema/"
    
    input_file_name = f"Schema_DB_{date_str}.csv"
    output_folder = os.path.join(output_folder_base, date_str)
    input_file_path = os.path.join(data_folder, input_file_name)
    output_path = os.path.join(output_folder)
    
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    current_df = None
    try:
        current_df = pd.read_csv(input_file_path, sep=";")
        logging.info(f"Input file path for current visualization: {input_file_path}")
        logging.info(f"Output path for current visualization: {output_path}")
        logging.info(f"Current DataFrame shape before duplicates preprocessing: {current_df.shape}")
        
        # Preprocessing on current_df
        current_df['UPDATE DATE'] = pd.to_datetime(current_df['UPDATE DATE'].str.split().str[0], format='%d/%m/%Y', errors='coerce')
        current_df.dropna(subset=['UPDATE DATE'], inplace=True)
        current_df.sort_values('UPDATE DATE', inplace=True)
        current_df.drop_duplicates(subset='ORDER CODE', keep='last', inplace=True)
        logging.info(f"Current DataFrame shape after duplicates preprocessing: {current_df.shape}")
    except Exception as e:
        logging.error(f"Error processing current data: {e}")
        return None, None, None

    total_rows_before = 0
    combined_df = pd.DataFrame()
    specific_dates_dt = pd.to_datetime(specific_dates, format="%d-%m-%Y")

    for file in os.listdir(data_folder):
        if file.endswith(".csv") and pd.to_datetime(file.split("_")[-1].split('.')[0], format="%d-%m-%Y") in specific_dates_dt:
            try:
                df = pd.read_csv(os.path.join(data_folder, file), sep=";")
                total_rows_before += df.shape[0]
                df['UPDATE DATE'] = pd.to_datetime(df['UPDATE DATE'].str.split().str[0], format='%d/%m/%Y', errors='coerce')
                df.dropna(subset=['UPDATE DATE'], inplace=True)
                df.sort_values('UPDATE DATE', inplace=True)
                df.drop_duplicates(subset='ORDER CODE', keep='last', inplace=True)
                combined_df = pd.concat([combined_df, df], ignore_index=True)
            except Exception as e:
                logging.error(f"Error processing file {file}: {e}")

    logging.info(f"Combined DataFrame shape before duplicates preprocessing: ({total_rows_before}, {combined_df.shape[1]})")
    logging.info(f"Combined DataFrame shape after duplicates preprocessing: {combined_df.shape}")

    return current_df, combined_df, output_path



# Setup paths and load data
date_str = "25-04-2024"
specific_dates = ["08-03-2024", "15-03-2024", "22-03-2024", "29-03-2024", "02-04-2024", "05-04-2024",
                  "08-04-2024", "11-04-2024", "15-04-2024",  "18-04-2024", "22-04-2024", "25-04-2024"]


current_df, combined_df, output_path = setup_paths(date_str, specific_dates)
