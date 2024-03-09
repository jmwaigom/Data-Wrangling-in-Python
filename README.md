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
As a result, there was one csv file for each month. These four files were then concatenated to create one massive parent file. 
