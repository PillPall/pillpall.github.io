---
layout: post
title:  "Using variables to Call Puppet resources"
date:   2019-05-13 12:00:00 +1000
categories: Puppet
---

Sometimes you may wish to use variables for calling a specific Puppet resources instead of using if-else conditions. Luckily Puppet allows us to use variables in Puppet resource names like classes, modules etc. ...

<!--excerpts-->

In my case I had several Puppet classes like `aem_curator::install_aem62`, `aem_curator::install_aem63` and `aem_curator::install_aem64`. To call the right Puppet class for the installation I used the following If-else conditions:


{% highlight puppet linenos %}
if $aem_profile == 'aem62' {

    aem_curator::install_aem62 { "${aem_id}: Install AEM profile ${aem_profile}":
      aem_artifacts_base      => $aem_artifacts_base,
      aem_base                => $aem_base,
    }

  } elsif $aem_profile == 'aem63' {

    aem_curator::install_aem63 { "${aem_id}: Install AEM profile ${aem_profile}":
      aem_artifacts_base      => $aem_artifacts_base,
      aem_base                => $aem_base,
    }

  } elsif $aem_profile == 'aem_64' {

    aem_curator::install_aem64 { "${aem_id}: Install AEM profile ${aem_profile}":
      aem_artifacts_base      => $aem_artifacts_base,
      aem_base                => $aem_base,
    }
{% endhighlight %}

As you can see this file can be getting to big to keep track of it very easily.

So instead of maintaining such a complex Puppet manifest I want to reduce it to one Puppet resource call by calling a class using the variable `aem_profile` in the class name. To do so we can call a Puppet resource within `Resource[]`.

{% highlight puppet linenos %}
Resource["aem_curator::install_${aem_profile}"]  { "${aem_id}: Install AEM profile ${aem_profile}":
  aem_artifacts_base      => $aem_artifacts_base,
  aem_base                => $aem_base,
}
{% endhighlight %}

As you can see, the class call now depends on the variable `${aem_profile}`. This variable defines which Puppet class we want to call.

In my case I reduced a manifest of 280 lines down to 40 lines [Example](https://github.com/shinesolutions/puppet-aem-curator/commit/87b303f39759b49a308f4e60c9d700cbfbaa8554#diff-fcd29db3229524a51a7f2645715cdaec).

Cheers
