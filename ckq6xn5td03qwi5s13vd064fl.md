## My first hackathon - Hack4Lem 2021

During last weekend I've participated in the Hack4Lem hackathon organized by [PKO Bank Polski](https://www.pkobp.pl/) and [Instytut Polska Przyszłości im. Stanisława Lema](https://instytutlema.pl/) (Stanislaw Lem's Institute "Polish of the Future"). It lasted 40 hours from June 18th 5PM to June 20th 9AM.

***
# Contents

1. [Deploy working AWS E2 instance](#deploy-working-aws-e2-instance)
2. [Use Python and Bash scripting whenever possible](#use-python-and-bash-scripting-whenever-possible)
3. [Don't put 100% effort into the solution](#dont-put-100-effort-into-the-solution)
4. [SwearJar - NLP Processing Ecosystem](#swearjar-nlp-processing-ecosystem)
5. [Conclusion](#conclusion)
6. [Additional readings](#additional-readings)
***

Because it was my first hackathon ever, I did know that I should focus on more favorable things that winning the prize. So from the two available categories 
1. Bank of the Future. 
2. Cybersecurity - Fighting Cyberbullying

I have chosen the latter one and set myself three goals.

[Back to top](#contents) ⤴

# Deploy working AWS E2 instance

Few weeks ago I have created an Amazon Web Services account having the idea of [private VPS setup](https://community.spiceworks.com/topic/2320459-what-os-would-you-recommend-for-pentester-s-vps) tests, but since then, I didn't do anything on cloud. This was the opportunity for me to push myself a bit and get that experience with AWS Dashboard, SSH access, network hardening and file transfer over rsync.

```
sudo rsync -e "ssh -i privatekey.pem" -rv {local_folder}/ ubuntu@{IP}:{remote_folder}/
```

[Back to top](#contents) ⤴

# Use Python and Bash scripting whenever possible

I though that choosing interaction with social media portals would pair nicely with scripting language. Python is my go-to language when I work with hacking challenges and I want to keep better at it. During this hackathon, I wanted to see how I can create Python server and in overall what are the best practiced when writing a standalone application. So far, my experience with Python was some simple request/response actions in a single file script.

[Back to top](#contents) ⤴

# Don't put 100% effort into the solution

I knew I won't join the finalists, neither by delivering the complete solution nor by skipping the night. My idea doesn't feel innovative enough, and I didn't see the point of putting accessibility features by force into it - so it won't fit the constraints of the competition we are challenged with.

What I wanted to have is a solution that will meet **my** personal goals

[Back to top](#contents) ⤴

# SwearJar - NLP Processing Ecosystem

SwearJar has a main processing unit that is hosted on the cloud (or other publicly available place) and is exposing its classification powers over the API. Other modules, called *Monnies* would send the text to it and receive the prediction value - is this an offensive phrase or not.

Now, depending on the Monnie, method of acquisition and action taken are different.

| Monnie | Method of acquisition  | Actions taken |
| -- | -- | -- |
|WordPress **Doubloon**| Comment, before it is submitted | Prevent posting, warn and educate commenter, send to moderation |
|JavaScript **Dime**| Any defined text field | Prevent submitting, warn and educate user |
|Twitter **Penny**| Search phrase, profile or any other Twitter query, ex. geolocation | Send a private message or tweet responding to the aggressive tweet |
|YouTube **Nickel**| Comments under the videos| Send a private message or respond to the aggresive tweet |

This was the main idea - but I know I wouldn't be able to prepare all of it in just 40 hours. So, I focused on the core Python module deployed on AWS (SweatJar) and Twitter Penny Monnie.

Here you can see GitHub repository:

👉 [SwearJar source code](https://github.com/KamilPacanek/swearjar-nlp/)

Please don't hesitate to fork and play with it, I don't mind if this idea pays forward.

[Back to top](#contents) ⤴

# Conclusion

Did I win something? No.

> Actually yes, but only from the Discord giveaways: Lem's book "Niezwyciężony" (["The Invincible"](https://en.wikipedia.org/wiki/The_Invincible)) and delivery food service coupons worth of 4 dinners for two.

Am I disappointed? No way! I have completed my goals, I have participated (and survived) my first hackathon, met some insightful people. So, all in all - I'm happy I've subscribed to this (with a little push from my wife).

Are you interested in what it looked from my perspective? See the highlights below my profile:

👉 [Hack4Lem Highlights](https://www.instagram.com/s/aGlnaGxpZ2h0OjE3OTMxNjA0MDg3NTgwOTkx)

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)

[Back to top](#contents) ⤴

