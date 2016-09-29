---
layout: post
title:  "Setting up AWS for Game Development, Part 1.5"
date:   2016-09-29 13:50:00 -0700
categories: aws lambda ec2 cloudwatch
---

Ok, so my friend Brian was [correct]({% post_url 2016-09-13-set-up-aws-part-1 %}), and Cloudwatch Events are the right way to handle EC2 scaling. 
 With that, here is an updated AWS Lambda handler for EC2 Cloudwatch Events:
 
<div class="project-wrapper">
    <p align="center">
        <img src="/img/loader.gif" />
    </p>
</div>

<script type="text/javascript">
$( document ).ready(function() {
    var req_data = {"githuburl": "https://raw.githubusercontent.com/gruegames/lambda_cloudwatch_ec2_events/master/README.md"};
    
    $.ajax({
        url: 'https://1r6a82hlr0.execute-api.us-east-1.amazonaws.com/prod/parse',
        method: 'POST',
        data: JSON.stringify(req_data),
        dataType: 'json',
        crossDomain: true,
        headers: {
            'X-Api-Key': 'QGTJTaLhDM8dNi11TC0vCwEIX7luKWM8NKChBrN4'
        },
        contentType: 'application/json',
        success: function(data) {
            $(".project-wrapper").html(data.value);
            $(".project-wrapper").find("pre").addClass("prettyprint");
            prettyPrint();
        }
    });
});
</script>

Step 2 will be setting up and using [Terraform](https://www.terraform.io/), now that this has been finished :)