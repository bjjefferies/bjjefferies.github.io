---
layout: post
title: "AB Testing in R"
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
    toc: yes
---

-   [AB Testing](#ab-testing)
    -   [Visualize the Experiment](#visualize-the-experiment)
    -   [Statistical Testing](#statistical-testing)
        -   [Return on Investment](#return-on-investment)
    -   [Conclusion](#conclusion)

# AB Testing

AB testing is a common tool utilized in marketing analytics in order to
determine if a campaign has been successful. AB testing is an
experiment, where some consumers are exposed to a campaign, while others
are not. If applied randomly, this experiment can tell us if the
campaign had any effect on consumer behavior and allow us to estimate
the impact of that effect in various metrics.

For this analysis I will be utilizing simulated hotel booking data. The
data is segmented by geographical zone and further by treatment/control
groups. We also have data from before the experimental online Adspend
campaign was implemented and data from during the campaign (pre/post).

``` r
# read in data
dat <- read_csv(file = "https://raw.githubusercontent.com/business-science/free_r_tips/refs/heads/master/073_ab_testing_infer/data/hotel_bookings_geo_experiment.csv")
```




## Visualize the Experiment

Below I display two charts showing booking totals for both the control
(no Adspend) and treatment (Adspend) groups. The charts show bookings
before, during and after the experimental campaign. Note the cyclical
nature of the charts, which indicates that there are certain days of the
week when advance booking of hotel rooms are higher than other days.

The charts do show what appears to be a slight bump in bookings for the
treatment group during the period the Adspend campaign was running.
However, we still need to determine if this was a statistically
significant difference, or perhaps just noise.

``` r
# create table for pre/post period aggregations
pre_post_agg_tbl <- dat %>% 
  group_by(assignment) %>% 
  summarize_by_time(
    .date_var = date,
    bookings = sum(bookings),
    cost = sum(cost),
    .by = "day"
  ) %>% 
  ungroup()

# create trial dates and pre-dates for shading
trial_dates <- c("2015-02-16", "2015-03-15") %>% as_date()
pre_dates <- c("2015-01-05", "2015-02-15") %>% as_date()

# create ggplot
pre_post_agg_tbl %>% 
  ggplot(aes(x = date, y = bookings, color = assignment)) +
  geom_path() +
  geom_smooth(se = F) +
  annotate(
    "rect",
    xmin = trial_dates[1],
    xmax = trial_dates[2],
    ymin = -Inf,
    ymax = Inf,
    fill = "blue",
    alpha = 0.2
  ) +
  scale_y_continuous(
    name = "Bookings Value ($)",
    labels = scales::comma
  ) + 
  labs(title = "Adspend Effect",
       x = "Dates (2015)") +
  facet_grid(facets = vars(assignment)) +
  theme(legend.position = "none")
```

![]({{site.url}}/assets/img/unnamed-chunk-3-1.png)



## Statistical Testing

To determine if there was a significant difference in bookings and
revenues due to the Adspend campaign, we will utilize a 2 sample T-test.
The two groups were independent of each other during the experiment, so
this test is appropriate.

The results in the below table show the following:

-   The estimated average treatment effect of the Adspend program was to
    increase bookings revenue by $96.20 per day during the experimental
    period.

-   The result is just on the cusp of the standard cutoff for
    statistical significance (p < 0.05). In this case the p-value is
    0.055. We can still maintain greater than 90% confidence that the
    effect of the Adspend campaing was not random. There is very little
    risk that the Adspend campaign would have negative effects (modeled
    by the lower confidence interval of -1.8).

``` r
# split the data into experiment and pre periods
dat_pre_dates <- dat %>% 
  filter(period == 0)

dat_experiment_dates <- dat %>% 
  filter(period == 1)
```

``` r
# run the two sided T-test looking at data during the experiment
two_sided_t_test <- t_test(
  dat_experiment_dates,
  response = bookings,
  explanatory = assignment,
  order = c("treatment", "control")
)

two_sided_t_test %>% pander::pander()
```

| statistic | t_df | p_value | alternative | estimate | lower_ci | upper_ci |
|:---------:|:----:|:-------:|:-----------:|:--------:|:--------:|:--------:|
|   1.923   | 2703 | 0.05453 |  two.sided  |  96.22   |  -1.872  |  194.3   |



### Return on Investment

Here we compare the cost of the Ad campaign with the increased revenue
it generated. Our dataset contained 50 geographic units in the treatment
group and 50 in the control group each day. To estimate the total
revenue generated we can take that figure, multiplied by our treatment
effect (increase of $96.20 in bookings per day) to arrive at a total
daily increase in revenue for the company of $4,810 per day.

The cost of the Ad campaign varied daily, but came out to $50,000 over
the course of the month long experiment, for an average cost of
$1,612.90 per day. We can measure the Return on Investment for the
Adspend campaign by dividing those daily figures. The ROI is $2.98,
meaning that for each dollar spent on Adspend we expect to see a return
of nearly $3.00 in revenue.

``` r
50 * 96.2
```

    ## [1] 4810

``` r
# show the total cost of the ad campaign in all markets
sum(dat_experiment_dates$cost) / 31
```

    ## [1] 1612.903

``` r
4810/1612
```

    ## [1] 2.983871



## Conclusion

The month long experiment with Adspend gives us a high level of
confidence to **recommend incorporating Adspend investments in all
geographic markets**. We can be more than 90% certain that spending on
Adspend increases booking revenues at a rate of $3 for each $1 spent on
Adspend services.

Of course, this experiment was conducted between February and March and
so results of Adspend spending in other months may differ. A longer term
run of the experiment, or running the same experiment in other seasons
would help in gaining greater confidence in these results. Another
future experiment could involve different levels of Ad spending and
different Ad services to compare outcomes and find the optimal levels
and type of Ads.
