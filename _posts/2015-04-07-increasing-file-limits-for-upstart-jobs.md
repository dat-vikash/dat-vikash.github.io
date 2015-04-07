---
layout: post
title: Increasing File descriptor limits for linux upstart jobs
date: 2015-04-07 8:58:00
disqus: y
___

 I was recently working on a JVM based project using the Play! Framework. This project required a single but beefy on premise server deploy and needed to handle multiple concurrent websocket connections as well as regular HTTP GETs, PUTs, POSTs, and DELETEs. When we hit production, we quickly ran into the following error:
{% highlight java %}
java.io.IOException: Too many open files
{% endhighlight %}

For some background information, java webservers (Tomcat, Jboss, Jetty, Netty) utilize file descriptors for loading complied files, opening both long and short term sockets, and of course serving any file based content. By default our VM OS, Ubuntu, has both a soft and hard limit of 1024 open files allowed. We tried setting the global limits using ulimit

{% highlight bash %}
ulimit -n 500000
{% endhighlight %} 

and also via the <i>/etc/security/limits.conf</i>

{% highlight bash %}  
  root           soft    nofile          500000
  root           hard    nofile          500000
{% endhighlight %}

but unfortunately we found that these limits were not being applied to our upstart job. After some serious googling, we found that Ubuntu [upstart](http://upstart.ubuntu.com/wiki/Stanzas#limit) provides a limit stanza. Using this we were edited our upstart configuration in <i>/etc/init/<job>.conf</i> 

{% highlight bash %}
  limit nofile 500000 500000
{% endhighlight %}

and finally the limit update took hold! To verify you can cat the limits for a PID:

{% highlight bash %}
  cat /proc/<PID>/limits
{% endhighlight %}
 
Hopefully this information can save others some time.

