---
layout: post
title: 'Knitting a temperature blanket'
slug: temp-blanket
date: '2025-01-22'
---

Recently I decided I was going to learn to knit. In particular, I wanted to make a temperature blanket.

A temperature blanket is a project wherein you knit (or crochet, or crosstitch, or some other craft) a row or line every day which represents something of that day's temperature.
You could...knit a row each for the high and low that day.
You could...knit a row for the average.
You could...knit the high during the day and the high during the night, splitting your stitches for the row by the number of hours of daylight.
There are many different options, none of which I'd actually thought about the repercussions before I just...started to learn how to knit by making the blanket. I really like the idea of visualising our weather with something physical, and being "forced" to keep doing a bit on it every day. It's very easy for me to get sucked into big long term goals, and get impatient when they take a while to work out, so the intention of knitting this is to be able to focus on just...the days going by.

Typically you localise the data to wherever you are, and then divide up the likely range of temperatures for your area by however many colours you feel like using. There's a super useful website called [temperature-blanket.com](https://temperature-blanket.com) which helps you figure out what you want to do, visualise what it's going to look like, and automatically calculate the optimal gauge for your area. By optimal its chosen mechanism is %, so it determines how to split up your gauge evenly across the colours so that you get a good mix of colours rather than it being all one colour. 

I initially opted for the average temperature for the day, but having seen on the tool that that creates probably the least interesting looking blanket, I switched to the high temperature. If I had my time again I would probably knit one for high, one for low, but there's always next year!

Back to the gauge, temperature-blanket.com determined for my area, using 11 colours, it should be split unevenly, so the first colour is -6-8 degrees, 8-10, 10-12, 12-14, 14-16, 16-17, 17-19, 19-21, 21-23, 23+. This all works out to give you roughly 8-10% in each colour. I determined after a few days of knitting that I was bored of the first colour and that a 12 degree range didn't accurately represent how cold some days in January have been, so I added an 11th colour and split the first gauge from -6 up to 3 degrees, then 3-8 etc. This does mean a few of my first ~10 or so rows are wrong, but I've decided to try to ignore that.

I also decided to do 2 rows per day because 1 didn't feel like it was growing fast enough, and the initial blanket pattern I was using used chunky yarn which wasn't common enough for me to find a lot of colours. I think I will regret this decision in 6 months time.

Then it snowed, so I decided to add in white for any days which it snowed. This again means I have missed two rows of snow because I bought the white a few days after the first batch.

As you can see, whenever I change my mind, I am also trying to exercise "imperfectionism". If I were sewing, or if I knew where the undo button was in knitting, I might have started again.

I also determined once we got through to summer that >23 degrees and >28 feel very different, so I've pre-emptively added an extra colour on that end too.
All of this seems like a pretty damning representation of how "average" and cold our weather is, to be honest.

## Data sources
Next we come to actual data collection. Temperature-blanket defaults to showing you last year's data, because if it showed and tried to calculate on this year's data, there just wouldn't be much there. So as I knitted through each line I found myself refreshing the web page again and again, anxiously waiting for [open-meteo]() to get new data for the next day.

Coming back to my [previous blog post]() which of course I never finished the project, I realised I already had a very good board from [pimoroni]() which had a temperature sensor on it, so all I needed to do was turn it on, hook up a backend and away we go. I also realised open-meteo probably takes a rough guess on my house based on whatever sensors it has in the area (I haven't extensively looked into it beyond working out that it's quite good and easy to use), so if I had my own set up, that would be much more accurate (and therefore satisfying).

## The stack
So now we have:
- a frontend container written in typescript, react with MUI for components.
- backend written in fastAPI, SQLModel and connecting to a SQLite database, because if I had to spend several hours doing migrations, setting up a container etc I would very quickly lose all energy for the project.
- my enviro grow outside sending data every 24 hours to my backend

The only problem here is that my IP address changes fairly regularly, so every time I went to check on the sensor, it would be in an error state connecting.
Luckily, we have a NAS, so after a few hours twiddling I got the containers running successfully on it, and a few days later we now have a hostname which is managed by the router so that should it's IP change (which happens less regularly), the sensor will still be able to send data without reconfiguration.

It took me a few days of modifications to the API before I determined I needed to volume mount the database, so that if I changed my mind, added an API etc then I didn't lose several hours of data. (it is all backed up on the sensor, but that requires bringing it indoors, plugging it in etc so ideally want to avoid doing that)

## Gauge management
Up next I needed a way of storing, somewhere, what my gauge would look like, i.e mapping the temperature ranges to the appropriate colours. I initially hard coded this into the frontend, but I decided I didn't want a massive switch statement and that as I had already changed my mind 2-3 times, I might want to do that again later.

So I added another database model for a gauge, and started using postman as my means of configuring it, because building a frontend for this when I was still in the initial "just get the damn thing working" phase felt like too much. In order to avoid back and forth over exact hex codes I opted to use [html colors]() which appear to have a lot more range than I first thought and fairly accurately represent the colours of yarn I'm using.

## Backfilling
Since all this configuration took a little bit of time, I had missed several days worth of data. I decided to write a dynamic `new_rows` endpoint for the frontend:
- first collecting the data for the year to date from the database
- for any gaps at the front (I've assumed, perhaps wrongly, once the sensor is set up, that I won't need this after a while), gather the days data from open-meteo. As mentioned this is an incredibly easy to use API - you can select on their home page which meteorological data you want, the fidelity (i.e days, hours) of the data and enter your own coordinates, and it will spit out all the python code you need to generate a pandas dataframe from the information.
- pull the appropriate gauge, calculate based on the gauge what the high, low, average colour will be for that day.
- return just what the frontend needs - the temperatures, date, and colours.

Later I might take the daily data, and if there isn't 48 readings (one every 30 minutes), verify the data against open meteo. 

## Project management
After creating this endpoint, I then realised my major problem and part of the reason I'd wanted to move off temperature-blanket as my source of info was that I was losing track of my rows. On top of that, entering or hard coding the sensor ID and gauge ID into the frontend just seemed a bit silly, especially as the sensor ID comes from the hardware UID which is a long sequence of numbers and letters.

So I created a new model called Project, which has a sensor, a gauge, a year, and a list of completed rows.
This way as the days went on and I knitted more, the backfill algorithm which is calling the database and open meteo every time wouldn't keep adding another day of data as I could adjust the algorithm based on the last day I had completed.
Since I have particular parameters by which I'm choosing to knit, I also added in a row type, which is either high, low, average or snow. To avoid conversion back and forth I set the enum values to be those which open meteo uses, which I thought was really clever.

## Daily calculations and erroneous data
Despite computer science basically being applied mathematics, much of what you end up doing in a web stack is just REST.
A chunk of the API I found engaging was how best to calculate and store what I actually needed to know, rather than having the dynamic `new_rows` endpoint getting allllll the day's data and manipulating it to get the high, low and average. High and low would actually be fairly easy - for each day, sort by temperature in each direction, get the first result. Simples.

Average on the other hand, despite not being something I was actually going to knit with, would need all the rows for the day. So I decided that on each new sensor reading being submitted, I would have another table for DailyReading, which would initialise to all the variables being whatever the temperature was if there wasn't an existing DailyReading that day, then as more data comes in, store the total number of readings, recalculate the average, and change the high and low depending whether the reading triggered those actions.

After adding a few test readings I also decided to do the inverse of this, and add a delete endpoint in case I had received any readings when I had brought the sensor indoors, for example.

An alternative here would be to add a computed table, but since SQLite is just a file, I would probably need to move to postgres in order to add this.
I could also add a background task which triggers in the backend at say, 00:01, rather than doing it on each addition. At the moment, from looking at the logs on the sensor, doing this calculation on each write isn't too expensive, but I will likely shift to a backend trigger when I get bored/want to learn a bit more fastAPI.

# Grand plans
I basically now have a working, useful project management system for my temperature blanket, which I stupidly refresh every time new data comes in (I now have it set to every 12 hours to make it a bit more interesting). What I'd like to do from here is:
- add frontends for sensor readings, maybe with some cool visualisations. Whilst this is mainly a mechanism for making my blanket more accurate, long term it would be cool to be able to watch our weather change.
- add authentication, then get it on a subdomain. My husband has also been working on his own projects on the NAS, and is now doing a bunch of sys admin stuff. I plan to piggyback off this and maybe launch weather.roseanddeer.com which will have some stuff public, but keep the project management under an account.
- Collect more data. I built the sensor collector endpoint around what pimoroni had already configured as the JSON with their super cool built in software, but I'm throwing away the humidity, pressure and moisture readings. 
- Perhaps forward it into homebridge or home assistant. I had previously built a homebridge extension for pimoroni using MQTT, so I could probably just either configure my backend to feed off MQTT, send into MQTT, or get the sensor to do both. I think probably send it to MQTT might make the most sense, this way I can also have the server communicate bidirectionally should homebridge want to collect a reading a second time or something.
- Submit the data to open weather sources. I have no idea if open meteo takes additional sensors, but I know there are other platforms for doing so.
- Open source it! I've built this to be as minimal and flexible as possible, so if others would get something useful from this then cool.

What I like about this project is I'm able to build it around my own needs, add frontends as they become useful and once I have a clear vision on what the backend looks like. It's also handy having a portfolio project to be able to use when I need to land new contracts, etc.
