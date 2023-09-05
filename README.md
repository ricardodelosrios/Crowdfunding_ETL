# Crowdfunding_ETL

## Introduction

The ETL project will be building an ETL pipeline using Python, Pandas, and Python dictionary methods or regular expressions to extract and transform the data. After transforming the data, it will create four CSV files and use the data from the CSV file to create an ERD and a table schema. Finally, it will load the data into a Postgres database.

## Folders and files

* There is a **Resources folder** in this project:

   * This folder contains 4 CSV files that were the exportable files from the exercise, and 2 xlsx files that are the original databases used for the project. Listed below are the names of the files:

      * [crowdfunding.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
      * [contacts.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
      * [category.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
      * [subcategory.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
      * [contacts.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
      * [campaign.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    
* There are **3 Files** in this project:
  
    * [ETL_Mini_Project_RDelosrios_SSuresh.ipynb](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/ETL_Mini_Project_RDelosrios_SSuresh.ipynb). In this file you will find the code to extract, transform         the data and create the Category, Subcategory, Campaign, Contacts DataFrames.
    * [crowdfunding_db_schema.sql](). This the schema that was built in PostgreSQL .
    * [ETL_Crowdfunding_ERD.png](). This diagrams was created using Quick Database Diagram.
    * [.gitignore](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/.gitignore):In this file, it saved the password to access to PGAdmin and be able to create the connection with the sqlalchemy library.

## Extract, Transform, Load (ETL) 

To start the ETL process, it is essential to say that it starts with two Excel files: [crowdfunding.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources) and [contacts.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources). These two files are the basis for developing the entire project.

The first step is to import the libraries. To run this script, it will need `pandas` that is a data manipulation and analysis library in Python, and by importing it as `pd`, it can use pd as a shorthand to access Pandas functions and classes in the code. In addition, it will be used `NumPy` library that is another library in Python for numerical and mathematical operations. It provides support for arrays and matrices, which are often used in data analysis and scientific computing.
Finally, a Pandas option is set using the `set_option` function. Specifically, it sets the maximum column width for displaying Pandas DataFrames to 400 characters. This option is useful when you have text or string data in the columns of the DataFrame and you want to ensure that the text is not truncated when you display the DataFrame.

```
import pandas as pd
import numpy as np
pd.set_option('max_colwidth', 400)
```
Once the libraries are imported, the `crowdfunding` Excel file is read into a Pandas DataFrame and given the name `crowdfunding_info_df`.

Later, to know the data structure in the DataFrame, the `info()` function is used and thus help in the process of exploration and analysis of the information. With this function we can get the number of rows and columns, column names, column data type, non-null values.

```
crowdfunding_info_df.info()
```
### Create the Category and Subcategory DataFrames

To create the category and subcategory DataFrames, the content of the columns must be parsed. For this, it was identified that these two variables are found in a single column called **"category and subcategory"**. For that reason, it had to be split using **'/'** as a separator. In addition, it was indicated that it should only do an **'n=1'** division. And **'expand=True'**, indicated that the result of the division should be divided into two columns.

```
crowdfunding_info_df[['category', 'subcategory']] = crowdfunding_info_df['category & sub-category'].str.split('/', n=1, expand=True)
```
After that, it is going to get the unique categories and subcategories in separate lists, and the number of distinct values lists.

```
categories= list(crowdfunding_info_df['category'].unique())
subcategories= list(crowdfunding_info_df['subcategory'].unique())
```

```
print(len(categories))
print(len(subcategories))
```

Then, it creates a NumPy array resulting in an array containing the numbers 1 through 9. Also, it creates another NumPy array resulting in an array containing the numbers 1 through 24.

```
category_ids = np.arange(1, 10)
subcategory_ids = np.arange(1, 25)
```
Then, use a list comprehension to create a new list called **cat_ids** by appending the string **"cat"** to each item in the list. The same is done with the list called **scat_ids** by adding **"subcat"** to each element of the list.

```
cat_ids = ['cat' + str(x) for x in category_ids] 
```
```
scat_ids = ['subcat' + str(x) for x in subcategory_ids]
```
After, a category DataFrame is created with the category_id array as the category_id and the list of categories as the category name.

```
category_df=pd.DataFrame({
       'category_id':cat_ids,
       'category':categories
})
```
The same is done for a subcategory DataFrame with the subcategory_id array as the subcategory_id and the subcategory list as the subcategory name.
```
subcategory_df=pd.DataFrame({
       'subcategory_id':scat_ids,
       'subcategory': subcategories
})
```
Finally, those DataFrames were exported to the Resources folder. [category.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources), [subcategory.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)

### Campaign DataFrame

The first step is to create a copy of the crowdfunding_info_df data frame and call it campaign_df. Then, you renamed 3 columns as per the instructions. As seen below:
```
campaign_df.rename(columns = {'blurb':'description', 'launched_at':'launched_date', 'deadline': 'end_date'}, inplace=True)
```
Then, the date format of the columns "launch_date" and "end_date" was changed. Using the **'pd.to_datetime(...)'** function, that function converts Unix timestamps to pandas datetime objects. The **unit="s"** parameter specifies that the timestamps are in seconds.
```
campaign_df["launched_date"] = pd.to_datetime(campaign_df["launched_date"], unit="s").dt.strftime("%Y-%m-%d")
campaign_df["end_date"] = pd.to_datetime(campaign_df["end_date"], unit="s").dt.strftime("%Y-%m-%d")
```
After that, it merged campaign_df with category_df in the "category" column, and then merged the result with subcategory_df in the "subcategory" column. For this, the **'pd.merge'** function was used, which is used to join two DataFrames.

```
campaign_merged_df = pd.merge(campaign_df, category_df, on= "category", how="left")
campaign_merged_df = pd.merge(campaign_merged_df, subcategory_df, on= "subcategory", how="left")
```
Columns that are not needed are then dropped. The clean file was exported to the resources folder as [campaign.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources).

### Create the Contacts DataFrame
It used Pandas to create the contacts DataFrame.
Loop through and convert each row of **contact_info_df ** to a dictionary and then join them to the list **'dict_values = []'**

Then, it created a contact_info DataFrame and added each list of values to the 'contact_id', 'name', and 'email' columns. To do that, it used a list of dictionaries called **'dict_values'** that contain the data and the column names for the DataFrame using the columns parameter.
```
contact_info_df = pd.DataFrame(dict_values, columns = column_names)
```
It was identified that the **First Name** and **Last Name** are found in a single column called **"name"**. For that reason, it had to be split using **'" "'** as a separator.  And **'expand=True'**, indicated that the result of the division should be divided into two columns.

```
contact_info_df[['first_name', 'last_name']] = contact_info_df.name.str.split(" ", expand = True)
```
Finally, the column called 'name' was removed and the other columns were reordered to export as [contacts.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)

## Create the Crowdfunding Database

### Data modeling

To carry out the modeling, it was used the webside Quick Database Diagrams, which serves as a visual representation of the structure of a database, helping to illustrate the relationships between different tables, entities and attributes within the database. 

This diagram was built using [Quick Database Diagram](https://www.quickdatabasediagrams.com/).

![alt text]()

### Data engineering

The information provided by the Entity Relationship Diagram was used to create a table schema for each of the four CSV files, and then, the data from each of the CSV files was imported into PostgreSQL.

The data from the tables and their relationships are in the [Schema]().

## Developers

| [<img src="https://avatars.githubusercontent.com/u/133066908?v=4" width=115><br><sub>Ricardo De Los Rios</sub>](https://github.com/ricardodelosrios) | [<img src="https://avatars.githubusercontent.com/u/118707567?v=4" width=115><br><sub>Subiksha Suresh</sub>](https://github.com/Subiksha-SS) |
| :---: | :---: |


