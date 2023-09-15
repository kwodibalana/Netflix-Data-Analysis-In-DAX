# Netflix-Data-Analysis-In-DAX
In this project, I do an exploratory data analysis with DAX on the Netflix data, available on kaggle.com

## Introduction
I recently decided to work on some data that I downloaded on Kaggle and I challenged myself to do some data analysis with the language named Data Analysis Expressions (DAX). DAX is a language a bit similar to SQL and allows the user to write queries. To run the queries, I use DAX Studio which is a tool to write, execute, and analyze DAX queries in Power BI, Power Pivot for Excel, and Analysis Services Tabular. It includes the possibilities to query, edit and execute, to edit formulas and measures editing, to format and highligth syntax. DAX Studio can be connected to an excel file, to Power BI, to...
The dataset we'll be working on, belongs to the streaming platform called Netflix. 

#### Project Goal
An exploratory data analysis is conducted. To do so, I tried to give answers to qestions I asked myself. These questions can be interesting 

## Data Source
The dataset comes from the kaggle site and is composed of two csv files : one for the movie/show titles  and the other one for the cast.

## Data dictionary

    id: The title ID on JustWatch.
    title: The name of the title.
    type: TV show or movie.
    description: A brief description.
    release_year: The release year.
    age_certification: The age certification.
    runtime: The length of the episode (SHOW) or movie.
    genres: A list of genres.
    production_countries: A list of countries that - produced the title.
    seasons: Number of seasons if it's a SHOW.
    imdb_id: The title ID on IMDB.
    imdb_score: Score on IMDB.
    imdb_votes: Votes on IMDB.
    tmdb_popularity: Popularity on TMDB.
    tmdb_score: Score on TMDB.

### Data tools

   - Excel : for the query to the csv files
   - Power Query : to clean the data
   - Power Pivot : to create the relationship between the tables
   - DAX Studio : for the analysis

### Connecting to the source of the data
First of all, I added the two csv files to a model in an excel sheet,using Power query connexion only with the csv files. Then, I launched DAX Studio Add-in in Excel. It is important to launch DAX studio from the excel sheet, to be able to connect DAX Studio to the excel sheet. Actually, by doing so, I connected DAX Studio to the data model.

## Data Cleaning
With Power Query, I did some column formatting. Columns imdb score, imdb vote, tmdb score were stored in text format. I transform it in integer/>Float. I could delete  unuseful columns but I didn't.

## Data Analysis
Here are the questions and details of the answers
  ### -  1) Number of movies/Tv show per year
The corresponding DAX code is :

             EVALUATE
             SUMMARIZE(titles,titles[release_year],"Nombre",COUNTROWS(titles))

             and the 10 first rows of output are :
             release_year	Nombre
             2008	43
             2010	53
             2012	84
             2011	61
             2014	137
             2016	312
             2015	195
             2017	475
             2018	743
             2020	829

This code can be improved by ordering the numbers.


 ### - 2) How many movies/TV shows from january to march 2023??
My DAX query is edited as followed :

             EVALUATE
             CALCULATETABLE(
                            ROW("Nombre",COUNTROWS(titles)),
                            FILTER(titles,titles[release_year]=2023)
                           )
              
  ### - 3) How many movies/TV shows by type in 2023???
For the answer to this question, I wrote the query like this :

            EVALUATE
            CALCULATETABLE(
                           SUMMARIZE(titles,titles[type],"Nombre",COUNTROWS(titles)),
                           FILTER(titles,titles[release_year]=2023)
                          )
      And the output is :

            type	Nombre
            TV Show	50
            Movie	47

  ### - 4) List of movies/Tv Shows of 2023 you would like to see , according to the category and the rating
To this question, I tried to respond by choosing the drama category. Here is the query edited :

            EVALUATE
            CALCULATETABLE (
                            SELECTCOLUMNS (
                                           credits,
                                           "Titre du film",RELATED(titles[title]),
                                           "Année de sortie", RELATED(titles[release_year]),
                                           "Genre",RELATED(titles[genres])
                                          ),
            titles[imdb_score]=8.0,
            titles[release_year]= 2023,
            titles[genres]="['drama']"
                           )
  
  ### - 5) List of actors who acted in a  given movie/Tv show
  The TV show that I chose is Stranger things.
  I edited the query like this :

            EVALUATE
            CALCULATETABLE (
                            SELECTCOLUMNS (
                                           credits,
                                           "Rôle", credits[role],
                                           "Nom de l'acteur", credits[name],
                                           "Titre du film",RELATED(titles[title]),
                                           "Année de sortie", RELATED(titles[release_year]),
                                           "Genre",RELATED(titles[genres])
                                          ),
            titles[title]="Stranger things"
                          )

   ### - 6) runtime of a movie
   I chose the movie Top Gun. The query is :

           EVALUATE
           CALCULATETABLE(
                          ROW("Durée",VALUES(titles[runtime])),
                          FILTER(titles,titles[title]="TOP GUN")
                         ) 

   ### - 7) Number of movie/TV show per director
   Here is the query edited :

           EVALUATE
           SUMMARIZECOLUMNS(credits[name],
                            FILTER(credits, credits[role]="DIRECTOR"),
                            "Nombre de films",DISTINCTCOUNT(titles[title]) 
                           )

  ### - 8) Average runtime for a specific director : Sylvester Stallone for example
The query to answer to this question is :

           EVALUATE
           SUMMARIZECOLUMNS(credits[name],
                            FILTER(credits,credits[name]="Sylvester Stallone"),
                            "Moyenne", AVERAGE(titles[runtime])
                           )

### - 9) What is the variation in % of movie/TV shows for the period 2021 - 2022???
Here is the query :

           DEFINE 
                MEASURE titles[Nombre 2021] =
                                             CALCULATETABLE(
                                                            ROW("Nombre",COUNTROWS(titles)),
                                                            FILTER(titles,titles[release_year]=2021)
                                                           )
                MEASURE titles[Nombre 2022] =
                                             CALCULATETABLE(
                                                            ROW("Nombre",COUNTROWS(titles)),
                                                            FILTER(titles,titles[release_year]=2022)
                                                           )
                MEASURE titles[Variation] =
                                           CALCULATE(([Nombre 2022]/[Nombre 2021])-1)
           EVALUATE 
           SUMMARIZECOLUMNS(titles,"Nombre de films 2021",[Nombre 2021],
                            "Nombre de films 2022",[Nombre 2022],
                            "Variation en %",[Variation]
                           )

  ### - 10) What is the variation in % of movie/TV shows per type for the period 2021 - 2022?
We proceed with the previous code and we present the variation per type :

          DEFINE 
               MEASURE titles[Nombre 2021] =
                                            CALCULATETABLE(
                                                           ROW("Nombre",COUNTROWS(titles)),
                                                           FILTER(titles,titles[release_year]=2021)
                                                          )
               MEASURE titles[Nombre 2022] =
                                            CALCULATETABLE(
                                                           ROW("Nombre",COUNTROWS(titles)),
                                                           FILTER(titles,titles[release_year]=2022)
                                                          )
               MEASURE titles[Variation] =
                                          CALCULATE(([Nombre 2022]/[Nombre 2021])-1)
          EVALUATE 
          SUMMARIZE(titles,titles[type],"Nombre de films 2021",[Nombre 2021],
                    "Nombre de films 2022",[Nombre 2022],
                    "Variation en %",[Variation]
                   ) 

### - 11) What is the variation in value of movie/TV shows per type for the period 2021 - 2022?
The code is :


         DEFINE 
              MEASURE titles[Nombre 2021] =
                                           CALCULATETABLE(
                                                          ROW("Nombre",COUNTROWS(titles)),
                                                          FILTER(titles,titles[release_year]=2021)
                                                         )
              MEASURE titles[Nombre 2022] =
                                           CALCULATETABLE(
                                                          ROW("Nombre",COUNTROWS(titles)),
                                                          FILTER(titles,titles[release_year]=2022)
                                                         )
             MEASURE titles[Variation] =
                                        CALCULATE([Nombre 2022]-[Nombre 2021])
         EVALUATE 
         SUMMARIZE(titles,titles[type],"Nombre de films 2021",[Nombre 2021],
                   "Nombre de films 2022",[Nombre 2022],
                   "Variation en valeur",[Variation]
                  )


### Keywords :
DAX, Power Query, Power Pivot, SQLBI, Netflix


###  - Descriptive Analysis

### Diagnostic Analysis

### Predictive Analysis

### Prescriptive Analysis

## References
