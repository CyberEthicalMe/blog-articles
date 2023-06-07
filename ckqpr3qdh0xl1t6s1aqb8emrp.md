---
title: "Wand of Chaos - Fabular CTF by SecuRing"
seoDescription: "Challenge solves for Różdżka Chaosu / Wand of Chaos fabular CTF organized by SecuRing."
datePublished: Sun Jul 04 2021 22:13:20 GMT+0000 (Coordinated Universal Time)
cuid: ckqpr3qdh0xl1t6s1aqb8emrp
slug: securing-ctf-rozdzka
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1625426786280/gshzBt_ik.png
tags: hacking, cybersecurity-1

---

The target site is running a modified [Vallheru](https://launchpad.net/vallheru) v1.6. This is a fabular MMO browser game that is based on [Gamers-Fusion](https://sourceforge.net/projects/gamers-fusion/) v2.5. Original Vallheru site is running under https://vallheru.eu/. SecuRing modified some of the scripts for this event and cut out functionalities that were not in scope of engagement.

> ⭐ ⭐ You are reading the award-winning write-up. More details [here](https://blog.cyberethical.me/traceback-2021#heading-securing-ctf-rozdzka-chaosu-wand-of-chaos). ⭐ ⭐

Knowing this, I have registered on the original Vallheru. I can immediately see the similarities between these two sites. More rough design of the CTF target site clearly indicates we are dealing with the game based on the same engine, but somewhat older version.

![Original Vallheru.eu game](https://cdn.hashnode.com/res/hashnode/image/upload/v1625399752925/oJZTX2QL7.png align="left")

Challenge details and target site is presented with Polish language, but because of this being a [disclosed event](https://www.facebook.com/SecuRingPL/posts/4303361866387309?comment_id=4304078719648957&reply_comment_id=4304281252962037) I'm preparing this write-up for a wider audience.

# Basic Information

| # |  |
| --- | --- |
| Type | CTF Event |
| Organized by | [SecuRing](https://securing.pl/) |
| Name | **Różdżka Chaosu** |
| Started | 2021/06/28 3PM UTC+2 |
| Ended | 2021/07/02 3PM UTC+2 |
| URLs | https://ctfd.rozdzka.securing.pl/ |
| Author | **Asentinn** / OkabeRintaro |
|  | https://ctftime.org/team/152207 |

%%[patreon-btn] 

> 🔔 `CyberEthical.Me` is maintained purely from your donations - consider one-time sponsoring with the [Sponsor](/sponsor) button or 🎁 [become a Patron](https://www.patreon.com/cyberethicalme) which also gives you some bonus perks.

# Web Challenges

## *Witamy w Dębinie!*

> *Trafiłeś na starą kartkę z księgi magii, która opisuje Różdżkę Chaosu należącą niegdyś do nadmorskiej czarownicy, Urszuli Jagi. Przybywasz do Miasteczka i ... No właśnie, co dalej? Dostań się do* [*gry*](https://vallheru.rozdzka.securing.pl/) *i spróbuj wyruszyć w podróż po jej świecie. Jako flagę podaj nazwę wsi znajdującej się niedaleko Dębiny - można tam dotrzeć na koniu.*

> *You came across an old page from the Book of Magic that describes the Chaos Wand that once belonged to the seaside witch, Urszula Jaga. You come to the town and ... Well, what next? Get to* [*the game*](https://vallheru.rozdzka.securing.pl/) *and try to go on a journey through its world. Enter the name of the village near Dębina as the flag - you can get there on horseback.*

This is a first challenge we have to complete to be able to explore the target. Upon landing on the site linked in the challenge description we are presented with the login form, with already prefilled password field.

![3608415208899.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625400851772/V2eramOm_.png align="left")

By viewing the source code of the page I was able to read both the password (`password`) and potential login name (`admin@localhost`).

![2322200766422.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625400865670/_tHbm9j19.png align="left")

Yes, I was able to login using the found credentials.

![28343128974.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625400891381/ha0V1T-fO.png align="left")

The challenge description says about cities reachable by horse - so lets go to the stables (`Dębina -> Stajnia`)

Here we can see only 1 village (other being forest and mountain)

![stajnie Debina](https://cdn.hashnode.com/res/hashnode/image/upload/v1625400904835/shvKKWCEi.png align="left")

* Flag: `Ulinia`
    

## *Nie jesteś władcą!*

> *Podobno niektórzy mieszkańcy Dębiny posiadają więcej przywilejów niż inni.*

> *Apparently, some residents of Dębina have more privileges than others.*

Well, although we have logged in as an `admin@localhost` - we suppose to find somebody with higher privileges.

Trying the `sqlmap` both on login form and [memberlist.php](https://vallheru.rozdzka.securing.pl/memberlist.php) - no success.

![4576960881265.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625400932095/hw4UJXZE5.png align="left")

But, one thing that is different from the original Vallheru - is that on the bottom of the page there is a link called "Źródło strony" ("Page source"). At the first glance I thought it was gonna launch the HTML source code explorer (`view-source:`) - but it's not!

![5633255122789.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402067453/cuEGu6inE.png align="left")

This launches **real** source code viewer

* `https://vallheru.rozdzka.securing.pl/source.php?file=stats.php`
    

So by passing the file directory in the GET parameter I we can read the file! This is amazing (not for security reasons of course) - because from now on I can do whiteboxing for the challenges.

By playing a bit with it I established that by using it I can view `php` and `tpl` files from `templates`, `includes` and `classes` folders (well, which was easy to do because I can see the `source.php` itself).

![4749105937133.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402083193/00MJ26HVx.png align="left")

Worth noting that both `config.php` and `sessions.php` are not accessible this way.

Now - knowing that SecuRing modified the game a little bit, I'm launching the `gobuster` against it to see if I can see this way some interesting files or paths.

```python
$ gobuster dir -w /usr/wl/dirbuster-m.txt -x txt,php -u https://vallheru.rozdzka.securing.pl  -k -o gbdir-vallheru.rozdzka.securing.pl-https.out
$ cat gbdir-vallheru.rozdzka.securing.pl-https.out

/images               (Status: 301) [Size: 345] [--> http://vallheru.rozdzka.securing.pl/images/]
/index.php            (Status: 200) [Size: 3106]
/news.php             (Status: 200) [Size: 862]
/rss.php              (Status: 200) [Size: 1005]
/templates            (Status: 301) [Size: 348]  [--> http://vallheru.rozdzka.securing.pl/templates/]
/forums.php           (Status: 200) [Size: 862]
/view.php             (Status: 200) [Size: 862]
/stats.php            (Status: 200) [Size: 862]
/library.php          (Status: 200) [Size: 862]
/mail.php             (Status: 200) [Size: 862]
/map.php              (Status: 200) [Size: 862]
/travel.php           (Status: 200) [Size: 862]
/staff.php            (Status: 200) [Size: 862]
/admin.php            (Status: 200) [Size: 862]
/chat.php             (Status: 200) [Size: 862]
/account.php          (Status: 200) [Size: 862]
/portal.php           (Status: 200) [Size: 862]
/memberlist.php       (Status: 200) [Size: 862]
/css                  (Status: 301) [Size: 342] [--> http://vallheru.rozdzka.securing.pl/css/]
/mission.php          (Status: 200) [Size: 862]
/team.php             (Status: 200) [Size: 862]
/updates.php          (Status: 200) [Size: 862]
/includes             (Status: 301) [Size: 347] [--> http://vallheru.rozdzka.securing.pl/includes/]
/log.php              (Status: 200) [Size: 862]
/source.php           (Status: 200) [Size: 0]
/core.php             (Status: 200) [Size: 862]
/avatars              (Status: 301) [Size: 346] [--> http://vallheru.rozdzka.securing.pl/avatars/]
/house.php            (Status: 200) [Size: 862]
/polls.php            (Status: 200) [Size: 862]
/ap.php               (Status: 200) [Size: 862]
/languages            (Status: 301) [Size: 348] [--> http://vallheru.rozdzka.securing.pl/languages/]
/js                   (Status: 301) [Size: 341] [--> http://vallheru.rozdzka.securing.pl/js/]
/cache                (Status: 301) [Size: 344] [--> http://vallheru.rozdzka.securing.pl/cache/]
/logout.php           (Status: 200) [Size: 807]
/ChangeLog            (Status: 200) [Size: 28018]
/market.php           (Status: 200) [Size: 862]
/city.php             (Status: 200) [Size: 862]
/sendmail.php         (Status: 405) [Size: 178]
/class                (Status: 301) [Size: 344] [--> http://vallheru.rozdzka.securing.pl/class/]
/grid.php             (Status: 200) [Size: 862]
/explore.php          (Status: 200) [Size: 862]
/newspaper.php        (Status: 200) [Size: 862]
/mailer               (Status: 301) [Size: 345] [--> http://vallheru.rozdzka.securing.pl/mailer/]
/portals.php          (Status: 200) [Size: 862]
/bank.php             (Status: 200) [Size: 862]
/addnews.php          (Status: 200) [Size: 862]
/hof.php              (Status: 200) [Size: 862]
```

And yeah, there are some interesting files - like `account.php` that was commented out on the page (and was stripped down by the SecuRing) - but most interesting one is a `admin.php`.

By viewing the [admin.php](https://vallheru.rozdzka.securing.pl/source.php?file=admin.php) source I can see that there is a rank validation check. If I'm not an `Admin` (and I'm not) I should see the `NOT_ADMIN` message.

![4321319807319.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402204828/XzL9VXvmA.png align="left")

So by navigating to the `admin.php` I can read the flag:

![5772352901459.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402221894/ExNhDJ2fp.png align="left")

* Flag: `WOC{NOT_ADMIN}`
    

## *Zdrój wiedzy*

> *Wierzysz, że odpowiednie przygotowanie i obszerny research jest kluczem do sukcesu. Pora więc udać się do źródeł!*

> *You believe that proper preparation and extensive research is the key to success. So it's time to go to the source!*

We are advised to go back to the sources - which of course means looking for the flag in the source code. Also, where "extensive research" with "sources" can be done? In a Library, of course! So, I'm navigating to the local Library (`Dębina -> [Dzielnica mieszkalna]Biblioteka`)

![1831722238972.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402237609/v8Lv-8gsR.png align="left")

Now by viewing the source of the page:

![3805004796495.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625402249263/0tG74-Fwk.png align="left")

* FLAG: `WOC{JRRTOLKIEN}`
    

## *Promocja*

> *Niedawno z okazji Dni Dębiny miejscowy płatnerz oferował bardzo korzystną promocję. Czy uda Ci się z niej jeszcze skorzystać?*

> *Recently, on the occasion of the Days of Dębina, a local armorer offered a very favorable promotion. Will you be able to use it yet?*

I'm navigating to the armorer (`Dębina -> [Wojenne Pola]Płatnerz`)

![1619416771533.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405119101/lM0tCwoxp.png align="left")

In the `armor.php` is the actual code handling the promotion code for the Days of Dębina. First the definition of the voucher:

![218531667209.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405130297/PUgKgQg1B.png align="left")

As it happens - the promotion works only for the item of `ID=1`. So I'm selecting `Zbroje` (`Armors`)

![3259212393073.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405149967/-iqNr1VUH.png align="left")

And the actual promotion handling:

![1871352040766.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405163671/We0owvQ9j.png align="left")

So, this time it is a matter of navigating through the if/else conditions.

So for this to work we:

* have to set query parameter `buy=1` (because promotion works for item of this ID)
    
* have to use **empty** `promo` query parameter to skip day selection (`strlen($_GET['promo']) > 0`) and `NO_MONEY` path (`!isset($_GET['promo'])`)
    

I'm going to exploit the `$_REQUEST` usage vulnerability here.

> `The $_REQUEST variable contains all the parameters passed in the request via GET ($_GET), POST ($_POST) and cookies ($_COOKIE).`

So, I'm setting the `promo` cookie - this way I can pass `[armId]` check (`$_GET['buy'] == $promos[$_REQUEST['promo']]['armId']`) finally setting the `$validPromo` to `$true`.

> *Remeber that cookies have the expiration date. If you are trying to exploit it once again after some time passes - make sure to refresh the* `promo` cookie

![2377383887257.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405189391/WllZ6l6NV.png align="left")

Now by navigating to the following page (mind query parameters)

* `https://vallheru.rozdzka.securing.pl/armor.php?promo=&buy=1`
    

I can buy the item and get the flag.

![5085072728959.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405203289/DyZj1pl-V.png align="left")

* FLAG: `WOC{moje_miasto_Debina}`
    

More about `$_REQUEST` vulnerability:

* [PHP Manual: $\_REQUEST](https://www.php.net/manual/en/reserved.variables.request.php)
    
* [PHP: Exploitation with $\_REQUEST while validating $\_GET](https://no-sec.net/php_exploitation_with_request_while_validation_of_get/)
    
* [How evil is $\_REQUEST and what are some acceptable Band-Aid countermeasures?](https://stackoverflow.com/questions/258625/how-evil-is-request-and-what-are-some-acceptable-band-aid-countermeasures)
    

## *Subskrybcja*

> *Ciekawe, co słychać w lokalnej gazecie?*

> *I wonder what's going on in the local newspaper?*

I'm navigating to the Local Newspaper (`Dębina -> [Społeczność]Redakcja gazety`) and try to read latest "Valweek".

![4225582677393.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405221823/p1CjvUD20.png align="left")

Alright, let's see the `newspaper.php`.

![3481887528452.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405251011/VE_wmhiIw.png align="left")

Basically, we have to crack the MD5 hash of `0e938153791264385292992237641232` - and set the original value in the `subscription` cookie.

No success on the [CrackStation](https://crackstation.net/):

![1856404782724.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405266316/Yd_rv5Qdn.png align="left")

Then... There is something odd in this hash... Then it hit me - **Magic Hashes**. I've heard about it during the HTB Battlegrounds that was streamed last moth!

> Exact moment IppSec and John Hammond explains that: https://www.youtube.com/watch?v=622IDqkJyGU&t=2205s

To quickly sum up - if MD5 hash meets following criteria

* comparison is **not** a strict type comparison (`==` instead `===`) and
    
* starts with `0e` and
    
* everything that follow the starting `0e` is a number,
    

then we can pass just any number that hash also meets the above criteria - and during the comparison `0e{digits}` going to be compared as a scientific notation number - then both are going to evaluate to `0`.

For example, `240610708` MD5 hash is `0e462097431906509019562988736854`

![788822717394.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405337502/7Mh_j3f9S.png align="left")

![4092071937814.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405350278/8kRj4AY1n.png align="left")

* Flag: `WOC{I<3free_press}`
    

Read more:

* [Magic Hashes](https://www.whitehatsec.com/blog/magic-hashes/)
    

## *Kukła*

> *Dobrze byłoby z kimś potrenować. Kto wie, jakie potworności czekają na Ciebie w lesie... Udaj się do Areny Walk i porozmawiaj z Wojownikiem.*

> *It would be good to train with someone. Who knows what monstrosities are waiting for you in the forest ... Go to the Arena Fight and talk to the Warrior.*

I'm browsing to the Arena (`Dębina -> [Wojenne Pola]Arena Walk`)

When I'm trying to fight the other player, Warrior mocks me and talks about some dummy.

![1421204515589.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405374509/jCTp7vHiK.png align="left")

Let's see the `battle.php`.

![Screenshot_1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405413454/DkpNYxMPF.png align="left")

This one looks a bit more complicated. We have to predict `rand()` value. Knowing that `rand` generates **pseudo**random numbers, it gives at least some hope.

At the first glance, this MD5 hash is calculated using the current time and semi-random string. At closer look - that UTC timestamp `date()` is returning is clipped only to the first 5 digits. Which.. not gonna change for a while.

> *PHP* `date()` returns current integer Unix timestamp - seconds since the Epoch (`00:00:00 UTC on 1 January 1970`)

**Then**, that random string and 5-digit number is used in a plus operator, as a left and right operands respectively. Keep in mind that we are using `PHP 7.4.20`. Let's do a quick online PoC (I've chosen the closest PHP version):

![2639055841340.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405429381/L82ZNlpkE0.png align="left")

Here is the snippet: http://sandbox.onlinephpfunctions.com/code/b0a420acf118ae09f43165b29b3a683fd7de7f99

So, it proves what I was thinking - random string doesn't matter, and only the first 5 digits are used to calculate the MD5 hash.

Then the first 8 digits are used to seed the random number generator using `srand()`, so the next `rand()` call will return the same value for the same seed used.

Taking all into consideration, above "dummy" code can be simplified to find the `fight` GET parameter ([online](http://sandbox.onlinephpfunctions.com/code/97a7e8f795b9beaea1703a65e269a6c42e5eb408)).

```php
$time = time();
$timeSubstr = substr($time, 0, 5);
$md5 = md5($timeSubstr);
$subsMd5 = substr($md5, 0, 8);
$hexdec = hexdec($subsMd5);
srand($hexdec);

echo rand();
```

For the time of writing this it was:

![5738216032864.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405477672/QH_i2NMVt.png align="left")

![Screenshot_11.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405487302/wlaUJKkI6.png align="left")

* FLAG: `WOC{Fortuna-sie-do-Ciebie-usmiechnela}`
    

## *Jubiler*

> *Pierścienie na wystawie wyglądają pięknie, ale nie są tanie...*

> *The rings on display look beautiful, but they are not cheap ...*

I go to the Jewelry shop (`Dębina -> [Zachodnia Strona]Jubiler`) and look on the `jewellershop.php`:

![4214279419449.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405516842/EFD-WNR7D.png align="left")

My first tries with bank to get more money failed - after some thought, probably because it will modify the data on the server, potentially ruining the experience for other CTF players.

So, scrolling through the file, it looks like I could try crafting.

![2057584657671.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405535225/mLLl0_PF-.png align="left")

Especially because it is using `extract()` function on `$_GET` variable. With this, we can modify effortlessly any variable that is accessible at the moment of `extract()` execution and possibly create new variables.

POC ([online](http://sandbox.onlinephpfunctions.com/code/e8fbccee22918963be688ca478ff982294ff3eaf)):

```php
$ringPrice=500;
$_fakeGET = ["ringPrice" => 0];

echo $ringPrice . "\n";
extract($_fakeGET);
echo $ringPrice . "\n";
```

![3432717486386.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405581539/YH0dKhEh8.png align="left")

Now, because there is an unknown `craft()` function called, and we should at least provide these `material` and `inscription` arguments. Full proof of concept:

```php
#craft.php

# Example of double $ sign
# $a = ["c" => "d"];
# $b = "a";
# echo $$b["c"];
# Output: d

# Actual Payload:
# ?ringPrice=0&temp[material]=ring&temp[inscription]=r&craft=temp&buy=1

$ringPrice = 500;

if(isset($_GET['craft'])) {
    $item = $_GET['craft'];
    echo $item . "<br/>";
    echo $ringPrice . "<br/>";
    print_r($_GET);
    extract($_GET);
    echo $ringPrice . "<br/>";
    echo $$item["material"] . "<br/>";
    echo $$item["inscription"] . "<br/>";
    $material = $$item["material"];
    $inscription = $$item["inscription"];
    echo $material . "<br/>";
    echo $inscription . "<br/>";
}
```

Because the script is using double $ notation, I have to provide in the `craft` parameter name of the other parameter (array) from which it will extract irrelevant data. Also, I want to buy the ring, so `buy=1` is appended.

![Screenshot_13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405641656/pSWKkFZcX.png align="left")

* FLAG: `WOC{nie_wszystko_zloto}`
    

Read more:

* [PHP Manual: extract()](https://www.php.net/manual/en/function.extract.php)
    

## *Prohibicja* (Unsolved)

> W Dębinie od wielu lat obowiązuje prohibicja. Krążą jednak plotki, że niektóre lokale nie stosują się do obowiązujących zakazów - a co gorsza, że pomagają im w tym radni. Gdyby tylko udało się dostać do ich prywatnych wiadomości... Aby zgłosić sprawę do sołtysa, potrzebne są następujące informacje: nazwa lokalu, pseudonim właściciela Podaj je jako flagę.

> Prohibition has been in force in Dębina for many years. However, there are rumors that some establishments do not comply with the applicable bans - and what is worse, that they are helped by councilors. If only their private messages could be found ... In order to report the matter to the village administrator, the following information is needed: the name of the premises, the nickname of the owner Enter it as a flag.

Location: `Skrzynka` (`Inbox`)?

![5074426518909.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405672760/Z8btJe0Nn.png align="left")

![3412044585587.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405681260/gbEEZZlX0.png align="left")

I can see there is finally a method to iterate over users here (using the `staff` POST parameter and `mid` as a member ID).

Some tests:

```python
data = {"staff":"1", "mid":"1"}
r = requests.post(f"{url}?send&step=send", cookies=cookies, headers=headers, data=data)

print(r.text)

# Output:
# <div class="error">Ten gracz nie jest ani władcą ani księciem!</div>
# <div class="error">NOT_MAIL</div>
```

And for non-existing player:

```python
data = {"staff":"5", "mid":"1"}
r = requests.post(f"{url}?send&step=send", cookies=cookies, headers=headers, data=data)

print(r.text)
```

![5881528921338.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405700912/vnJRANLak.png align="left")

With that, I can easily create a python script to find existing users and find out who is the higher privilege user. And maybe then read messages of other players.

```python
# iterate_users.py

import requests
import urllib3
#import time

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

host = "vallheru.rozdzka.securing.pl"
url = f"https://{host}/mail.php"

headers = {"Content-Type":"application/x-www-form-urlencoded"}
cookies = {"PHPSESSID":"f38c6ec82fe8f41605cd8baa8e309071"}
data = {}

for i in range(0,100000):
    #time.sleep(0.5)
    print(f"{i}      ", end="\r")
    data = {"staff":i, "mid":1}
    r = requests.post(f"{url}?send&step=send", cookies=cookies, headers=headers, data=data)

    if "Nie ma takiego gracza!" not in r.text:
        print(f"User found!: {i}")
    if "Ten gracz nie jest ani władcą ani księciem!" not in r.text:
        print(f"Admin or staff found!: {i}")

print("end")
```

I was running this script for hours, but to no avail... (only first 4 players got recognized).

# Cryptography Challenges

## *ROTmistrz*

> \_ Jeden z mieszczan wspomniał, że warto porozmawiać z rotmistrzem. Zazwyczaj można go spotkać w Auli Gladiatorów. Kto jest autorem sentencji, którą przytacza rotmistrz?\_

> *One of the townspeople mentioned that it is worth talking to the captain. He can usually be found in the Hall of Gladiators. Who is the author of the sentence quoted by the captain?*

This is an easy one, especially because hint is in the name of the challenge. Location: `Dębina -> [Wojenne Pola]Aula Gladiatorów`.

![4045794532999.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405740862/yvktdX6AM.png align="left")

I'm pasting the encoded text to the CyberChef, choosing the ROT13 encoding and changing the rotation number until I get understandable Polish text:

![5636506658835.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405748184/Y6nwsQLLLH.png align="left")

And author of the "Nie wystarczy dużo wiedzieć, żeby być mądrym" is Heraklit.

![4131944761777.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405787215/f2TbsBN6a.png align="left")

* Flag: `Heraklit z Efezu`
    

## *Rozmowa*

> *Najciekawsze rozmowy usłyszysz w miejscu wyszynku. Czasami kręcą się tam podejrzane typy...*

> *You will hear the most interesting conversations at the selling place. Sometimes dodgy guys are hanging around there ...*

Location: `Karczma` (left sidebar)

![4287963807207.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405804380/eUFDlkaQY.png align="left")

Well, I must admit this would be too hard for non-Polish speakers. By looking at the conversation, `Grajek` person obviously stands out. The structure of the words, context and casing reminds me of one scene from one particular TV series... Is it really it? Let's try character substitution for the phrase I think is far too obvious. I've copied the whole conversation, leaving each phrase in a separate array entry.

```python
# translate.py
#Cvhwrk ekg Msferusbhms
#Grosza daj Wiedzminowi

chat = [
"Jf pkookef mku, wnsfmk wqvhubl pkve, ih r Cfvmkrlu r Vsdss mlvawrlo bk wrokq. ",
...
"Cvhwrk ekg Msferusbhms, wkqsfmqk nhjvrkwbsg, wkqsfmqk nhjvrkwbsg, oh, h, h! Cvhwrk ekg Msferusbhms, hpvhbil oaerqhwis!"
]

for txt in chat:
    txt = txt.lower()
    transTable = txt.maketrans("cvhwrkegmsfeub", "groszadjwiedmn", "")
    txt = txt.translate(transTable)
    print(txt)
```

![675118869669.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405860288/PiySjs6zl.png align="left")

Yeah! This is it. And because I know the words for the [song](https://www.youtube.com/watch?v=d9XiYH47Jbs) I can fill the mapping.

* `"cvhwrkegmsfeubqnjpoailyx" -> "groszadjwiedmnkptblucyhf"`
    

And after whole translation:

![2825552556311.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405871092/bAB-EjaYvL.png align="left")

* Flag: `WOC{Zmeczone kruki lataja dzis nisko}`
    

Listen more:

* I also like this version...: https://www.youtube.com/watch?v=PcQfjH6PFUE
    

## *Pęk kluczy*

> *Udało Ci się odnaleźć pęk kluczy. Ciekawe, czy którymś z nich można otworzyć lochy?*

> *You have found a keychain. I wonder if one of them can open the dungeons?*

Location: `Dębina -> [Zamek]Lochy`

![5343978366541.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405920273/eyW2XrL6Y.png align="left")

We are given the `pek_kluczy.zip` file.

```python
$ file pek_kluczy.zip

pek_kluczy.zip: Zip archive data, at least v2.0 to extract
```

The text on the page says we are looking for the "big, old, rusted" key, with "no decorations".

I'm extracting the file and, first things first, running `binwalk -M` on each of the `*.png` files to see any hidden data. But there is none. Then I'm browsing the files and... number 004 looks like one that fits to the description.

![5168987158422.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405950205/sebJCej40.png align="left")

I'm importing the GPG key:

![5768246124967.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405988554/NxTh_alfF.png align="left")

Then I parse the input text from the page, that I've previously copied to the `pek-kluczy-encrypted.txt`:

```python
cat pek-kluczy-encrypted.txt | base64 -d > loch.enc
```

And finally, I'm decrypting it with the previously imported private key.

![1437032787597.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625405996451/FHQ_yWZx0K.png align="left")

* FLAG: `WOC{10643b61f084e340f4c4982e21c981f5}` (I like how this flag looks like)
    

# Binary Challenges

## *Szafa grająca* (Unsolved)

> *Kto wie, co kryje w sobie szafa grająca... Podaj napis wygrawerowany na monecie.*

> *Who knows what's inside a jukebox ... Enter the name engraved on the coin.*

We are given binary file `chal`.

```text
chal: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, 
interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=3d0d4575d315a306f878ace5004d145c15f859ec, for GNU/Linux 3.2.0, not stripped
```

Running `strings`:

![4766390669743.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625406017407/o5MCCGLHan.png align="left")

Decompiling in the `ghidra`:

![2028258366385.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625406025705/84S79XK6R.png align="left")

Ok, at this point I know that `scanf`can be exploited to write over the memory and what I suppose to do here is to somehow move to `admin` method where there is a `/bin/cat` and `/flag` variables .

![3155903847208.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625406033870/riA0OIBS-.png align="left")

But I don't know how.

# Steganography Challenges

## *Zabytkowy zegar* (Unsolved)

> *W przewodniku wielokrotnie wspominany jest zabytkowy zegar miejski, który znajduje się na ścianie zamku. Wypadałoby go zobaczyć...*

> *The guide mentions the historic city clock on the castle wall many times. It would be good to see him...*

Location: `Dębina -> [Zamek]Zegar miejski`

![389363524409.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625406058532/IpTrLhwjC_.png align="left")

I'm downloading the image. Hmm, nothing with the `binwalk`

![1505870444504.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1625419480668/dn_tTDJ0E.png align="left")

Then I'm throwing everything I can from this [cheatsheet](https://pequalsnp-team.github.io/cheatsheet/steganography-101), but to no avail.

> Like what you see? Join the [Hashnode.com](/join) now. Things that are awesome:

> ✔ Automatic GitHub Backup

> ✔ Write in Markdown

> ✔ Free domain mapping

> ✔ CDN hosted images

> ✔ Free in-built newsletter service

> By using my link you can help me unlock the ambasador role, which cost you nothing and gives me some additional features to support my content creation mojo.

# Additional readings

> 📌 Follow the `#CyberEthical` hashtag on the social media

> 👉 Instagram: [@cyber.ethical.me](https://www.instagram.com/cyber.ethical.me/)

> 👉 LinkedIn: [Kamil Gierach-Pacanek](https://www.linkedin.com/in/kamilpacanek)

> 👉 Twitter: [@cyberethical\_me](https://twitter.com/cyberethical_me)

> 👉 Facebook: [@CyberEthicalMe](https://facebook.com/CyberEthicalMe)