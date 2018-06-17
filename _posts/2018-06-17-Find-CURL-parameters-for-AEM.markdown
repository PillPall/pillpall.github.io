---
layout: post
title:  "Find cURL parameters for AEM"
date:   2018-06-17 12:00:00 +1000
categories: AEM
---

To determine which URL and parameters we need to manage AEM via cURL or similar(Ruby/Python) we simply do the steps manually via a browser while using the **Web-Developer tool Network** from the browser or we use something like **tcpdump** or **wireshark**.

<!--excerpts-->

## Example create a truststore
To find out what we need to create a Truststore in AEM via cURL we need to do the following steps.
* Browse to http://localhost:4502/libs/granite/security/content/useradmin.html
* Edit a User
* Open Web-Developer tool Network in Firefox
* Under **Account settings** click on **Create TrustStore**

Now you will see in the Network tool a **POST** call/method:
![Network Console]({{ "/assets/posts/aem_cURL/1_network_console.png" | absolute_url }})

When you click on it you will see the **request url** and the **request method**. Which gives us the following cURL command so far:

```
curl -u 'admin:admin' -X POST http://localhost:4502/libs/granite/security/post/truststore
```

To know which options we need to add to our cURL command we need to click on **Edit and Resend**. In the **request body** we will see all options we need to add to our cURL command.
![Request Body]({{ "/assets/posts/aem_cURL/2_new_request.png" | absolute_url }})

The request body contains the parameter we need to add to the cURL command. It will look something similar to
```
newPassword=admin&rePassword=admin&%3Aoperation=createStore&%3Acq_csrf_token=eyJleHAiOjE1MjkyMzM2MzMsImlhdCI6asdfUyOTIzMzAzM30.TnWwTuYva_kpoLnS0x_Y_asdfAQNuOwYMVCE
```

We can delete everything after **&%3Acq_csrf_token** as we do not provide a authentication token which will give us the following request body

```
newPassword=admin&rePassword=admin&%3Aoperation=createStore
```

Now we have all Parameters we need to know to create a Truststore via cURL **newPassword**, **rePassword** & **:operation**. Our cURL command to create a Truststore is as follows:
```
curl -u 'admin:admin' -X POST 'http://localhost:4502/libs/granite/security/post/truststore?newPassword=admin&rePassword=admin&%3Aoperation=createStore'
```
or:
```
curl -u 'admin:admin' -F 'newPassword=admin' -F 'rePassword=admin' -F ':operation=createStore' http://localhost:4502/libs/granite/security/post/truststore
```


Following these steps you may be able to create your own cURL command.

Cheers!
