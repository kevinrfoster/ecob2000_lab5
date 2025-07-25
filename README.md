Lab 5
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2025

For this lab, we improve some of our regression models to explain wages.

*Note that since exam is next week, this lab will not turn into
homework.*

We will use a new dataset, the American Community Survey (ACS) from the
US Census, from 2021. I put it on class Slack. Note that for this data
you need an additional package, ‘haven’. This data includes information
on college students’ choices of major.

``` r
library(ggplot2)
library(tidyverse)
library(haven)
library(viridis)

setwd("..//ACS_2021_PUMS//") # your directory structure will be different
load("acs2021_recoded.RData")
```

This is a rather big file, with 3,252,599 observations. Run something
like this, to get a sense of what’s there.

``` r
summary(acs2021)
```

### Note on dataset usage

Depending on your computer, that dataset might really slow down
everything if it’s too big. So you can take a subsample. But you have
choices about how to do that so you need to pick the one that’s
appropriate.

One option is to just take a random subsample, for instance, here we
randomly select 50% of the observations:

``` r
set.seed(12345)
NN_obs <- length(acs2021$AGE)
select1 <- (runif(NN_obs) < 0.50)
smaller_data <- subset(acs2021,select1)
```

But that might not be a very smart option if we’re only going to be
looking at a particular group of people. For this lab, we’ll be looking
at how college major affects wages so … let’s first drop people who
don’t have a college major coded (including all the people who didn’t
get a college degree), for instance,

``` r
acs_coll <- acs2021 %>% filter(DEGFIELD != "N/A")
```

Now that has just 867,623 observations, a bit more than 1/4 of the size.
If needed, you can first choose a filter and then make a random
subsample of those. Then you can delete the big data from current
working memory and your computer won’t be as pokey and slow.

You might want a smaller subsample. Maybe something like this,

``` r
acs_subgroup <- acs2021 %>% filter((AGE >= 25) & (AGE <= 55) & 
                                     (LABFORCE == 2) & 
                                     (WKSWORK2 > 4) & 
                                     (UHRSWORK >= 35) &
                                     (Hispanic == 1) & 
                                     (female == 1) & 
                                     ((educ_college == 1) | (educ_advdeg == 1)))
```

Which would be people between 25 and 55 (so-called prime age), who are
in the labor force, working full year, usually fulltime, who are
Hispanic females with a college degree. If you start from a really big
dataset, then you can drill far down.

*Although, really, you should make sure you know what each one of those
restrictions is doing – recall the first rule of analysis, “Know your
data”. Check the codebook (in the zip or also in this repo for your
convenience). Some simple summary stats would be good here.*

You’ve got to be able to manage your data.

### Example of effect of major

Here is some code that shows the relationship of major to earnings,
among majors offered here,

``` r
compare_fields <- (   (acs2021$DEGFIELDD == "Civil Engineering") |
                  (acs2021$DEGFIELDD == "Electrical Engineering") |
                  (acs2021$DEGFIELDD == "Mechanical Engineering") |
                  (acs2021$DEGFIELDD == "Computer Science") | 
                  (acs2021$DEGFIELDD == "Psychology") |
                  (acs2021$DEGFIELDD == "Political Science and Government") |
                  (acs2021$DEGFIELDD == "Economics") |
                  (acs2021$EDUC == "Grade 12")  )

acs_compare_f <- acs2021 %>% filter((compare_fields & (AGE >= 25) & (AGE <= 65)) )

acs_compare_f$DEGFIELDD <- fct_drop(acs_compare_f$DEGFIELDD)


acs_compare_f$degree_recode <- recode_factor(acs_compare_f$DEGFIELDD, 
                                            "Economics" = "Econ",
                                            "Civil Engineering" = "Engr",
                                            "Electrical Engineering" = "Engr",
                                            "Mechanical Engineering" = "Engr",
                                            "Computer Science" = "CS",
                                            "Political Science and Government" = "PoliSci",
                                            "Psychology" = "Psych",
                                            .default = "HS")


p_age_income <- ggplot(data = acs_compare_f,
                       mapping = aes(x = AGE,
                                     y = INCWAGE,
                                     color = degree_recode,
                                     fill = degree_recode))

p_age_income + geom_smooth(aes(color=degree_recode, fill=degree_recode)) + 
  scale_color_viridis_d(option = "inferno", end = 0.85) + 
  scale_fill_viridis_d(option = "inferno", end = 0.65) + 
  labs(x = "Age", y = "Income",color = "field") + guides(fill = "none")
```

![](unnamed-chunk-6-1.png)<!-- -->

That shows earnings over different ages. Econ majors have the highest
earnings, followed by Engineering, then Computer Science (although they
start highest) and the rest of the majors. You might have a lot of
questions about that, including *why?*, *how?*, and *really?!* You will
dig into those with some regressions.

### Note on coding

I want you to advance your coding. The following bits of code do the
same thing:

``` r
attach(acs2021)
model_v1 <- lm(INCWAGE ~ AGE)
detach()

model_v2 <- lm(acs2021$INCWAGE ~ acs2021$AGE)

model_v3 <- lm(INCWAGE ~ AGE, data = acs2021)
```

I prefer the last one, v3. I think v2 is too verbose (especially once
you have dozens of variables in your model) while v1 is liable to errors
since, if the `attach` command gets too far separated from the `lm()`
within your code chunks, can have unintended consequences. Later you
will be doing robustness checks, where you do the same regression on
slightly different subsets. (For example, compare a model fit on all
people 18-65 vs people 25-55 vs other age ranges.) In that case v3
becomes less verbose as well as less liable to have mistakes.

However specifying the dataset, as with v3, means that any preliminary
data transformations should modify the original dataset. So if you
create `newvarb <- f(oldvarb)`, you have to carefully set
`dataset$newvarb <- f(dataset$oldvarb)` and think about which dataset
gets that transformation. The coding forces you to think about which
dataset gets the transformation, which is good.

While we’ve used ‘attach’ and ‘detach’ previously (and, whew boy,
everybody sometimes has trouble with cleaning up *detach* after a bunch
of *attach* commands!) I think it’s time to take off the training wheels
and rampdown your use of ‘attach’ and ‘detach’.

### For the lab

Try a regression with age and age-squared in addition to your other
controls, something like this:

``` r
lm((INCWAGE ~ Age + I(Age^2) + ... ) )
```

What is the peak of predicted wage? What if you add higher order
polynomials of age, such as $`Age^3`$ or $`Age^4`$? Do a hypothesis test
of whether all of those higher-order polynomial terms are *jointly*
significant. Describe the pattern of predicted wage as a function of
age. What if you used $`log(Age)`$? (And why would polynomials in
$`log(Age)`$ be useless? Experiment.)

Recall about how dummy variables work. If you added educ_hs in a
regression using the subset given above, what would that do?
(Experiment, if you aren’t sure.) What is interpretation of coefficient
on *educ_college* in that subset? What would happen if you put both
*educ_college* and *educ_advdeg* into a regression? Are your other dummy
variables in the regression working sensibly with your selection
criteria?

Why don’t we use polynomial terms of dummy variables? Experiment.

What is the predicted wage, from your model, for a few relevant cases?
Do those seem reasonable?

What is difference in regression from using log wage as the dependent
variable? Compare the pattern of predicted values from the two models
(remember to take exp() of the predicted value, where the dependent is
log wage). Discuss.

Try some interactions, like this,

``` r
lm(INCWAGE ~ Age + I(Age^2) + female + I(female*Age) + I(female*(Age^2) + ... ) 
```

and explain those outputs (different peaks for different groups).

What are the other variables you are using in your regression? Do they
have the expected signs and patterns of significance? Explain if there
is a plausible causal link from X variables to Y and not the reverse.
Explain your results, giving details about the estimation, some
predicted values, and providing any relevant graphics. Impress.
