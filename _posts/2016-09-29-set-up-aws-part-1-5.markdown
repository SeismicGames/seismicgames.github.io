---
layout: post
title:  "Setting up AWS for Game Development, Part 1.5"
date:   2016-09-29 13:50:00 -0700
categories: aws lambda ec2 cloudwatch
author: keith
---

Ok, so my friend Brian was [correct]({% post_url 2016-09-13-set-up-aws-part-1 %}), and Cloudwatch Events are the right way to handle EC2 scaling. 
 With that, here is an [updated](https://github.com/SeismicGames/lambda_cloudwatch_ec2_events) AWS Lambda handler for EC2 Cloudwatch Events. 
 Please check it out. 

Step 2 will be setting up and using [Terraform](https://www.terraform.io/), now that this has been finished :)

{% include author.html %}
