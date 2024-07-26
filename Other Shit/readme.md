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


# Configure logging
warnings.simplefilter(action="ignore")
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def preprocess_df(df, date_col='UPDATE DATE', order_col='ORDER CODE'):
    """
    Preprocess the given DataFrame by converting the date column to datetime,
    sorting by the date, and dropping duplicates based on the order column.

    Parameters:
    - df (pandas.DataFrame): The DataFrame to preprocess.
    - date_col (str): The name of the column containing date information. Defaults to 'UPDATE DATE'.
    - order_col (str): The name of the column to check for duplicates. Defaults to 'ORDER CODE'.

    Returns:
    - pandas.DataFrame: The preprocessed DataFrame with duplicates removed and sorted by date.
    """
    # Convert the date column to datetime objects, splitting off the time if present
    df[date_col] = pd.to_datetime(df[date_col].str.split().str[0], format='%d/%m/%Y', errors='coerce')
    # Drop rows with missing dates and sort by the date column
    df = df.dropna(subset=[date_col]).sort_values(date_col)
    # Drop duplicate rows based on the order column, keeping the last occurrence
    df = df.drop_duplicates(subset=order_col, keep='last')
    return df

def setup_paths(date_str, specific_dates):
    """
    Set up the file paths for input and output based on the given date string and specific dates.
    Create the necessary directories if they do not exist.

    Parameters:
    - date_str (str): The date string used to identify the specific files and directories.
    - specific_dates (list): A list of date strings representing specific dates to process.

    Returns:
    - tuple: A tuple containing the current DataFrame, the combined DataFrame of all specific dates,
             and the output path for the visualizations.
    """
    # Define the base folder for data and output
    base_folder = "Visualization"
    data_folder = f"{base_folder}/Data/Orders/"
    output_folder = f"{base_folder}/Graphs/Orders/{date_str}/"
    
    # Create the output directory if it doesn't exist
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    # Construct the full path for the input file
    input_file_path = os.path.join(data_folder, f"Order_Monitoring_{date_str}.csv")
    
    # Attempt to load and preprocess the current DataFrame
    try:
        current_df = pd.read_csv(input_file_path, sep=";")
        logging.info(f"Loaded data from {input_file_path}")
        current_df = preprocess_df(current_df)
        logging.info(f"DataFrame shape after preprocessing: {current_df.shape}")
    except Exception as e:
        logging.error(f"Error processing current data: {e}")
        # Return None for each expected output to indicate failure
        return None, None, None

    # Initialize a counter for the total number of rows before preprocessing
    total_rows_before = 0
    # Initialize an empty DataFrame to combine data from specific dates
    combined_df = pd.DataFrame()
    # Convert the list of specific date strings to datetime objects
    specific_dates_dt = pd.to_datetime(specific_dates, format="%d-%m-%Y")

    # Iterate over files in the data folder
    for file in os.listdir(data_folder):
        # Extract the date part from the filename
        file_date_str = file.split("_")[-1].split('.')[0]
        # Check if the file is a CSV and its date is in the list of specific dates
        if file.endswith(".csv") and pd.to_datetime(file_date_str, format="%d-%m-%Y") in specific_dates_dt:
            try:
                # Load the file into a DataFrame
                df = pd.read_csv(os.path.join(data_folder, file), sep=";")
                # Update the total row count
                total_rows_before += df.shape[0]
                # Preprocess the DataFrame and combine it with the existing data
                df = preprocess_df(df)
                combined_df = pd.concat([combined_df, df], ignore_index=True)
            except Exception as e:
                logging.error(f"Error processing file {file}: {e}")

    # Log the total number of rows before and after preprocessing
    logging.info(f"Total rows before preprocessing: {total_rows_before}")
    logging.info(f"Combined DataFrame shape after preprocessing: {combined_df.shape}")

    # Return the current DataFrame, the combined DataFrame, and the output folder path
    return current_df, combined_df, output_folder

def remove_stores(current_df, combined_df):
    """
    Remove specific stores from the current and combined DataFrames based on order type codes and store names.
    Additionally, calculate the days since 'RUN TIME' for each order.

    Parameters:
    - current_df (pandas.DataFrame): The DataFrame containing the current visualization data.
    - combined_df (pandas.DataFrame): The DataFrame containing the combined data for trend analysis.

    Returns:
    - tuple: A tuple containing the updated current and combined DataFrames.
    """
    # Convert specified columns to datetime format
    date_columns = ['RUN TIME', 'DATE', 'MODIFIED TIME', 'UPDATE DATE']
    for col in date_columns:
        # Apply conversion only if the column exists in the DataFrame
        if col in current_df.columns:
            current_df[col] = pd.to_datetime(current_df[col], dayfirst=True)
        if col in combined_df.columns:
            combined_df[col] = pd.to_datetime(combined_df[col], dayfirst=True)

    # Define order types to be removed 
    order_type_to_drop = ["ZL2", "ZL3", "ZDE", "ZBC"]
    # Remove rows with specified order types
    current_df = current_df[~current_df['ORDER TYPE CODE'].isin(order_type_to_drop)]
    combined_df = combined_df[~combined_df['ORDER TYPE CODE'].isin(order_type_to_drop)]
    logging.info(f"DataFrame shape after removing order types {order_type_to_drop}: current_df: {current_df.shape}, combined_df: {combined_df.shape}")

    # Define substrings for stores to be removed
    substrings_to_drop = ['retailpos', 'indirectretailer', 'fieldCoach']
    # Remove rows where BASE STORE contains any of the specified substrings
    current_df = current_df[~current_df['BASE STORE'].str.contains('|'.join(substrings_to_drop), case=False, na=False)]
    combined_df = combined_df[~combined_df['BASE STORE'].str.contains('|'.join(substrings_to_drop), case=False, na=False)]
    logging.info(f"DataFrame shape after removing stores with substrings {substrings_to_drop}: current_df: {current_df.shape}, combined_df: {combined_df.shape}")
    
    # Calculate the number of days since 'RUN TIME' for each order
    current_df['DAYS SINCE RUN TIME'] = (current_df['RUN TIME'] - current_df['DATE']).dt.days
    combined_df['DAYS SINCE RUN TIME'] = (combined_df['RUN TIME'] - combined_df['DATE']).dt.days
    
    return current_df, combined_df

def apply_LSP_conditions(current_df, combined_df, conditions_config):
    # Initial dataframes shapes for both current and combined
    initial_shape_current = current_df.shape 
    initial_shape_combined = combined_df.shape

    for country, config in conditions_config.items():
        status = config['status']
        days = config['Aging vs creation']

        # Conditions to keep the orders
        condition_to_keep_current = ~((current_df['COUNTRY'] == country) &
                                      (current_df['PMI ORDER STATUS'] == status) &
                                      ((current_df['RUN TIME'] - current_df['DATE']).dt.days <= days))
        current_df = current_df[condition_to_keep_current]

        condition_to_keep_combined = ~((combined_df['COUNTRY'] == country) &
                                       (combined_df['PMI ORDER STATUS'] == status) &
                                       ((combined_df['RUN TIME'] - combined_df['DATE']).dt.days <= days))
        combined_df = combined_df[condition_to_keep_combined]

        # Log the shape after conditions applied
        logging.info(f"After applying conditions: current_df shape {current_df.shape}, combined_df shape {combined_df.shape}")

    # Final logging after processing
    logging.info(f"Final DataFrame shape for current_df: {current_df.shape}, initially was: {initial_shape_current}")
    logging.info(f"Final DataFrame shape for combined_df: {combined_df.shape}, initially was: {initial_shape_combined}")

    return current_df, combined_df

def visualize_graph_1_current(current_df, output_path, include_countries=None, exclude_countries=[]):
    """
    Visualize the count of orders per country per status in a bar plot.

    Parameters:
    - current_df (pandas.DataFrame): DataFrame containing order data.
    - output_path (str): Path to save the output graph image.
    - include_countries (list): List of countries to include in the visualization. Defaults to None, which includes all.
    - exclude_countries (list): List of countries to exclude from the visualization. Defaults to an empty list.

    Returns:
    - None: The function saves the plot to the specified path and displays it.
    """
    try:
        # Apply country filters if specified
        if include_countries:
            current_df = current_df[current_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            current_df = current_df[~current_df['COUNTRY'].isin(exclude_countries)]

        # Define relevant order statuses for the visualization
        relevant_statuses = ['CREATED', 'PROCESSED', 'RECEIVED', 'SHIPPED']
        # Group data by country and order status, counting unique orders
        grouped_data = current_df.groupby(['COUNTRY', 'PMI ORDER STATUS'])['ORDER CODE'].nunique().reset_index()
        # Filter the grouped data for relevant statuses
        filtered_data = grouped_data[grouped_data['PMI ORDER STATUS'].isin(relevant_statuses)]
        # Calculate the total number of orders for scaling the percentages
        total_orders = filtered_data['ORDER CODE'].sum()

        # Check if there is data to visualize
        if filtered_data.empty or total_orders == 0:
            logging.info("No data available for visualization based on the selected criteria.")
            return

        # Set up the subplot grid for the bar plots
        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5), constrained_layout=True)
        fig.suptitle('Count of Orders Per Country Per Status')

        # Iterate over each status to create a subplot
        for i, status in enumerate(relevant_statuses):
            status_data = filtered_data[filtered_data['PMI ORDER STATUS'] == status]
            if status_data.empty:
                logging.info(f"No data for status '{status}'. Skipping this subplot.")
                continue

            # Sort data by order count and calculate percentage of total orders
            status_data.sort_values(by='ORDER CODE', ascending=False, inplace=True)
            status_total = status_data['ORDER CODE'].sum()
            percentage_of_total = (status_total / total_orders) * 100

            # Determine the subplot position
            row, col = divmod(i, 2)
            ax = sns.barplot(x='COUNTRY', y='ORDER CODE', data=status_data, ax=axes[row, col], palette='viridis')
            ax.set_title(f'Status: {status} ({percentage_of_total:.1f}% of total)')
            ax.set_xlabel('Country')
            ax.set_ylabel('Order Count')
            ax.tick_params(axis='x', rotation=45)
            ax.grid(axis='y', linestyle='--', alpha=0.7)

            # Annotate bars with the count
            for bar in ax.patches:
                ax.annotate(f'{int(bar.get_height())}', (bar.get_x() + bar.get_width() / 2., bar.get_height()),
                            ha='center', va='bottom')

            # Add a label for the total count
            ax.text(0.95, 0.95, f'Total: {status_total:,}', transform=ax.transAxes,
                    horizontalalignment='right', verticalalignment='top', fontsize=10, color='black', weight='bold')

        # Save and display the plot
        plt.savefig(os.path.join(output_path, 'Order_Status_Count.png'))
        plt.show()

    except Exception as e:
        logging.error(f"An error occurred during visualization: {e}")

def visualize_graph_2_over_under(current_df, output_path, include_countries=None, exclude_countries=[]):
    """
    Visualize the distribution of orders being under or over 30 days old from the 'UPDATE DATE' for each country and status.

    Parameters:
    - current_df (pandas.DataFrame): DataFrame containing the current data.
    - output_path (str): Path where the visualization will be saved.
    - include_countries (list, optional): Countries to include in the visualization. Defaults to None.
    - exclude_countries (list, optional): Countries to exclude from the visualization. Defaults to an empty list.

    Returns:
    - None: The function saves the plot to the specified path and displays it.
    """
    try:
        # Filter the DataFrame based on the included and excluded countries
        if include_countries:
            current_df = current_df[current_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            current_df = current_df[~current_df['COUNTRY'].isin(exclude_countries)]

        # THIS SEGMENT IS NOT NEEDED ANYMORE -- TO DELETE
            #print(current_df.shape)
            #current_df['RUN TIME'] = pd.to_datetime(current_df['RUN TIME'], errors='coerce')
            #current_df['UPDATE DATE'] = pd.to_datetime(current_df['UPDATE DATE'], errors='coerce')

        # Calculate the age of the orders and categorize them
        current_date = pd.Timestamp('now').normalize()  # This creates a Timestamp for the current date and normalizes to midnight
        # Subtracting the 'UPDATE DATE' from the current date to get a Timedelta object
        current_df['ORDER AGE'] = (current_date - current_df['UPDATE DATE']).dt.days
        # Apply the lambda function to categorize based on 'ORDER AGE'
        current_df['AGE CATEGORY'] = current_df['ORDER AGE'].apply(lambda x: 'Under 30 days' if x < 30 else 'Over 30 days')
        
        # Group the data by country, order status, and age category
        relevant_statuses = ['CREATED', 'PROCESSED', 'RECEIVED', 'SHIPPED']
        grouped_data = current_df.groupby(['COUNTRY', 'PMI ORDER STATUS', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
        filtered_data = grouped_data[grouped_data['PMI ORDER STATUS'].isin(relevant_statuses)]

        # Sort the data for consistent bar order
        filtered_data.sort_values(by=['PMI ORDER STATUS', 'ORDER CODE'], ascending=[True, False], inplace=True)

        # Check if there is data to visualize
        if filtered_data.empty:
            logging.info("No data available for visualization.")
            return

        # Plotting setup
        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5))
        fig.suptitle('Order Age Distribution by Country and Status')

        # Define custom colors for the age categories
        custom_palette = {'Under 30 days': 'DodgerBlue', 'Over 30 days': 'LightSalmon'}

        # Create a bar plot for each order status
        for i, status in enumerate(relevant_statuses):
            ax = axes[i//2, i%2]
            status_data = filtered_data[filtered_data['PMI ORDER STATUS'] == status]

            # Skip plotting if no data is available for the status
            if status_data.empty:
                logging.info(f"No data available for status '{status}'.")
                continue

            sns.barplot(x='COUNTRY', y='ORDER CODE', hue='AGE CATEGORY', data=status_data, ax=ax, palette=custom_palette)
            ax.set_title(f'Orders in {status} Status')
            ax.set_xlabel('Country')
            ax.set_ylabel('Unique Order Count')
            ax.tick_params(axis='x', rotation=45)
            ax.grid(axis='y', linestyle='--', alpha=0.7)
            ax.yaxis.set_major_locator(ticker.MaxNLocator(integer=True))

            # Annotate bars with the count
            for p in ax.patches:
                height = p.get_height()
                if height > 0:  # Only annotate bars with a positive height
                    ax.annotate(f'{int(height)}', (p.get_x() + p.get_width() / 2., height), ha='center', va='bottom')

            # Adjust legend position
            ax.legend(title='Age Category', loc='upper right')

        # Adjust layout and save the plot
        plt.tight_layout(rect=[0, 0, 1, 0.95])
        plt.savefig(os.path.join(output_path, 'Order_Age_Distribution.png'))
        plt.show()

    except Exception as e:
        logging.error(f"An error occurred during visualization: {e}")

def visualize_graph_4_specific(current_df_no_lsp, output_path, country):
    try:
        # Filter for 'SHIPPED' orders
        shipped_orders_df = current_df_no_lsp[current_df_no_lsp['PMI ORDER STATUS'] == 'SHIPPED']

        # Calculate 'ORDER AGE' and create 'AGE CATEGORY'
        current_date = datetime.now()
        shipped_orders_df['ORDER AGE'] = (current_date.date() - shipped_orders_df['UPDATE DATE'].dt.date).apply(lambda x: x.days)
        shipped_orders_df['AGE CATEGORY'] = pd.cut(shipped_orders_df['ORDER AGE'], 
                                                   bins=[-np.inf, 7, 15, 30, np.inf], 
                                                   labels=['Up to 7D', '7-15D', '15-30D', 'Over 30D'])
        
        # Group data by 'COUNTRY' and 'AGE CATEGORY'
        filtered_data_g4 = shipped_orders_df.groupby(['COUNTRY', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
        logging.info(f"DataFrame shape for Graph 4: {filtered_data_g4.shape}")

        # Filter data for the specified country
        country_data = filtered_data_g4[filtered_data_g4['COUNTRY'] == country]

        # Create the bar plot
        plt.figure(figsize=(10, 6))
        ax = sns.barplot(x='AGE CATEGORY', y='ORDER CODE', data=country_data, palette='coolwarm')
        plt.title(f'Order Age Distribution in {country}')
        plt.xlabel('Age Category')
        plt.ylabel('Unique Order Count')
        plt.xticks(rotation=45)
        plt.grid(axis='y', linestyle='--', alpha=0.7)

        # Add labels on top of the bars
        for p in ax.patches:
            ax.annotate(f"{int(p.get_height())}", (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

        plt.tight_layout()
        plt.savefig(os.path.join(output_path, f'{country}_Age_Distribution.png'))
        plt.show()

    except Exception as e:
        logging.error(f"An error occurred during visualization: {e}")

def visualize_graph_5_trend_updated(combined_df, output_path, specific_dates, include_countries=None, exclude_countries=[]):
    """
    Generate and save a trend visualization of order counts over time, filtered by country and order status.

    This function creates a 2x2 subplot, each representing the trend of orders for different statuses.
    It filters the data based on included and excluded countries, groups the data by country and order status,
    and plots the trend of order counts over time using the provided specific dates.

    Parameters:
    - combined_df (pandas.DataFrame): The DataFrame containing the combined data for trend analysis.
    - output_path (str): The directory path where the output graph image will be saved.
    - specific_dates (list): The list of dates to be included in the trend analysis.
    - include_countries (list, optional): The list of countries to include in the visualization. Defaults to None.
    - exclude_countries (list, optional): The list of countries to exclude from the visualization. Defaults to an empty list.

    Returns:
    - None: The function saves the plot to the specified path and displays it.
    """
    try:
        # Apply filters based on the list of countries to include or exclude
        if include_countries:
            combined_df = combined_df[combined_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            combined_df = combined_df[~combined_df['COUNTRY'].isin(exclude_countries)]

        # Exit if there is no data left after filtering
        if combined_df.empty:
            logging.info("No data available after applying country filters.")
            return

        # Group the data by country, order status, and run time, then count the unique orders
        grouped_data = combined_df.groupby(["COUNTRY", "PMI ORDER STATUS", "RUN TIME"])["ORDER CODE"].count().reset_index()

        # Exit if there is no data to plot after grouping
        if grouped_data.empty:
            logging.info("No data available for plotting after grouping.")
            return

        # Prepare the figure and subplots
        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5))
        plt.subplots_adjust(right=0.85)  # Adjust the right space to accommodate the legend
        fig.suptitle('Order Trend by Status')

        # Define the order statuses to be visualized
        order_statuses = ["CREATED", "RECEIVED", "PROCESSED", "SHIPPED"]

        # Plot the data for each order status
        for i, status in enumerate(order_statuses):
            ax = axes[i//2, i%2]
            status_data = grouped_data[grouped_data["PMI ORDER STATUS"] == status]

            # Skip the status if there is no data
            if status_data.empty:
                logging.info(f"No data for status '{status}'. Skipping this subplot.")
                continue

            # Plot the trend for each country
            for country in status_data["COUNTRY"].unique():
                country_data = status_data[status_data["COUNTRY"] == country]
                ax.plot(country_data["RUN TIME"], country_data["ORDER CODE"], marker="o", label=country)

            # Format the subplot
            ax.set_ylim(bottom=0)
            ax.xaxis.set_major_locator(ticker.MaxNLocator(nbins='auto', integer=True))
            ax.yaxis.set_major_locator(ticker.MaxNLocator(integer=True))
            ax.set_title(f'Orders in {status} Status')
            ax.set_xlabel('Date')
            ax.set_ylabel('Order Count')
            ax.tick_params(axis='x', rotation=45)
            ax.legend(title="Country", loc='upper left', bbox_to_anchor=(1,1))
            ax.grid(axis='y', linestyle='--', alpha=0.7)

        # Save and display the plot
        plt.tight_layout(rect=[0, 0, 1, 0.9])  # Adjust the layout to prevent overlap
        plt.savefig(os.path.join(output_path, 'Order_Trend_Analysis.png'))
        plt.show()
        logging.info(f"Visualization saved successfully at: {output_path}")

    except Exception as e:
        logging.error(f"An error occurred during visualization: {e}")




# Setup paths and load data

date_str = "01-08-2024" # -- Date which the report is sent out
specific_dates = ["02-05-2024", "06-05-2024", "09-05-2024", "13-05-2024", "16-05-2024", "20-05-2024",
                  "23-05-2024", "27-05-2024", "30-05-2024", "03-06-2024", "06-06-2024", "10-06-2024",
                  "13-06-2024", "17-06-2024", "20-06-2024", "24-06-2024", "27-06-2024", "01-07-2024",
                  "04-07-2024", "08-07-2024", "11-07-2024", "15-07-2024", "18-07-2024", "22-07-2024",
                  "25-07-2024", "29-07-2024", "01-08-2024"] # All Monday and Thursdays for the trend evolution






conditions_config = {
    "CZ": {
        "status": "SHIPPED",
        "Aging vs creation": 15
    }
}




current_df, combined_df, output_path = setup_paths(date_str, specific_dates)
current_df, combined_df = remove_stores(current_df, combined_df)

# Saving extra dataframes before applying the condition to CZ, to show shipped 15 days after creation
current_df_no_lsp = current_df.copy()
combined_df_no_lsp = combined_df.copy()

# Applying LSP condition for 15 days for CZ
current_df, combined_df = apply_LSP_conditions(current_df, combined_df, conditions_config)






visualize_graph_1_current(current_df, output_path)
    #visualize_graph_1_current(current_df, output_path, exclude_countries=['CZ', 'FR', 'DE'])
    #visualize_graph_1_current(current_df, output_path, include_countries=['CZ', 'FR', 'DE'])

visualize_graph_2_over_under(current_df, output_path)
    #visualize_graph_2_over_under(current_df, output_path, exclude_countries=['CZ', 'FR', 'DE'])
    #visualize_graph_2_over_under(current_df, output_path, include_countries=['CZ', 'FR', 'DE'])

#visualize_graph_4_specific(current_df, output_path, country='CZ')


visualize_graph_5_trend_updated(combined_df, output_path, specific_dates)
    #visualize_graph_5_trend_updated(combined_df, output_path, specific_dates, include_countries=['CZ', 'FR', 'DE'])
    #visualize_graph_5_trend_updated(combined_df, output_path, specific_dates, exclude_countries=['CZ', 'FR', 'DE'])




import os
import pandas as pd
import seaborn as sns
from datetime import datetime
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import warnings
import matplotlib.dates as mdates
warnings.filterwarnings("ignore", category=FutureWarning)

def setup_paths(date_str, specific_dates):
    """
    Set up input and output file paths based on the given date string, load the data into a DataFrame for current visualization,
    and prepare a combined DataFrame for trend analysis based on specific dates.
    
    :param date_str: A string representing the date in the format "dd-mm-yyyy" for current visualization.
    :param specific_dates: A list of strings representing dates for trend analysis.
    :return: DataFrame for current visualization, DataFrame for trend analysis, and the output path for saving visualizations.
    """
    data_folder = "Visualization/Data/Not Exported/"
    output_folder_base = "Visualization/Graphs/Not Exported/"
    
    # For current visualization
    input_file_name = f"Order_Not_Exported_{date_str}.csv"
    output_folder = os.path.join(output_folder_base, date_str)
    input_file_path = os.path.join(data_folder, input_file_name)
    output_path = os.path.join(output_folder)
    
    # Create the output directory if it does not exist
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)
    
    print(f"Input file path for current visualization: {input_file_path}")
    print(f"Output path for current visualization: {output_path}")
    
    # Load the data into a DataFrame for current visualization
    current_df = pd.read_csv(input_file_path, sep=";")
    print(f"Initial DataFrame shape for current visualization: {current_df.shape}")

    # For trend analysis
    specific_dates_dt = pd.to_datetime(specific_dates, format="%d-%m-%Y")  # Correct format specification
    combined_df = pd.DataFrame()

    # Read data from each CSV file and append to combined_df for trend analysis
    for file in os.listdir(data_folder):
        if file.lower().endswith(".csv"):
            file_date_str = file.split("_")[-1].split('.')[0]
            extraction_date = pd.to_datetime(file_date_str, format="%d-%m-%Y")  # Ensure this matches the format in the filenames
            if extraction_date in specific_dates_dt:
                df = pd.read_csv(os.path.join(data_folder, file), sep=";")
                df["Extraction Date"] = extraction_date
                combined_df = combined_df._append(df, ignore_index=True)

    print(f"Combined DataFrame shape for trend analysis: {combined_df.shape}")

    return current_df, combined_df, output_path






def preprocess_data_for_not_exported(current_df, combined_df):
    """
    Preprocess the data for visualizations of "Not Exported" orders.
    - current_df: DataFrame for the current date visualization.
    - combined_df: DataFrame for trend analysis across multiple dates.
    """
    print(f"Initial DataFrame shape for current visualization: {current_df.shape}")
    # Preprocess current_df
    current_df['UPDATE DATE'] = pd.to_datetime(current_df['UPDATE DATE'].str.split().str[0], format='%d/%m/%Y', errors='coerce')
    current_df.dropna(subset=['UPDATE DATE'], inplace=True)
    current_df.sort_values('UPDATE DATE', inplace=True)
    current_df.drop_duplicates(subset='ORDER CODE', keep='last', inplace=True)
    # Calculate 'ORDER AGE' and 'AGE CATEGORY' for Over Under Analysis with a 5-day threshold
    current_date = datetime.now()
    current_df['ORDER AGE'] = (current_date.date() - current_df['UPDATE DATE'].dt.date).apply(lambda x: x.days)
    current_df['AGE CATEGORY'] = current_df['ORDER AGE'].apply(lambda x: 'Under 5 days' if x < 5 else 'Over 5 days')

    print(f"DataFrame shape after preprocessing for current visualization: {current_df.shape}")

    # For Graph 1: Simple count by country
    simple_count_by_country = current_df.groupby(['COUNTRY'])['ORDER CODE'].nunique().reset_index()
    print(f"DataFrame shape for Graph 1 (Simple count by country): {simple_count_by_country.shape}")

    # For Graph 2: Over Under Analysis
    over_under_analysis = current_df.groupby(['COUNTRY', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
    print(f"DataFrame shape for Graph 2 (Over Under Analysis): {over_under_analysis.shape}")

    # For Graph 3: Trend analysis, no further preprocessing needed here for combined_df
    print(f"Initial DataFrame shape for trend analysis: {combined_df.shape}")
    trend_analysis = combined_df.groupby(["COUNTRY", "Extraction Date"])["ORDER CODE"].count().reset_index()
    print(f"DataFrame shape for Graph 3 (Trend Analysis): {trend_analysis.shape}")

    return simple_count_by_country, over_under_analysis, trend_analysis

def visualize_graph_1(simple_count_by_country, output_path):
    """
    Visualize the simple count of "Not Exported" orders per country.
    
    :param simple_count_by_country: DataFrame with the count of orders per country.
    :param output_path: Path where the visualization should be saved.
    """
    fig, ax = plt.subplots(figsize=(10, 6))
    fig.suptitle('Not Exported Orders Per Country')

    # Since all orders are in 'CREATED' status, we directly visualize without status distinction
    sns.barplot(x='COUNTRY', y='ORDER CODE', data=simple_count_by_country, ax=ax, palette='viridis')
    ax.set_xlabel('Country')
    ax.set_ylabel('Unique Order Count')
    ax.tick_params(axis='x', rotation=45)
    ax.grid(axis='y', linestyle='--', alpha=0.7)

    # Annotate bars with the count of orders
    for p in ax.patches:
        ax.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

    plt.tight_layout()
    # Ensure the output directory exists
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    plt.savefig(os.path.join(output_path, 'Not_Exported_Orders_Count.png'))
    plt.show()
    print(f"File saved successfully at: {output_path}")

def visualize_graph_2(filtered_data_g2, output_path):
    """
    Visualize the Over vs Under 5 days analysis for "Not Exported" orders per country.
    
    :param filtered_data_g2: DataFrame with the count of orders per country, including 'AGE CATEGORY'.
    :param output_path: Path where the visualization should be saved.
    """
    fig, ax = plt.subplots(figsize=(10, 6))
    fig.suptitle('Over vs Under 5 Days Per Country')

    sns.barplot(x='COUNTRY', y='ORDER CODE', hue='AGE CATEGORY', data=filtered_data_g2, palette='coolwarm', ax=ax)
    ax.set_xlabel('Country')
    ax.set_ylabel('Unique Order Count')
    ax.tick_params(axis='x', rotation=45)
    ax.grid(axis='y', linestyle='--', alpha=0.7)

    for p in ax.patches:
        ax.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

    plt.tight_layout()
    # Ensure the output directory exists
    os.makedirs(os.path.dirname(output_path), exist_ok=True)
    plt.savefig(os.path.join(output_path, 'Not_Exported_Over_Under_5_days.png'))
    plt.show()
    print(f"File saved successfully at: {output_path}")

def visualize_graph_3(filtered_data_g3, specific_dates, output_path):
    """
    Visualize the trend analysis for "Not Exported" orders over time.
    
    :param filtered_data_g3: DataFrame with aggregated order counts per country over time.
    :param specific_dates: List of specific dates for trend analysis.
    :param output_path: Path where the visualization should be saved.
    """
    specific_dates_dt = pd.to_datetime(specific_dates, format='%d-%m-%Y')
    
    fig, ax = plt.subplots(figsize=(12, 7))
    fig.suptitle('Order Trend Analysis')

    # Calculate total orders across all countries for each specific date
    total_orders = filtered_data_g3.groupby("Extraction Date")["ORDER CODE"].sum()

    # Plot individual country data
    for country in filtered_data_g3["COUNTRY"].unique():
        country_data = filtered_data_g3[filtered_data_g3["COUNTRY"] == country]
        ax.plot(country_data["Extraction Date"], country_data["ORDER CODE"], marker="o", label=country)

    # Plot the total orders line
    ax.plot(total_orders.index, total_orders.values, marker="o", label="Total", linestyle="--", color="black")

    ax.yaxis.set_major_locator(ticker.MaxNLocator(integer=True))
    ax.set_xlabel('Extraction Date')
    ax.set_ylabel('Order Count')
    ax.set_xticks(specific_dates_dt)
    ax.set_xticklabels(specific_dates_dt.strftime('%Y-%m-%d'), rotation=45, ha="right")
    ax.legend(title="Country")
    ax.grid(axis='y', linestyle='--', alpha=0.7)

    plt.tight_layout()
    plt.savefig(os.path.join(output_path, 'Not_Exported_Trend_Analysis.png'))
    plt.show()
    print(f"File saved successfully at: {output_path}")

def visualize_all_graphs_together(filtered_data_g1, filtered_data_g2, filtered_data_g3, specific_dates, output_path):
    """
    Visualizes all three graphs (Trend Analysis, Over & Under, Simple Count) side by side in a single figure.
    
    :param filtered_data_g1: DataFrame for simple count by country.
    :param filtered_data_g2: DataFrame for Over & Under analysis.
    :param filtered_data_g3: DataFrame for trend analysis.
    :param specific_dates: List of specific dates for trend analysis.
    :param output_path: Path where the combined visualization should be saved.
    """
    # Convert specific dates for plotting
    specific_dates_dt = pd.to_datetime(specific_dates, format='%d-%m-%Y')
    
    # Create figure and subplots
    fig, axes = plt.subplots(1, 3, figsize=(20, 6), sharey='row')
    fig.suptitle('Not Exported - Order Analysis')
    
    # Trend Analysis on the first subplot
    for country in filtered_data_g3["COUNTRY"].unique():
        country_data = filtered_data_g3[filtered_data_g3["COUNTRY"] == country]
        axes[0].plot(country_data["Extraction Date"], country_data["ORDER CODE"], marker="o", label=country)
    
    # Calculate total orders across all countries for each specific date
    total_orders = filtered_data_g3.groupby("Extraction Date")["ORDER CODE"].sum()
    axes[0].plot(total_orders.index, total_orders.values, marker="o", label="Total", linestyle="--", color="black")

    axes[0].set_title('Trend Analysis')
    axes[0].set_xlabel('Extraction Date')
    axes[0].set_ylabel('Order Count')
    axes[0].set_xticks(specific_dates_dt)
    axes[0].set_xticklabels(specific_dates_dt.strftime('%Y-%m-%d'), rotation=45, ha="right")
    axes[0].legend(title="Country")
    axes[0].grid(axis='y', linestyle='--', alpha=0.7)

    # Over & Under Analysis on the second subplot
    over_under_barplot = sns.barplot(x='COUNTRY', y='ORDER CODE', hue='AGE CATEGORY', data=filtered_data_g2, palette='coolwarm', ax=axes[1])
    axes[1].set_title('Over & Under 5 Days')
    axes[1].set_xlabel('Country')
    axes[1].tick_params(axis='x', rotation=45)
    axes[1].grid(axis='y', linestyle='--', alpha=0.7)
    # Add labels on top of each bar for Over & Under Analysis
    for p in over_under_barplot.patches:
        over_under_barplot.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                                    textcoords='offset points')

    # Simple Count by Country on the third subplot
    simple_count_barplot = sns.barplot(x='COUNTRY', y='ORDER CODE', data=filtered_data_g1, palette='viridis', ax=axes[2])
    axes[2].set_title('Count by Country')
    axes[2].set_xlabel('Country')
    axes[2].tick_params(axis='x', rotation=45)
    axes[2].grid(axis='y', linestyle='--', alpha=0.7)
    # Add labels on top of each bar for Simple Count by Country
    for p in simple_count_barplot.patches:
        simple_count_barplot.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                                    textcoords='offset points')


    plt.tight_layout()
    plt.savefig(os.path.join(output_path, 'Not Exported Analysis.png'))
    plt.show()
    print(f"File saved successfully at: {output_path}")

def visualize_all_graphs_together_newaxis(filtered_data_g1, filtered_data_g2, filtered_data_g3, specific_dates, output_path):
    """
    Visualizes all three graphs (Trend Analysis, Over & Under, Simple Count) side by side in a single figure.
    
    :param filtered_data_g1: DataFrame for simple count by country.
    :param filtered_data_g2: DataFrame for Over & Under analysis.
    :param filtered_data_g3: DataFrame for trend analysis.
    :param specific_dates: List of specific dates for trend analysis.
    :param output_path: Path where the combined visualization should be saved.
    """
    # Convert specific dates for plotting
    specific_dates = pd.to_datetime(specific_dates, format='%d-%m-%Y')
    
    # Create figure and subplots
    fig, axes = plt.subplots(1, 3, figsize=(20, 6), sharey='row')
    fig.suptitle('Not Exported - Order Analysis')
    
    # Trend Analysis on the first subplot
    for country in filtered_data_g3["COUNTRY"].unique():
        country_data = filtered_data_g3[filtered_data_g3["COUNTRY"] == country]
        axes[0].plot(country_data["Extraction Date"], country_data["ORDER CODE"], marker="o", label=country)

    
    # Calculate total orders across all countries for each specific date
    total_orders = filtered_data_g3.groupby("Extraction Date")["ORDER CODE"].sum()


    specific_dates = pd.to_datetime(specific_dates, format='%m-%d').date
    axes[0].plot(total_orders.index, total_orders.values, marker="o", label="Total", linestyle="--", color="black")
    axes[0].xaxis.set_major_locator(ticker.MaxNLocator(nbins='auto', integer=True))
    axes[0].xaxis.set_major_formatter(ticker.FuncFormatter(lambda x, _: datetime.fromordinal(int(x)).strftime('%m-%d')))
    axes[0].yaxis.set_major_locator(ticker.MaxNLocator(integer=True))
    axes[0].tick_params(axis='x', rotation=45)

    axes[0].set_title('Trend Analysis')
    axes[0].set_xlabel('Extraction Date')
    axes[0].set_ylabel('Order Count')
    axes[0].legend(title="Country")
    axes[0].grid(axis='y', linestyle='--', alpha=0.7)

    # Over & Under Analysis on the second subplot
    over_under_barplot = sns.barplot(x='COUNTRY', y='ORDER CODE', hue='AGE CATEGORY', data=filtered_data_g2, palette='coolwarm', ax=axes[1])
    axes[1].set_title('Over & Under 5 Days')
    axes[1].set_xlabel('Country')
    axes[1].tick_params(axis='x', rotation=45)
    axes[1].grid(axis='y', linestyle='--', alpha=0.7)
    # Add labels on top of each bar for Over & Under Analysis
    for p in over_under_barplot.patches:
        over_under_barplot.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                                    textcoords='offset points')

    # Simple Count by Country on the third subplot
    simple_count_barplot = sns.barplot(x='COUNTRY', y='ORDER CODE', data=filtered_data_g1, palette='viridis', ax=axes[2])
    axes[2].set_title('Count by Country')
    axes[2].set_xlabel('Country')
    axes[2].tick_params(axis='x', rotation=45)
    axes[2].grid(axis='y', linestyle='--', alpha=0.7)
    # Add labels on top of each bar for Simple Count by Country
    for p in simple_count_barplot.patches:
        simple_count_barplot.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                                    textcoords='offset points')


    plt.tight_layout()
    plt.savefig(os.path.join(output_path, 'Not Exported Analysis.png'))
    plt.show()
    print(f"File saved successfully at: {output_path}")




# Removed
# "24-03-2024", "25-03-2024", "29-03-2024"


# Setup paths and load data
date_str = "26-07-2024" # Date from the report

specific_dates = ["02-05-2024", "03-05-2024", "06-05-2024", "07-05-2024", "08-05-2024",
                   "09-05-2024", "10-05-2024", "13-05-2024", "14-05-2024", "15-05-2024",
                   "16-05-2024","17-05-2024", "20-05-2024","21-05-2024","22-05-2024",
                   "23-05-2024","24-05-2024", "26-05-2024","27-05-2024", "28-05-2024",
                   "29-05-2024", "30-05-2024", "03-06-2024", "04-06-2024", "05-06-2024",
                   "06-06-2024","07-06-2024", "10-06-2024", "11-06-2024", "12-06-2024",
                   "13-06-2024", "14-06-2024", "17-06-2024", "18-06-2024", "19-06-2024",
                   "20-06-2024","21-06-2024", "24-06-2024", "25-06-2024", "26-06-2024",
                   "27-06-2024",'01-07-2024','02-07-2024','04-07-2024','05-07-2024',
                   '08-07-2024','09-07-2024','10-07-2024', '11-07-2024', '12-07-2024',
                   '15-07-2024', '16-07-2024', '17-07-2024', '18-07-2024','19-07-2024',
                   '22-07-2024', '23-07-2024',  '24-07-2024',  '25-07-2024', "26-07-2024"]




current_df, combined_df, output_path = setup_paths(date_str, specific_dates)
filtered_data_g1, filtered_data_g2, filtered_data_g3 = preprocess_data_for_not_exported(current_df, combined_df)

#visualize_graph_1(filtered_data_g1, output_path)
#visualize_graph_2(filtered_data_g2, output_path)
#visualize_graph_3(filtered_data_g3, specific_dates, output_path)
#visualize_all_graphs_together(filtered_data_g1, filtered_data_g2, filtered_data_g3, specific_dates, output_path)

 
visualize_all_graphs_together_newaxis(filtered_data_g1, filtered_data_g2, filtered_data_g3, specific_dates, output_path)




