## Infosecurity Europe 2021: Day 3

# Introduction

[Infosecurity Europe 2021](https://online.infosecurityeurope.com/) is an online conference that is organized/hosted by Reed Exhibitions. It is split in two events
* Virtual/Live Sessions: 13-15 July
* On-demand Sessions: 16-29 July

Please, find my notes from the last day below.

> If you have missed the [first](https://blog.cyberethical.me/infosecurity-europe-2021-day-1) or [second](https://blog.cyberethical.me/infosecurity-europe-2021-day-2) day check my other articles.

***
# Contents
1. [Introduction](#introduction)
2. [_Aston Martin's Road to Zero Threats_ by Robin Smith](#aston-martins-road-to-zero-threats-by-robin-smith)
3. [_Tackling Ransomware Head On: How Microsoft is Disrupting the Criminal Networks_ by Marja Laitinen, Sarah Armstrong-Smith](#tackling-ransomware-head-on-how-microsoft-is-disrupting-the-criminal-networks-by-marja-laitinen-sarah-armstrong-smith)
4. [_4 Ways Attackers Sidestep Endpoints_ by Scott Walker](#4-ways-attackers-sidestep-endpoints-by-scott-walker)
5. [_Advanced Attack Simulation_ by Pavel Mucha](#advanced-attack-simulation-by-pavel-mucha)
6. [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

# _Aston Martin's Road to Zero Threats_ by Robin Smith

* five pillars (framework) of intelligence approach by Aston Martin
 * identify
 * protect
 * detect
 * respond
 * recover
* if cyber crime would be classified as a country - it would be ranked higher than 183 countries (13th place in global revenue ranking)
* when Cyber Intelligence fails?
 * failing to act on gather information
 * not prioritizing fixes
 * failing to verify and update threat feeds 
* Threat Lead Security Testing
 * Bank of England and CREST launched [CBEST](https://www.crest-approved.org/schemes/cbest/index.html) and [STAR](https://www.crest-approved.org/what-is-star-fs/index.html) testing frameworks in 2014
 * CBEST introduced a codified, detailed and threat-led approach to conducting security testing
 * goals:
  * realistic tests based upon a set of evidence of threats observed in the wild
  * tailored to the organization
  * hold organizations accountable to test findings and highlighting areas to improve resilience
  * broader in scope than a standard pen test and often focused on critical functions where applicable
* [CTIM](https://cyber-threat-intelligence.com/maturity/ctim-whitepaper.pdf) Model
 * based on existing intelligence management process within law enforcement
 * focused on producing actionable intelligence and products for review
 * harmonizes with management planning and business development
 * agile and lean to ensure resource management

[Back to top](#contents) ⤴

# _Tackling Ransomware Head On: How Microsoft is Disrupting the Criminal Networks_ by Marja Laitinen, Sarah Armstrong-Smith

* Microsoft Digital Crimes Unit, strategy and area of focus
 * Business Email Compromise
 * Azure Fraud
 * Malware
 * Ransomware
 * Online Child Exploitation
 * Tech Support Fraud
 * Business Operations Integrity
* ransomware continues to be the most common reason behind Microsoft incident response engagements (2019/10 - 2020/06)
* cybercriminals perform massive wide-ranging sweeps of the internet for vulnerable entry points, then access at the most advantageous time to strike
* in some indicents ransomware entire network took 45 minutes counting from "patient zero"


 | Commodity Ransomware | Human Operated Ransomware |
 | --- | --- |
 |Targets individuals|Targets entire company|
 |Pre-programmed attacks that are best-effort|Customized attacks driven by determined human intelligence|
 |Opportunistic data encryption|Calculated data encryption/data exfiltration|
 |Unlikely to cause catastrophic business disruption|Guaranteed to cause catastrophic and visible business disruption|
 |Successful defense is malware remediation|Successful defense is adversary eviction|
 

* How Human Operated Ransomware operation looks like
 * client attack (email, a browser, etc.)
 * datacenter attacks (SSH, RDP, etc.)
 * _attacker gains access to organization, horizontal movement and spreading starts_
 * looped until elevated access found: credential theft and malware installation
 * _attacker gains administrative permissions access_
 * data exfiltration
 * data encryption
 * backdoor installation
 * _extortion and ransom demands_
* security goal is attackers disruption
 * increase attacker costs at the lowest possible cost
* paying ransom won't prevent attacker from staying inside your network
* priorities of ransomware protection
 * prepare for the worst - recover without paying
 * limit the scope of damage - protect privileged roles
 * make it harder to get in - incrementally remove risks
 * secure backups and test them
 * use [MFA](https://en.wikipedia.org/wiki/Multi-factor_authentication)
* [Human-Operated Ransomware Mitigation Project Plan](https://docs.microsoft.com/en-us/security/compass/human-operated-ransomware#protect-your-organization-against-ransomware-and-extortion)

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

[Back to top](#contents) ⤴

# _4 Ways Attackers Sidestep Endpoints_ by Scott Walker

* example of Fortinet FortiOS [System File Leak](https://us-cert.cisa.gov/ncas/current-activity/2020/11/27/fortinet-fortios-system-file-leak) that leads to revealing plaintext passwords related to the same Fortinet Vulnerable IPs list [CVE-2018-13379](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-13379)
* current dark marketplace is dominated by initial access methods for sale - it means there are people who just find the entry points to some organization but are not committing to exploiting this; and other group of people who don't want to spend time on searching for entry points themselves - so they are willing to buy that information from others
* popular gateway vulnerabilities 
 * [SonicWall](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-20016)
 * [Pulse VPN flaw](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-11510)
* well, it is very hard to keep up with the zero-days and CVEs so..
* **are you ready to be compromised?**
* you only need one person tricked, to have malicious actor inside your network
* how can you make your Microsoft 365 Applications more secure?
 * watch out for guest accounts and unsuspecting administrators
 * Graph API can be used to make changes - be aware of that
* don't give your new employees more access than he need
 * Tesla December 28, 2020 example - employee almost immediately after being employed started to upload files to his personal Dropbox account
* sometimes files are not necessary to gather valuable information - directory names can be sufficient
* one of example of pulling the data out of the network is to set up a web server and get them via HTTP requests
* others can be [DNS Tunneling](https://www.infoblox.com/glossary/dns-tunneling/)
* the biggest AD vulnerability to this day - [Zerologon](https://www.trendmicro.com/en_us/what-is/zerologon.html)
 * 80% of Ryuk infections was introduced using this vulnerability

[Back to top](#contents) ⤴

# _Advanced Attack Simulation_ by Pavel Mucha

That was the single most interesting session of the whole three days. The way Pavel was speaking was very engaging. Unfortunately, the demo he was doing was streamed by the platform in the awful quality. I'm in contact with Cybereason to get this demo in a better quality.

Update: here is an older recording that Pavel provided me - but still amazing.
* [Cyber Attack Simulation by Pavel Mucha](https://www.youtube.com/watch?v=Il72UUDj4rY)

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

**This is a part of [Infosecurity Europe 2021](/series/infosecurity-europe-2021) series. Follow the links below to read about other days.**

[Back to top](#contents) ⤴