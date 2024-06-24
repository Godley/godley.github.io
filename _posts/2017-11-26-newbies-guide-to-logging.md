---
layout: post
title: 'A basic guide to (Python) Logging'
slug: basic-guide-to-logging
date: '2017-11-26 16:02:00+00:00'
---
Recently, I had the following revellation
![](images/2017/logging.png)

Educators, be those tutorial writers, former teachers, or current teachers are using print statements everywhere because they don't know that industry doesn't do that. (and why should they?)

Or at least, having 2 former teachers respond in that way leads me to believe this. I thought I'd write a short guide on how and why to use logging instead of print.

## What's the difference?

Here's some code using print:
```
def adder(num1, num2):
    print("numbers: " + str(num1) + ", " + str(num2))
    return num1 + num2

result = adder(1, 2)
print("Result was: %i" % result)
```

Here's the same code using logging best practice:
```
import logging # which is built into python

# get a logger object with a useful name. If the program is ran from this file, __name__ will be "__main__", otherwise will be the name of the current file if imported.
LOGGER = logging.getLogger(__name__) 
LOGGER.setLevel(level=logging.DEBUG)

def adder(num1, num2):
    # formatting string, means "we're expecting 2 integers"
    LOGGER.debug("Numbers: %i, %i", num1, num2)
    return num1 + num2

# enables us to use the main method from other languages, avoids the code being called by mistake when importing the file
if __name__ == '__main__':
    logging.basicConfig(level=logging.DEBUG) # allows us to tune ALL of the loggers that have been used in this code (i.e if we called library code which used loggers, they would also use DEBUG). 
    result = adder(1, 2)
    LOGGER.info("Result was: %i", result)
```

The output of this program looks like this:
```
python3 myprog.py
DEBUG:__main__:Numbers: 1, 2
INFO:__main__:Result was: 3
```

## Why should I debug this way? 
A lot of people are probably going to be thinking this is a lot more code than just calling print. Mostly, I think logging is a long term gain in this regard for these reasons:

1. It allows you to have levels of information from your output, depending what you're doing. There are 5 levels built into python:
    - debug: highest verbosity of output. For use when you're trying to figure out what the hell is wrong with your program.
    - info: information which is intended to let you know whereabouts in your program you've got to.
    - warn: information which could indicate something went wrong in execution, like "tried to send status to x endpoint but endpoint wasn't available" but won't cause any serious problems
    - error: something went wrong. Could be anything from "we hit an exception but we can deal with it" to "your user entered something incorrectly"
    - exception: we hit an exception?


1. You can tune the level without needing to remove or comment out those lines. Generally you can do this at the logger object level, e.g using `LOGGER.setLevel`, or for your whole program, e.g `logging.basicConfig(level=logging.<level_in_caps_here>)`

1. By default your output now gives you some context of whereabouts you are in the program, as shown by the example output I mentioned earlier. Particularly with big programs or programs with a lot of different objects/classes, that makes it easier to instantly pin point where you are in the program.

1. With a few lines of code extra to what I have above, you can create different output formats without needing to remember to do that across all of your calls to print. Common example is adding timestamps, like this:
    ```
    ```
    - For more examples take a look at the [python logging cookbook](https://docs.python.org/3/howto/logging-cookbook.html)

1. You're teaching people what they're likely to actually use in industry.


Please note that while my examples given here are all python, the python logging library originates from log4j (java library) and there are similar abilities built into most other well used languages.