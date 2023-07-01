---
layout: post
title: SA-AwsAssets
date: 2022-11-02
categories: ["Splunk Apps"]
tags: ["aws", "assets", "enterprise security", "supporting add-on"]
image:
    path: /assets/img/sa-awsassets/sa-awsassets-cover.png
    lqip: /assets/img/sa-awsassets/sa-awsassets-cover.png
    alt: A supporting add-on for Splunk Enterprise Security.
---

The SA-AwsAssets is another supporting add-on to make it easier to start with the [Asset & Identity framework](https://docs.splunk.com/Documentation/ES/latest/Admin/Addassetandidentitydata){: target="blank" } for Splunk Enterprise Security (ES). Similar to the [SA-CrowdstrikeDevices](http://zachthesplunker.com/sa-crowdstrikedevices){: target="blank" } and [SA-SentinelOneDevices](http://zachthesplunker.com/sa-sentinelone){: target="blank" }, this new add-on takes the data ingested from [AWS](https://aws.amazon.com/){: target="blank" } and allows it to be directly utilized within Splunk ES.

## App Setup

The setup is easy and can be accomplished in just a few short steps.

1. Ingest AWS data into Splunk and have the [AWS add-on](https://splunkbase.splunk.com/app/1876){: target="blank" } installed.
2. Install [SA-AwsAssets](https://splunkbase.splunk.com/app/6660){: target="blank" } to your Enterprise Security search head.
3. Update the default [search macro](https://docs.splunk.com/Splexicon:Searchmacro){: target="blank" } if the index you are using for the `aws:metadata` sourcetype data is not `index=aws_security`.

Additional [configurations](https://splunk-sa-aws.ztsplunker.com/configure/){: target="blank" } can be made (and are recommended), but most of the work is taken care of automatically!

## Resources

This add-on is developed and maintained under my personal GitHub account and is not affiliated with or sanctioned by the Splunk or AWS teams. If you are familiar with Splunk on a technical level, feel free to fork the [GitHub](https://github.com/ZachChristensen28/SA-AwsAssets){: target="blank" } branch and submit a pull request. If not, you can submit an [issue or feature request](https://github.com/ZachChristensen28/SA-AwsAssets/issues){: target="blank" }.

This app has passed [Splunk AppInspect](https://dev.splunk.com/enterprise/docs/developapps/testvalidate/appinspect/){: target="blank" } and is cloud ready.

- Splunkbase: <https://splunkbase.splunk.com/app/6660>{: target="blank" }
- GitHub: <https://github.com/ZachChristensen28/SA-AwsAssets>{: target="blank" }
- Documentation: <https://splunk-sa-aws.ztsplunker.com/>{: target="blank" }
- Prerequisites: <https://splunk-sa-aws.ztsplunker.com/quickstart/prerequisites/>{: target="blank" }
