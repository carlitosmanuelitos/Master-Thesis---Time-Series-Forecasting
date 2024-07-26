import os
import pandas as pd
import seaborn as sns
from datetime import datetime
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import warnings
import matplotlib.dates as mdates
import logging
from functools import wraps

warnings.filterwarnings("ignore", category=FutureWarning)

# Configuration Section
CONFIG = {
    'DATA_FOLDER': "Visualization/Data/Not Exported/",
    'OUTPUT_FOLDER_BASE': "Visualization/Graphs/Not Exported/",
    'DATE_FORMAT': "%d-%m-%Y",
    'CSV_SEPARATOR': ";",
    'LOG_FILE': 'order_analysis.log',
    'LOG_LEVEL': logging.INFO,
    'LOG_FORMAT': '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    'CURRENT_DATE': "26-07-2024",
    'SPECIFIC_DATES': [
        "02-05-2024", "03-05-2024", "06-05-2024", "07-05-2024", "08-05-2024",
        "09-05-2024", "10-05-2024", "13-05-2024", "14-05-2024", "15-05-2024",
        "16-05-2024", "17-05-2024", "20-05-2024", "21-05-2024", "22-05-2024",
        "23-05-2024", "24-05-2024", "26-05-2024", "27-05-2024", "28-05-2024",
        "29-05-2024", "30-05-2024", "03-06-2024", "04-06-2024", "05-06-2024",
        "06-06-2024", "07-06-2024", "10-06-2024", "11-06-2024", "12-06-2024",
        "13-06-2024", "14-06-2024", "17-06-2024", "18-06-2024", "19-06-2024",
        "20-06-2024", "21-06-2024", "24-06-2024", "25-06-2024", "26-06-2024",
        "27-06-2024", '01-07-2024', '02-07-2024', '04-07-2024', '05-07-2024',
        '08-07-2024', '09-07-2024', '10-07-2024', '11-07-2024', '12-07-2024',
        '15-07-2024', '16-07-2024', '17-07-2024', '18-07-2024', '19-07-2024',
        '22-07-2024', '23-07-2024', '24-07-2024', '25-07-2024', "26-07-2024"
    ]
}

# Hardcoded configurations
GRAPH_TITLE = 'Not Exported - Order Analysis'
GRAPH_SIZE = (20, 6)
TREND_ANALYSIS_TITLE = 'Trend Analysis'
OVER_UNDER_TITLE = 'Over & Under 5 Days'
COUNT_BY_COUNTRY_TITLE = 'Count by Country'
AGE_THRESHOLD = 5

# Logging setup
logging.basicConfig(level=CONFIG['LOG_LEVEL'],
                    format=CONFIG['LOG_FORMAT'],
                    filename=CONFIG['LOG_FILE'])

logger = logging.getLogger(__name__)

# Error handling decorator
def log_exception(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logger.exception(f"Exception in {func.__name__}: {str(e)}")
            raise
    return wrapper

def validate_date_str(date_str):
    """
    Validate if a given date string matches the expected format.

    Args:
        date_str (str): The date string to validate.

    Returns:
        bool: True if the date string is valid, False otherwise.

    Raises:
        ValueError: If the date string doesn't match the expected format.
    """
    try:
        datetime.strptime(date_str, CONFIG['DATE_FORMAT'])
        return True
    except ValueError:
        logger.error(f"Invalid date format: {date_str}. Expected format: {CONFIG['DATE_FORMAT']}")
        return False

@log_exception
def read_csv_file(file_path):
    """
    Read a CSV file and return a pandas DataFrame.

    Args:
        file_path (str): Path to the CSV file.

    Returns:
        pd.DataFrame: DataFrame containing the CSV data.

    Raises:
        FileNotFoundError: If the file is not found.
        pd.errors.EmptyDataError: If the file is empty.
        pd.errors.ParserError: If there's an error parsing the CSV file.
    """
    if not os.path.exists(file_path):
        logger.error(f"File not found: {file_path}")
        raise FileNotFoundError(f"File not found: {file_path}")

    try:
        df = pd.read_csv(file_path, sep=CONFIG['CSV_SEPARATOR'])
        
        # Check for required columns
        required_columns = ['ORDER CODE', 'UPDATE DATE', 'COUNTRY']
        missing_columns = [col for col in required_columns if col not in df.columns]
        if missing_columns:
            raise ValueError(f"Missing required columns: {', '.join(missing_columns)}")

        # Check for empty DataFrame
        if df.empty:
            logger.warning(f"Empty CSV file: {file_path}")

        return df
    except pd.errors.EmptyDataError:
        logger.error(f"Empty CSV file: {file_path}")
        raise
    except pd.errors.ParserError:
        logger.error(f"Error parsing CSV file: {file_path}")
        raise

@log_exception
def setup_paths(date_str, specific_dates):
    """
    Set up input and output file paths and load data for analysis.

    This function validates input dates, creates necessary directories,
    loads the current data, and combines data from multiple dates for trend analysis.

    Args:
        date_str (str): The current date for analysis in the format specified in CONFIG['DATE_FORMAT'].
        specific_dates (list): List of dates for trend analysis in the format specified in CONFIG['DATE_FORMAT'].

    Returns:
        tuple: A tuple containing:
            - current_df (pd.DataFrame): DataFrame with current date's data.
            - combined_df (pd.DataFrame): DataFrame with data from all specific dates.
            - output_path (str): Path where output files will be saved.

    Raises:
        ValueError: If date_str or any date in specific_dates is invalid.
        FileNotFoundError: If input files are not found.
    """
    # Validate input dates
    if not validate_date_str(date_str):
        raise ValueError(f"Invalid date string: {date_str}")

    for date in specific_dates:
        if not validate_date_str(date):
            raise ValueError(f"Invalid date in specific_dates: {date}")

    # Set up file paths
    input_file_name = f"Order_Not_Exported_{date_str}.csv"
    output_folder = os.path.join(CONFIG['OUTPUT_FOLDER_BASE'], date_str)
    input_file_path = os.path.join(CONFIG['DATA_FOLDER'], input_file_name)
    output_path = output_folder

    # Create output directory if it doesn't exist
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    logger.info(f"Input file path: {input_file_path}")
    logger.info(f"Output path: {output_path}")

    # Load current date's data
    current_df = read_csv_file(input_file_path)
    logger.info(f"Initial DataFrame shape: {current_df.shape}")

    # Load and combine data for trend analysis
    specific_dates_dt = pd.to_datetime(specific_dates, format=CONFIG['DATE_FORMAT'])
    combined_df = pd.DataFrame()

    for file in os.listdir(CONFIG['DATA_FOLDER']):
        if file.lower().endswith(".csv"):
            file_date_str = file.split("_")[-1].split('.')[0]
            extraction_date = pd.to_datetime(file_date_str, format=CONFIG['DATE_FORMAT'])
            if extraction_date in specific_dates_dt:
                df = read_csv_file(os.path.join(CONFIG['DATA_FOLDER'], file))
                df["Extraction Date"] = extraction_date
                combined_df = combined_df._append(df, ignore_index=True)

    logger.info(f"Combined DataFrame shape: {combined_df.shape}")

    return current_df, combined_df, output_path

@log_exception
def preprocess_data_for_not_exported(current_df, combined_df):
    """
    Preprocess the data for not exported orders analysis.

    This function performs several data cleaning and transformation steps:
    1. Converts 'UPDATE DATE' to datetime format.
    2. Removes rows with invalid dates.
    3. Calculates the age of each order.
    4. Categorizes orders based on their age.
    5. Aggregates data for different types of analysis.

    Args:
        current_df (pd.DataFrame): DataFrame containing the current date's data.
        combined_df (pd.DataFrame): DataFrame containing data from multiple dates for trend analysis.

    Returns:
        tuple: A tuple containing three DataFrames:
            - simple_count_by_country: Aggregated count of orders by country.
            - over_under_analysis: Analysis of orders over and under 5 days old by country.
            - trend_analysis: Trend of order counts over time by country.

    Raises:
        ValueError: If input DataFrames are empty.
    """
    logger.info("Preprocessing data for not exported orders")

    # Validate input DataFrames
    if current_df.empty or combined_df.empty:
        raise ValueError("Input DataFrames cannot be empty")

    # Convert 'UPDATE DATE' to datetime and handle errors
    current_df['UPDATE DATE'] = pd.to_datetime(current_df['UPDATE DATE'].str.split().str[0], 
                                               format='%d/%m/%Y', errors='coerce')
    
    # Check for and log any rows where date conversion failed
    invalid_dates = current_df[current_df['UPDATE DATE'].isna()]
    if not invalid_dates.empty:
        logger.warning(f"Found {len(invalid_dates)} rows with invalid dates")
        logger.debug(f"Invalid date rows: {invalid_dates['ORDER CODE'].tolist()}")

    # Remove rows with invalid dates
    current_df.dropna(subset=['UPDATE DATE'], inplace=True)

    # Sort and remove duplicates
    current_df.sort_values('UPDATE DATE', inplace=True)
    current_df.drop_duplicates(subset='ORDER CODE', keep='last', inplace=True)

    # Calculate order age and categorize
    current_date = datetime.now()
    current_df['ORDER AGE'] = (current_date.date() - current_df['UPDATE DATE'].dt.date).apply(lambda x: x.days)
    current_df['AGE CATEGORY'] = current_df['ORDER AGE'].apply(lambda x: 'Under 5 days' if x < AGE_THRESHOLD else 'Over 5 days')

    # Aggregate data for different analyses
    simple_count_by_country = current_df.groupby(['COUNTRY'])['ORDER CODE'].nunique().reset_index()
    over_under_analysis = current_df.groupby(['COUNTRY', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
    trend_analysis = combined_df.groupby(["COUNTRY", "Extraction Date"])["ORDER CODE"].count().reset_index()

    return simple_count_by_country, over_under_analysis, trend_analysis

def plot_trend_analysis(ax, filtered_data_g3):
    """
    Plot the trend analysis graph.

    Args:
        ax (matplotlib.axes.Axes): The axes to plot on.
        filtered_data_g3 (pd.DataFrame): DataFrame containing trend data.
    """
    for country in filtered_data_g3["COUNTRY"].unique():
        country_data = filtered_data_g3[filtered_data_g3["COUNTRY"] == country]
        ax.plot(country_data["Extraction Date"], country_data["ORDER CODE"], marker="o", label=country)

    total_orders = filtered_data_g3.groupby("Extraction Date")["ORDER CODE"].sum()
    ax.plot(total_orders.index, total_orders.values, marker="o", label="Total", linestyle="--", color="black")

    ax.set_title(TREND_ANALYSIS_TITLE)
    ax.set_xlabel('Extraction Date')
    ax.set_ylabel('Order Count')
    ax.legend(title="Country")
    ax.grid(axis='y', linestyle='--', alpha=0.7)

def plot_over_under_analysis(ax, filtered_data_g2):
    """
    Plot the over and under analysis graph.

    Args:
        ax (matplotlib.axes.Axes): The axes to plot on.
        filtered_data_g2 (pd.DataFrame): DataFrame containing over/under analysis data.
    """
    sns.barplot(x='COUNTRY', y='ORDER CODE', hue='AGE CATEGORY', data=filtered_data_g2, palette='coolwarm', ax=ax)
    ax.set_title(OVER_UNDER_TITLE)
    ax.set_xlabel('Country')
    ax.tick_params(axis='x', rotation=45)
    ax.grid(axis='y', linestyle='--', alpha=0.7)

    for p in ax.patches:
        ax.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                    textcoords='offset points')

def plot_count_by_country(ax, filtered_data_g1):
    """
    Plot the count by country graph.

    Args:
        ax (matplotlib.axes.Axes): The axes to plot on.
        filtered_data_g1 (pd.DataFrame): DataFrame containing count by country data.
    """
    sns.barplot(x='COUNTRY', y='ORDER CODE', data=filtered_data_g1, palette='viridis', ax=ax)
    ax.set_title(COUNT_BY_COUNTRY_TITLE)
    ax.set_xlabel('Country')
    ax.tick_params(axis='x', rotation=45)
    ax.grid(axis='y', linestyle='--', alpha=0.7)

    for p in ax.patches:
        ax.annotate(f'{int(p.get_height())}', (p.get_x() + p.get_width() / 2., p.get_height()),
                    ha='center', va='center', fontsize=10, color='black', xytext=(0, 5),
                    textcoords='offset points')

@log_exception
def visualize_all_graphs_together_newaxis(filtered_data_g1, filtered_data_g2, filtered_data_g3, specific_dates, output_path):
    """
    Create a combined visualization of all graphs.

    Args:
        filtered_data_g1 (pd.DataFrame): DataFrame for count by country.
        filtered_data_g2 (pd.DataFrame): DataFrame for over/under analysis.
        filtered_data_g3 (pd.DataFrame): DataFrame for trend analysis.
        specific_dates (list): List of dates for trend analysis.
        output_path (str): Path to save the output graph.

    Raises:
        ValueError: If input data is invalid or output path doesn't exist.
    """
    # Input validation
    if any(df.empty for df in [filtered_data_g1, filtered_data_g2, filtered_data_g3]):
        raise ValueError("Input DataFrames cannot be empty")
    if not specific_dates:
        raise ValueError("specific_dates list cannot be empty")
    if not os.path.exists(os.path.dirname(output_path)):
        raise ValueError(f"Output directory does not exist: {os.path.dirname(output_path)}")

    # Create figure and subplots
    fig, axes = plt.subplots(1, 3, figsize=GRAPH_SIZE, sharey='row')
    fig.suptitle(GRAPH_TITLE)

    # Plot each graph
    plot_trend_analysis(axes[0], filtered_data_g3)
    plot_over_under_analysis(axes[1], filtered_data_g2)
    plot_count_by_country(axes[2], filtered_data_g1)

    # Adjust layout and save
    plt.tight_layout()
    plt.savefig(os.path.join(output_path, 'Not Exported Analysis.png'))
    plt.close()

    logger.info(f"Graph saved successfully at: {output_path}")

# Main execution
if __name__ == "__main__":
    logger.info("Starting order analysis script")
    
    try:
        current_df, combined_df, output_path = setup_paths(CONFIG['CURRENT_DATE'], CONFIG['SPECIFIC_DATES'])
        filtered_data_g1, filtered_data_g2, filtered_data_g3 = preprocess_data_for_not_exported(current_df, combined_df)
        visualize_all_graphs_together_newaxis(filtered_data_g1, filtered_data_g2, filtered_data_g3, CONFIG['SPECIFIC_DATES'], output_path)
    except Exception as e:
        logger.error(f"An error occurred during script execution: {str(e)}")
    else:
        logger.info("Order analysis script completed successfully")
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
        
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

# Configuration Section
CONFIG = {
    'BASE_FOLDER': "Visualization",
    'DATA_FOLDER': "Visualization/Data/Orders/",
    'OUTPUT_FOLDER_BASE': "Visualization/Graphs/Orders/",
    'DATE_FORMAT': "%d-%m-%Y",
    'CSV_SEPARATOR': ";",
    'LOG_FILE': 'order_analysis.log',
    'LOG_LEVEL': logging.INFO,
    'LOG_FORMAT': '%(asctime)s - %(levelname)s - %(message)s',
    'CURRENT_DATE': "01-08-2024",
    'SPECIFIC_DATES': [
        "02-05-2024", "06-05-2024", "09-05-2024", "13-05-2024", "16-05-2024", "20-05-2024",
        "23-05-2024", "27-05-2024", "30-05-2024", "03-06-2024", "06-06-2024", "10-06-2024",
        "13-06-2024", "17-06-2024", "20-06-2024", "24-06-2024", "27-06-2024", "01-07-2024",
        "04-07-2024", "08-07-2024", "11-07-2024", "15-07-2024", "18-07-2024", "22-07-2024",
        "25-07-2024", "29-07-2024", "01-08-2024"
    ],
    'ORDER_TYPE_TO_DROP': ["ZL2", "ZL3", "ZDE", "ZBC"],
    'SUBSTRINGS_TO_DROP': ['retailpos', 'indirectretailer', 'fieldCoach'],
    'RELEVANT_STATUSES': ['CREATED', 'PROCESSED', 'RECEIVED', 'SHIPPED'],
    'AGE_THRESHOLD': 30,
    'CONDITIONS_CONFIG': {
        "CZ": {
            "status": "SHIPPED",
            "Aging vs creation": 15
        }
    }
}

# Configure logging
warnings.simplefilter(action="ignore")
logging.basicConfig(level=CONFIG['LOG_LEVEL'], format=CONFIG['LOG_FORMAT'], filename=CONFIG['LOG_FILE'])
logger = logging.getLogger(__name__)

def log_exception(func):
    """Decorator to log exceptions raised by functions."""
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            logger.exception(f"Exception in {func.__name__}: {str(e)}")
            raise
    return wrapper

@log_exception
def preprocess_df(df, date_col='UPDATE DATE', order_col='ORDER CODE'):
    """
    Preprocess the given DataFrame by converting the date column to datetime,
    sorting by the date, and dropping duplicates based on the order column.

    Args:
        df (pd.DataFrame): The DataFrame to preprocess.
        date_col (str): The name of the column containing date information.
        order_col (str): The name of the column to check for duplicates.

    Returns:
        pd.DataFrame: The preprocessed DataFrame with duplicates removed and sorted by date.
    """
    df[date_col] = pd.to_datetime(df[date_col].str.split().str[0], format='%d/%m/%Y', errors='coerce')
    df = df.dropna(subset=[date_col]).sort_values(date_col)
    df = df.drop_duplicates(subset=order_col, keep='last')
    return df

@log_exception
def setup_paths(date_str, specific_dates):
    """
    Set up the file paths for input and output based on the given date string and specific dates.
    Create the necessary directories if they do not exist.

    Args:
        date_str (str): The date string used to identify the specific files and directories.
        specific_dates (list): A list of date strings representing specific dates to process.

    Returns:
        tuple: A tuple containing the current DataFrame, the combined DataFrame of all specific dates,
               and the output path for the visualizations.
    """
    output_folder = os.path.join(CONFIG['OUTPUT_FOLDER_BASE'], date_str)
    os.makedirs(output_folder, exist_ok=True)

    input_file_path = os.path.join(CONFIG['DATA_FOLDER'], f"Order_Monitoring_{date_str}.csv")
    
    try:
        current_df = pd.read_csv(input_file_path, sep=CONFIG['CSV_SEPARATOR'])
        logger.info(f"Loaded data from {input_file_path}")
        current_df = preprocess_df(current_df)
        logger.info(f"DataFrame shape after preprocessing: {current_df.shape}")
    except Exception as e:
        logger.error(f"Error processing current data: {e}")
        return None, None, None

    total_rows_before = 0
    combined_df = pd.DataFrame()
    specific_dates_dt = pd.to_datetime(specific_dates, format=CONFIG['DATE_FORMAT'])

    for file in os.listdir(CONFIG['DATA_FOLDER']):
        file_date_str = file.split("_")[-1].split('.')[0]
        if file.endswith(".csv") and pd.to_datetime(file_date_str, format=CONFIG['DATE_FORMAT']) in specific_dates_dt:
            try:
                df = pd.read_csv(os.path.join(CONFIG['DATA_FOLDER'], file), sep=CONFIG['CSV_SEPARATOR'])
                total_rows_before += df.shape[0]
                df = preprocess_df(df)
                combined_df = pd.concat([combined_df, df], ignore_index=True)
            except Exception as e:
                logger.error(f"Error processing file {file}: {e}")

    logger.info(f"Total rows before preprocessing: {total_rows_before}")
    logger.info(f"Combined DataFrame shape after preprocessing: {combined_df.shape}")

    return current_df, combined_df, output_folder

@log_exception
def remove_stores(current_df, combined_df):
    """
    Remove specific stores from the current and combined DataFrames based on order type codes and store names.
    Additionally, calculate the days since 'RUN TIME' for each order.

    Args:
        current_df (pd.DataFrame): The DataFrame containing the current visualization data.
        combined_df (pd.DataFrame): The DataFrame containing the combined data for trend analysis.

    Returns:
        tuple: A tuple containing the updated current and combined DataFrames.
    """
    date_columns = ['RUN TIME', 'DATE', 'MODIFIED TIME', 'UPDATE DATE']
    for col in date_columns:
        if col in current_df.columns:
            current_df[col] = pd.to_datetime(current_df[col], dayfirst=True)
        if col in combined_df.columns:
            combined_df[col] = pd.to_datetime(combined_df[col], dayfirst=True)

    current_df = current_df[~current_df['ORDER TYPE CODE'].isin(CONFIG['ORDER_TYPE_TO_DROP'])]
    combined_df = combined_df[~combined_df['ORDER TYPE CODE'].isin(CONFIG['ORDER_TYPE_TO_DROP'])]
    logger.info(f"DataFrame shape after removing order types: current_df: {current_df.shape}, combined_df: {combined_df.shape}")

    current_df = current_df[~current_df['BASE STORE'].str.contains('|'.join(CONFIG['SUBSTRINGS_TO_DROP']), case=False, na=False)]
    combined_df = combined_df[~combined_df['BASE STORE'].str.contains('|'.join(CONFIG['SUBSTRINGS_TO_DROP']), case=False, na=False)]
    logger.info(f"DataFrame shape after removing stores: current_df: {current_df.shape}, combined_df: {combined_df.shape}")
    
    current_df['DAYS SINCE RUN TIME'] = (current_df['RUN TIME'] - current_df['DATE']).dt.days
    combined_df['DAYS SINCE RUN TIME'] = (combined_df['RUN TIME'] - combined_df['DATE']).dt.days
    
    return current_df, combined_df

@log_exception
def apply_LSP_conditions(current_df, combined_df, conditions_config):
    """
    Apply LSP conditions to filter orders based on country-specific criteria.

    Args:
        current_df (pd.DataFrame): The current DataFrame to filter.
        combined_df (pd.DataFrame): The combined DataFrame to filter.
        conditions_config (dict): Configuration specifying conditions for each country.

    Returns:
        tuple: Filtered current and combined DataFrames.
    """
    initial_shape_current = current_df.shape 
    initial_shape_combined = combined_df.shape

    for country, config in conditions_config.items():
        status = config['status']
        days = config['Aging vs creation']

        condition_to_keep_current = ~((current_df['COUNTRY'] == country) &
                                      (current_df['PMI ORDER STATUS'] == status) &
                                      ((current_df['RUN TIME'] - current_df['DATE']).dt.days <= days))
        current_df = current_df[condition_to_keep_current]

        condition_to_keep_combined = ~((combined_df['COUNTRY'] == country) &
                                       (combined_df['PMI ORDER STATUS'] == status) &
                                       ((combined_df['RUN TIME'] - combined_df['DATE']).dt.days <= days))
        combined_df = combined_df[condition_to_keep_combined]

        logger.info(f"After applying conditions: current_df shape {current_df.shape}, combined_df shape {combined_df.shape}")

    logger.info(f"Final DataFrame shape for current_df: {current_df.shape}, initially was: {initial_shape_current}")
    logger.info(f"Final DataFrame shape for combined_df: {combined_df.shape}, initially was: {initial_shape_combined}")

    return current_df, combined_df

@log_exception
def visualize_graph_1_current(current_df, output_path, include_countries=None, exclude_countries=[]):
    """
    Visualize the count of orders per country per status in a bar plot.

    Args:
        current_df (pd.DataFrame): DataFrame containing order data.
        output_path (str): Path to save the output graph image.
        include_countries (list, optional): List of countries to include in the visualization.
        exclude_countries (list, optional): List of countries to exclude from the visualization.
    """
    try:
        if include_countries:
            current_df = current_df[current_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            current_df = current_df[~current_df['COUNTRY'].isin(exclude_countries)]

        grouped_data = current_df.groupby(['COUNTRY', 'PMI ORDER STATUS'])['ORDER CODE'].nunique().reset_index()
        filtered_data = grouped_data[grouped_data['PMI ORDER STATUS'].isin(CONFIG['RELEVANT_STATUSES'])]
        total_orders = filtered_data['ORDER CODE'].sum()

        if filtered_data.empty or total_orders == 0:
            logger.info("No data available for visualization based on the selected criteria.")
            return

        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5), constrained_layout=True)
        fig.suptitle('Count of Orders Per Country Per Status')

        for i, status in enumerate(CONFIG['RELEVANT_STATUSES']):
            status_data = filtered_data[filtered_data['PMI ORDER STATUS'] == status]
            if status_data.empty:
                logger.info(f"No data for status '{status}'. Skipping this subplot.")
                continue

            status_data.sort_values(by='ORDER CODE', ascending=False, inplace=True)
            status_total = status_data['ORDER CODE'].sum()
            percentage_of_total = (status_total / total_orders) * 100

            row, col = divmod(i, 2)
            ax = sns.barplot(x='COUNTRY', y='ORDER CODE', data=status_data, ax=axes[row, col], palette='viridis')
            ax.set_title(f'Status: {status} ({percentage_of_total:.1f}% of total)')
            ax.set_xlabel('Country')
            ax.set_ylabel('Order Count')
            ax.tick_params(axis='x', rotation=45)
            ax.grid(axis='y', linestyle='--', alpha=0.7)

            for bar in ax.patches:
                ax.annotate(f'{int(bar.get_height())}', (bar.get_x() + bar.get_width() / 2., bar.get_height()),
                            ha='center', va='bottom')

            ax.text(0.95, 0.95, f'Total: {status_total:,}', transform=ax.transAxes,
                    horizontalalignment='right', verticalalignment='top', fontsize=10, color='black', weight='bold')

        plt.savefig(os.path.join(output_path, 'Order_Status_Count.png'))
        plt.close()
        logger.info(f"Graph 1 saved successfully at: {output_path}")

    except Exception as e:
        logger.error(f"An error occurred during visualization of Graph 1: {e}")

@log_exception
def visualize_graph_2_over_under(current_df, output_path, include_countries=None, exclude_countries=[]):
    """
    Visualize the distribution of orders being under or over 30 days old from the 'UPDATE DATE' for each country and status.

    Args:
        current_df (pd.DataFrame): DataFrame containing the current data.
        output_path (str): Path where the visualization will be saved.
        include_countries (list, optional): Countries to include in the visualization.
        exclude_countries (list, optional): Countries to exclude from the visualization.
    """
    try:
        # Filter the DataFrame based on the included and excluded countries
        if include_countries:
            current_df = current_df[current_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            current_df = current_df[~current_df['COUNTRY'].isin(exclude_countries)]

        # Calculate the age of the orders and categorize them
        current_date = pd.Timestamp('now').normalize()
        current_df['ORDER AGE'] = (current_date - current_df['UPDATE DATE']).dt.days
        current_df['AGE CATEGORY'] = current_df['ORDER AGE'].apply(lambda x: f'Under {CONFIG["AGE_THRESHOLD"]} days' if x < CONFIG["AGE_THRESHOLD"] else f'Over {CONFIG["AGE_THRESHOLD"]} days')
        
        # Group the data by country, order status, and age category
        grouped_data = current_df.groupby(['COUNTRY', 'PMI ORDER STATUS', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
        filtered_data = grouped_data[grouped_data['PMI ORDER STATUS'].isin(CONFIG['RELEVANT_STATUSES'])]

        # Sort the data for consistent bar order
        filtered_data.sort_values(by=['PMI ORDER STATUS', 'ORDER CODE'], ascending=[True, False], inplace=True)

        # Check if there is data to visualize
        if filtered_data.empty:
            logger.info("No data available for visualization.")
            return

        # Plotting setup
        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5))
        fig.suptitle('Order Age Distribution by Country and Status')

        # Define custom colors for the age categories
        custom_palette = {f'Under {CONFIG["AGE_THRESHOLD"]} days': 'DodgerBlue', f'Over {CONFIG["AGE_THRESHOLD"]} days': 'LightSalmon'}

        # Create a bar plot for each order status
        for i, status in enumerate(CONFIG['RELEVANT_STATUSES']):
            ax = axes[i//2, i%2]
            status_data = filtered_data[filtered_data['PMI ORDER STATUS'] == status]

            # Skip plotting if no data is available for the status
            if status_data.empty:
                logger.info(f"No data available for status '{status}'.")
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
        plt.close()
        logger.info(f"Graph 2 saved successfully at: {output_path}")

    except Exception as e:
        logger.error(f"An error occurred during visualization of Graph 2: {e}")

@log_exception
def visualize_graph_4_specific(current_df_no_lsp, output_path, country):
    """
    Visualize the age distribution of shipped orders for a specific country.

    Args:
        current_df_no_lsp (pd.DataFrame): DataFrame containing order data without LSP conditions applied.
        output_path (str): Path to save the output graph image.
        country (str): The specific country to visualize.
    """
    try:
        shipped_orders_df = current_df_no_lsp[current_df_no_lsp['PMI ORDER STATUS'] == 'SHIPPED']

        current_date = datetime.now()
        shipped_orders_df['ORDER AGE'] = (current_date.date() - shipped_orders_df['UPDATE DATE'].dt.date).apply(lambda x: x.days)
        shipped_orders_df['AGE CATEGORY'] = pd.cut(shipped_orders_df['ORDER AGE'], 
                                                   bins=[-np.inf, 7, 15, 30, np.inf], 
                                                   labels=['Up to 7D', '7-15D', '15-30D', 'Over 30D'])
        
        filtered_data_g4 = shipped_orders_df.groupby(['COUNTRY', 'AGE CATEGORY'])['ORDER CODE'].nunique().reset_index()
        logger.info(f"DataFrame shape for Graph 4: {filtered_data_g4.shape}")

        country_data = filtered_data_g4[filtered_data_g4['COUNTRY'] == country]

        plt.figure(figsize=(10, 6))
        ax = sns.barplot(x='AGE CATEGORY', y='ORDER CODE', data=country_data, palette='coolwarm')
        plt.title(f'Order Age Distribution in {country}')
        plt.xlabel('Age Category')
        plt.ylabel('Unique Order Count')
        plt.xticks(rotation=45)
        plt.grid(axis='y', linestyle='--', alpha=0.7)

        for p in ax.patches:
            ax.annotate(f"{int(p.get_height())}", (p.get_x() + p.get_width() / 2., p.get_height()), ha='center', va='bottom')

        plt.tight_layout()
        plt.savefig(os.path.join(output_path, f'{country}_Age_Distribution.png'))
        plt.close()
        logger.info(f"Graph 4 for {country} saved successfully at: {output_path}")

    except Exception as e:
        logger.error(f"An error occurred during visualization of Graph 4: {e}")

@log_exception
def visualize_graph_5_trend_updated(combined_df, output_path, specific_dates, include_countries=None, exclude_countries=[]):
    """
    Generate and save a trend visualization of order counts over time, filtered by country and order status.

    Args:
        combined_df (pd.DataFrame): The DataFrame containing the combined data for trend analysis.
        output_path (str): The directory path where the output graph image will be saved.
        specific_dates (list): The list of dates to be included in the trend analysis.
        include_countries (list, optional): The list of countries to include in the visualization.
        exclude_countries (list, optional): The list of countries to exclude from the visualization.
    """
    try:
        if include_countries:
            combined_df = combined_df[combined_df['COUNTRY'].isin(include_countries)]
        if exclude_countries:
            combined_df = combined_df[~combined_df['COUNTRY'].isin(exclude_countries)]

        if combined_df.empty:
            logger.info("No data available after applying country filters.")
            return

        grouped_data = combined_df.groupby(["COUNTRY", "PMI ORDER STATUS", "RUN TIME"])["ORDER CODE"].count().reset_index()

        if grouped_data.empty:
            logger.info("No data available for plotting after grouping.")
            return

        fig, axes = plt.subplots(2, 2, figsize=(14, 8.5))
        plt.subplots_adjust(right=0.85)
        fig.suptitle('Order Trend by Status')

        for i, status in enumerate(CONFIG['RELEVANT_STATUSES']):
            ax = axes[i//2, i%2]
            status_data = grouped_data[grouped_data["PMI ORDER STATUS"] == status]

            if status_data.empty:
                logger.info(f"No data for status '{status}'. Skipping this subplot.")
                continue

            for country in status_data["COUNTRY"].unique():
                country_data = status_data[status_data["COUNTRY"] == country]
                ax.plot(country_data["RUN TIME"], country_data["ORDER CODE"], marker="o", label=country)

            ax.set_ylim(bottom=0)
            ax.xaxis.set_major_locator(ticker.MaxNLocator(nbins='auto', integer=True))
            ax.yaxis.set_major_locator(ticker.MaxNLocator(integer=True))
            ax.set_title(f'Orders in {status} Status')
            ax.set_xlabel('Date')
            ax.set_ylabel('Order Count')
            ax.tick_params(axis='x', rotation=45)
            ax.legend(title="Country", loc='upper left', bbox_to_anchor=(1,1))
            ax.grid(axis='y', linestyle='--', alpha=0.7)

        plt.tight_layout(rect=[0, 0, 1, 0.9])
        plt.savefig(os.path.join(output_path, 'Order_Trend_Analysis.png'))
        plt.close()
        logger.info(f"Graph 5 saved successfully at: {output_path}")

    except Exception as e:
        logger.error(f"An error occurred during visualization of Graph 5: {e}")

@log_exception
def main():
    """Main function to orchestrate the order analysis process."""
    logger.info("Starting order analysis script")

    try:
        current_df, combined_df, output_path = setup_paths(CONFIG['CURRENT_DATE'], CONFIG['SPECIFIC_DATES'])
        if current_df is None or combined_df is None or output_path is None:
            logger.error("Failed to set up paths and load data. Exiting.")
            return

        current_df, combined_df = remove_stores(current_df, combined_df)

        current_df_no_lsp = current_df.copy()
        combined_df_no_lsp = combined_df.copy()

        current_df, combined_df = apply_LSP_conditions(current_df, combined_df, CONFIG['CONDITIONS_CONFIG'])

        visualize_graph_1_current(current_df, output_path)
        visualize_graph_2_over_under(current_df, output_path)
        visualize_graph_4_specific(current_df_no_lsp, output_path, country='CZ')
        visualize_graph_5_trend_updated(combined_df, output_path, CONFIG['SPECIFIC_DATES'])

        logger.info("Order analysis script completed successfully")
    except Exception as e:
        logger.error(f"An error occurred during script execution: {e}")

if __name__ == "__main__":
    main()
