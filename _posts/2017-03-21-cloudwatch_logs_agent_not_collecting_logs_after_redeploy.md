---
layout: default
title:  "CloudWatch Logs agent not collecting logs after redeploy"
date:   2017-03-21 01:37:55 -0700
categories: writings
---

### Problem:
I ran into an issue where I would redeploy an Elastic Beanstalk app and CloudWatch
would stop collecting logs for one of my four environments even though the logs
are still being sent to stdout.


### Understanding the problem:
When the CloudWatch Logs Agent starts, it compares the first line of the log file
to the previous first line to determine if it is a different log file.
Because the first few lines in the log file will always be the same startup of Puma
web server message:

{% highlight ruby %}
=> Booting Puma
=> Rails 4.2.3 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
. . .
{% endhighlight %}

CloudWatch assumes the log files are the same and will not collect newer logs.

### Solution:
In configuring my log groups in the `.ebextensions` folder of my app, the
CloudWatch Logs Agent [has a setting](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html)
 for `file_fingerprint_lines` that will use
the line (lines) number(s) to generate the fingerprint.
In `application.rb` I output a timestamp in the `config.after_initialize`
callback so that the fingerprint will differ on each deploy.
{% highlight ruby %}
config.after_initialize do
  Rails.logger.info 'Rails initialized'
end
{% endhighlight %}


And now the first few lines of my log file will always be

{% highlight ruby %}
=> Booting Puma
=> Rails 4.2.3 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
{"timestamp":"2017-03-21T01:28:54-0700","severity":"INFO","source":"application","message":"Rails initialized"}
. . .
{% endhighlight %}

The timestamp is always on line 5, and therefore I have `file_fingerprint_lines` set to `5`

{% include analytics.html %}
