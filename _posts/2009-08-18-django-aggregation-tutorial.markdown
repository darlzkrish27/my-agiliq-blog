---
layout: post
title:  "Django aggregation tutorial"
date:   2009-08-18 18:08:29+05:30
tags: aggreagtion
author: shabda
---
One of the new and most awaited features with Django 1.1 was aggregation. As usual,
Django comes with a very [comprehensive documentation](http://docs.djangoproject.com/en/dev/topics/db/aggregation/) for this. Here, I have tried to
put this in how-to form.

<a href="#howtos">Jump to howtos</a> or <a href="http://github.com/uswaretech/Shiny-New-Django-1.1/tree/master">Get source on Github</a>.

Essentially, aggregations are nothing but a way to perform an operation on group of rows. In databases,
they are represented by operators as `sum`, `avg` etc.

To do these operations Django added two new methods to querysets.

1. `aggregate`
2. `annotate`

When you are have a queryset you can do two operations on it,

1. Operate over the rowset to get a single value from it. (Such as sum of all salaries in the rowset)
2. Operate over the rowset to get a value for *each row in the rowset* via some related table.

The thing to notice is that option 1, will create one row from rowset, while option 2 will
not change the number of rows in the rowset. If you are into analogies, you can think that
option 1 is like a [reduce](http://docs.python.org/library/functions.html#reduce) and option 2 is like a [map](http://docs.python.org/library/functions.html#map).

In sql terms, aggregate is a operation(SUM, AVG, MIN, MAX), without a group by,
while annotate is a operation with a group by on rowset_table.id. (Unless explicitly overriden).

<a name="howtos" ></a>

Ok enough talk, on to some actual work. Here is a fictional models.py representing
a HRMS application. We will use this to see how to use aggreagtion to solve
some common problems.

    from django.db import models
    
    class Department(models.Model):
        dept_name = models.CharField(max_length = 100)
        established_on = models.DateField()
        
        def __unicode__(self):
            return self.dept_name
        
    class Level(models.Model):
        level_name = models.CharField(max_length = 100)
        pay_min = models.PositiveIntegerField()
        pay_max = models.PositiveIntegerField()
        
        def __unicode__(self):
            return self.level_name
        
    class Employee(models.Model):
        emp_name = models.CharField(max_length = 100)
        department = models.ForeignKey(Department)
        level = models.ForeignKey(Level)
        reports_to = models.ForeignKey('self', null=True, blank=True)
        
        pay = models.PositiveIntegerField()
        joined_on = models.DateField()
        
    class Leave(models.Model):
        employee = models.ForeignKey(Employee)
        leave_day = models.DateField()
        
        
    """
    #Populate DB, so we can do some meaningful queries.
    #Create Dept, Levels manually.
    #Get the names file from http://dl.getdropbox.com/u/271935/djaggregations/names.pickle
    #Or the whole sqlite database from http://dl.getdropbox.com/u/271935/djaggregations/bata.db
    import random
    from datetime import timedelta, date
    import pickle
    names = pickle.load(file('/home/shabda/names.pickle'))
    for i in range(1000):
        emp = Employee()
        emp.name = random.choice(names)
        emp.department = random.choice(list(Department.objects.all()))
        emp.level = random.choice(Level.objects.all())
        try: emp.reports_to = random.choice(list(Employee.objects.filter(department=emp.department)))
        except:pass
        emp.pay = random.randint(emp.level.pay_min, emp.level.pay_max)
        emp.joined_on = emp.department.established_on + timedelta(days = random.randint(0, 200))
        emp.save()
    """
    
    """
    employees = list(Employees.objects.all())
    for i in range(100):
        employee = random.choice(employees)
        leave = Leave(employee = employee)
        leave.leave_day = date.today() - timedelta(days = random.randint(0, 365))
        leave.save()
        
    """


#### Find the total number of employees.

In sql you might want to do something like,

`select count(id) from hrms_employee`

Which becomes,

`Employee.objects.all().aggregate(total=Count('id'))`

If fact doing a `connection.queries.pop()` shows the exact query.

`SELECT COUNT("hrms_employee"."id") AS "total" FROM "hrms_employee"`

But wait, we have a builtin method already for that, `Employee.objects.all().count()`, so lets try something else.

#### Find the total pay of employees.

The CEO wants to find out what is the total salary expediture, this also converts
the queryset to a single value, so we want to `.aggregate` here.

`Employee.objects.all().aggregate(total_payment=Sum('pay'))`

Gives you the total amount you are paying to your employees.

#### Find the total number of employees, per department.

Here we want a value per row in queryset, so we need to use aggregate here. Also,
there would be one aggregated value per dpeartment, so we need to annotate Department
queryset.

`Department.objects.all().annotate(Count('employee'))`

If you are only interested in name of department and employee count for it, you can do,
`Department.objects.values('dept_name').annotate(Count('employee'))`

The sql is

    SELECT "hrms_department"."dept_name", COUNT("hrms_employee"."id") AS "employee__count" FROM "hrms_department" LEFT OUTER JOIN "hrms_employee" ON ("hrms_department"."id" = "hrms_employee"."department_id") GROUP BY "hrms_department"."dept_name"
    
#### Find the total number of employees, for a specific department.

Here you could use either of `.annotate` or `.aggregate`,

    Department.objects.filter(dept_name='Sales').values('dept_name').annotate(Count('employee'))
    Department.objects.filter(dept_name='Sales').aggregate(Count('employee'))

If you see the SQLs, you will see that `.annotate` did a `group by`, while the `.aggregate`
did not, but as there was only one row, `group by` had no effect.

#### Find the total number of employees, per department, per level

This time, we can not annotate either Department model, or the Level model, as we
need to `group by` both department and level. So we will annotate on Employee

    Employee.objects.values('department__dept_name', 'level__level_name').annotate(Count('id'))
    
This leads to the sql,

    SELECT "hrms_department"."dept_name", "hrms_level"."level_name", COUNT("hrms_employee"."id") AS "id__count" FROM "hrms_employee" INNER JOIN "hrms_department" ON ("hrms_employee"."department_id" = "hrms_department"."id") INNER JOIN "hrms_level" ON ("hrms_employee"."level_id" = "hrms_level"."id") GROUP BY "hrms_department"."dept_name", "hrms_level"."level_name
    
#### Which combination of Employee and Deparments employes the most people

We can order on the annotated fields, so the last query is modified to,

    Employee.objects.values('department__dept_name', 'level__level_name').annotate(employee_count = Count('id')).order_by('-employee_count')[:1]
    
#### Which employee name is the most common.

We can want to `group by emp_name`, so `emp_name` is added to values. After that we order on the annotated field
and get the first element, to get the most common name.

`Employee.objects.values('emp_name').annotate(name_count=Count('id')).order_by('-name_count')[:1]`
 
 
------------
This was a overview of how django annotations work. These remove a whole class of queries for which
you had to use custom sql queries in the past.
 
-----------
###Resources

1. [Source on Github](http://github.com/uswaretech/Shiny-New-Django-1.1/tree/master)
2. [sqlite file for this model to test](http://dl.getdropbox.com/u/271935/djaggregations/bata.db)
3. [Aggregation on Django docs](http://docs.djangoproject.com/en/dev/topics/db/aggregation/)

--------------------
Want to build a Django app? [Talk to us](http://uswaretech.com/contact/)



