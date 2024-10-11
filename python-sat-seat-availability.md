Title: Using Python to Find an Available Seat for the SAT
Slug: python-sat-seat-availability
Date: 2024-10-10
Category: articles
Tags: Linux, Python
Summary: Trying to register for the SAT but there&rsquo;s no availability? Here&rsquo;s how to find a seat using Python.l
<!-- Status: draft -->

My son recently took the SAT at our local high school. In the months leading up to the test, reserving a seat for him turned out to be quite the challenge. Several high schools in nearby cities had to cancel their SAT testing, which led to a surge in demand for seats at our local testing sites. Reservations at the [SAT Test Center web site](https://satsuite.collegeboard.org/sat/test-center-search) filled up only minutes after being released. 

If we wanted to reserve a seat, we would need to know immediately when they released more, or, if someone cancelled. This seemed like an easy problem to solve using Python. 

Since Python is installed by default with most Linux distributions, it&rsquo;s the perfect candidate for automating simple tasks like this. After spinning up a [Linode](https://linode.com) Nanode, I created a script that would check seat availability and send an email notification if one was available, then scheduled it as a cron. 

##How it Works

Here&rsquo;s the script:
```
#/usr/bin/python3

import requests
import smtplib

url = 'https://aru-test-center-search.collegeboard.org/prod/test-centers?date=2024-08-24&zip=xxxxx&country=US'
alert = ['jane@example.com','john@example.com']
mailfrom = "me@example.com"
sitecode = "08239"

#query a list of sites and convert the response to json
sites = requests.get(url).json()

#loop through results
for site in sites:
    #if site code matches my test location
    if site['code'] == sitecode:
        #if a seat is available
        if site['seatAvailability'] is True:
            server = smtplib.SMTP('localhost')
            #send alerts
            for recipient in alert:
                msg = "From: " + mailfrom + "\r\n"
                msg = msg + "To: " + recipient + "\r\n"
                msg = msg + "Subject: SAT seat available!\r\n"
                msg = msg + "\r\n"
                msg = msg + "https://satsuite.collegeboard.org/sat/test-center-search\n"
                server.sendmail(mailfrom, recipient, msg)
            server.quit()
        #we are only interested in our local site, so we break out of the loop
        break
```

####Step 1: Add your test date and zip code to the URL

When you visit the web site to reserve a seat, it makes an asynchronous call to an API, which returns a JSON string. Using devtools, I was able to extract the URL. Make sure your URL includes the date you want to test and your zip code. For example:
```
url = 'https://aru-test-center-search.collegeboard.org/prod/test-centers?date=2024-08-24&zip=90210&country=US'
```

###Step 2: Find your site code

To find your site code, you will need to dig through the JSON response from the above URL. Search for the name of the testing location, then find the code. The key for this value is "code" and the value will be a five digit number. Once you have that code, plug it in to the script:
```
sitecode = "08239"
```

###Step 3: Enter the email address(es) where you want to be notified

I configured the script to notify both my wife and I, to increase our chances of quickly responding to a seat being made available. Enter one or more email addresses into this array:
```
alert = ['jane@example.com','john@example.com']
```

###Step 4: Schedule the cron

Don&rsquo;t forget this last step! The [Linux crontab](https://www.geeksforgeeks.org/crontab-in-linux-with-examples/) is a simple but powerful utility for scheduling jobs. Each user on a Linux system can create and maintain their own crontab file using the `crontab -e` command. I created the following job to run every minute:
```
* * * * * /home/jjriv/sat-seat-finder.py
```

###That&rsquo;s it!

Your work is done. Sit back and relax. But don&rsquo;t forget to check your inbox. 
