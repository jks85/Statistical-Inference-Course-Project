---
title: "Analyzing Tooth Growth in Guinea Pigs"
author: "Julian Simington"
date: "2024-10-25"
output: 
  pdf_document:
    keep_md: yes
---

## Overview

This project uses the `ToothGrowth` data set in R to analyze the effect of Vitamin C on tooth growth in guinea pigs. The experimental results were analyzed using a t-test and permutation tests, though other analyses are possible (see Appendix A.7). The report also includes justifications for each inference procedure, as well as conclusions regarding the experimental data.



## Exploratory Analysis  

Guinea pigs were administered varying doses of Vitamin C via two delivery methods.The box plots below compare tooth length to delivery method, grouped by dose level. The vitamin C was administered at dose levels of `0.5`, `1`, or `2` mg/day. The `OC` and `VC` delivery methods refer respectively to *orange juice* and *ascorbic acid*. See Appendix A.1 for more details regarding the experiment.


![](ToothGrowth_files/figure-latex/boxplots-1.pdf)<!-- --> 

Comparing *within* dose levels, delivery method `OJ` has a higher median tooth length at dose levels `0.5 mg/day` and `1 mg/day`, while the delivery methods appear to have equal medians at a dose level of `2 mg/day`. Additionally, within dose levels the variation of tooth length appears to be different for the two delivery methods. Furthermore we see positive outliers in tooth length for `VC` at `1 mg/day` and `OJ` at `2 mg/day`.  See Appendix A.3 for the code that generates the boxplots. 

Comparing *across* dose levels suggests a positive association between dose level and tooth length. For both delivery methods the median tooth length increases with dose level. However, we do not analyze this relationship in detail.
  
A few summary statistics for each group are included below. The statistics provide further evidence of unequal variance, but provide more evidence of symmetry for some of the data.




```
## # A tibble: 6 x 5
## # Groups:   dose [3]
##   dose  supp  median  mean    sd
##   <fct> <fct>  <dbl> <dbl> <dbl>
## 1 0.5   OJ     12.2  13.2   4.46
## 2 0.5   VC      7.15  7.98  2.75
## 3 1     OJ     23.5  22.7   3.91
## 4 1     VC     16.5  16.8   2.52
## 5 2     OJ     26.0  26.1   2.66
## 6 2     VC     26.0  26.1   4.80
```

## Choosing Inference Methods for Analysis

After analyzing the sample data to verify assumptions, the inference procedures and hypotheses below were chosen. See Appendix A.4 for an explanation of the decision making process.

- 0.5 mg/day Dose Level
  - Two sample t-test for independent groups with unequal variance
  - Null Hypothesis: No difference in mean tooth growth between `OJ` and `VC` delivery method
  - Alt Hypothesis: There is a difference in mean tooth growth between `OJ` and `VC` delivery method
- 1 mg/day and 2 mg/day Dose Level
  - Permutation Test
  - Null Hypothesis: No difference in mean tooth growth between `OJ` and `VC` delivery method
  - Alt Hypothesis: There is a difference in mean tooth growth between `OJ` and `VC` delivery method  
  
## Hypothesis Tests/P-values
All tests were performed at the 5% significance level. Permutation tests were performed using 10,000 simulations. The RNG seed was set to `5`. Since multiple hypothesis tests were performed the p-values were adjusted. The Bonferroni method was chosen for strictness. Both raw and adjusted p-values are reported. See Appendix A.6 for code.


The p-values from each hypothesis test are given below:  
  
- 0.5 mg/day Dose Level t-test
  - p-value = 0.006
  - adjusted p-value = 0.019
- 1 mg/day Dose Level permutation test
  - p-value = 0.002
  - adjusted p-value = 0.005
- 2 mg/day Dose Level permutation test
  - p-value = 0.956
  - adjusted p-value = 1  
  
## Conclusions

Based on the adjusted p-values, at the 5% significance level we reject the null hypotheses for dose levels `0.5` and `1` mg/day, but fail to reject at `2` mg/day. Therefore we conclude there is a statistically significant difference in mean tooth length when Vitamin C is administered via *orange juice* or *ascorbic acid*. The data also suggests that the mean tooth length is larger when administered via *orange juice* at dose levels of `0.5` or `1` mg/day. The current hypothesis tests could be tweaked to address this in a follow-up analysis, but it is not addressed here in order to meet the project guidelines.

\newpage

## Appendix

Note that some code chunks have been set to `eval = FALSE` so that the code is only **displayed**.


### A.1 Package Dependencies


``` r
install.packages('dplyr', repos = 'https://cran.r-project.org/')
library(dplyr)
install.packages('ggplot2', repos = 'https://cran.r-project.org/')
library(ggplot2)
```

### A.2 Experimental Design Summary

Details of the full experiment can be found [here](https://www.sciencedirect.com/science/article/abs/pii/S002231662318566X?via%3Dihub).Additionaly the `R` documenation of the `ToothGrowth` data set contains more information.

### A.3 Exploratory Data Analysis Code
  
The code below corresponds to box plots from the *Exploratory Data Analysis* section (`Tooth Length vs Delivery Method` grouped by dose). It also computes summary statistics.

``` r
data('ToothGrowth') # load data set (built into R)
ToothGrowth$dose <- as.factor(ToothGrowth$dose) # convert dose to factor
g3 <- ggplot(ToothGrowth, aes(x = supp, y = len, fill = dose))
treatment_boxplots <- g3 + geom_boxplot() + facet_wrap(vars(dose)) +
                  xlab('Delivery Method') +
                  ylab('Tooth Length') +
                  ggtitle('Tooth Length vs Delivery Method') +
                  theme(plot.title = element_text(hjust = 0.5))
```


``` r
treatment_boxplots # tooth length vs delivery method box plots
```



``` r
group_stats <- ToothGrowth %>% 
               group_by(as.factor(dose),supp) %>% 
               summarise(median = median(len), mean = mean(len), sd = sd(len))
```


``` r
group_stats
```
  

### A.4 Conditions for Selected Inference Procedures

The guidelines require *confidence intervals and/or hypothesis tests*. The box plots indicate subgroups within the data so we conduct **hypothesis tests on the difference in mean tooth length between the `OJ` and `VC` delivery methods**. See Appendix A.7 for a discussion of other possible inference methods.  

The boxplots for dose level `0.5` contain some skew, but no outliers. QQ plots (see Appendix A.5) suggest that we can reasonably assume normality. Treating the delivery groups as independent due to random assignment, we can run a two sample t-test for independent groups. 

The box plots for `1` mg/day and `2` mg/day doses each contain outliers, which violate the assumptions for a t-test. Under a null hypothesis of no difference in mean tooth length allows us to treat delivery labels as *exchangeable*.  Additionally for each of the two doses there are 20!/(10!*10!) unique labelings of the tooth data. As a result a permutation test is reasonable for each dose level.



### A.5 QQ Plots  

The code below plots QQ plots to assess normality of the sample data from dose level 0.5 mg/day



``` r
par(mfrow = c(1,3))
qqnorm(OJ_0.5$len, xlab = 'Theoretical Z-Scores',  
       ylab = 'OJ 0.5 Sample Length Quantiles')
qqnorm(VC_0.5$len, xlab = 'Theoretical Z-Scores', 
       ylab = 'VC 0.5 Sample Length Quantiles')
qqnorm(rnorm(10))
```

![](ToothGrowth_files/figure-latex/unnamed-chunk-1-1.pdf)<!-- --> 
  
The departures from normality in the sample data look reasonable when compared to simulated data from a normal distribution. We can reasonably treat this portion of the sample data as being drawn from a normal distribution.

### A.6 Statisical Inference Code


``` r
# t-test on dose level 0.5 mg/day
OJ_0.5 <- ToothGrowth %>% filter(supp == 'OJ' & dose == 0.5) # OJ @ 0.5 mg dose
VC_0.5 <- ToothGrowth %>% filter(supp == 'VC' & dose == 0.5) # VC @ 0.5 mg dose
t_test_0.5 <- t.test(OJ_0.5$len,VC_0.5$len, alternative = 'two.sided') # t-test
###################
set.seed(5) # set seed for reproducibility of permutation tests

#test_diff function computes difference in group means for a given set of labels
test_diff <- function(resp,group){ 
                mean(resp[group == 'OJ']) - mean(resp[group =='VC'])}
###################
# permutation test on dose level 1 mg/day

dose_1 <- ToothGrowth %>% filter(dose == 1) # select 1 mg dose data
observed_diff_1 <- 
        test_diff(dose_1$len, 
                  dose_1$supp) # observed diff in group means from dose 1 data
permutation_diffs_1 <- # difference in group means for permuted delivery labels
          sapply(1:10000, function(i) test_diff(dose_1$len,sample(dose_1$supp)))

p_val_dose_1 <- mean(abs(permutation_diffs_1) > abs(observed_diff_1)) # p-value
###################
# permutation test on dose level 2 mg/day
dose_2 <- ToothGrowth %>% filter(dose == 2) # 2 mg dose data
observed_diff_2 <- 
        test_diff(dose_2$len, 
                  dose_2$supp) # observed diff in group means from dose 2 data
permutation_diffs_2 <- # difference in group means for permuted delivery labels
          sapply(1:10000, function(i) test_diff(dose_2$len,sample(dose_2$supp)))
p_val_dose_2 <- mean(abs(permutation_diffs_2) > abs(observed_diff_2))
adj_p_vals <- p.adjust(c(t_test_0.5$p.value, # adjusted p-values 
                         p_val_dose_1,p_val_dose_2), 
                         method = 'bonferroni') 
```


### A.7 Other Potential Inference Procedures  
  
It is possible to analyze tooth length vs dose level using regression techniques. Additionally, it is possible to perform an ANOVA hypothesis test to compare mean tooth length across the six treatment groups. However, both methods are outside of the scope of the class and inconsistent with guidelines.

