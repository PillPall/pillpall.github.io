---
layout: post
title:  "Using variables to Call Puppet resources"
date:   2019-05-13 12:00:00 +1000
categories: Puppet
---

Sometimes you may wish to use variables in order to call the correct Puppet resources. Especially when you use several if-else conditions to call the correct Puppet resource.

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

As you can see this is quite exhausting to only keep track of such a file.

So instead of maintaining such a complex file I am calling the right class directly by using variables in the resource call:

{% highlight puppet linenos %}
Resource["aem_curator::install_${aem_profile}"]  { "${aem_id}: Install AEM profile ${aem_profile}":
  aem_artifacts_base      => $aem_artifacts_base,
  aem_base                => $aem_base,
}
{% endhighlight %}

As you can see to call a Puppet resource with a variable name in it, we need to call the puppet resource within `Resource[]`. This allows us to use variables to call Puppet resources like Classes or modules.

In my case I reduced a manifest of 280 lines down to 40 lines [Example](https://github.com/shinesolutions/puppet-aem-curator/commit/87b303f39759b49a308f4e60c9d700cbfbaa8554#diff-fcd29db3229524a51a7f2645715cdaec).

Cheers
