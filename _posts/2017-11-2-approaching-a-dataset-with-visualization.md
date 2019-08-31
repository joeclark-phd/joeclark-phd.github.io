---
title: Approaching a dataset with visualization
---

I'd like to give my students some simple guidelines for how to use data visualization to look at a new dataset.  What to do first, second, and so on.  Here's what I'm going to suggest.

## Examine individual variables

First, take one variable at a time.  Which are the most important ones, considering the audience and the purpose of your work?  What are the mean, median, and mode?  Accordingly, your first *visualizations* may be histograms or box-and-whisker plots, maybe Pareto diagrams.  These go beyond the statistics by showing us the overall "shape" of the distributions, revealing things like Normal distributions, skewness, and fat or thin tails.

## Compare subsets of the data on single variables

Once you have a sense of how the data is distributed overall, you can begin slicing and dicing it by some categorical dimension(s).  This can be as simple as a bar chart comparing a single statistic across categories, or it can be a small-multiples diagram that repeats a histogram once for each subset of the data.  **Comparison** is the name of the game here.  How do men differ from women, or how does Canada differ from the UK, or how does 2016 differ from 2017, on your key variable?

## Changes over time

Line graphs, particularly with *time* as the X axis, are very meaningful to us perhaps because storytelling is a natural way that we humans think.  Line graphs are easy to generate in software like Excel or Tableau, and they transform what was a single point metric into a story of rise and fall and seasonality.  These are particularly enriching visualizations if your data has a time dimension.

## Relationships between continuous variables

As we begin to get a handle on the data one variable at a time, we may begin to form theories to explain or predict it, theories that can be represented as relationships between variables.  Is television viewing related to socioeconomic status?  Are Red Sox wins correlated with the weather on game day?  Does student satisfaction depend on class size?  The classic data visualization to compare two such variables is the XY scatterplot, with each data point plotted on both dimensions.  Lines or curves that indicate significant relationships can be seen, if they exist, and so can outliers.

Incidentally, the list above could also serve as the outline to a data visualization term paper.  I'm just sayin'...
