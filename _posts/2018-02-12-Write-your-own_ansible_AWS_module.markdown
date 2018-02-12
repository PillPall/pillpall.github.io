---
layout: post
title:  "Write your own AWS Ansible module"
date:   2018-02-12 12:00:00 +1000
categories: ansible
---
To write our own Ansible module for using AWS services, we can rely on the already integrated utility for AWS which allows us to connect to the AWS services in an very easy way.

Let's assume we want to write a module to download and upload a object from S3 and save it to a path we define.
<!--excerpts-->

## Let's start

...with the fun part first and end with the less fun part.

Open your favourite text editor and create a new empty file named my_s3_module.py.

Add in the top part

{% highlight python linenos %}
#!/usr/bin/python
# Copyright (c) 2018 COPYRIGHT
# GNU General Public License v3.0+ (see COPYING or https://www.gnu.org/licenses/gpl-3.0.txt)
{% endhighlight %}

The first line tells Ansible where to find the python binary and line 2 & 3 are reserved for copyright declaration.

Now we add the python libraries we want to import

{% highlight python linenos %}

from ansible.module_utils.basic import AnsibleModule
from ansible.module_utils.ec2 import ansible_dict_to_boto3_tag_list, AnsibleAWSError, boto3_conn, connect_to_aws, ec2_argument_spec, get_aws_connection_info
import traceback

try:
    import boto3
    HAS_BOTO = True
except ImportError:
    HAS_BOTO = False
{% endhighlight %}

The first line imports the basic AnsibleModule library.
The second line import the important Ansible AWS libraries we use to connect to our AWS service.

We use the try block to import the boto3 library and to declare the variable `HAS_BOTO` with True or False.

After we imported our libraries we add to the end of our file the following code block:

{% highlight python linenos %}

if __name__ == '__main__':
    main()

{% endhighlight %}

Ansible calls our python module as a script and jumps therefore in the  `__main__` scope and calls the method `main`. More information can be found here [(Link)](https://docs.python.org/3/library/__main__.html).

### Module options:
Now we describe our main method by adding our own code in between.

{% highlight python linenos %}
def main():
  argument_spec = ec2_argument_spec()
  argument_spec.update(dict(
      s3_bucket = dict(required=True, type='str'),
      s3_prefix = dict(required=True, type='str'),
      local_dest = dict(default='/tmp/', type='str'),
      mode = dict(required=True, type='str', choices=['download', 'upload']),

  module = AnsibleModule(argument_spec=argument_spec)
  ))
{% endhighlight %}
In the first line we create a new dict `argument_spec` from the imported module `ec2_argument_spec()`. This adds the following AWS options to our Ansible module:

{% highlight python linenos %}
# from the Sourcecode
# https://github.com/ansible/ansible/blob/devel/lib/ansible/module_utils/ec2.py

ec2_url=dict(),
aws_secret_key=dict(aliases=['ec2_secret_key', 'secret_key'], no_log=True),
aws_access_key=dict(aliases=['ec2_access_key', 'access_key']),
validate_certs=dict(default=True, type='bool'),
security_token=dict(aliases=['access_token'], no_log=True),
profile=dict(),
region=dict(aliases=['aws_region', 'ec2_region']),
{% endhighlight %}

So we don't have to care about this anymore.

In the second line we update our dict and add our own module options using the following schema:

{% highlight python linenos %}

argument_spec.update(dict(
  option_name = dict(required=True/False, type='str/bool', default='default-value', choices=['Choice1', 'Choice2']),

{% endhighlight %}

The next line declares `module` to call the Class `AnsibleModule` and pass our arguments, so we can use them in our Ansible module [(more information)](https://github.com/ansible/ansible/blob/devel/lib/ansible/module_utils/basic.py).

### Let's connect to AWS
Now we add the following code block:
{% highlight python linenos %}
  if not HAS_BOTO:
      module.fail_json(msg='boto3 required for this module')

  region, ec2_url, aws_connect_params = get_aws_connection_info(module, boto3=True)

  if not region:
      module.fail_json(msg='region must be specified')

  conn_type = 'resource'
  resource = 's3'

  try:
      s3_connection = boto3_conn(module, conn_type=conn_type,
              resource=resource, region=region,
              endpoint=ec2_url, **aws_connect_params)
  except botocore.exceptions.NoCredentialsError as e:
      module.fail_json(msg='cannot connect to AWS', exception=traceback.format_exc(e))

{% endhighlight %}

Within this code block we check if the boto3 client is available, if not exit with error. If so, we check if we can connect to AWS if not exit with an error.

No we try to connect to our AWS resource. To do so we call the method boto3_conn, which opens a boto3 session, with the type of connection we want. Following are available from the boto3 documentation [(Link)](http://boto3.readthedocs.io/en/latest/reference/core/session.html)

{% highlight python linenos %}
# client
conn_type='client'

# resource
conn_type='resource'
{% endhighlight %}

Than we tell boto3 which resource we want to call from AWS. A list of resource can be found here [(Link)](http://boto3.readthedocs.io/en/latest/reference/services/).

{% highlight python linenos %}
resource='AWS_RESOURCE'
{% endhighlight %}

### What to do ?
Now that we are connected to our AWS Service we actually want to do something

{% highlight python linenos %}
mode = module.params.get("mode")

  if mode == 'download':
      result = download(s3_connection, module)
  elif mode == 'upload':
      result = upload(s3_connection, module)
  else:
      module.fail_json(msg='Error: unsupported mode. Supported modes are download and upload')
{% endhighlight %}

We save the parameter `mode` and check if we want to download or upload something otherwise we print an error with an unsupported mode. Now we start with the method definition download()

### S3 Download
{% highlight python linenos %}
def download(s3_connection, module):

    s3_bucket = module.params.get('s3_bucket')
    s3_prefix = module.params.get('s3_prefix')
    local_dest = module.params.get('local_dest')

    try:
        s3_bucket_connection = s3_connection.Bucket(s3_bucket)
        result = s3_bucket_connection.download_file(s3_prefix, local_dest)
        response = dict(changed=True,
                        item=dict(source='s3://' + s3_bucket + '/' + s3_prefix,
                                  dest=local_dest),
                        message=result)
        return response
    except Exception as e:
        module.fail_json(msg="Error: Can't download file from s3 - " + str(e), exception=traceback.format_exc(e))
{% endhighlight %}

We are passing two options to our module the `connection parameters` and `ansible arguments`. In the first three lines we set some variables. Than we actually try to download our file from the S3 Bucket. As from the boto3 documentation taken [(Link)](http://boto3.readthedocs.io/en/latest/reference/services/s3.html#S3.Bucket.download_file) we need to connect to our `s3_bucket` at first and than we can download the file from `s3_prefix` and save it to `local_dest`. After downloading we create a new dict with the result of our download.

If the download is going to be fail our module will throw an error with an explanation.

### S3 Upload
The next step is to define our upload module.
{% highlight python linenos %}
def upload(s3_connection, module):
    s3_bucket = module.params.get("s3_bucket")
    s3_prefix = module.params.get("s3_prefix")
    local_dest = module.params.get("local_dest")

    try:
        s3_bucket_connection = s3_connection.Bucket(s3_bucket)
        result = s3_bucket_connection.upload_file(local_dest, s3_prefix)
        response = dict(changed=True,
                        item=dict(source=local_dest,
                                  dest='s3://' + s3_bucket + '/' + s3_prefix),
                        message=result)
        return response
    except Exception as e:
        module.fail_json(msg="Error: Can't upload configuration file - " + str(e), exception=traceback.format_exc(e))

{% endhighlight %}

The uploading follows exact the same syntax like the download. More information can be taken from the boto3 Doc[(Link)](http://boto3.readthedocs.io/en/latest/reference/services/s3.html#S3.Object.upload_file)

### Let me out !
After describing our modules we need to add the exit procedure in order to get our Ansible module running and actually to exit properly. Therefore we add at the end of our main method following piece of code
{% highlight python linenos %}
  module.exit_json(**result)
{% endhighlight %}

### Documentation Documentation Documentation
As we all know how important documentation is we need to add the documentation for our Ansible module. As taken from the Ansible Doc [(Link)](http://docs.ansible.com/ansible/latest/dev_guide/developing_modules_documenting.html) the documentation has to be in the top part after the copyright block and before importing the modules.

{% highlight python linenos %}
ANSIBLE_METADATA = {'metadata_version': '1.1',
                    'status': ['preview'],
                    'supported_by': 'community'}
DOCUMENTATION = '''
---
module: my_s3_module
MODULE_DESCRIPTION
requirements:
  - boto3 >= 1.5.0
  - python >= 2.6.0

options:
    s3_bucket:
        description:
          - Name of the S3 Bucket
        required: true
        default: null
    s3_prefix:
      description:
        - Location of the file in the s3 Bucket
      required: true
      default: null
    local_dest:
      description:
        - Local path to store / upload a s3 object
      required: true
      default: /tmp
    mode:
      description:
        - Behaviour of our Ansible module
      required: true
      default: null
      choices: ['download', 'upload']
'''
EXAMPLES = '''
- name: Download a s3 object
  aws_s3_download:
    s3_bucket: mbloch
    s3_prefix: test/hello.txt
    local_dest: ~/hello.txt
    mode: download
  register: dl_output

- name: Upload a s3 object
  aws_s3_download:
    region: ap-southeast-2
    aws_secret_key: MYSecretKey
    aws_access_key: MyAccessKey
    s3_bucket: mbloch
    s3_prefix: test/world.txt
    local_dest: ~/world.txt
    mode: upload
  register: ul_output
'''
RETURN = '''
dest:
    description: Destination path of your downloaded/uploaded file
    returned: (s3://)(/)path/to/download/or/upload
    type: String
    sample: /tmp/hello.txt

source:
    description: Source path of your downloaded/uploaded file
    returned: (s3://)(/)path/to/download/or/upload
    type: String
    sample: /tmp/hello.txt

message:
    descripton: return value of our s3 upload/download
    return: null
    type: null
    sample: null
'''
{% endhighlight %}

### How to use it ?
And done is our Ansible module. Now how can we use it ? create a simple playbook

{% highlight yaml linenos %}
---
- hosts: localhost
  gather_facts: no
  connection: local

  tasks:
    - name: Upload text file
      my_s3_module:
        s3_bucket: mbloch
        s3_prefix: hello.txt
        local_dest: "{{ playbook_dir }}/hello.txt"
        mode: upload

    - name: Download text file
      my_s3_module:
        s3_bucket: mbloch
        s3_prefix: hello.txt
        local_dest: /tmp/hello2.txt
        mode: download

{% endhighlight %}

One important note, the name of your Ansible module inside your playbook is the file name of your python script. As you might remember, I called my script `my_s3_module.py`

Create a new folder called `library` in the folder of your playbook and place your Ansible module inside of it. Ansible will look up for a folder called `library` inside the playbook folder.

### First run
Now if we run our playbook it tries to upload the file `hello.txt` inside of our playbook directory to our s3bucket and afterwards we download it to /tmp/hello2.txt.

{% highlight shell linenos %}
MichaelBloch:ansible mbloch$ aws s3 ls mbloch/
MichaelBloch:ansible mbloch$ ls -l /tmp/hello2.txt
ls: /tmp/hello2.txt: No such file or directory
MichaelBloch:ansible mbloch$ echo 'Hello World!' > hello.txt
MichaelBloch:ansible mbloch$ cat hello.txt
Hello World!
MichaelBloch:ansible mbloch$ ansible-playbook my_s3_module.yaml

PLAY [localhost] *******************************************************************************************************************************************************************************************

TASK [Upload text file] ************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Download text file] **********************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *************************************************************************************************************************************************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0   

MichaelBloch:ansible mbloch$ aws s3 ls mbloch/hello.txt
2018-02-12 21:41:02         13 hello.txt
MichaelBloch:ansible mbloch$ cat /tmp/hello2.txt
Hello World!
{% endhighlight %}


### Conclusions

Neither I'm a good programmer nor I'm well experienced in coding, atm, but I made it to write my own module :)

You can find my source codes here [(Sourc Code)](https://github.com/PillPall/examples/tree/master/ansible-aws-module)
