
![readme banner 5](https://github.com/user-attachments/assets/4f0fcaaa-5e82-40fa-b715-ab8ecf6b5112)


The Gross Domestic Product (GDP) of a country is an influential factor in the life expectancy of individuals in its population. This correlation can be measured and used in further life expectancy analysis to identify causal relationships. 

## SQL
### Upload GDP and Life Expectancy data to BigQuery

-- `API_NY.GDP.MKTP.CD_DS2_en_csv_v2_3403845.csv`

and

-- `life_expectancy_table.csv`

### Unpivot the GDP table for consistency
````
CREATE OR REPLACE TABLE `pathway-to-discovery.gdp_by_country.unpivoted_gdp` AS
SELECT
  country_name,
  year,
  gdp
FROM
  `pathway-to-discovery.gdp_by_country.gdp_new_one`
UNPIVOT (
  gdp FOR year IN (
    `1995`, `1996`, `1997`, `1998`, `1999`,
    `2000`, `2001`, `2002`, `2003`, `2004`,
    `2005`, `2006`, `2007`, `2008`, `2009`,
    `2010`, `2011`, `2012`, `2013`, `2014`,
    `2015`, `2016`, `2017`, `2018`, `2019`,
    `2020`, `2021`, `2022`, `2023`
  )
);
````

### Convert the year datatype from FLOAT to INTEGER
````
CREATE OR REPLACE TABLE `pathway-to-discovery.gdp_by_country.unpivoted_gdp` AS
SELECT
  country_name,
  CAST(year AS INT64) AS year,  -- Casting the year column to INTEGER
  gdp
FROM
  `pathway-to-discovery.gdp_by_country.unpivoted_gdp`
````
### Join the life expectancy and GDP tables
````
SELECT
  gdp_data.country_name,
  gdp_data.year,
  gdp_data.gdp,
  life_expectancy_data.life_expectancy
FROM
  `pathway-to-discovery.gdp_by_country.unpivoted_gdp` AS gdp_data
JOIN
  `bigquery-public-data.census_bureau_international.mortality_life_expectancy` AS life_expectancy_data
ON
  gdp_data.country_name = life_expectancy_data.country_name
  AND gdp_data.year = life_expectancy_data.year
WHERE 
  gdp_data.country_name IS NOT NULL
  AND gdp_data.year IS NOT NULL
  AND gdp_data.gdp IS NOT NULL
  AND life_expectancy_data.life_expectancy IS NOT NULL
ORDER BY
  gdp_data.year ASC,
  gdp_data.country_name ASC;

````

## Tableau
### Load the resulting file into Tableau

File: `gdpandlifeexp.csv`

### Create a dashboard to visualize the data, allowing filtering by country and year

[Link: GDP and Life Expectancy Dashboard](https://public.tableau.com/views/GDPandLifeExpectancy_17254988476380/Dashboard1?:language=en-US&:sid=&:redirect=auth&:display_count=n&:origin=viz_share_link)

## Analysis in R Studio
![Rplot02](https://github.com/user-attachments/assets/6bb68b56-cbc7-43ec-980d-e00e0dede6c1)


### Load the .csv file into R Studio
````
library(dplyr)
library(ggplot2)
````
### Calculate the correlation for each year
````
correlation_data <- gdpandlifeexp %>%
group_by(year) %>%
summarize(correlation = cor(gdp, life_expectancy, use = "complete.obs"))
````
### Create a line plot to show the change in correlation over time
````
plot <- ggplot(correlation_data, aes(x = year, y = correlation)) +
geom_line(color = "blue") +
geom_point(color = "red") +
labs(title = "Change in Correlation between GDP and Life Expectancy Over Time",
x = "Year", y = "Correlation Coefficient") +
theme_minimal() +
theme(axis.text.x = element_text(angle = 45, hjust = 1)) # Rotate x-axis labels for better readability
````
### Print the plot
````
print(plot)
````
## Conclusion
GDP is an important factor that affects life expectancy in some countries. This analysis measured the correlation between countries and across time. This data can be used to control for the GDP factor when conducting analysis of life expectancy and measuring alternative factors. 
