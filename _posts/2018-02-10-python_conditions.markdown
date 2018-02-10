---
layout: post
title:  "Python statements/conditions"
date:   2018-02-10 01:00:00 +1000
categories: python
---
Following operators can be used in a loop or to check conditions in a if...else... statement:

{% highlight python linenos %}
Check Boolean
not False

Check Int/String
< 	less than
<= 	less than or equal to
> 	greater than
>= 	greater than or equal to
== 	equal
!= 	not equal

{% endhighlight %}
<!--excerpts-->
### if...elif...else... statement
{% highlight python linenos %}
#!/bin/python
# check if var is set/true
var = True

if not var:
  print "I'm false"
elif var:
  print "I'm true"

# Check if integer smaller than x
x = 1

if x <= 1:
  print "smaller or equal 1"
elif x <= 10:
  print "smaller or equal 10"
else:
  print "bigger than 10"

# Check if variable test is set

#test = "Hello World"

try: test
except NameError: test = None

if test is None:
  print "Variable test is not set"
else:
  print test  
{% endhighlight %}

### While loop
{% highlight python linenos %}
#!/bin/python
# easy loop do while var is False
var = False
while (var == False):
    print "var is false"
    var = True

# More complex loop print message "var is False" 10 times before break out of while"
var = False
i = 0
while (var == False):
    while ( i <= 10 ):
        i = i + 1
        print "var is False"
    #var = True  
    break

{% endhighlight %}
