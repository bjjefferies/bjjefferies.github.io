---
layout: post
title: "Happiness and Income, Australia vs. Mexico"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
    toc: yes
---

-   [Project Overview](#project-overview)
-   [Summary Statistics](#summary-statistics)
    -   [Income Levels](#income-levels)
    -   [Happiness Levels](#happiness-levels)
-   [Correlations](#correlations)
    -   [Mexico](#mexico)
    -   [Australia](#australia)
    -   [Income Effect Comparison](#income-effect-comparison)
-   [Summary and Discussion](#summary-and-discussion)
-   [Footnotes](#footnotes)

## Project Overview

This report draws on Data from the [World Values Survey, Wave
7](https://www.worldvaluessurvey.org/wvs.jsp) which was completed
between 2017 and 2022. The nearly 300 question questionaire was
completed by hundreds of thousands of respondents from 66 countries and
captures a wide array of variables.

The goal of this report is to analyze the connection between income and
happiness. Both of these indicators were measured with self-reported
questions.  
For happiness level, respondents rated themselves on a 4 point
scale<sup>1</sup>:

-   1 - “not at all happy”
-   2 - “not very happy”
-   3 - “rather happy”
-   4 - “very happy”

For the income level, respondents were asked to place themselves
relative to the income groups in their country.

-   1 - Lowest Group
-   10 - Highest Group

<center>
*Pre-processing Happiness Data code*
</center>

``` r
dat <- read.csv("WVS_Cross-National_Wave_7_csv_v6_0.csv")

nrow(dat)  # check total rows 97,220
unique(dat$Q46)  #look at unique values for Q46

nrow(dat) == sum(dat$Q46 == 1) + sum(dat$Q46 == 2) + sum(dat$Q46 == 3) + sum(dat$Q46 == 4) + sum(dat$Q46 == -1) + sum(dat$Q46 == -2) + sum(dat$Q46 == -5)  # confirms there are no missing data rows

dat$Happy <- ifelse(dat$Q46 >= 0, dat$Q46, NA)   #replace non-response with NA

# Change order of data 1 - 4, etc.
# first confirm number of 1's and 4's
sum(dat$Happy == 1, na.rm = T)   # 30093
sum(dat$Happy == 4, na.rm = T)   # 2104

dat <- dat %>% 
  mutate(Happy = case_when(Happy == 1 ~ 4,
                          Happy == 2 ~ 3,
                          Happy == 3 ~ 2,
                          Happy == 4 ~ 1))

# confirm switch happened properly
sum(dat$Happy == 1, na.rm = T)   # 2104
sum(dat$Happy == 4, na.rm = T)   # 30093


###  Q288 - Income Levels

## Convert non-response to NA Q288

dat <- dat %>% 
  mutate(Q288 = ifelse(Q288 >= 0, Q288, NA))

## Mutate to variable Income

dat <- dat %>% 
  mutate(Income = Q288)
```

<br>

## Summary Statistics

### Income Levels

Comparing Mexico and Australia means comparing two very different
countries in economic terms. According to the [OECD Better Life
Index](https://www.oecdbetterlifeindex.org/topics/income/), Australia’s
mean household wealth ranks #5 globally compared to Mexico at place #34.

The survey asks respondents to place themselves in the income group,
relative to their country, that they see themselves in. The chart below
shows the distribution of self-reported income levels for each country,
which does reflect what we might imagine when comparing a richer country
with a poorer one: in Mexico the majority of respondents place
themselves under level 5, while in Australia the distribution looks
“normal” around the middle income group.

``` r
#subset data to only Mexico and Australia
sub.mx.aus <- dat[ dat$B_COUNTRY_ALPHA %in% c("MEX", "AUS"), ]

ggplot(data = sub.mx.aus, 
       aes(x = Income, fill = factor(B_COUNTRY_ALPHA), 
           by = factor(B_COUNTRY_ALPHA),
           y = after_stat(prop))) + 
  geom_bar( position = "dodge") +
  labs(title = "Income Level Proportions for Australia and Mexico",
       subtitle = "1 - \"Lowest Group\"  10 - \"Highest Group\"",
       y = "proportion of respondents") +
  theme(legend.title = element_blank()) +
  scale_fill_brewer(palette = "Dark2") +
  scale_x_continuous(name = "Self-Reported Income Level",
                     breaks = c(1:10))
```

![]({{site.url}}/assets/img/income-levels-1.png)

<br>

### Happiness Levels

On the happiness scale, it can be seen below that a similar proportion
in both Mexico and Australia reported being “rather happy” or “very
happy”. However, between the two groups a much greater proportion of
Mexican respondents reported being very happy. This leads to a higher
group mean for Mexico (3.5) than Australia (3.2).

``` r
#3.5
mx.avg.happy <- sub.mx.aus %>% 
  filter(B_COUNTRY_ALPHA == "MEX") %>% 
  pull(Happy) %>% 
  mean(, na.rm = T)

#3.2
aus.avg.happy <- sub.mx.aus %>% 
  filter(B_COUNTRY_ALPHA == "AUS") %>% 
  pull(Happy) %>% 
  mean(, na.rm = T)


ggplot(data = sub.mx.aus, 
       aes(x = Happy, fill = factor(B_COUNTRY_ALPHA), 
           by = factor(B_COUNTRY_ALPHA),
           y = after_stat(prop))) + 
  geom_bar( position = "dodge") +
  labs(title = "Happiness Level Proportions for Australia and Mexico",
       subtitle = "1 = \"not\" 2 = \"not very\" 3 = \"rather\" 4 = \"very\"",
       y = "proportion of respondents") +
  theme(legend.title = element_blank()) +
  geom_vline(aes(xintercept = mx.avg.happy,
             color = "Mex.Mean"),
             lty = "dashed") +
  geom_vline(aes(xintercept = aus.avg.happy,
             color = "Aus.Mean"),
             lty = "dashed") +
  scale_fill_brewer(palette = "Dark2") +
  scale_color_manual(name = "Statistics", 
                     values = c(Mex.Mean = "#D95F02", Aus.Mean = "#1B9E77")) +
  scale_x_continuous(name = "Self-Reported Happiness Level")
```

![]({{site.url}}/assets/img/happiness-levels-1.png)

<br>

**T-Test:** It turns out this is a statistically significant difference
in group means (confidence level of 99%), indicating that there is a
difference in self-perceived happiness levels between Mexico and
Australia.

``` r
t.test(Happy ~ B_COUNTRY_ALPHA,
       data = sub.mx.aus)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  Happy by B_COUNTRY_ALPHA
    ## t = -13.096, df = 3497.3, p-value < 2.2e-16
    ## alternative hypothesis: true difference in means between group AUS and group MEX is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.3221971 -0.2382844
    ## sample estimates:
    ## mean in group AUS mean in group MEX 
    ##          3.217175          3.497415

<br>

## Correlations

Running a linear regression for both countries shows a statistically
significant relationship between an increase in Income and an increase
in happiness. The model run describes the following mathematical
relationship:

<center>
*H**a**p**p**i**n**e**s**s* = *b*<sub>0</sub> + *b*<sub>1</sub> ⋅ *I**n**c**o**m**e*
</center>

<br>

### Mexico

In Mexico, the result is a statistically significant (p \< 0.01) effect
of .02. Meaning for every increase of 1 in a respondents perceived
income level (10% change), we expect a .02 increase in reported
happiness (0.5% change). This is not a large effect.

``` r
sub.mex <- dat[dat$B_COUNTRY_ALPHA == "MEX", ]

fit.happy.mx <- lm(Happy ~ Income, sub.mex)

summary(fit.happy.mx)
```

    ## 
    ## Call:
    ## lm(formula = Happy ~ Income, data = sub.mex)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.5707 -0.4911  0.4492  0.5089  0.5686 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 3.411533   0.032442 105.160  < 2e-16 ***
    ## Income      0.019889   0.006679   2.978  0.00294 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.6617 on 1717 degrees of freedom
    ##   (22 observations deleted due to missingness)
    ## Multiple R-squared:  0.005138,   Adjusted R-squared:  0.004558 
    ## F-statistic: 8.867 on 1 and 1717 DF,  p-value: 0.002944

<br>

The graph below helps us visualize the relationship over the
distribution of happiness responses by income level.

``` r
ggplot(data = sub.mex, aes(x = Income, y = Happy)) +
  geom_jitter(aes(alpha = 0.9), colour = "#D95F02") +
  geom_abline(slope = 0.019889,
              intercept = 3.411533,
              color = "red",
              linewidth = 1) +
  scale_x_continuous(name = "Income",
                     breaks = seq(1,10)) +
  scale_y_continuous(name = "Happiness on 4 points scale") +
  ggtitle(label = "Happiness Levels by Income Group in Mexico",
          subtitle = "Red line shows statistical relationship") +
  theme(legend.position = "none")
```

![]({{site.url}}/assets/img/happy-by-income-mx-1.png)

<br>

### Australia

In Australia, increases in Income also have a statistically significant
effect on Happiness (p \< 0.001). The effect is also positive at 0.05,
over twice the effect that Income has in Mexico.

``` r
sub.aus <- dat[dat$B_COUNTRY_ALPHA == "AUS", ]

fit.happy.aus <- lm(Happy ~ Income, data = sub.aus)

summary(fit.happy.aus)
```

    ## 
    ## Call:
    ## lm(formula = Happy ~ Income, data = sub.aus)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.3133 -0.2751 -0.1604  0.6867  0.9925 
    ## 
    ## Coefficients:
    ##             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  2.95648    0.03759  78.650  < 2e-16 ***
    ## Income       0.05098    0.00678   7.519 8.77e-14 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.5971 on 1742 degrees of freedom
    ##   (69 observations deleted due to missingness)
    ## Multiple R-squared:  0.03144,    Adjusted R-squared:  0.03088 
    ## F-statistic: 56.54 on 1 and 1742 DF,  p-value: 8.765e-14

<br>

The graph of the effect in Australia is presented below.

``` r
ggplot(data = sub.aus, aes(x = Income, y = Happy)) +
  geom_jitter(aes(alpha = 0.5), colour = "#1B9E77") +
  geom_abline(slope = fit.happy.aus$coefficients[2],
              intercept = fit.happy.aus$coefficients[1],
              color = "red",
              linewidth = 1) +
  scale_x_continuous(name = "Income",
                     breaks = seq(1,10)) +
  scale_y_continuous(name = "Happiness on 4 points scale") +
  ggtitle(label = "Happiness Levels by Income Group in Australia",
          subtitle = "Red line shows statistical relationship") +
  theme(legend.position = "none")
```

![]({{site.url}}/assets/img/happy-by-income-aus-1.png)

<br>

### Income Effect Comparison

In order to determine if indeed there is a statistically significant
difference between the effect Income has on happiness in Australia
compared to Mexico, we can run a multiple regression analysis that
incorporates an “interaction term” between Income and Country. If that
interaction term’s coefficient is large enough and statistically
significant, we can say that there is a difference in the effect income
has on happiness depending on what country the respondent lives in.

The mathematical model we now look at takes Australia as the baseline
and controls for Mexico as a variable:

<center>
*H**a**p**p**i**n**e**s**s* = *b*<sub>0</sub> + *b*<sub>1</sub> ⋅ *I**n**c**o**m**e* + *b*<sub>2</sub> ⋅ *M**e**x**i**c**o* + *b*<sub>3</sub> ⋅ *M**e**x* : *I**n**c**o**m**e*
</center>

**Results:** As can be read below, the interaction term’s coefficient
*b*<sub>3</sub> is -0.03 and is statistically significant (p \< 0.01).
We can therefore conclude that Income has *less* of an effect on
happiness in Mexico than it does in Australia.

``` r
lm.interact <- lm(Happy ~ Income * B_COUNTRY_ALPHA, data = sub.mx.aus)

summary(lm.interact)
```

    ## 
    ## Call:
    ## lm(formula = Happy ~ Income * B_COUNTRY_ALPHA, data = sub.mx.aus)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -2.5707 -0.4153 -0.1094  0.5288  0.9925 
    ## 
    ## Coefficients:
    ##                            Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                2.956481   0.039662  74.541  < 2e-16 ***
    ## Income                     0.050978   0.007153   7.126 1.25e-12 ***
    ## B_COUNTRY_ALPHAMEX         0.455052   0.050270   9.052  < 2e-16 ***
    ## Income:B_COUNTRY_ALPHAMEX -0.031089   0.009571  -3.248  0.00117 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 0.63 on 3459 degrees of freedom
    ##   (91 observations deleted due to missingness)
    ## Multiple R-squared:  0.06205,    Adjusted R-squared:  0.06124 
    ## F-statistic: 76.28 on 3 and 3459 DF,  p-value: < 2.2e-16

<br>

## Summary and Discussion

The results of this investigation lead one to believe that even though
people in Mexico see themselves as poorer on average, in line with the
statistical reality of the Mexican economy, that does not seem to have
much of an effect on their perceived level of happiness. Even though
Australia is a much wealthier country overall, that wealth does not
translate into greater levels of happiness overall.

As a matter of policy, it seems that increasing personal income in
Mexico would have only a very marginal effect on the population’s
happiness. The same can be said of Australia in real terms, though in
comparison to Mexico the effect would be much larger.

These findings are simple and should not be mistakenly taken as an
argument for any causation. There are potentially numerous confounding
factors, such as differences in culture and language (for instance, how
was the #3 happiness level “rather happy” translated into Spanish).
There are also many hundreds of variables that were not accounted for in
this statistical analysis.

Interestingly, these findings do fit into a common narrative that people
living in poor countries are surprisingly happy and those that live in
wealthier countries are less happy, perhaps because of the human
tendency to compare ourselves to the perceived wealth of others.

<br>

## Footnotes

1.  The original question in the survey ranked the happiness questions
    in inverse order (ie. “very happy” was 1, “not at all happy” was 4).
    Code block 2 shows the process by which I recoded this variable.
