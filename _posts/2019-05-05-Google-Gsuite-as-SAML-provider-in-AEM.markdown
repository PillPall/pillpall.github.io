---
layout: post
title:  "Google Gsuite as SAML provider in AEM"
date:   2019-05-05 12:00:00 +1000
categories: AEM
---

Just recently I had to test my SAML automation for Adobe Experience Manager and faced the problem that I couldn't test it because I don't have a running SAML server I can use for testing. But luckily I found out how I can configure Google Suite as SAML provider in AEM.

<!--excerpts-->

First thing, for more information about SAML in AEM I recommend you this blog post about [SAML and how to configure SAML in AEM](https://shinesolutions.com/2018/12/03/sso-with-saml-authentication-using-shibboleth-idp/).

The steps to configure Google Gsuite as SAML provider in AEM are as follows:

* Create new SAML Application in Gsuite
* Configure Gsuite as SAML provider in AEM

## Create new SAML App in Google Gsuite

To create a new SAML app in Google Gsuite login to your Gsuite Admin account and create a new custom SAML app

* Copy `SSO URL`
* Download Certificate
* `ACS URL` e.g. `https://author.aem-opencloud.com:5432/saml_login`
* `Entity ID` e.g. `AEMSSO`
* Attribute mapping:

|Application Attribute       |Information | User attribute|
| ---|---|---|
| mail       | Basic Information | Primary Email |
| givenName       | Basic Information | First Name |
| familyName       | Basic Information | Last Name |

Now that we configured our SAML application in Google Gsuite we can configure SAML in AEM.

## Configure Gsuite as SAML provider in AEM

For configuring Gsuite as SAML provider in AEM
* Upload certificate to the AEM Global Truststore.
* Configure **Adobe Granite SAML 2.0 Authentication Handler** with the following options:

path: `/`

service.ranking: `5002`

idpUrl: the copied **SSO URL** e.g. `https://accounts.google.com/o/saml2/idp?idpid=C03abcde1f`

idpCertAlias: Certificate aliase name from the Global Truststore


Now that Google Gsuite is successfully configured as SAML provider in AEM you can try to login with your google Account to AEM.

Cheers
