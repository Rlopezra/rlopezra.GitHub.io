---
layout: single
classes: wide
title: "An Alternative to Linear and Logistic Regression for Survey Analysis - Using Ordinal Logistic Regression"
date: 2021-01-21
tags:
  - R
  - Bayes
  - Analysis
  - Probability
  - Regression
comments: true
---


Regression in Survey Analysis
================

In the User Experience (UX) field, most quantitative data that you deal
with is generated from surveys. A question that is in almost every
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
Additionally, these differences can be compounded since they can vary by
individual.

A cool thing about OLR and what makes it different is that the model
assumes that the dependent variable (User Satisfaction) comes from a 
latent continuous space (distribution)
and it finds K-1 thresholds to partition the
probability space (K being the number of values in the dependent
variable). In simpler words, an OLR tells us the probability of
respondent answering to each value!

## Performing an OLR

For the example, I’m using [Stack Overflow’s 2020 Developer
Survey](https://www.kaggle.com/aitzaz/stack-overflow-developer-survey-2020)
and running a Bayesian Regression using BRMS package with uninformed priors. The
reason I’m taking a Bayesian approach is because the results are easier
to interpret. Also, I’m using uniformed priors since I do not
have any prior knowledge or assumptions about the population surveyed. The
model is a simple one, the explanatory variable is Job Satisfaction and the
explanatory variables are Age (in years) and Job Seek (are they
looking for a new job). 

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

We ran our model and we have our results, but how do we interpret the results? For
simplicity, let's focus on the interpreting the intercepts. The intercepts can be 
interpreted as the thresholds that separate the response values under a normal 
curve, so Intercept[1] is the threshold that separates “Very Dissatisfied” 
and “Somewhat Dissatisfied." Since I used the probit link function, the intercepts are
Z-scores and we can convert them to probabilities using pnorm:

``` r
pnorm(-2.12) 
```
    ## 0.01700302

On a normal distribution centered around 0, 1.7% of respondents are expected to respond "Very Dissatisfied."
    
![Imgur Image](https://i.imgur.com/XP03SrF.jpg)

It's important to note that in OLR when using the probit link function, the distribution of the dependent variable can differ by respondent but the
thresholds remain the same. Lets see what happens when the thresholds are illustrated above a normal distribution
centered around -1.41 :

![Imgur Image](https://i.imgur.com/J0wzoWP.jpg)

And if we use the pnorm function:

``` r
pnorm(-2.12, -1.41) 
```
    ## 0.2388521

On a normal distribution centered around -1.41, 23.89% of respondents are expected to respond "Very Dissatisfied!"

Why did I choose to center the distribution around -1.41? Because that is the estimated effect size for respondents who are seeking new job opportunities!


Let's extend this analysis by taking advantage of the fact that Bayesian Analysis deals with posterior probabilities, the values/data that 
we believe is more likely to occur. Lets predict the outcomes  of two 33-years old programmers that responded 
differently to the question "Which of the following best describes your current  job-seeking status" and simulated 
the results 100 times for each programmer. The great thing about this process is that it is easier
to interpret the results. Instead of making an inference about the population parameter like the Frequentist approach, the Bayesian
approach tells us what are the most probable outcomes based on our prior knowledge and the data that we saw. 

Examining the graph below, for 33-years old programmers that responded that they are seeking a new job, the percent of them responding 
"Very Dissatisfied" ranges between 25% and 32%, while 4% of "Not Seeking" respondents respond "Very Dissatisfied." For "Somewhat Dissatisfied,"
between 3-5% of "Not Seeking" and 22-30% of "Seeking" would respond as "Somewhat Dissatisfied." The OLR model already tells us that ~50% of "Seeking"
and ~8% of "Not Seeking" respondents are at least "Somewhat Dissatisfied" with their job. The graph illustrates that as we go higher in our ordinal value (satisfaction), the proportion of respondents increased for "Not Seeking" while it decreased for "Seeking" respondents.



![Imgur Image](https://imgur.com/m9sMRTT.jpg)

## When to Use OLR

I’m a firm believer in tailoring your analysis and reports to your target audience. When deciding which methodology to 
choose, it is important to consider your team’s data culture. If the results are going to be shared with a wide 
audience that doesn’t have the contextual information of why a UX study (survey) was conducted, then I recommend choosing 
the methodology the uses the KPI that anyone reading the report can understand at a glance. My opinion is that the loss 
of model fit will be offset by the interpretability of the report. If the stakeholders must devote some time and effot to 
understand the report, then they’re very likely to ignore it.

I do think that OLR is a great methodology to use when the target audience is a small group that you can collaborate 
with in designing the study and explaining the model. In addition, I would use it when the research question is 
focused on understanding the likes and dislikes of a specific group or subset of the population.  


## Appendix

### Resources

- [Introduction to Ordinal Regression (video)](https://www.youtube.com/watch?v=jWIJ7P1G9P4)
- [Ordinal Regression - Wikipedia](https://en.wikipedia.org/wiki/Ordinal_regression)
- [Ordinal Regression Models in Psychology: A Tutorial - OSF](https://psyarxiv.com/x8swp/)
- [Examples of Using R for Modeling Ordinal Data](http://users.stat.ufl.edu/~aa/ordinal/R_examples.pdf)
- [Ordinal Regression - Michael Betancourt](https://betanalpha.github.io/assets/case_studies/ordinal_regression.html)

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

sat_ord <-c("Very dissatisfied", "Slightly dissatisfied", "Neither satisfied nor dissatisfied", 
            "Slightly satisfied","Very satisfied")

cow_plot <-  rep(NA, 5)

for(i in 1:5){
  
  graph <- f[, , i] %>%
  data.frame() %>% 
  set_names(nd %>% pull(JobSeek)) %>%
  gather(key = "JobSeek", value = "prob") %>%
  mutate(satisfaction = '1') %>%
  ggplot( aes(x=prob, color=JobSeek)) +
  geom_histogram(fill="white")+
  ylim(c(0,60)) +
  facet_grid(JobSeek ~.) +
  theme(legend.position = "none") + 
  labs(x = sat_ord[i], y = ' ')
  
  nam <- paste("p", i, sep = "")
  assign(nam, graph)
}

plot_grid(p1, p2, p3, p4, p5, labels = c('', '',' ', '',''), label_size = 12)
```
