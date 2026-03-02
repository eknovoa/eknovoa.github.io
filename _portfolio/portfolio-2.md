
---
title: "Comparative Analysis of Texas Housing Markets (2010 - 2015)"
excerpt: "<img src='https://images.unsplash.com/photo-1609704001758-682c89a4d8b3?q=80&w=2531&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'>"
collection: portfolio
---

*by: Erin Novoa*

You can also view the pdf of this project on my [GitHub](https://github.com/eknovoa/analysis_projects_in_R/blob/main/MarketComparison.pdf).



**Objective**

This comparative analysis examined whether significant differences exist
in median housing prices among the four largest Texas cities — **Austin,
Dallas, Houston, and San Antonio**—using the `txhousing` dataset from
R’s `ggplot2` package. The study focused on the five-year period from
2010 to 2015 to capture market trends.

**Methodology**

- **Statistical Model:** A One-Way Analysis of Variance (ANOVA) was
  utilized to test the null hypothesis that mean housing prices are
  equal across all four cities.

- **Post-Hoc Analysis:** Following a significant ANOVA result, a **Tukey
  Honestly Significant Difference (HSD)** test was performed to identify
  specific pairwise differences between cities.

- **Data Integrity:** The analysis utilized a balanced sample (n = 67
  per city) and verified assumptions of normality and variance.

**Key Findings**

- **Significant Market Variance:** The ANOVA confirmed that city
  location has a statistically significant impact on housing prices
  (F(3, 264) = 61.1, p < .001), rejecting the null hypothesis that
  these markets perform identically.

- **Austin as the Market Leader:** Austin is statistically the most
  expensive city in the group. Pairwise comparisons show it maintains a
  significantly higher median price point than Dallas, Houston, and San
  Antonio.

- **San Antonio as the Most Accessible:** San Antonio represents the
  most affordable major market, with median prices significantly lower
  than those in Austin and Dallas.

- **Mid-Tier Markets:** While Dallas and Houston both fall between the
  extremes of Austin and San Antonio, Dallas generally maintains a
  higher median price than Houston.

------------------------------------------------------------------------

## Setup data in R

``` r
library(ggplot2)
library(dplyr)
```


## 1. Exploration Data Analysis

``` r
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

We focus on the records from the `txhousing` dataset that were between
2010 and 2015, inclusive and where the city column had a value equal to
`Houston`, `Dallas`, `Austin`, or `San Antonio`.

``` r
txhousing.after.2010 <- txhousing[txhousing$year >= 2010, ]
txhousing.major.cities <- txhousing.after.2010[txhousing.after.2010$city %in% c('Houston', 'Dallas', 'Austin', 'San Antonio'), ]
txhousing.major.cities
```

    ## # A tibble: 268 × 9
    ##    city    year month sales    volume median listings inventory  date
    ##    <chr>  <int> <int> <dbl>     <dbl>  <dbl>    <dbl>     <dbl> <dbl>
    ##  1 Austin  2010     1   985 230799130 175300     9931       5.7 2010 
    ##  2 Austin  2010     2  1249 293085886 182600    10825       6.2 2010.
    ##  3 Austin  2010     3  1987 461231875 179400    11846       6.7 2010.
    ##  4 Austin  2010     4  2230 513122847 185100    12270       6.7 2010.
    ##  5 Austin  2010     5  2286 543258166 188600    12786       6.9 2010.
    ##  6 Austin  2010     6  2190 584250558 200500    13353       7.2 2010.
    ##  7 Austin  2010     7  1640 453983948 213600    13336       7.4 2010.
    ##  8 Austin  2010     8  1675 432655897 195200    12575       7.1 2011.
    ##  9 Austin  2010     9  1406 342118926 190500    11834       6.8 2011.
    ## 10 Austin  2010    10  1336 342862397 193300    11003       6.5 2011.
    ## # ℹ 258 more rows

Let’s check the sample size across the 4 groups (cities) to make sure it
is an even distribution.

``` r
# Check how many data points are in each city
table(txhousing.major.cities$city)
```

    ## 
    ##      Austin      Dallas     Houston San Antonio 
    ##          67          67          67          67

We can also assume that that the groups are independent of each other.
For example, a housing price in Austin is independent of a housing price
in Dallas.

Using a boxplot below, we will check the distribution of our data to see
if we have any outliers. If we do, this can be problematic in an ANOVA
test since the test relies on the mean and standard deviation which are
both highly sensitive to extreme values.

``` r
boxplot(txhousing.major.cities$median, horizontal = TRUE)
```

![](MarketComparison_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
txhousing.major.cities[txhousing.major.cities$median > 260000, ]
```

    ## # A tibble: 4 × 9
    ##   city    year month sales     volume median listings inventory  date
    ##   <chr>  <int> <int> <dbl>      <dbl>  <dbl>    <dbl>     <dbl> <dbl>
    ## 1 Austin  2015     4  2801  931744729 270300     6560       2.5 2015.
    ## 2 Austin  2015     5  2999 1026501450 271200     7009       2.7 2015.
    ## 3 Austin  2015     6  3301 1086689918 270200     7419       2.8 2015.
    ## 4 Austin  2015     7  3466 1150381553 264600     7913       3   2016.

We do see that we have some high outliers. However, after further
investigation we can see that there are only 4 median price values,
located in Austin, and they are not that massive so we can continue with
the ANOVA test.

## 2. Formal Hypotheses

H<sub>o</sub>: $\mu$<sub>Houston</sub> = $\mu$<sub>Dallas</sub> = $\mu$<sub>Austin</sub> = $\mu$<sub>San Antonio</sub> (the four cities are equal in terms of mean price)

H<sub>a</sub>: at least one mean price for one of the cities is different than
the other cities

## 3. Performing One-Way ANOVA

We conduct a one-way ANOVA to examine if the mean home price (`median`
column) differs across cities (`city` column).

``` r
anova.model <- aov(median ~ city, data = txhousing.major.cities)
summary(anova.model)
```

    ##              Df    Sum Sq   Mean Sq F value Pr(>F)    
    ## city          3 9.359e+10 3.120e+10    61.1 <2e-16 ***
    ## Residuals   264 1.348e+11 5.106e+08                   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

- The p-value is `<2e-16`, which is effectively **0**. Since it is less
  than 0.05, we can reject the null hypothesis.

- Therefore, there is a statistically significant difference in the
  median housing prices between at least two of the four cities.

- Also, the F-value is 61.1, which is quite high. This means the
  variance *between* the cities is much higher than the variance
  *within* the cities.

## 4. ANOVA Model Quality Check

We will use the Q-Q plot (Quantile-Quantile plot) to compare the
distribution of our residuals against a perfect theoretical normal
distribution. This will allow us to do a quality control check for our
ANOVA model.

``` r
# look at the Q-Q plot to ensure normality
# Before finalizing your report, always check the normality of the residuals.
plot(anova.model, which = 2)
```

![](MarketComparison_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

The points seem to fall roughly along a straight, diagonal line which
indicates that the residuals are normally distributed which is a core
assumption of ANOVA. We do see that some deviation from the line at the
ends which is likely due to the outliers that we noted in the initial
Exploratory Data Analysis. Overall, the majority of residuals on this
Q-Q plot follow the diagonal, the assumption of normality is
sufficiently met, and we can trust our p-value.

## 5. Post-Hoc Tests with Tukey HSD

Since we know there is a difference in at least 2 out of the 4 cities,
the next step is to confirm the `TukeyHSD()` output to state which
cities are different.

``` r
tukey.results <- TukeyHSD(anova.model)
tukey.results
```

    ##   Tukey multiple comparisons of means
    ##     95% family-wise confidence level
    ## 
    ## Fit: aov(formula = median ~ city, data = txhousing.major.cities)
    ## 
    ## $city
    ##                           diff       lwr        upr     p adj
    ## Dallas-Austin       -32349.254 -42442.65 -22255.861 0.0000000
    ## Houston-Austin      -41007.463 -51100.86 -30914.070 0.0000000
    ## San Antonio-Austin  -49285.075 -59378.47 -39191.681 0.0000000
    ## Houston-Dallas       -8658.209 -18751.60   1435.184 0.1210571
    ## San Antonio-Dallas  -16935.821 -27029.21  -6842.428 0.0001199
    ## San Antonio-Houston  -8277.612 -18371.01   1815.781 0.1493015

``` r
par(mar = c(5, 6, 4, 2) + 3.0) 

plot(tukey.results, las = 1, col = "blue")
```

![](MarketComparison_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

The vertical axes of this plot lists the city pairs (e.g.,
`Austin-Dallas`), and the horizontal axis shows the difference in median
prices. If the blue line for a pair crosses the vertical dashed line at
0, there is no statistically significant difference between the pair of
cities. If the position of the bar is to the left of 0, the first city
in the pair is cheaper than the second. Otherwise if the bar is to the
right of 0, the first city in the pair is more expensive than the
second.

Let’s discuss each city pair:

- **Dallas - Austin:** Dallas is significantly cheaper than Austin.

- **Houston - Austin:** Houston is significantly cheaper than Austin.

- **San Antonio - Austin:** San Antonio is significantly cheaper than
  Austin.

- **Houston - Dallas:** There is no statistical significant difference
  between the two cities.

- **San Antonio - Dallas:** San Antonio is significantly cheaper than
  Dallas.

- **San Antonio - Houston:** There is no statistical significant
  difference between the two cities.

## 6. Conclusion

Looking at the Tukey HSD plot, since the entire confidence interval for
those specific pairs is located to the left of the 0 line, we can say
with high confidence that for the `txhousing` market dataset from 2010 -
2015, **Austin is the most expensive city** among this group, and **San
Antonio is generally the least expensive**, with Dallas and Houston
falling somewhere in between.

We can view this conclusion with a set of boxplots.

``` r
ggplot(txhousing.major.cities, aes(x = city, y = median, fill = city)) +
  geom_boxplot(alpha = 0.7) +
  labs(
    title = "Median Housing Prices by City (2010 - 2015)",
    x = "City",
    y = "Median House Price ($)"
  ) +
  theme_bw() +
  scale_fill_brewer(palette = "Dark2")
```

![](MarketComparison_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->
