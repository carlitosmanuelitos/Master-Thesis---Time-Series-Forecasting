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

############ __________________________________________________________

import os
import logging
import pandas as pd
from datetime import datetime

# Configure logging
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def setup_paths(date_str, specific_dates):
    data_folder = "Data/Schema/"
    output_folder_base = "Visualization/Graphs Schema/Schema/"
    
    # Format the date string for file path
    formatted_date_str = datetime.strptime(date_str, "%d-%m-%Y").strftime("%d_%m_%Y")
    input_file_name = f"Schema_DB_{formatted_date_str}.csv"
    output_folder = os.path.join(output_folder_base, date_str)
    input_file_path = os.path.join(data_folder, input_file_name)
    
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    try:
        # Load the most recent dataset
        current_df = pd.read_csv(input_file_path, sep=";")
        logging.info(f"Loaded data for date: {date_str}")
    except Exception as e:
        logging.error(f"Error loading current data: {e}")
        return None, None, None

    # Combine datasets based on specific dates
    combined_df = pd.DataFrame()
    formatted_specific_dates = [datetime.strptime(date, "%d-%m-%Y").strftime("%d_%m_%Y") for date in specific_dates]
    for file in os.listdir(data_folder):
        file_date = file.split("_")[-1].split(".")[0]  # Extract date part from file name
        if file_date in formatted_specific_dates:
            try:
                df = pd.read_csv(os.path.join(data_folder, file), sep=";")
                combined_df = pd.concat([combined_df, df], ignore_index=True)
                logging.info(f"Data from {file} added to combined dataset")
            except Exception as e:
                logging.error(f"Error processing file {file}: {e}")

    return current_df, combined_df, output_folder

# Parameters
date_str = "07-05-2024"
specific_dates = ["23-04-2024", "30-04-2024", "07-05-2024"]  # Example dates for inclusion

current_df, combined_df, output_path = setup_paths(date_str, specific_dates)

