# Do Vegetarian Recipes Contain Less Protein?

## Introduction

With the rise of the internet, it was natural that many people would
shift away from cookbooks and classes to webpages where anyone
can find a recipe for whatever they're craving in minutes.

Websites like food.com (where these datasets were scraped from by 
researchers Bodhisattwa Prasad Majumder Shuyang Li, Jianmo Ni, and
Julian McAuley) contain endless assortments of recipes. 
Because of the less curated and structured nature of sites 
where users submit their own recipes, analyzing recipe data helps us
see the big picture of what kinds of recipes are being submitted and
whether they're suitable additions to a balanced diet.

As someone who avoids eating meat, the internet is a great tool for
finding new recipes to try. Something that dwells in many minds
of vegetarians and vegans is whether or not they're eating healthy
amounts of protein. This goal of this project is to find out whether
online vegetarian recipes are offering the same amounts of protein 
as non-vegetarian recipes. If they aren't, those who avoid meat may
need to be careful to find additional ways to incorporate protein into
their diets.

To explore this question, I have combined the aforementioned datasets
of recipe posts and their comments into one dataframe containing
234429 rows and 18 columns, where each row represents a review
of a recipe on food.com.

Some relevant columns of the dataset are as follows:

| column name | description |
| tags | A list of categories the food belongs to. We use this list to determine whether recipes are vegetarian or not |
| nutrition | A list of data about the macronutrients of the food. We use this column to find the protein and calories of a recipe |

