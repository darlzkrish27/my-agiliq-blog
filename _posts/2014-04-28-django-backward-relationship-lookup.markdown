---
layout: post
title:  "Django backward relationship lookup"
date:   2014-04-28 10:30:01+05:30
tags: django
author: akshar
---
I often limit the lookup to fields of the model and forget about backward relations. 

Consider the following relationship:

    class Group(models.Model):
        name = models.CharField(max_length=100)

    class Student(models.Model):
        name = models.CharField(max_length=100)
        group = models.ForeignKey(Group)

A group can have many students.

We want to get the groups based on certain conditions on model `Student`. An example is getting the groups which contain a student named 'stud1'. If you can get it using model `Group` and without using `Student`, you can skip this post.

In such a scenario, we tend to use `Student` model. It's more intuitive because the Foreign Key relationship exists from Student to Group.

Let's load the following data to test our queries:

    group1 = Group.objects.create(name='Group 1')
    group1 = Group.objects.create(name='Group 2')
    Student.objects.create(name='stud1', group=group1)
    Student.objects.create(name='stud2', group=group1)
    Student.objects.create(name='stud3', group=group1)
    Student.objects.create(name='stud1', group=group2)

### Get the groups which contain student named `stud1`.

#### Intuitive way:

We first try to get all the students which satisfy the criteria. Query for that would be:

    Student.objects.filter(name='stud1')

And then we try to get the groups of those students. If we use the `Student` model, we can't get a queryset of `Group`. So our approach would be to first get the ids of desired groups and then get a queryset of Group using those ids.

    group_ids = Student.objects.filter(name='stud1').values_list('group', flat=True)
    groups = Group.objects.filter(id__in=group_ids)

#### Less obvious way:
What if we could use the `Group` model directly?

    Group.objects.filter(student__name='stud1')

This gives the exact same result as given by the *Intuitive way*.

So, even though there is no field called `student` on `Group` and we didn't specify any relationship to Student from Group, Django is smart enough to figure out the relationship for us.

### Get the groups with name `Group1` which contain students named `stud1`

#### Intuitive way:

    group_ids = Student.objects.filter(name='stud1', group__name='Group1').values_list('group', flat=True)
    groups = Group.objects.filter(id__in=group_ids)

#### Less obvious way:

    groups = Group.objects.filter(name='Group1', student__name='stud1')

Gives the exact same result as given by preceding two queries.

I always missed the backward relationship while following Django tutorial or whenever I read the docs. I tried finding it while writing this post to see if the docs missed mentioning it. I was wrong yet again, there is a section which mentions it.

    https://docs.djangoproject.com/en/dev/topics/db/queries/#lookups-that-span-relationships

