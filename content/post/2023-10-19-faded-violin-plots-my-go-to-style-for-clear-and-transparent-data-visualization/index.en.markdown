---
title: 'Creating Report-Ready Charts for Group Comparisons in R: A Step-By-Step Guide'
subtitle: 'Featuring: The faded violin plot (*viofade*)'
author: Dallas Novakowski
date: '2023-10-19'
slug: faded-violin-plots-my-go-to-style-for-clear-and-transparent-data-visualization
summary: "This post is a rundown of a workflow that I use for preparing group-comparison graphs that are ready to include in nearly any kind of report. This post is sectioned out piece-by-piece to better demonstrate the impacts of each function and argument for ggplot, and also incorporates statistical hypothesis tests."

categories: []
tags: 
  - raincloud
  - charts
  - plotting
  - graphs
  - ggplot2
  - shadecloud
  - fadecloud
  - r
  - simple
  - code
  - data visualization
  - experiments
  - raincloud
  - boxplot
  - ggplot
  - ggdist
  - group comparisons
  - dataviz
  - how to create a graph
  - plotting data in R 
  - plotting data in Excel
  - plotting data in Python
  - chart types
  - types of graphs
  - graph types
  - data plotting techniques
  - graphing data
  - scatter plot
  - line chart
  - bar chart
  - bar graph
  - histogram
  - box plot
  - box and whisker plot
  - data plotting tools
  - data analysis and visualization best practices
  - data visualization tutorials
  - data visualization software comparison
  - data visualization tips
  - data visualization examples
  - Data Visualization in R
  - R Programming
  - Group Comparison Charts
  - Report-Ready Charts
  - Data Analysis in R
  - Data Visualization Tutorial
  - Step-By-Step Guide
  - Data Reporting
  - Chart Creation in R
  - R Data Visualization
  - Group Comparison in Statistics
  - Graphical Data Analysis
  - Creating Visual Reports
  - Data Visualization Techniques
  - R Data Visualization Tools
  - Data Presentation
  - Statistical Analysis in R
  - R Plotting
  - Data Visualization Best Practices
  - Data Analysis Tutorials
  - Statistical Graphics in R
  - Data Visualization for Reports
  - Comparative Analysis in R
  - Data Visualization Tips
  - R Data Visualization Examples
  - Data Visualization for Research
  - Creating Publication-Quality Charts
  - Group Comparison Techniques
  - R Data Visualization Walkthrough
  - Visual Data Analysis
  - R Plotting Libraries
  - Data Presentation in R
  - Visualizing Group Differences
  - Effective Data Visualization
  - R Graphics
  - Group Comparison Plots
  - R Data Visualization Guide
  - Reporting and Visualization
  - Data Visualization for Presentations
  - R Data Visualization Step-By-Step
  - Data Visualization Showcase
  - R Graphics Packages
  - Data Presentation Tips
  - Creating Graphs in R
  - Comparative Data Visualization
  - Visualizing Group Data
  - Data Visualization for Analysis
  - Customizing Charts in R
  - R Data Visualization Resources
  - Data Visualization Essentials
  - Group Comparison Visuals
  - Chart Design in R
  - Advanced Data Visualization
  - Data Visualization for Decision Making
  - R Data Plotting Guide
  - Effective Visual Communication
  - Visualizing Group Statistics
  - R Data Visualization Best Practices
  - Data Visualization for Non-Programmers
  - R Data Visualization for Beginners
  - Data Visualization Basics
  - R Data Visualization Fundamentals
  - Simple Charts in R
  - Easy Data Visualization in R
  - Beginner-Friendly Data Visualization
  - Introduction to Data Visualization in R
  - Data Visualization for Newbies
  - Chart Creation for Beginners
  - R Data Visualization Starter Guide
  - Visualizing Data for Novices
  - Data Visualization Simplified
  - Beginning Data Visualization with R
  - Charting for R Beginners
  - Getting Started with R Graphics
  - Data Visualization for Data Science Beginners
  - Quick R Charting for Novices
  - R Data Visualization for Absolute Beginners
  - Step-by-Step Charting for Beginners
  - Basic Data Visualization in R
  - Charting Made Easy with R
  - Data Visualization for Beginners
  - R Data Visualization for Data Analysis Newbies
  - Easy Chart Creation in R
  - Visualizing Data in R for Starters
  - Data Visualization for R Beginners
  - Beginner's Guide to Data Visualization in R
  - Simple R Charting Tutorial
  - Introduction to Data Visualization with R for Beginners
  - Basic R Graphics
  - Data Visualization Step-by-Step for Novices
  - Beginner's Data Visualization Tips
  - R Data Visualization Made Simple
  - Entry-Level Data Visualization in R
  - Starting with Data Visualization in R
  - Beginner-Friendly Data Presentation in R
  - Easy R Graphs for Beginners
  - Data Visualization Crash Course for R Newcomers
  - Visualizing Data with R: A Beginner's Approach
  - R Data Visualization 101
  - Getting Started with R Data Visualization: Beginner's Edition
  - Visualizing Statistical Data
  - Statistical Data Visualization
  - Data Visualization for Statistical Analysis
  - Visualizing Descriptive Statistics
  - Statistical Graphics in R
  - Visualizing Hypothesis Testing
  - Correlation Visualization
  - Regression Analysis Visuals
  - Statistical Inference and Visualization
  - Visualizing Probability Distributions
  - Exploratory Data Analysis with Visualization
  - Visualizing Confidence Intervals
  - Statistical Plots in R
  - Data Visualization for ANOVA
  - Visualization of Multivariate Analysis
  - Data Visualization for T-Tests
  - Visualizing Statistical Models
  - Combining Charts with Statistical Tests
  - Statistical Data Visualization Best Practices
  - Visualization for Inferential Statistics
authors: []
lastmod: '2023-10-19T17:15:21-07:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
toc_depth: 2
---




<details>
<summary> <b> Click here if you just want some code that is ready to copy and paste!</b> </summary>

![](pingu_dope.gif)



```r
# Load-packages ----

library(palmerpenguins) # dataset
library(cowplot) #  publication-ready plots
library(tidyverse) # ggplot and tidy functions
library(ggdist) # density slabs
library(EnvStats) # stat_n_text() for inserting sample sizes
library(ggtext) # formatting text elements with markdown syntax
library(afex) # anova functions
library(car) # anova functions
library(emmeans) # estimated marginal means and pairwise comparisons
library(effectsize) # effect sizes
library(ggpubr) # significance brackets
library(weights) # rounding values
library(report) # for citing packages
library(janitor) #for cleaning variable names

# Define color palette
nova_palette <- c("#78AAA9", "#FFDB6E")


# load-data ----

# assigns data to a dataframe we call "df"
df <- palmerpenguins::penguins

# drop rows with missing values
df <- df[complete.cases(df)==TRUE, ]

df <- rename(df,  flipper = flipper_length_mm)

# peek at the structure of our dataframe 
str(df)

# This function can handle multiple grouping variables, up to a three-way design
make_summary <- function(data, dv, grouping1, grouping2, grouping3){
  data %>%
    group_by({{grouping1}},{{grouping2}},{{grouping3}}) %>%
    dplyr::summarise(
      mean = round(mean({{dv}}),2),
      min = round(min({{dv}}),2),
      max = round(max({{dv}}),2),
      n = n(),
      std_dev = round(sd({{dv}}),2),
      se = round(sd({{dv}}) /sqrt(n()),2),
      y25 = round(quantile({{dv}}, 0.25)),
      y50 = round(median({{dv}})),
      y75 = round(quantile({{dv}}, 0.75)),
      loci = round(mean({{dv}}),1) - 1.96*se,
      upci = round(mean({{dv}}),1) + 1.96*se
    )
  
}



# Fit data
flipper_fit <- stats::aov(flipper ~ species, data = df)
# Run anova
flipper_anova <- car::Anova(flipper_fit)

# Extract effect size (partial eta squared) from anova
flipper_anova_pes <- effectsize::eta_squared(flipper_anova,
                                             alternative="two.sided",
                                             verbose = FALSE,
                                             generalized= FALSE)
# Convert anova table into dataframe
flipper_anova <- data.frame(flipper_anova)

# import effect size estimates and confidence intervals to anova dataframe
flipper_anova$pes_ci95_lo <- flipper_anova_pes$CI_low
flipper_anova$pes_ci95_hi <- flipper_anova_pes$CI_high
flipper_anova$pes <- flipper_anova_pes$Eta2

# round all numeric columns to 2 decimal places
flipper_anova <- flipper_anova %>%
  dplyr::mutate_if(is.numeric, function(x) round(x, 2))

# display anova dataframe
knitr::kable(flipper_anova)




# Extract estimated marginal means
flipper_emmeans <- emmeans::emmeans(flipper_fit, specs = pairwise ~ species)

# convert estimated marginal mean contrasts to dataframe
flipper_emmeans_contrasts <- data.frame(flipper_emmeans$contrasts)

# Convert estimated marginal means to dataframe
flipper_emmeans_tidy <- data.frame(flipper_emmeans$emmeans)



#merging regular summary stats with emmeans
merge_emmeans_summary <- function(summary_data,emmeans_tidy){
  summary_data$emmean <- emmeans_tidy$emmean
  summary_data$emmean_se <- emmeans_tidy$SE
  summary_data$emmean_loci <- emmeans_tidy$lower.CL
  summary_data$emmean_upci <- emmeans_tidy$upper.CL
  
  summary_data
}

# Use `merge_emmeans_summary()` to combine with our basic `flipper_summary` into a single model-enhanced dataframe:

# Order the dataframes based on dependent variables - Not necessary here, but good practice that helps for factorial designs
flipper_summary <- flipper_summary[order(flipper_summary$species), ]
flipper_emmeans_tidy <- flipper_emmeans_tidy[order(flipper_emmeans_tidy$species), ]

# merge dataframes
flipper_summary <- merge_emmeans_summary(summary_data = flipper_summary,
                                         emmeans_tidy = flipper_emmeans_tidy)
# Round numeric values
flipper_summary <- flipper_summary %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_summary)



#Now we have our analysis data to work with! From here, we can get the data ready to actually work with. To start, we can make a custom function, `report_pval_full()`, that helps us convert p-values to our desired formatting.


# rounding or abbreviating p-values
report_pval_full <- function(pval, italicize = TRUE) {
  ifelse(pval < .001, paste0(
    ifelse(italicize == TRUE, "*p*", "p"), " < .001"),
    paste0(
      ifelse(italicize == TRUE, "*p*", "p")," = ", ifelse(pval >= .01,weights::rd(pval,2),
                                                          weights::rd(pval,3))
    )
  )
}

#Create and prepare dataframe for pairwise comparisons

# convert p-value to non-italicized version (for significance brackets)
flipper_emmeans_contrasts$p_no_it <-  report_pval_full(flipper_emmeans_contrasts$p.value, italicize = FALSE)

# convert p-value to italicized version (for in-text citations and captioning graph)
flipper_emmeans_contrasts$p.value <- report_pval_full(flipper_emmeans_contrasts$p.value)


flipper_emmeans_contrasts <- flipper_emmeans_contrasts %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_emmeans_contrasts)





report_tidy_anova_etaci <- function(tidy_frame, # your tidy anova dataframe
                                    term, # the predictor in your tidy anova dataframe
                                    effsize = TRUE, # display the effect size
                                    ci95 = TRUE, # display the 95% CI
                                    ci.lab = TRUE, # display the "95% CI" label
                                    teststat = TRUE, # display the F-score and degrees of freedom
                                    pval = TRUE # display the p-value
){
  text <- paste0(
    ifelse(effsize == TRUE, paste0(
      "\u03b7^2^ = ", #unicode for eta square
      round(as.numeric(tidy_frame[term,"pes"]),2)), 
      ""),
    ifelse(ci95 == TRUE, paste0(
      ifelse(ci.lab == TRUE,
             paste0(", 95% CI ["), " ["),   
      round(as.numeric(tidy_frame[term,"pes_ci95_lo"]),2), ", ", 
      round(as.numeric(tidy_frame[term,"pes_ci95_hi"]),2),"]"),
      ""),
    ifelse(teststat == TRUE, 
           paste0(", *F*(", tidy_frame[term,"Df"],
                  ", ", tidy_frame["Residuals","Df"], ") = ",
                  round(as.numeric(tidy_frame[term,"F.value"]), 2)), 
           ""),
    ifelse(pval == TRUE, paste0(
      ", ",  report_pval_full(tidy_frame[term,"Pr..F."])),
      "")
  )
  
  return(text)
}

# visualize the thing!

ggplot(data = df, # specify the dataframe that we want to pull variables from
       aes(y = flipper, # our dependent/response/outcome variable 
           x = species  # our grouping/independent/predictor variable
       )) +  
  cowplot::theme_half_open() + # nice theme for publication
  ggdist::stat_slab(side = "both",  # turn from slab to violin
                    aes(fill_ramp = stat(level)),  # specify shading
                    fill = nova_palette[1], # specify used to fill violin
                    .width = c(.50, 1), # specify shading quantiles
                    scale = .4) + # change size of violin
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))  +   
  geom_text(data = flipper_summary, aes(x = species, 
                                        y = mean, 
                                        label = round(mean,1)),
            color="black", 
            size = 2.5, 
            vjust = 5) +
  geom_text(data = flipper_summary, 
            aes(label = paste("n =", n), 
                y = flipper_summary[which.min(flipper_summary$min),]$mean -
                  3*flipper_summary[which.min(flipper_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 2, 
            color = "grey60") +
  
  guides(fill_ramp = "none") + # get rid of legend element for fading quantiles
  labs(subtitle = report_tidy_anova_etaci(flipper_anova,"species")) +
  theme(plot.subtitle = ggtext::element_markdown()) +
  ggpubr::geom_bracket( 
    tip.length = 0.02, # the downard "tips" of the bracket
    vjust = 0, # moves your text label (in this case, the p-value)
    xmin = 1, #starting point for the bracket
    xmax = 2, # ending point for the bracket
    y.position = 220, # vertical location of the bracket
    label.size = 2.5, # size of your bracket text
    label = paste0(flipper_emmeans_contrasts[flipper_emmeans_contrasts$contrast == 
                                               'Adelie - Chinstrap', "p_no_it"]) # content of your bracket text
  ) + 
  ggpubr::geom_bracket( 
    tip.length = 0.02, 
    vjust = 0,
    xmin = 2, 
    xmax = 3, 
    y.position = 227 ,
    label.size = 2.5,
    label = paste0("My hypothesis, ", 
                   flipper_emmeans_contrasts[flipper_emmeans_contrasts$contrast == 
                                               'Chinstrap - Gentoo', "p_no_it"])) +
  theme(axis.title = element_text(face="bold")) + 
  ylab("Flipper length (mm)") +
  xlab("Species")




# Now Try a Factorial Design
## Summarize the Data

flipper_fact_summary <- make_summary(data = df, dv = flipper, grouping1 = species, grouping2 = sex)

knitr::kable(flipper_fact_summary)


## Analyze the Data
# Fit data
flipper_fact_fit <- stats::aov(flipper ~ species*sex, data = df)
# Run anova
flipper_fact_anova <- car::Anova(flipper_fact_fit, type = 2)

#Extract estimated marginal means with `emmeans::emmeans()` and merge with basic summary dataframe with our custom `merge_emmeans_summary()` function.

# Extract estimated marginal means
flipper_fact_emmeans <- emmeans::emmeans(flipper_fact_fit, specs = pairwise ~ species:sex)

# Convert estimated marginal means to dataframe
flipper_fact_emmeans_tidy <- data.frame(flipper_fact_emmeans$emmeans)


flipper_fact_summary <- flipper_fact_summary[order(flipper_fact_summary$species, 
                                                   flipper_fact_summary$sex), ]

flipper_fact_emmeans_tidy <- flipper_fact_emmeans_tidy[order(flipper_fact_emmeans_tidy$species,
                                                             flipper_fact_emmeans_tidy$sex), ]


flipper_fact_summary <- merge_emmeans_summary(summary_data = flipper_fact_summary,
                                                  emmeans_tidy = flipper_fact_emmeans_tidy)

flipper_fact_summary <- flipper_fact_summary %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_fact_summary)


#Calculate effect sizes with `effectsize::eta_squared()`


flipper_fact_anova_pes <- effectsize::eta_squared(flipper_fact_anova,
                                             alternative="two.sided",
                                             verbose = FALSE,
                                             generalized= FALSE)

flipper_fact_anova <- data.frame(flipper_fact_anova)

flipper_fact_anova[1:3,"pes_ci95_lo"] <- flipper_fact_anova_pes$CI_low
flipper_fact_anova[1:3,"pes_ci95_hi"] <- flipper_fact_anova_pes$CI_high
flipper_fact_anova[1:3,"pes"] <- flipper_fact_anova_pes$Eta2


flipper_fact_anova <- flipper_fact_anova %>%
  dplyr::mutate_if(is.numeric, function(x) round(x, 3))

knitr::kable(flipper_fact_anova)



# convert estimated marginal mean contrasts to dataframe
flipper_fact_emmeans_contrasts <- data.frame(flipper_fact_emmeans$contrasts)

# convert p-value to non-italicized version (for significance brackets)
flipper_fact_emmeans_contrasts$p_no_it <-  report_pval_full(flipper_fact_emmeans_contrasts$p.value, italicize = FALSE)

# convert p-value to italicized version (for in-text citations and captioning graph)
flipper_fact_emmeans_contrasts$p.value <- report_pval_full(flipper_fact_emmeans_contrasts$p.value)


flipper_fact_emmeans_contrasts <- flipper_fact_emmeans_contrasts %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_fact_emmeans_contrasts)

## Vizualize the Data


viofade_fact <- ggplot(data = df,
                       aes(y = flipper, # our dependent/response/outcome variable 
                           x = species,  # our grouping/independent/predictor variable
                           fill = sex)) +  # our third grouping/independent/interaction variable
  ggdist::stat_slab(side = "both",  
                    aes(fill_ramp = stat(level)),
                    .width = c(.50, 1),
                    scale = .4,
                    position = position_dodge(width = .7)) +
  scale_fill_manual(values = nova_palette) +
  # stat_summary(fun.data = "mean_cl_normal", 
  #              show.legend = F, 
  #              color = "black", 
  #              position = position_dodge(width = .7))  + 
  geom_pointrange(data = flipper_fact_summary,
                  aes(x = species, 
                      y = emmean, 
                      ymin = emmean_loci, 
                      ymax = emmean_upci),
                  position = position_dodge(.7), show.legend = F) +
  geom_text(data = flipper_fact_summary, aes(x = species, 
                                             y = emmean, 
                                             label = round(emmean,1)),
            color="black", 
            size = 2.5, 
            vjust = 5, 
            position = position_dodge(width = .7)) +
  # stat_summary(aes(label=round(..y..,0)), 
  #              fun=mean, geom="text",  
  #              position = position_dodge(width = .7), 
  #              size=2.5, 
  #              vjust = 5) +
  geom_text(data = flipper_fact_summary, 
            aes(label = paste("n =", n), 
                y = flipper_fact_summary[which.min(flipper_fact_summary$min),]$mean -
                  3*flipper_fact_summary[which.min(flipper_fact_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 2, 
            color = "grey60", 
            position = position_dodge(width = .7)) +
  guides(fill_ramp = "none") +
  scale_y_continuous(expand = expansion(mult = c(0.03, 0))) +
  cowplot::theme_half_open() +  
  theme(axis.title = element_text(face="bold"), 
        legend.title = element_text(face="bold")) + 
  labs(x = "Species", 
       y = "Flipper length (mm)", 
       fill = "Sex") +
  ggpubr::geom_bracket(inherit.aes = FALSE, # necessary for factorial design
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = 1.175, # You need to play with these by hand
                       xmax = 1.825, 
                       y.position = 210 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Chinstrap female - Adelie male', "p_no_it"])) +
  ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = .825, 
                       xmax = 1.175, 
                       y.position = 217 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                       'Adelie female - Adelie male', "p_no_it"])) +
  ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = .825, 
                       xmax = 2.175, 
                       y.position = 225 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Adelie female - Chinstrap male', "p_no_it"])) +
    ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = -0.02, 
                       vjust = 0.6,
                       xmin = 2.175, 
                       xmax = 3.175, 
                       y.position = 185 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Chinstrap male - Gentoo male', "p_no_it"])
  )
  
  
viofade_fact_bracket




# Now Compare Just Two Groups

#Use our workhorse `make_summary()`:


flipper_sex_summary <- make_summary(data = df, dv = flipper, grouping1 = sex)

knitr::kable(flipper_sex_summary)

#Run t a between-subjects t-test with `t.test()`, extract and format it with `report::report()` and `data.frame()`, and clean the names with `janitor::clean_names()`:

flipper_sex_ttest <- t.test(flipper ~ sex, data=df) %>%
  report::report() %>%
  data.frame() %>%
  janitor::clean_names()

knitr::kable(flipper_sex_ttest)

#`report_tidy_t()` is another useful custom function for doing some in-text or in-plot reporting of a t-test. `report::report_statistics()` is also a notable function, but I haven't figured out how to selectively extract the elements. Now we create the `report_tidy_t()` function:


report_tidy_t <- function(tidy_frame, 
                          ci.lab = TRUE, 
                          test.stat = FALSE, 
                          point = TRUE){
  
  text <- paste0(ifelse(point==TRUE,paste0("*d* = ",
                                           round(tidy_frame$d,2)),""
                        ), 
                 ifelse(ci.lab == TRUE,
                        paste0(", 95% CI [", round(tidy_frame$d_ci_low  ,2), ", ", 
                               round(tidy_frame$d_ci_high  ,2),"]"),
                        paste0(" [", round(tidy_frame$d_ci_low  ,2), ", ", 
                               round(tidy_frame$d_ci_high  ,2),"]")),
                 
                 ifelse(test.stat == TRUE,paste0(", *t*","(", 
                                                 round(tidy_frame$df_error, 2) 
                                                 ,")"," = ", round(tidy_frame$t,2)),""),
                 ", *p* ", 
                 
                 ifelse(tidy_frame$p < .001, "< .001",
                        ifelse(tidy_frame$p > .01,paste("=", tidy_frame$p %>% round(2)),
                               paste("=", tidy_frame$p %>% round(3)))
                        )
                 ) 
  return(text)
}

#Now we can specify the whole plot, using everything we've learned:

viofade_flipper_sex <- ggplot(data = df, # specify the dataframe that we want to pull variables from
       aes(y = flipper, # our dependent/response/outcome variable 
           x = sex  # our grouping/independent/predictor variable
       )) +  
  # insert faded violin slab
  ggdist::stat_slab(side = "both",  # turn from slab to violin
                    aes(fill_ramp = stat(level)),  # specify shading
                    fill = nova_palette[1], # specify used to fill violin
                    .width = c(.50, 1), # specify shading quantiles
                    scale = .4) + # change size of violin
  # insert mean text
  geom_text(data = flipper_sex_summary, # our externally-defined summary dataframe
            aes(x = sex, # x-position of geom_text
                y = mean, # y-position of geom_text
                label = round(mean,1)), # actual content
            color="black", 
            size = 4, # size of text
            vjust = 3.5) + # some vertical nudging downwards
  # Insert mean and 95% confidence interval
  geom_pointrange(data = flipper_sex_summary, # our externally-defined summary dataframe
                  aes(x = sex,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  )) +
  # insert sample sizes
  geom_text(data = flipper_sex_summary, # specify our custom dataframe
            aes(label = paste("n =", n), # the actual text content
                y = flipper_summary[which.min(flipper_summary$min),]$mean -
                  3*flipper_summary[which.min(flipper_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 3,  # size of text
            color = "grey60") +
  guides(fill_ramp = "none") + # get rid of legend element for fading quantiles
  cowplot::theme_half_open() + # nice theme for publication
  labs(subtitle = report_tidy_t(flipper_sex_ttest, test.stat = T)) + # test statistic 
  theme(plot.subtitle = ggtext::element_markdown()) + # enable markdown styling 
  theme(axis.title = element_text(face="bold")) + #bold axis titles
  ylab("Flipper length (mm)") + # change y-axis label 
  xlab("Sex") # change x-axis label


viofade_flipper_sex
```


</details>

I have been enjoying making some guides for data visualization, namely trying to improve on the raincloud plot via the new [fadecloud](https://dallasnova.rbind.io/post/efficient-data-visualization-with-faded-raincloud-plots-delete-boxplot/) as well as [faded dotplots and shadeplots](https://dallasnova.rbind.io/post/creating-simple-and-transparent-data-graphs-using-faded-dotplots-and-shadeplots/). However, these aren't the plot types I usually choose for publication. This post is a rundown of a workflow that I use for preparing group-comparison graphs that are ready to include in nearly any kind of report.

<!-- I believe these methods have their place, but I find when I actually present my data, I don't actually use these methods: there are still a lot of objects visible that make the addition of significance brackets overly noisy.  -->

<!-- My past posts have tended to dump a lot of code at once.  -->

In addition to being a personal resource, this post is intended to be more accessible for newcomers to the R and ggplot ecosystems. Although I assume a working understanding of R, this post is sectioned out piece-by-piece to better demonstrate the impacts of each function and argument for ggplot. 


# Free Learning Resources

With newcomers in mind, here are some no-cost resources

- R fundamentals
  - [R for Data Science (2e)](https://r4ds.hadley.nz/)
  - [Prime Hints For Running A Data Project In R](https://kkulma.github.io/2018-03-18-Prime-Hints-for-Running-a-data-project-in-R/)
  - [How to organize your analyses with R Studio Projects](https://www.rforecology.com/post/organizing-your-r-studio-projects/)
  - [What They Forgot to Teach You About R](https://rstats.wtf/)
  - [The tidyverse style guide](https://style.tidyverse.org/files.html)
  - [Functions](http://adv-r.had.co.nz/Functions.html)
  - [Reproducible Analysis With R](https://nceas.github.io/sasap-training/materials/reproducible_research_in_r_fairbanks/)
  - [R Workflow blog post](https://www.fharrell.com/post/rflow/) and [e-book](https://hbiostat.org/rflow/)
  - [Data Skills for Reproducible Research](https://psyteachr.github.io/reprores-v3/)
  - [Beyond Basic R - Introduction and Best Practices](https://waterdata.usgs.gov/blog/intro-best-practices/)
  - [Data visualization with ggplot2 :: Cheat Sheet](https://rstudio.github.io/cheatsheets/html/data-visualization.html)
    - [The pdf](https://rstudio.github.io/cheatsheets/data-visualization.pdf)
    - [More RStudio/Posit cheat sheets](https://rstudio.github.io/cheatsheets/)
  - [How to Structure RStudio Projects](https://rfortherestofus.com/2021/08/rstudio-project-structure/)
  - [Organising Data in R](https://www.ucd.ie/ecomodel/Resources/Sheet2b_data_organisation_WebVersion.html)
- Stats
  - [Statistical rethinking with brms, ggplot2, and the tidyverse](https://bookdown.org/content/c23e1ece-b5d6-4cab-8d8e-978c2e5b7a53/)
  - [Statistical Rethinking videos](https://www.youtube.com/playlist?list=PLDcUM9US4XdPz-KxHM4XHt7uUVGWWVSus)
  - [One Way ANOVA with R](https://bcdudek.net/anova/index.html)
  - [Understanding Statistical Power and Significance Testing: an interactive visualization](https://rpsychologist.com/d3/nhst/)
  - [easystats package](https://easystats.github.io/easystats/index.html)
  - [Statistics: Data analysis and modelling](https://mspeekenbrink.github.io/sdam-book/)
    - [An R companion to Statistics: data analysis and modelling](https://mspeekenbrink.github.io/sdam-r-companion/index.html)
  - [Analysis of Factorial Designs foR Psychologists](https://github.com/mattansb/Analysis-of-Factorial-Designs-foR-Psychologists)
  - [Guide to Effect Sizes and Confidence Intervals](https://matthewbjane.quarto.pub/effect-size-and-confidence-intervals-guide/)


<!-- I believe that practices that are easy to pick up are more likely to catch on. My hope is that readers will find it easier to pick up better practices in communicating their data! -->


# Notes on the Workflow

After a few projects, I have found that I get my best results by summarizing and analyzing my data before trying to create any publication-ready visualizations. There are a few benefits:

1. This process automatically creates the graph with the most up-to-date test statistics, all in a reproducible and portable process. Otherwise, when you make changes, you visualize, run the analysis, and have to go back and manually update any revised stats in the plot. Similarly, if your significance tests are done as part of your plotting (e.g., with r package [`ggsignif`](https://const-ae.github.io/ggsignif/index.html), you can end up struggling with making sure your in-text and graphed statistical tests match.

<!-- lol i'm just here to make space -->

2. You can easily use this process to visualize estimated marginal means, which are a product of your statistical model. This allows you to match your statistics to the visualized data.

<!-- , instead of reporting statistics on estimated marginal means (controlling for covariates/confounders) that mismatch the visualized raw means (which do not control for those extra variables). -->

3. Summary data is often needed for reporting and extra scripting functions. Instead of having to go back to your scripts to get summary statistics, they are readily and consistently available as part of your data processing and analysis.

<!-- lol i'm just here to make space -->

4. This workflow is easily adapted to multivariate and univariate analyses, potentially cutting down on time and decisions down the road.

**Please do not take this guide as license to start summarizing and analyzing your data without properly exploring and cleaning it.** This workflow is intended for creating the final product (i.e., after you have screened for influential observations, missingness, etc.).


# Load Packages


```r
library(palmerpenguins) # dataset
library(cowplot) #  publication-ready plots
library(tidyverse) # ggplot and tidy functions
library(ggdist) # density slabs
library(EnvStats) # stat_n_text() for inserting sample sizes
library(ggtext) # formatting text elements with markdown syntax
library(afex) # anova functions
library(car) # anova functions
library(emmeans) # estimated marginal means and pairwise comparisons
library(effectsize) # effect sizes
library(ggpubr) # significance brackets
library(weights) # rounding values
library(report) # for citing packages
library(janitor) #for cleaning variable names

# Define color palette
nova_palette <- c("#78AAA9", "#FFDB6E")
```

# Load Data

![](palmer_penguin.png)

We'll use the `palmerpenguins` dataset, which is easy to get running right away

<figure>
  <img src="penguin_slip.gif" alt="Commuting penguins" width="80%">
  <figcaption>Although, with my programming speed, 'running' might be a strong word</figcaption>
</figure>



```r
# assigns data to a dataframe we call "df"
df <- palmerpenguins::penguins

# drop rows with missing values
df <- df[complete.cases(df)==TRUE, ]

df <- rename(df,  flipper = flipper_length_mm)

# peek at the structure of our dataframe 
str(df)
```

```
## tibble [333 x 8] (S3: tbl_df/tbl/data.frame)
##  $ species       : Factor w/ 3 levels "Adelie","Chinstrap",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ island        : Factor w/ 3 levels "Biscoe","Dream",..: 3 3 3 3 3 3 3 3 3 3 ...
##  $ bill_length_mm: num [1:333] 39.1 39.5 40.3 36.7 39.3 38.9 39.2 41.1 38.6 34.6 ...
##  $ bill_depth_mm : num [1:333] 18.7 17.4 18 19.3 20.6 17.8 19.6 17.6 21.2 21.1 ...
##  $ flipper       : int [1:333] 181 186 195 193 190 181 195 182 191 198 ...
##  $ body_mass_g   : int [1:333] 3750 3800 3250 3450 3650 3625 4675 3200 3800 4400 ...
##  $ sex           : Factor w/ 2 levels "female","male": 2 1 1 1 2 1 2 1 2 2 ...
##  $ year          : int [1:333] 2007 2007 2007 2007 2007 2007 2007 2007 2007 2007 ...
```

For your own real-life data importing, the function you use depends on your file type. I usually use the following: `readr::read_csv()` (csv files), `readxl::read_excel()` (excel files), or `haven::read_sav()` (SPSS files). 

You can find tutorials for data importing at: 

- [geeksforgeeks](https://www.geeksforgeeks.org/how-to-import-data-from-a-file-in-r-programming/)
- [statology](https://www.statology.org/import-csv-into-r/)
- [thedatastudent](https://thedatastudent.com/how-to-import-data-in-r-for-beginners/)
- [statmethods](https://www.statmethods.net/data-input/importingdata.html)
- [intro2r](https://www.r-tutor.com/r-introduction/data-frame/data-import)
- [scientifictools](https://scientifictools.org/courses/entry/5-how-to-import-data-and-datasets-in-r-using-rstudio/)
- [sthda](http://sthda.com/english/wiki/importing-data-into-r)
- [R for data science](https://r4ds.hadley.nz/import)

One of the tricky parts of data importing is defining the right paths to your data. Best practice is to use [RStudio Projects](https://r4ds.hadley.nz/workflow-scripts.html#rstudio-projects), and use a consistent subfolder scheme for organizing your data/output/figures/reports/etc.

For project organization schemes, see:

- [rfortherestofus](https://rfortherestofus.com/2021/08/rstudio-project-structure/)
- [Krista L. DeStasio](https://kdestasio.github.io/post/r_best_practices/)
- [Telethonkids](https://telethonkids.wordpress.com/2019/07/24/how-do-you-organise-your-r-project-this-is-what-we-do/)

# Summarize the Data

`make_summary` is my workhorse custom function for any group comparison analyses; its role is getting our variable's major summary statistics in one place:


```r
# This function can handle multiple grouping variables, up to a three-way design
make_summary <- function(data, dv, grouping1, grouping2, grouping3){
  data %>%
    group_by({{grouping1}},{{grouping2}},{{grouping3}}) %>%
    dplyr::summarise(
      mean = round(mean({{dv}}),2),
      min = round(min({{dv}}),2),
      max = round(max({{dv}}),2),
      n = n(),
      std_dev = round(sd({{dv}}),2),
      se = round(sd({{dv}}) /sqrt(n()),2),
      y25 = round(quantile({{dv}}, 0.25)),
      y50 = round(median({{dv}})),
      y75 = round(quantile({{dv}}, 0.75)),
      loci = round(mean({{dv}}),1) - 1.96*se,
      upci = round(mean({{dv}}),1) + 1.96*se
      )
      
}
```


Now we can use our custom `make_summary()` function to make a summary table:


```r
# Apply our custom function to create summary dataframe
flipper_summary <- make_summary(data = df, dv = flipper, grouping1 = species)

# Show summary dataframe in table
knitr::kable(flipper_summary)
```



|species   |   mean| min| max|   n| std_dev|   se| y25| y50| y75|     loci|     upci|
|:---------|------:|---:|---:|---:|-------:|----:|---:|---:|---:|--------:|--------:|
|Adelie    | 190.10| 172| 210| 146|    6.52| 0.54| 186| 190| 195| 189.0416| 191.1584|
|Chinstrap | 195.82| 178| 212|  68|    7.13| 0.86| 191| 196| 201| 194.1144| 197.4856|
|Gentoo    | 217.24| 203| 231| 119|    6.59| 0.60| 212| 216| 222| 216.0240| 218.3760|


# Analyze the Data

So above you can see a variety of variables in this dataset. We will focus on the variables `species` (our "x-axis" variable) and `flipper` (our "y-axis" variable) for this exercise.

One of the major benefits of using R for data visualization is that you can make reproducible workflows where your graphs can incorporate your most up-to-date statistical tests. So, let's run some basic analyses here.

Anova with `stats::aov()` and `car::Anova()`:


```r
# Fit data
flipper_fit <- stats::aov(flipper ~ species, data = df)
# Run anova
flipper_anova <- car::Anova(flipper_fit)
```

Calculate effect sizes with `effectsize::eta_squared()`, and import them into the ANOVA table:


```r
# Extract effect size (partial eta squared) from anova
flipper_anova_pes <- effectsize::eta_squared(flipper_anova,
                                             alternative="two.sided",
                                             verbose = FALSE,
                                             generalized= FALSE)
# Convert anova table into dataframe
flipper_anova <- data.frame(flipper_anova)

# import effect size estimates and confidence intervals to anova dataframe
flipper_anova$pes_ci95_lo <- flipper_anova_pes$CI_low
flipper_anova$pes_ci95_hi <- flipper_anova_pes$CI_high
flipper_anova$pes <- flipper_anova_pes$Eta2

# round all numeric columns to 2 decimal places
flipper_anova <- flipper_anova %>%
  dplyr::mutate_if(is.numeric, function(x) round(x, 2))

# display anova dataframe
knitr::kable(flipper_anova)
```



|          |   Sum.Sq|  Df| F.value| Pr..F.| pes_ci95_lo| pes_ci95_hi|  pes|
|:---------|--------:|---:|-------:|------:|-----------:|-----------:|----:|
|species   | 50525.88|   2|  567.41|      0|        0.74|         0.8| 0.77|
|Residuals | 14692.75| 330|      NA|     NA|        0.74|         0.8| 0.77|

Since we have a model now, we can extract the pairwise comparisons with `emmeans::emmeans()`:

The particular strength of using estimated marginal means is that it lets us compare levels of our dependent variable, while accounting for the influence of other covariates in the model. Not particularly relevant for a one-way design, but this method is pre-built to visualize the results of an ANCOVA.


```r
# Extract estimated marginal means
flipper_emmeans <- emmeans::emmeans(flipper_fit, specs = pairwise ~ species)

# convert estimated marginal mean contrasts to dataframe
flipper_emmeans_contrasts <- data.frame(flipper_emmeans$contrasts)

# Convert estimated marginal means to dataframe
flipper_emmeans_tidy <- data.frame(flipper_emmeans$emmeans)
```


Make a custom function, `merge_emmeans_summary()`, to merge summary and emmeans. Particularly useful for multivariate analyses:


```r
#merging regular summary stats with emmeans
merge_emmeans_summary <- function(summary_data,emmeans_tidy){
  summary_data$emmean <- emmeans_tidy$emmean
  summary_data$emmean_se <- emmeans_tidy$SE
  summary_data$emmean_loci <- emmeans_tidy$lower.CL
  summary_data$emmean_upci <- emmeans_tidy$upper.CL

  summary_data
}
```

Use `merge_emmeans_summary()` to combine with our basic `flipper_summary` into a single model-enhanced dataframe:


```r
# Order the dataframes based on dependent variables - Not necessary here, but good practice that helps for factorial designs
flipper_summary <- flipper_summary[order(flipper_summary$species), ]
flipper_emmeans_tidy <- flipper_emmeans_tidy[order(flipper_emmeans_tidy$species), ]

# merge dataframes
flipper_summary <- merge_emmeans_summary(summary_data = flipper_summary,
                                                  emmeans_tidy = flipper_emmeans_tidy)
# Round numeric values
flipper_summary <- flipper_summary %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_summary)
```



|species   |   mean| min| max|   n| std_dev|   se| y25| y50| y75|   loci|   upci| emmean| emmean_se| emmean_loci| emmean_upci|
|:---------|------:|---:|---:|---:|-------:|----:|---:|---:|---:|------:|------:|------:|---------:|-----------:|-----------:|
|Adelie    | 190.10| 172| 210| 146|    6.52| 0.54| 186| 190| 195| 189.04| 191.16| 190.10|      0.55|      189.02|      191.19|
|Chinstrap | 195.82| 178| 212|  68|    7.13| 0.86| 191| 196| 201| 194.11| 197.49| 195.82|      0.81|      194.23|      197.42|
|Gentoo    | 217.24| 203| 231| 119|    6.59| 0.60| 212| 216| 222| 216.02| 218.38| 217.24|      0.61|      216.03|      218.44|




Now we have our analysis data to work with! From here, we can get the data ready to actually work with. To start, we can make a custom function, `report_pval_full()`, that helps us convert p-values to our desired formatting.


```r
# rounding or abbreviating p-values
report_pval_full <- function(pval, italicize = TRUE) {
  ifelse(pval < .001, paste0(
    ifelse(italicize == TRUE, "*p*", "p"), " < .001"),
         paste0(
           ifelse(italicize == TRUE, "*p*", "p")," = ", ifelse(pval >= .01,weights::rd(pval,2),
                weights::rd(pval,3))
         )
         )
}
```



Create and prepare dataframe for pairwise comparisons


```r
# convert p-value to non-italicized version (for significance brackets)
flipper_emmeans_contrasts$p_no_it <-  report_pval_full(flipper_emmeans_contrasts$p.value, italicize = FALSE)

# convert p-value to italicized version (for in-text citations and captioning graph)
flipper_emmeans_contrasts$p.value <- report_pval_full(flipper_emmeans_contrasts$p.value)


flipper_emmeans_contrasts <- flipper_emmeans_contrasts %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_emmeans_contrasts)
```



|contrast           | estimate|   SE|  df| t.ratio|p.value    |p_no_it  |
|:------------------|--------:|----:|---:|-------:|:----------|:--------|
|Adelie - Chinstrap |    -5.72| 0.98| 330|   -5.84|*p* < .001 |p < .001 |
|Adelie - Gentoo    |   -27.13| 0.82| 330|  -32.92|*p* < .001 |p < .001 |
|Chinstrap - Gentoo |   -21.41| 1.01| 330|  -21.11|*p* < .001 |p < .001 |





# Vizualize the Data


## Set canvas

The metaphor I like to use with ggplot is that it's like painting. First you buy and setup your canvas, then you can start adding elements that get layered on top of each other - so the order of your ggplot elements matters!

![](pingu_paints.gif)

Now we will create our `ggplot()` canvas, which everything else will be layered upon. 



```r
canvas <- ggplot(data = df, # specify the dataframe that we want to pull variables from
                  aes(y = flipper, # our dependent/response/outcome variable 
                      x = species  # our grouping/independent/predictor variable
                      ))
```

We specify `cowplot::theme_half_open()` at this early stage because we want to override some arguments in this theme with some new elements later on.


```r
canvas <-  canvas +  
  cowplot::theme_half_open() # nice theme for publication

canvas # display our blank canvas
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-14-1.png" width="672" />
 <!-- +     ## set max for y-axis -->

## Add Mean and 95% CI

You could always just plot the mean and 95% CI (confidence intervals) and nothing else. However, like barplots, this comes at the cost of omitting the entire range of responses. We will use `geom_pointrange()` because it easily accepts data from an external summary dataframe:

- You can also customize `geom_pointrange()`, for example, with `fatten = 2, size = .4`
- Alternative: We can also use `stat_summary(fun.data = "mean_cl_normal")`. Does not require external dataframe, but I think creating summary data is good practice, and future-proofs this procedure for ANCOVAs.


<!-- This is basically what barplots/dynamite plots accomplish. It also shows one of my biggest issues with barplots: the defaults ignore the rest of the response range. If the means don't appear in a certain value range, these axis values aren't shown. The small range risks misrepresenting the overall data; if the groups have any sort of decent amount of variance, those observations won't even be indirectly represented by blank canvas space. -->


```r
canvas + # this is our previously-defined canvas object, it passes the appropriate variables and theme
  # This function puts out the dot-whisker for the mean + 95% CI
    geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                      ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-15-1.png" width="672" />


## Adding Density Slab

<!-- These issues of response ranges are made clear when we use density slabs.  -->

Density and violin plots are based on [kernel density](https://rkabacoff.github.io/datavis/Univariate.html#Kernel), basically, a smoothed histogram of the data. Here, we add a density slab with `ggdist::stat_slab()`:


```r
canvas  +
  # density slab
  ggdist::stat_slab() + # add a density slab
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-16-1.png" width="672" />

You can see the y-axis stretches far higher and lower (170 - 230) than it did previously (190 - ~210). The major strength of this density slab method is that it visualizes the spread and skew of the data, which by default forces us to visualize the entire range of observations. 

<!-- This can be especially important if we have meaningful thresholds and cutoffs for our response variable (example: in our data - is there a certain length of flipper where penguins start to become noticeably worse swimmers?). -->

<!-- Overall, I'm an advocate for the practice of showing our data over just trying to show group differences. -->

Perusing twitter conversations, some people do not like the aesthetic of a 'lopsided' density slab. On a more practical side, it becomes a bit more tricky to add elements like text for the mean in a legible way. To add some symmetry, we can change the density slab to violin using `ggdist::stat_slab(side = "both")`:


```r
canvas +  
  # density slab
  ggdist::stat_slab(
    side = "both") + # change the density slab to violin
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-17-1.png" width="672" />

## Fade and Style the Violin Plot


Many of us want to pick nice colors for our plots, in R, colors are defined with HEX codes (in this case, `nova_palette[1]` is  `#78AAA9`). 

You can find a [generator for colorblind-accessible color palettes here](https://venngage.com/tools/accessible-color-palette-generator), a useful tool for exploring [HEX colors here, complete with colorblindness simulator at the bottom of the page](https://www.colorhexa.com/78aaa9), and a [brief education primer, complete with colorblindness palette simulator, here](https://davidmathlogic.com/colorblind/).

Now we can color the violin geom with our desired Hex using `fill`. Notably, this only works when the `fill argument` is not inside of `aes()`:


```r
canvas +  
  # density slab
  ggdist::stat_slab(side = "both", 
                    fill = nova_palette[1]) + # color the violin geom with desired color
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-18-1.png" width="672" />


Instead of using boxplots, I like to fade my violins according to their quantile grouping. I find it makes the violin more informative, and has less visual clutter compared to the boxplot. We add the fading by modifying `ggdist::stat_slab()` with the argument, `fill_ramp = stat(level)`:


```r
canvas +  
  # density slab
  ggdist::stat_slab(side = "both", 
                    fill = nova_palette[1],  
                    aes(fill_ramp = stat(level))) +  # fade violins according to their quantile grouping
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-19-1.png" width="672" />

You can see by the legend that the fading matches to the inner 66% of the data, then the inner 95%, with no fading at the outer 5%. The inner 66% of the data holds no special meaning to me. I find that the inner/outer 50% is more intuitive . 

- Alternative: change from 66% to 68%, reflecting +/-1 standard deviation, giving a more direct parallel to a critical descriptive statistic (as well as cohen's d).

We can change the shaded quantiles from the inner 66% to 50% (and eliminate the 95% shading) using `ggdist::stat_slab(..., .width = c(.50, 1))`.  



```r
canvas +  
  # density slab
  ggdist::stat_slab(side = "both",  
                    aes(fill_ramp = stat(level)), 
                    fill = nova_palette[1],
                    .width = c(.50, 1)) +  # change quantiles for shading from 66% to 50% (and eliminate the 95% shading) 
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-20-1.png" width="672" />

The violins are quite wide, let's adjust their widths using `scale = .4`.


```r
canvas +  
  # density slab
  ggdist::stat_slab(side = "both",  
                    aes(fill_ramp = stat(level)), 
                    fill = nova_palette[1],
                    .width = c(.50, 1),
                    scale = .4) +  # adjust slab width
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-21-1.png" width="672" />

The fading quantiles are probably better discussed orally or as a figure note, not being directly relevant to any statistical tests. We can get rid of the legend element for fading quantiles with `guides(fill_ramp = "none")`. Here, we also assign this ggplot to a new object, `viofade`, so we can see new code additions in the chunks more easily:


```r
viofade <- canvas +  
  # density slab
  ggdist::stat_slab(side = "both",  
                    aes(fill_ramp = stat(level)), 
                    fill = nova_palette[1],
                    .width = c(.50, 1),
                    scale = .4) + 
  geom_pointrange(data = flipper_summary, # our externally-defined summary dataframe
                  aes(x = species,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  ))  +
  guides(fill_ramp = "none") # get rid of legend element for fading quantiles

viofade
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-22-1.png" width="672" />

This is a satisfactory version of the complete viofade, but like any makeover, there is more styling we can do.


## Adding Extra Elements

I find that having the written values of the different means makes the comparisons a bit more concrete. Again, you could use` stat_summary()`, but we will keep using our `flipper_summary` data, this time with `geom_text()`.



```r
viofade +
  geom_text(data = flipper_summary, 
            aes(x = species, 
                y = mean, 
                label = round(mean,1)),
            color="black", 
            size = 4, 
            vjust = 5)
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-23-1.png" width="672" />

```r
# You could use this function if you want:
   # stat_summary(aes(label = round(..y..,1)), 
   #                         fun = mean, 
   #                         geom = "text",vjust = 5)
```

Our `viofade` object does not include raw data. This is intentional because objects such as rainclouds, fadeclouds, and shadeplots tend to look very noisy after I add significance brackets. However, I still want people to see the sample size for our groups, which we can do using `geom_text()` in combination with our `flipper_summary` dataframe. 

- Alternative: Use `EnvStats::stat_n_text()` if you don't want to use a summary dataframe, although I don't think this method works with a factorial design (i.e., you can't "dodge" the positions of these elements)
- If you want your sample sizes under your axis values, [check out this discussion](https://stackoverflow.com/questions/40102613/ggplot2-adding-sample-size-information-to-x-axis-tick-labels)


```r
viofade_text <- viofade +
  geom_text(data = flipper_summary, aes(x = species, 
                                        y = mean, 
                                        label = round(mean,1)),
            color="black", 
            size = 4, 
            vjust = 5) +
  geom_text(data = flipper_summary, 
            aes(label = paste("n =", n), 
                y = flipper_summary[which.min(flipper_summary$min),]$mean -
                  3*flipper_summary[which.min(flipper_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 3, 
            color = "grey60") 

# You could use this for sample sizes if you want
 # EnvStats::stat_n_text(color = "grey60")

viofade_text
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-24-1.png" width="672" />

## Adding Test Statistics

Especially for presentations and posters, it can be useful to have significance tests embedded directly in the plot. This is where we really benefit from having run some analyses beforehand.


### Create Reporting Functions

Using our pre-run analyses is made much easier if we write a helper function, so here we create a custom function, `report_tidy_anova_etaci()` that facilitate reporting of significance and effect size statistics for our ANOVA (for eta squared).




```r
report_tidy_anova_etaci <- function(tidy_frame, # your tidy anova dataframe
                                    term, # the predictor in your tidy anova dataframe
                                    effsize = TRUE, # display the effect size
                                    ci95 = TRUE, # display the 95% CI
                                    ci.lab = TRUE, # display the "95% CI" label
                                    teststat = TRUE, # display the F-score and degrees of freedom
                                    pval = TRUE # display the p-value
                                    ){
  text <- paste0(
    ifelse(effsize == TRUE, paste0(
    "\u03b7^2^ = ", #unicode for eta square
    round(as.numeric(tidy_frame[term,"pes"]),2)), 
    ""),
    ifelse(ci95 == TRUE, paste0(
                 ifelse(ci.lab == TRUE,
                        paste0(", 95% CI ["), " ["),   
                 round(as.numeric(tidy_frame[term,"pes_ci95_lo"]),2), ", ", 
                 round(as.numeric(tidy_frame[term,"pes_ci95_hi"]),2),"]"),
           ""),
    ifelse(teststat == TRUE, 
           paste0(", *F*(", tidy_frame[term,"Df"],
                  ", ", tidy_frame["Residuals","Df"], ") = ",
                  round(as.numeric(tidy_frame[term,"F.value"]), 2)), 
           ""),
    ifelse(pval == TRUE, paste0(
                 ", ",  report_pval_full(tidy_frame[term,"Pr..F."])),
           "")
    )
  
  return(text)
}
```

We can add our omnibus ANOVA test by embedding our custom function in the `labs` element, `labs(subtitle = report_tidy_anova_etaci(flipper_anova,"species"))`:


```r
viofade_text + 
  labs(subtitle = report_tidy_anova_etaci(flipper_anova,"species"))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-26-1.png" width="672" />

By default, ggplot doesn't allow markdown styling, so we have weird stars instead of our desired formatting. We can enable markdown styling in the subtitle Using `theme(plot.subtitle = ggtext::element_markdown())`


```r
viofade_text_stats <- viofade_text + 
  labs(subtitle = report_tidy_anova_etaci(flipper_anova,"species")) + 
  theme(plot.subtitle = ggtext::element_markdown())

viofade_text_stats
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-27-1.png" width="672" />

We can also customize our anova extraction function, `report_tidy_anova_etaci()` to show less information in the subtitle.


```r
viofade_text + 
  labs(subtitle = report_tidy_anova_etaci(flipper_anova, 
                                          "species", 
                                          teststat = FALSE, 
                                          pval = FALSE)) + 
  theme(plot.subtitle = ggtext::element_markdown())
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-28-1.png" width="672" />

## Add Significance Brackets


Adding significance brackets using the function `ggpubr::geom_bracket()`, and our previously-defined object `flipper_emmeans_contrasts`. You'll have to manually select and place the appropriate contrast.



```r
viofade_text_stats_bracket1 <- viofade_text_stats +
  ggpubr::geom_bracket( 
                       tip.length = 0.02, # the downard "tips" of the bracket
                       vjust = 0, # moves your text label (in this case, the p-value)
                       xmin = 1, #starting point for the bracket
                       xmax = 2, # ending point for the bracket
                       y.position = 220, # vertical location of the bracket
                       label.size = 2.5, # size of your bracket text
                       label = paste0(flipper_emmeans_contrasts[flipper_emmeans_contrasts$contrast == 
                                                                  'Adelie - Chinstrap', "p_no_it"]) # content of your bracket text
  )

viofade_text_stats_bracket1
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-29-1.png" width="672" />

You can also add some additional text to the labels for your significance brackets using `ggpubr::geom_bracket(..., label = paste0(...))`



```r
viofade_text_stats_bracket2 <- viofade_text_stats_bracket1 +
    ggpubr::geom_bracket( 
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = 2, 
                       xmax = 3, 
                       y.position = 227 ,
                       label.size = 2.5,
                       label = paste0("My hypothesis, ", 
                                      flipper_emmeans_contrasts[flipper_emmeans_contrasts$contrast == 
                                                                  'Chinstrap - Gentoo', "p_no_it"]))
```

Style on the axis titles with `theme()`, `ylab()`, and `xlab()` 


```r
viofade_text_stats_bracket2 +
  theme(axis.title = element_text(face="bold")) + 
  ylab("Flipper length (mm)") +
  xlab("Species")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-31-1.png" width="672" />




# Now Try a Factorial Design



## Summarize the Data

Our data


```r
flipper_fact_summary <- make_summary(data = df, dv = flipper, grouping1 = species, grouping2 = sex)

knitr::kable(flipper_fact_summary)
```



|species   |sex    |   mean| min| max|  n| std_dev|   se| y25| y50| y75|     loci|     upci|
|:---------|:------|------:|---:|---:|--:|-------:|----:|---:|---:|---:|--------:|--------:|
|Adelie    |female | 187.79| 172| 202| 73|    5.60| 0.65| 185| 188| 191| 186.5260| 189.0740|
|Adelie    |male   | 192.41| 178| 210| 73|    6.60| 0.77| 189| 193| 197| 190.8908| 193.9092|
|Chinstrap |female | 191.74| 178| 202| 34|    5.75| 0.99| 187| 192| 196| 189.7596| 193.6404|
|Chinstrap |male   | 199.91| 187| 212| 34|    5.98| 1.02| 196| 200| 203| 197.9008| 201.8992|
|Gentoo    |female | 212.71| 203| 222| 58|    3.90| 0.51| 210| 212| 215| 211.7004| 213.6996|
|Gentoo    |male   | 221.54| 208| 231| 61|    5.67| 0.73| 218| 221| 225| 220.0692| 222.9308|



## Analyze the Data



```r
# Fit data
flipper_fact_fit <- stats::aov(flipper ~ species*sex, data = df)
# Run anova
flipper_fact_anova <- car::Anova(flipper_fact_fit, type = 2)
```


Extract estimated marginal means with `emmeans::emmeans()` and merge with basic summary dataframe with our custom `merge_emmeans_summary()` function.


```r
# Extract estimated marginal means
flipper_fact_emmeans <- emmeans::emmeans(flipper_fact_fit, specs = pairwise ~ species:sex)


# Convert estimated marginal means to dataframe
flipper_fact_emmeans_tidy <- data.frame(flipper_fact_emmeans$emmeans)


flipper_fact_summary <- flipper_fact_summary[order(flipper_fact_summary$species, 
                                                   flipper_fact_summary$sex), ]

flipper_fact_emmeans_tidy <- flipper_fact_emmeans_tidy[order(flipper_fact_emmeans_tidy$species,
                                                             flipper_fact_emmeans_tidy$sex), ]


flipper_fact_summary <- merge_emmeans_summary(summary_data = flipper_fact_summary,
                                                  emmeans_tidy = flipper_fact_emmeans_tidy)

flipper_fact_summary <- flipper_fact_summary %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_fact_summary)
```



|species   |sex    |   mean| min| max|  n| std_dev|   se| y25| y50| y75|   loci|   upci| emmean| emmean_se| emmean_loci| emmean_upci|
|:---------|:------|------:|---:|---:|--:|-------:|----:|---:|---:|---:|------:|------:|------:|---------:|-----------:|-----------:|
|Adelie    |female | 187.79| 172| 202| 73|    5.60| 0.65| 185| 188| 191| 186.53| 189.07| 187.79|      0.66|      186.49|      189.10|
|Adelie    |male   | 192.41| 178| 210| 73|    6.60| 0.77| 189| 193| 197| 190.89| 193.91| 192.41|      0.66|      191.11|      193.71|
|Chinstrap |female | 191.74| 178| 202| 34|    5.75| 0.99| 187| 192| 196| 189.76| 193.64| 191.74|      0.97|      189.83|      193.64|
|Chinstrap |male   | 199.91| 187| 212| 34|    5.98| 1.02| 196| 200| 203| 197.90| 201.90| 199.91|      0.97|      198.00|      201.82|
|Gentoo    |female | 212.71| 203| 222| 58|    3.90| 0.51| 210| 212| 215| 211.70| 213.70| 212.71|      0.74|      211.25|      214.17|
|Gentoo    |male   | 221.54| 208| 231| 61|    5.67| 0.73| 218| 221| 225| 220.07| 222.93| 221.54|      0.72|      220.12|      222.97|

Calculate effect sizes with `effectsize::eta_squared()`


```r
flipper_fact_anova_pes <- effectsize::eta_squared(flipper_fact_anova,
                                             alternative="two.sided",
                                             verbose = FALSE,
                                             generalized= FALSE)

flipper_fact_anova <- data.frame(flipper_fact_anova)

flipper_fact_anova[1:3,"pes_ci95_lo"] <- flipper_fact_anova_pes$CI_low
flipper_fact_anova[1:3,"pes_ci95_hi"] <- flipper_fact_anova_pes$CI_high
flipper_fact_anova[1:3,"pes"] <- flipper_fact_anova_pes$Eta2


flipper_fact_anova <- flipper_fact_anova %>%
  dplyr::mutate_if(is.numeric, function(x) round(x, 3))

knitr::kable(flipper_fact_anova)
```



|            |    Sum.Sq|  Df| F.value| Pr..F.| pes_ci95_lo| pes_ci95_hi|   pes|
|:-----------|---------:|---:|-------:|------:|-----------:|-----------:|-----:|
|species     | 50185.027|   2| 784.583|  0.000|       0.798|       0.851| 0.828|
|sex         |  3905.604|   1| 122.119|  0.000|       0.195|       0.347| 0.272|
|species:sex |   329.042|   2|   5.144|  0.006|       0.003|       0.073| 0.031|
|Residuals   | 10458.107| 327|      NA|     NA|          NA|          NA|    NA|


## Vizualize the Data


```r
viofade_fact <- ggplot(data = df,
                       aes(y = flipper, # our dependent/response/outcome variable 
                           x = species,  # our grouping/independent/predictor variable
                           fill = sex)) +  # our third grouping/independent/interaction variable
  ggdist::stat_slab(side = "both",  
                    aes(fill_ramp = stat(level)),
                    .width = c(.50, 1),
                    scale = .4,
                    position = position_dodge(width = .7)) +
  scale_fill_manual(values = nova_palette) +
  # stat_summary(fun.data = "mean_cl_normal", 
  #              show.legend = F, 
  #              color = "black", 
  #              position = position_dodge(width = .7))  + 
  geom_pointrange(data = flipper_fact_summary,
                  aes(x = species, 
                      y = emmean, 
                      ymin = emmean_loci, 
                      ymax = emmean_upci),
                  position = position_dodge(.7), show.legend = F) +
  geom_text(data = flipper_fact_summary, aes(x = species, 
                                             y = emmean, 
                                             label = round(emmean,1)),
            color="black", 
            size = 2.5, 
            vjust = 5, 
            position = position_dodge(width = .7)) +
  # stat_summary(aes(label=round(..y..,0)), 
  #              fun=mean, geom="text",  
  #              position = position_dodge(width = .7), 
  #              size=2.5, 
  #              vjust = 5) +
  geom_text(data = flipper_fact_summary, 
            aes(label = paste("n =", n), 
                y = flipper_fact_summary[which.min(flipper_fact_summary$min),]$mean -
                  3*flipper_fact_summary[which.min(flipper_fact_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 2, 
            color = "grey60", 
            position = position_dodge(width = .7)) +
  guides(fill_ramp = "none") +
  scale_y_continuous(expand = expansion(mult = c(0.03, 0))) +
  cowplot::theme_half_open() +  
  theme(axis.title = element_text(face="bold"), 
        legend.title = element_text(face="bold")) + 
  labs(x = "Species", 
       y = "Flipper length (mm)", 
       fill = "Sex")


viofade_fact
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-36-1.png" width="672" />



```r
# convert estimated marginal mean contrasts to dataframe
flipper_fact_emmeans_contrasts <- data.frame(flipper_fact_emmeans$contrasts)

# convert p-value to non-italicized version (for significance brackets)
flipper_fact_emmeans_contrasts$p_no_it <-  report_pval_full(flipper_fact_emmeans_contrasts$p.value, italicize = FALSE)

# convert p-value to italicized version (for in-text citations and captioning graph)
flipper_fact_emmeans_contrasts$p.value <- report_pval_full(flipper_fact_emmeans_contrasts$p.value)


flipper_fact_emmeans_contrasts <- flipper_fact_emmeans_contrasts %>%
  mutate_if(is.numeric, function(x) round(x, 2))

knitr::kable(flipper_fact_emmeans_contrasts)
```



|contrast                          | estimate|   SE|  df| t.ratio|p.value    |p_no_it  |
|:---------------------------------|--------:|----:|---:|-------:|:----------|:--------|
|Adelie female - Chinstrap female  |    -3.94| 1.17| 327|   -3.36|*p* = .01  |p = .01  |
|Adelie female - Gentoo female     |   -24.91| 0.99| 327|  -25.04|*p* < .001 |p < .001 |
|Adelie female - Adelie male       |    -4.62| 0.94| 327|   -4.93|*p* < .001 |p < .001 |
|Adelie female - Chinstrap male    |   -12.12| 1.17| 327|  -10.32|*p* < .001 |p < .001 |
|Adelie female - Gentoo male       |   -33.75| 0.98| 327|  -34.40|*p* < .001 |p < .001 |
|Chinstrap female - Gentoo female  |   -20.97| 1.22| 327|  -17.17|*p* < .001 |p < .001 |
|Chinstrap female - Adelie male    |    -0.68| 1.17| 327|   -0.58|*p* = .99  |p = .99  |
|Chinstrap female - Chinstrap male |    -8.18| 1.37| 327|   -5.96|*p* < .001 |p < .001 |
|Chinstrap female - Gentoo male    |   -29.81| 1.21| 327|  -24.63|*p* < .001 |p < .001 |
|Gentoo female - Adelie male       |    20.30| 0.99| 327|   20.40|*p* < .001 |p < .001 |
|Gentoo female - Chinstrap male    |    12.80| 1.22| 327|   10.47|*p* < .001 |p < .001 |
|Gentoo female - Gentoo male       |    -8.83| 1.04| 327|   -8.52|*p* < .001 |p < .001 |
|Adelie male - Chinstrap male      |    -7.50| 1.17| 327|   -6.39|*p* < .001 |p < .001 |
|Adelie male - Gentoo male         |   -29.13| 0.98| 327|  -29.69|*p* < .001 |p < .001 |
|Chinstrap male - Gentoo male      |   -21.63| 1.21| 327|  -17.87|*p* < .001 |p < .001 |




```r
viofade_fact_bracket <- viofade_fact + 
ggpubr::geom_bracket(inherit.aes = FALSE, # necessary for factorial design
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = 1.175, # You need to play with these by hand
                       xmax = 1.825, 
                       y.position = 210 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Chinstrap female - Adelie male', "p_no_it"])) +
  ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = .825, 
                       xmax = 1.175, 
                       y.position = 217 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                       'Adelie female - Adelie male', "p_no_it"])) +
  ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = 0.02, 
                       vjust = 0,
                       xmin = .825, 
                       xmax = 2.175, 
                       y.position = 225 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Adelie female - Chinstrap male', "p_no_it"])) +
    ggpubr::geom_bracket(inherit.aes = FALSE, 
                       tip.length = -0.02, 
                       vjust = 0.6,
                       xmin = 2.175, 
                       xmax = 3.175, 
                       y.position = 185 ,
                       label.size = 2.1,
                       label = paste0(flipper_fact_emmeans_contrasts[flipper_fact_emmeans_contrasts$contrast == 
                                                                  'Chinstrap male - Gentoo male', "p_no_it"])
  )
  
  
viofade_fact_bracket
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-38-1.png" width="672" />


<details>
<summary> <b> Click here if you want an ultra-stat-detailed plot</b> </summary>

![](pingu_shuffle.gif)

Not exactly the amount of information I would put in the graph, but you can really get your analysis fully integrated with the plot, just by using the same subtitle function as before:


```r
 viofade_fact_bracket +
    labs(subtitle = paste0("**species**: ", report_tidy_anova_etaci(flipper_fact_anova,"species"), "<br>",
                           "**sex**: ", report_tidy_anova_etaci(flipper_fact_anova,"sex"), "<br>",
                           "**species*sex**: ",report_tidy_anova_etaci(flipper_fact_anova,"species:sex"))
         )  + 
  theme(plot.subtitle = ggtext::element_markdown(size = 10))
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-39-1.png" width="672" />


</details>

# Now Compare Just Two Groups

Use our workhorse `make_summary()`:


```r
flipper_sex_summary <- make_summary(data = df, dv = flipper, grouping1 = sex)

knitr::kable(flipper_sex_summary)
```



|sex    |   mean| min| max|   n| std_dev|   se| y25| y50| y75|     loci|     upci|
|:------|------:|---:|---:|---:|-------:|----:|---:|---:|---:|--------:|--------:|
|female | 197.36| 172| 222| 165|   12.50| 0.97| 187| 193| 210| 195.4988| 199.3012|
|male   | 204.51| 178| 231| 168|   14.55| 1.12| 193| 200| 219| 202.3048| 206.6952|

Run t a between-subjects t-test with `t.test()`, extract and format it with `report::report()` and `data.frame()`, and clean the names with `janitor::clean_names()`:


```r
flipper_sex_ttest <- t.test(flipper ~ sex, data=df) %>%
  report::report() %>%
  data.frame() %>%
  janitor::clean_names()

knitr::kable(flipper_sex_ttest)
```



|parameter |group | mean_group1| mean_group2| difference|   ci|    ci_low|   ci_high|         t| df_error|       p|method                  |alternative |          d|   d_ci_low|  d_ci_high|
|:---------|:-----|-----------:|-----------:|----------:|----:|---------:|---------:|---------:|--------:|-------:|:-----------------------|:-----------|----------:|----------:|----------:|
|flipper   |sex   |    197.3636|     204.506|  -7.142316| 0.95| -10.06481| -4.219821| -4.807866| 325.2784| 2.3e-06|Welch Two Sample t-test |two.sided   | -0.5331566| -0.7539304| -0.3115901|

`report_tidy_t()` is another useful custom function for doing some in-text or in-plot reporting of a t-test. `report::report_statistics()` is also a notable function, but I haven't figured out how to selectively extract the elements. Now we create the `report_tidy_t()` function:


```r
report_tidy_t <- function(tidy_frame, 
                          ci.lab = TRUE, 
                          test.stat = FALSE, 
                          point = TRUE){
  
  text <- paste0(ifelse(point==TRUE,paste0("*d* = ",
                                           round(tidy_frame$d,2)),""
                        ), 
                 ifelse(ci.lab == TRUE,
                        paste0(", 95% CI [", round(tidy_frame$d_ci_low  ,2), ", ", 
                               round(tidy_frame$d_ci_high  ,2),"]"),
                        paste0(" [", round(tidy_frame$d_ci_low  ,2), ", ", 
                               round(tidy_frame$d_ci_high  ,2),"]")),
                 
                 ifelse(test.stat == TRUE,paste0(", *t*","(", 
                                                 round(tidy_frame$df_error, 2) 
                                                 ,")"," = ", round(tidy_frame$t,2)),""),
                 ", *p* ", 
                 
                 ifelse(tidy_frame$p < .001, "< .001",
                        ifelse(tidy_frame$p > .01,paste("=", tidy_frame$p %>% round(2)),
                               paste("=", tidy_frame$p %>% round(3)))
                        )
                 ) 
  return(text)
}
```

Now we can specify the whole plot, using everything we've learned:


```r
ggplot(data = df, # specify the dataframe that we want to pull variables from
       aes(y = flipper, # our dependent/response/outcome variable 
           x = sex  # our grouping/independent/predictor variable
       )) +  
  # insert faded violin slab
  ggdist::stat_slab(side = "both",  # turn from slab to violin
                    aes(fill_ramp = stat(level)),  # specify shading
                    fill = nova_palette[1], # specify used to fill violin
                    .width = c(.50, 1), # specify shading quantiles
                    scale = .4) + # change size of violin
  # insert mean text
  geom_text(data = flipper_sex_summary, # our externally-defined summary dataframe
            aes(x = sex, # x-position of geom_text
                y = mean, # y-position of geom_text
                label = round(mean,1)), # actual content
            color="black", 
            size = 4, # size of text
            vjust = 3.5) + # some vertical nudging downwards
  # Insert mean and 95% confidence interval
  geom_pointrange(data = flipper_sex_summary, # our externally-defined summary dataframe
                  aes(x = sex,  # our independent variable
                      y = mean, # our outcome/dependent variable
                      ymin = loci,  # lower-bound confidence interval
                      ymax = upci # upper-bound confidence interval
                  )) +
  # insert sample sizes
  geom_text(data = flipper_sex_summary, # specify our custom dataframe
            aes(label = paste("n =", n), # the actual text content
                y = flipper_summary[which.min(flipper_summary$min),]$mean -
                  3*flipper_summary[which.min(flipper_summary$min),]$std_dev), #  dynamically set the location of sample size (3 standard deviations below the mean of the lowest-scoring group)
            size = 3,  # size of text
            color = "grey60") +
  guides(fill_ramp = "none") + # get rid of legend element for fading quantiles
  cowplot::theme_half_open() + # nice theme for publication
  labs(subtitle = report_tidy_t(flipper_sex_ttest, test.stat = T)) + # test statistic 
  theme(plot.subtitle = ggtext::element_markdown()) + # enable markdown styling 
  theme(axis.title = element_text(face="bold")) + #bold axis titles
  ylab("Flipper length (mm)") + # change y-axis label 
  xlab("Sex") # change x-axis label
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-43-1.png" width="672" />



# Conclusion

There are lots of ways and functions to analyze and visualize your data. The aim of this post is to guide readers (my future self included) through a single, seamless workflow to bring data products together in a way that you can readily publish in reports, journal articles, theses, posters, presentations, and anything else. See [my post on faded dotplots and shadeplots](https://dallasnova.rbind.io/post/creating-simple-and-transparent-data-graphs-using-faded-dotplots-and-shadeplots/#comparing-to-alternatives) to see a variety of plotting alternatives.

Thanks for reading. I know we all have things to do and places to be.

![](penguin_commute.gif)

# Loaded Packages




Bates D, Mächler M, Bolker B, Walker S (2015). "Fitting Linear Mixed-Effects Models Using lme4." _Journal of Statistical Software_, *67*(1), 1-48. doi: 10.18637/jss.v067.i01 (URL: https://doi.org/10.18637/jss.v067.i01).
 
Bates D, Maechler M, Jagan M (2023). _Matrix: Sparse and Dense Matrix Classes and Methods_. R package version 1.5-4, <URL: https://CRAN.R-project.org/package=Matrix>.
 
Ben-Shachar MS, Lüdecke D, Makowski D (2020). "effectsize: Estimation of Effect Size Indices and Standardized Parameters." _Journal of Open Source Software_, *5*(56), 2815. doi: 10.21105/joss.02815 (URL: https://doi.org/10.21105/joss.02815), <URL: https://doi.org/10.21105/joss.02815>.
 
Firke S (2023). _janitor: Simple Tools for Examining and Cleaning Dirty Data_. R package version 2.2.0, <URL: https://CRAN.R-project.org/package=janitor>.
 
Fox J, Weisberg S (2019). _An R Companion to Applied Regression_, Third edition. Sage, Thousand Oaks CA. <URL: https://socialsciences.mcmaster.ca/jfox/Books/Companion/>.
 
Fox J, Weisberg S, Price B (2022). _carData: Companion to Applied Regression Data Sets_. R package version 3.0-5, <URL: https://CRAN.R-project.org/package=carData>.
 
Grolemund G, Wickham H (2011). "Dates and Times Made Easy with lubridate." _Journal of Statistical Software_, *40*(3), 1-25. <URL: https://www.jstatsoft.org/v40/i03/>.
 
Harrell Jr F (2023). _Hmisc: Harrell Miscellaneous_. R package version 5.1-0, <URL: https://CRAN.R-project.org/package=Hmisc>.
 
Horst AM, Hill AP, Gorman KB (2020). _palmerpenguins: Palmer Archipelago (Antarctica) penguin data_. doi: 10.5281/zenodo.3960218 (URL: https://doi.org/10.5281/zenodo.3960218), R package version 0.1.0, <URL: https://allisonhorst.github.io/palmerpenguins/>.
 
Kassambara A (2023). _ggpubr: 'ggplot2' Based Publication Ready Plots_. R package version 0.6.0, <URL: https://CRAN.R-project.org/package=ggpubr>.
 
Kay M (2023). _ggdist: Visualizations of Distributions and Uncertainty_. doi: 10.5281/zenodo.3879620 (URL: https://doi.org/10.5281/zenodo.3879620), R package version 3.3.0, <URL: https://mjskay.github.io/ggdist/>.
 
Lenth R (2023). _emmeans: Estimated Marginal Means, aka Least-Squares Means_. R package version 1.8.6, <URL: https://CRAN.R-project.org/package=emmeans>.
 
Makowski D, Lüdecke D, Patil I, Thériault R, Ben-Shachar M, Wiernik B (2023). "Automated Results Reporting as a Practical Tool to Improve Reproducibility and Methodological Best Practices Adoption." _CRAN_. <URL: https://easystats.github.io/report/>.
 
Millard SP (2013). _EnvStats: An R Package for Environmental Statistics_. Springer, New York. ISBN 978-1-4614-8455-4, <URL: https://www.springer.com>.
 
Müller K, Wickham H (2023). _tibble: Simple Data Frames_. R package version 3.2.1, <URL: https://CRAN.R-project.org/package=tibble>.
 
Pasek J, Tahk wsafA, Culter scmfRAcbG, Schwemmle. M (2021). _weights: Weighting and Weighted Statistics_. R package version 1.0.4, <URL: https://CRAN.R-project.org/package=weights>.
 
R Core Team (2021). _R: A Language and Environment for Statistical Computing_. R Foundation for Statistical Computing, Vienna, Austria. <URL: https://www.R-project.org/>.
 
Singmann H, Bolker B, Westfall J, Aust F, Ben-Shachar M (2023). _afex: Analysis of Factorial Experiments_. R package version 1.3-0, <URL: https://CRAN.R-project.org/package=afex>.
 
Wickham H (2016). _ggplot2: Elegant Graphics for Data Analysis_. Springer-Verlag New York. ISBN 978-3-319-24277-4, <URL: https://ggplot2.tidyverse.org>.
 
Wickham H (2022). _stringr: Simple, Consistent Wrappers for Common String Operations_. R package version 1.5.0, <URL: https://CRAN.R-project.org/package=stringr>.
 
Wickham H (2023). _forcats: Tools for Working with Categorical Variables (Factors)_. R package version 1.0.0, <URL: https://CRAN.R-project.org/package=forcats>.
 
Wickham H, Averick M, Bryan J, Chang W, McGowan LD, François R, Grolemund G, Hayes A, Henry L, Hester J, Kuhn M, Pedersen TL, Miller E, Bache SM, Müller K, Ooms J, Robinson D, Seidel DP, Spinu V, Takahashi K, Vaughan D, Wilke C, Woo K, Yutani H (2019). "Welcome to the tidyverse." _Journal of Open Source Software_, *4*(43), 1686. doi: 10.21105/joss.01686 (URL: https://doi.org/10.21105/joss.01686).
 
Wickham H, François R, Henry L, Müller K, Vaughan D (2023). _dplyr: A Grammar of Data Manipulation_. R package version 1.1.2, <URL: https://CRAN.R-project.org/package=dplyr>.
 
Wickham H, Henry L (2023). _purrr: Functional Programming Tools_. R package version 1.0.1, <URL: https://CRAN.R-project.org/package=purrr>.
 
Wickham H, Hester J, Bryan J (2023). _readr: Read Rectangular Text Data_. R package version 2.1.4, <URL: https://CRAN.R-project.org/package=readr>.
 
Wickham H, Vaughan D, Girlich M (2023). _tidyr: Tidy Messy Data_. R package version 1.3.0, <URL: https://CRAN.R-project.org/package=tidyr>.
 
Wilke C (2020). _cowplot: Streamlined Plot Theme and Plot Annotations for 'ggplot2'_. R package version 1.1.1, <URL: https://CRAN.R-project.org/package=cowplot>.
 
Wilke C, Wiernik B (2022). _ggtext: Improved Text Rendering Support for 'ggplot2'_. R package version 0.1.2, <URL: https://CRAN.R-project.org/package=ggtext>.

![](see_nothing.gif)