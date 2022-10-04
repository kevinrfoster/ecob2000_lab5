Lab 5
================

### Econ B2000, MA Econometrics

### Kevin R Foster, the Colin Powell School at the City College of New York, CUNY

### Fall 2022

For this lab, we improve some of our regression models to explain wages.

*Note that since exam is next week, this lab will not turn into
homework.*

Work with a group to prepare a 5-min presentation by one of the group
members about experiment process and results. You get 75 min to prepare.

Build on the previous lab in creating useful models.

Concentrate on a smaller subset than previous. For instance if you
wanted to look at wages for Hispanic women with at least a college
degree, you might use

``` r
attach(acs2017_ny)
use_varb <- (AGE >= 25) & (AGE <= 55) & (LABFORCE == 2) & (WKSWORK2 > 4) & (UHRSWORK >= 35) & (Hispanic == 1) & (female == 1) & ((educ_college == 1) | (educ_advdeg == 1))
dat_use <- subset(acs2017_ny,use_varb) # 
detach()
```

*Although, really, you should make sure you know what each one of those
restrictions is doing – recall the first rule of analysis, “Know your
data”. So some simple summary stats would be good here.*

Your group should pick a different subset.

Try a regression with age and age-squared in addition to your other
controls, something like this:

``` r
lm((INCWAGE ~ Age + I(Age^2) + ... ) )
```

What is the peak of predicted wage? What if you add higher order
polynomials of age, such as
![Age^3](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;Age%5E3 "Age^3")
or
![Age^4](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;Age%5E4 "Age^4")?
Do a hypothesis test of whether all of those higher-order polynomial
terms are jointly significant. Describe the pattern of predicted wage as
a function of age. What if you used
![log(Age)](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;log%28Age%29 "log(Age)")?
(And why would polynomials in
![log(Age)](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;log%28Age%29 "log(Age)")
be useless? Experiment.)

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
