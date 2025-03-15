# What Goes Into a Rating?

Author: Vanessa Lau

## Overview

This data science project, conducted at UCSD for the DSC80: Practice and Application of Data Science Course, focuses on exploring factors that can and do contribute to a recipe's average rating.

## Introduction

Food, and by extension cuisine, is something that is deeply entwined with the human race and its plethora of cultures. Whether through word of mouth or writing, recipes have been created and passed down for generations worldwide. But despite different people and cultures having different tastes, there are some commonalities found across the board. Ingredients like salt, sugar, pepper, and garlic are found globally, and different types of dumplings can be found from China to Germany to Peru. Thus, **I want to explore the factors that may influence how people rate different recipes**. To do so, I will be analyzing two datasets, each respectively consisting of recipes and user ratings and reviews posted from 2008 to 2018 - both inclusive - on [food.com](https://www.food.com/).

The first dataset, `recipes`, contains 83782 rows, indicating 83782 unique recipes, with 12 columns recording the following information:

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | The recipe's name                                                                                                                                                                                 |
| `'id'`             | The recipe's ID                                                                                                                                                                                   |
| `'minutes'`        | The minutes to prepare the recipe                                                                                                                                                                 |
| `'contributor_id'` | The ID of the user who submitted the recipe                                                                                                                                                       |
| `'submitted'`      | The date the recipe was submitted                                                                                                                                                                 |
| `'tags'`           | Food.com's tags for recipe                                                                                                                                                                        |
| `'nutrition'`      | The recipe's nutritional information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]. PDV standing for “percentage of daily value” |
| `'n_steps'`        | The number of steps in the recipe                                                                                                                                                                 |
| `'steps'`          | The recipe's steps, ordered                                                                                                                                                                   |
| `'description'`    | The user-provided description                                                                                                                                                                     |
| `'ingredients'`    | The list of the recipe's ingredients                                                                                                                                                              |
| `'n_ingredients'`  | The number of ingredients in the recipe                                                                                                                                                           |

The second dataset, `interactions`, contains 731927 rows and each row contains a review from the user on a specific recipe. The columns it includes are:

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | The reviewer's ID   |
| `'recipe_id'` | The recipe ID       |
| `'date'`      | The date of interaction |
| `'rating'`    | The rating given    |
| `'review'`    | The review's text   |

## Data Cleaning and Exploratory Data Analysis

Before conducting any data analyses, I performed the following data cleaning steps on the two datasets.

1. Left merge the recipes and interactions datasets on 'id' and 'recipe_id', respectively.

   - This step helps match each recipe with its interaction(s). Everything is saved in the `recipes_interactions_merged_df` DataFrame.

2. Check data types of all the columns.

   - This step is to help evaluate what data cleaning steps and/or data type conversions may be needed.
   - | Column             | Description |
     | :----------------- | :---------- |
     | `'name'`           | object      |
     | `'id'`             | int64       |
     | `'minutes'`        | int64       |
     | `'contributor_id'` | int64       |
     | `'submitted'`      | object      |
     | `'tags'`           | object      |
     | `'nutrition'`      | object      |
     | `'n_steps'`        | int64       |
     | `'steps'`          | object      |
     | `'description'`    | object      |
     | `'ingredients'`    | object      |
     | `'n_ingredients'`  | int64       |
     | `'user_id'`        | float64     |
     | `'recipe_id'`      | float64     |
     | `'date'`           | object      |
     | `'rating'`         | float64     |
     | `'review'`         | object      |

3. Fill all ratings of 0 with np.nan.

   - Ratings are given on a scale from 1 (lowest) to 5 (highest). Any ratings of 0 indicates missing a value in rating, whether that be from the reviewer leaving only a review or the recipe having no interactions. Thus, to avoid bias in the ratings, all values of 0 were filled with np.nan.

4. Add column `'avg_rating'`, containing the average rating for the respective recipe.

   - Since a recipe can receive different ratings from different users, by taking an average of all of a given recipe's ratings, we can get a better sense of a more overall or general reaction to the recipe.

5. Rename columns for clarity.

   - Since there are some duplicate and similar feature names in the new merged dataset, I renamed some of them for clarity while I'd be working with them. `'submitted'` became `'recipe_submitted'`, and `'date'` became `'review_date'`.

6. Split values in the nutrition column to individual columns of floats.

   - Despite the fact that the values in the `'nutrition'` column look like lists of floats, they are in fact string representations of lists of floats. Thus, to convert these values to actual lists of strings, I used  `.apply` to apply `json.loads` on the `'nutrition'` column. Then to separate these lists of floats into their own separate columns, each representing a different nutritional aspect, I loaded these lists of floats into their own DataFrame before merging said Dataframe to the larger `recipes_interactions_merged_df` DataFrame. This makes it easier to directly work with each nutritional aspect individually (e.g. `'calories (#)'`).

7. Convert `'recipe_submitted'` and `'review_date'` to datetime.

   - These two columns are both initially stored as objects, so I converted them to datetime to allow easier analysis on trends over time.

8. Convert `'tags'` and `'ingredients'` to lists of strings

   - Similarly to `'nutrition'`, these two features contain string representations of lists of strings (representative of tags and ingredients respectively). Here, I applied the same steps as I did when cleaning the `'nutrition'` feature, with the slight adjustment of using a custom function to apply to `'ingredients'` and use regex to properly clean and convert its values. This allows for text analysis later (e.g. bag-of-words, tfidf)
  
9. Add a column containing the year the recipe was submitted

   - While the `'recipe_submitted'` column has already been converted to datetime, I chose to add an additional column containing just the Year that the recipe was submitted for the sake of ease during later data analysis on trends over time.

#### Result
These are the columns of the resulting cleaned df.

| Column                  | Description    |
| :---------------------- | :------------- |
| `'name'`                | object         |
| `'id'`                  | int64          |
| `'minutes'`             | int64          |
| `'contributor_id'`      | int64          |
| `'recipe_submitted'`    | datetime64[ns] |
| `'tags'`                | object         |
| `'nutrition'`           | object         |
| `'n_steps'`             | int64          |
| `'steps'`               | object         |
| `'description'`         | object         |
| `'ingredients'`         | object         |
| `'n_ingredients'`       | int64          |
| `'user_id'`             | float64        |
| `'recipe_id'`           | float64        |
| `'review_date'`         | datetime64[ns] |
| `'rating'`              | float64        |
| `'review'`              | object         |
| `'avg_rating'`          | object         |
| `'calories (#)'`        | float64        |
| `'total fat (PDV)'`     | float64        |
| `sugar (PDV)'`          | float64        |
| `'sodium (PDV)'`        | float64        |
| `'protein (PDV)'`       | float64        |
| `'saturated fat (PDV)'` | float64        |
| `'carbohydrates (PDV)'` | float64        |
| `'recipe_submitted_year'`| int64         |


THe cleaned dataframe ended up with 234429 rows and 26 columns. Here are the first 5 rows of non-duplicate recipes of our cleaned dataframe for illustration. Since there are many columns in the merged dataframe, Those that are most relevant and/or were altered are displayed below. Scroll right to view more columns.

| name                                 |     id |   minutes | recipe_submitted    |   rating |   average rating |   calories (#) |   sugar (PDV) | recipe_submitted_year   |  
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 |        4 |                4.0 |          138.4 |            50.0 | 2008         |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 |        5 |                5.0 |          595.1 |           211.0 | 2011        | 
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 |        5 |                5.0 |          194.8 |             6.0 | 2008        |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 |        5 |                5.0 |          878.3 |           326.0 | 2008         |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 |        5 |                5.0 |          267   |            12.0 | 2012        |


### Univariate Analysis

<iframe
  src="assets/rating_bar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When looking at the distribution of ratings in the dataset, there is an obvious exponentially increasing trend of the amount of a certain rating being given to the rating itself. In the bar graph above, rating categories 1 and 2 hardly have any ratings, while 4 and especially 5 have the most ratings.

<iframe
  src="assets/calories_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When looking at this boxplot of the distributions of calories among unique recipes in the dataframe, we can see that recipes tend to have less than roughly 500 calories, with an average of about 300 calories. However, there are many outliers to this trend that are cut out of the graph due to the graph being too small to view had they been included in the image.

### Bivariate Analysis

<iframe
  src="assets/calories_vs_year_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When looking at the boxplots of recipes' calories separated by year, there is an overall trend of recipes uploaded to the website becoming higher in calorie as time goes on. There is little variation and only slight fluctuation from 2008 to 2014-2015, but from then on, the calories have sloly trended upwards.

<iframe
  src="assets/ratings_vs_calories_box.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When looking at the boxplots of recipes' calories separated by rating, there seems to be an overall slight downwards trend in calories as rating increases in regards to the first quartile, median, and third quartile.


<iframe
  src="assets/avg_vs_calories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the amount of calories of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall upward trend that directly correlates calories to rating, however, there are many outliers, especially in the average rating 5 category.

<iframe
  src="assets/avg_vs_total_fat_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the total fat (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall upward trend. Similarly to the previous graph, there are many outliers, especially in the average rating 5 category. Furthermore, the relatinship appears a bit non-linear, however, again, that is influenced by the outliers.

<iframe
  src="assets/avg_vs_sugar_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the sugar (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is a bit of an overall upward trend. Similarly to the previous graph, there are many outliers, especially in the average rating 5 category. The relatinship appears a more linear than that of the one comparing total fat, however, again, that is influenced by and dependent on the outliers.

<iframe
  src="assets/avg_vs_sodium_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the sodium (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall upward trend. Similarly to the previous graph, there are a few outliers, most notably in the average rating 3 category. Furthermore, the relatinship appears a bit non-linear, however, again, that is influenced by the outliers.

<iframe
  src="assets/avg_vs_protein_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the protein (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall upward trend. Similarly to the previous graph, there are many outliers, especially in the average rating 5 category. Furthermore, the relatinship appears a bit non-linear, however, again, that is influenced by the outliers.

<iframe
  src="assets/avg_vs_sat_fat_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the saturated fat (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall slight upward trend. Similarly to the previous graphs, there are a few outliers. Furthermore, the relatinship appears a bit non-linear, however, again, that is influenced by the outliers.

<iframe
  src="assets/avg_vs_carb_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
When comparing the carbohydrates (PDV) of a recipe to its average rating (rounded to the nearest integer) in a scatter plot, there is an overall upward trend. Similarly to the previous graph, there are a few outliers, most notably in the average rating 3 and 5 categories.


### Interesting Aggregates

For this section, I wanted to explore the relationship of the proportions of each rating that were given out each year. I was curious as to if there were any trends over time.

|   recipe_submitted_year |   1.0 |   2.0 |   3.0 |   4.0 |   5.0 |
|------------------------:|------:|------:|------:|------:|------:|
|                    2008 |  1110 |  1031 |  3202 | 17112 | 69248 |
|                    2009 |   707 |   657 |  2015 | 10037 | 45735 |
|                    2010 |   340 |   279 |   849 |  4518 | 21959 |
|                    2011 |   211 |   157 |   438 |  2517 | 12771 |
|                    2012 |   204 |   103 |   300 |  1547 |  9616 |
|                    2013 |   166 |    92 |   275 |  1200 |  7355 |
|                    2014 |    62 |    28 |    49 |   213 |  1923 |
|                    2015 |    21 |     5 |    14 |    45 |   461 |
|                    2016 |    16 |     5 |    10 |    30 |   201 |
|                    2017 |    21 |     6 |    13 |    56 |   247 |
|                    2018 |    12 |     5 |     7 |    32 |   160 |

The pivot table above shows a joint distribution of ratings vs year, but as all the values are counts currently, it can be a bit difficult to get a comprehensive sense of the proportions at first glance.

|   recipe_submitted_year |   1.0 |   2.0 |   3.0 |   4.0 |   5.0 |
|------------------------:|------:|------:|------:|------:|------:|
|                    2008 |  0.01 |  0.01 |  0.03 |  0.19 |  0.76 |
|                    2009 |  0.01 |  0.01 |  0.03 |  0.17 |  0.77 |
|                    2010 |  0.01 |  0.01 |  0.03 |  0.16 |  0.79 |
|                    2011 |  0.01 |  0.01 |  0.03 |  0.16 |  0.79 |
|                    2012 |  0.02 |  0.01 |  0.03 |  0.13 |  0.82 |
|                    2013 |  0.02 |  0.01 |  0.03 |  0.13 |  0.81 |
|                    2014 |  0.03 |  0.01 |  0.02 |  0.09 |  0.85 |
|                    2015 |  0.04 |  0.01 |  0.03 |  0.08 |  0.84 |
|                    2016 |  0.06 |  0.02 |  0.04 |  0.11 |  0.77 |
|                    2017 |  0.06 |  0.02 |  0.04 |  0.16 |  0.72 |
|                    2018 |  0.06 |  0.02 |  0.03 |  0.15 |  0.74 |

To address this, I converted the pivot table to show the conditional probabilities of ratings given year (rounded to two decimal places) by dividing the initial table by the sum of itself along axis 1.

Now, we can easily see that as time has progressed, the proportion of lower ratings has slightly and slowly increased, while the proportion of higher scores has slightly and slowly decreased. Although I cannot be entirely sure of the exact cause, potential reasons for this may be related to the internet becoming exponentially larger and having exponentially more resources and recipe options over the course of the decade in question.

## Assessment of Missingness

There are several columns that contain non-trivial missingness in the merged dataframe. These columns include, but are not limited to, `'rating'` and `'description'`.

### NMAR Analysis

I believe that the missingness of the `'description'` column is NMAR. This is because some users may forgo uploading descriptions for their recipes if the recipe names are explanatory enough and/or if the descriptions would be very generic (e.g. name: Orange Juice, description: juicing an orange). Many users may only be compelled to write a description for their recipes if, for example, the recipe is for an ethnic dish that is not very well known to the website's primary audience (e.g. Western people with acceess to the internet), or has a sentimental story attached of how it was passed down for generations in their family.

### Missingness Dependency

However, as there are significantly more instances of missingness in the `'rating'` feature, I decided to analyze the missingness of this column in the merged DataFrame. To do this, I will be investigating its missingness and its dependence on the `'minutes'` and `'recipe_submitted_year'` columns, starting with the former.

#### Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the recipe's cooking time (minutes).

**Alternate Hypothesis:** The missingness of ratings does depend on the recipe's cooking time (minutes).

**Test Statistic:** The absolute mean difference of the distribution of the group without missing ratings' cooking time and the distribution of the group with missing ratings' cooking time.

**Significance Level:** 0.05

To test this set of hypotheses, I ran a permutation test by shuffling the missingness labels of rating 1000 times to create 1000 simulations of absolute mean differences in the two distributions.

<iframe
  src="assets/minutes_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.4523** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.103)** is greater than the significance level of 0.05, we **fail reject the null hypothesis**. The missingness of `'rating'` does not depend on the cooking time of the recipe (`'minutes'`).

#### Recipe Submitted Year and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the year the recipe was submitted.

**Alternate Hypothesis:** The missingness of ratings does depend on the year the recipe was submitted.

**Test Statistic:** The absolute mean difference of the distribution of the group without missing ratings' recipe upload year and the distribution of the group with missing ratings' recipe upload year.

**Significance Level:** 0.05

To test this set of hypotheses, I ran a permutation test by shuffling the missingness labels of rating 1000 times to create 1000 simulations of absolute mean differences in the two distributions.

<iframe
  src="assets/year_missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **0.2963** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is less than the significance level of 0.05, we **reject the null hypothesis**. The missingness of `'rating'` does depend on the year the recipe was uploaded.

## Hypothesis Testing

Another factor that may influence a recipe's rating is the amount of calories it contains. Typically speaking, foods like sugary desserts and other junk foods are typically well-liked, which may lead to them receiving a higher rating because they are enjoyed more, or, users may rate them lower if they are too sweet or greasy. Additionally, these foods tend to be higher in calories than say some pickles due factors like sugars and fats. As a result, I'm curious as to whether there is a significant relationship between a recipe's calories and its rating.

To explore this, I ran a permutation test with the following null and alternative hypotheses, test statistic, and significance level.

**Null Hypothesis:** The amount of calories have no effect on rating

**Alternative Hypothesis:** The amount of calories have an effect on rating

**Test Statistic:** The K-S Statistic

**Significance Level:** 0.05

My choices of test, hypotheses, and test statistic were good choices for several reasons. Firstly, I don't have any information on the underlying population, and I want to see if the different caloric distributions of the difference ratings came from the same population. I decided to do two-sided hypotheses due to the fact that if a certain recipe is too low in calories (e.g. salad with no dressing) or too high in calories (e.g. cupcake with too much sugar), either could affect the rating. Finally, the choice of a K-S test statistic worked best for my test because the K-S statistic is designed for continuous distributions like that of calories, and it can be compared across the 5 different classes of ratings.

<iframe
  src="assets/step_four_emp_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Conclusion of Permutation Test

Since the **p_value** that we found **(0.0)** is less than the significance level of 0.05, we **reject the null hypothesis**. A recipe's amount of calories does have an affect on its rating. One plausible explanation could be the one I provided above regarding insufficient or excessive amounts of sauce.

## Framing a Prediction Problem

For my prediction problem, I've decided to use some of the features in the merged recipes and interactions dataframe to try and predict the average rating of a given recipe. Although the average ratings of each recipe end up being decimals as a result of finding the average of all the individual ratings, reviewers seem to be given the choice to rate a recipe on an integer scale from 1 to 5. Thus, by limiting a recipe's average rating to being an integer by rounding it, we can make this a multiclass classification problem. 

The response variable would be the average rating of a recipe. I chose this because, not only is it the theme of the entire project, but the exploratory data analysis done between this variable and other features such as calories, total fat, sugar, etc., suggests that there is a correlation between rating and each of these features individually. Additionally the average rating of a recipe gives a good sense of people's general thoughts of it overall.

To evaluate my model, I will be using an f1 score instead of accuracy due to the fact that the a majority of the ratings in the dataset are either 4 or 5. Since accuracy can be more easily swayed due to the imbalanced proportions of classes, it would be better to use f1 scoring because it has the option to use either macro f1 (checks if the model performs well on all classes regardless of size) or weighted f1 (checks how the model performs on the real-world distribution of classes).

At the time of prediction, we would have all the information from the features in the recipes dataset. These are features related to the year the recipe was uploaded, its nutritional facts, its tags and ingredients, etc. This would not include the features in the interactions dataset, as if we had that information, we could simply calculate the average rating of each recipe.

## Baseline Model

For my baseline model, I've decided to use a random forest classifier and split my data into both training and test sets. The features I decided to use for this model are `'minutes'` (quantitative), each of the individual nutritional columns (quantitative), `'recipe_submitted_year'` (quantitative), and `'avg_rating'` (rounded to the nearest integer; ordinal; the response variable).

For my transformations, during preprocessing, I applied sklearn's PolynomialFeatures with arguments degree = 2 and include_bias = True to the following features: ['total fat (PDV)', 'sugar (PDV)', 'saturated fat (PDV)', 'carbohydrates (PDV)']. Additionally I applied a RobustScaler to all of the nutritional-related features, as, during my exploratory data analysis in step 2, I saw that these features had many outliers.

This baseline model received a macro f1 score of 0.28 and a weighted f1 score of 0.60. The individual f1 scores for each average rating category are as such in ascending order of rating: 0.03, 0.08, 0.18, 0.44, and 0.66. Although this model performs better in terms of the macro f1 score than simply predicting every recipe to have an average rating of 5 (macro f1 = 0.17, weighted f1 = 0.63), I still don't think it is quite "good" yet. This is because for every average rating category aside from 5, this baseline model's f1 score for the individual category cannot even predict them right a majority of the time. As of now, this baseline model predicts better when the true average rating of a recipe should be 4 or 5, likely due to the overrepresentation of these clases in the original dataset and subsequently the training data.

## Final Model

For my final model, I used the following as my features:

`'minutes'`
This column is the cooking time of the recipe in minutes. I chose to include this feature in my final model due to the fact that if a recipe takes an excessive amount of time to make, there may a correlation to the skill level required (e.g. Japanese Souffle Cheesecake), which may result in a higher chance of users giving it a lower rating due to the difficulty of the recipe affecting its results. Additionally, if a recipe takes excessively long for mediocre results, it can further affect whether a user gives a lower rating.

`'calories (#)'`
This column contains the total calories of a recipe. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. Additionally, in the box plot comparing ratings and calories, as ratings increased, the median amount of calories decreased. To transform this feature, I applied both PolynomialFeatures(degree = 2, include_bias=True) and a RobustScaler(). I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'total fat (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. To transform this feature, I applied both PolynomialFeatures(degree = 2, include_bias=True) and a RobustScaler(). For the PolynomialFeatures, I set include_bias to True because it was the outliers that made the correlation non-linear, and then I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'sugar (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. To transform this feature, I applied both PolynomialFeatures(degree = 2, include_bias=True) and a RobustScaler(). For the PolynomialFeatures, I set include_bias to True because it was the outliers that made the correlation non-linear, and then I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'sodium (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'protein (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'saturated fat (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. To transform this feature, I applied both PolynomialFeatures(degree = 2, include_bias=True) and a RobustScaler(). For the PolynomialFeatures, I set include_bias to True because it was the outliers that made the correlation non-linear, and then I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'carbohydrates (PDV)'`
This column contains the total fat of a recipe in terms of PDV. When looking at the bivariate table between this features and average ratings during the exploratory data analysis, we saw that there was an upward trend between the two in the scatter plot due to the outliers. To transform this feature, I applied both PolynomialFeatures(degree = 2, include_bias=True) and a RobustScaler(). For the PolynomialFeatures, I set include_bias to True because it was the outliers that made the correlation non-linear, and then I used the RobustScaler rather than the StandardizedScaler because the RobustScaler handles the outliers more effectively and better minimizes their effects.

`'ingredients'`
This column contains the ingredients of a recipe as a list of strings. I chose this to add to my final model because certain ingredients can be more likely to be associated with higher or lower ratings (e.g. chocolate vs vegemite). By applying tfidf to this feature, I can assign importance to each ingredient in a given recipe as associated with their commonality overall. Furthermore, if a recipe uses appealing or trendy ingredients, they may be more highly rated by users.

`'tags'`
This column contains the tags of a recipe as a list of strings. I chose this to add to my final model because certain tags can be more likely to be associated with specific audiences (e.g. halal, vegetarian, etc.). By applying tfidf to this feature, I can assign importance to each tag in a given recipe as associated with their commonality overall in addition to user preferences (e.g. easy). Furthermore, if a recipe uses appealing or trendy tags, they may be more highly rated by users.

Again, similarly to my base model, I used a RandomForestClassifier as my modeling algorithm. Although I initially wanted to use GridSearchCV to tune my hyperparameter of max_depth for the RandomForestClassifier, ultimately, due to device constraints and issues, I had to due a very limited RandomizedSearchCV. When passed the max_depth options of 2, 12, and 22, this RandomizedSearchCV determined 12 to be the best max_depth value. I needed to perform a search for this hyperparameter due to the fact that Decision trees are prone to having high variance and can have some overfitting, but by using the hyperparameter of max_depth, I can help control the variance and reduce the model's overfitting. 

When I run my RandomForestClassifier with max_depth set to 12, my model has a macro f1 score of 0.35 and a weighted f1 score of 0.74. Compared to my baseline model, these are improvements of 0.07 and 0.14 respectively. Additionally, the f1 score for each of the average rating categories are as follows in ascending order of average rating: 0.05, 0.17, 0.18, 0.54, and 0.82. This final model is a marked improvement across the board in comparison to the original baseline model, and I believe it is decent as it outperforms simply assigning every recipe an average rating of 5 (macro f1 = 0.17, weighted f1 = 0.63). 

## Fairness Analysis

**Null Hypothesis**: My model is fair. Its precision for recipes with higher calories and lower calories are roughly the same. Any differences are due to random chance.

**Alternative Hypothesis**: My model is unfair. Its precision for recipes with lower calories is different than its precision for recipes with higher calories.

**Test Statistic**: Absolute Difference in macro f1 scores

**Significance Level**: 0.05

**P-Value**: 0.0

**Observed Statistic**: 0.05

For my permutation test, I created a new column that separated the data into two groups: lower or higher calorie. The dividing boundary was 300 calories due to the fact that that was the roughly the median when I sorted the all the data into calorie classes (0-300, 300-600, etc.) and plotted the counts of each class on a bar graph. From here, I calculated the observed absolute difference in macro f1 scores between the discrepancies of the lower and higher calories from my test and predictive ratings before performing the permutation test, in which I shuffled the higher and lower labels, and recalculated the test statistic for 1000 permutations. Then after calculating the p-value of 0.0, I concluded that as the p-value is less than the significance level of 0.05, I reject the null hyothesis that the model is fair. The model's precision for recipes with lower calories is different than its precision for recipes with higher calories.
