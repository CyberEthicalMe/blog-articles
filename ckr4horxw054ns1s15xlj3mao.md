## Infosecurity Europe 2021: Day 2

# Introduction

[Infosecurity Europe 2021](https://online.infosecurityeurope.com/) is an online conference that is organized/hosted by Reed Exhibitions. It is split in two events
* Virtual/Live Sessions: 13-15 July
* On-demand Sessions: 16-29 July

Please, find my notes from the second day below.

> If you have missed the first day, [go here for the notes](https://blog.cyberethical.me/infosecurity-europe-2021-day-1)

***
# Contents
1. [Introduction](#introduction)
* [<em>The War of Attrition in Cyber Space</em> by Ian Hill](#the-war-of-attrition-in-cyber-space-by-ian-hill)
* [<em>Active Directory Security: Why Do We Fail and What Do Admins and Auditors Miss?</em> by Sylvain Cortes](#active-directory-security-why-do-we-fail-and-what-do-admins-and-auditors-miss-by-sylvain-cortes)
* [<em>Smarter Security Operations with a Hybrid SOC</em> by Jaimon Thomas, Sinu Peter](#smarter-security-operations-with-a-hybrid-soc-by-jaimon-thomas-sinu-peter)
* [<em>Cloud Native Application Security: Embracing Developer-First Security for the Cloud Era</em> by Guy Podjarny](#cloud-native-application-security-embracing-developer-first-security-for-the-cloud-era-by-guy-podjarny)
* [<em>Why Ransomware Prevention Demands a Multi-layered, “Prevention First” Approach</em> by Justin Vaughan-Brown, Matt Logan](#why-ransomware-prevention-demands-a-multi-layered-prevention-first-approach-by-justin-vaughan-brown-matt-logan)
* [Additional readings](#additional-readings)
***

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

# _The War of Attrition in Cyber Space_ by Ian Hill

* China has the largest military force (by number)
* focus is switching more and more to cyber capability
* disadvantages of kinetic warfare
 * massing of conventional forces is difficult to conceal and requires weeks of preparation
 * state of the art kinetic weaponry requires high level of skill and training
 * kinetic weaponry can be easily tracked and traced
 * the effect of kinetic attack unfolds over an observable period of time
 * all the above allows for defenders to prepare beforehand
 * it's very expensive
* advantages of cyber warfare
 * occurs at speed of light, giving little time to react
 * initial strike is likely to eliminate any effective defense or response
 * requires only a relatively small force to launch an attack
 * can be executed safely within own borders
 * deniability ("it wasn't us")
 * it is way cheaper
* cyber weapon must be developed in secret and once it is used, it won't be as effective as the first time, or even become unusable
* kinetic weapons can be reused many times (at least technology)
* cyber weapon doesn't act as deterrent
* SolarWinds as an example of attacks on US companies and agencies that resolves as act of espionage rather than act of cyber warfare; intelligence gathered can be weaponized and used in follow-up attacks
* cyberspace is becoming the front line of a new Cold War; recent attacks: JBS, Fujitsu, Colonial Pipeline Cold
* direct attacks against companies involved in fighting against Covid-19; [Irish National Health Service](https://www.bbc.com/news/world-europe-57184977) attacked directly
* in 2017 shopping conglomerate Maersk became the unexpected victim of a [state sponsored NotPetya attack](https://portswigger.net/daily-swig/when-the-screens-went-black-how-notpetya-taught-maersk-to-rely-on-resilience-not-luck-to-mitigate-future-cyber-attacks) aimed predominately at Ukraine's financial institutions
* the primary targets are and will be the critical infrastructure - Public Safety, Food Production, Utilities, Banking, Space, Communications, Transport, Defense and National Security, Government, Health
* we are talking about potential of [Cyber Blitzkrieg](https://www.zdnet.com/article/cyber-blitzkrieg-replaces-cyber-pearl-harbor/) - risk of multi-vector, multi-wave destructive cyberattacks against country's infrastructure
* recent usage of weaponized, mechanized AI - [first-ever swarm drone attacks](https://www.newscientist.com/article/2282656-israel-used-worlds-first-ai-guided-combat-drone-swarm-in-gaza-attacks/)
* United States is reportedly launching a series of secret cyberattacks against Russia in retaliation for its alleged involvement in the widespread SolarWinds attacks; it is also considering retaliation against China for the alleged MS Exchange server attack in March
* security posture
 * cyberattacks will continue to increase - Cyber Criminal, State Sponsored, Hacktivism to name few
 * we should be responsible for each other - educate ourselves, don't keep findings in the closet
 * if you haven't been hit so far - it is only a matter of time
 * we must defend our presence in cyberspace as an extension of the defense of country's land
 * Zero Trust, Automatization & Orchestration, Delegation - key points
* many small companies form part of a supply chain for critical infrastructure or public industries - it doesn't matter if you are a small company; if you're a part of critical supply chain - you are in a radar
* another global conflict is inevitable and cyber warfare will play a pivotal part
* we need to improve our cyber defenses, encourage and support new cyber talent
* there is a demand for national cyber awareness program


> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

>✔ Automatic GitHub Backup

>✔ Write in Markdown

>✔ Free domain mapping

>✔ CDN hosted images

>✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

# _Active Directory Security: Why Do We Fail and What Do Admins and Auditors Miss?_ by Sylvain Cortes

* AD is more than 20 years old - we didn't care of its security and now ransomware takes advantage of it
* AD is used by almost every organization in the world
* malicious actors can safely assume you are using AD - 60% of new malware include specific code that targets Active Directory
* ransomware context
 * objectives to compromise: data privacy, data integrity and data availability
 * popular misconceptions:
  * ransomware is not created by some kids in garage, but is operated by well-organized professional groups
  * "we are small organization, our data is not valuable" - false, the first part of attack is not targeted to anything, it is automated process
  * using AV/EDR is sufficient - false, one of the first steps of ransomware attack is to disable local defenses
* conventional tools are ineffective
 * AV/EDR can be fooled by changing signature (couple of bytes difference throws the detection off)
 * SIEM - too many logs to catch relevant information, also some malicious actions doesn't create any entries
 * Attack Detection is not enough - you should prevent the attack
* AD security must act in real time and prevent attacks
* first fix Windows non-patched CVE + fix AD misconfigurations

# _Smarter Security Operations with a Hybrid SOC_ by Jaimon Thomas, Sinu Peter

* challenges with traditional outsourcing models
 * standardized operating and commercial models
 * SLA-driven rather than KPI-driven
 * absence of incident handling support
 * shift of accountability
* continuously baseline and fill detection and response gaps
* have an agile process to keep up with changing attacker techniques
* be aligned to industry frameworks for more accurate benchmarking
* you need right people, process and technology to do this without relinquishing control to a 3rd part provider


# _Cloud Native Application Security: Embracing Developer-First Security for the Cloud Era_ by Guy Podjarny

* everyone wants to be a [tech company](https://www.businessinsider.com/marianne-lake-says-jpmorgan-is-a-tech-company-2016-2?IR=T)
* DevOps relies on independent teams
* we need a new approach to Application Security - proposing a Dev-First Cloud Native Application Security
* developers like developer tools; to encourage them to use secure solutions in their job, we must adapt software to their workflow
* security audits, developers fix
* security transformation is a must for all

# _Why Ransomware Prevention Demands a Multi-layered, “Prevention First” Approach_ by Justin Vaughan-Brown, Matt Logan


* companies are most afraid of ransomware, quickly followed by zero-day attacks
* why ransomware grown to become headline issues?
 * Ransomware-as-a-Service
 * global pandemic
 * threat vectors expanding
 * double extortion
 * threats being missed in the noise
* threat landscape is evolving
 * on average, 80% of successful breaches are zero-day attacks
 * only 12% of ransomware attacks are prevented
* steps to take
 * isolate the compromise host on the network
 * track and record of endpoint activities
 * review the timeline and activities of events
 * collect and document additional threat indicators
 * if it is possible, quarantine and remove the threat
* we are seeing a growing pattern that malicious party asks for ransom to unlock the files/machines and simultaneously sells the data on the dark web
* deep learning is a subset of machine learning (subset of AI) which makes the computation of multi-layer neural network feasible
* deep learning solves the biggest ML weakness - low noise are causing increase in false positives from **manual** feature engineering
 * ML: accuracy with unknown threats 50-70% with 1-2% of false positives
 * Deep Neural Networks: accuracy with unknown threats 99% with 0.1% of false positives from **raw data**

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

**This is a part of [Infosecurity Europe 2021](/series/infosecurity-europe-2021) series. Follow the links below to read about other days.**
