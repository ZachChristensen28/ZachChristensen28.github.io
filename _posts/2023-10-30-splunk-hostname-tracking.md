---
layout: post
title: Hostname Tracking in Splunk
date: 2023-10-30
categories: ["Experiments", "Walk-through"]
tags: ["assets", "advanced"]
image:
    path: /assets/img/hostname-tracking/hostname-tracking-cover.webp
    lqip: /assets/img/hostname-tracking/hostname-tracking-cover.webp
    alt: Track hostnames over time
---

Keeping track of the relationship between hostnames and IP addresses over time gives important context during incident response or any other forensic activity. This article will walk-through one way of how to accomplish this in Splunk.

## The Problem

It may be challenging to look back in your logs and identify what a hostname was for the ip `192.168.24.20` three months ago. We could create a dashboard to correlate the DHCP events (or another data source) around the time of the event and output the hostname at that time. But what if there was another way. A way to automatically output the hostname during that time to the event during search-time. 

## Solution Walk-through

The following details how to accomplish this using [time-based lookups](https://docs.splunk.com/Documentation/Splunk/latest/Knowledge/Configureatime-boundedlookup){: target="blank" } in Splunk.

![Hostname Switch](/assets/img/hostname-tracking/hostname-tracking-example.png){: .shadow w="800" h="500" }
_Figure 3: Expected outcome_

### What we will need

1. A data source that provides mappings from hostname to IP addresses.
    - Examples:
      - CrowdStrike Device logs
      - DHCP logs
      - DNS logs
      - etc...
2. A KVStore lookup.
3. A scheduled search to update the hostname mappings.

### Data Source

The data source choice will come down to what is available in your environment. We are really looking for a data source that provides accurate IP to Hostname mapping. For this example, I will be leveraging a built-in external command to query my DNS servers in real-time for the hostname of an IP.

> This method will put significant load on your DNS servers in large environments. It is recommended to use an existing data source in Splunk to correlate the hostname. 
{: .prompt-warning }

### KVStore lookup

[Splunk KVStores](https://docs.splunk.com/Documentation/Splunk/latest/Admin/AboutKVstore){: target="blank" } are a great way to store content that changes frequently. Compared to CSV lookups, KVStores can handle large amounts of records efficiently and are well-suited for this use case.

> KVStores are typically configured from configurations files, but an app like the [Splunk App for Lookup File Editing](https://splunkbase.splunk.com/app/1724){: target="blank" } can be used to configure them from the Splunk web interface.
{: .prompt-tip }

#### Configuration files

**collections.conf**

```python
[zts_ip_hostname_tracker_collection]
field.clientip              = string
field.clienthost            = string
field._time                 = time
accelerated_fields.clientip = {"clientip": 1}
```
{: file="collections.conf" }

In this case we are keeping it simple and only storing the IP, hostname, and the time. Update the stanza's name `zts_ip_hostname_tracker_collection` to the name you would like to use.

**transforms.conf**

```python
[zts_ip_hostname_tracker]
external_type = kvstore
collection    = zts_ip_hostname_tracker_collection
fields_list   = _key, _time, clientip, clienthost
time_field    = _time
```
{: file="transforms.conf" }

Define the lookup and add the `time_field` to be equal to the time field in the collections.conf file. Again you can change the stanza `zts_ip_hostname_tracker` to a name you would prefer. 

### Scheduled Search

To populate the KVStore lookup and to keep it up to date, we need to create a search using our data source of choice.

In my case, I will be using firewall data that I will then be performing a reverse IP lookup against my DNS servers. 

#### Base Search

```python
index=firewall (`zts_local_ip(src)` OR `zts_local_ip(dest)`) _index_earliest=-6m@m _index_latest=-1m@m
```
{: file="SPL" }

`zts_local_ip(1)`: [Search macro](https://docs.splunk.com/Documentation/Splunk/latest/Knowledge/Usesearchmacros){: target="blank" } that expands to my internal IP ranges.

```python
$ip$ IN("10.0.0.0/8","172.16.0.0/12","192.168.0.0/16") 
```
{: file="zts_local_ip(1) search macro" }

The use of `_index_earliest/latest` is used to look at that event index time rather than the event timestamp. 

#### Define fields and table

```python
| eval 
    local_src=case(cidrmatch("10.0.0.0/8", src), src, cidrmatch("172.16.0.0/12", src), src, cidrmatch("192.168.0.0/16", src), src),
    local_dest=case(cidrmatch("10.0.0.0/8", dest), dest, cidrmatch("172.16.0.0/12", dest), dest, cidrmatch("192.168.0.0/16", dest), dest),
    clientip=mvappend(local_src, local_dest)
| stats max(_time) as _time count by clientip
```
{: file="SPL" }

`local_src` and `local_dest` are being defined to collect the internal IP in the event. `clientip` then combines those fields into one field. The stats command tables each client ip with the latest time. 

#### DNS lookup

```python
| lookup dnslookup clientip OUTPUT clienthost
| stats max(_time) as _time by clientip clienthost
```
{: file="SPL" }

`dnslookup` is a built-in external command that will take in a `clientip` and output `clienthost`. This will query the configured DNS servers of the OS. We then pipe the results into stats to get a list of IPs and hostnames by the latest time.

#### Output to our lookup

```python
| outputlookup append=true zts_ip_hostname_tracker
```
{: file="SPL" }

Using the `outputlookup` command we write the results to our KVStore file `zts_ip_hostname_tracker`. Be sure to update the lookup name to the one you chose. `append=true` is set to not overwrite the lookup but just to add to it each time the search runs. 

#### Full example

```python
index=firewall (`zts_local_ip(src)` OR `zts_local_ip(dest)`) _index_earliest=-6m@m _index_latest=-1m@m
| eval 
    local_src=case(cidrmatch("10.0.0.0/8", src), src, cidrmatch("172.16.0.0/12", src), src, cidrmatch("192.168.0.0/16", src), src),
    local_dest=case(cidrmatch("10.0.0.0/8", dest), dest, cidrmatch("172.16.0.0/12", dest), dest, cidrmatch("192.168.0.0/16", dest), dest),
    clientip=mvappend(local_src, local_dest)
| stats max(_time) as _time count by clientip
| lookup dnslookup clientip OUTPUT clienthost
| stats max(_time) as _time by clientip clienthost
| outputlookup append=true zts_ip_hostname_tracker
```
{: file="SPL" }

**Example output**

| clientip | clienthost | _time |
|:---------|:-----------|:------|
10.0.10.100 | myhost34 | 2023-10-30 12:22:48.605
192.168.24.154 | myhost12 | 2023-10-30 12:10:32.891
192.168.24.20 | myhost77 | 2023-10-30 12:22:49.343

#### Schedule

Save your search as a Report and to run over "All Time" (since we have our _index_earliest/latest set). In my environment I set this to run every five minutes with the Highest Priority to ensure it always run when it needs to. Determine the best window for your search to run in your environment.

## Optimize

Now that we have our search creating records in our lookup table. We can see that this lookup table will grow exceptional large very quickly. Here are two strategies for keeping your lookup size from growing to large. 

1. Create a retention policy based on time.
    - You can create a secondary search that looks at your lookup table and "cleans" out old records by time. For example, after 90 days the record will be removed.
2. <small><strong>(Definitley do this)</strong></small> Update the search we just created to write to the lookup **_only_** when the hostname has changed.

### Optimize scheduled search

All we need to do to write to the lookup table only when there is a change of hostname is to perform an additional lookup and do the check.

```python
| lookup zts_ip_hostname_tracker clientip OUTPUT clienthost as previous_hostname
| where NOT clienthost==previous_hostname
```
{: file="SPL" }

Here we are just looking up the existing hostname and comparing it to the one that was just querired. The "where" condition will then remove any events that have a match allowing us to write to the lookup only when there is a change or new record.

**Full example with change**

```python
index=firewall (`zts_local_ip(src)` OR `zts_local_ip(dest)`) _index_earliest=-6m@m _index_latest=-1m@m
| eval 
    local_src=case(cidrmatch("10.0.0.0/8", src), src, cidrmatch("172.16.0.0/12", src), src, cidrmatch("192.168.0.0/16", src), src),
    local_dest=case(cidrmatch("10.0.0.0/8", dest), dest, cidrmatch("172.16.0.0/12", dest), dest, cidrmatch("192.168.0.0/16", dest), dest),
    clientip=mvappend(local_src, local_dest)
| stats max(_time) as _time count by clientip
| lookup dnslookup clientip OUTPUT clienthost
| lookup zts_ip_hostname_tracker_slim clientip OUTPUT clienthost as previous_hostname
| where NOT clienthost==previous_hostname
| stats max(_time) as _time by clientip clienthost
| outputlookup append=true zts_ip_hostname_tracker
```
{: file="SPL" }

This slight change will keep your lookups from growing out of control. However, you will still want to keep an eye on the size of your lookups. If they grow to large, you can begin experiencing performance issues. See the [Docs](https://docs.splunk.com/Documentation/ES/latest/Admin/TroubleshootperformancelargeKVStore){: target="blank" } for more information.

## Create Auto-lookup

> See [Splunk Documentation](https://docs.splunk.com/Documentation/SplunkCloud/latest/Knowledge/Makeyourlookupautomatic){: target="blank" } for more information on automatic lookups.
{: .prompt-info }

Now that we have our lookup table file populated with hostname information, we can begin applying it to our data for the hostname to automatically be outputted during search time. 

### props.conf

```python
[bro:conn:json]
LOOKUP-zts_src_ip_hostname_tracker = zts_ip_hostname_tracker clientip AS src OUTPUT clienthost AS src_hostname
LOOKUP-zts_dest_ip_hostname_tracker = zts_ip_hostname_tracker clientip AS dest OUTPUT clienthost AS dest_hostname
```
{: file="props.conf" }

In this example the `src` and `dest` fields are being targeted for automatic enrichment with a new field to appear as either `src_hostname` or `dest_hostname`. Keep in mind you can change all of the names and even target additional fields. Also note that this will only work for the `bro:conn:json` sourcetype. 

If you wanted this to apply to all sourcetypes to automatically enrich fields for all data sources, you can change the stanza name to `default`:

```python
[default]
LOOKUP-zts_src_ip_hostname_tracker = zts_ip_hostname_tracker clientip AS src OUTPUT clienthost AS src_hostname
LOOKUP-zts_dest_ip_hostname_tracker = zts_ip_hostname_tracker clientip AS dest OUTPUT clienthost AS dest_hostname
```
{: file="props.conf" }

## Conclusion

With just a small amount of setup, you can enrich your data with hostnames that are time-specific. This way you can search an event from a few months ago and see what the hostname for that IP was at that time. This will provide great context during investigations and will save a lot of time and effort when trying to track down a hostname from a specific time period.