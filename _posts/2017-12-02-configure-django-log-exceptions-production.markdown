---
layout: post
title:  "Configure Django to log exceptions in production"
date:   2017-12-02 08:19:18+05:30
tags: django
author: akshar
---
Django default logging behaviour for unhandled exceptions is:

#### With DEBUG=True (Development)

* Log the exception on console/stream.
* Show the exception on page, i.e in http response.

#### With DEBUG=False (Production)

* Do not log the exception on console/stream.
* Do not show the exception in response.
* Send an email to admin if email settings are configured correctly.

Usually not logging the exception on console isn't a problem since an exception email is sent to you which can help you know the source of exception. But this assumes that email settings are configured correctly else you will not receive the exception email.

You might not have email settings configured correctly and don't want to get into that right away. You might instead want to log the exception on console even with DEBUG=False. This post would help you in such scenario.

#### Default logging configuration

Django's default logging setting is:

	DEFAULT_LOGGING = {
		'version': 1,
		'disable_existing_loggers': False,
		'filters': {
			'require_debug_false': {
				'()': 'django.utils.log.RequireDebugFalse',
			},
			'require_debug_true': {
				'()': 'django.utils.log.RequireDebugTrue',
			},
		},
		'formatters': {
			'django.server': {
				'()': 'django.utils.log.ServerFormatter',
				'format': '[%(server_time)s] %(message)s',
			}
		},
		'handlers': {
			'console': {
				'level': 'INFO',
				'filters': ['require_debug_true'],
				'class': 'logging.StreamHandler',
			},
			'django.server': {
				'level': 'INFO',
				'class': 'logging.StreamHandler',
				'formatter': 'django.server',
			},
			'mail_admins': {
				'level': 'ERROR',
				'filters': ['require_debug_false'],
				'class': 'django.utils.log.AdminEmailHandler'
			}
		},
		'loggers': {
			'django': {
				'handlers': ['console', 'mail_admins'],
				'level': 'INFO',
			},
			'django.server': {
				'handlers': ['django.server'],
				'level': 'INFO',
				'propagate': False,
			},
		}
	}

Without any explicit `settings.LOGGING` configured in settings.py, this is the default logging configuration Django works with. You can ignore `django.server` part.

Any unhandled Django exception is handled in function `handle_uncaught_exception`. The relevant code is on <a href="https://github.com/django/django/blob/1.11.7/django/core/handlers/exception.py#L124" target="_blank">Github</a>

Error is logged using `logger.error` in this function. This logger is an instance of `django.request`. Since logger `django` is a parent of `django.request`, so log records are propogated to logger `django`.

As you can see from `DEFAULT_LOGGING`, logger `django` has two handlers.

* console
* mail_admins

As you can see from `DEFAULT_LOGGING`, handler `console` has a filter called `require_debug_true` because of which this handler doesn't handle log records in production, i.e when DEBUG=False.

#### Logging to console in production

So you can create a new handler which directs `ERROR` log records to Stream when DEBUG=False.

This handler would look like

	'console_debug_false': {
		'level': 'ERROR',
		'filters': ['require_debug_false'],
		'class': 'logging.StreamHandler',
	}

And you can ask logger `django` to use this handler by adding this handler in `loggers['django']['handlers']`.

Your final `settings.LOGGING` would look like following:

	LOGGING = {
		'version': 1,
		'disable_existing_loggers': False,
		'filters': {
			'require_debug_false': {
				'()': 'django.utils.log.RequireDebugFalse',
			},
			'require_debug_true': {
				'()': 'django.utils.log.RequireDebugTrue',
			},
		},
		'formatters': {
			'django.server': {
				'()': 'django.utils.log.ServerFormatter',
				'format': '[%(server_time)s] %(message)s',
			}
		},
		'handlers': {
			'console': {
				'level': 'INFO',
				'filters': ['require_debug_true'],
				'class': 'logging.StreamHandler',
			},
			'console_debug_false': {
				'level': 'ERROR',
				'filters': ['require_debug_false'],
				'class': 'logging.StreamHandler',
			},
			'django.server': {
				'level': 'INFO',
				'class': 'logging.StreamHandler',
				'formatter': 'django.server',
			},
			'mail_admins': {
				'level': 'ERROR',
				'filters': ['require_debug_false'],
				'class': 'django.utils.log.AdminEmailHandler'
			}
		},
		'loggers': {
			'django': {
				'handlers': ['console', 'console_debug_false', 'mail_admins'],
				'level': 'INFO',
			},
			'django.server': {
				'handlers': ['django.server'],
				'level': 'INFO',
				'propagate': False,
			}
		}
	}

If you don't want emails to be sent to admins, in case email settings aren't configured correctly, then you should remove `mail_admins` from `loggers['django']['handlers']`.


