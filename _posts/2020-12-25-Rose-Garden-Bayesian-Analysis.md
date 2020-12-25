---
layout: single
classes: wide
title: "Bayes Theorem and the Rose Garden Massacre"
date: 2020-12-12
tags:
  - R
  - Bayes
  - Analysis
  - Probability
comments: true
---

A Bayesian Analysis of the Rose Gardern Massacre
================

This post is a continuation of my post [Bayes Theorem and the Rose
Garden Massacre](https://rlopezra.github.io/Bayes-Theorem-Rose-Garden/)
where I will conduct a Bayesian Analysis for the event’s infection, test
positivity, sensitivity, and specificity rates. I’ll be using some of
the information from the previous post as initial values in the Bayesian
Analysis. For this post, I’m also using some of the code provided in the
book [Doing Bayesian
Analysis](https://sites.google.com/site/doingbayesiandataanalysis/) to
create some of my graphs.

## Why do Bayesian Analysis?

When it comes to dealing with samples, there is a lot of uncertainty
when estimating the statistic of interest. One of the benifits of
Bayesian Analysis is the use of prior knowledge (or assumptions) about
the distribution of our data and leveraging that knowledge to create a
“posterior” probability. The upside of using a posterior probability
is that it combines the data that we observed with our prior knowledge
to create a probability space that puts more weight in outcomes that we
believe are more likely to occur.

The graphic below summarizes the idea of Bayesian Analysis. Let’s
pretend we’re epidemiologist trying to estimate the prevalence of
COVID-19 in our populationn of interest. Our sample showed that a large
proportion of individuals tested positive positive for COVID-19 at a
much higher rate than we expected. We know that COVID-19 is highly
contagious but it has low prevalency, so we combine our prior knowledge
of COVID-19 and our sample data to create a posterior distribution
that’s a compromise of our assumptions and the data.

 ![Imgur Image](https://imgur.com/0VSXKwH.jpeg)

## What’s the Difference

The easiest way to explain the difference between a confidence interval
and credible interval is to go over how each interval is interpreted. My
professor, Dr. Eric Suess, explained it to me as “Confidence interval is
looking at the event space horizontally while a credible interval does
it vertically.” Yeah…that doesn’t make sense at first so let me
elaborate with an example.

For this example, let’s say we have a confidence and credible interval
for the percent of respondents that said they were satisfied with the
customer service at their local grocery store. For simplicity, both
intervals have an mean of 75, with 95% lower bound of 71 and upper bound
of 79.

A confidence interval is a “Frequentist” method to capture the
population’s paramater, in our example, the percent of individuals that
are satisfied with the customer service at their local grocery store. A
95% confidence interval of (71, 79) and mean of 75 can be interpreted as
the true population mean lies between 71 and 79, with 75 being the
estimated population mean. The 95% confidence means that if the
experiment/sample was repeated 100 times, it is expected that 95 (± a
few) of those experiments would contain the populations true value. It’s
important to note that we can **NOT** give a probability of confidence
that the true value lies within a range (ex: We are 95% confident that
true value is between 71 and 79). Once a confidence interval is
calculated, it either contains or does not contain the population’s true
parameter.

[Here’s a link to a shiny app that does a great job illustrating
confidence intervals.](https://istats.shinyapps.io/ExploreCoverage/)

The credible interval is a “Bayesian” approach that uses a probability
distribution (“posterior”) to calculate the values with the highest
probability of occuring (“the most credible”). For our example, a 95%
credible interval is interpreted as “the proportion that are satisfied
with the customer service at their local grocery store has a 95%
probability of falling within 71 and 79 percent.”

I hope after these two examples that my professor’s explanation makes
sense. A confidence interval looks at the event space horizontally by
having a range where the true paramater can lie in, while a credible
interval looks at the height of probability distribution returns the 
interval that is the most credible to occur.

## Creating Credible Intervals

In a recent Washing Post article [Rose Garden ceremony attendees who
tested positive for
coronavirus](https://www.washingtonpost.com/graphics/2020/politics/coronavirus-attendees-barrett-nomination-ceremony/)
the image shows approximately 300 people in attendance.

“All the values inside the HDI have higher probability density
(i.e. credibility) than any value outside the HDI. The HDI thereforre
includes the most credible values.” pg 342

    ## Loading required package: rjags

    ## Loading required package: coda

    ## Linked to JAGS 4.3.0

    ## Loaded modules: basemod,bugs

    ## 
    ## *********************************************************************
    ## Kruschke, J. K. (2015). Doing Bayesian Data Analysis, Second Edition:
    ## A Tutorial with R, JAGS, and Stan. Academic Press / Elsevier.
    ## *********************************************************************

``` r
set.seed(19)
prob_nu <- 0.05

n <- 300   
y <- rbinom(n, 1, prob_nu)
sum(y)
```

    ## [1] 20

JAGS need the data to be in a list of values, vectors, and matrices.

``` r
dataList <- list(
  y = y,
  Ntotal = n
)
```

Creating the model

``` r
modelString <- "
model {
  for ( i in 1:Ntotal ) {
    y[i] ~ dbern( nu )   # Likelihood in code
  }
  nu = sens*pi + (1 - spec)*(1-pi)
  pi ~ dbeta( 1 , 1 )               # prior
  sens ~ dbeta( 910 , 90 )          # prior eta
  spec ~ dbeta( 950 , 50 )          # prior theta
}
"
```

Creating the initial values for the parameters

``` r
pi_init <- 0.004   # prevalence = P(L)
sens_init <- 0.91  # sensitivity = P(+|L)
spec_init <- 0.95  # specificity = 1 - P(+|L^c)

initsList = list( pi=pi_init,
                  sens=sens_init,
                  spec=spec_init) 
```

Running the model

``` r
jagsModel <- jags.model( file = textConnection(modelString), 
                         data = dataList, 
                         inits = initsList, 
                         n.chains = 5, 
                         n.adapt = 10000 )
```

    ## Compiling model graph
    ##    Resolving undeclared variables
    ##    Allocating nodes
    ## Graph information:
    ##    Observed stochastic nodes: 300
    ##    Unobserved stochastic nodes: 3
    ##    Total graph size: 314
    ## 
    ## Initializing model

``` r
update( jagsModel, n.iter = 5000 )

codaSamples = coda.samples( jagsModel, 
                            variable.names = c("nu","pi","sens","spec"),
                            n.iter=5000 )

#save( codaSamples, file=paste0("Mcmc.Rdata") )
```

Hold for text

``` r
plotPost( codaSamples[,"nu"] , main="nu" , xlab=bquote(nu) , 
          cenTend="median"  )
```

 ![Imgur Image](https://imgur.com/JzWzOpn.jpeg)

    ##         ESS      mean    median      mode hdiMass     hdiLow    hdiHigh compVal
    ## nu 13511.87 0.0720847 0.0707557 0.0664064    0.95 0.04855557 0.09948329      NA
    ##    pGtCompVal ROPElow ROPEhigh pLtROPE pInROPE pGtROPE
    ## nu         NA      NA       NA      NA      NA      NA

``` r
plotPost( codaSamples[,"pi"] , main="pi" , xlab=bquote(pi) , 
          cenTend="median"  )
```


 ![Imgur Image](https://imgur.com/H76oFB3.jpeg)

    ##         ESS       mean     median       mode hdiMass       hdiLow    hdiHigh
    ## pi 10464.99 0.02643318 0.02448011 0.02176192    0.95 1.450392e-06 0.05599159
    ##    compVal pGtCompVal ROPElow ROPEhigh pLtROPE pInROPE pGtROPE
    ## pi      NA         NA      NA       NA      NA      NA      NA

``` r
plotPost( codaSamples[,"sens"] , main="Sensitivity" , xlab=bquote(eta) , 
          cenTend="median"  )
```

![Imgur Image](https://imgur.com/zcRisRg.jpeg)

    ##          ESS      mean    median      mode hdiMass    hdiLow   hdiHigh compVal
    ## eta 15556.68 0.9098501 0.9101295 0.9102942    0.95 0.8927684 0.9274046      NA
    ##     pGtCompVal ROPElow ROPEhigh pLtROPE pInROPE pGtROPE
    ## eta         NA      NA       NA      NA      NA      NA

``` r
plotPost( codaSamples[,"spec"] , main="Specificity" , xlab=bquote(theta) , 
          cenTend="median"  )
```

![Imgur Image](https://imgur.com/LezRRgp.jpeg)

    ##            ESS     mean    median      mode hdiMass    hdiLow   hdiHigh compVal
    ## theta 12490.22 0.950693 0.9509878 0.9523356    0.95 0.9374013 0.9630593      NA
    ##       pGtCompVal ROPElow ROPEhigh pLtROPE pInROPE pGtROPE
    ## theta         NA      NA       NA      NA      NA      NA

Hold for text
