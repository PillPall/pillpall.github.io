---
layout: post
title:  "How to use puppet to check if a file / directory exists"
date:   2019-10-10 12:00:00 +1000
categories: Puppet
---

I searched for a very long time an easy way to use exists conditions on files and directories. I recently discovered that you can use the built-in function `find_file` for this.


<!--excerpts-->

The built-in function `find_file` allows you to check if a certain file or a directory on the operating system exists. If it exists puppet will response with a `string` containing the path to the file or directory. If it does not exists puppet will response with an `undef` data type. This allows us to use this in if/else conditions.

## How to use puppet to check if a file exists ?

This is a simple file existence check:

{% highlight puppet linenos %}

$file_path = '/tmp/test_file'

$file_exists = find_file($file_path)

if $file_exists  {
  notify{"File ${file_path} exist":}
} else {
  notify{"File ${file_path} does not exist":}
}
{% endhighlight %}

## How to use puppet to check if a directory exists ?

This is a simple directory existence check:

{% highlight puppet linenos %}

$dir_path = '/tmp/test_path'

$path_exists = find_file($dir_path)

if $path_exists  {
  notify{"Path ${dir_path} exist":}
} else {
  notify{"Path ${dir_path} does not exist":}
}
{% endhighlight %}

As you can see the usage for a file and a directory is the same.

You can find more informations about Puppets built-in function and the built-in function `find_file` in the puppet documentation [Link](https://puppet.com/docs/puppet/5.5/function.html).

Cheers
