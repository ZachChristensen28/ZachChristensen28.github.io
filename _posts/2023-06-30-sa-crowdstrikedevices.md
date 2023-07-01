---
layout: post
title: SA-CrowdstrikeDevices
date: 2022-09-19
categories: ["Splunk Apps"]
tags: ["crowdstrike", "assets", "enterprise security", "supporting add-on"]
image:
    path: /assets/img/sa-crowdstrikedevices/sa-crowdstrikedevices-cover.png
    lqip: /assets/img/sa-crowdstrikedevices/sa-crowdstrikedevices-cover.png
    alt: A supporting add-on for Splunk Enterprise Security.
---
Quickly populate your asset database with data from CrowdStrike.

I have been working with [Splunk Enterprise Security (ES)](https://www.splunk.com/en_us/products/enterprise-security.html){: target="blank" } for over 5 years now and a recurring theme I run into with customers is that **setting up Assets and Identities is hard!** You may feel the same way, not knowing where to turn.

## What is the Asset Database in Splunk ES?

In Splunk Enterprise Security, the [Asset & Identity framework](https://docs.splunk.com/Documentation/ES/latest/Admin/Addassetandidentitydata){: target="blank" } is a fundamental component that allows for an IP to map to a hostname. It also can provide rich context about the operating system, device type (server, workstation, etc.), and even if the system has vulnerabilities. A properly configured asset database will provide a security analyst with a wealth of context when investigating security incidents. Having your asset database configured is also strongly recommended if you plan to use [Risk Based Alerting](https://lantern.splunk.com/Security/Product_Tips/Enterprise_Security/Implementing_risk-based_alerting){: target="blank" } with Splunk ES.

## The Challenge

Even if you have been using Splunk ES for some time you may find that you still have an empty or incomplete asset database. Trying to find the right data source to use becomes a challenge and is compounded by the [SPL](https://docs.splunk.com/Splexicon:SPL){: target="blank" } and lookup tables that are needed to create and maintain the database.

To tackle the first challenge of using a good data source, [CrowdStrike](https://www.crowdstrike.com/){: target="blank" } device data can be brought into Splunk along with the standard detection events. CrowdStrike provides rich information about the devices it is deployed to that we can use in the asset database. The next issue becomes, how can this device data be configured for Splunk ES?

## SA-CrowdStrikeDevices: The Easy Button

I created the SA-CrowdStrikeDevices supporting add-on for Splunk Enterprise Security to streamline the process of populating the asset database with information provided by CrowdStrike. To begin ingesting the device data in Splunk you first need the [CrowdStrike Falcon Devices Technical Add-On](https://splunkbase.splunk.com/app/5570/){: target="blank" } which is built and maintained by CrowdStrike. Once the device data is in Splunk, the SA-CrowdStrikeDevices add-on can be used to bridge the gap between ingesting the device data into Splunk to actually being able to use it in Splunk ES.

## App Setup is easy and only takes three steps:

1. Bring in device data using the [CrowdStrike-built Devices Technical add-on](https://splunkbase.splunk.com/app/5570/){: target="blank" }.
2. Install [SA-CrowdStrikeDevices](https://splunkbase.splunk.com/app/6573/){: target="blank" } to an Enterprise Security search head.
3. Update the default [search macro](https://docs.splunk.com/Splexicon:Searchmacro){: target="blank" } if the index you are using for the CrowdStrike device data is not `index=crowdstrike`. 

Additional [configurations](https://splunk-sa-crowdstrike.ztsplunker.com/configure/){: target="blank" } can be made (and are recommended) but the big hurdle of initial setup is completed by this new add-on.

## Resources

This add-on is developed and maintained under my personal GitHub account and is not affiliated with or sanctioned by the Splunk or CrowdStrike teams. If you are familiar with Splunk on a technical level, feel free to fork the GitHub branch and submit a pull request. If not, you can simply submit an issue or feature request.

This app has passed [Splunk AppInspect](https://dev.splunk.com/enterprise/docs/developapps/testvalidate/appinspect/){: target="blank" } and is cloud ready.

- Splunkbase: <https://splunkbase.splunk.com/app/6573/>{: target="blank" }
- GitHub: <https://github.com/ZachChristensen28/SA-CrowdStrikeDevices>{: target="blank" }
- Documentation: <https://splunk-sa-crowdstrike.ztsplunker.com/>{: target="blank" }
- Prerequisites: <https://splunk-sa-crowdstrike.ztsplunker.com/quickstart/prerequisites/>{: target="blank" }