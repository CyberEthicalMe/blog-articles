## Infosecurity Europe 2021: Day 1

# Introduction

[Infosecurity Europe 2021](https://online.infosecurityeurope.com/) is an online conference that is organized/hosted by Reed Exhibitions. It is split in two events
* Virtual/Live Sessions: 13-15 July
* On-demand Sessions: 16-29 July

Today was a first day of the conference. Each session lasts 30-40 minutes, and they were really cybersecurity focused. My favorite was the one by [Javvad Malik](https://twitter.com/J4vv4D) about ransomware threat - really straight-to-the-point approach, just like I like it.

The only disadvantages we, as participants, have to overcome was muting/no sound issue that was happening on the beginning of each session, that raised each time over 50% communication on chat. Yes, I can be picky, but sometimes quality of the stream was not going well with presenters slides - because slides were prepared for bigger screen - **but** it can be mitigated by listening to the specialists.

Please, find my notes below.

***
# Contents
1. [Introduction](#introduction)
* [Discussing the Rising Nation-State Cyber Activity and its Impacts by Robert Hannigan](#discussing-the-rising-nation-state-cyber-activity-and-its-impacts-by-robert-hannigan)
* [Solving The Cyber Skills Gap by Dr. Andrea Cullen, Lorna Armitage](#solving-the-cyber-skills-gap-by-dr-andrea-cullen-lorna-armitage)
* [Digital Empathy: Building a People First Approach to Cybersecurity by Sian John](#digital-empathy-building-a-people-first-approach-to-cybersecurity-by-sian-john)
* [Top Use Cases For Automated Penetration Testing by Eric Graves, Shakel Ahmed](#top-use-cases-for-automated-penetration-testing-by-eric-graves-shakel-ahmed)
* [Minimizing End-User Disruption During Incident Response on MacOS by Mark Walker](#minimizing-end-user-disruption-during-incident-response-on-macos-by-mark-walker)
* [A How to Guide for Cybersecurity Pros: Navigating the Top Trends in Third-Party Risk Management) by Chris Paterson](#a-how-to-guide-for-cybersecurity-pros-navigating-the-top-trends-in-third-party-risk-management-by-chris-paterson)
* [Now That Ransomware Has Gone Nuclear, How Can You Avoid Becoming the Next Victim? by Javvad Malik](#now-that-ransomware-has-gone-nuclear-how-can-you-avoid-becoming-the-next-victim-by-javvad-malik)
* [5 Steps to Zero Trust by Tom Rossdale](#5-steps-to-zero-trust-by-tom-rossdale)
* [Disrupting the Cybersecurity Kill Chain by Detecting Domain Reconnaissance by Harish Sekar](#disrupting-the-cybersecurity-kill-chain-by-detecting-domain-reconnaissance-by-harish-sekar)
* [The Life and Times of Open Source Libraries by Fulya Sengil](#the-life-and-times-of-open-source-libraries-by-fulya-sengil)
* [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

# _Discussing the Rising Nation-State Cyber Activity and its Impacts_ by Robert Hannigan

* first occurrence of cyberattack homicide - woman in Düsseldorf wasn't admitted to the hospital because ransomware attack on a hospital ([source](https://www.wired.co.uk/article/ransomware-hospital-death-germany))
* most ransomware attacks are not reported - something to have in mind when reading statistics
* we are observing a rise of ransomware-as-a-service; today you don't need to have to be a hacker to create and infect devices with malicious software
* companies should worry about their vendors/suppliers security posture -attacks can be sourced from connected networks
* it is good to look in the future, but let's go to basics - harden these attack vectors  we know; "get right present and past first"
* don't be afraid to tell vendors what they can improve - they are not your enemies, and they want to deliver more secure solutions
* raise security funding across supply chain
* we are in the turning point in security private sector - we are becoming more aware;
let's move from assessing to taking actions

# _Solving The Cyber Skills Gap_ by Dr. Andrea Cullen, Lorna Armitage

* causes of skills shortage:
 * lack of collaboration, understanding and guidance
 * too many skills and certifications requested on job adverts
 * lack of role models
* CAPSLOCK presentation about graduates of their program - dog handler, store manager, retail assistant and music composers got a cybersecurity positions
* invitation to hire their learners or join next cohort - ([more details](https://capslock.ac))

# _Digital Empathy: Building a People First Approach to Cybersecurity_ by Sian John

* all businesses are now digital, partially because of epidemic - and people are not prepared to bear this responsibility
* criminals also moved to digital space, we can think about them like criminal enterprises; now they're exploiting people
* having a business should include cyber threat as a cost, because of ransomware endemic
* positive side of this is that physical acts of crime is diminishing - ex. lower rate of bank robberies
* we must think about digital transformation of modern enterprise - all slower or faster are continuously evolving by adapting to new technologies - SaaS, Cloud, IoT; 

> Something that is called Hype Cycle is shown, by here presented by different name - if you don't know what Hype Cycle is, see the Gartner.com which publishes Hype Cycle of Emerging Technologies each year ([and other](https://www.gartner.com/smarterwithgartner/6-trends-on-the-gartner-hype-cycle-for-the-digital-workplace-2020/)); it is basicaly a curve that shows how new technology moves through 5 stages - Innovation Trigger, Peak of Inflated Expectations, Trough of Disillusionment, Slope of Enlightenment, Plateau of Productivity (if it reaches it)

* we are facing a problem - how do we enable productivity without compromising security? Danger of [Shadow IT](https://www.forcepoint.com/cyber-edu/shadow-it) appearing if countermeasures = counterproductivity

> Anegdote: There was a time I was working for a company that have such strong policy, that we have to ask them to install the software via remote connection. It comes to that, not so funny back then, situation that I have to wait couple hours for IT worker to connect to my machine and install some Visual Studio feature that I had to have to contnue working on project. In result we discovered that we have permission to run Visual Studio with elevated privileges (because that particular project type requires it) so we run the PowerShell from inside the VS to install software or configure system by ourselves.
This is a perfect example how countermeasures was so troublesome and counterproductive that developers are almost forced to come up with some Shadow IT solution to be able to perform their work.

* think about how users work and behave - understand what they do to make their job done
* don't assume that what we do (in regard to hardening our systems) will be appreciated and followed by everybody; for this to work we have to think like users and incorporate security to their workflow, instead of disrupting it
* use technology to fit security into productivity
 * zero trust
 * passwordless authentication
 * automated detection and response systems

# _Top Use Cases For Automated Penetration Testing_ by Eric Graves, Shakel Ahmed

* basically a [PenTera](https://www.pentera.io/) capabilities presentation
* three use cases of PenTera was described
 * _Automize Process - Continuous Testing_ for National Health Service
 * _Saving Time for the Read Team_ - head of the Read Team said that PenTera saved his team 27 mandays in a month, by running attack engines at machine speed and gaining instant reports
 
 > which is impressive
 
 * _Validation of Existing Cybersecurity_ - validation of existing cybersecurity stack and decision on which technologies to move forward with, when standardizing
 
# _Minimizing End-User Disruption During Incident Response on MacOS_ by Mark Walker

* impact on users
 * stress
 * reputation
 * privacy
 * productivity
* metrics that matter
 * endpoint availability
 * productivity impact
 * user [RTO or RPO](https://www.druva.com/blog/understanding-rpo-and-rto/)
* we should not be perceived as a threat or enemy - currently users are afraid of reporting incidents
* focus on end users

# _A How to Guide for Cybersecurity Pros: Navigating the Top Trends in Third-Party Risk Management)_ by Chris Paterson

* challenges of 3rd Party Risk Management
 * Reliance on others
 * Data sprawl
 * Endless assessments
 * Increasing scrutiny
 * Unknown weak links
 * Burden vendors
 * Constant changes
* vendors don't always have the time of resources

# _Now That Ransomware Has Gone Nuclear, How Can You Avoid Becoming the Next Victim?_ by Javvad Malik

* starts with asymmetric encryption with 2013 Cryptolocker appearance, after that, we are still at snowball effect
* cryptocurrency channels makes easier to collect ransom
* Ransomware Business
 * at first used to encrypt immediately, doesn't care about content
 * then spread like a work and encrypts,
 * now stays as an Advanced Persistent Threat to collect data undetected
 * encrypting VMs and backups
 * develops new business branch - Ransomware-as-a-service
* today, victims almost always pay (even if they say they don't)
* having big security budged doesn't make you immune
* possible scenarios after stealing data from company
 * leak all the data (Allied Universal, City of Pensacola, Boeing, SpaceX)
 * threaten to reveal dirt (Lady Gaga, Madonna's attorneys)
 * data auction (relatively easily to find at the Dark Web )
 * credential stealing (to reuse after some time)
 * threaten victim's employees
 * use stolen data to spear phish partners and customers
 * public shaming
* protecting, security perimeter, post-factum
 * complete and restore tested backups of critical systems
 * elevated privileges hygiene
 * after a breach, change all passwords, not only internal ones
 * encrypt data
 * cybersecurity insurance
 * media incident response team
 * IDS, HIDS, NIDS,
 * ability to detect massive number of suddenly changing files
 * Data Leak Prevention tools
 * network traffic analysis
 * honeypots
* establish what you will say to employees after ransomware compromise
* are you never paying the ransom? if so, does the management know what are the consequences?

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

# _5 Steps to Zero Trust_ by Tom Rossdale

* it is not a question if an attacked will get in, but when
* Forrester's 5 steps to Zero Trust
 * identify your sensitive data
 * map the flows of your sensitive data
 * architect your Zero Trust microperimiters
 * continuously monitor your Zero Trust ecosystem security analytics
 * embrace security automation and orchestration

# _Disrupting the Cybersecurity Kill Chain by Detecting Domain Reconnaissance_ by Harish Sekar

* Surveying the domain for critical information can help attackers target their attacks.
* cyberattacks are a growing threat
* Active Directory (AD) is a constant target to compromise because it is widely used
* AD handles authentication and authorization for all users in an organization

> Harish Sekar is the first person to have a live hacking/coding fragment - different attacks on LDAP

* domain recon is the process of investigating and identifying the critical or vulnerable parts in an organization's network; it can help an attacker plan their next moves and target their attacks
* when attacker gets password policy - he will be having better time cracking the password hashes.
* Directory services need more protection
* anybody could be an attacker

> even coworker/insider

* whitelisting applications usage to be monitored
* disabling Shell won't be enough, need deeper understanding

# _The Life and Times of Open Source Libraries_ by Fulya Sengil

* graphs were too small to read in stream resolution - but presenter was talking about statistics taken from Veracode analysis

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

**This is a part of [Infosecurity Europe 2021](/infosecurity-europe-2021) series. Follow the links below to read about other days.**