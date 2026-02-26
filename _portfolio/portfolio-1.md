---
title: "Predicting Market Activity: A Linear Regression Analysis of Texas Housing Inventory (2010 – 2015)"
excerpt: "<img src='https://images.unsplash.com/flagged/photo-1558954157-aa76c0d246c6?q=80&w=1931&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'>"
collection: portfolio
---

*by: Erin Novoa*


**Project Overview**

Data Source: `txhousing` dataset built in `ggplot2` package in RStudio

**Objectives**
- Exploratory Data Analysis
- Hypotheses Testing
- Linear Regression Model Analysis
- Model Validity

You can also view full project on my [GitHub](https://github.com/eknovoa/analysis_projects_in_R/blob/main/TXHousing_Regression.pdf).

---

This project analyzes the relationship between housing inventory and
market liquidity across Texas (2010–2015) using the `txhousing` dataset
in the `ggplot2` package in R. I developed a linear regression model
revealing that active listings explain 63.4% of sales volume variation
($R^2 = 0.6342$), with each additional listing generating approximately
\$46,987 in market value. While highly significant ($p < 0.001$),
residual diagnostics indicate that market predictability is highest in
small to mid-sized cities but faces increased volatility in high-growth
metropolitan hubs like Houston and Dallas.

## Setup R for the analysis

``` r
library(ggplot2)
```

## 1. Data Exploration (EDA)

``` r
# view txhousing data
head(txhousing)
```

    ## # A tibble: 6 × 9
    ##   city     year month sales   volume median listings inventory  date
    ##   <chr>   <int> <int> <dbl>    <dbl>  <dbl>    <dbl>     <dbl> <dbl>
    ## 1 Abilene  2000     1    72  5380000  71400      701       6.3 2000 
    ## 2 Abilene  2000     2    98  6505000  58700      746       6.6 2000.
    ## 3 Abilene  2000     3   130  9285000  58100      784       6.8 2000.
    ## 4 Abilene  2000     4    98  9730000  68600      785       6.9 2000.
    ## 5 Abilene  2000     5   141 10590000  67300      794       6.8 2000.
    ## 6 Abilene  2000     6   156 13910000  66900      780       6.6 2000.

``` r
years <- unique(txhousing$year)
years
```

    ##  [1] 2000 2001 2002 2003 2004 2005 2006 2007 2008 2009 2010 2011 2012 2013 2014
    ## [16] 2015

``` r
# save a subset of the txhousing data for the years 2010 - 2015, inclusive
txhousing_after_2010 <- txhousing[txhousing$year >= 2010,]
txhousing_after_2010
```

    ## # A tibble: 3,082 × 9
    ##    city     year month sales   volume median listings inventory  date
    ##    <chr>   <int> <int> <dbl>    <dbl>  <dbl>    <dbl>     <dbl> <dbl>
    ##  1 Abilene  2010     1    73  9130783 112200      868       6.4 2010 
    ##  2 Abilene  2010     2    93 10372904  98300      830       6.1 2010.
    ##  3 Abilene  2010     3   133 16517713 114000      854       6.3 2010.
    ##  4 Abilene  2010     4   161 18788002 103600      859       6.3 2010.
    ##  5 Abilene  2010     5   200 22804393  99300      914       6.5 2010.
    ##  6 Abilene  2010     6   169 23216943 127900      932       6.7 2010.
    ##  7 Abilene  2010     7   159 22363123 127300      915       6.6 2010.
    ##  8 Abilene  2010     8   144 17504580 122000      936       6.7 2011.
    ##  9 Abilene  2010     9   116 15475763 121300      899       6.5 2011.
    ## 10 Abilene  2010    10   111 14570529 111900      863       6.4 2011.
    ## # ℹ 3,072 more rows

The summary statistics of the dataset.

``` r
summary(txhousing_after_2010)
```

    ##      city                year          month            sales       
    ##  Length:3082        Min.   :2010   Min.   : 1.000   Min.   :   6.0  
    ##  Class :character   1st Qu.:2011   1st Qu.: 3.000   1st Qu.:  81.0  
    ##  Mode  :character   Median :2012   Median : 6.000   Median : 160.0  
    ##                     Mean   :2012   Mean   : 6.239   Mean   : 541.8  
    ##                     3rd Qu.:2014   3rd Qu.: 9.000   3rd Qu.: 436.8  
    ##                     Max.   :2015   Max.   :12.000   Max.   :8945.0  
    ##                                                     NA's   :8       
    ##      volume              median          listings         inventory     
    ##  Min.   :1.019e+06   Min.   : 55000   Min.   :    0.0   Min.   : 0.000  
    ##  1st Qu.:1.150e+07   1st Qu.:119325   1st Qu.:  638.8   1st Qu.: 4.400  
    ##  Median :2.441e+07   Median :139750   Median : 1173.0   Median : 6.800  
    ##  Mean   :1.214e+08   Mean   :146639   Mean   : 2636.4   Mean   : 7.893  
    ##  3rd Qu.:7.837e+07   3rd Qu.:168075   3rd Qu.: 2343.5   3rd Qu.: 9.500  
    ##  Max.   :2.568e+09   Max.   :304200   Max.   :40409.0   Max.   :55.900  
    ##  NA's   :8           NA's   :8        NA's   :46        NA's   :49      
    ##       date     
    ##  Min.   :2010  
    ##  1st Qu.:2011  
    ##  Median :2013  
    ##  Mean   :2013  
    ##  3rd Qu.:2014  
    ##  Max.   :2016  
    ## 

``` r
boxplot(txhousing_after_2010$listings, horizontal = TRUE)
```

![](./images/boxplot1.png)

``` r
boxplot(txhousing_after_2010$volume, horizontal = TRUE)
```

![](./images/boxplot2.png)<!-- -->

The data is heavily right-skewed due to larger cities like
Houston/Dallas vs smaller markets. We can see the difference in the
cities like Houston and Dallas versus other cities in Texas by viewing
the barplot below. We can see our “Big Four” Cities like Houston,
Dallas, San Antonio, and Austin all having more total active listings
compared to other cities in Texas.

``` r
# 2. Sum the listings by city
sum_listings <- aggregate(listings ~ city, data = txhousing_after_2010,
                          FUN = sum, na.rm = TRUE)

# 3. Sort by total listings (descending)
sum_listings <- sum_listings[order(sum_listings$listings, decreasing = TRUE), ]
sum_listings
```

    ##                     city listings
    ## 20               Houston  1794158
    ## 12                Dallas  1115936
    ## 37           San Antonio   694922
    ## 4                 Austin   543026
    ## 16            Fort Worth   235063
    ## 14               El Paso   233039
    ## 10         Collin County   224539
    ## 15             Fort Bend   220107
    ## 5               Bay Area   212130
    ## 30     Montgomery County   203907
    ## 43                 Tyler   191440
    ## 32     NE Tarrant County   150614
    ## 13         Denton County   146426
    ## 28               McAllen   145491
    ## 11        Corpus Christi   144598
    ## 25     Longview-Marshall   113632
    ## 6               Beaumont   111620
    ## 19             Harlingen   105974
    ## 23     Killeen-Fort Hood    96789
    ## 9  Bryan-College Station    94070
    ## 26               Lubbock    91456
    ## 45                  Waco    90773
    ## 2               Amarillo    86834
    ## 3              Arlington    84245
    ## 41         Temple-Belton    68325
    ## 17             Galveston    61410
    ## 1                Abilene    61007
    ## 46         Wichita Falls    60134
    ## 40    South Padre Island    56308
    ## 39       Sherman-Denison    54463
    ## 22             Kerrville    51278
    ## 42             Texarkana    44934
    ## 8            Brownsville    43156
    ## 35           Port Arthur    41407
    ## 7        Brazoria County    37021
    ## 24                Laredo    36968
    ## 29               Midland    36891
    ## 21                Irving    35563
    ## 18               Garland    33976
    ## 36            San Angelo    33296
    ## 27                Lufkin    26231
    ## 34                 Paris    22528
    ## 44              Victoria    21289
    ## 33                Odessa    19929
    ## 31           Nacogdoches    17455
    ## 38            San Marcos     9654

``` r
par(mar = c(8, 7, 4, 2))
par(mgp = c(4.5, 1, 0))

# 4. Plot the top 10
barplot(height = sum_listings$listings[1:10] / 1000, 
        names.arg = sum_listings$city[1:10],
        las = 2, 
        col = "seagreen",
        cex.names = 0.8,
        main = "Total Active Listings (2010-2015) for Top 10 cities",
        ylab = "Total Listings (in Thousands)")
```

![](./images/barplot.png)<!-- -->

The Texas housing market, the “Big Four” cities (Houston, Dallas, San
Antonio, and Austin) account for a vast majority of the state’s
inventory. This concentration explains the high number of outliers in
the initial boxplot analysis and suggests that a single linear model may
be heavily influenced by these metropolitan hubs.

Examining the strength of the relationship between the number of active
listings and the volume of sales for the years of 2010 to 2015.

``` r
plot(txhousing_after_2010$listings, txhousing_after_2010$volume,
     ylab = "Volume of Sales ($)",
     xlab = "Number of Active Listings",
     main = "Number of Active Listings vs. Volume of Sales, 2010 - 2015")
```

![](./images/scatterplot1.png)<!-- -->

There appears to be a positive correlation between the two quantitative
variables.

``` r
cor(txhousing_after_2010$listings, txhousing_after_2010$volume, use = "complete.obs")
```

    ## [1] 0.7963502

The two quantitative variables, listings and volume, have a correlation
value of 0.7963502 rounded to 0.8 have a positive strong linear
association.

## 2. Formal Hypotheses

To determine the validity of this model, I tested the following to see
whether the slope for the linear regression model was different from
zero:

- H<sub>o</sub>: β<sub>1</sub> = 0 and there is no relationship between listings and
  volume

- H<sub>a</sub>: β<sub>1</sub> ≠ 0 and there is a significant relationship between
  listings and volume

## 3. The Model: Linear Regression

``` r
model <- lm(volume ~ listings, data = txhousing_after_2010)
```

``` r
plot(txhousing_after_2010$listings, txhousing_after_2010$volume,
     ylab = "Volume of Sales ($)",
     xlab = "Number of Active Listings",
     main = "Number of Active Listings vs. Volume of Sales, 2010 - 2015")

# add regression line, "line of best fit"
abline(model, col="red", lwd=2)
```

![](./images/scatterplot2.png)<!-- -->

However, is the linear model a good fit for these two variables and how
well does it predict the outcome?

``` r
model
```

    ## 
    ## Call:
    ## lm(formula = volume ~ listings, data = txhousing_after_2010)
    ## 
    ## Coefficients:
    ## (Intercept)     listings  
    ##    -1163195        46987

$$
Volume = -1,163,195 + 46,987*Listings
$$

The linear model is predicting the volume of sales given the number of
active listings.

The y-intercept of the linear model is -1,163,195. It mathematically
represents the predicted volume of sales at zero listings, however, it
is outside the observable range of the data (extrapolation) and not a
possible value that we would realistically encounter. Therefore, we will
focus on the slope for the remainder of this analysis.

The slope is 46,987, which means that for every single additional
listing added to the market, the total volume of sales is predicted to
increase by \$46,987, on average.

``` r
summary(model)
```

    ## 
    ## Call:
    ## lm(formula = volume ~ listings, data = txhousing_after_2010)
    ## 
    ## Residuals:
    ##        Min         1Q     Median         3Q        Max 
    ## -1.054e+09 -4.288e+07 -1.949e+07 -2.413e+05  1.448e+09 
    ## 
    ## Coefficients:
    ##               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) -1163195.4  3591551.0  -0.324    0.746    
    ## listings       46987.2      647.9  72.523   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 174100000 on 3034 degrees of freedom
    ##   (46 observations deleted due to missingness)
    ## Multiple R-squared:  0.6342, Adjusted R-squared:  0.6341 
    ## F-statistic:  5260 on 1 and 3034 DF,  p-value: < 2.2e-16

**The Reliability of the Model:**

Looking at the R^2 value, 0.6342, we can say that 63.4% of the
variation in the total sales volume is explained by the number of
listings. However, there is around 36.6% of the story still missing that
likely comes from other factors that this model did not include.

**The Statistical Significance:**

Since the P-value is less than 2e^-16, and it is less than 0.05 and
the slope was not equal to 0, we reject the Null Hypothesis. We believe
we have convincing evidence that there is a statistical significance
with the relationship between the listings and volume variables in the
Texas housing market.

## 4. Model Accuracy (Residuals Plot)

``` r
plot(model, which = 1)
```

![](./images/residuals.png)<!-- -->

This model appears to have a fan shape indicating it is not very
accurate for the Texas market as a whole. It is more accurate for small
to mid-size markets but loses precision as the total number of listings
increase (or the city size increases like Houston or Dallas that have
higher numbers of active listings, as we saw in our barplot in section
1).

## Conclusion:

Between 2010 and 2015, the Texas housing market followed a predictable
pattern: for every new listing added, the market generated on average
approximately **\$46,987** in total volume. While this model is highly
significant (p < 2e^-16), the R^2 of **0.634** suggests that
inventory alone doesn’t tell the whole story. Investors should look at
listings as a ‘baseline’ indicator, but must supplement this with local
economic or additional housing data to account for the remaining 36.6%
of market volatility.

------------------------------------------------------------------------

## Future Project Expansion:

Future iterations of this model could use **Multiple Regression** to see
if adding another variable like, ‘Year’ improves the R^2 above 0.634.

