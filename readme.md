# Determining Whether Press Release Language Has Changed Since the Start of the Pandemic

## Table of Contents
* [Executive Summary](#Executive-Summary)
* [Problem Statement](#Problem-Statement)
* [Data Collection & Cleaning](#Data-Collection-&-Cleaning)
* [Modeling](#Modeling)
* [Conclusion](#Conclusion)
* [Repo Contents]()

## Executive Summary
Press releases play an importance role in how companies communicate with key stakeholders, such as investors, customers and media.

Since the start of the pandemic, companies have had to continue to find ways to communicate their actions with stakeholders during a time when many were unable to connect with them in person.

This project seeks to determine if press release language before and after the start of the pandemic has changed. In order to determine whether or not there is a difference in how companies speak in press releases since the start of the pandemic, this project leverages Natural Language Processing (NLP) and several binary classification models to predict which time period a press release was published. 

For the purposes of this project, all press releases published before January 2020 will be categorized as `before`, while all press releases published starting January 1, 2020 until present day will .be categorized as `after`. I chose January 2020 as the start date of the pandemic because, although lockdowns in the U.S. did not start until March, companies were beginning to have supply chain issues as lockdowns in China caused significant manufacturing delays. 

My baseline R2 score is 0.618228. Across all of the models I tested, they all performed better than the baseline, with the Logistic Regression models performing the best. 
* Logistic Regression with an l2 penalty scored  0.874316 on the training data and 0.783439 on the test data. 
* While Logistic Regression with no penalty (i.e., none) scored higher on the test data with a score of 0.853503, it was significantly more overfit to the training data, with an R2 score of 1.0.

## Problem Statement
Use NLP and binary classification models to determine whether or not the top five Fortune 100 companies have been changed how they communicate through press releases.

## Data Collection & Cleaning

The data collection and cleaning portion of this project occurred in six distinct phases:

### 1. Gather information from Fortune's website

First, I had to gather the most recent Fortune 100 list from Fortune's website, which included the company's name, the company's rank and the links to each company's profile page. 

Once I had the information from the Fortune 100 list, I iterated through each link and found the company's website, as listed on the company's profile page on Fortune. 

In a few instances, the link provided by Fortune was incorrect, so I manually reassigned those links.

### 2. Find the newsroom links for each company from their company page

Once I had a link for each company's main/corporate website, I scraped all of the links from the HTML on the company's homepage and used the `fuzzywuzzy` library to determine how close each link was to `news`, `press` and `corporate` to account for differences in how each company might refer to their newsroom pages. 

From there, I was able to assess the links and create a dictionary that contained each company's newsroom link, which was then assigned to each company in the `final` column of the dataframe.

### 3. Gather HTML from newsrooms

Once I had the newsroom url for each company, I limited the data to just the top five Fortune 100 companies, and added the `loop_url`, `type` and `page_type` columns. The categories I assigned to each of these columns were then used to iterate through multiple pages of the companies' newsrooms and collect all of the links available on each page. The HTML data was then saved into the [html](./html) folder.

### 4. Parse HTML from files

Following the HTML collection, I created a loop that went through all of the files in the [html](./html) folder, as well as all of the tags in each row of the file, to gather all of the links from each page. 

I followed this up by filtering out unnecessary links (i.e., those that don't lead to press releases)using `.str.contains()` with regular expressions, and determined whether or not the link needed a base (i.e., whether or not it had `https://....com` in front of it), and added that in to the dicitonary that contained the regular expressions, as appropriate.

### 5. Gather press release text

Now that I had all of the links to the press releases, I started to collect the full text, titles and HTML from each press release hosted on the company's website and saved each company's full text in the [press_releases](./press_releases) folder.

### 6. Data cleaning

After I had collected all of the data from the press releases, I had to go through and find the dates for when each press release was published so I could assign the label for modeling. I also cleaned the data to remove mentions of the years, as well as other key words so as to not leak the target into the model. The full list of potential leak words that I removed from the data include: '2021', '2020', '2019', 'Covid-19', 'Covid', 'COVID-19', 'COVID',   'Coronavirus', 'coronavirus' and 'pandemic'.

## Modeling

By using binary classification models, I can determine whether or not press release language has changed by achieving an R2 score higher than the baseline for the data collected. The baseline for this project is 0.

I tested two different types of models, Logistic Regression and KNearest Neighbors, and tested multiple different hyperparameters for each model. 

For my Logistic Regression models, I tested the models with the penalty set to both 'l2' and 'none'. For the KNearest Neighbors models, I tested the models with n_neighbors set to 3, 5 and 8. The training and test scores for each model are below.

|Model|Hyperparameter|Train score|Test score|
|-----|-----|-----|-----|
|Logistic Regression|penalty: 'l2'|0.8743169398907104|0.7834394904458599|
|Logistic Regression|penalty: 'none'|1.0|0.8535031847133758|
|KNearest Neighbors|n_neighbors: 8|0.7996357012750456|0.7643312101910829|
|KNearest Neighbors|n_neighbors: 5|0.8187613843351548|0.7473460721868365|
|KNearest Neighbors|n_neighbors: 3|0.8588342440801457|0.7303609341825902|

## Conclusion

Following a robust data collection and modeling process, and due to all of the models beating the baseline score, I have concluded that companies have changed the way they communicate through press releases in significant enough ways that machine learning can detect the difference.

## Repo Contents

|Name|Type|Description|Output|
|-----|-----|-----|-----|
|[html](./html)|Directory||None|
|[links](./links)|Directory||None|
|[press_releases](./press_releases)|Directory||None|
|[01-gather-data-from-fortune-website.ipynb](./01-gather-data-from-fortune-website.ipynb)|Jupyter Notebook|Contains code for gathering information from Fortune's website.|[fortune_100_data.csv](./fortune_100_data.csv)|
|[02-finding-newsroom-urls.ipynb](./02-finding-newsroom-urls.ipynb)|Jupyter Notebook|Contains code for gathering HTML from company websites to identify the matches for company newsrooms.|[fortune_100_data_w_links.csv](./fortune_100_data_w_links.csv)|
|[03-gathering-html-from-newsrooms.ipynb](./03-gathering-html-from-newsrooms.ipynb)|Jupyter Notebook|Contains code to loop through newsroom pages and collect all of the HTML from each page.|Updated [fortune_100_data_w_links.csv](./fortune_100_data_w_links.csv) file. Files in the [html](./html) folder.|
|[04-parsing-html-from-files.ipynb](./04-parsing-html-from-files.ipynb)|Jupyter Notebook|Contains code to parse all the HTML collected by the code in the previous notebook in order to obtain all the links from the newsrooms.||
|[05-gathering-press-release-text.ipynb](./05-gathering-press-release-text.ipynb)|Jupyter Notebook|||
|[06-data-cleaning.ipynb](./06-data-cleaning.ipynb)|Jupyter Notebook|||
|[07-NLP-modeling.ipynb](./07-NLP-modeling.ipynb)|Jupyter Notebook|||
|[fortune_100_data.csv](./fortune_100_data.csv)|.csv file|Output from [01-gather-data-from-fortune-website.ipynb](./01-gather-data-from-fortune-website.ipynb). Contains information gathered from Fortune's website.|None|
|[fortune_100_data_w_links.csv](./fortune_100_data_w_links.csv)|.csv file|Output from the culmination of [02-finding-newsroom-urls.ipynb](./02-finding-newsroom-urls.ipynb) and [03-gathering-html-from-newsrooms.ipynb](./03-gathering-html-from-newsrooms.ipynb). Contains data from Fortune website, along with additional information from each company's website.|None|