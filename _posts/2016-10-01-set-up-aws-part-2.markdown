---
layout: post
title:  "Setting up AWS for Game Development, Part 2"
date:   2016-10-01 09:3ç0:00 -0700
categories: aws ec2 active-directory
author: keith
---

Before I jump into Terraform, I wanted to go over how we set up our Ubuntu images for use in AWS. There are a lot of ways to
do this, but I prefer to use Active Directory to manage users. The reason is simple, I want one place to manage users everywhere, 
and almost every tool we use has an Active Directory or LDAP plugin. Remember, you want to be able to spin machines up on the fly,
and not have to worry about how you will be able to access these machines once they are launched. I'm not going to go into how to 
set up AD, as there are guides everywhere for that. I'm just going to go over how I set up Ubuntu/EC2 to use AD.

**Note:** AWS does have the Simple AD service, which we tried, but it just wasn't ready for what we wanted yet. I plan on 
revisiting it at a later date.

Make sure to replace AD.EXAMPLE.COM and DC.AD.EXAMPLE.COM with your AD domain and domain controller information.

1. Set up the sshPublicKey AD attribute, following the excellent directions [here](https://www.balabit.com/documents/scb-latest-guides/en/scb-guide-admin/html/proc-scenario-usermapping.html).
2. Add the users you want to have sudo access to a 'devops' AD group (create if needed). Also add the users SSH public key to the new 'sshPublicKey' attribute as well.
3. Set up a Kerberos keytab file on an existing Ubuntu instance.

        $ ktutil
        ktutil:  addent -password -p Administrator@AD.YOURDOMAIN.COM -k 1 -e RC4-HMAC
        Password for Administrator@AD.YOURDOMAIN.COM:
        ktutil:  wkt Administrator.keytab
        ktutil:  q
4. Upload `Administrator.keytab` to S3 and grab the S3 bucket and file path.
5. Create an IAM role and give it S3 read permissions.
6. Create an EC2 t2.micro instance using Ubuntu 16.04 as the base image. Be sure to give it the IAM role you just created.
7. Run the user-data.sh script with the following values:
    * **DOMAIN_CONTROLLER** - name of the AD DC (e.g. dc.ad.example.com)
    * **DOMAIN_CONTROLLER_IP** - IP of the machine above
    * **AD_HOST** - AD domain (e.g. ad.example.com)
    * **BASE_DN** - Base DN for AD search (e.g. "dc=ad,dc=example,dc=com")
    * **S3_BUCKET** - S3 bucket where Administrator.keytab was stored
    * **SEARCH_DOMAINS** - DNS search domains, can be empty (e.g. "example.com search.local")

    {% gist SingleMalt/a03355969ef2831d2c0294cbc115bccc %}

To test, just `su - <ad_user>` and log in. You can also run `id <ad_user>` to see the id and groups of a user. 

Sources:

* <http://www.itadmintools.com/2011/07/creating-kerberos-keytab-files.html>
* <http://funwithlinux.net/2014/04/join-ubuntu-14-04-to-active-directory-domain-using-realmd/>
* <https://www.balabit.com/documents/scb-latest-guides/en/scb-guide-admin/html/proc-scenario-usermapping.html>
* <https://blogs.aws.amazon.com/security/post/Tx1XG3FX6VMU6O5/A-safer-way-to-distribute-AWS-credentials-to-EC2>
* <http://serverfault.com/questions/653792/ssh-key-authentication-using-ldap>

{% include author.html %}