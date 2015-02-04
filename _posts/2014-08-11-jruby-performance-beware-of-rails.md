---
layout: post
title: 'JRuby Performance: Beware of Rails deprecated warnings'
date: 2014-08-11 02:52:18.000000000 -04:00
categories:
- Ruby
tags: []
status: publish
type: post
published: true
comments: true
meta:
  _edit_last: '1'
  _wp_old_slug: hello-world
  _s2mail: 'yes'
author:
  login: amunda
  email: basit.hanif@gmail.com
  display_name: Abdul Munda
  first_name: ''
  last_name: ''
---
 
We recently upgraded our JRuby version from 1.3.x to 1.7.12 and suddenly noticed a huge regression in performance. Our requests were starting to slow down by 500 ms and that was huge for our customers.

We investigate what could be the potential issue and we used some low level tools like jstack to really nail down where threads were spending most of the time. After investigating, we saw that in our call trace lot of Thread.getStackTrace() calls were made by the app. This is an expensive call in Java.

We looked closely of where the calls were coming from and found out that deprecated warnings in Rails methods were causing these calls. Our first thought was that we will just disable the deprecated warnings in Rails by using this flag:

{% highlight ruby %}
ActiveSupport::Deprecation.silenced =true
{% endhighlight %}

But we still continue to see getStackTrace calls in our call traces. We further dig down in Rails code and found out that it was the Kernel 'caller' method which was the culprit.

{% highlight ruby %}
ActiveSupport::Deprecation.warn "Calling `assign_shortcuts` is deprecated and has no effect anymore.", caller
{% endhighlight %}

Caller method in JRuby will call the getStackTrace method and this was used by 'ActiveSupport::Deprecation.warn' method to report the exact line of the offending method.

Since it was passed in as an argument to deprecated warnings method, logic for switching on/off deprecated warnings has no effect. So the 'silenced' flag didn't help at all.

So our solution was to fix all the deprecated warnings and wherever we couldn't fix the warnings, we had to monkey patch Rails and completely remove the deprecated method call from the method.

So please beware of Rails deprecated warnings in JRuby!
