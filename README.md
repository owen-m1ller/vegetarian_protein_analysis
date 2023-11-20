# Do Vegetarian Recipes Contain Less Protein?

## Introduction

With the rise of the internet, it's natural that many people have
shifted away from cookbooks and classes to webpages where anyone
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

| Column name | Description |
| tags | A list of categories the food belongs to. We use this list to determine whether recipes are vegetarian or not |
| nutrition | A list of data about the macronutrients of the food. We use this column to find the protein and calories of a recipe |

## Data Cleaning

To prepare to answer our question, the dataframe must be adapted into
a useful form. The main components of this process were fixing the data
types of the columns, extracting features useful to answering our
question, and to check our dataframe for supsicious entries.

### Fixing data types

It is apparent after using the `info()` command that the `submitted`
and `date` columns are not in datetime format. Also, 
the `user_id`, `recipe_id`, and `rating` columns are not stored as
integers. These are easily adjusted to save space and improve clarity

### Extracting useful features

Dealing with the data stored in `tags`, `nutrition`, and `steps` is more
troublesome because the data is stored a text version of a list.
By converting the data into sets of tags, ingredients, and steps, we
start to see the bigger picture of what we are working with.

Below is a sample of tags in the `tags` column.

`{'',
 '1-day-or-more',
 '15-minutes-or-less',
 '3-steps-or-less',
 '30-minutes-or-less',
 '4-hours-or-less',
 '5-ingredients-or-less',
 '60-minutes-or-less',
 'Throw the ultimate fiesta with this sopaipillas recipe from Food.com.',
 'a1-sauce',
 'african',
 'american',
 'amish-mennonite',
 'angolan',
 'appetizers',
 'apples',
 ... }`

The `tags` column contains 549 unique tags, the `steps` column contains
840168 unique steps, and the `ingredients` column contains 11170 unique
ingredients. Since there is so much variety in the `steps` and 
`uniqueness` columns, it is not worth examining them to find patterns
in our dataset.  The `tags` column is much more promising. If we were
using this data to create a predictive model, a one-hot encoding of the
tags column might be very useful data. For the purpose of a hypothesis
test, this overcomplicates the process and clutters our dataframe.
Instead of a one-hot encoding, we will create a new feature: 
`is_vegetarian`. `is_vegetarian` will be a boolean variable found by
testing whether vegetarian or vegan is a tag present in each of the
lists of tags.

Each element in the nutrients column contains a list of length six
with the following structure:
`[calories, total_fat (percent daily value), sugar (percent daily value),
protein (percent daily value), saturated fat (percent daily value), 
carbohydrates (percent daily value)]` Since all the lists are the
same length and each index corresponds to a feature, we can split this
list into 6 new columns in our dataframe. We now have the feature
`protein_pdv` corresponding to the percent of the daily recommended
amount of protein in each recipe.

### Examining suspicious data

The `protein_pdv` column is essential for our analysis.
Unfortunately, many entries of this column are 0 even when the 
corresponding recipe is something abundant in protein like a ham dish.
A strategy to mark these false 0's as NaN values is required. 
Since there are also many recipes for different drinks or 
brines which truly do have no protein, this could compromise our 
analysis. Luckily each recipe has a sizeable list of tags describing
it. Some of these tags contain
ingredients like ham, mushroom, lentils, tofu, or duck. By changing 
the values of `protein_pdv` to NaN for rows where `protein_pdv`
is 0 and a tag signaling that there is protein in the dish is present, we
can largely deal with this issue.

Finally, since the number of comments any recipe has is irrelevant to
its protein content, we should groupby `recipe_id`. If we don't,
the recipes with more comments will be weighted more heavily in our
analysis than those which have fewer comments. We can also drop
columns not relevant to our analysis

The first 5 rows of the cleaned dataframe are displayed as follows.

|   calories |   total_fat_pdv |   protein_pdv |   saturated_fat_pdv |   carbohydrates_pdv | is_vegetarian   |
|-----------:|----------------:|--------------:|--------------------:|--------------------:|:----------------|
|      386.1 |              34 |            41 |                  62 |                   8 | False           |
|      377.1 |              18 |            13 |                  30 |                  20 | False           |
|      326.6 |              30 |            37 |                  51 |                   5 | False           |
|      577.7 |              53 |            14 |                  67 |                  21 | False           |
|      386.9 |               0 |             1 |                   0 |                  33 | False           |

## Univariate Analysis

<iframe src="assets/protein-hist.html" width=800 height=600 frameBorder=0></iframe>

Since we are studying protein concentration in vegetarian vs. 
non-vegetarian food, it's important to view
how protein content is distributed as a whole. One noticeable takeaway is
that there are no obvious subgroups. If we saw two spikes in the graph
it would suggest that there is a singular distinguishing factor that
separates high-protein and low-protein food.

## Bivariate Analysis

<iframe src="assets/protein_and_vegetarian.html" width=800 height=600 frameBorder=0></iframe>

The natural first two variables to explore the relationship between are
`is_vegetarian` and `protein_pdv`. By creating an overlayed histogram,
we can see the distribution of protein in each type of food. The
vegetarian food appears to be much more concentrated around the
low-protein area of the graph.

To find correlating variables, I created a correlation matrix. The most
notable correlation was between `calories` and `protein_pdv` with a
correlation coefficient of about .6. When observing a scatter plot
of `calories` vs. `protein_pdv`, this is not so obvious.

<iframe src="assets/calories-protein.html" width=800 height=600 frameBorder=0></iframe>

When the graph is also colored by `is_vegetarian`, it seems that
vegetarian recipes contain less calories and less protein
<iframe src="assets/calories-protein-color.html" width=800 height=600 frameBorder=0></iframe>

## Interesting Aggregates

If we create a pivot table by grouping by the `is_vegetarian` column, we notice
that there is a stark difference in the average protein in vegetarian foods
and the average protein in non-vegetarian foods. Other macronutrients
also differ by a large amount except for carbohydrates.

|   calories |   total_fat_pdv |   protein_pdv |   saturated_fat_pdv |   carbohydrates_pdv |
|-----------:|----------------:|--------------:|--------------------:|--------------------:|
|    447.312 |         34.3638 |       36.179  |             43.0458 |             13.7943 |
|    334.349 |         22.9797 |       17.6606 |             24.6999 |             13.7446 |

Both this pivot-table and the previous plot demonstrate that
vegetarian recipes may contain both less protein and calories.
This poses a problem for our analysis since most people eat
only a certain amount of calories per day. Vegetarians could
be eating more and making up for the smaller amount of protein they
are eating per recipe. To account for this, we will use a new column
in our hypothesis test: `protein_per_cal`. By studying this variable
we should get a better sense of how much protein vegetarians and
non-vegetarians would be getting over the course of each day.

# Assessment of Missingness

## NMAR Analysis

There are no missing values in the `is_vegetarian` column so the
relevant column to analyze missingness of is `protein_pdv`.

Reflecting on the data-generating process, it's likely that
the values in `protein_pdv` are either inputted by the users
who submit their recipes or are generated by food.com.

If the values are inputted by the users, the missing values
are likely from users who didn't feel the need to input
nutritional information. Whether someone wanted to take the
time to input nutritional information or not doesn't have
any connection with how much protein is in the recipe,
so in this situation the data is not NMAR.

If the values are generated by food.com, perhaps with an
algorithm, then it's likely that missingness would be
connected with how difficult it is to calculate the protein
of a dish. Once again, there is no obvious connection between
how difficult it is to calculate protein concentration of a food
and how much protein is in the dish, so we can conclude that data
is not NMAR.

## Missingness Dependency

To test whether the missingness of protein depends on other columns,
we can perform permutation tests. We will use the difference of means
as our test statistic and have a significance level of 0.5.

The first permutation test was used to test whether protein was
missing conditionally based on whether the food was vegetarian or not.
The null hypothesis assumes that the values in `is_vegetarian` when
protein is missing and when protein is not missing come from the same
distribution. Our p-value was .02, meaning we reject the null
hypothesis and that my system of marking protein values as NaN was
likely biased towards recipes containing meat.

The second permutation test was to test whether protein was missing
conditionally on calorie count. Interestingly enough, our p-value was 0
and so once again we reject the null hypothesis, leading us to
believe that protein is missing conditionally at random
based on calorie count.

<iframe src="assets/dist-cals-protein.html" width=800 height=600 frameBorder=0></iframe>

The rejection of the null hypothesis in both of these tests makes
sense since we'll alright seen some correlation between whether
a recipe is vegetarian and its calorie count.

As a demonstration that this kind of permutation test fails to reject
the null hypothesis when we test with a completely unrelated column,
if we conduct the same test using `id` as our column, we find a p-value
of .1. 


# Hypothesis Testing

It is now time to conduct our hypothesis test. Our null hypothesis
is that the protein per calorie in vegetarian recipes on food.com 
is the same or greater than the protein per calorie in non-vegetarian 
dishes. Our alternative hypothesis is that vegetarian recipes on
food.com have less protein per calorie than non-vegetarian recipes do.

Typically, the larger your dataset and the lower the variance, the
smaller your significance level should be. In our case, our dataset is
quite large with 83782 rows and the variance of the `protein_per_cal`
column is also quite low. Hence we will use a significance level of .01.
We will have 1000 permutations in our test and use the difference in
means as our test statistic. 

<iframe src="assets/empirical.html" width=800 height=600 frameBorder=0></iframe>

The p-value from our permutation test is 0, even with 1000 iterations.
In the above chart, the dark line on the vertical axis
represents the observed difference in means.
Hence we reject the null hypothesis, suggesting that vegetarian 
recipes likely contain less protein per calorie than non-vegetarian
recipes. People nervous about obtaining enough protein throughout 
their day may need to put in some extra care to get 
protein from additional sources if they are relying completely on 
these online recipes.

