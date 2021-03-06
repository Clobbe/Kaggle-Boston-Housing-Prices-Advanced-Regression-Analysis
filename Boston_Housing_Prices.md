Kaggle Project : Boston Housing - Advanced Regression Techniques
================
Clobbe Norman
11/19/2017

1.What are we dealing with?
===========================

Need to clean up this bulletlist later + write an introduction to the case and thank Pedro Marcelino for the inspiration and [guidance on his approach to this project](https://www.kaggle.com/pmarcelino/comprehensive-data-exploration-with-python)

1.  **Understand the problem** We'll look at each variable and do a philosophical analysis about their meaning and importance for this problem.\*

2.  **Univariable study** We'll just focus on the dependent variable ('SalePrice') and try to know a little bit more about it.

3.  **Multivariate study** We'll try to understand how the dependent variable and independent variables relate.

4.  **Basic cleaning** We'll clean the dataset and handle the missing data, outliers and categorical variables.

5.  **Test assumptions** We'll check if our data meets the assumptions libraryd by most multivariate techniques.

Pick up the right toolset
-------------------------

Meaning, importing the goodies from `tidyverse` for easy data wrangling, `ggplot2` for some nice visualization and `broom` for making sure we don't miss anything while creating our models later on.

``` r
df.test.raw <- read_csv('test.csv', col_names = T)
```

    ## Error in eval(expr, envir, enclos): could not find function "read_csv"

``` r
df.train.raw <- read_csv('train.csv', col_names = T)
```

    ## Error in eval(expr, envir, enclos): could not find function "read_csv"

``` r
df.test <- df.test.raw
```

    ## Error in eval(expr, envir, enclos): object 'df.test.raw' not found

``` r
df.train <- df.train.raw
```

    ## Error in eval(expr, envir, enclos): object 'df.train.raw' not found

Now let's have a look at the the data that we loaded into R.

``` r
df.train %>% 
  glimpse()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Wow! That's impressive - 1,460 observations and 81 variables.

From this quick overview of the dataset it seems like R interpret all the text varibles as `<chr>`, character variables. Something which will mess up later when we want to build our model and doing some visualization.

Let's fix that by turning these characters into proper factors with levels instead.

``` r
df.test <- df.test %>% 
  unclass() %>% 
  as.data.frame()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

``` r
df.train <- df.train %>% 
  unclass() %>% 
  as.data.frame()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Let's have a look now again at the variables.

``` r
df.train %>%
  glimpse()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Let's look at the variables
---------------------------

``` r
df.train %>% 
  colnames() %>% 
  sort()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

We did some variable evaluation, tidious but very useful. It was done in a regular [Google Spreadsheet.](https://docs.google.com/spreadsheets/d/16RMnBO7TQLbaJIiphSrcFAlJq7ewtXyojskUUsE__FM/edit?usp=sharing)

From the initial evaluation with determine that some variables are more interesting for houses and some for apartment houses.

By following the recommendation from previous author we grouped variables into the three categories:

-   `building`

-   `location`

-   `space`

We then also evaluated how much influence each variable would have on the price. Classifying each variable with an expectation of either `Hi`, `Med` or `Low` influence.

The problem is getting more and more tangible and now it's just about validating wether the selected variables indeed are positive correlated with an increase in price.

To get the overview and find which variables that correlate with increased price it's convenient to do a matrix of plot who just take care of everything and give us an image with everything we're interested in.

### Variables expected to have 'Hi' influence

![Correlation matrix plot for hi influence variables](https://dl.dropboxusercontent.com/s/bry1ahaxnyu1s2c/Screenshot%202017-11-25%2023.08.43.png) Looks like some of the variables we expected to have high influence are positively correlated with the price. The highlighted variables are:

-   `OverallQual` (*perhaps the most fluffy variable in this dataset - the overall quality of materials and finish*)

-   `YearBuilt` (*the year when the house was built*)

-   `YearRemodAdd` (*the year when the house last was remodule*)

-   `TotRmsAbvGrd` (*the number of rooms above grade (bathrooms not included)*)

2.What does target varible `SalePrice` look like
================================================

The scope for this project is to build a model that from a set of variables (which we're currently trying to find) will be the basis for a model which in turn can predict the price - SalePrice.

Let's find out what we know about `SalePrice`.

``` r
df.train %>% 
  select(SalePrice) %>% 
  summary()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

### First off - this looks perfect!

There's no zero's, meaning that the variable don't have any outliers that could later on affect our model.

Let's have a look at the distribution of `SalePrice`.

``` r
df.train %>% 
  ggplot(aes(x = SalePrice)) +
  geom_histogram(
      aes(y = ..density..),
      fill = 'blue',
      alpha = 0.4) +
  geom_density(alpha = 0, size = 1)
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Seems like we're dealing with a right skewed distribution, meaning it's deviating from a good ole normal distribution.

Let's find out just how skewed the distribution is.

``` r
library(moments)

df.train %>% 
  select(SalePrice) %>% 
  summarise(
      Skewness = skewness(SalePrice),
      Kurtosis = kurtosis(SalePrice)
  )
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

What does these numbers tell us?

Let's start with `Skewness`. This is simply a number for symmetry which in our case mean that the `mean(SalePrice)` is bigger than the `median(SalePrice)` and as we've already concluded this mean that our distribution is *right skewed* (as seen in plot above).

What about the `Kurtosis`? This is a metric for the *"fatness"* of the tails in our distribution. More specific what this says can be read at [MedCal.org](https://www.medcalc.org/manual/skewnesskurtosis.php)

Let's dig deeper into the relationship
======================================

So up until now we've only looked at the relationship between `SalePrice` and numeric variables. What about the categorical variable `OverallQual`? We already know that's it's related with `SalePrice` but not how much.

``` r
df.train %>% 
  ggplot(aes(x = as.factor(OverallQual), y = SalePrice, fill = as.factor(OverallQual))) + 
    geom_boxplot(outlier.alpha = 0.3,
                 outlier.stroke = 0.5) +
  
    theme(panel.grid.major = element_blank(),
          legend.position="none") +
    
  xlab('OverallQual')
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Well, this is nothing new but now we know how the `Overall Quality` is associated with `SalePrice`

Let's dig deeper on the second categorical variable

``` r
df.train %>% 
  ggplot(aes(x = as.factor(YearBuilt), y = SalePrice, fill = as.factor(YearBuilt))) + 
    geom_boxplot(alpha = 0.8,
                 outlier.alpha = 0.6,
                 position = 'jitter',
                 color = NA) +
  
    geom_smooth(aes(group=1),
                method = "lm",
                se=FALSE,
                color="black") +
  
  theme(panel.grid.major = element_blank(),
        legend.position="none",
        axis.ticks.x = element_blank(),
        axis.text.x = element_blank()) +
  
  xlab('YearBuilt')
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

#### Nice colors! But what does it tell us?

As seen above the black trendline helps us to determine that there is a positive association between `SalePrice` and variable `YearBuilt`.

### So to sum things up…

We've now found that:

-   `YearBuilt`, `OverallQual`, `YearRemodAdd` and `TotRmsAbvGrd` are all variables that's lineraly related with `SalePrice` .

> *"But, hey! That's only 4 variables out of 81 available. Don't you miss out on a lot of potential variables that could have siginificant affect on the target?"*

In Pedro's guide he refer to that the trick for this particular case seems to be `variable selction` rather than `variable engineering`. Which seems intutively right when you think about it since we're given 81 where most of them could be variables that describe the sale price of a house (goingin through and evaluating the variables in a spreadsheet).

So far the selection of variables was based soley on intuition. Next we'll approach the problem more objectively.

3. ~~Intuition~~ , let's go the engineering way
===============================================

To get a sense of the case that we're dealing with we approached this task in an intuitive fashion which is good, but it wasn't too objective, even thou that was our intention. Doing this I had to set a side my analytical engineering mind for a while and trust Pedro's guidance.

Luckily it's time to let the numbers do their thing and for us to approach the `varible selection` in a more objective manner.

Let's get started!

### Heatmap and faceting plots

In order to find which variables that correlate with the target `SalePrice` we could visualize this with a correlation matrix (heatmap style) that will help us determine which varibles to choose. Along with this we're also gonna plot each variable against `SalePrice` as a scatter plots. Which we'll then faceting to get a hold of the bigger picture even more.

Let's start with a heatmap!

``` r
library(reshape2) #loading the right package

#calculating correlations and tiding the data frame for numeric variables
df.train.cor <- df.train %>% 
  select_if(is.numeric) %>% 
  cor() %>% 
  melt()
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

In order to make it easy to filter and eventually dig deeper into the relations between the variables I'm restructuring the data frame with the `melt()` function in `reshape2`-package.

This turn out data frame into a tidy data frame with on row for each correlation between the variable and it's value, like this:

``` r
head(df.train.cor)
```

    ## Error in head(df.train.cor): object 'df.train.cor' not found

``` r
#plotting heatmap with ggplot2

df.train.cor %>% 
  ggplot(aes(x = Var1, y = Var2, fill = value)) +
  geom_tile() +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

Oh! Cool that's what I'm talking about, but it's not to easy to decipher. Let's work on that by minimize the number of variables to look at through some nice piping and filtering in [dplyr](http://dplyr.tidyverse.org/).

We're interested in reducing the noise that make it harder for us distinguish which variables are contributing to an increase on the target `SalePrice`.

At first iteration I'm setting threshold to `0.5` and we'll see wether we need to increase or decrease it.

``` r
df.train.cor %>% 
  filter(value >= 0.5) %>%
  ggplot(aes(x = Var1, y = Var2, fill = value)) +
  geom_tile() + 
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

``` r
##lägg till finlir med att ta bort variabler som inte är av intresse
```

That's much better!

With this plot it's now easier for us to decipher [the signal from the noise](https://www.amazon.com/Signal-Noise-Many-Predictions-Fail-but/dp/0143125087/ref=sr_1_1?ie=UTF8&qid=1512298650&sr=8-1&keywords=signal+and+the+noise).

What we can see among the numerical variables now is that there's 10 that set them self from the crowed:

-   `OverallQual`
-   `YearBuilt`
-   `YearRemodAdd`
-   `TotalBsmtSF`
-   `X1stFlrSF`
-   `GrLivArea`
-   `FullBath`
-   `TotRmsAbvGrd`
-   `GarageCars`
-   `GarageArea`

''' städa upp och skriv rent / reflektera över om Pedros reflektioner återspeglas…

According to our crystal ball, these are the variables most correlated with 'SalePrice'. My thoughts on this:

'OverallQual', 'GrLivArea' and 'TotalBsmtSF' are strongly correlated with 'SalePrice'. Check! 'GarageCars' and 'GarageArea' are also some of the most strongly correlated variables. However, as we discussed in the last sub-point, the number of cars that fit into the garage is a consequence of the garage area. 'GarageCars' and 'GarageArea' are like twin brothers. You'll never be able to distinguish them. Therefore, we just need one of these variables in our analysis (we can keep 'GarageCars' since its correlation with 'SalePrice' is higher). 'TotalBsmtSF' and '1stFloor' also seem to be twin brothers. We can keep 'TotalBsmtSF' just to say that our first guess was right (re-read 'So... What can we expect?'). 'FullBath'?? Really? 'TotRmsAbvGrd' and 'GrLivArea', twin brothers again. Is this dataset from Chernobyl? Ah... 'YearBuilt'... It seems that 'YearBuilt' is slightly correlated with 'SalePrice'. Honestly, it scares me to think about 'YearBuilt' because I start feeling that we should do a little bit of time-series analysis to get this right. I'll leave this as a homework for you. Let's proceed to the scatter plots. '''

With these variables let's label them now as either `medium` or `high` like we did in part 1.

``` r
interestingVariables <- c('OverallQual','YearBuilt','YearRemodAdd','TotalBsmtSF','X1stFlrSF',
                          'GrLivArea','FullBath','TotRmsAbvGrd','GarageCars','GarageArea')

df.train.cor.filtered <- df.train.cor %>% 
  filter(Var1 == 'SalePrice' &
           value > 0.5 &
           value < 1) %>%
  mutate(label = ifelse(value < 0.75, yes = 'medium', no = 'high'))
```

    ## Error in eval(expr, envir, enclos): could not find function "%>%"

As one can see the labels doesn't contribute that much to our case, but it was a good and valid hypothesis to try. So for now, let's continue without it.

### Health check - can we trust this?

Anyhow these 10 variables are correlated we an increase in price. Let's do some health check on each correlation to see that it's also statistically significant as well.

We'll be doing this by using `map()` from the `purrr`-package to generate a linear model ( `lm()` ) including the nice coefficients for each of the variables.

``` r
library(tidyr)
```

    ## 
    ## Attaching package: 'tidyr'

    ## The following object is masked from 'package:reshape2':
    ## 
    ##     smiths

``` r
library(purrr)


df.train.model <- data.frame(variable = interestingVariables,stringsAsFactors = FALSE)


df.train.model$model <- nest(lm(SalePrice ~ df.train[,which(colnames(df.train) == df.train.model$variable[1])], df.train))
```

    ## Error in is.data.frame(data): object 'df.train' not found

``` r
df.train.model
```

    ##        variable
    ## 1   OverallQual
    ## 2     YearBuilt
    ## 3  YearRemodAdd
    ## 4   TotalBsmtSF
    ## 5     X1stFlrSF
    ## 6     GrLivArea
    ## 7      FullBath
    ## 8  TotRmsAbvGrd
    ## 9    GarageCars
    ## 10   GarageArea

``` r
  df.train.model %>% 
  mutate(model = map(df.train, ~lm(SalePrice ~ df.train.model$variable, .)))
```

    ## Error in function_list[[k]](value): could not find function "mutate"

``` r
# Perform a linear regression on each item in the data column
by_year_country %>%
  nest(-country) %>%
  mutate(model = map(data, ~ lm(percent_yes ~ year, . )))
```

    ## Error in eval(expr, envir, enclos): object 'by_year_country' not found

``` r
  mutate(tidied = tidy(model))
```

    ## Error in eval(expr, envir, enclos): could not find function "mutate"

``` r
#exempel på hur gather funkar => riktigt användbart!
  votes_gathered <- votes_joined %>%
  gather(topic, has_topic, me:ec) %>%
  filter(has_topic == 1)
```

    ## Error in eval(expr, envir, enclos): object 'votes_joined' not found

``` r
  # Filter for only the slope terms
slope_terms <- country_coefficients %>%
  filter(term == "year") %>%
  mutate(p.adjusted = p.adjust(p.value))
```

    ## Error in eval(expr, envir, enclos): object 'country_coefficients' not found

``` r
# Add p.adjusted column, then filter
slope_terms %>%
  filter(p.adjusted < 0.05)
```

    ## Error in eval(expr, envir, enclos): object 'slope_terms' not found

1.  gen lm för topp 10 var
2.  kolla outliers via boxplot för topp 10 + kolla Pedro's
3.  välj ut variabler och testa vilka som faktiskt skulle kunna predicta SalePrice
4.  Utvärdera resultat =&gt; välj modell
