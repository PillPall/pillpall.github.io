---
layout: post
title:  "Ruby conditions if..else.. and RuboCop guard clause etc. ..."
date:   2018-02-20 08:00:00 +1000
categories: Ruby
---

Examples of how to make if else clauses in Ruby nicer and avoid rubocop errors like

```Favor modifier if usage when having a single-line body. Another good alternative is the usage of control flow &&/||.```

```Use self-assignment shorthand +=.```

```Use a guard clause instead of wrapping the code inside a conditional expression.```

<!--excerpts-->

More information about making Ruby code looks nicer can be found in the RuboCop docu [(Link)](http://www.rubydoc.info/gems/rubocop/RuboCop/Cop/Style)

### ConditionalAssignment
Rubocop Error:

```Style/ConditionalAssignment: Use the return of the conditional for variable assignment and comparison```
{% highlight ruby linenos %}
# save return value to variable
puts 'save return value to variable'
x = 1

# Bad :-(
if x <= 1
  response = true
else
  response = false
end
puts response

# Good :-)
puts false unless x <= 1
# or
puts true if x <= 1
{% endhighlight %}

output:
{% highlight bash linenos %}
$# rspec test.rb
save return value to variable
true
true
{% endhighlight %}

### IfUnlessModifier
Rubocop Error:

```Style/IfUnlessModifier: Favor modifier if usage when having a single-line body. Another good alternative is the usage of control flow &&/||.```

{% highlight ruby linenos %}
# String comparison
puts 'String comparison'
header = 'HEAD'
head = 'HEAD'

# Bad :-(
if header == head
  puts true
end

# Good :-)
puts true if header.eql? head
# or
puts false unless header.eql? head
{% endhighlight %}

output:
{% highlight bash linenos %}
$# rspec test.rb
String comparison
true
true
{% endhighlight %}


### SelfAssignment

Rubocop Error:

```Style/SelfAssignment: Use self-assignment shorthand +=.```

{% highlight ruby linenos %}
puts 'easy calculation'
i = 1

# Bad :-(
i = 1
i = i + 1
puts i

# Good :-)
i = 1
i += 1
puts i
{% endhighlight %}

output:
{% highlight bash linenos %}
$# rspec test.rb
easy calculation
2
2
{% endhighlight %}

### GuardClause

Rubocop Error:

```Style/GuardClause: Use a guard clause instead of wrapping the code inside a conditional expression```

{% highlight ruby linenos %}
# Bad :-(
puts 'def my_method'
def my_method
  i = 5
  if i > 0
    puts 'i greater 0'
  end
end

# Good :-)
def my_method
  i = 5
  puts 'i greater 0' unless i <0
end
{% endhighlight %}

output:
{% highlight bash linenos %}
$# rspec test.rb
def my_method
i greater 0
i greater 0
{% endhighlight %}

### More complex example

Here is a more complex example:

{% highlight ruby linenos %}
# Calculate errors
puts 'calculate errors'
i = 0
exit_code = 0
exist = true
response_code = 2
ii = 5

# Bad :-(
if exist == true
  while i < ii
    if response_code == 2
      exit_code = exit_code + 1
    end
    i = i + 1
  end
end
if exit_code < 0
  puts false
else
  puts true
end

# Good :-)
if exist.eql? true
  while i < ii
    exit_code += 1 unless response_code.eql? 0
    i += 1
  end
end
puts true unless exit_code > 0
# or
puts true if exit_code.eql? 0
# or
puts false unless exit_code.eql? 0
{% endhighlight %}

output:
{% highlight bash linenos %}
$# rspec test.rb
calculate errors
true
false
{% endhighlight %}

As you can see, you can massively reduce your code.
