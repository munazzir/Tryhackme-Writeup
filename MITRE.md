MITRE
=====

we will focus on other projects/research that the US-based non-profit MITRE Corporation has created for the cybersecurity community, specifically:

+ ATT&CK® (Adversarial Tactics, Techniques, and Common Knowledge) Framework
+ CAR (Cyber Analytics Repository) Knowledge Base
+ SHIELD (sorry, not a fancy acronym) Active Defense
+ AEP (ATT&CK Emulation Plans)



# APT (Advanced Persistent Threat)

This can be considered a team/group **(threat group)** , or even country **(nation-state group)**, that engages in long-term attacks against organizations and/or countries.
The term 'advanced' can be misleading as it will tend to cause us to believe that each APT group all have some super-weapon, e.i. a zero-day exploit, that they use. That is not the case. As we will see a bit later, the techniques these APT groups use are quite common and can be detected with the right implementations in place.

Ex: [FireEye's current list of APT groups.](https://www.fireeye.com/current-threats/apt-groups.html)


TTP is an acronym for Tactics, Techniques, and Procedures, but what does each of these terms mean?

+ The Tactic is the adversary's goal or objective.
+ The Technique is how the adversary achieves the goal or objective.
+ The Procedure is how the technique is executed.



## ATT&CK®

Framework define according to their website
`"MITRE ATT&CK® is a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations."`

In 2013, MITRE began to address the need to record and document common TTPs (Tactics, Techniques, and Procedures) that APT (Advanced Persistent Threat) groups used against enterprise Windows networks. This started with an internal project known as FMX (Fort Meade Experiment). Within this project, selected security professionals were tasked to emulated adversarial TTPs against a network, and data was collected from the attacks on this network. The gathered data helped construct the beginning pieces of what we know today as the ATT&CK® framework.

ATT&CK® Framework first solely focus on Windows but in overthetime its expanded to cover other platforms, such as macOS and Linux. The framework is heavily contributed to by many sources, such as security researchers and threat intelligence reports. Note this is not only a tool for blue teamers. The tool is also useful for a penetration tester and/or red teamer.

If you haven't done so, navigate to the [ATT&CK®.](https://attack.mitre.org/)

Lastly, the same data can be viewed via the **MITRE ATT&CK® Navigator**: "The ATT&CK® Navigator is designed to provide basic navigation and annotation of ATT&CK® matrices, something that people are already doing today in tools like Excel. So they've  designed it to be simple and generic - you can use the Navigator to visualize your defensive coverage, your red/blue team planning, the frequency of detected techniques, or anything else you want to do."

You can access ATT&CK® Navigator for Carbanak [here.](https://mitre-attack.github.io/attack-navigator//#layerURL=https%3A%2F%2Fattack.mitre.org%2Fgroups%2FG0008%2FG0008-enterprise-layer.json)


## CAR (Cyber Analytics Repository) Knowledge Base

The official definition of **CAR** is "The MITRE Cyber Analytics Repository (CAR) is a knowledge base of analytics developed by MITRE based on the MITRE ATT&CK® adversary model. CAR defines a data model that is leveraged in its pseudocode representations but also includes implementations directly targeted at specific tools (e.g., Splunk, EQL) in its analytics. With respect to coverage, CAR is focused on providing a set of validated and well-explained analytics, in particular with regards to their operating theory and rationale."

You can find some **CAR** analytic reports [here.](https://car.mitre.org/analytics/)

It also have [CAR ATT&CK® Navigator layer](https://mitre-attack.github.io/attack-navigator/beta/enterprise/#layerURL=https%3A%2F%2Fraw.githubusercontent.com%2Fmitre-attack%2Fcar%2Fmaster%2Fdocs%2Fcar_attack%2Fcar_attack.json/) to view all the analytics easily.



## SHIELD Active Defense

Per the website, `"Shield is an active defense knowledge base MITRE is developing to capture and organize what we are learning about active defense and adversary engagement. Derived from over 10 years of adversary engagement experience, it spans the range from high level, CISO ready considerations of opportunities and objectives, to practitioner friendly discussions of the TTPs available to defenders."`

The U.S. Department of Defense defines **active defense** as `“The employment of limited offensive action and counterattacks to deny a contested area or position to the enemy.”`

Shield Active Defense is similar to the ATT&CK® Matrix, but the tactics and techniques provided to us enable us to trap and/or engage (with) an adversary active within the network. For example, we can plant decoy credentials on a resource and monitor if/when the account's credentials are used elsewhere within the network. By doing this, we are alerted to the adversary's presence and provides the opportunity to learn about their tools and tactics. The information that is gathered can be classified as **threat intelligence**. 

You can Shield Matrix [here](https://shield.mitre.org/matrix/).

![shield1](https://assets.tryhackme.com/additional/mitre/shield1.png)

Each column will list the techniques associated with each tactic (again, as the ATT&CK® Matrix). By clicking on any column headers, we will be redirected to a page providing more information.


 
## AEP (ATT&CK Emulation Plans)

If these tools provided to us by MITRE are not enough, under MITRE ENGENUITY, we have CTID, the Adversary Emulation Library, and ATT&CK® Emulation Plans.

### CITD 

MITRE formed an organization named The [Center of Threat-Informed Defense (CTID)](https://mitre-engenuity.org/ctid/). This organization consists of various companies and vendors from around the globe. Their objective is to conduct research on cyber threats and their TTPs and share this research to improve cyber defense for all.

Some of the companies and vendors who are participants of CTID:

+ AttackIQ
+ Verizon
+ Microsoft
+ Red Canary
+ Splunk

There are 3 ATT&CK® Emulation Plans currently available:

+ [APT3](https://attack.mitre.org/groups/G0022/)
+ [APT29](https://github.com/center-for-threat-informed-defense/adversary_emulation_library/tree/master/apt29) 
+ [FIN6](https://github.com/center-for-threat-informed-defense/adversary_emulation_library/tree/master/fin6)

