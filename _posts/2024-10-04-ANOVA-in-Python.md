---
layout: post
title: "ANOVA in Python"
author: Gracie Stirk
description: A step by step guide to doing an ANOVA analysis in Python
image: "/assets/images/pretty-trees.jpg"
---

--- 
---
### ANOVA in Python
ANOVA is a catchy name for ANalysis Of VAriance. It's a useful tool for analyzing multiple means at the same time, specifically three or more means. This blog post is going to assume that the reader has a basic understanding of hypothesis testing and the python language.

For now, we'll do a one-way ANOVA. Let's say I want to see if actors in certain genres of film have different heights on average. Maybe I want to see if action heros are typically portrayed by taller actors on average. For the purpose of this post, I made up some data around this.

## Getting Started

The first step in this process will be to import the python libraries we need.

```python
import numpy as np
import pandas as pd
import scipy.stats as sp
from tabulate import tabulate
```

We'll primarily be using scipy.stats for its useful functions relating to ANOVA and variance.

Next to read in the dataset:

```python
data = pd.read_csv('actor_height.csv')
print(data)
```

For this analysis, we'll be using the genre of film the actor stars in as our explanatory variable and we'll be using the height (in inches) of the actor as our response variable.

It's good practice to make sure that it's appropriate to do this analysis, so lets make sure that each group is approximately normal.

## Checking Assumptions

First we'll sort the data into four different groups.

```python
romance = data[data.loc[:, 'Genre'] == 'Romance']
horror = data[data.loc[:, 'Genre'] == 'Horror']
comedy = data[data.loc[:, 'Genre'] == 'Comedy']
action = data[data.loc[:, 'Genre'] == 'Action']
```

Next we'll check the skewness with a function in scipy.stats.

```python
rom_skew = sp.skew(romance.loc[:,'Height'])
hor_skew = sp.skew(horror.loc[:,'Height'])
com_skew = sp.skew(comedy.loc[:,'Height'])
act_skew = sp.skew(action.loc[:,'Height'])
```
From this we can see that the skewness is:
```python
skews = [["Romance", rom_skew],["Horror", hor_skew],["Comedy", com_skew],["Action", act_skew]]

print(tabulate(skews, headers=["Genre", "Skewness"], tablefmt="grid"))
```

There are many different methods to check if data is approximately normal, but checking if the skewness is between -0.5 and +0.5 is the technique I'm most familiar with. We can see from the above table that all of the genres are approximately normal. None of them are particularly left or right-skewed. 

We also need to check the standard deviations to make sure they're approximately equal.

We can use a similar technique to what we did with skewness. The library scipy.stats can calculate a sample standard deviation for each category.

```python
rom_sd = sp.tstd(romance.loc[:,'Height'])
hor_sd = sp.tstd(horror.loc[:,'Height'])
com_sd = sp.tstd(comedy.loc[:,'Height'])
act_sd = sp.tstd(action.loc[:,'Height'])
```

And we can look at the results in another table:

```python
stds = [["Romance", rom_sd],["Horror", hor_sd],["Comedy", com_sd],["Action", act_sd]]

print(tabulate(stds, headers=["Genre", "Standard Deviations"], tablefmt="grid"))
```

The general rule I've learned to check this assumption is that the maximum standard deviation should not be more than 2x the smallest standard deviation. The horror genre has a standard deviation of 1.61, and then comedy genre has a standard deviation of 3.07, so that assumption is met as well!

So now we can move on to the fun stuff!

## The Fun Stuff

The first thing we should do is check to see if we need to reject our null hypothesis! Our null hypothesis is that the mean heights for all actors is the same across all genres, and our alternative hypothesis is that at least one is different.

The scipy.stats library has a function built in to do a one-way ANOVA analysis that we can use this way to get the F-statistic and the p-value.

```python
F_stat, p_val = sp.f_oneway(romance['Height'],horror['Height'],comedy['Height'],action['Height'])
print("The F-Statistic is " + str(F_stat) + ".")
print("The p-value is " + str(p_val) + ".")
```