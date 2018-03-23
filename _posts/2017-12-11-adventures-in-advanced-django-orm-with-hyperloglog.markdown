---
layout: post
title:  "Adventures in advanced Django ORM with HyperLogLog"
date:   2017-12-11 22:54:38+05:30
tags: django
author: shabda
---

Counting distinct object is a common operation. Most databases and Django make its quite simple to do so. Given appropriate tables or models you would be doing something like


    select event_type, count(distinct user_id)
    from github_events
    group by 1;


With appropriateley defined Django model you would do

    GithubEvents.objects.values(
        "event_type"
    ).annotate(
        distinct_user_count=Count(
            "user_id", distinct=True
        )
    )

Count distinct is however [quite slow on large datasets](https://en.wikipedia.org/wiki/Count-distinct_problem). There are a few faster methods which give an approximate count in much less time. HyperLogLog is one of the most common methods. We will try implementing that in Django ORM without going to raw SQL and see how far we can get. (Spoiler: We will go far but won't be able to do it.)

We will use a table of github events. Citusdata has some interesting data sets, they have a [Github event data in CSV](https://examples.citusdata.com/events.csv) which we will use today. We will use Postgres.


Lets create the table and load the data.


    CREATE TABLE github_events
        (
            event_id bigint,
            event_type text,
            event_public boolean,
            repo_id bigint,
            payload jsonb,
            repo jsonb,
            user_id bigint,
            org jsonb,
            created_at timestamp
        );

    COPY github_events FROM events.csv CSV

We then do a insepctdb on created table to get our models.

    python manage.py inspectdb github_events

This gives us


    class GithubEvents(models.Model):
        event_id = models.BigIntegerField(blank=True, null=True)
        event_type = models.TextField(blank=True, null=True)
        event_public = models.NullBooleanField()
        repo_id = models.BigIntegerField(blank=True, null=True)
        payload = models.TextField(blank=True, null=True)  # This field type is a guess.
        repo = models.TextField(blank=True, null=True)  # This field type is a guess.
        user_id = models.BigIntegerField(blank=True, null=True)
        org = models.TextField(blank=True, null=True)  # This field type is a guess.
        created_at = models.DateTimeField(blank=True, null=True)

        class Meta:
            managed = False
            db_table = 'github_events'


Change the `event_id` to be the primary_key. Other fields stay as is

    event_id = models.BigIntegerField(primary_key = True)

To get the HyperLogLog based distinct counts, grouped by event_type we need to the following SQL.


    SELECT
      counted_data.event_type,
      CASE WHEN num_uniques < 2.5 * 512
      AND num_zero_buckets > 0 THEN (
        (
          0.7213 / (1 + 1.079 / 512)
        ) * (
          512 * log(
            2,
            (512 :: numeric) / num_zero_buckets
          )
        )
      ):: int ELSE num_uniques END AS approx_distinct_count
    FROM
      (
        SELECT
          event_type,
          (
            (
              pow(512, 2) * (
                0.7213 / (1 + 1.079 / 512)
              )
            ) / (
              (
                512 - count(1)
              ) + sum(
                pow(2, -1 * bucket_hash)
              )
            )
          ):: int AS num_uniques,
          512 - count(1) AS num_zero_buckets
        FROM
          (
            SELECT
              event_type,
              hashtext(user_id :: varchar) & (512 - 1) AS bucket_num,
              31 - floor(
                log(
                  2,
                  min(
                    hashtext(user_id :: varchar) & ~(1 << 31)
                  )
                )
              ) AS bucket_hash
            FROM
              github_events
            GROUP BY
              1,
              2
          ) AS bucketed_data
        GROUP BY
          1
        ORDER BY
          1
      ) AS counted_data
    ORDER BY
      1

We will not go into the details of the SQL. This SQL is adapted from Periscopedata, and [they have a very nice tutorial here](https://www.periscopedata.com/blog/hyperloglog-in-pure-sql).

We will not be discussing the constants used or the algorithm. You should read the Periscopedata article or [read the original paper](algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf).

We will try to convert this SQL to a Django ORM queryset and see how far we can go. This is a pretty advanced SQL with

1. Nested Sub-queries
2. `hashtext` which is an undocumented Postgres function
3. Bitwise operations
4. Multiple grouping and mathematical functions

Trying to convert this to ORM will allow us to use some adavanced Django ORM features.


Lets start with the innermost subquery

    SELECT
      event_type,
      hashtext(user_id :: varchar) & (512 - 1) AS bucket_num,
      31 - floor(
        log(
          2,
          min(
            hashtext(user_id :: varchar) & ~(1 << 31)
          )
        )
      ) AS bucket_hash
    FROM
      github_events
    GROUP BY
      1,
      2


The `hashtext(user_id :: varchar)` is used twice, so let us start by implementing that. `hashtext` is not part of `django.contrib.postgres`, but we can get it by subclassing `Func`

    # Some imports we are going to need for the rest of the tutorial.
    from django.db.models import (DateTimeField, ExpressionWrapper, F,
         IntegerField, Value, Min, Count, TextField)
    from django.db.models.functions import Cast

    class HashText(Func):
        function = 'HASHTEXT'


With this we can do

    GithubEvents.objects.annotate(
        hashed_user_id=HashText(
            Cast("user_id", TextField())
        )
    )

With this we can do:


    GithubEvents.objects.annotate(
        hashed_user_id=HashText(
            Cast("user_id", TextField())
        )
    ).annotate(
        bucket_number=ExpressionWrapper(
            F("hashed_user_id").bitand(511), output_field=IntegerField()
        )
    )

We are taking the annotated `hashed_user_id` and applying a bitwise AND. We
need to wrap this in a `ExpressionWrapper` as Django doesn't have sufficient data to calculate the output field.

At this point we have the two fields we need to group on, so we can do

    GithubEvents.objects.annotate(
        hashed_user_id=HashText(
            Cast("user_id", TextField())
        )
    ).annotate(
        bucket_number=ExpressionWrapper(
            F("hashed_user_id").bitand(511), output_field=IntegerField()
        )
    ).values("event_type", "bucket_number", hashed_user_id=hashed_user_id)

Now we need our aggregated bucket hash corresponding to this expression:

    31 - floor(
        log(
          2,
          min(
            hashtext(user_id :: varchar) & ~(1 << 31)
          )
        )
      )

So we define a new function class, and use it to annotate further

    class FloorLog2(Func):
        template = 'FLOOR(LOG(2, (%(expressions)s)))'

    grouped_gs = GithubEvents.objects.annotate(
        hashed_user_id=HashText(
            Cast("user_id", TextField())
        )
    ).annotate(
        bucket_number=ExpressionWrapper(
            F("hashed_user_id").bitand(511), output_field=IntegerField()
        )
    ).values(
        "event_type", "bucket_number",
        hashed_user_id=F("hashed_user_id")
    ).annotate(
        bucket_hash=31-FloorLog2(
            Min("hashed_user_id").bitand(2147483647)
        )
    )

Lets look at the generated query

    str(grouped_gs.query)

It gives us

    'SELECT "github_events"."event_type", HASHTEXT("github_events"."user_id"::text) AS "hashed_user_id", (HASHTEXT("github_events"."user_id"::text) & 511) AS "bucket_number", (31 - FLOOR(LOG(2, ((MIN(HASHTEXT("github_events"."user_id"::text)) & 2147483647))))) AS "bucket_hash" FROM "github_events" GROUP BY "github_events"."event_type", HASHTEXT("github_events"."user_id"::text), (HASHTEXT("github_events"."user_id"::text) & 511)'


Exactly what we want in our innermost subquery.


Now we want to get to the middle subquery in our SQL, which is

    SELECT
          event_type,
          (
            (
              pow(512, 2) * (
                0.7213 / (1 + 1.079 / 512)
              )
            ) / (
              (
                512 - count(1)
              ) + sum(
                pow(2, -1 * bucket_hash)
              )
            )
          ):: int AS num_uniques,
          512 - count(1) AS num_zero_buckets
        FROM
          (
            ... [First Subquery]
          ) AS bucketed_data
        GROUP BY
          1
        ORDER BY
          1

Our first attempt to get this, might be to add another level of `values` and anotate. Doing this we get.

    class SumPow2(Func):
        template = "sum(pow(2, -1 * %(expressions)s ))"

    grouped_qs_2 = grouped_gs.values(
        "event_type", bucket_hash=F("bucket_hash")
    ).annotate(
        num_zero_buckets=512-Count("event_type"),
        num_uniques=ExpressionWrapper(
            188686.824458612/(512-Count("event_type") + SumPow2('bucket_hash')), output_field=IntegerField()
        )
    )
    str(grouped_qs_2.query)

Looking at the generated query we see


    'SELECT "github_events"."event_type", (31 - FLOOR(LOG(2, (MIN((HASHTEXT("github_events"."user_id"::text) & 2147483647)))))) AS "bucket_hash", (512 - COUNT("github_events"."event_type")) AS "num_zero_buckets", (188686.824458612 / ((512 - COUNT("github_events"."event_type")) + sum(pow(2, -1 * (31 - FLOOR(LOG(2, (MIN((HASHTEXT("github_events"."user_id"::text) & 2147483647)))))) )))) AS "num_uniques" FROM "github_events" GROUP BY "github_events"."event_type"'


This is not what we want. Rather than the nested group by we see only the outher most group by, corresponding to the last `values` call we added.

My next approach was to try wrapping this in a subquery expression - [added in Django 1.11](https://docs.djangoproject.com/en/1.11/releases/1.11/), but
there is no way to select with a group by from a subquery expression.

At this point I gave up as there seemed to be no way to nest group by expressions. (Though I would ove to be wrong on this). I was able to go much further than I initially expected.


### References


* [HyperLogLog: the analysis of a near-optimal
cardinality estimation algorithm](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)
* [HyperLogLog in Pure SQL](https://www.periscopedata.com/blog/hyperloglog-in-pure-sql)
* [Distributed count(distinct) with HyperLogLog on Postgres](https://www.citusdata.com/blog/2017/04/04/distributed_count_distinct_with_postgresql/)


