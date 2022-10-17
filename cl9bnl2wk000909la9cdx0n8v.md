# Mega Sekurak Hacking Party CTF October 2022

%%[support-cta]

# Background

This article shows my approach on selection of the challenges I was able to solve during the competition. For the complete set I recommend reading the write-ups of the winner of this CTF **[🥇gynvael](https://gynvael.coldwind.pl/?id=756)** (who by the way completely dominated the scoreboard).

During this 2 day (or more like 36hrs) I was capable of solving 4 out of 10 challenges (2 easy, 1 medium and 1 hard) which gave me 12th place among ~100 participants. Fortunately, this CTF didn't promote quick solves, so it only counted whether you were able to submit the flag or not. For me, it is a perfect approach because it is much less stressful knowing you can deal with your personal life and come back "loosing" only time (and because you can do nothing about it yet, I'm trying to neglect that).

%%[follow-cta]

# Challenge: postgres

> You have access to the Postgres console. Just read the flag from flag table.

![2022-10-15-12-26-12.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665941561757/0vrcyS3oT.png align="left")

**Difficulty**: hard (429/500 points)  
**Given**: telnet service, one request per session/connection with 5s timeout

Very pleasant task, that description sounded obviously too easy. 
```sql
>> select * from flag;
( 'Close', 'but not there yet')
```

So, my next step was to try to enumerate as much as possible about this database to find the other tables or maybe some metadata about the tables. Because of the way how the task was written, my idea was that **that** `flag` table is in the other schema, or flag is in the  `flag` table metadata. Unfortunately, during the recon, I've discovered that I cannot easily read the names of databases and tables - which means I cannot also read the names of the columns. So, I've tried to look in other ways.

```sql
>> select * from pg_database;
permission denied for table pg_database

>> select * from pg_tables;
permission denied for view pg_tables

>> select * from current_schema;
('public',)

>> select * from current_schemas(true);
(['pg_catalog', 'public'],)

>> SHOW ALL;
('allow_in_place_tablespaces', 'off', 'Allows tablespaces directly inside pg_tblspc, for testing.')

>> SHOW search_path
('"$user", public',)
```
Then I've stumbled upon [this](https://dba.stackexchange.com/a/246996) answer on Stack Exchange, and voilà.

```sql
>> SELECT to_json((SELECT t FROM public.flag t LIMIT 1))
({'val1': 'Close', 'CTF_a2*****************************4dd89': 'but not there yet'},)
```

Table `flag` has flag as a title on one of its columns.

**Reading**:
* https://www.postgresqltutorial.com/postgresql-administration/postgresql-show-tables/

# Challenge:  deobf

> Deobfuscate the code, or call appropriate function after executing it, to get the flag.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666042786807/UZRuWxwm9.png align="left")

**Difficulty**: medium (275/500 points)  
**Given**: obfuscated JS code

These I like. I've watched many times [John Hammond](https://www.youtube.com/watch?v=mhOWdH2zwMk) dealing with ransomware stagers/payloads, and I've done similar challenges before. But man, this one gave me a headache. JavaScript tends to be less understandable than PowerShell due to its caveats, like clousure and some weird inline arrays within parenthesis calls.
First step I did is obviously to put that stack of meaningless words through https://beautifier.io/ to get "stack of meaningless words" - but prettier.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666043337290/jiUOcI9z9.png align="left")

Dealing with that was really pain in the ass and it took me way too long to realize that according to the task description I can just call the appropriate function to get the flag, so wasting no more time I've slapped a few `console.log(..)` here and there (focusing on values returned from functions) and got the flag in the output.

```js
console.log(window[_0x553b6f(0xca, 'xKir')](_0x553b6f(0xc2, 'rW2u')))
```
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665941939535/9jVyo-C5_.png align="left")

# Challenge:  traversal

> Try using the Path traversal vulnerability

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1666043528883/6Et664Ptq.png align="left")

**Difficulty**: easy (50/500 points)  
**Given**: shared instance web application and partial source code

Description and source code made that challenge pretty trivial.

```csharp
[Route("")]
public class HomeController : Controller
{
    [HttpGet("/download")]
    public async Task<IActionResult> Download([Required] string filename)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest();
        }
        if (filename.Contains(".."))
        {
            return BadRequest();
        }
        var model = new HomeModel();
        var fullPath = Path.Combine(model.BaseDirectory, filename);
        var file = await System.IO.File.ReadAllBytesAsync(fullPath);

        return File(file, System.Net.Mime.MediaTypeNames.Text.Plain);

    }
    public IActionResult Index()
    {
        return View(new HomeModel());
    }

}
```
I know that when the second argument in `Path.Combine` is rooted path, the first one is ignored ([source](https://learn.microsoft.com/en-us/dotnet/api/system.io.path.combine?view=net-6.0). This and similar working methods can be found in other languages, environments or their versions, so it is always good to verify that, even if that means executing it blindly.  
Also, doing it that way I bypass the double-dot (`..`) filter.

So, when the first (or last, depending how to look at it) part was done: now's the time on determining how to make the server execute the `Path.Combine` with the `/tmp/flag`. I have fired up the Burp, intercept the GET request and played a bit with it in the repeater. Fortunately, simple URL encoding was enough, so I've got the flag.


![2022-10-15-23-07-39.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942203007/k56cz5kKg.png align="left")

# Challenge:  math-basic

> To get the flag automate sending answers to simple math questions

![2022-10-15-08-55-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942425345/FTX035B-g.png align="left")

**Difficulty**: easy (50/500 points)  
**Given**: telnet server, 15 seconds session

This is "more coding, less hacking" type of challenge that resolves to reading lines with `telnetlib` (or other, see the [gynvael's solution](https://gynvael.coldwind.pl/?id=756)) library and send solves accordingly. I've used parts of the script from [Google 2021 challenge](https://blog.cyberethical.me/google-ctf-2021-filestore#exploit), [perfect square](https://www.mathsisfun.com/definitions/perfect-square.html) definition and [prime number searching algorithm](https://geekflare.com/prime-number-in-python/).

![2022-10-15-10-38-13.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665942440392/_DvLAKaxV.png align="left")

```py
#!/usr/bin/env python3`

import string
import math
from telnetlib import Telnet

host = "192.46.238.159"
port = 1337

def tcSend(tc, msg):
    tc.write(msg.encode('ascii') + b"\n")

def tcReadUntil(tc, end):
    return tc.read_until(end.encode('ascii'), 5)

def tcReadNextTask(tc):
    lines = []
    try:
        tcOutput = tcReadUntil(tc, ">>")
        lines = tcOutput.splitlines()
        return lines[-2]
    except:
        print(lines)
        return ''

# https://geekflare.com/prime-number-in-python/
def isPrime(n):
  for i in range(2,int(math.sqrt(n))+1):
    if (n%i) == 0:
      return False
  return True

def solveTask(text):
    tokens = [x.decode('ascii') for x in text.split()]
    try:
        if 'perfect' in tokens:
            n = int(tokens[1])
            return math.sqrt(n).is_integer()
        if 'prime?' in tokens:
            n = int(tokens[1])
            return isPrime(n)
        if 'divisible' in tokens:
            n1 = int(tokens[1])
            n2 = int(tokens[4][:-1])
            return n1 % n2 == 0
        else:
            return print(tokens)
    except Exception as e:
        print(tokens)
        print(e)

if __name__ == "__main__":
    accepted_characters = ''
    with Telnet(host, port) as tc:
        while True:
            taskText = tcReadNextTask(tc)
            if taskText == '':
                break
            answer = 'Y' if solveTask(taskText) else 'N'
            tcSend(tc, answer)

```


# Additional readings

%%[join-cta]