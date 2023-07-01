---
layout: post
title: SA-SentinelOneDevices
date: 2022-09-29
categories: ["Splunk Apps"]
tags: ["sentinelone", "assets", "enterprise security", "supporting add-on"]
image:
    path: /assets/img/sa-sentinelone/sa-sentinelone-cover.png
    lqip: /assets/img/sa-sentinelone/sa-sentinelone-cover.png
    alt: A supporting add-on for Splunk Enterprise Security.
---

The SA-SentinelOneDevices is another supporting add-on to make it easier to get started with the [Asset & Identity framework](https://docs.splunk.com/Documentation/ES/latest/Admin/Addassetandidentitydata){: target="blank" } for Splunk Enterprise Security (ES). Similar to the [SA-CrowdstrikeDevices](http://zachthesplunker.com/sa-crowdstrikedevices){: target="blank" }, this new add-on takes the data ingested from [SentinelOne](https://www.sentinelone.com/){: target="blank" } and allows it to be directly utilized within Splunk ES.

App Setup

The setup is easy and can be accomplished in just a few short steps.

1. Bring in device data using the [SentinelOne App For Splunk](https://splunkbase.splunk.com/app/5433){: target="blank" }.
1. Install SA-SentinelOneDevices to an Enterprise Security search head.
1. Update the default [search macro](https://docs.splunk.com/Splexicon:Searchmacro){: target="blank" } if the index you are using for the 1. SentinelOne device data is not `index=sentinelone`.

Additional [configurations](https://splunk-sa-sentinelone.ztsplunker.com/configure/){: target="blank" } can be made (and are recommended), but most of the work is taken care of automatically!

## Resources

This add-on is developed and maintained under my personal GitHub account and is not affiliated with or sanctioned by the Splunk or SentinelOne teams. If you are familiar with Splunk on a technical level, feel free to fork the GitHub branch and submit a pull request. If not, you can submit an issue or feature request.

This app has passed [Splunk AppInspect](https://dev.splunk.com/enterprise/docs/developapps/testvalidate/appinspect/){: target="blank" } and is cloud ready.

- Splunkbase: <https://splunkbase.splunk.com/app/6612>{: target="blank" }
- GitHub: <https://github.com/ZachChristensen28/SA-SentinelOneDevices>{: target="blank" }
- Documentation: <https://splunk-sa-sentinelone.ztsplunker.com/>{: target="blank" }
- Prerequisites: <https://splunk-sa-sentinelone.ztsplunker.com/quickstart/prerequisites/>{: target="blank" }