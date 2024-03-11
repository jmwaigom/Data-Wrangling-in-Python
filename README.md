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
engaged. Quick observations show that there are some null values in some columns. Some columns like'id' and date have incorrect datatypes

![columns3](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/a7718bc0-7c37-460b-a848-54688e977e96)

Null values will be explored first. To return only columns that had null values, the following code was executed. It appears that bathrooms and calendar_updated columns are completely empty.
For that reason, they were dropped.

```
# This code returns all columns whether they have null values or not
all_columns = subset_df.isnull().sum()

# Filtering to display columns with null values only
null_value_columns = all_columns[all_columns > 0]

# Sorting columns displaying the ones with most null values at the top
null_value_columns.sort_values(ascending=False)
```
![column4](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/40745584-f833-49d5-97d1-aeb623cc57a8)

```
# Code to drop 'bathrooms' and 'calendar_updated' columns
subset_df.drop(columns=['bathrooms','calendar_updated'],inplace=True)
```
For numerical columns, NaN was replaced with median value. It made more sense to fill NaN with values that represent 50% of the data in the respective columns.
For non-numerical/string columns, Null values were replaced with the string 'unknown'.

```
# For bedrooms, replacing NaN with median value
subset_df['bedrooms'] = subset_df['bedrooms'].fillna(value=subset_df['bedrooms'].median())

# For host_neighbourhood, replacing NaN with "unknown"
subset_df['host_neighbourhood'] = subset_df['host_neighbourhood'].fillna('unknown')

# For host_location, replacing NaN with "unknown"
subset_df['host_location'] = subset_df['host_location'].fillna('unknown')

# For host_is_superhost, replacing NaN with "unknown"
subset_df['host_is_superhost'] = subset_df['host_is_superhost'].fillna('unknown')

# For review_scores_rating, replacing NaN with median value
median_value = subset_df['review_scores_rating'].median()
subset_df['review_scores_rating'] = subset_df['review_scores_rating'].fillna(median_value)

# For review_scores_accuracy, replacing NaN with median value
median_review_accuracy = subset_df['review_scores_accuracy'].median()
subset_df['review_scores_accuracy'] = subset_df['review_scores_accuracy'].fillna(median_review_accuracy)

# For reviews_per_month. replacing NaN with median value
med_rev_per_month = subset_df['reviews_per_month'].median()
subset_df['reviews_per_month'] = subset_df['reviews_per_month'].fillna(med_rev_per_month)

# For host_response_time, replacing NaN with "unknown"
subset_df['host_response_time'] = subset_df['host_response_time'].fillna('unknown')

# For host_response_rate, replacing NaN with median value
''' First removing %, changing dtype to float then replacing NaN with median value '''
subset_df['host_response_rate'] = subset_df['host_response_rate'].str.replace('%','').astype(float)
med_resp_rate = subset_df['host_response_rate'].median()
subset_df['host_response_rate'] = subset_df['host_response_rate'].fillna(med_resp_rate)

# For host_acceptance_rate, replacing NaN with median value: Same procedure as host_response_rate
subset_df['host_acceptance_rate'] = subset_df['host_acceptance_rate'].str.replace('%','').astype(float)
med_acc_rate = subset_df['host_acceptance_rate'].median()
subset_df['host_acceptance_rate'] = subset_df['host_acceptance_rate'].fillna(med_acc_rate)

# For description, replacing NaN with "unknown"
subset_df['description'] = subset_df['description'].fillna('unknown')

# For beds, replacing NaN with median value
med_beds = subset_df['beds'].median()
subset_df['beds'] = subset_df['beds'].fillna(med_beds)

```
To confirm that all null values were handled correctly, the following code was executed
```
subset_df.isnull().sum()
```
As it can be seen below, there are no more null values displayed in the dataset

![column5](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/281b8c49-592b-46a5-a825-9b28f1fbbef8)

The next task was to explore datatypes to ensure data in each column was of correct datatype
```
# This code displays datatype for each column
subset_df.dtypes
```
![column6](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/c4775466-0a0f-43d2-9345-3d148238adfe)

```
The following columns need to be changed datatypes
float to int: beds, bedrooms, number_of_reviews, host_listings_count,
               host_total_listings_count, reviews_per_month, minimum and maximum nights
 float to object: id, host_id
 object to float: price_x, price_y, adjusted_price
 object to date: date, host_since

# Converting float to int
subset_df = subset_df.astype({'beds': 'int','bedrooms':'int','number_of_reviews':'int',
                             'host_listings_count':'int','host_total_listings_count':'int',
                             'reviews_per_month':'int','minimum_nights_x':'int',
                             'maximum_nights_x':'int'})

# Converting float to object
subset_df = subset_df.astype({'id':'object','host_id':'object'})

# Converting object to float
# This code removes the dollar sign and any commas first then converting it to float
price_columns = ['price_x','price_y','adjusted_price']
subset_df[price_columns] = subset_df[price_columns].replace('[\$,]', '', regex=True).astype(float)

# Converting object to date
date_columns = ['date','host_since']
subset_df[date_columns] = subset_df[date_columns].apply(pd.to_datetime, format='%m/%d/%Y')

# Checking to make sure the datatypes have converted correctly
subset_df.dtypes

```
![column7](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/b8e28682-f5fe-4cd9-84d9-2e29afd1e634)

Thereafter, 'id' and 'name' columns were changed names to 'listing_id' and 'listing_name'. This would make them easily distinguishable.
The following code was written to achieve that
```
# Renaming 'id' column to 'listing_id' and 'name' to 'listing_name'
subset_df.rename(columns={'id':'listing_id','name':'listing_name'}, inplace=True)

#Checking to see whether the renaming was successful
subset_df[['listing_id','listing_name']].head()

```
![column8](https://github.com/jmwaigom/Data-Wrangling-in-Python/assets/155841258/a572a3ab-f28a-46b8-8d26-18fa5e440395)

### Limitations
There was a substantial number of null values in a few columns. For numerical columns, these nulls were replaced with median values and for
string/object columns, they were replaced by 'unknown'. These values are not actual data points, hence they can potentially lead to a relatively
skewed or incorrect analysis. Caution and disclaimer should be used when working with columns that had null values














