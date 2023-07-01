---
layout: post
title: 'Splunk ES: Risk Notable Urgency'
date: 2023-01-11
permalink: /risk-notable-urgency/
categories: ["Walk-through"]
tags: ["rba", "enterprise security"]
image:
    path: /assets/img/risk-notable-urgency/risk-notable-urgency-cover.png
    lqip: /assets/img/risk-notable-urgency/risk-notable-urgency-cover.png
---

In Splunk Enterprise Security, the Urgency levels for the out-of-the-box Risk notables will not be assigned correctly. Add this simple solution to fix it.

By default, a risk object's priority is not taken into account for the Urgency of a Notable event, even if it is configured in the Asset and Identity (A&I) database. The Notable Event's Urgency level can help an analyst prioritize which events to begin working on. Although other fields can help filter higher-priority events, it's also a good idea to have the Urgency field to use when needed.

The Urgency level is a combination of an Asset/Identity's priority plus the severity of the event. The default Urgency Lookup can be found in Content Management in the Enterprise Security App. Only the following fields are used to determine priority:

For identities
: `user` or `src_user`

For assets
: `dest`, `src`, or `dvc`

![Notable Urgency Matrix](/assets/img/risk-notable-urgency/notable_urgency.png){: .shadow w="500" h="600" }
_Figure 1: [Urgency Matrix](https://docs.splunk.com/Documentation/ES/latest/User/Howurgencyisassigned?ref=zachthesplunker.com){: target="blank" }_

## Problem Example

In the following example, the risk_object `alexisc@zachthesplunker.com` is listed as a "critical" priority user in the Identity Database _(see Figure 2: Critical Priority User)_.

![Critical Priority User](/assets/img/risk-notable-urgency/identity-center.png){: .shadow w="900" h="300" }
_Figure 2: Critical Priority User - `alexisc@zachthesplunker.com`_

And although the severity of the event is high, a medium level of Urgency is produced _(see Figure 1: Urgency Matrix)_. 

> Expected Result
> 
> **High** severity + **Critical** priority = **Critical** urgency
{: .prompt-info }

> Actual Result
> 
> The actual Urgency level is set to "Medium" since a valid field (user/src_user) is not found within the event. This interprets the user priority as "unknown."
>
> **High** severity + **Unknown** priority = **Medium** urgency
{: .prompt-danger }

![Incident Review](/assets/img/risk-notable-urgency/incident-review.png){: .shadow w="800" h="600" }
_Figure 3: Incident Review - Risk Notable_

## Solution

The solution is quite simple. By adding a few lines of SPL to the Risk Notables, the Urgency level works as intended.

```python
...
| eval 
    user=case(risk_object_type=="user", risk_object),
    src=case(isnull(user), risk_object)
```
{: file="SPL" }

Depending on the `risk_object_type` either the `user` or `src` field will be populated with the risk_object allowing for the urgency level to be set correctly.

> The Urgency level of the event now aligns with the expected results, and we now see a "Critical" urgency.
{: .prompt-tip }

![Incident Review - Correct Urgency](/assets/img/risk-notable-urgency/incident-review-correct.png){: .shadow w="800" h="600" }
_Figure 4: Incident Review - Risk Notable - Correct Urgency_

## Conclusion

By adding a few lines of SPL, you can correct the behavior of Risk Notables by not setting the correct Urgency Level. This will give you another way to triage events and gain the ability to represent criticality in your Incident Review dashboard more accurately.

---

> We have a whole community dedicated to Splunk RBA. Feel free to join us and our upcoming meetings!
>
> Visit <https://rba.community/>{: target="blank" } to learn more.
> 
> ![The RBA Community](/assets/img/rba-community-light.png){: .light w="200" }
> ![The RBA Community](/assets/img/rba-community-dark.png){: .dark w="200" }
{: .prompt-info }