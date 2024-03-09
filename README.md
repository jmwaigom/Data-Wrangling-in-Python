# Albany Airbnb Data
### Table of Contents
### Project Overview

Jamie, a Senior Data Analyst at Airbnb wants to conduct a performance analysis of properties in the months of July, August, September and October of 2023. 
This would require him to gather and clean a large and messy data that's located in different locations. At the moment, he is very busy working on 
another project, so he has assigned you the task to clean the data. He has provided with all the folders where the data is located. To make his job 
easier, he wants to see all the data he needs for analysis condensed in one file, transformed and ready to be used. He also wants to know if there any 
limitations that he would need to be aware of. This project was assigned to him recently, so he didn't have a chance to look at the data and figure out
what information he would want to use. At this point in time, all he knows is that he needs to analyze inventory, availability, pricing and performance
trends over the four months. He doesn't need to analyze reviews at the moment. Also, he only needs to analyze same properties over the four months. 
This means, if there are any new properties at any point within the four months, they should be ignored. This is the only guiding information he can
provide for now/ He's relying on your best judgment to anticipate the kind of data he would require to accomplish his task. 

### Data Source
The data for this project is housed in a parent folder called Albany. This folder contains four other folders, one for each month (July, August, September and October). The names of the folders are as follows: "July_3rd_2023", "Aug_9th_2023", "Sept_2nd_2023" and "Oct_1st_2023". Each folder contains several csv files for each month. For each month, the csv provided are: "listings", "calendar","reviews" "reviews summarized" and "listings summarized". The files of interest for this project are listings and calendar. Since Jamie wants to see all the data he requires in one file, the best approach is to merge and concatenate all these files into one file with just the relevant data. 

### Approach
For each month,the listings and calendar files were merged together using 'id' column on listings file and 'listing_id' column on calendar file. 
As a result, there was one csv file for each month. These four files were then concatenated to create one massive parent file. The following code
was written to create one all-inclusive, duplicate free file called 'parent_data.csv'

```
# Importing requred libraries
import os
import pandas as pd

# The following code will merge listings and calendar files 
def merge_files(folder_path):
    listings_file = None
    calendar_file = None
    
    # Locating listings and calendar files
    for file in os.listdir(folder_path):
        if file.startswith('listings') and file.endswith('.csv'):
            listings_file = os.path.join(folder_path, file)
        elif file.startswith('calendar') and file.endswith('.csv'):
            calendar_file = os.path.join(folder_path, file)
    
    # Checking if listings and calendar files are found
    if listings_file is None or calendar_file is None:
        print(f"Error: Listings or calendar file not found in folder {folder_path}")
        return None
    
    # Reading listings and calendar files
    listings_df = pd.read_csv(listings_file)
    calendar_df = pd.read_csv(calendar_file)
    
    # Merging based on 'id' and 'listing_id' columns
    merged_df = pd.merge(listings_df, calendar_df, left_on='id', right_on='listing_id', how='inner')
    return merged_df

# Creating a list of folders
folders = ['Aug_9th_2023', 'July_3rd_2023', 'Oct_1st_2023', 'Sept_2nd_2023']

#  Creating a list to store merged DataFrames from each folder
merged_dfs = []

# Iterating over folders
for folder in folders:
    folder_path = os.path.join('Albany/',folder)
    merged_df = merge_files(folder_path)
    if merged_df is not None:
        merged_dfs.append(merged_df)

# Concatenating DataFrames from all folders
parent_df = pd.concat(merged_dfs, ignore_index=True)

# Droping duplicate rows
parent_df.drop_duplicates(inplace=True)

# Writing final DataFrame to a CSV file
parent_df.to_csv('parent_data.csv', index=False)
```
After parent_df (dataframe which represents 'parent_data.csv' in Pandas) was generated, two copies were created as a backup in case irreversible issues were encountered
down the line during the cleaning process. The following code was executed to achieve that. Moving forward, 'parent_subfile_one' was chosen instead of 'parent_df'.
A code was written to display its columns

```
parent_subfile_one = parent_df.copy()
parent_subfile_two = parent_df.copy()

# Checking columns available in parent_subfile_one
parent_subfile_one.columns

```

![columns](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/56be8c7f-4176-44c5-9b4a-1a99091e5dbf)

As shown above, this dataframe contains 90 columns. Here is where I used my best judgment to decide which columns Jamie would need to conduct his analysis. To achieve this,
a subset was created with only relevant columns that Jamie would require. Some columns may be duplicates (e.g. price_x, price_y, minimum_nights_x, minimum_nights_y) To explore 
this further, the potentially duplicate columns will be included in the subset and analyzed later.The following code was executed to achieve that. Duplicate rows were dropped 
(just to be safe). The subset dataframe was henceforth used for further data wrangling and manipulation.
```
# Creating a subset dataframe/dataset with only relevant columns
subset_df = parent_subfile_two[['id','name','date','description','host_id','host_name','host_since',
              'host_location','host_response_time','host_response_rate',
              'host_acceptance_rate','host_is_superhost','host_neighbourhood',
              'host_listings_count','host_total_listings_count','host_verifications',
              'host_has_profile_pic','host_identity_verified','property_type',
              'room_type','accommodates','bathrooms','bedrooms','beds','amenities',
              'price_x','price_y','adjusted_price','minimum_nights_x','minimum_nights_y',
              'maximum_nights_x','maximum_nights_y', 'calendar_updated','has_availability',
              'number_of_reviews','review_scores_rating','review_scores_accuracy',
              'reviews_per_month']].copy()

# Drop duplicates
subset_df = subset_df.drop_duplicates()

# Check for duplicates
duplicates_count = subset_df.duplicated().sum()
duplicates_count

```
Now that a subset has been created, it can be explored further. The following code displays the top two rows and all columns. This gives an quick impression of the dataframe
```
pd.set_option('display.max_columns',40)
subset_df.head(2)

```
![columns2](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/1532b2ae-f8e9-4f53-acfc-7ae51141ca76)

This subset contains 395,660 rows and 38 columns. This was obtained using the code subset_df.shape. To get a general idea of its composition, the code subset_df.info() was
engaged. The result shows that there are null values in some columns. 

![columns3](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/a7718bc0-7c37-460b-a848-54688e977e96)
