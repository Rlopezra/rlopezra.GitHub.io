---
layout: single
classes: wide
title: "An Alternative to Linear and Logistic Regression - Using Ordinal"
date: 2021-01-21
tags:
  - R
  - Bayes
  - Analysis
  - Probability
comments: true
---


Logistic Regression in Survey Analysis
================

In the User Experience (UX) field, most quantitative data that you deal
with is generated from surveys. A question that is in in almost every
survey that’s also heavily scrutinized is the question:

How satisfied are you?

In fact, the majority of UX surveys are created to understand why an
individual is satisfied or dissatisfied with a service or product. When
surveys have specific questions and hypotheses regarding user behavior,
they are a powerful tool that provide meaningful inferences that
influence workstreams, product design and development. In fact, a change
in how users respond to the question can change product direction.

So how is user satisfaction analyzed? User satisfaction is usually
measured on a 5-point Likert scale that ranges from “Very Dissatisfied”
to “Very Satisfied” and it is either treated as a continuous numerical
variable or a categorical variable. When satisfaction is treated a
numerical value, the average is the Key Performance Indicator (KPI) and
a Linear Regression is used to show the relationship between the
dependent and explanatory variables. When it is treated as categorical,
“Very Satisfied” and “Somewhat Satisfied” are lumped together as
“Satisfied” and the KPI is the proportion of “Satisfied” respondents.
In this case, the dependent variable is a binary outcome (were they
satisfied or not) and a Logistic Regression is used to analyze the
relationship between the dependent and explanatory variables.

## What is Ordinal Logistic Regression?

As the name of the suggests, what sets Ordinal Logistic Regression (OLR)
apart from Linear and Logistic Regression is that the dependent variable
is treated as ordinal, a scale where there is a rank/order for the
values. For example, in our case “Somewhat Dissatisfied” is greater than
“Very Dissatisfied” and “Very Satisfied” is the greatest value. It is
important note that a distinction between numerical and ordinal value,
is the intervals between the values of the numerical variable are
equally spaced while we can’t assume the same for ordinal variables. For
a numerical value, the distance between “Very Dissatisfied” to “Somewhat
Dissatisfied” is the same between “Somewhat Satisfied” to “Very
Satisfied.” For an ordinal variable, the distance between “Very
Dissatisfied” to “Somewhat Dissatisfied” could be larger than the
difference between “Somewhat Satisfied” to “Very Satisfied.”
Additionally, these differences can be compounded since they can very by
individual.

A cool thing about OLR and what makes it different is that the model
assumes that the dependent variable comes from a latent continuous space
(User Satisfaction) and it find K-1 thresholds to partition the
probability space (K being the number of values in the dependent
variable). In simpler words, an OLR tells us the probability of each
response being selected

## Performing an OLR

For the example, I’m going to use the [Stack Overflow’s 2020 Developer
Survey](https://www.kaggle.com/aitzaz/stack-overflow-developer-survey-2020)
and run a Bayesian Regression using BRMS with uninformed priors. The
reason I’m taking a Bayesian approach is because the results are easier
to interpret and I’m using uniformed priors since I do not
have any prior knowledge or assumptions about the population surveyed.

``` r
library(tidyverse)
library(brms)
library(scico)

#Setting color pallete
sl <- scico(palette = "lajolla", n = 9)

#Reading the data
survey_data <-read.csv('survey_results_public.csv')

sat_ord <-c("Very dissatisfied", "Slightly dissatisfied", "Neither satisfied nor dissatisfied", 
            "Slightly satisfied","Very satisfied")

#Setting the factor levels for the dependent variable, creating a numeric version and recoding Job Seek for easier readability
cleaned_data <- survey_data %>%
  drop_na() %>%
  filter(JobSeek %in% c("I am not interested in new job opportunities", "I am actively looking for a job" )) %>%
  mutate(JobSat = factor(JobSat, levels = sat_ord),
         JobSatNum = as.numeric(JobSat),
        JobSeek = recode(JobSeek, "I am not interested in new job opportunities" = "Not Seeking", 
         "I am actively looking for a job" = "Seeking"))

cumul_model <-
   brm(JobSatNum ~ 1 + JobSeek + Age , 
       data=cleaned_data, 
       family=cumulative("probit"),
       chains = 4, cores = 4,)
```

``` r
summary(cumul_model)
```

    ##  Family: cumulative 
    ##   Links: mu = probit; disc = identity 
    ## Formula: JobSatNum ~ 1 + JobSeek + Age 
    ##    Data: cleaned_data (Number of observations: 1596) 
    ## Samples: 4 chains, each with iter = 2000; warmup = 1000; thin = 1;
    ##          total post-warmup samples = 4000
    ## 
    ## Population-Level Effects: 
    ##                Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
    ## Intercept[1]      -2.12      0.14    -2.38    -1.85 1.00     4644     3195
    ## Intercept[2]      -1.52      0.13    -1.78    -1.26 1.00     5306     3271
    ## Intercept[3]      -1.24      0.13    -1.50    -0.99 1.00     5717     3072
    ## Intercept[4]      -0.51      0.13    -0.76    -0.26 1.00     5978     3085
    ## JobSeekSeeking    -1.40      0.07    -1.53    -1.28 1.00     3630     2496
    ## Age               -0.00      0.00    -0.01     0.00 1.00     6377     3003
    ## 
    ## Family Specific Parameters: 
    ##      Estimate Est.Error l-95% CI u-95% CI Rhat Bulk_ESS Tail_ESS
    ## disc     1.00      0.00     1.00     1.00 1.00     4000     4000
    ## 
    ## Samples were drawn using sampling(NUTS). For each parameter, Bulk_ESS
    ## and Tail_ESS are effective sample size measures, and Rhat is the potential
    ## scale reduction factor on split chains (at convergence, Rhat = 1).

## Interpreting the results

![Imgur Image](https://i.imgur.com/XP03SrF.jpg)

![Imgur Image](https://imgur.com/m9sMRTT.jpg)

## When to Use OLR

## Appendix

### Resources

### Standard Normal Plot Code

``` r
library(scico)

sl <- scico(palette = "lajolla", n = 9)


tibble(x = seq(from = -3.5, to = 3.5, by = .01)) %>%
  mutate(d = dnorm(x)) %>% 
  
  ggplot(aes(x = x, ymin = 0, ymax = d)) +
  geom_ribbon(fill = sl[3]) +
  geom_vline(xintercept = fixef(cumul_model)[1:4, 1], color = "white", linetype = 3) +
  scale_x_continuous(NULL, breaks = fixef(cumul_model)[1:4, 1],
                     labels = parse(text = str_c("theta[", 1:4, "]"))) +
  scale_y_continuous(NULL, breaks = NULL, expand = expansion(mult = c(0, 0.05))) +
  ggtitle("Standard normal distribution underlying the ordinal Y data:",
          subtitle = "The dashed vertical lines mark the posterior means for the thresholds.") +
  coord_cartesian(xlim = c(-3, 3)) + 
  theme(panel.background = element_rect(fill='white', colour='white'))
```

### Predicted Outcomes Plot

``` r
library(cowplot)

n_iter <- 100


JobSeek = c( "Seeking", "Not Seeking" )
Age = c(33,33)

nd <-  tibble(JobSeek, Age)


f <-
  fitted(cumul_model,
         newdata = nd,
         summary = F,
         nsamples = n_iter)


p1 <- f[, , 1] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")  +
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = 'Very Dissatisfied', y = ' ')

p2 <- f[, , 2] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")+
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = 'Somewhat Dissatisfied', y = ' ')

p3 <- f[, , 3] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")+
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = 'Neither Satisfied nor Dissatisfied', y = ' ')

p4 <- f[, , 4] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")+
  xlim(0,1) +
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = 'Somewhat Satisfied', y = ' ')

p5 <- f[, , 5] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")+
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = 'Very Satisfied', y = ' ')

plot_grid(p1, p2, p3, p4, p5, labels = c('', '',' ', '',''), label_size = 12)
```
