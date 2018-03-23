---
layout: post
title:  "Django timezones"
date:   2014-08-20 10:30:01+05:30
tags: django
author: akshar
---
This post tries to clear various concepts related to Django timezones. Settings relevant to timezone are TIME_ZONE and USE_TZ. We will see various scenarios with different values for these settings and will figure out how TIME_ZONE and USE_TZ behave.

Once through this post, you will stop playing guessing games with Django timezones.

I will use sqlite throughout this post.

I assume that you understand the general concept of timezone. If not then see the <a href="#concept_timezone">section at bottom</a> before you read further.

<h3 id="installing_pytz">Installing pytz</h3>

Working with timezones is much easier if you have pytz installed. Django recommends using it too.

	pip install pytz

### Naive and aware datetimes

Before you can understand Django timezones, it's important to understand naive and aware datetimes.

#### Naive datetime
Suppose you schedule a Google hangout for 28 July, 6:30 PM with someone. You do not give any timezone information. That person has no way to know if you mean 6:30 PM in India or 6:30 PM in USA. This 6:30 PM is open to interpretation. It is a naive datetime.

	>>> from datetime import datetime
	>>> dt = datetime.now()
	>>> dt
	datetime.datetime(2014, 7, 29, 12, 55, 20, 672052)
	>>> print dt.tzinfo
	None                  #This prints None.

You can check **tzinfo** attribute on a datetime instance to know if the datetime instance in naive or aware. If `tzinfo` in None, then it is a naive datetime.

Also **dt** would be same as your computer's time. Check it using **date** command on a new terminal. Though system's datetime has a timezone info associated with current time, Python's datetime.now() doesn't give the timezone information by default.

	~ $ date
	Tue Jul 29 12:55:22 IST 2014

#### Aware datetime
If you say 3:30 AM New York time, it is an aware datetime. This 3:30 AM is not open to interpretation.

	>>> import pytz
	>>> tz = pytz.timezone('America/New_York')
	>>> new_york_dt = datetime.now(tz) # We pass timezone instance to now()
	>>> new_york_dt
	datetime.datetime(2014, 7, 29, 3, 43, 58, 507746, tzinfo=<DstTzInfo 'America/New_York' EDT-1 day, 20:00:00 DST>)
	>>> print new_york_dt.tzinfo
	America/New_York

As you can see `tzinfo` attribute is set on the datetime instance if we pass a pytz.timezone instance to datetime.now(). Here **new_york_dt** is an aware datetime instance.

You can read <a href="https://docs.python.org/2/library/datetime.html" target="_blank">Python docs on datetime</a> to know more about naive and aware datetimes.

### Setting up the project

I will try everything inside a virtual environment. My virtualenv is named django_timezone and path is /tmp.

	(django_timezone)/tmp $ django-admin.py startproject tz

I am using Django 1.6.5. Let's will install pytz too.

	(django_timezone)/tmp/tz $ pip install pytz

Django timezone behaviour depends on settings.USE_TZ and settings.TIME_ZONE. Their default values are:

	TIME_ZONE = 'UTC'
	USE_TZ = True

We need a DateTime field to go through various scenarios. Create an app which will contain a model with a DateTimeField

	(django_timezone)/tmp/tz $ python manage.py startapp article

In article/models.py

	class Article(models.Model):
		description = models.TextField()
		publish_date = models.DateTimeField()

Run syncdb, make sure to add 'article' to INSTALLED_APPS. Make sure to register Article in admin.

### Using datetime.now() from shell

At this point, we have the following settings

	TIME_ZONE = 'UTC'
	USE_TZ = True

Start a shell.

	(django_timezone)/private/tmp/tz $ python manage.py shell

	>>> from datetime import datetime
	>>> dt = datetime.now()
	>>> print dt
	2014-07-29 09:52:33.417034

I am in India at GMT+05:30 and right now it is 3:22 PM in India. But now() tells me it's 09:52 AM which is 5 hours 30 minutes behind India's time. now() is telling the UTC time. And TIME_ZONE is set as 'UTC'. So, now() uses TIME_ZONE to tell time. But this datetime `dt` is naive, as you can verify by printing dt.tzinfo.

#### Make USE_TZ = False

Make sure to set settings.USE_TZ=False

	>>> from datetime import datetime
	>>> dt = datetime.now()
	>>> print dt
	2014-07-29 09:55:49.481732

now() still gives the UTC time. And `dt` is naive as can be verified by printing `dt.tzinfo`.

This verifies datetime.now() is independent of settings.USE_TZ.

It seems it depends on TIME_ZONE. Let's change settings.TIME_ZONE to 'America/New_York' and see if output of now() differs. New York is at UTC-04:00. So now() should start saying 05:55 AM.

	settings.TIME_ZONE = 'America/New_York'

Restart the shell and invoke now()

	>>> from datetime import datetime
	>>> dt = datetime.now()
	>>> print dt
	2014-07-29 05:58:27.056390

now() tells the current time of New York. This verifies that datetime.now() is dependent on TIME_ZONE. Though this datetime is naive too; verify using dt.tzinfo.

Reset TIME_ZONE to 'UTC' and USE_TZ to True.

### Using django.utils.timezone.now()

**Edit**: The striked out lines is what I wrote earlier, but the good folks on reddit corrected me.

<p><del>It behaves similar to datetime.now() with one difference. Where datetime.now() always gives naive datetime object, this gives a naive or aware datetime object depending on value of USE_TZ.</del></p>

It always returns UTC time. It is not dependent on settings.TIME_ZONE. It gives a native or aware datetime object depending on values of settings.USE_TZ

#### When USE_TZ is True

	>>> from django.utils.timezone import now
	>>> dt = now()
	>>> dt
	datetime.datetime(2014, 7, 29, 10, 22, 24, 163837, tzinfo=<UTC>)
	>>> print dt.tzinfo
	UTC

<p><del>This gives the UTC time because we have TIME_ZONE set to UTC. This behaviour is similar to datetime.now(). But in addition, the datetime object also has tzinfo attribute set on it and so it is an aware datetime object.</del></p>

We got an aware datetime object and time is represented in UTC.

##### Verifying that timezone.now() doesn't depend on TIME_ZONE

Change settings.TIME_ZONE

	TIME_ZONE = 'Asia/Kolkata'

Restart the shell and use timezone.now()

	>>> from django.utils.timezone import now
	>>> dt = now()
	>>> dt
	datetime.datetime(2014, 7, 29, 10, 23, 24, 163837, tzinfo=<UTC>)
	>>> print dt.tzinfo
	UTC

So even though we set timezone as 'Asia/Kolkata', now() returns UTC time.

#### When USE_TZ is False

Make sure to change settings.USE_TZ=False

	>>> from django.utils.timezone import now
	>>> dt = now()
	>>> dt
	datetime.datetime(2014, 7, 29, 10, 28, 10, 353942)
	>>> print dt.tzinfo
	None

This behaviour is exactly similar to datetime.now(). You get the time in same timezone as settings.TIME_ZONE, and it is a naive datetime.

Reset USE_TZ to True.

### Before we proceed

When we say timezone support is enabled, it means USE_TZ = True. When we say timezone support is disabled, it means USE_TZ = False.

### How Article.publish_date behaves in form

Let's first consider the scenario where timezone is enabled.

#### Timezone is enabled, i.e USE_TZ=True

Go to article creation page <a href="http://localhost:8000/admin/article/article/add/" target="_blank">http://localhost:8000/admin/article/article/add/</a>

We are concerned with publish_date. If you click on **Today**, it will enter the date taken from your system time. When you click on **Now**, it will enter the time taken from your system time. `Today` and `Now` work using Javascript, and Javascript doesn't have any idea of Django TIME_ZONE and USE_TZ. Javascript only has access to your system time. That's why `Today` and `Now` will take the value from your system's time irrespective of Django TIME_ZONE and USE_TZ values.

I am in **India which is at UTC+05:30**, and it is **4:13 PM** for me. Clicking on `Now` puts 4:13 PM in my text field, even though I have TIME_ZONE='UTC'. In **'UTC', it is 10:43 AM (4:13 PM - 5 hours 30 minutes) right now**. Even though TIME_ZONE is set to 'UTC', clicking on `Now` uses my system time, because Javascript doesn't have a knowledge of settings.TIME_ZONE

Save the article. Check the value of publish_date saved in db.

	>>> article = Article.objects.get()
	>>> article.publish_date
	datetime.datetime(2014, 7, 29, 16, 13, 28, tzinfo=<UTC>)
	
So Django tells that publish_date is a timezone aware object with publish_date being shown as 4:13 PM, UTC. This is a bug, we will see an implication of this bug soon. When I submitted the form, I submitted with the assumption that I am submitting time in India timezone. I expect publish_date to say 4:13 PM, India time(hereafter referred as IST). Or it should say 10:43 AM, UTC. We will see how such scenario should be handled and fix this bug later on in this post.

##### Check how is it saved in db.

	python manage.py dbshell

	sqlite> select * from article_article;
	1|First Article|2014-07-29 16:13:28

It only tells the time and doesn't tell anything about timezone. So, db doesn't save timezone information. Django puts a timezone info on datetime instance based on TIME_ZONE. That's why we saw article.publish_date in 'UTC' timezone.

**Note:** Postgres can save timezone information too. But we will not worry about it in this post.

##### What happened when I submitted publish_date from browser

* Browser submitted 4:16 PM to server.
* Browser did not submit any timezone info.
* Django saves 4:16 PM in db without any timezone info
* When getting publish date of Article, Django gets 4:16 PM from db, without any timezone info.
* In addition, Django checks value of TIME_ZONE and so adds tzinfo attribute on publish_date and makes it timezone aware.

#### Timezone is disabled, i.e USE_TZ=False

Make sure to modify settings.USE_TZ=False

Go to article creation page <a href="http://localhost:8000/admin/article/article/add/" target="_blank">http://localhost:8000/admin/article/article/add/</a>

You will still see `Today` and `Now` buttons picking up the system time. They are in no way dependent on TIME_ZONE and USE_TZ.

Create an article from the admin. And then get it and print its publish_date.

	>>> article = Article.objects.get(id=2)
	>>> article.publish_date
	datetime.datetime(2014, 7, 30, 15, 37, 18)

We get a naive datetime object in this scenario. Django didn't add any tzinfo attribute on datetime instance after fetching it from db.

Reset USE_TZ to True.

### Implication of the bug

Suppose your project is used by publishers in many countries. Project's TIME_ZONE is set to 'UTC'. I am a publisher from India. When I use admin form, I submit date and time in IST. Consider the last article I created. I created the article with publish_date as 3:37 PM, 30th July. My intention was 3:37 PM IST. I expect that article will be shown on the site after 3:37 PM IST.

But Django when getting this article from db considers the publish_date as 3:37 PM UTC, because settings.TIME_ZONE is UTC.

Placeholder code for view which shows articles.

	from django.utils.timezone import now
	current_time = now()
	Article.objects.filter(publish_date__lte=current_time)

When it is 3:38 PM IST, I expect my article to appear on site. But it won't because when it is 3:38 PM IST, django.utils.timezone.now() will say 10:08 AM UTC. And publish_date is 3:37 PM UTC as per Django, so the filter() will not get my article.

### Fix for this

Django has a concept of <a href="https://docs.djangoproject.com/en/1.6/topics/i18n/timezones/#default-time-zone-and-current-time-zone">default and current timezone.</a>

#### Default timezone and current timezone

Default timezone is the timezone defined by settings.TIME_ZONE.

Django documentation says, current timezone is the timezone used for rendering. It has an additional purpose and it can fix our buggy scenario. 

##### Additional purpose

We can say to Django that current timezone is IST. So, when we submit a form, date and time we pass will be considered in IST and then will be converted to default timezone(UTC) and will then be saved in the database.

Supposing current timezone is set to IST.

* I post 3:37 PM.
* Current timezone is set to IST, we will see how to set current timezone soon.
* Django will know I mean 3:37 PM, IST.
* Django will convert 3:37 PM, IST to UTC(because default timezone for project is UTC)
* And will save converted UTC time in db. Converted UTC time would be 10:07 AM.
* And our filter condition will work properly.

#### Setting current timezone

This can be done using django.utils.timezone.activate

Django <a href="https://docs.djangoproject.com/en/1.6/topics/i18n/timezones/#selecting-the-current-time-zone" target="_blank">example</a> uses session and middleware to set timezone. We will use middleware too but instead of bothering with sessions to keep track of user's timezone, I will hardcode the current timezone.

Let's add a file article/middleware.py with following content:

	import pytz

	from django.utils.timezone import activate

	class TimezoneMiddleware(object):
		def process_request(self, request):
			activate(pytz.timezone('Asia/Kolkata'))

Using `activate()` sets the current timezone. With our middleware, **current timezone is set to IST**. Since we are using a middleware, current timezone will be set in every request.

Add this middleware to settings.MIDDLEWARE_CLASSES:

	MIDDLEWARE_CLASSES = (
		'article.middleware.TimezoneMiddleware',
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
	)

Visit Article add page in admin.

Enter 'Date' for Publish date as '2014-07-30' and 'Time' as '15:37:08'. Click 'Save'.

#### Verify how is it saved in db:

	sqlite> select * from article_article where id=3;
	3|Third article|2014-07-30 10:07:18

So, time we submitted was converted to UTC from IST and then saved to db.

##### Why?
* We submitted 15:37:08 as time.
* Middleware ran and set the current timezone as IST.
* Django assumed that the time we submitted was in IST because current timezone was set to IST.
* Django sees values of TIME_ZONE which is UTC. So it converted the time to UTC and then saved in db.

#### Check how is it shown in admin.

	See <a href="http://localhost:8000/admin/article/article/3/" target="_blank">article 3 details</a>

Publish Time gets shown as '15:37:18'. This is because middleware runs on every request. And our middleware sets the current timezone to IST. So, whatever our site gets from db, is converted to IST and then shown to us, which is perfect.

#### Check how is it shown on shell.

	(django_timezone)/private/tmp/tz $ python manage.py shell
	>>> article = Article.objects.get(id=3)
	>>> article.publish_date
	datetime.datetime(2014, 7, 30, 10, 7, 18, tzinfo=<UTC>)

Django gets the time from db. In db, time is saved as 10:07:18. Django reads the value of TIME_ZONE and sets the TIME_ZONE as tzinfo attribute on datetime object.

### Can current timezone work if timezone is not activated

Set USE_TZ to False.

Create a new Article from admin.

Set Publish date as 2014-07-30, and publish time as 15:37:18. Save it.

#### Check how is it saved in db.

	sqlite> select * from article_article where id=4;
	4|Fourth article|2014-07-30 15:37:18

So, even though our code tries to set current timezone through middleware, current timezone wasn't set because we have timezone support disabled.

Django didn't convert our IST time to UTC before saving to db.

#### How is it shown on shell

	>>> article = Article.objects.get(id=4)
	>>> article.publish_date
	datetime.datetime(2014, 7, 30, 15, 37, 18, tzinfo=<UTC>)

#### View the Article 4 details in admin.

	http://localhost:8000/admin/article/article/4/

Admin shows the publish time same as what was saved in db. No conversion between timezones was done.

Reset USE_TZ to True.

### Notes

* USE_TZ=True means timezone support is enabled, USE_TZ=False means timezone is not enabled.
* TIME_ZONE determines in what timezone datetime.now and django.utils.timezone.now return you the time.
* datetime.now always gives naive datetime objects.
* django.utils.timezone.now gives naive datetime objects if timezone support is not enabled, and gives aware objects if timezone support is enabled.
* Current timezone is set using `django.utils.timezone.activate`.
* Current timezone activation works only if timezone support is enabled, i.e USE_TZ should be True.
* When current timezone is enabled, Django converts the datetime you POST into settings.TIME_ZONE and then saves this converted datetime in database.
* With current timezone enabled, Django converts the datetime it fetches from db into current timezone. Datetime instances with current timezone is displayed in the user facing forms.

<h3 id="concept_timezone">Concept of timezone</h3>
Every country is at a different position on Earth and all countries do not face the Sun at the same instant. India and United States are almost on opposite sides of each other on Earth. When India faces the Sun, USA doesn't face it. At that instant, it is light(day) in India and dark(night) in USA. Earth rotates on its axis. So after few hours, USA would face the Sun and India wouldn't face it.

People want the clock to show between 6 AM to 6 PM during the day and between 6 PM to 6 AM during night. People in India have their clocks set such that it says between 6 AM to 6 PM when India faces the Sun. At the same instant, USA doesn't face Sun and it is dark there; so people in US have their clocks set such that it shows a time between 6 PM to 6 AM. USA and Indian clocks show different time at the same instant.

But India and USA aren't mathematical terms. There should be a way to know what time is it exactly in USA when it is 6:30 PM, 28 July in India. So we need to know how many hours we need to add or subtract to this time to get the time in USA. This is where timezones come into picture.

There is a reference place on Earth relative to which time on other places is calculated. That place is Greenwich. And its timezone is conventionally called GMT + 0:00. GMT and UTC can be used interchangeably. So Greenwich's timezone is GMT+00:00 or UTC+00:00. India's timezone is GMT+05:30 or UTC+05:30. When it is 8:00 AM in Greenwich, it is 1:30 PM in India. New York's timezone is GMT-04:00 during summer. So when it is 8:00 AM in Greenwich, it is 4:00 AM in New York.

Whole explanation of timezone is outside the scope of this blog. Read <a href="http://en.wikipedia.org/wiki/Time_zone" target="_blank">about in on Wikipedia</a> to know more.

<a href="#installing_pytz">Resume at top</a>

