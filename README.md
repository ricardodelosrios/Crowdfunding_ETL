# Crowdfunding_ETL 🚀

## Introduction 📖

Welcome to the Crowdfunding ETL project! This exciting project focuses on creating an ETL pipeline using Python, Pandas, and Python dictionary methods or regular expressions to extract and transform data. After this process, four CSV files are generated, and the information is used to create an Entity-Relationship Diagram (ERD) and a table schema. Finally, the data is loaded into a PostgreSQL database.

## Folders and Files 📂

* **Resources Folder:**
  * Here, you'll find 4 CSV files and 2 xlsx files, which are the original databases used in the project. Some of them include:
    * [crowdfunding.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    * [contacts.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    * [category.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    * [subcategory.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    * [contacts.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
    * [campaign.csv](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources)
  
* **Files:**
  * [ETL_Mini_Project_RDelosrios_SSuresh.ipynb](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/ETL_Mini_Project_RDelosrios_SSuresh.ipynb): Contains the code to extract and transform data, as well as to create DataFrames for Categories, Subcategories, Campaigns, and Contacts.
  * [crowdfunding_db_schema.sql](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/crowdfunding_db_schema.sql): Schema for the PostgreSQL database.
  * [ETL_Crowdfunding_ERD.png](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/ETL_Crowdfunding_ERD.png): Entity-Relationship Diagram generated with Quick Database Diagram.
  * [.gitignore](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/.gitignore): File storing the password for PGAdmin access and establishing the connection with the SQLAlchemy library.

## Extract, Transform, Load (ETL) ⚙️

To start the ETL process, it's essential to mention that it begins with two Excel files: [crowdfunding.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources) and [contacts.xlsx](https://github.com/Subiksha-SS/Crowdfunding_ETL/tree/main/Resources). These files are the foundation of the entire project.

The first step is to import the necessary libraries, such as `pandas` for data manipulation and `numpy` for numerical and mathematical operations. Then, the Excel file is read, and the data structure is displayed using the `info()` function.

```python
import pandas as pd
import numpy as np
pd.set_option('max_colwidth', 400)

crowdfunding_info_df = pd.read_excel('crowdfunding.xlsx')
crowdfunding_info_df.info()
```
### Create Category and Subcategory DataFrames 🗂️

To create the Category and Subcategory DataFrames, an analysis of the column containing this information is performed. The str.split() function is used to split the column into two based on the '/' separator.

```python
crowdfunding_info_df[['category', 'subcategory']] = crowdfunding_info_df['category & sub-category'].str.split('/', n=1, expand=True)
```

Lists of unique categories and subcategories are obtained, along with the number of distinct values in each list.

```python
categories = list(crowdfunding_info_df['category'].unique())
subcategories = list(crowdfunding_info_df['subcategory'].unique())

print(len(categories))
print(len(subcategories))
```
Next, identifiers for categories and subcategories are created, and DataFrames are generated for each.

```python
category_ids = np.arange(1, 10)
subcategory_ids = np.arange(1, 25)

cat_ids = ['cat' + str(x) for x in category_ids]
scat_ids = ['subcat' + str(x) for x in subcategory_ids]

category_df = pd.DataFrame({
    'category_id': cat_ids,
    'category': categories
})

subcategory_df = pd.DataFrame({
    'subcategory_id': scat_ids,
    'subcategory': subcategories
})

# Export DataFrames to the Resources folder
category_df.to_csv('Resources/category.csv', index=False)
subcategory_df.to_csv('Resources/subcategory.csv', index=False)
```

### Campaign DataFrame🚀

A copy of the main DataFrame is created, and columns are renamed as per the instructions.

```python
campaign_df = crowdfunding_info_df.copy()
campaign_df.rename(columns={'blurb': 'description', 'launched_at': 'launched_date', 'deadline': 'end_date'}, inplace=True)

```
Then, the date format is adjusted, and it's merged with the Category and Subcategory DataFrames.

```python
campaign_df["launched_date"] = pd.to_datetime(campaign_df["launched_date"], unit="s").dt.strftime("%Y-%m-%d")
campaign_df["end_date"] = pd.to_datetime(campaign_df["end_date"], unit="s").dt.strftime("%Y-%m-%d")

campaign_merged_df = pd.merge(campaign_df, category_df, on="category", how="left")
campaign_merged_df = pd.merge(campaign_merged_df, subcategory_df, on="subcategory", how="left")

```
Unnecessary columns are dropped, and the cleaned file is exported.

```python
campaign_merged_df.drop(['category', 'subcategory'], axis=1, inplace=True)
campaign_merged_df.to_csv('Resources/campaign.csv', index=False)

```

### Contacts DataFrame 📞
Pandas is used to create the Contacts DataFrame.

```python
# Loop to convert each row to a dictionary
dict_values = []

# Create DataFrame with dictionary values
contact_info_df = pd.DataFrame(dict_values, columns=column_names)

# Split the 'name' column into 'first_name' and 'last_name'
contact_info_df[['first_name', 'last_name']] = contact_info_df.name.str.split(" ", expand=True)

# Remove the 'name' column and rearrange the other columns
contact_info_df = contact_info_df[['contact_id', ... (add other columns here)]]
```
Columns are split and reordered, and the resulting DataFrame is exported.

```python
contact_info_df[['first_name', 'last_name']] = contact_info_df.name.str.split(" ", expand=True)
contact_info_df.drop(['name'], axis=1, inplace=True)
contact_info_df = contact_info_df[['contact_id', 'first_name', 'last_name', 'email', ... (add other columns here)]]

# Export the DataFrame to the Resources folder
contact_info_df.to_csv('Resources/contacts.csv', index=False)
```

## Create the Crowdfunding Database 🏦

### Data modeling 📊

For data modeling, the Quick Database Diagrams website was used, providing a visual representation of the database structure

This diagram was built using [Quick Database Diagram](https://www.quickdatabasediagrams.com/).

![alt text](https://github.com/Subiksha-SS/Crowdfunding_ETL/blob/main/ETL_Crowdfunding_ERD.png)

### Data engineering ⚙️

The information provided by the Entity Relationship Diagram was used to create a table schema for each of the four CSV files, and then, the data from each of the CSV files was imported into PostgreSQL.

The data from the tables and their relationships are in the [Schema]().

## Developers

| [<img src="https://avatars.githubusercontent.com/u/133066908?v=4" width=115><br><sub>Ricardo De Los Rios</sub>](https://github.com/ricardodelosrios) | [<img src="https://avatars.githubusercontent.com/u/118707567?v=4" width=115><br><sub>Subiksha Suresh</sub>](https://github.com/Subiksha-SS) |
| :---: | :---: |


