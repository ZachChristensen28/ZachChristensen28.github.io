---
layout: post
title: 'SOAR Enrichment: Powershell Encoded Commands'
date: 2023-03-31
permalink: /soar-powershell/
categories: ["Walk-through"]
tags: ["soar", "enrichment", "attack range"]
image:
    path: /assets/img/soar-powershell/soar-powershell-cover.png
    lqip: /assets/img/soar-powershell/soar-powershell-cover.png
---

Splunk SOAR helps you as a security analyst to focus on what's essential, security—taking away meaningless time on tasks that could easily be automated.

As a former security analyst, one thing I found annoying was spending time on activities that I knew I could automate. I eventually created simple scripts to aid in my incident response so I was not spending time just getting the information I needed. And when I say simple, I mean simple. For example, I created a script where I could input an IOC. It would launch multiple browsers plugging in the IOC into multiple reputation sources, so I didn't have to perform the task manually. Not impressive automation, but it saved me a few clicks. After a while, I graduated with more complex automation scripts, but they still weren't as effective as they could be. 

I then came across [Splunk SOAR](https://www.splunk.com/en_us/products/splunk-security-orchestration-and-automation.html){: target="blank" }, known as Phantom at the time. Phantom opened up a new world of automation I had no time to create. I used the community edition to perform simple enrichment tasks and later moved on to using full SOAR capabilities to conditional remediate events based on severity. These automation playbooks would interact with our firewall or EDR solution to perform containment actions in a few seconds. This was especially useful on weekends when I could not complete the tasks until Monday. Since automation is so powerful, I wanted to show a simple workflow that is a big timesaver that has helped many organizations I work with today.

## Day-in-the-life: No Automation

Using an [out-of-the-box search](https://research.splunk.com/endpoint/c4db14d9-7909-48b4-a054-aa14d89dbb19/){: target="blank" } designed for Splunk Enterprise Security, we can detect Powershell commands that have been encoded. As the description for this rule states, "_[Encoded Powershell]_ is typically used by Administrators to run complex scripts, but commonly used by adversaries to hide their code." Find more information about this search at <https://research.splunk.com>{: target="blank" }.

![Malicious PowerShell Process - Encoded Command](/assets/img/soar-powershell/es-ir-encoded-command.png){: .shadow w="900" h="600" }
_[Malicious PowerShell Process - Encoded Command](https://research.splunk.com/endpoint/c4db14d9-7909-48b4-a054-aa14d89dbb19/){: target="blank" }_

Examining the process command line from the output of this alert, we can see a base64 encoded command.

![Base64 encoded command](/assets/img/soar-powershell/base64-string.png){: .shadow w="600" h="800" }
_Base64 encoded command_

The logical next step for me is understanding what this command is doing. To that end, I will navigate to a base64 decoding tool to closely examine what this Powershell command is doing. In this case, I will be using the web-based tool called [Cyberchef](https://gchq.github.io/CyberChef/){: target="blank" }.  

![Result from Cyberchef](/assets/img/soar-powershell/cyberchef-results.png){: .shadow w="800" h="600" }
_Result from Cyberchef_

> Not all analysts have the same skillset, and decoding a base64 command may have taken a junior analyst some time to figure out.
{: .prompt-info }

The output shows that the Powershell command opens up a reverse shell to the IP `116.125.120[.]88` over port 8080. Not looking good so far...

Next, we should check the reputation of this IOC. In this example, I'll use VirusTotal, but it would be pretty standard to check multiple sources. 

![Reputation results from VirusTotal](/assets/img/soar-powershell/vt-results.png){: .shadow w="800" h="800" }
_Reputation results from VirusTotal_

Obviously, I picked a good example of malicious activity, which would probably ruin the rest of this analyst's day with incident response. If we continued to look closer at this IP, we would find that it has recently been involved with Ransomware and other nefarious activities.

From here, we would have more than enough information to know that we need to begin our remediation and mitigation efforts. 

This example was simple, yet it would have still been time-consuming for the analyst, depending on the skill level. And each analyst's workflow may have looked different to reach this conclusion. Next, we will look at how we can use the power of SOAR to perform these same tasks but automate them!

## Day-in-the-life: with Automation

Using the same example and rules from earlier, we will look at how to automate this process to speed up the initial investigation and accelerate our response while maintaining consistency. 

This next section will use the [community edition of Splunk SOAR](https://www.splunk.com/en_us/products/splunk-security-orchestration-and-automation.html){: target="blank" } and [out-of-the-box](https://github.com/phantomcyber){: target="blank" } content to build this automation flow. First, we will design a playbook that mimics the steps we took.

![Splunk SOAR Playbook](/assets/img/soar-powershell/soar-playbook.png){: .shadow w="400" h="600" }
_Splunk SOAR Playbook_

The first thing we will do is decode the base64 encoded command using a pre-built function. These functions come pre-installed when you install Splunk SOAR. Next, using regex, another pre-built function, we will automatically extract any IPv4 addresses for us. Once we know the IPv4 address, we will submit it to VirusTotal for analysis. 

> Although I only use VirusTotal, it would be as easy to configure multiple reputation sources or call a different playbook containing many additional reputation lookups.
{: .prompt-tip }

I will then format the results using a format block, add a note and then use an action block to update the notable event in Splunk Enterprise Security. The following is the output in Splunk ES.

![Partial output from Splunk Enterprise Security](/assets/img/soar-powershell/es-soar-output.png){: .shadow w="600" h="800" }
_Partial output from Splunk Enterprise Security_

As we can see, all the relevant information is presented to me when I look at the event. I no longer have to navigate to multiple tabs or try to figure out how to decode base64. This also empowers entry-level analysts to focus on security and not get caught up in which tools to use, making the investigation process more streamlined and consistent. And as a bonus, looking at the runtime statistics, this only took Splunk SOAR 204ms to do!

![Runtime statistics for playbook](/assets/img/soar-powershell/soar-stats.png){: .shadow w="500" }
_Runtime statistics for playbook_

## That's a wrap

Splunk SOAR helps you as a security analyst to focus on what's essential, security—taking away meaningless time on tasks that could easily be automated. We could have just as quickly added a workflow to auto-contain this system and hunt for other compromised systems in the environment, all using Splunk SOAR playbooks. 

Hopefully, this inspires you more to pursue automation. It's easy to start with simple use cases that provide immediate value. And the Splunk SOAR community edition is free for up to 100 actions a day! That's a lot of automation you can do!

## Tools used

- [Splunk Attack Range](https://github.com/splunk/attack_range){: target="blank" } - This is awesome! Check it out if you haven't already. 
- [Splunk Enterprise Security](https://www.splunk.com/en_us/products/enterprise-security.html?locale=en_us){: target="blank" }
- [Splunk SOAR](https://www.splunk.com/en_us/products/splunk-security-orchestration-and-automation.html){: target="blank" }
- [Splunk Security Content](https://research.splunk.com/){: target="blank" } - Great resource for out-of-the-box content!
