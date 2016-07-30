# Python Logging
### A meditation on **silent** failures

^ Quick intro: Jess Unrein. Platform Engineer with Sprout. Dev Bootcamp graduate and jazz pianist.

---

## Who is this talk for?

^ This is a novice level talk for people who have never encountered logging or know that it's important but they're not sure where to start.

---

## Why logging?

^ TDD is a hot button issue that many people talk passionately about. The first Python codebase I worked on had excellent test coverage. After I'd been working there for about six months our DevOps guy started getting really irritated that we thought that was sufficient, and asked me to get started on logging.

^ When I asked what needed to be logged he just told me, "You know, the important stuff." It can be difficult to figure out exactly what that means when you're just starting out.

^ 	While testing is incredibly important for development, logging gets short shrift here. Logging is what make sure things in production work the way they should after you've released your application into the wild.

^ The Zen of Python teaches us that errors should never pass silently. 

^ Does not make a distinction between errors and failures, which seems pretty important to me.


---

# [fit] Errors vs. **Failures**

^ An error is something that causes your code to barf. Unless you've explicitly silenced something, it should let you know in a loud way.

^ However, not every failure is an error. Just because your code didn't scream at you doesn't mean everything functioned as intended.

^ Good logging and monitoring can help you detect **failures** that won't show up in a stack trace

---

## Shouldn't I just test my code?

^ Sure. In a perfect world where we all write perfect code. But you can't always anticipate the things that will cause failures in your application that are not the result of the code you wrote.

^ Tests are better at detecting the presence of the wrong outcome than the absence of the right outcome.

---

## Dependency changes:
### a horror story

^ A big thing that can cause problems in your application is an unanticipated or undocumented change when you update your dependencies.

^ That's a bit dramatic, but a while ago I was working on a codebase where we expected to be able to send out this welcome email, but only when they were marked as "active" by an admin user. "Pending" users should not get the email on account creation

^ SQLAlchemy changed the way their `History` object presented information and suddenly our email stopped going out

^ We didn't even notice for an embarrasingly long time. Didn't anticipate this dependency change,  so we did not create unit tests

^ Better logging would have helped us diagnose the problem sooner

---

# [fit] What is **logging**?

^ Logging is record keeping for your app

^ It gives critical information about errors you might see, behaviors you don't see but should, and gives you clues to detect problems that you might not otherwise run across.

^ **Printing** to stdout is not good logging practice!

^ Printing should be for outputting information to a user in a CLI. It can be useful for on the fly output checking during active development, but it is not a suitable application or library monitoring tool.

^ Logging should be methodical and collected in a dedicated file that does not conflict with whatever stdout might have. Print also doesn't provide key functionality that the Python logging module provides

---

# **Anatomy** of a python logger

^ What exactly can a logger do for you that printing can't?

---

# Three key logger functions

- Level
- Handler
- Formatter

---

# Logging levels

^ Not every logging statement contains the same level of information

^ Transactional events should be stored and treated differently than error or critical level events. Grouping logical units of logging together can help you keep track of what's going on and aid you in diagnosing problems quickly.

---

- `NOTSET`
- `DEBUG`
- `INFO`
- `WARNING`
- `ERROR`
- `CRITICAL`

^ Debug level logs might record how many database queries were executed in a batch so that if you start to see slow performance you have an idea of where to look. Debug level logging should genereally only be turned on in development environments

^ Info level logs might be something that record events like user account registration or sending out push notifications. These are for transactions that you want to keep track of, but are expected

^ Warning level logs are things that don't break your application, but you want to keep tabs on. This might be something like unexpected user behaviors, or edge cases that you want to make sure don't get out of control.

^ Error level logs and above should be where you find stack traces to point you to explicit, noisy failures in your application. For example, faliure to connect to your database would be an error or critical level logging message.

---

# Where should my logs go?

^ logfiles vs stdout and stderr

^ stderr like stdout can get very noisy, and you don't want to pipe things into stderr that aren't explicit errors in your program. It can make it much harder to diagnose the root cause of errors in your application.

^ Logs should go into their own separate log files. You can set up where the logs go using FileHandlers.


---

### Handler

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

handler = logging.FileHandler("info.log")
handler.setLevel(logging.INFO)

logger.addHandler(handler)

logger.info("Helpful logging message, id: {}".format(1))
logger.error("{} error encountered on line {}".format(KeyError, 10))
```

Output:

```python
Helpful logging message, id: 1
<type 'exceptions.KeyError'> error encountered on line 10
```

^ Handlers are how we separate different levels of logging information into the appropriate files.

^ You can set a level to a handler and it will record log messages of that level **or above** to that file

---

# What should I log?

^ Key transactions

^ Errors

^ Unusual behaviors / anticipated edge cases

^ The log messages we got before were fine, but they were little better than print statements. Without formatting we don't even necessarily know what level of logs we're getting at any given time.

^ There might be general information we want for every logging statement that we don't want to write out every time we craft a logging call. This is where formatters come in.

---

## Basic Formating

```python
import logging

...

formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

logger.setHandler(handler)

logger.info("Helpful log message, id: {}".format(1))
logger.error("{} error encountered on line {}".format(KeyError, 10))
```

Output:

```python
2016-07-29 16:39:59,313 - __main__ - INFO - Helpful logging message, id: 1'
2016-07-29 17:05:45,290 - __main__ - ERROR - <type 'exceptions.KeyError'> error encountered on line 10
```
^ Gives us a bit more information and enough information to have a better idea of what's going on.

---

### Advanced formatting

#### - External libraries
#### - Subclassing **`logging.Formatter`**

^ The logging messages we saw above are fine for glancing through, but when you get to a larger production environment you'll probably want to move to machine readable logs.

^ Json logs are nice because other services can consume them and help you with monitoring.

^ The downside to json logs is that they can be more difficult from a human-readability standpoint when you're searching through the logs on your own.

^ There are several external libraries available that subclass `logging.Formatter` for you to return json logs. There are a few that are specifically designed to be compatible with logstash

^ Things you might want: what server did the log statement originate from? Was there a logged in user when the event happened? Maybe you want to be able to add custom parameters for log statements, like a dictionary of parameters passed to a function. Something like this might help you keep track of sending too many notifications to a particular subset of users.

^ Just ideas, but there's a wealth of information you might be leaving on the table that could be critical to effectively debugging a problem.

---

Creating your own formatter:

```python
import logging
import json

class MyFormatter(logging.Formatter):
	def format(self, record):
		my_message = dict(
			name = record.name,
			timestamp = str(datetime.datetime.now()),
			level = record.levelname,
			message = record.msg,
		)
		return json.dumps(my_message)

{"timestamp": "2016-07-28 14:01:24.420356", "message": "This is a helpful message", "name": "__main__", "level": "INFO"}
```

---

## How much is **too** much?

^ Logging is great, but overly noisy logs can encourage deafness.

^ Proper file handling and level setting can help mitigate this problem, but there can be too much of a good thing.

^ Your logging is excessive if it's significantly slowing down your processes or you can't find the information you need to in a reasonable amount of time. I know that's vague but logging really boils down to what's helpful for you.

^ Re: noisy logs. Sometimes it's useless until it suddnely isn't. Leveling is important. When in doubt, put in the debug logs.

^ Log files can get big quick. They collect a lot of information, and especially with high volume apps this can get to be a problem. Compressing and archiving outdated log files will be necessary at some point, so make sure you start thinking about how to handle that situation early on.

---

## `RotatingFileHandler`

```python
import logging

handler = logging.RotatingFileHandler(path, maxBytes=20,
                                  backupCount=5)
```

#### Limits the maximum **size** of a logfile
#### Optional backups appended with `*1.log`, `*2.log` etc
#### This file rotates as it approaches 20 bytes and keeps 5 backups

---

## `TimedRotatingFileHandler`

```python
handler = logging.TimedRotatingFileHandler(path,
                                       when="d",
                                       interval=2,
                                       backupCount=5)
```
#### Limits the amount of **time** a log will stick around
#### Available intervals -> secs, mins, hours, days, or weekdays (0-6)
#### This file rotates every 2 days with 5 backup log files

---

## I have logs!


### Now what?

^ What should I do with my logs?

^ Start watching them! Not only can it keep you up to date on your application's key transactions, but using warn statements for unexpected but non-breaking behavior can let you know if your users are interacting with your application in ways that you didn't expect

^ logging levels come back into play here. If you use a monitoring system, you don't want it to notify you for routine level events. However, monitoring systems allow you to do cool things like notify you if you experience an abnormally low volume or absence of an event you expect to see a lot of, like user logins.

^ Log monitoring systems that can help put your logs to good use

^ newrelic insights
^ logstash
^ splunk
^ papertrail

---

# Thank You
<br><br>
Slides and presenter notes available [on github](https://github.com/thejessleigh/talks/tree/master/python_talks/intro_logging)
<br><br><br><br>
[@JLUnrein](https://twitter.com/JLUnrein) | [jessunrein.com](http://jessunrein.com)
