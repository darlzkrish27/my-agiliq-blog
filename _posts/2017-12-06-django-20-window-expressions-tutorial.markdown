---
layout: post
title: "Django 2.0 Window expressions tutorial"
date:   2017-12-06 05:22:38
modified:   2017-12-06 16:52:38+05:30
author:   shabda
tags:   django
---

Django 2.0 was [released
recently](https://docs.djangoproject.com/en/2.0/releases/) and among the
most exciting things for me is support for Window expressions, which
allows adding an OVER clause to querysets. We will use Window
expressions to analyze the commits data to the Django repo.

So what is an over clause?
--------------------------

An over clause is of this format

``` {.sourceCode .sql}
SELECT depname, empno, salary,
  avg(salary)
    OVER (PARTITION BY depname)
FROM empsalary;
```

Compare this to a similar GROUP BY statement

``` {.sourceCode .sql}
SELECT depname, avg(salary)
FROM empsalary
GROUP BY depname;
```

The difference is a GROUP BY has as many rows as grouping elements, here
number of depname. An over clause adds the the aggregated result to each
row of the select.

[Postgres documentation
says](https://www.postgresql.org/docs/9.1/static/tutorial-window.html),
\"A window function performs a calculation across a set of table rows
that are somehow related to the current row. This is comparable to the
type of calculation that can be done with an aggregate function. But
unlike regular aggregate functions, use of a window function does not
cause rows to become grouped into a single output row --- the rows
retain their separate identities. Behind the scenes, the window function
is able to access more than just the current row of the query result.\"
This is true for all other DB implementation as well.

What are real world uses of over expressions?
---------------------------------------------

We will use the Django ORM with the Window expression to to some
analysis on the most prolific committers to Django. To do this we will
export the commiter names and time of commit to a csv.

``` {.sourceCode .bash}
git log  --no-merges --date=iso --pretty=format:'%h|%an|%aI' > commits.iso.csv
```

This is not ranking of Django developers, just of their number of
commits, which allows us an interestig dataset. I am grateful to
everyone who has contributed to Django - they have made my life
immesureably better.

With some light data wrangling using Pandas, we transform this to a per
author, per year data and import to Postgres. Our table structure looks
like this.

``` {.sourceCode .sql}
experiments=# \d commits_by_year;
   Table "public.commits_by_year"
    Column     |  Type   | Modifiers
---------------+---------+-----------
 id            | bigint  |
 author        | text    |
 commit_year   | integer |
 commits_count | integer |
```

We define a model to interact with this table.

``` {.sourceCode .python3}
from django.db import models


class Committer(models.Model):
    author = models.CharField(max_length=100)
    commit_year = models.PositiveIntegerField()
    commits_count = models.PositiveIntegerField()

    class Meta:
        db_table = 'commits_by_year'
```

Lets quickly test if our data is imported. [You can get a csv from
here](https://github.com/shabda/experiments/blob/master/data/commits_by_year.csv),
and import to Postgres to follow along.

``` {.sourceCode .python3}
In [2]: Committer.objects.all().count()
Out[2]: 2318
```

Let us setup our environment and get the imports we need.

``` {.sourceCode .python3}
## Some ORM imports which we are going to need

from django.db.models import Avg, F, Window
from django.db.models.functions import  Rank, DenseRank, CumeDist
from django_commits.models import Committer

# We will use pandas to display the queryset in tanular format
import pandas
pandas.options.display.max_rows=20

# An utility function to display querysets
def as_table(values_queryset):
    return pandas.DataFrame(list(values_queryset))
```

Lets quickly look at the data we have.

``` {.sourceCode .python3}
as_table(Committer.objects.all().values(
  "author", "commit_year", "commits_count"
))
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>author</th>
      <th>commit_year</th>
      <th>commits_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tim Graham</td>
      <td>2017</td>
      <td>373</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sergey Fedoseev</td>
      <td>2017</td>
      <td>158</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mariusz Felisiak</td>
      <td>2017</td>
      <td>113</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Claude Paroz</td>
      <td>2017</td>
      <td>102</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Mads Jensen</td>
      <td>2017</td>
      <td>55</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Simon Charette</td>
      <td>2017</td>
      <td>40</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Jon Dufresne</td>
      <td>2017</td>
      <td>33</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Anton Samarchyan</td>
      <td>2017</td>
      <td>27</td>
    </tr>
    <tr>
      <th>8</th>
      <td>François Freitag</td>
      <td>2017</td>
      <td>17</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Srinivas Reddy Thatiparthy</td>
      <td>2017</td>
      <td>14</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2308</th>
      <td>Malcolm Tredinnick</td>
      <td>2006</td>
      <td>175</td>
    </tr>
    <tr>
      <th>2309</th>
      <td>Georg Bauer</td>
      <td>2006</td>
      <td>90</td>
    </tr>
    <tr>
      <th>2310</th>
      <td>Russell Keith-Magee</td>
      <td>2006</td>
      <td>86</td>
    </tr>
    <tr>
      <th>2311</th>
      <td>Jacob Kaplan-Moss</td>
      <td>2006</td>
      <td>83</td>
    </tr>
    <tr>
      <th>2312</th>
      <td>Luke Plant</td>
      <td>2006</td>
      <td>20</td>
    </tr>
    <tr>
      <th>2313</th>
      <td>Wilson Miner</td>
      <td>2006</td>
      <td>12</td>
    </tr>
    <tr>
      <th>2314</th>
      <td>Adrian Holovaty</td>
      <td>2005</td>
      <td>1015</td>
    </tr>
    <tr>
      <th>2315</th>
      <td>Jacob Kaplan-Moss</td>
      <td>2005</td>
      <td>130</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>Georg Bauer</td>
      <td>2005</td>
      <td>112</td>
    </tr>
    <tr>
      <th>2317</th>
      <td>Wilson Miner</td>
      <td>2005</td>
      <td>20</td>
    </tr>
  </tbody>
</table>
<p>2318 rows × 3 columns</p>
</div>
We will now use the Window expression to get the contributors ranked by
number of commits, within each year. We will go over the code in detail,
but lets look at the queryset and results.

``` {.sourceCode .python3}
# Find out who have been the most prolific contributors
# in the years 2010-2017

dense_rank_by_year = Window(
    expression=DenseRank(),
    partition_by=F("commit_year"),
    order_by=F("commits_count").desc()
)

commiters_with_rank = Committer.objects.filter(
        commit_year__gte=2010, commits_count__gte=10
    ).annotate(
        the_rank=dense_rank_by_year
    ).order_by(
        "-commit_year", "the_rank"
    ).values(
        "author", "commit_year", "commits_count", "the_rank"
    )
as_table(commiters_with_rank)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>author</th>
      <th>commit_year</th>
      <th>commits_count</th>
      <th>the_rank</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tim Graham</td>
      <td>2017</td>
      <td>373</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sergey Fedoseev</td>
      <td>2017</td>
      <td>158</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mariusz Felisiak</td>
      <td>2017</td>
      <td>113</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Claude Paroz</td>
      <td>2017</td>
      <td>102</td>
      <td>4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Mads Jensen</td>
      <td>2017</td>
      <td>55</td>
      <td>5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Simon Charette</td>
      <td>2017</td>
      <td>40</td>
      <td>6</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Jon Dufresne</td>
      <td>2017</td>
      <td>33</td>
      <td>7</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Anton Samarchyan</td>
      <td>2017</td>
      <td>27</td>
      <td>8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>François Freitag</td>
      <td>2017</td>
      <td>17</td>
      <td>9</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Srinivas Reddy Thatiparthy</td>
      <td>2017</td>
      <td>14</td>
      <td>10</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>171</th>
      <td>Joseph Kocherhans</td>
      <td>2010</td>
      <td>53</td>
      <td>11</td>
    </tr>
    <tr>
      <th>172</th>
      <td>Ramiro Morales</td>
      <td>2010</td>
      <td>53</td>
      <td>11</td>
    </tr>
    <tr>
      <th>173</th>
      <td>Jacob Kaplan-Moss</td>
      <td>2010</td>
      <td>42</td>
      <td>12</td>
    </tr>
    <tr>
      <th>174</th>
      <td>Chris Beaven</td>
      <td>2010</td>
      <td>29</td>
      <td>13</td>
    </tr>
    <tr>
      <th>175</th>
      <td>Malcolm Tredinnick</td>
      <td>2010</td>
      <td>26</td>
      <td>14</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Honza Král</td>
      <td>2010</td>
      <td>20</td>
      <td>15</td>
    </tr>
    <tr>
      <th>177</th>
      <td>Carl Meyer</td>
      <td>2010</td>
      <td>17</td>
      <td>16</td>
    </tr>
    <tr>
      <th>178</th>
      <td>Ian Kelly</td>
      <td>2010</td>
      <td>17</td>
      <td>16</td>
    </tr>
    <tr>
      <th>179</th>
      <td>Simon Meers</td>
      <td>2010</td>
      <td>11</td>
      <td>17</td>
    </tr>
    <tr>
      <th>180</th>
      <td>Gary Wilson Jr</td>
      <td>2010</td>
      <td>10</td>
      <td>18</td>
    </tr>
  </tbody>
</table>
<p>181 rows × 4 columns</p>
</div>
Lets look a the the ORM code in more detail here.

``` {.sourceCode .python3}
# We are creating the Window function part of our SQL query here
dense_rank_by_year = Window(
    # We want to get the Rank with no gaps
    expression=DenseRank(),
    # We want to partition the queryset on commit_year
    # Each distinct commit_year is a different partition
    partition_by=F("commit_year"),
    # This decides the ordering within each partition
    order_by=F("commits_count").desc()
)


commiters_with_rank = Committer.objects.filter(
        commit_year__gte=2010, commits_count__gte=10
    # Standard filter oprtation, limit rows to 2010-2017
    ).annotate(
    # For each commiter, we are annotating its rank
        the_rank=dense_rank_by_year
    ).order_by(
        "-commit_year", "the_rank"
    ).values(
        "author", "commit_year", "commits_count", "the_rank"
    )
as_table(commiters_with_rank)
```

Now lets try getting the average commits per commiter for each year
along with the other data.

``` {.sourceCode .python3}
avg_commits_per_year = Window(
    # We want the average of commits per committer, with each partition
    expression=Avg("commits_count"),
    # Each individual year is a partition.
    partition_by=F("commit_year")
)

commiters_with_yearly_average = Committer.objects.filter().annotate(
      avg_commit_per_year=avg_commits_per_year
    ).values(
        "author", "commit_year", "commits_count", "avg_commit_per_year"
    )
# We could have done further operation with avg_commit_per_year
# Eg: F(commits_count) - F(avg_commit_per_year),
# would tell us committers who commit more than average
as_table(commiters_with_yearly_average)
```

This gives us

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>author</th>
      <th>avg_commit_per_year</th>
      <th>commit_year</th>
      <th>commits_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Wilson Miner</td>
      <td>319.250000</td>
      <td>2005</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adrian Holovaty</td>
      <td>319.250000</td>
      <td>2005</td>
      <td>1015</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Jacob Kaplan-Moss</td>
      <td>319.250000</td>
      <td>2005</td>
      <td>130</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Georg Bauer</td>
      <td>319.250000</td>
      <td>2005</td>
      <td>112</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Russell Keith-Magee</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>86</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Jacob Kaplan-Moss</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>83</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Luke Plant</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>20</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Wilson Miner</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>12</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Adrian Holovaty</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>854</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Malcolm Tredinnick</td>
      <td>188.571429</td>
      <td>2006</td>
      <td>175</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2308</th>
      <td>Adam Johnson</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>13</td>
    </tr>
    <tr>
      <th>2309</th>
      <td>Tom</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>13</td>
    </tr>
    <tr>
      <th>2310</th>
      <td>Srinivas Reddy Thatiparthy</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>14</td>
    </tr>
    <tr>
      <th>2311</th>
      <td>François Freitag</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>17</td>
    </tr>
    <tr>
      <th>2312</th>
      <td>Anton Samarchyan</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>27</td>
    </tr>
    <tr>
      <th>2313</th>
      <td>Jon Dufresne</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>33</td>
    </tr>
    <tr>
      <th>2314</th>
      <td>Simon Charette</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>40</td>
    </tr>
    <tr>
      <th>2315</th>
      <td>Mads Jensen</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>55</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>Claude Paroz</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>102</td>
    </tr>
    <tr>
      <th>2317</th>
      <td>Mariusz Felisiak</td>
      <td>4.916084</td>
      <td>2017</td>
      <td>113</td>
    </tr>
  </tbody>
</table>
<p>2318 rows × 4 columns</p>
</div>
You could try other Window functions such as CumeDist, Rank or Ntile.

``` {.sourceCode .python3}
from django.db.models.functions import CumeDist
cumedist_by_year = Window(
    expression=CumeDist(),
    partition_by=F("commit_year"),
    order_by=F("commits_count").desc()
)

commiters_with_rank = Committer.objects.filter(
        commit_year__gte=2010, commits_count__gte=10
    ).annotate(
        cumedist_by_year=cumedist_by_year
    ).order_by(
        "-commit_year", "the_rank"
    ).values(
        "author", "commit_year", "commits_count", "cumedist_by_year"
    )
as_table(commiters_with_rank)
```

Until now, we have partitioned on commit\_year. We can partition on
other fields too. We will partition on author to find out how their
contributions have changed over the years using the Lag window
expression.

``` {.sourceCode .python3}
from django.db.models.functions import Lag
from django.db.models import Value
commits_in_previous_year = Window(
    expression=Lag("commits_count", default=Value(0)),
    partition_by=F("author"),
    order_by=F("commit_year").asc(),
)

commiters_with_pervious_year_commit = Committer.objects.filter(
        commit_year__gte=2010, commits_count__gte=10
    ).annotate(
        commits_in_previous_year=commits_in_previous_year
    ).order_by(
        "author", "-commit_year"
    ).values(
        "author", "commit_year", "commits_count", "commits_in_previous_year"
    )
as_table(commiters_with_pervious_year_commit)
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>author</th>
      <th>commit_year</th>
      <th>commits_count</th>
      <th>commits_in_previous_year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adam Chainz</td>
      <td>2016</td>
      <td>42</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adam Chainz</td>
      <td>2015</td>
      <td>12</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Adam Johnson</td>
      <td>2017</td>
      <td>13</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Adrian Holovaty</td>
      <td>2012</td>
      <td>40</td>
      <td>98</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Adrian Holovaty</td>
      <td>2011</td>
      <td>98</td>
      <td>72</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Adrian Holovaty</td>
      <td>2010</td>
      <td>72</td>
      <td>0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Akshesh</td>
      <td>2016</td>
      <td>31</td>
      <td>0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Alasdair Nicol</td>
      <td>2016</td>
      <td>13</td>
      <td>19</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Alasdair Nicol</td>
      <td>2015</td>
      <td>19</td>
      <td>17</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Alasdair Nicol</td>
      <td>2013</td>
      <td>17</td>
      <td>0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>171</th>
      <td>Timo Graham</td>
      <td>2012</td>
      <td>13</td>
      <td>70</td>
    </tr>
    <tr>
      <th>172</th>
      <td>Timo Graham</td>
      <td>2011</td>
      <td>70</td>
      <td>60</td>
    </tr>
    <tr>
      <th>173</th>
      <td>Timo Graham</td>
      <td>2010</td>
      <td>60</td>
      <td>0</td>
    </tr>
    <tr>
      <th>174</th>
      <td>Tom</td>
      <td>2017</td>
      <td>13</td>
      <td>0</td>
    </tr>
    <tr>
      <th>175</th>
      <td>Unai Zalakain</td>
      <td>2013</td>
      <td>17</td>
      <td>0</td>
    </tr>
    <tr>
      <th>176</th>
      <td>Vajrasky Kok</td>
      <td>2013</td>
      <td>14</td>
      <td>0</td>
    </tr>
    <tr>
      <th>177</th>
      <td>areski</td>
      <td>2014</td>
      <td>15</td>
      <td>0</td>
    </tr>
    <tr>
      <th>178</th>
      <td>eltronix</td>
      <td>2016</td>
      <td>10</td>
      <td>0</td>
    </tr>
    <tr>
      <th>179</th>
      <td>wrwrwr</td>
      <td>2014</td>
      <td>21</td>
      <td>0</td>
    </tr>
    <tr>
      <th>180</th>
      <td>Łukasz Langa</td>
      <td>2013</td>
      <td>15</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<p>181 rows × 4 columns</p>
</div>
I hope this tutorial has been helpful in understanding the window
expressions. While still not as felxible as SqlAlchemy, Django ORM has
become extremely powerful with recent Django releases. Stay tuned for
more advanced ORM tutorials.
