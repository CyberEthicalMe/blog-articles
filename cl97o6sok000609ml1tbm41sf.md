# How to add an Active Zone in Cloudflare

> Article contains promotional codes for domain activation and renewal together with [SeoHost.pl referral link](https://seohost.pl/?ref=34505)

On 29th of September 2022, Cloudflare [announced](https://blog.cloudflare.com/making-phishing-defense-seamless-cloudflare-yubico/) a partnership with Yubico offering a great deal for purchasing YubiKeys. It seems like the response of the community was underestimated because they quickly changed the offer, including some requirements before claiming the promotion.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665698903360/y17VH7k-m.png align="left")

One of the easiest and possible most impactful for your operations is establishing a Cloudflare Active Zone.

# What is an Active Zone

%%[join-cta]

Active Zone is an active domain that was added to your Cloudflare account.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665683111833/N39x7lm-d.png align="left")

# How to add a new Zone

1. Navigate to the main dashboard under *Websites*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665683250721/xDFLdcR6I.png align="left")
* Click *Add site*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665683395824/-7HqpCAEJ.png align="left")
* Enter your site **root** domain.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665683461839/SHdxnR3ow.png align="left")
* Choose "Free" option on the bottom of the page.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665684219664/iXzzvjZCT.png align="left")
* Now Cloudflare will scan your existing DNS records, so when you switch your nameservers, previous setting are still in effect.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665684570553/GxUzENkD5.png align="left")
> At this step almost always you want to proxy all reqests through Cloudflare - because that's the whole idea. But if for some reason you want your subdomain resolution to hit directly to the service server, bypassing Cloudflare proxy - this is the place to specify that.
* Click *Continue*.
* You will be welcomed by the overview page where are your Cloudflare nameservers.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665685995798/mhWy2lDez.png align="left")

# Update nameservers

Now, to make your zone active, you have to **replace** your existing nameservers with those provided by the Cloudflare.
> It is important that you remove old ones and leave only Cloudflare's ones, despite the fact it doesn't say so on the overview page (empty *Remove these nameservers*)

The process varies depending on your registrar/domain provider. Detailed process for most popular registrars is available on [Cloudflare Docs](https://developers.cloudflare.com/dns/zone-setups/full-setup/setup/). I will show it on example of [SeoHost.pl](https://seohost.pl/?ref=34505) - which services I am using for last couple of years.

> Use following codes on checkout:  
> 💸 **CYBETHME - pay 25% less on domain activation**  
> 💸 **CYBETHMEAGAIN - pay 15% less on domain renewal**

1. Navigate to *Domain* view.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665699830391/_aBZCVcyJ.png align="left")
* Select the domain you have added to Cloudflare.
* Now in the DNS Setting section, navigate to DNS domain delegations.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665699593094/tHzwZ-uNI.png align="left")
* Remove all (default) nameservers. Add Cloudflare nameservers.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665699696436/3GGbtgO0g.png align="left")
* Click *Save*.

That's it. After verification process ends on Cloudflare, your website (Zone) should change status from "Pending Nameserver Update" to "Active".

%%[support-cta]

# Bonus: Redirect root to subdomain

If you would like to have your root domain to redirect to the subdomain (as it is done with https://cyberethical.me) you have to take [additional steps](https://developers.cloudflare.com/fundamentals/get-started/basic-tasks/manage-subdomains/#redirect-root-domain-to-a-subdomain). It may depend on your original configuration, my registrar was doing that internally by custom rewrite rules.

1. Navigate to your Zone/Website dashboard on Cloudflare.
* Ensure you have CNAME DNS entry for your subdomain.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665700236537/TP83o_JET.png align="left")
> Entry may differ depending on your configuration. Attached image is the example of how subdomain blogs words on Hashnode
* Ensure you have **proxied DNS A record for your root/naked domain**. This can be achieved by entering either domain name or `@` as a record name with arbitrary IP address and with Proxied status.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665700748189/kuNUQ7ZUe.png align="left")
or
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665700795799/JsE2yskmI.png align="left")
> Arbitrary IP, becasue in the next step we will be adding redirection to a subdomain, so traffic never reaches that IP .
* Navigate to *Bulk Redirects* under *Rules*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665701041039/7IS4YV1le.png align="left")
* Click *Create a new Bulk Redirects list*
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665701094140/8x-5_quWY.png align="left")
* Click *Create new list*
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665701162313/76eCRKiqJ.png align="left")
* Select *Redirect* content type, enter name (doesn't really matter).
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665701248825/aO783tPMJ.png align="left")
* Click *Add item*.
* Fill entry as shown below. Click *Save*.
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665701375359/ntrefkW4C.png align="left")

Now, when you enter the naked domain, you should be redirected to the subdomain (with HTTPS).

# Additional readings

%%[follow-cta]

* [Debunking 5 MYTHS About Yubikey by Shannon Morse](https://www.youtube.com/watch?v=vjTA6DeD9y8)
* [Get started with Cloudflare](https://developers.cloudflare.com/learning-paths/get-started/)
