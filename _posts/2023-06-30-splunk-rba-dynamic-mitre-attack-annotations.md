---
layout: post
title: 'Splunk RBA: Dynamic Mitre ATT&CK Annotations'
date: 2022-11-12
permalink: /splunk-rba-dynamic-mitre-annotations/
categories: ["Walk-through"]
tags: ["rba", "enterprise security", "mitre att&ck", "sandfly security"]
image:
    path: /assets/img/rba-mitre/rba-mitre-cover.png
    lqip: /assets/img/rba-mitre/rba-mitre-cover.png
---

How to dynamically add [MITRE ATT&CK](https://attack.mitre.org/){: target="blank" } annotations from an existing data source to use with [Splunk Risk Based Alerting (RBA)](https://www.splunk.com/en_us/blog/security/risk-based-alerting-the-new-frontier-for-siem.html){: target="blank" }.

RBA has been a game changer for increasing the fidelity of alerts you receive while substantially reducing false positives. When setting up a Risk Rule (RR) with RBA, we can annotate the event to align with a Cyber Security Framework (CSF). But what happens if we want to leverage the CSF mappings that an existing data source already provides? This is precisely what we want to take a closer look at today.

## The Challenge

At the time of writing, populating MITRE ATT&CK tactic/techniques (or other CSF annotations) dynamically from a data source is currently not supported. Instead, the annotations are set statically when creating a correlation search. To make it dynamic based on fields from the events, we need to normalize the fields to which the Risk Framework expects using SPL.

We will look at a specific data source that analyzes an endpoint and maps events to the MITRE ATT&CK framework. Next, we will incorporate these mappings into the Risk Framework within Splunk Enterprise Security. We will be looking at [Sandfly Security](https://www.sandflysecurity.com/){: target="blank" } for our data source. Sandfly Security is an agentless Linux scanner that can be turned on in seconds and won't impact performance or reliability. They also have a free tier that allows scanning a few hosts to help evaluate the product (<https://www.sandflysecurity.com/>{: target="blank" }). Be sure to check them out!

## How it works

To make [CSF annotations dynamic](https://docs.splunk.com/Documentation/ES/latest/Admin/Configurecorrelationsearches#Use_security_framework_annotations_in_correlation_searches){: target="blank" }, we must normalize the data's fields to the following.

| Field Name | Expected Value | Example |
|:-----------|:---------------|:--------|
| `annotations.mitre_attack` | MITRE Technique ID | T1021 |
| `annotations._frameworks` | Cyber security framework used | mitre_attack |
| `annotations._all` | All CSF mappings | T1021 |

> For this use case, both `annotations.mitre_attack` and `annotations._all` will be identical since the data produced by Sandfly only maps to the MITRE ATT&CK framework.
{: .prompt-info }

The following walkthrough will use Sandfly Security data as the source, but theoretically, the concepts can be applied to any data source that provides CSF mappings.

## Getting Started

The first thing we need to do is build a Risk Rule in Splunk Enterprise Security. A Risk Rule (RR) is a correlation search that only has the "Risk Analysis" action as opposed to the traditional alerts that have the "Notable" action. We will use this Risk Analysis action to create a risk object and assign it a score. We will not, however, add any MITRE ATT&CK annotations at this stage. Instead, we will accomplish this in the SPL.

### Building the search

> Update for Splunk Enterprise Security > 7.3 [Jump to section](#updated-search-for-es--73)
{: .prompt-tip }

We will build a search to generate a risk event for Sandfly Alarms. These alarms occur when Sandfly detects suspicious activity on an endpoint. For this article, we will not go into detail about the construction of this search. Instead, we will look at the elements that allow us to map to the Risk Framework.

Starting from line three of the below image _(Figure 1)_, we extract the MITRE ATT&CK technique ID followed by a rename and eval statements to map the expected fields. The example output from this search can be seen in Figure 2.

![RR - Sandfly Alarms](/assets/img/rba-mitre/rr-sandfly-alarms.png){: .shadow w="800" h="500" }
_Figure 1: RR - Sandfly Alarms_

![RR - Sandfly Alarms output](/assets/img/rba-mitre/rr-output.png){: .shadow w="800" h="500" }
_Figure 2: RR - Sandfly Alarms output_

Now that we have a search that extracts the fields we need, we can put together the correlation search. Again, I will not go into detail on how to do this, but the following is an example of the Risk Analysis action _(see Figure 3)_.

![Risk Analysis](/assets/img/rba-mitre/risk-analysis.png){: .shadow w="800" h="500" }
_Figure 3: Risk Analysis action_

This generates a risk event with the correct fields we need to move forward _(see Figure 4)_.

![Risk event](/assets/img/rba-mitre/risk-event.png){: .shadow w="800" h="500" }
_Figure 4: Example risk event_

### Create the Risk Incident Rule

The next thing we need to do is build a Risk Incident Rule (RiR). This will allow us to analyze the risk events produced by the Risk Rule we just created.

The following is a RiR _(Figure 5)_ that looks at the Risk data model for only the sandfly events. On line four, notice the threshold of three tactics and more than two different signatures. We will run this search over the last seven days to identify suspicious behavior.

![RiR - Sandfly ATT&CK Tactic Threshold Exceed for Object over previous 7 days](/assets/img/rba-mitre/rir-sandfly.png){: .shadow w="800" h="500" }
_Figure 5: RiR - Sandfly ATT&CK Tactic Threshold Exceeded for Object over previous 7 Days_

This RiR will generate a Risk Notable on the Incident Review dashboard that looks like the following _(see Figure 6)_.

![Incident Review](/assets/img/rba-mitre/incident-review.png){: .shadow w="800" h="500" }
_Figure 6: Incident Review Dashboard_

We can then view all the risk events in the Risk Event Timeline to further analyze the activity. Notice that in both the Incident Review dashboard and the Risk Event Timeline, the Technique IDs we extracted are automatically enriched with their associated Tactic and Tactic IDs _(see Figure 7)_.

![Risk Event Timeline](/assets/img/rba-mitre/risk-timeline.png){: .shadow w="800" h="500" }
_Figure 7: Risk Event Timeline_

## Updated Search for ES > 7.3

Later versions of Splunk Enterprise Security have a much simpler process for add annotations via SPL. Now we can just target the MITRE Technique ID and use a json object to format the data the way we need.

### Before

```python
...
| rename mitre_attack AS annotations.mitre_attack
| eval 
    "annotations._frameworks"="mitre_attack",
    "annotations._all"='annotations.mitre_attack',
...
```
{: file="SPL" }

### After

```python
...
| eval annotations=case(isnotnull(mitre_attack), json_object("mitre_attack", mitre_attack)),
...
```
{: file="SPL" }

## Conclusion

Splunk RBA allows us to take a different approach to the way we generate alerts by allowing us to chain together suspicious behavior. Leveraging data sources, such as [Sandfly Security](https://www.sandflysecurity.com/){: target="blank" }, which contains MITRE ATT&CK framework mappings, will enable us to significantly enhance how we alert on events. Applying the simple principles of extracting the Technique IDs using SPL during our rule creation allows us to dynamically set our CSF mappings without the need to "hard code" the annotations when setting up the correlation search. Although this example was explicitly designed for Sandfly Security, the concepts can be applied to any data source that provides Cyber Security Framework annotations.

---

> We have a whole community dedicated to Splunk RBA. Feel free to join us and our upcoming meetings!
>
> Visit <https://rba.community/>{: target="blank" } to learn more.
> 
> ![The RBA Community](/assets/img/rba-community-light.png){: .light w="200" }
> ![The RBA Community](/assets/img/rba-community-dark.png){: .dark w="200" }
{: .prompt-info }