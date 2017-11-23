Just another price prediction - Kaggle Boston Housing
================
Clobbe Norman
11/19/2017

Kaggle :: Boston Housing - Advanced Regression Techniques
=========================================================

Need to clean up this bulletlist later

-   Understand the problem We'll look at each variable and do a philosophical analysis about their meaning and importance for this problem.\*

-   Univariable study We'll just focus on the dependent variable ('SalePrice') and try to know a little bit more about it.

-   Multivariate study We'll try to understand how the dependent variable and independent variables relate.

-   Basic cleaning We'll clean the dataset and handle the missing data, outliers and categorical variables.

-   Test assumptions We'll check if our data meets the assumptions required by most multivariate techniques.

Pick up the right toolset
-------------------------

Look at the col names to find cool variables
--------------------------------------------

``` r
sort(colnames(df.train))
```

    ##  [1] "Alley"         "BedroomAbvGr"  "BldgType"      "BsmtCond"     
    ##  [5] "BsmtExposure"  "BsmtFinSF1"    "BsmtFinSF2"    "BsmtFinType1" 
    ##  [9] "BsmtFinType2"  "BsmtFullBath"  "BsmtHalfBath"  "BsmtQual"     
    ## [13] "BsmtUnfSF"     "CentralAir"    "Condition1"    "Condition2"   
    ## [17] "Electrical"    "EnclosedPorch" "ExterCond"     "Exterior1st"  
    ## [21] "Exterior2nd"   "ExterQual"     "Fence"         "FireplaceQu"  
    ## [25] "Fireplaces"    "Foundation"    "FullBath"      "Functional"   
    ## [29] "GarageArea"    "GarageCars"    "GarageCond"    "GarageFinish" 
    ## [33] "GarageQual"    "GarageType"    "GarageYrBlt"   "GrLivArea"    
    ## [37] "HalfBath"      "Heating"       "HeatingQC"     "HouseStyle"   
    ## [41] "Id"            "KitchenAbvGr"  "KitchenQual"   "LandContour"  
    ## [45] "LandSlope"     "LotArea"       "LotConfig"     "LotFrontage"  
    ## [49] "LotShape"      "LowQualFinSF"  "MasVnrArea"    "MasVnrType"   
    ## [53] "MiscFeature"   "MiscVal"       "MoSold"        "MSSubClass"   
    ## [57] "MSZoning"      "Neighborhood"  "OpenPorchSF"   "OverallCond"  
    ## [61] "OverallQual"   "PavedDrive"    "PoolArea"      "PoolQC"       
    ## [65] "RoofMatl"      "RoofStyle"     "SaleCondition" "SalePrice"    
    ## [69] "SaleType"      "ScreenPorch"   "Street"        "TotalBsmtSF"  
    ## [73] "TotRmsAbvGrd"  "Utilities"     "WoodDeckSF"    "X1stFlrSF"    
    ## [77] "X2ndFlrSF"     "X3SsnPorch"    "YearBuilt"     "YearRemodAdd" 
    ## [81] "YrSold"

We did some variable evaluation, tidious but very useful. It was done in a regular Google Spreadsheet.

From the initial evaluation with determine that some variables are more interesting for houses and some for apartment houses.

By following the recommendation from previous author we grouped variables into the three categories: \* building \* location \* space

``` r
df.train.segment.building <- df.train[,segment.building]
df.train.segment.location <- df.train[,segment.location]
df.train.segment.space <- df.train[,segment.space]
```

### Creating a linear model between each variable in each segment to find

Not sure yet wether to generate lm() first and then plot each model or create all plots with ggplot with lm() directely and then plot all with facet\_grid…

``` r
#df.segment.building.model <- data.frame(variable = colnames(df.train.segment.building), model = 'na', rsq = 'na')

#df.segment.location.model <- data.frame(variable = colnames(df.train.segment.location), model = 'na', rsq = 'na')

#df.segment.space.model <- data.frame(variable = colnames(df.train.segment.space), model = 'na', rsq = 'na')

# Code copy + pasted from DataCamp to gen
# multiple plots, maybe it's possible to gen models with purrr-package?
# library(plyr)
# my_plots <- dlply(mtcars, .(cyl), function(df){
#   ggplot(df, aes(mpg, wt)) +
#     geom_point() +
#     xlim(range(mtcars$mpg)) +
#     ylim(range(mtcars$wt)) +
#     ggtitle(paste(df$cyl[1], 'cylinders'))
# })
# 
# 
# models <- by_country %>% 
#   mutate(model = map(data, country_model)) %>%
#   # model results are summarised in tidy dataframes using broom
#   mutate(glance = map(model, broom::glance),
#          rsq    = glance %>% map_dbl("r.squared"),
#          tidy   = map(model, broom::tidy),
#          augment= map(model, broom::augment))
```

Note that the `echo = FALSE` parameter was added to the code chunk to prevent printing of the R code that generated the plot.
