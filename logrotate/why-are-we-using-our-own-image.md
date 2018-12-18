# Why are we using our own image

There is no official logrotate image present, and we have a very generic image which accepts complete logrotate config file in which we can mention as many directories as we want to rotate. We can also pass cron schedule as an environment variable.