---
layout: post
---

Most recently, [CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44228) pushed all of us into adopting slapdash measures to protect our applications. One such newfound technique that gained some traction was WAF or the Web Application Firewall. A WAF protects your web applications or APIs against common web exploits that may affect availability, compromise security, or consume excessive resources. AWS supports this via the [AWS WAF](https://aws.amazon.com/waf/) offering and provides the payer accounts with the flexibility to deploy them centrally on the child accounts using [AWS Firewall Manager](https://aws.amazon.com/firewall-manager/). My experience with both of these offerings is quite limited but a single iteration of the deployment cycle for both of these services highlighted a few glaring gotchas hidden in plain sight or sometimes missing from the docs. I hope this guide will help the folks in security or infrastructure to make more informed deployments. 👀



## 8KB Payloads

If you take a peek at the AWS docs for WAF [here](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-fields.html), you would find a charming red box with the following message:

> Only the first 8 KB (8,192 bytes) of the request body are forwarded to AWS WAF for inspection.                                                                 For information about how to manage this, see [Web request body inspection](https://docs.aws.amazon.com/waf/latest/developerguide/web-request-body-inspection.html). 

I was quite puzzled the first time I read this. It immediately makes you think; all someone needs to do is pad the malicious payload with 8KB of data and AWS WAF would just let it through. This detailed blog [post](https://osamaelnaggar.com/blog/aws_waf_dangerous_defaults/) does an excellent job at demonstrating this situation. The desultory solution by AWS requires you to block the requests with >8KB of payload data. Technically, if you want your application to allow 8KB or more at any point, you're out of luck. I like to believe that most of us read the docs thoroughly before deploying any AWS service, but in a situation contrary to my beliefs, this gotcha has the potential to break applications. 

<br />

## Replacement Discrepancy

When using the AWS Firewall Manager to centrally deploy WAF WebACLs within all the child accounts, you encounter the following options in the policy configuration stage.
<br />
<p align="center">
<img src="https://raw.githubusercontent.com/lprimeroo/lprimeroo.github.io/master/aws-waf-1.jpg" style="border:3px solid black">
</p>
<br />

Basically, if you have an ALB or API Gateway resource running without a WAF WebACL in a child account, selecting the auto-remediate option will automatically move those resources behind the WebACL created by this policy. However, the gotcha is in the following line:

> Replace web ACLs that are currently associated with in-scope resources with the web ACLs created by this policy

In theory, if you keep this option unchecked, any resources behind an existing independent WAF WebACL will remain untouched by the WebACL created by the Firewall Manager policy. This is important because the independent WebACLs within the child accounts may already be running custom rules. This theory holds true as long as the independent WebACLs are running WAFv2. If the independent WebACLs are set up using the now deprecated WAF Classic, leaving this option unchecked is a futile effort. I discovered it the hard way when a bunch of applications behind WAF Classic got moved over to the WAFv2 policy created by the Firewall Manager despite leaving this option unchecked. Most of those WAF Classic instances had a number of custom rules and rate limits, none of which got transferred to the new WebACL, and eventually broke their protection. 

When I reached out to AWS support regarding this issue, they were initially in denial. After they had done their fair share of POC work to replicate the issue, they accepted it and suggested to use tags to exclude the resources from any Firewall manager policy in the future. Unsurprisngly, there is no mention of this issue or its remediation on their documentation site.  

<br />

## Migration Caveats

While Firewall Manager will sneakily migrate your resources from WAF Classic to WAFv2 (read last section), there is no easy way of doing it manually or rather independently. Well, there is an entire [page](https://docs.aws.amazon.com/waf/latest/developerguide/waf-migrating-procedure-automatic.html) within the AWS documentation about carrying out this migration using a CloudFormation template, but it also mentions a few caveats. This template or any other way of migrating will NOT move over your custom rules, managed rules, rate limit rules, or logging mechanisms as mentioned [here](https://docs.aws.amazon.com/waf/latest/developerguide/waf-migrating-caveats.html). Almost makes me think, if it would be faster to spin up your own Terraform or CF rather than using the half-assed migration steps by AWS? 🤔


<br />

## False Positives

We've been running AWS WAF on a number of services for the past few months. We also use the WAF offerings by Fastly, Akamai, and Cloudflare to protect some of our projects and often notice AWS WAF to have a noticeably high false positive rate compared to its counterparts. A Web Application Firewall is not a "live and let live" kinda tool. You have to constantly monitor the false positives and false negatives and tweak the rules accordingly, for the desired level of protection. And AWS WAF is certainly not an exception.

<br />


## Bad Parsing

In a weird bug that we encountered in production, a bunch of legitimate multi-part requests with XML data were being blocked by the WAF. Upon further investigation and confirmation from the AWS Support, it seems like their exists a parsing error within AWS WAF, that as of the date of this post hasn't been fixed. As it turned out, a compressed XML payload without line breaks gets blocked, whereas, augmenting the XML with line breaks allows the request to go through the WebACL. Definitely, one of those head-scratching issues.

<br />

## Extra Pennies per Month 

This gotcha isn't a massive one but belongs more in the "uncomfortable" zone. The Firewall Manager is going to create WebACLs in the child accounts irrespective of whether those accounts have any resources (ALB, APIGW, etc) or the replacement box is unchecked. A WebACL costs $5 a month. If we deploy just a single rule within that WebACL, it'll cost us $1 extra. Consider you have 10 accounts that don't need a WAF now but may require one in the future and that's why you have them added as child accounts in the Firewall Manager policy.  Until that day arrives, you would be paying 10 x 6 = $60 extra every month. This is probably pennies for companies even if they have 100+ accounts that don't need a WAF. I just had to put it out there. Thx.
