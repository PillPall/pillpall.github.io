---
layout: post
title:  "Create self-signed Certificate with Cloudformation"
date:   2020-08-27 12:00:00 +1000
categories: AWS
---

AWS Cloudformation allows you to create self-signed certificates during the stack creation process which you can then use to attach it to other AWS resources like an ELB.

I recently discovered two ways of creating self-signed certificates automated with Cloudformation.

<!--excerpts-->

## AWS Certificate Manager Private CA (ACMPCA)

AWS Certificate Manager Private CA(ACMPCA) Service allows you to create and manage a Private CA as easy as it can get. With a Private CA in ACMPCA you are able to create a Self-Signed Certificate in AWS Certificate Manager (ACM).

The integration of ACMPCA in Cloudformation allows you to integrate the creation of a Private CA & the creation of a Self-Signed certificate with ACM into you stack creation process. But keep in mind one Private CA costs around 400$ a month.

### Create Private CA
The process of creating a Private CA with ACMPCA is as follows
* Create Internal Root CA
* Create Internal Root CA Certificate
* Activate internal Root CA
{% highlight yaml linenos %}
InternalRootCA:
  Type: 'AWS::ACMPCA::CertificateAuthority'
  Properties:
    Type: ROOT
    KeyAlgorithm: RSA_2048
    SigningAlgorithm: SHA256WITHRSA
    Subject:
      Country: US
      State: California
      Locality: San Francisco
      Organization: MyOrganization
      OrganizationalUnit: MyOrganizationalUnit
      CommonName: My own Root CA
      SerialNumber: '12345678'
    RevocationConfiguration:
      CrlConfiguration:
        Enabled: false
InternalRootCACertificate:
  Type: 'AWS::ACMPCA::Certificate'
  Properties:
    CertificateAuthorityArn: !Ref InternalRootCA
    CertificateSigningRequest: !GetAtt
      - InternalRootCA
      - CertificateSigningRequest
    SigningAlgorithm: SHA256WITHRSA
    TemplateArn: 'arn:aws:acm-pca:::template/RootCACertificate/V1'
    Validity:
      Type: YEARS
      Value: 5
InternalRootCAActivation:
  Type: 'AWS::ACMPCA::CertificateAuthorityActivation'
  Properties:
    CertificateAuthorityArn: !Ref InternalRootCA
    Certificate: !GetAtt
      - InternalRootCACertificate
      - Certificate
    Status: ACTIVE
{% endhighlight %}

### Create Self-Signed Certificate in ACM
To create your Self-Signed Certificate in ACM you need to wait until the Root CA was activated & you need the ARN of the created Root CA.
{% highlight yaml linenos %}
ACMInternalCertificate:
  DependsOn: InternalRootCAActivation
  Type: AWS::CertificateManager::Certificate
  Properties:
    CertificateAuthorityArn: !Ref InternalRootCA
    DomainName: 'example.com'
    SubjectAlternativeNames:
      - "\*.example.com"
{% endhighlight %}
  is  and the other one is to use a Python Lambda function to create a self-signed certificate and upload it to AWS Certificate Manager.


## Lambda and Custom Resource
If you don't want to create your Private CA in ACMPCA  you can use Lambda and Custom Resources to create a Self-Signed Certificate and store it in ACM. The Python module `pyopenssl` allows you to create a Private CA & Self-Signed Certificate with Python.

{% highlight python linenos %}
import random
from OpenSSL import crypto
# Create CA
ca_key = crypto.PKey()
ca_key.generate_key(crypto.TYPE_RSA, 2048)
ca_cert = crypto.X509()
ca_cert.set_version(2)
ca_cert.set_serial_number(random.randrange(100000))
ca_subj = ca_cert.get_subject()
ca_subj.C = "US"
ca_subj.ST = "California"
ca_subj.L = "San Francisco"
ca_subj.O = "MyOrganization""
ca_subj.OU = "MyOrganizationalUnit"
ca_subj.CN = "My own Root CA"
ca_cert.add_extensions([
    crypto.X509Extension(b"subjectKeyIdentifier", False, b"hash", subject=ca_cert),
])
ca_cert.add_extensions([
    crypto.X509Extension(b"authorityKeyIdentifier", False, b"keyid:always", issuer=ca_cert),
])
ca_cert.add_extensions([
    crypto.X509Extension(b"basicConstraints", False, b"CA:TRUE"),
    crypto.X509Extension(b"keyUsage", False, b"keyCertSign, cRLSign"),
])
ca_cert.set_issuer(ca_subj)
ca_cert.set_pubkey(ca_key)
ca_cert.sign(ca_key, 'sha256')
ca_cert.gmtime_adj_notBefore(0)
ca_cert.gmtime_adj_notAfter(10*365*24*60*60)
ca_certifictate_pem = crypto.dump_certificate(crypto.FILETYPE_PEM, ca_cert).decode('utf-8')

# Create Self-Signed Certificate
client_key = crypto.PKey()
client_key.generate_key(crypto.TYPE_RSA, 2048)
client_cert = crypto.X509()
client_cert.set_version(2)
client_cert.set_serial_number(random.randrange(100000))
client_subj = client_cert.get_subject()
client_subj.C = "US"
client_subj.ST = "California"
client_subj.L = "San Francisco"
client_subj.O = "MyOrganization""
client_subj.OU = "MyOrganizationalUnit"
client_subj.CN = "example.com"
client_cert.add_extensions([
    crypto.X509Extension(b"basicConstraints", False, b"CA:FALSE"),
    crypto.X509Extension(b"subjectKeyIdentifier", False, b"hash", subject=client_cert),
])
client_cert.add_extensions([
    crypto.X509Extension(b"authorityKeyIdentifier", False, b"keyid:always", issuer=ca_cert),
    crypto.X509Extension(b"extendedKeyUsage", False, b"serverAuth"),
    crypto.X509Extension(b"keyUsage", False, b"digitalSignature"),
])
client_cert.add_extensions([
    crypto.X509Extension(b'subjectAltName', False,
        ','.join([
            'DNS:\*.%s' % fqdn
]).encode())])
client_cert.set_issuer(ca_subj)
client_cert.set_pubkey(client_key)
client_cert.gmtime_adj_notBefore(0)
client_cert.gmtime_adj_notAfter(9*365*24*60*60)
client_cert.sign(ca_key, 'sha256')
certifictate_pem = crypto.dump_certificate(crypto.FILETYPE_PEM, client_cert).decode('utf-8')
private_key_pem = crypto.dump_privatekey(crypto.FILETYPE_PEM, client_key).decode('utf-8')
response = {
    'certificate': certifictate_pem,
    'private_key': private_key_pem,
    'certificate_chain': ca_certifictate_pem
}
{% endhighlight %}

Now that you have the self-signed certificate, private key & certificate chain you can use boto to upload it to ACM.

{% highlight python linenos %}
import boto3
acm = boto3.client('acm')
response = acm.import_certificate(
      Certificate=certificate,
      PrivateKey=private_key,
      CertificateChain=certificate_chain,
  )
{% endhighlight %}
