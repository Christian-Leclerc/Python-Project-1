# Final-Project-Transforming-and-Analyzing-Data-with-SQL
*by Christian Leclerc b.ing, Data Analyst, [LHL Bootcamp](https://www.lighthouselabs.ca)*
<br>
***Note:***
This file is better viewed using [GitHub](https://github.com) preview screen (or any Markdown viewer) to be able to use the relative links.

## Project/Goals
This repository contains the code and documentation for my data analysis project on ***Statistical Modelling with Python***
The project aimed to demonstrate my comprehension and skills related to the content of the *Data Analyst - FLEX [LHL Bootcamp](https://www.lighthouselabs.ca)* program so far, by demonstrating if a relationship between the number of bikes in a particular location and the characteristics of the POIs (Point of interests) in that location exits.

My goal, as a student, is to identify my strengths and weaknesses during the project along with increasing my knowledge of the coding and the tools available.

## Table of Contents
- [Project Background](#project-background)
- [Process](#process)
- [Results](#results)
- [Challenges](#challenges)
- [Future Goals](#future-goals)
- [Tech stacks](#tech-stacks)

## Project Background
The location chosen was the city of Montréal, Québec, Canada and Bixi is the principal bike rental company.
There is 800 bike stations on the island and about 8000 Point of Interests with mainly the category of dining and restaurant (including bars).
Montréal is known for having a lot of well rated restaurants at affordable price (or not...) in all the range of taste and decorum.

The data had to be pulled from different imposed medium API:<br>
- [Citybikes API](http://api.citybik.es/v2/) to collect the number of stations, their locations and number of bikes.
- [FourSquare API](https://location.foursquare.com/developer/) to collect the information about the POIs.
- [Yelp API](https://www.yelp.com/developers) for comparing the quality of informations about POIs with FourSquare.

The gitHub repo contains .md files (Markdown) with instructions and questions to answer about the dataset.

## Process
### Step 1: Importing the data from several API
>Refer to [city_bikes.ipynb](/notebooks/city_bikes.ipynb) and
[yelp_foursquare_EDA.ipynb](/notebooks/yelp_foursquare_EDA.ipynb) Notebooks for a comprehensive overview of my process.

For all APIs, this process was always the same:
1) Read the API documentation
2) Setup the request parameters
3) Send the request (for Yelp, I had to do it by batch)
4) Parse the json response
5) Create a pandas dataframe with the dataset
All dataset created can be found in the [data](/data) folder.


### Step 2: Exploratory Analysis
>Please consult [yelp_foursquare_EDA.ipynb](/notebooks/yelp_foursquare_EDA.ipynb) Notebook to understand my approach and my first impressions.. 

Inside each Notebook, I conducted some pre-cleaning of the dataset and some exploratory analysis like:
- Creating histograms or boxplot using matplotlib and seaborn.
- Evaluating outliers
- Convert datatypes and format
- Validating the API data, some data were out of the parameters, not expected.


### Step 3: Feature Engineering
>You will find the steps in [joining_data.ipynb](/notebooks/joining_data.ipynb) Notebook for cleaning and preparing the dataset. 

This was probably the biggest part of the project. Each time you were trying to combine 2 API together, you would find a missing attribute. So back and forth between the parsing of the information and the cleaning part. Tiedous, but so important.<br>
I followed this process to get to a final dataset ready for the database creation:
- Create meaningful columns (combine or create new columns)
- Filling missing values (along with a way to log the change)
- Joining the dataframes (different API)
- Cleaning the dataframe (datatypes)
- Evaluate relationship
- Evaluate correlation

All these steps are necessary before loading the data into a database.

### Step 4: Creating a database
>Please refer to Database section of the [joining_data.ipynb](/notebooks/joining_data.ipynb) Notebook to see the process and coding. 

The project asked to have a database done with Python SQLite3. I decided to use what I have learned from my previous [SQL-project-1](https://github.com/Christian-Leclerc/SQL_Project-1) and created a fully relational database. With that type of database, it is easier to pull query for future questions and can be updated with new data even from other API. Here's my pseudo ERD made in Excel to understand better the relations between the tables:

![ERD](/images/ERD.png) <a name="erd"></a>
(Data is fictional)

Here's a description of the table:
- **stations**: to hold information related to the stations location in the city along with their total bikes
- **pois**: to hold the information about each POI (including poi_name, poi_id, review_count, rating, popularity, price)
- **categories**: simulates the api category of Foursquare
- **poi_category**: joining table for poi and categories (many-to-many)
- **prices**: to holds the definition of the prices number
- **poi_detail**: creates the relation between data (station, poi_id, distance to the station)
- **api**: if we have time, we could try combining the poi_id of both API. This table collect api_number, poi_id and api_name.

To better manage the connection with the database, I decided to seperate the task in 3 phases:
- Phase 1a: Ceate the tables
- Phase 1b: Insert the data
- Phase 2: Read the databse (and do some quaity checks)

Phase 1 can be done entirely with one single connection and cursor.

Phase 2, I could have use the same connection, but I decided to go with the pandas read_sql function which automatically manage opening and closing connections.


### Step 5: Conduct regression modelling
>Please consult the [model_building.ipynb](/notebooks/model_building.ipynb) Notebook for a complete description of the regression attemps. 

Having the database setup and running, I could easily query the necessary data based on my findings during the EDA. Then the long and iterative process of modeling can start.

Even thought I didnt have a lot of dependant variables, I decided to go with backward selection for the linear regression.

The process was to:
a) Run the full model
b) Remove the highest p-value variable et re-run the model
c) Repeat until satisfactory adjusted R².

My strategy was to evaluate different models because the variables were all correlated and I wasn't expecting much of the regression. My models were based on these insights:

- Review count was totally skewed. Enought to think that their was a kind of categorical underlying aspect of the charactericts: something like Underrated and Well rated.<br>
  
![Yelp_hist](/images/Yelp_hist.png)

- Prices was strongly correlated to the other variables while none of the independant variables had a strong relationship with the dependant variable total_bikes

![Heat map](/images/correlation.png)

After several models, I chose the best one and conducted some tests on the residual (just to convince me that it was really bad).

![The "model"](/images/model.png)

Tests conducted for the residuals were done to evaluate the linear regression assumptions:
+ Linearity: straight line relationship between the x and y
+ Independence of data points
+ Normality of distribution of errors
+ Homoscedasticity: similar sizes of errors
+ No multicollinearity: x’s don't affect each other

Of course, the model all failed these tests. It was a fun ride !!!

## Results
With the data I have collected from the APIs, it was not possible to determined a relationship between the number of bikes in each station and the characteristics of the POIs in that station.
The characteristics evaluated were:
- Number of Poi's per station
- Average of:
    + rating
    + prices
    + popularity
    + number of review
    + Distance to the station

Prices was strongly correlated to the other variables and each then had a weak relationship with the dependant variable. Thus any linear regression model I have attempt didn't provide a good adjusted R² (less than 10%).

It would be hard to try to predict the number of bikes per station based on these POIs characteristics.<br>

As far as I am concerned, Montréal is a small city compare to New York for example. New York as almost 2000 stations and Montréal as 800. Thats a lot of stations for a small area. So, I guess, even if we know how much bike we need with a model, the space is limited, so it affect distance and quantity of bike availabe. Thought the ratings of POIs should be important, we mostly only covers the 'restaurant' category. But, when you rent a bike, its to see the environment outside, the achitecture, the people.<br>

If we really need to predict the quanity of bike, we would need to consider all these other aspect.

## Challenges
There is 2 major parts of the project that were most challenging for me:<br>
- Repeated cleaning of the data<br> 
The cleaning phase seemed infinite. Before, between and after any action on the pandas dataframe, something odd was going on with the datatypes and I had to watch them closely (or getting an error). <br>
- Deciding of which independant variables to evaluate<br>
As a beginner data analyst, it's hard to figure out in advance, based on the data you see in your face, which variable should be use to answer the question. I didn't have much characteristic to check, but even then it seems too much.

Of course, I could state that the missing values was an issue, but its the bread and butter of the job, need to get used to it !


## Future Goals

If I had more time, I would have collected the density of the population on the island around the stations to see if there is ENOUGH stations (not just bikes) per population per area. Because, we could think that bike rentals are a majority of transport for tourists, but in fact, most of the time the bikes are gone because people use it to go to work.

## Tech stacks

- Language:
    - Python
    - SQL
    - MacOs terminal
- Tools:
    - gitHub, gitHub Desktop
    - Jupyter Lab, Notebook
    - VSCode
    - Postman
    - Excel
